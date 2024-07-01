---
title: "Understanding the Raft Consensus Algorithm"
date: 2024-06-26
draft: false
---

## Introduction

In the realm of distributed systems, achieving consensus among nodes is a fundamental challenge. The Raft consensus algorithm, developed by Diego Ongaro and John Ousterhout, has emerged as a popular choice for its simplicity, understandability, and strong consistency guarantees. In this blog post, we'll take a deep dive into the Raft algorithm, exploring its key components, working principles, and practical applications.

## Advantages of Raft

Raft offers several advantages over other consensus algorithms:

1. **Simplicity**: Raft is designed to be easy to understand and implement. Its clear separation of concerns and well-defined states make it more approachable compared to other consensus algorithms like Paxos.

2. **Strong Consistency**: Raft provides strong consistency guarantees, ensuring that all nodes in the cluster have the same view of the data. This is crucial for applications that require strict data integrity and consistency.

3. **Fault Tolerance**: Raft is designed to handle node failures and network partitions gracefully. It can continue to operate and maintain data consistency as long as a majority of nodes are available.

## Understanding Raft

Raft is a consensus algorithm designed to manage replicated state machines in a distributed system. It aims to provide a reliable and efficient way for multiple nodes to agree on a shared state, even in the presence of failures or network partitions. Raft is based on the idea of electing a leader node that is responsible for managing the replication of the shared state to the other nodes, known as followers.

Raft consists of three main components:

1. **Leader Election**: In Raft, nodes are in one of three states: leader, candidate, or follower. Leader election is triggered when the current leader fails or when a follower times out waiting for communication from the leader. The candidate nodes compete for votes, and the node receiving the majority of votes becomes the new leader.

2. **Log Replication**: Once a leader is elected, it receives client requests and appends them to its log. The leader then replicates the log entries to the follower nodes to ensure that all nodes have the same sequence of commands. Raft uses a strong consistency model, requiring a majority of nodes to acknowledge the replication before committing the log entries.

3. **Safety**: Raft guarantees the safety property through the use of term numbers. Each term has a unique incrementing number, and a leader can only be elected in a new term. This mechanism prevents conflicting decisions from being made in different terms, ensuring that no two nodes can have different values for the same log entry.

## Raft in Action

