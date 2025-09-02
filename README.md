# MIT 6.5840 Distributed Systems Labs — Spring 2024

This repository contains my personal notes and reflections on the programming labs from MIT 6.5840: Distributed Systems, a graduate-level course that explores the principles behind fault tolerance, replication, and scalability in distributed systems.

I independently completed all five labs and wrote supplemental notes for Labs 1, 2, 3, and 5.
Note: Due to the course’s academic integrity policy, I am not permitted to publish or share the actual lab code.

---

## 📌 Labs Overview
All labs were completed using Go and verified with the official MIT 6.5840 test harness.  
In addition, I ran extended stress tests (1,000+ iterations) to ensure stability, since certain bugs only surface rarely.

### Lab 1: MapReduce
- Implemented a simplified single-machine MapReduce framework.  
- Handled task scheduling and crash recovery.  
- Ensured correctness of intermediate files by leveraging Go’s atomic `os.Rename` operation.  
- [Notes →](./notes/lab1.md)

### Lab 2: Key/Value Server
- Built a basic key-value store with unreliable RPC.  
- Added client request IDs to ensure idempotency and exactly-once semantics.  
- [Notes →](./notes/lab2.md)

### Lab 3: Raft
- Implemented the Raft consensus protocol.  
- Features include leader election, log replication, and persistence.  
- [Notes →](./notes/lab3.md)

### Lab 4: Fault-tolerant Key/Value Service
- Combined Lab 2 and Lab 3 to build a Raft-backed KV store.  
- Provided fault tolerance with exactly-once semantics.  
- (No separate notes — this lab mainly integrated earlier components.)

### Lab 5: Sharded Key/Value Service
- Extended the Raft-based KV store to support sharding and reconfiguration.  
- Implemented a shard controller, shard migration, and safe reconfiguration.  
- [Notes →](./notes/lab5.md)

---

## 🎯 Key Takeaways
- Gained hands-on experience with **distributed systems fundamentals**:
  - Consensus protocols (Raft)  
  - Fault tolerance and recovery  
  - Replicated services with exactly-once semantics  
  - Sharding and dynamic reconfiguration  
- Extended my learning beyond the labs by studying classic papers (e.g., GFS, MapReduce) and course lectures, which deepened my understanding of the **trade-offs and design principles** in real-world systems.  
- Writing detailed notes for multiple labs reinforced not only correctness but also the reasoning behind key design choices.
