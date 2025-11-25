# Design a News Feed System

A News Feed system displays a dynamic list of posts (text, images, videos) from people a user follows. Examples: **Facebook News Feed**, **Instagram Feed**, **Twitter Timeline**.

---

## 1. Requirements & Design Scope

### 1.1 Clarifying Questions
Before designing, clarify:

1. **Platform**  
   Web, mobile, or both?

2. **Content Types**  
   Text, images, videos, links?

3. **Feed Ordering**  
   Reverse chronological, algorithmic ranking, or hybrid?

4. **Scale**  
   - Average followers per user  
   - Daily Active Users (DAU)  
   - Posts per second  
   - Data retention period  

### 1.2 Functional Requirements
- Users can publish posts  
- View home feed with pagination/infinite scroll  
- Privacy features (mute/block, hidden posts)  
- Real-time updates  
- Low latency feed retrieval (<100ms from cache)

### 1.3 Non-Functional Requirements
- High availability  
- Read-heavy optimization  
- Horizontal scalability  
- Eventual consistency in noncritical flows  
- Durability for all posts  

---

## 2. High-Level Architecture Overview

The system has two core flows:

- **Feed Publishing (Write Path)**  
- **Feed Retrieval (Read Path)**  

![Architecture](#)  

---

## 3. Feed Publishing (Write Path)

Occurs whenever a user creates a new post.

### 3.1 Flow
1. Client → Web Server (auth, rate limit)  
2. Post Service writes post to **Post DB** + **Post Cache**  
3. Fanout Service distributes post to followers  
4. News Feed Cache is updated  

### 3.2 Web Servers
- Authentication (JWT/OAuth)  
- Rate limiting (token bucket / leaky bucket)  
- Input validation  
- Media upload handling  

---

## 4. Fanout Service

Responsible for pushing post IDs to follower feeds.

### 4.1 Fan-Out Strategies

#### **A. Fan-Out-On-Write (Push Model)**
Precomputes each follower’s newsfeed when the post is created.

**Pros**
- Very fast feed reads  
- Real-time updates

**Cons**
- High write amplification  
- Wastes compute for inactive users  
- **Hot Key Problem** with celebrities

---

#### ❗ Hot Key Problem Explained

Occurs when a celebrity (millions of followers) posts something.  
One write → millions of feed updates.

This results in:
- Overloaded database/cache shards  
- High latency  
- CPU/memory exhaustion  
- Retry storms  
- Cascading failures  

**Mitigations**
- Hybrid model  
- Sharded fanout queues  
- Write batching  
- Throttling celebrity writes  
- Move high-follower users to the pull model  

---

#### **B. Fan-Out-On-Read (Pull Model)**
Feed is computed when the user opens the app.

**Pros**
- No wasted computation  
- No hot key issues  
- Lower write load

**Cons**
- Higher read latency  
- Heavy dependency on DB lookups  
- Not ideal for real-time UX  

---

#### **C. Hybrid Model (Used in real systems)**
- Normal users → **Push**  
- Celebrities → **Pull**  
- Reduces hot keys  
- Keeps reads fast for most users  

---

### 4.2 How Fanout Works Internally
1. Fetch follower IDs from the **Graph DB**  
2. Apply privacy/mute/block filters  
3. Send fanout tasks to **Message Queues (Kafka/SQS)**  
4. Workers consume tasks and update **Feed Cache**: feed_cache[user_id] = list<post_ids>



---

## 5. Feed Retrieval (Read Path)

Triggered when a user loads their home feed.

### 5.1 Flow
1. Client → CDN (static assets)  
2. CDN → Web Server → Feed Service  
3. Fetch post IDs from **Feed Cache**  
4. Fetch metadata from:
- User Profile Cache  
- Post Cache  
- Post DB fallback  
5. Merge, enrich, sort  
6. Return JSON feed to client  

### 5.2 Optimizations
- Store only **post IDs** in feed cache (lightweight)  
- Time-based pagination  
- Prefetch next feed batch  
- Deduplicate posts across shards  

---

## 6. Storage Layer

### 6.1 Post Database
Recommended: NoSQL (Cassandra, DynamoDB, MongoDB)

Schema:
post_id, user_id, content, media_url, timestamp, privacy

### 6.2 Graph Database
Stores follower/following relationships.

followers[user] = list<user_ids>
following[user] = list<user_ids>

Options:
- Neo4j  
- MySQL adjacency list  
- Distributed graph store  

### 6.3 Feed Cache
In-memory (Redis/Memcached).


Options:
- Neo4j  
- MySQL adjacency list  
- Distributed graph store  

### 6.3 Feed Cache
In-memory (Redis/Memcached).

feed[user_id] = list<post_ids ordered by timestamp>

### 6.4 Media Storage
- S3 or distributed file storage  
- CDN for global delivery  
- Video transcoding pipeline  

---

## 7. Message Queue Layer

Used for reliable fanout.

- Kafka topics per “post_publish”  
- Workers process batches  
- DLQs for failure handling  
- Sharded queues for scalability  

---

## 8. Ranking & Personalization (Optional Advanced Feature)

### Ranking factors:
- Recency  
- User interactions (likes, comments)  
- Content type preference  
- User affinity score  
- Engagement prediction  

Ranking pipeline:
1. Feature extraction  
2. ML-based scoring  
3. Result blending  
4. Final sorted feed  

---

## 9. Scaling Considerations

### 9.1 Database Scaling
- Vertical scaling (initially)  
- Horizontal scaling via sharding  

### 9.2 SQL vs NoSQL
- SQL: relational, transactions  
- NoSQL: high throughput, scalable → preferred for posts & feeds  

### 9.3 Replication
- Master → Slave replication  
- Read replicas for heavy feed reads  

### 9.4 Consistency Models
- Strong consistency for post creation  
- Eventual consistency for feed updates  

### 9.5 Sharding Strategies
- User ID hash  
- Geographic region  
- Active vs inactive user buckets  

---

## 10. System Design Best Practices

- **Stateless web tier** for easier scaling  
- **Cache aggressively** (feeds, posts, users)  
- **Use message queues** to decouple write path  
- **Support multiple data centers**  
- **Monitor key metrics**:  
  - Feed load latency  
  - QPS (reads & writes)  
  - Cache hit ratio  
  - Queue backlog  
  - Worker utilization  

---

## 11. Example API Endpoints

### **POST /posts**
Creates a new post.

### **GET /feed?user_id=U&cursor=C**
Returns timeline with pagination.

### **POST /follow**
Follow another user.

---

## 12. Final Summary

This News Feed System balances read latency, write performance, and scalability using:

- Hybrid fanout architecture  
- Distributed caching  
- Message queue–based feed fanout  
- NoSQL storage  
- Graph DB for relationships  
- Multi-region deployment  
- Ranking + personalization (optional)

This design model closely resembles production systems at **Facebook, Twitter, Instagram, LinkedIn**, etc.

---


