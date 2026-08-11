# Distributed System Overview

## Time and Clock

### Lamport clocks
The simplest logical clock is the Lamport timestamp. Each node maintains a counter that increases with every event.
Lamport clocks doesn't find causality relationship.
Lamport clocks have a limitation: if event A has a lower timestamp than event B, we cannot tell if A caused B or if they were concurrent.
### Vector Clocks
Each node maintains a vector with one entry per node.
VC1 vs VC2	Meaning
All entries in VC1 <= VC2	VC1 happened before VC2
All entries in VC2 <= VC1	VC2 happened before VC1
Neither	Concurrent events
[2,1,0] vs [1,2,0]
2 > 1 (first entry)
1 < 2 (second entry)
These events are concurrent (neither happened before the other)
Vector clocks have a practical problem: they grow linearly with the number of participants. With thousands of nodes or actors, the overhead becomes significant. 
### Hybrid Logical Clocks (HLC)
 Hybrid Logical Clocks combine physical time with a logical counter so timestamps stay close to wall-clock time while preserving happened-before ordering in message flows.
The important nuance: HLCs do not detect concurrency the way vector clocks do. They are a compact ordering tool, not a full causal-history data structure. 
(Physical Time, Logical Counter)

## Failure Detection and Handling
Failure detection has a basic difficulty: from the outside, a crashed node and a very slow node can look identical.

**Failure severity hierarchy:**

| **Failure Type** | **Description** | **Handling Difficulty** |
|---|---|---|
| **Crash** | Node stops, does not recover | Easiest |
| **Crash-recovery** | Node stops, may recover | Moderate |
| **Omission** | Drops some messages | Moderate |
| **Timing** | Responds outside expected time | Hard |
| **Byzantine** | Arbitrary, possibly malicious | Hardest |


### The Failure Detection Problem
When you send a message and get no response, you face an impossible question: Is B dead? Or just slow?Or did the network drop the message ?

**The fundamental problem:**

| **Scenario** | **Symptom** | **Reality** |
|---|---|---|
| Node crashed | No response | Node is dead |
| Node is slow | No response (yet) | Node is alive |
| Network partition | No response | Node is alive, unreachable |
| Message lost | No response | Node is alive |

### Heartbeat-Based Detection
The simplest solution: have nodes periodically say "I am alive." If you stop hearing from a node, assume it is dead.

**Heartbeat parameters:**

| **Parameter** | **Description** | **Trade-off** |
|---|---|---|
| **Interval** | How often to send | More frequent = more overhead |
| **Timeout** | When to declare failure | Too short = false positives |
| **Threshold** | Missed beats before failure | Higher = slower detection |

### Phi Accrual Failure Detector
Binary heartbeat detection has a problem: network conditions vary. A node that usually responds in 1ms might occasionally take 100ms during a GC pause. 
Phi accrual detection adapts to these patterns by computing a suspicion level instead of a binary judgment.

How it works
Track historical heartbeat arrival times
Model arrival times as a probability distribution
Compute φ = -log₁₀(P(heartbeat not received given history))
Higher φ = higher suspicion of failure

**Benefits:**

| **Benefit** | **Explanation** |
|---|---|
| **Adaptive** | Adjusts to network conditions |
| **Probabilistic** | Gives confidence level, not binary |
| **Fewer false positives** | Accounts for variable latency |
Used by Cassandra, Akka, and other distributed systems.

### Gossip-Based Detection
Centralized failure detection has a single point of failure. Gossip protocols solve this by having nodes share their observations with each other, like rumors spreading through a crowd.

How gossip detection works
1. Each node periodically picks random node to gossip with
2. Exchange heartbeat information about all known nodes
3. If multiple nodes report node X as unresponsive, X is likely failed
4. Quorum requirement reduces false positives


### Handling Detected Failures

**Failure handling strategies:**

| **Strategy** | **When to Use** | **Implementation** |
|---|---|---|
| **Retry** | Transient failures | Exponential backoff with jitter |
| **Failover** | Node failure | Route to replica |
| **Circuit breaker** | Cascading failures | Stop calling failing service |
| **Graceful degradation** | Partial outage | Return cached/default data |

## Consensus and Coordination
At some point, distributed nodes need to agree on something: who is the current leader, what is the order of operations, whether a transaction should commit, or which configuration version is active. The basic question: given multiple nodes that might propose different values, how do we ensure they all agree on exactly one?

**Consensus requirements:**

| **Property** | **Description** |
|---|---|
| **Agreement** | All non-faulty nodes decide same value |
| **Validity** | If all propose same value, that is decided |
| **Termination** | All non-faulty nodes eventually decide |
| **Integrity** | Each node decides at most once |

