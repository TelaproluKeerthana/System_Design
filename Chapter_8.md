# Chapter 8: DESIGN A URL SHORTENER

A URL shortener takes long URLs and generates short, unique aliases that redirect users back to the original URL.  
Well-known examples: **Bitly, TinyURL, goo.gl (deprecated)**.  

At first glance, it looks like a small system. But at **scale (100M+ URLs/day)**, it involves **database design, caching, scalability, fault tolerance, and hashing/collision handling**.  

---

## Step 1: Understanding the Problem and Clarifying Questions

Before diving into design, ask clarifying questions:

- **Traffic volume**: How many short URLs do we need to support per day?  
  → Requirement: **100 million new URLs/day**  

- **Allowed characters for short URLs**:  
  → Typically alphanumeric (0–9, a–z, A–Z), total = **62 characters**  

- **Operations**: Are updates/deletes allowed?  
  → For simplicity: **No updates or deletions** (immutable once created)  

- **Core functionalities**:
  1. Shorten a long URL → generate a short URL  
  2. Redirect short URL → long URL  
  3. Ensure **scalability, high availability, fault tolerance**  

---

## Step 2: Traffic & Storage Estimates

- **Write operations**:
Daily writes = 100,000,000
Seconds per day = 24 * 3600 = 86,400
Writes per second = 100,000,000 / 86,400 ≈ 1160 writes/sec

- **Read operations**:
Assume read:write ratio = 10:1
Reads/sec = 1160 * 10 = 11,600 reads/sec

- **Long-term storage**:
10 years = 100M/day * 365 * 10 ≈ 365 billion URLs
Avg URL length = 100 characters
Storage = 365B * 100 bytes ≈ 36.5 TB

---

## Step 3: High-Level Design

### API Endpoints
- **Shorten URL**  
- `POST /api/v1/data/shorten`  
- Request: `{ "longUrl": "https://example.com/some/very/long/path" }`  
- Response: `{ "shortUrl": "https://tiny.com/abc123" }`

- **Redirect URL**  
- `GET /{shortKey}`  
- Example: `GET /abc123` → returns 301/302 redirect to original URL  

### Redirect Strategy
- **301 Permanent Redirect**  
- Browser caches the redirect → reduces load on servers  
- Best when short-to-long mapping never changes  

- **302 Temporary Redirect**  
- Request always hits the shortening service first  
- Useful for **tracking clicks and analytics**  
- Slightly higher load due to no browser caching  

| Aspect         | Option 1 (301)       | Option 2 (302)     |
|----------------|----------------------|---------------------|
| Performance    | Faster, browser cache| Slower, always hits server |
| Analytics      | Limited              | Full tracking       |
| Use case       | Permanent mapping    | Tracking + analytics|

---

## Step 4: Deep Design Components

### 1. Data Model
Relational DB schema:
id | shortUrl | longUrl | createdAt | expiration(optional)
- `id` = primary key (autoincrement or UUID)  
- `shortUrl` = unique short key  
- `longUrl` = original full URL  
- Indexes on `shortUrl` for fast lookup  

For **365B records**, a single DB won’t scale → we need **sharding and replication**.

---

### 2. Shortening Algorithm

Two main approaches:

#### **Base62 Encoding (Preferred)**  
- Generate a unique integer ID (auto-increment, distributed ID generator like Snowflake, etc.)  
- Encode ID into base62 → [0-9, a-z, A-Z]  
- Example:  
ID = 125
Base62(125) = "cb"
Short URL = tiny.com/cb

- Advantages:  
- Deterministic, fast, collision-free  
- 62^7 ≈ 3.5 trillion combinations → enough for decades of usage  

#### **Hashing (Alternative)**  
- Use MD5/SHA1 on long URL → take first 7 chars as short key  
- Must check DB for collisions  
- If collision → retry with salt until unique  
- Optimizations: **Bloom filters** can reduce DB lookups  

**Comparison**:  
- Base62 → clean, no collisions, simple  
- Hashing → better distribution but requires collision handling  

---

### 3. Redirect Flow
1. User clicks short URL  
2. Request goes to **Load Balancer → API Server**  
3. Server checks **Cache (Redis)**  
 - HIT: return long URL (fast)  
 - MISS: query DB → update cache → return redirect  
4. If not found → return **404 Not Found**  

---

## Step 5: Scaling Considerations

- **Caching**:  
- Use Redis/Memcached for frequently accessed short URLs  
- LRU eviction to manage memory  
- Very high cache hit ratio expected (popular URLs dominate)  

- **Database Scaling**:  
- Shard based on `shortUrl` prefix or hash  
- Replication (leader-follower) for read-heavy workloads  
- Periodic archiving of inactive URLs to cold storage  

- **Rate Limiting**:  
- Prevent abuse (spam, bots)  
- Enforce limits per IP or user  

- **High Availability**:  
- Multi-region deployment  
- Load balancing + failover across replicas  

- **Analytics Integration**:  
- Track clicks, geo, device, referrer  
- Don’t block redirect flow → log asynchronously (Kafka + OLAP DB)  

---

## Step 6: Gotchas & Trade-offs

- **301 vs 302** → performance vs analytics  
- **Storage cost** → must plan for replication and indexing overhead  
- **Hot URLs** → popular short links may overwhelm DB without caching  
- **Expiration Policy** → allow deleting/archiving old unused links  
- **Consistency vs Availability** →  
- Eventual consistency is acceptable for redirects  
- Strong consistency needed for short URL creation (no duplicates)  

---

## Step 7: Extended Discussion Points (Interview Tips)

If time allows, mention:  
- **Rate limiter**: Protects against abuse and spammy bulk requests  
- **Web server scaling**: Horizontal autoscaling with load balancer  
- **Database scaling**: Sharding, replication, partitioning  
- **Analytics**: Store in separate system (e.g., Kafka → Snowflake)  
- **Security**: Prevent malicious/phishing URLs, validate domains  
- **Monitoring**: Metrics for latency, error rate, cache hit ratio  

---

## Final Thoughts

A URL shortener seems simple, but **at scale (100M/day, 365B total over 10 years)** it requires careful consideration of:

- Traffic estimates & storage  
- Key generation strategies (Base62 vs Hashing)  
- Database sharding & caching  
- Redirect strategies (301 vs 302)  
- High availability & fault tolerance  
- Rate limiting & abuse prevention  
- Analytics & monitoring  

This system design balances **simplicity, scalability, and performance** while leaving room for analytics and business insights. 
