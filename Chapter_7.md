# Chapter 7: Design a Unique ID Generator in Distributed Systems

In distributed systems, generating **unique IDs** is challenging because there is no single centralized system like in traditional databases with auto-increment primary keys.

---

## Step 1: Define Requirements

An effective ID generator must satisfy the following:

- IDs must be **globally unique**
- IDs are **numeric only**
- IDs fit within a **64-bit integer**
- IDs are **time-ordered** (newer IDs > older IDs)
- Capable of generating **10,000+ unique IDs per second**

---

## Step 2: Explore High-Level Approaches

Several solutions can be considered:

### 1. Multi-Master Replication
- Uses auto-increment with an **increment step = number of servers**
- Example: Server A generates IDs `1, 4, 7…` and Server B generates `2, 5, 8…`
- **Cons:**
  - Difficult to scale across multiple data centers
  - IDs may not be evenly time-ordered across servers

---

### 2. UUID (Universally Unique Identifier)
- 128-bit values, commonly used for uniqueness
- Each server can generate IDs independently → **no synchronization needed**
- **Pros:**
  - Scales easily, avoids conflicts
- **Cons:**
  - Not numeric (contains alphanumeric data)
  - Exceeds 64-bit requirement
  - IDs are not time-ordered

---

### 3. Ticket Server
- A centralized service issues sequential IDs to clients
- **Pros:**
  - Generates numeric IDs
  - Simple to implement
- **Cons:**
  - **Single point of failure** if only one ticket server exists
  - Can become a performance bottleneck
  - Multiple ticket servers require careful coordination

---

### 4. Twitter Snowflake
- A **widely adopted solution** for scalable unique ID generation
- Generates a 64-bit integer composed of the following parts:

| Bits | Field          | Description |
|------|---------------|-------------|
| 1    | Sign bit       | Always 0 (positive number) |
| 41   | Timestamp      | Milliseconds since a custom epoch (e.g., Twitter Epoch) |
| 5    | Datacenter ID  | Identifies the data center |
| 5    | Machine ID     | Identifies the machine within a data center |
| 12   | Sequence No.   | Counter for IDs generated within the same millisecond |

- **Pros:**
  - Fits within 64 bits
  - Time-ordered IDs
  - No central bottleneck → distributed & scalable

---

✅ **Conclusion:**  
While multi-master replication, UUIDs, and ticket servers work in certain contexts, **Twitter’s Snowflake algorithm** strikes the best balance for distributed systems requiring **scalable, numeric, 64-bit, time-ordered unique IDs**.
