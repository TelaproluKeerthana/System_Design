# Chapter 9: Design a Web Crawler  

## 📖 Overview  
A **web crawler** (also known as a spider or bot) is a program that systematically browses the internet to discover and collect new or updated content. Crawlers form the backbone of search engines, archiving services, and many monitoring applications.  

At a high level, a crawler:  
1. Starts with a list of **seed URLs**.  
2. Downloads the corresponding web pages.  
3. Extracts all hyperlinks on those pages.  
4. Adds new links to the **URL frontier** (queue of URLs to visit).  
5. Repeats the process until coverage goals are met.  

---

## 🚀 Use Cases  
- **Search Engine Indexing** – updating search engine databases with fresh content.  
- **Web Archiving** – preserving snapshots of the internet for future use.  
- **Web Mining** – extracting knowledge, e.g., financial institutions mining annual reports.  
- **Web Monitoring** – detecting copyright/trademark violations or brand misuse.  

---

## ⚙️ Core Algorithm  
1. **Input**: a set of seed URLs.  
2. **Download**: fetch web pages from these URLs.  
3. **Extract**: identify new hyperlinks within downloaded content.  
4. **Enqueue**: add new URLs to the frontier (to-be-visited queue).  
5. **Repeat** until no new URLs remain or resource limits are reached.  

---

## ❓ Key Interview Questions to Clarify Requirements  
- **Purpose**: search indexing, mining, or monitoring?  
- **Scale**: how many pages per month? (e.g., 1B pages).  
- **Content Types**: HTML only, or include PDFs, images, videos?  
- **Freshness**: Should we re-crawl updated/edited pages?  
- **Retention**: How long do we store crawled content? (e.g., 5 years).  
- **Deduplication**: Should duplicate pages be ignored? (Yes).  

---

## 🌐 Design Characteristics  
- **Scalability** – the web is massive; crawlers must parallelize tasks across servers.  
- **Robustness** – handle invalid HTML, server crashes, and failed requests gracefully.  
- **Politeness** – avoid overwhelming servers (rate-limit requests, respect `robots.txt`).  
- **Extensibility** – support new content types (PDFs, images) without redesigning.  

---

## 🧩 Major Components  

### 1. Seed URLs  
- Starting points for the crawl process.  
- Example: for a university website, start with its domain.  
- Strategy: choose based on **topic**, **region**, or **popularity**.  

### 2. URL Frontier  
- Maintains two sets:  
  - URLs **to be downloaded** (queue).  
  - URLs **already visited**.  
- Implemented as a **FIFO queue** with added support for:  
  - **Politeness** (delay requests to the same server).  
  - **Prioritization** (important URLs first).  
  - **Freshness** (frequently updated pages get re-crawled).  
- **Storage approach**: hybrid of in-memory buffers + disk persistence.  

### 3. HTML Downloader  
- Fetches pages via HTTP/HTTPS.  
- Must:  
  - Respect **robots.txt** (rules set by website owners).  
  - Use **timeouts** to avoid waiting on slow/unresponsive servers.  
  - Handle retries and exceptions.  

### 4. DNS Resolver  
- Converts domain names → IP addresses.  
- Bottleneck if repeated often → use **DNS caching** for performance.  

### 5. Content Parser  
- Parses HTML into a structured form.  
- Extracts links, cleans content, and validates against malicious/invalid input.  
- Isolated as a separate component to avoid slowing down other modules.  

### 6. Content Seen (Duplicate Detection)  
- Prevents storing duplicate content.  
- Approaches:  
  - **Hashing** HTML content.  
  - Compare hash values for fast deduplication.  

### 7. Content Storage  
- Stores crawled pages.  
- Choices depend on:  
  - Size of data.  
  - Access frequency.  
  - Lifespan (e.g., retain 5 years).  
- Strategy:  
  - **Memory**: frequently accessed content.  
  - **Disk**: long-term archival storage.  

### 8. URL Extractor  
- Identifies new links from downloaded HTML.  
- Resolves **relative paths** into full URLs (e.g., `/about` → `https://example.com/about`).  

### 9. URL Filter  
- Excludes unwanted URLs:  
  - Blocklisted domains.  
  - File extensions (e.g., `.jpg`, `.exe`).  
  - Non-HTML resources (if out of scope).  

### 10. URL Seen  
- Tracks already visited URLs.  
- Prevents:  
  - Re-downloading the same content.  
  - Infinite loops (e.g., dynamically generated links).  

---

## 🔍 System Deep Dive  

### Traversal Strategy  
- **DFS**: goes deep into one site, risks missing breadth.  
- **BFS**: better coverage, but can overwhelm servers.  
- **Priority BFS**: enhance with ranking signals (PageRank, traffic, update frequency).  

### URL Frontier Enhancements  
- **Politeness**:  
  - Delay multiple requests to the same host.  
  - Avoids DoS-like behavior.  
- **Priority**: rank URLs by importance.  
- **Freshness**: periodically re-crawl updated pages.  

### HTML Downloader  
- Must consult **robots.txt** before downloading:  
  ```txt
  User-agent: Googlebot  
  Disallow: /private/

- Some sites disallow entire sections.

- Crawlers should obey to avoid bans.

### Performance Optimizations

- Distributed Crawling – split URL space across multiple servers.

- Threaded Downloaders – multiple threads per server.

- DNS Caching – reduce lookup latency.

- Short Timeouts – skip unresponsive servers.

- Locality-Aware Crawling – deploy crawling servers worldwide.

### Robustness

- Consistent Hashing – balance load when adding/removing servers.

- Checkpointing States – save crawl progress to resume after crashes.

- Graceful Exception Handling – recover from failures without halting.

- Validation – ensure pages are well-formed before storage.

### Detecting Problematic Content

- Duplicate Content – detect via hashes/checksums.

- Spider Traps – avoid infinite URL spaces (e.g., calendars, autogenerated pages).

- Data Noise – filter ads, spammy pages, irrelevant links.

## ✅ Summary

A scalable web crawler requires a careful balance of:

    Efficiency – distributed, parallel crawling.

    Politeness – respect for server resources (robots.txt, rate limits).

    Robustness – crash tolerance, duplicate handling, error recovery.

    Extensibility – easy adaptation to new content types.

By combining intelligent URL management, high-performance downloading, careful storage, and strict deduplication, a crawler can handle billions of pages per month while staying efficient and respectful of the web ecosystem.
