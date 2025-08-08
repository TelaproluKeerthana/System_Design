# Chapter 5: Consistent Hashing

Consistent hashing is a widely used technique to **distribute requests across multiple servers** while minimizing data movement when servers are added or removed.

---

## 1. The Rehashing Problem

When using a simple modulo-based distribution like:

server_index = hash(key) % total_servers


- Data is evenly spread **only while the number of servers stays constant**.
- If a server **fails** or is **added**, `total_servers` changes.
- This causes **massive remapping** of keys, resulting in:
  - **Cache misses**
  - Increased latency (due to fetching from the wrong server)
  
---

## 2. What is Consistent Hashing?

**Definition:**  
Consistent hashing ensures that **only a small fraction** of keys (`k/n`) need to be remapped when servers change, instead of **all keys**.

- **k** = number of keys
- **n** = number of slots (servers)

The main idea:  
Rather than directly mapping keys to servers, both keys and servers are placed on a **hash ring**.

---

## 3. Hash Ring

1. **Hash Space**  
   - For a secure hash function (e.g., SHA-1), the output space is `0` to `2^160 - 1`.
2. **Circular Arrangement**  
   - Arrange this space into a circle → **hash ring**.

**Diagram:**  
[Server 1] ---- [Key A] ---- [Server 2] ---- [Key B] ---- [Server 3] ---- [Key C] ---- (back to Server 1)

---

## 4. Mapping Keys to Servers

1. **Hash Servers** – Map each server’s ID (e.g., IP address) into the ring.
2. **Hash Keys** – Map each key into the ring.
3. **Server Lookup** – For a given key:
   - Move **clockwise** on the ring until the next server is found.
   - That server stores the key.

---

## 5. Adding and Removing Servers

### Adding a Server
- Only keys between the **previous server** and the **new server** get remapped.
- Example: If there are 3 servers and 4 keys, adding **Server 4** causes only the closest keys to be reassigned.

### Removing a Server
- Keys mapped to the removed server are reassigned to the **next server clockwise**.

---

## 6. Issues with Basic Approach

1. **Uneven Partition Sizes**  
   - Adding/removing servers can cause some partitions to become large.
2. **Non-Uniform Key Distribution**  
   - Keys may cluster in certain partitions, creating load imbalance.

---

## 7. Virtual Nodes (VNodes)

**Definition:**  
A server can be represented by **multiple points** (virtual nodes) on the hash ring.

- Each virtual node is treated as a separate server in hashing.
- Results in **more uniform distribution**.
- Standard deviation of load decreases as the number of virtual nodes increases.

**Trade-off:**  
Requires extra storage for mapping virtual nodes.

---

## 8. Finding Affected Keys (With Virtual Nodes)

- When a **new server** is added:
  - Keys between the **previous virtual node** and the **new server's virtual node** are reassigned.
- Reduces the number of moved keys compared to basic hashing.

---

## 9. Summary Table

| Feature                     | Basic Hashing (`% total_servers`) | Consistent Hashing | Consistent Hashing + VNodes |
|-----------------------------|------------------------------------|--------------------|-----------------------------|
| Key Movement on Server Change | High (~100%)                     | Low (~k/n)         | Very Low                    |
| Load Distribution           | Even (until change)               | Possibly uneven    | Uniform                     |
| Complexity                  | Low                                | Medium             | Medium-High                 |

---

## 10. References
- [Amazon Dynamo Paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [Consistent Hashing - MIT 6.824 Notes](https://pdos.csail.mit.edu/6.824/papers/chash.pdf)
- [YouTube - Consistent Hashing Visualization](https://www.youtube.com/watch?v=zaRkONvyGr8)