Raft has found wide adoption in various distributed systems and applications. Here are a few notable examples: [etcd](https://github.com/coreos/etcd/tree/master/raft) and [consul](https://www.consul.io/docs/internals/consensus.html).

1. **etcd**: etcd is a distributed key-value store that uses Raft as its consensus algorithm. It is commonly used for storing configuration data, service discovery, and distributed locking in containerized and cloud-native environments.

2. **HashiCorp Consul**: Consul is a distributed service mesh and key-value store that employs Raft for consensus. It provides features like service discovery, health checking, and configuration management for distributed systems.

## Implementation

The Skeleton Code is provided below:

(From MIT [6.824 Distributed System public educational resources.](https://github.com/nsiregar/mit-go/tree/master/src/raft))

I'll go through all functions in skeleton code one by one, and briefly talk about its functionality and how it works, but no actual code will be provided to prevent undesirable effects.

For better understanding, I'll draw a brief diagram to illustrate my understanding of the inter-relationship between components in Raft protocol.

```go
func (rf *Raft) GetState() (int, bool) {}
```

This function is a getter function that gets the term (returning an integer) and role (a boolean on if it is leader) of the current node.

---

```go
func (rf *Raft) persist() {}
```

To ensure durability and recover from crashes, the Raft state is persisted to stable storage. This function saves current state to persistent state, it enables the node to retrieve the information stored as persistent state after crashing and restarting. The current state includes: current term, voted for, and log entries.

---

```go
func (rf *Raft) readPersist(data []byte) {}
```

This function will read the saved persistent state stored by the persist function above. Restores the state from the persisted data during initialization.

---

```go
type RequestVoteArgs struct {}`, `type RequestVoteReply struct {}
```

RPC (Remote Procedure Call) is a protocol that allows a program to execute a procedure or function on another computer as if it were a local call.
These two RPC are served as data carriers among nodes for the leader election process.
Leader Election: The leader election process is implemented in the startElection function. When a node's election timeout expires, it transitions to the candidate state, increments its current term, and votes for itself. It then sends RequestVote RPCs to all other peers concurrently using goroutines.

---

```go
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {}
```

This RequestVote RPC handler, implemented in the RequestVote function, handles incoming vote requests. It checks the candidate's log against its own log to determine whether to grant the vote. If the candidate's log is up-to-date, the node grants the vote and resets its election timeout. Once the candidate decides to start an election, this function will be called concurrently to all peers to get votes from peers.

---

```go
func (rf *Raft) sendRequestVote(server int, args *RequestVoteArgs, reply *RequestVoteReply) bool {}
```

This is a function to call all peer nodes in a for loop and request votes.

---

```go
func (rf *Raft) Start(command interface{}) (int, int, bool) {}
```

Log Replication: Once a leader is elected, it begins log replication. When a client sends a command to the leader via the Start function, the leader appends the command to its log and sends AppendEntries RPCs to all followers to replicate the log entries.

---

```go
type AppendEntriesArgs struct {}
type AppendEntriesReply struct {}
```

In general, AppendEntriesArgs and AppendEntriesReply structs in Raft are used for facilitating communication and synchronization in the Raft consensus algorithm. It helps with mechanisms including:

- Log replication: The leader sends new log entries to followers.
- Heartbeat mechanism: Empty AppendEntries calls act as heartbeats.
- Consistency checks: Verify log consistency between leader and followers.
- Commit index updates: Inform followers of the leader's commit progress.
- Term number synchronization: Ensure all nodes are on the same term.

---

```go
func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {}
```

The AppendEntries RPC handler, implemented in the AppendEntries function, handles incoming log entries from the leader. It checks for log consistency, appends the entries to its own log, and updates the commitIndex based on the leader's commitIndex.
The leader uses nextIndex and matchIndex to keep track of the replication progress for each follower. It optimistically sends log entries to followers and retries with a reduced nextIndex if the follower's log is inconsistent.
Committing Logs: Once a log entry has been replicated to a majority of servers, it can be committed. The updateCommitIndex function determines the highest log entry that has been replicated on a majority of servers and updates the commitIndex accordingly.

---

```go
func (rf *Raft) ticker() {}
```

Handling Timeouts and Heartbeats: The ticker function plays a crucial role in handling election timeouts and heartbeats. If a node is a leader, it periodically sends heartbeats (AppendEntries RPCs with empty entries) to maintain its authority and prevent new elections. If a follower's election timeout expires without receiving a heartbeat, it transitions to the candidate state and starts a new election.

## Conclusion

The Raft consensus algorithm has revolutionized the way distributed systems achieve consensus and maintain data consistency. Its simplicity, strong consistency guarantees, and fault tolerance have made it a go-to choice for building reliable distributed applications.

For further exploration, I highly recommend reading the original Raft paper by Diego Ongaro and John Ousterhout, as well as exploring the various open-source projects and tools that have embraced Raft as their consensus algorithm.

Remember, the journey of implementing Raft is not just about the destination, but also about the valuable lessons learned along the way. Embrace the challenges, learn from the community, and continue refining your implementation. Happy coding!

## Summary of Tools

- Debug Tools: [https://blog.josejg.com/debugging-pretty/](https://blog.josejg.com/debugging-pretty/)
- Visualization Tools: [https://thesecretlivesofdata.com/raft/](https://thesecretlivesofdata.com/raft/)

## Acknowledgement

This blog post reflects my personal experience and understanding of implementing the Raft Protocol. It is not affiliated with or endorsed by any institution and should not be used as an official educational reference. The content is based on my interpretation of various resources and may contain unintentional errors. It is not intended to infringe on any copyrights. If you find any similarities to other works or have concerns about the content, please contact me for immediate review and potential removal. Readers are encouraged to verify information from authoritative sources for critical applications.

## Reference

Special Thanks To:

- CS 4740, Prof. Chang Lou, University of Virginia
  [https://changlousys.github.io/CS4740/spring24/about/](https://changlousys.github.io/CS4740/spring24/about/)
  For developing and teaching cloud computing class, and indirectly lead to the creation of this blog.

- In Search of an Understandable Consensus Algorithm, Diego Ongaro and John Ousterhout, Stanford University
  [https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14)

- Raft Consensus Algorithm, Geek for geeks
  [https://www.geeksforgeeks.org/raft-consensus-algorithm/](https://www.geeksforgeeks.org/raft-consensus-algorithm/)

- Raft Algorithm, Wikipedia
  [https://en.wikipedia.org/wiki/Raft\_(algorithm)](<https://en.wikipedia.org/wiki/Raft_(algorithm)>)

- Raft Introduction, Github Page
  [https://raft.github.io/](https://raft.github.io/)

- 6.5840 Distributed Systems: Massachusetts Institute of Technology
  [https://pdos.csail.mit.edu/6.824/](https://pdos.csail.mit.edu/6.824/)
