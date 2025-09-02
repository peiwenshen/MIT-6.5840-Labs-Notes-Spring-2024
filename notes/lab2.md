# 6.5840-golabs-2024 Lab2 Key/Value Server
## Overview
The task is to build a key/value server that satisfies [linearizability](https://pdos.csail.mit.edu/6.824/papers/linearizability-faq.txt).

## Implementation


### Test Results + Spoiler Alert

![image](https://hackmd.io/_uploads/rJppaxfo0.png)

This lab was relatively simpler.  
The core idea is to use a **counter** together with a **map** to avoid duplicate operations, and another **map** to return cached results for past requests in order to maintain linearizability.

### RPC
#### how rpc struct be defined


```go=
type PutAppendArgs struct {
	Key       string
	Value     string
	Op        string // "Put" or "Append"
	ClientID  int64
	RequestID int32
}

type PutAppendReply struct {
	Value string
}

type GetArgs struct {
	Key       string
	ClientID  int64
	RequestID int32
}

type GetReply struct {
	Value string

}
```


#### Server and Client 


##### Server

```go=
type KVServer struct {
	mu sync.Mutex
	mapKV        map[string]string
	requestCache map[int64]string
	latestReqId  map[int64]int32
}
```


##### Client

```go=
type Clerk struct {
	server *labrpc.ClientEnd
	id        int64
	requestId int32
}
```

