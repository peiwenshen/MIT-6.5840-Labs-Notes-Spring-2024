# 6.5840 Lab 3: Raft
There are already many introductions to Raft online, including plenty of implementation details.  
Here I mainly want to share some **things worth paying attention to** — so that future students who get stuck don’t have to read others’ code, but can still pick up useful ideas.

1. **Debug logging:** The earlier and more detailed, the better. You will rely on it heavily later. It’s useful to run multiple instances at the same time, especially since many bugs only show up once in hundreds of runs. Add sanity checks into logs too (e.g., `commitIndex` vs `appliedIndex`).  
2. **Use goroutines extensively:** Run RPCs in their own goroutines. Also move `apply` into a separate goroutine (since channels are blocking). Without this, the system can deadlock or stall.  
3. **`defer` behavior:** Remember it is FIFO and scope-specific.  
4. **Tricky bug with network partitions:** After a disconnection and recovery, RPCs may arrive in a different order than they were sent. For example, receiving an earlier but still valid `AppendEntries` can incorrectly truncate the log.  
5. **Snapshots:** Once you add snapshots, you need to maintain an offset. Writing helper functions to fetch logs makes things much easier.  
6. **Locks:** Every time you release and reacquire a lock, the `term` or `state` may have changed. Always re-check.  
7. **Go slice + GC:** Ensure there are no references left, otherwise GC won’t free memory. Worth looking up details.  
8. **PreLogExist optimization:** There are efficient ways to calculate this (see link below).  

Must-read: [Student’s Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/)