# 6.5840-golabs-2024 Lab1 MapReduce
## Overview
The main task is to implement a single-machine version of MapReduce.  
The original implementation runs on GFS, and the system model and programming model are shown in the figures below.

![System Model](https://hackmd.io/_uploads/Sk5oUicO0.png)

![Programming Model](https://hackmd.io/_uploads/S1D97NpdC.png)

## Implementation
> You should put your implementation in `mr/coordinator.go`, `mr/worker.go`, and `mr/rpc.go`.

The **coordinator** is responsible for assigning tasks, the **worker** executes `map` and `reduce`, and **rpc** defines the data types exchanged between them.  

Workers are **stateless** and continuously request jobs from the coordinator. All map tasks must finish before reduce tasks can begin. In the original MapReduce (running on GFS), intermediate files need to be moved across machines. In this single-machine version, it is enough to ensure that intermediate files are unique and complete during reduce. Therefore, using `ioutil.TempFile` and `os.Rename` is sufficient to handle file operations without worrying about moving files around.  

A simple state array is used to track job status. For **fault tolerance**, timestamps are recorded at assignment; if a task times out, the state is reset and reassigned to another worker. Thanks to `os.Rename`, we don’t need to worry about file conflicts even when jobs are reassigned.  


```go=
// Rename renames (moves) oldpath to newpath.
// If newpath already exists and is not a directory, Rename replaces it.
// OS-specific restrictions may apply when oldpath and newpath are in different directories.
// Even within the same directory, on non-Unix platforms Rename is not an atomic operation.
// If there is an error, it will be of type *LinkError.
func Rename(oldpath, newpath string) error {
	return rename(oldpath, newpath)
}
```

### Test Results + Spoiler Alert
![F19E25D5-A363-472B-8335-18079384C9F5](https://hackmd.io/_uploads/r1wsxCCOR.jpg)


Below are the most important parts of the implementation, which I believe will help clarify the details.

### RPC
#### how rpc struct be defined
```go=
// Worker requests a job from the coordinator.
// Coordinator replies with one of the following four job types:
const (
	MapJob = iota + 1
	ReduceJob
	WaitJob // All jobs are assigned; worker should sleep and retry later
	Done    // All jobs are finished; worker can exit
)

// job state
const (
	NotAssigned = iota
	Assigned
	Finished
)

// from client to server, report job done
type JobArgs struct {
	JobType int
	JobID   int
}

// from server to client, assign job
type JobReply struct {
	JobType        int
	JobID          int
	MapFilePath    string
	MapFilesNum    int
	ReduceFilesNum int
}
```

### Coordinator
#### data shared with all threads
```go=
type Coordinator struct {
	inputFiles       []string    // Input files; unlike the paper, we don’t split files, just use them directly
	mMap             int         // Number of map tasks (equal to number of input files)
	nReduce          int         // Number of reduce tasks (given)
	mapState         []int       // Job state for each map task
	reduceState      []int       // Job state for each reduce task
	mapAssignTime    []time.Time // Record assignment time; if a task isn’t finished within threshold, mark as unassigned again
	reduceAssignTime []time.Time // Same logic as map tasks
	m                *sync.Mutex // Mutex for synchronization
}
```
#### how coordinator assign jobs
```
function AskForJob(args, reply):
    acquire lock
    if there are unassigned map tasks:
        assign a map task to the worker
        update task state and record assignment time
    else if all map tasks are finished and there are unassigned reduce tasks:
        assign a reduce task to the worker
        update task state and record assignment time
    else if all tasks are assigned but not finished:
        set reply.JobType to WaitJob
        reassign tasks that have timed out
    else:
        set reply.JobType to Done
    release lock
```

### Worker
#### how worker handler assign jobs
```
function Worker(mapf, reducef):
    while true:
        job = askForJob()
        if job.JobType == MapJob:
            DoMapJob(mapf, job.MapFilePath, job.ReduceFilesNum, job.JobID)
            ok = reportJobDone(MapJob, job.JobID, "")
            if not ok:
                log error "report job done failed"
        else if job.JobType == ReduceJob:
            DoReduceJob(reducef, job.MapFilesNum, job.JobID)
            ok = reportJobDone(ReduceJob, job.JobID, "")
            if not ok:
                log error "report job done failed"
        else if job.JobType == WaitJob:
            sleep 1 second
        else:
            break
```
#### map
```
function DoMapJob(mapf, fileName, nReduceNum, jobID):
    file = open fileName
    content = read file
    close file
    kva = mapf(fileName, content)
    intermediate = []
    append intermediate with kva

    intermediateFiles = create nReduceNum empty lists
    for each kv in intermediate:
        reduceIndex = hash(kv.Key) % nReduceNum
        append kv to intermediateFiles[reduceIndex]

    for i in range of intermediateFiles:
        intermediateFileName = "inter-mr-jobID-i"
        intermediateFile = create temp file
        for each kv in intermediateFiles[i]:
            write kv to intermediateFile
        close intermediateFile
        rename intermediateFile to intermediateFileName
```
#### reduce
```
function DoReduceJob(reducef, mapJobNum, jobID):
    intermediate = []
    for i in range of mapJobNum:
        intermediateFileName = "inter-mr-i-jobID"
        file = open intermediateFileName
        while not end of file:
            kv = read from file
            append intermediate with kv
        close file

    sort intermediate by key

    oname = "mr-out-jobID"
    ofile = create oname
    i = 0
    while i < length of intermediate:
        j = i + 1
        while j < length of intermediate and intermediate[j].Key == intermediate[i].Key:
            j = j + 1
        values = get values from intermediate[i] to intermediate[j-1]
        output = reducef(intermediate[i].Key, values)
        write output to ofile
        i = j
    close ofile
```