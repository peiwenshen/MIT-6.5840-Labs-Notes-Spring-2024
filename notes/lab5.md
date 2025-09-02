# 6.5840 Lab 5: Sharded Key/Value Service

### Preface
I consider this lab the most difficult of the series. In terms of difficulty, I would rank them: **Lab 5 > Lab 3 >> Lab 4 = Lab 2 = Lab 1**.

Lab 4 focuses on building a service on top of a single Raft cluster, while Lab 5 is about building a service across multiple Raft clusters.

There was a gap of several months between finishing Lab 5A and finally tackling the rest. During that time, I often tried to convince myself that completing nine labs versus ten wouldn’t make much difference. But in the end, with a *“finish what you start"* mindset, I picked it back up last week, spent an entire week on it, and finally completed it yesterday. I planned to run 1,000 tests — and at the time of writing this note, it had already reached test #379. It turned out to be more stable than I expected; I thought I would need another long debugging session. Hopefully, it can run all the way through.

![截圖 2025-07-07 下午6.33.32](https://hackmd.io/_uploads/S1A5M7YBel.png)



### Pitfall
> If you put a map or a slice in a Raft log entry, and your key/value server subsequently sees the entry on the applyCh and saves a reference to the map/slice in your key/value server's state, you may have a race. Make a copy of the map/slice, and store the copy in your key/value server's state. The race is between your key/value server modifying the map/slice and Raft reading it while persisting its log.

I initially misunderstood this point — I thought doing a deep copy before sending the entry into Raft would be sufficient.

### Review & Details
Looking back, three key implementation points stand out:

1. **Mismatch issue carried over from Lab 4:**  
   I initially assumed that after writing to Raft and then applying it on the same machine, the `commandIndex` and `commandTerm` would always match.  
   However, if there’s a disconnection or a shard migration, a new write request may reach the state machine with mismatched index/term.  
   The eventual solution was: since `requestID` only ever increases, just record the latest one. If the request is the newest, treat it as valid even if index/term mismatch occurs.

2. **Embed shard ID directly into all state maps:**  
   ```go=
   mapKV        map[int]map[string]string       // shardID → key → val
   latestReqId  map[int]map[int64]int32         // shardID → clientID → latestReqID
   requestCache map[int]map[int64]RequestResult // shardID → clientID → result
   ```
    Doing this saves a lot of trouble and complexity when managing sharded state.
3.	All state changes should happen inside apply:
    At first, I tried to handle shard-pulling (deciding which shard to fetch next) outside of the apply process, but it caused lots of confusion. Eventually, I realized it’s cleaner and safer to handle all state transitions strictly within apply.