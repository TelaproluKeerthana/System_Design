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

Data Replication

    Data is replicated across N servers (N is configurable).
    
    Replicas are placed in different data centers for disaster recovery.
    
    Replication improves availability and fault tolerance.
    
    Replication is usually asynchronous for performance.

Consistency

Since data is distributed across multiple replicas, all of them must be synchronized.

Quorum consensus helps ensure consistency:

    W → Write quorum (number of replicas that must acknowledge a write).
    
    R → Read quorum (number of replicas to check during a read).
    
    If W + R > N, strong consistency is guaranteed.

Consistency models:

    Strong consistency → Always read the latest value.
    
    Weak consistency → Reads may return stale data.
    
    Eventual consistency → Given enough time, all replicas converge.

Inconsistency Resolution

    Versioning: Each update creates a new immutable version.
    
    Vector clocks: Store [server, version] pairs to track causality.
    
    Conflicts can be detected and resolved by comparing versions.

Handling Failures

Failure Detection

    Simple heartbeats are unreliable in large clusters.
    
    Gossip protocol:
    
          Nodes periodically exchange membership and heartbeat info.
          
          Consensus from multiple peers is used to mark a node as down.

Temporary Failures

    Sloppy quorum: Write/read from the first available nodes instead of strict quorum.
    
    Data is later replicated back to the original node once it recovers.

Permanent Failures

    Anti-entropy protocols (using Merkle Trees) compare replicas and sync differences.
    
    Merkle tree: Each non-leaf node stores a hash of its children → allows efficient detection of differences without scanning all data.

System Architecture  

    flowchart TD
    Client[Client] --> API[API Layer]
    API --> Coordinator[Coordinator Node]
    Coordinator --> Node1[Storage Node 1]
    Coordinator --> Node2[Storage Node 2]
    Coordinator --> Node3[Storage Node 3]
    Node1 <--> Node2
    Node2 <--> Node3
    Node1 <--> Node3

    Clients talk to the system through APIs.
    
    Coordinator node routes requests.
    
    Each storage node is equal (decentralized, no single point of failure).
    
    Data is replicated across multiple nodes.
    
    Nodes can be added/removed automatically.


Write Path

sequenceDiagram

    participant C as Client
    participant Co as Coordinator
    participant CL as Commit Log
    participant MC as MemTable (In-Memory)
    participant ST as SSTable (Disk)

    C->>Co: Write Request (key, value)
    Co->>CL: Append to Commit Log
    Co->>MC: Write to MemTable
    MC->>ST: Flush to SSTable when full
    Co-->>C: Acknowledge Write


Read Path

sequenceDiagram
   
    participant C as Client
    participant Co as Coordinator
    participant MC as MemTable (In-Memory)
    participant BF as Bloom Filter
    participant ST as SSTable (Disk)

    C->>Co: Read Request (key)
    Co->>MC: Check in MemTable
    alt Found in MemTable
        MC-->>Co: Return Value
    else Not Found
        Co->>BF: Check Bloom Filter
        BF->>ST: Lookup Key in SSTable
        ST-->>Co: Return Value
    end
    Co-->>C: Send Value




  

