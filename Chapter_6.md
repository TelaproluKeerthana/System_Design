Chapter 6: Design a Key-Value Store

A key-value store is like a key-value database where keys are uniquely identified — they can be hashed keys or text keys. For performance reasons, shorter keys tend to work more efficiently.

  Understanding the requirements:
    
    Size of a key-value pair is small (less than 10 KB).
    
    Ability to store large amounts of data.
    
    High availability: System responds quickly even during failures.
    
    High scalability: able to handle large datasets.
    
    Automatic scaling: servers should be added or removed automatically based on traffic.
    
    Tunable consistency.
    
    Low latency.

Single Server Key-Value Store:

If designed for a single server, it can be as simple as a HashMap storing key-value pairs. However:

    Memory is a limitation — not all data can fit in memory.
    
    Use a combination of cache + disk storage.
    
    As data grows beyond storage capacity, management becomes difficult.
    For large-scale systems, a distributed key-value store is a better approach.

Distributed Key-Value Store:

Distributes key-value pairs across multiple servers to achieve scalability and fault tolerance.

CAP Theorem:

A distributed system can only guarantee two of the three:

    Consistency (C): All clients see the same data regardless of the node they connect to.
    
    Availability (A): Every request receives a response (success or failure), even during node failures.
    
    Partition Tolerance (P): System continues to operate despite network partitions.

Trade-offs:

    CA systems are not realistic under real-world network partitions — in practice, you must choose CP or AP.
    
    For banking apps → prioritize Consistency (CP).
    
    For social media feeds → prioritize Availability (AP), tolerating slightly stale data.

Core Components & Techniques:

    Data partition
    
    Data replication
    
    Consistency
    
    Inconsistency resolution
    
    Handling failures
    
    System architecture diagram
    
    Write path
    
    Read path

Data Partition:

    Data cannot be stored on a single server must be split across multiple servers.
    
    Consistent hashing is preferred for distributing keys across servers, minimizing data movement when servers are added/removed.
    
    Supports automatic scaling and heterogeneity (number of virtual nodes proportional to server capacity).