### FLP Impossibility
FLP proved that deterministic consensus cannot guarantee termination in a fully asynchronous system if even one node can crash. This does not mean consensus is impossible in practice;
How real systems work despite FLP:

| **Approach** | **How It Works** |
|---|---|
| **Timing assumptions** | Assume partial synchrony |
| **Randomization** | Make algorithm non-deterministic |
| **Failure detectors** | Assume imperfect failure detection |

### Paxos
A Proposer cannot succeed unless a majority of acceptors agree. Proposer reuse the value from acceptor if some nodes agreed on the same cycle.  
https://medium.com/@angusmacdonald/paxos-by-example-66d934e18522  
Prepare: Lock the proposal number and collect past decisions. 
Accept: Use the latest past decision (if any) and ask for final approval.  
Majority Check: The organizer counts $2/3$ or $3/5$ approvals before declaring a winner/

### Raft
Works based on election system with term.

### Coordination Services
The practical advice for interviews: do not implement consensus yourself unless the question is specifically about building a consensus-backed service. Use a coordination service like ZooKeeper, etcd, or Consul for metadata and coordination.
Use coordination services for small, important metadata: leaders, leases, locaks, membership, configuration versions. Avoid using them as a general database or high-throughput message bus.

## Consistency Models
* When you read data from a distributed system, what guarantees do you have about what you will see? Consistency models answer this question.  
* Different parts of the same product often need different guarantees.  
* Consistency is not binary. Stronger models usually require more coordination, which can add latency or reduce availability during failures

### Eventual Consistency
The weakest useful guarantee: if you stop writing, eventually all replicas will have the same data. But "eventually" might be seconds, and in the meantime, different readers might see different values.

**Properties:**

| **Property** | **Value** |
|---|---|
| **Read consistency** | May read stale data |
| **Convergence** | Expected if updates stop |
| **Availability** | High |
| **Latency** | Low |

Use cases: DNS, caches, view counters, likes.

### Causal Consistency
* Causal consistency is useful for user-facing collaboration and social features. 
* It guarantees that if event A caused event B, everyone who sees B also sees A before B. 
* Unrelated concurrent events can appear in different orders.  
Rules  
* If A happened before B on same process, everyone sees A before B
* If B read from A, everyone sees A before B
* Concurrent events can be seen in any order  
Use cases: Social feeds, collaborative editing, chat.

### Sequential Consistency
All operations appear to happen in some sequential order, and operations from each process appear in program order.
Properties: There is a total order of all operations, and each process's operations appear in program order. This does not guarantee real-time ordering.  
Usecase  : Instagram post order from same person should be in order but not across all users.

Real-Time Leaderboard and Gaming Servers: In stateful multiplayer games, sequential consistency guarantees that a sequence of actions (e.g., player movement, item pickup, health deduction) is processed in the same order across all player instances, ensuring fairness and sync.

### Linearizability (Strong Consistency)
* The strongest common distributed data guarantee: operations appear to happen instantaneously at some point between when you call them and when they return. 
* The system behaves as if there is only one copy of the data. 
* This is the consistency meaning CAP usually assumes.

| **Cost** | **Reason** |
|---|---|
| **Higher latency** | Coordination required |
| **Lower availability** | Cannot serve during partitions |
| **More complex** | Requires consensus |
### Conflict Resolution

Last-write-wins (LWW) is the simplest approach: attach a timestamp to each write and keep the one with the highest timestamp. It is easy to implement but can discard a concurrent update without warning, because the "later" timestamp may reflect clock skew rather than true ordering.

Conflict-free Replicated Data Types (CRDTs) take a different approach. They are data structures designed so that concurrent updates always merge deterministically, without coordination and without losing data. Examples include grow-only counters, sets that track additions and removals, and sequence types used in collaborative text editors.


**Conflict resolution for eventual consistency:**

| **Approach** | **Merge Behavior** | **Trade-off** |
|---|---|---|
| **Last-write-wins** | Keep the highest-timestamp value | Simple, but can drop concurrent writes |
| **CRDTs** | Merge concurrent updates by construction | No data loss, but limited to specific data types and carries more metadata |

CRDTs are why tools like collaborative editors and shared shopping carts can stay available during a partition and still converge to a sensible state once replicas reconnect. They pair naturally with the eventual and causal consistency models above.

## Distributed System Patterns
Problem which occuring in distributed system.
### Leader Election

### Distributed Locking
### Saga Pattern
### Event Sourcing
### CQRS
### Circuit Breaker

## Observability and Debugging
### Distributed Tracing
### Key Metrics
### Structured Logging
