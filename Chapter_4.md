A rate limiter is a system component that restricts the number of client requests allowed within a specified time window. Any request exceeding the allowed threshold is blocked or throttled.

Rate limiting is crucial in large-scale systems to:

✅ Prevent resource starvation and protect APIs from Denial of Service (DoS) attacks.

✅ Reduce infrastructure costs by controlling unnecessary or excessive traffic.

✅ Optimize third-party API usage, especially when billed per request.

✅ Prevent server overloads caused by bots or malicious actors.

✅ Ensure fair usage among all users of a system.

🧠 Step 1: Understand the Problem and Define Scope


Ask the following key design questions:

Do we need client-side or server-side rate limiting?

What is the expected number of users and request volume?

How should throttled users be informed (e.g., HTTP 429 status)?

Is the system distributed? Should the rate limiter be a shared service?

Should rate limits vary by API, by user/IP, or globally?

🏗️ Step 2: High-Level Design


📍 Where to Place the Rate Limiter
Client-Side Rate Limiting:
Generally not recommended. Clients can be tampered with or forged. Malicious users may bypass these limits.

Server-Side Rate Limiting (API Layer):
Most common and secure. Rate limiting logic is implemented at the API server or via middleware.

Middleware or Gateway-Based Rate Limiting:
Effective in microservice or distributed environments. This allows centralized enforcement before traffic reaches internal services.

🔁 Step 3: Choose a Rate Limiting Algorithm


Different algorithms offer trade-offs in accuracy, memory, and performance:

| **Algorithm**              | **Description**                                                         | **Use Case**                      |
| -------------------------- | ----------------------------------------------------------------------- | --------------------------------- |
| **Token Bucket**           | Allows bursts; tokens are added periodically and consumed per request   | Most flexible, real-time APIs     |
| **Leaky Bucket**           | Requests processed at a constant rate; overflow is discarded or delayed | Smooth, consistent rate control   |
| **Fixed Window Counter**   | Counts requests in fixed intervals (e.g., 1 min)                        | Simple, fast, but prone to spikes |
| **Sliding Window Log**     | Logs each request with timestamps; checks against a rolling window      | Accurate but memory intensive     |
| **Sliding Window Counter** | Hybrid of fixed window + averaging over sliding time                    | More accurate than fixed window   |


💧 Token Bucket Algorithm


The Token Bucket algorithm is commonly used due to its flexibility and efficiency.

A bucket with a fixed capacity holds "tokens".

Tokens are added at a regular interval (refill rate).
[]
Each request consumes one token.

If a request arrives and no tokens are available, it is rejected or delayed.

Allows bursts as long as tokens are available, but enforces a steady rate long-term.

⛽ Use Case Consideration
Decide how many buckets are needed based on use cases:

Example: If a system must allow 5 friend requests/day and 10 messages/minute, use two separate buckets — one per rule.

✅ Pros


Simple to Implement: Straightforward logic, widely supported in libraries and frameworks.

Memory Efficient: Requires only counters and a timestamp for refill logic.

Supports Bursts: Unlike Leaky Bucket, it allows occasional high-traffic bursts.

Flexible Configuration: Adjustable bucket size and refill rate enable fine-tuned control.

Real-Time Enforcement: Immediate decision on whether to accept or reject a request.

❌ Cons


Tuning Complexity: Choosing the right refill rate and bucket size depends on expected traffic, making configuration tricky.

Time Synchronization: In distributed systems, maintaining accurate time or using centralized storage (e.g., Redis) adds complexity.

Not Ideal for Hard Limits: If exact request limits per interval are needed, sliding window counters may offer stricter enforcement.
