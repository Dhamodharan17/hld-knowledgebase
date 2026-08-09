# Index of system design topics

> Summaries of various system design topics, including pros and cons. **Everything is a trade-off.**
>
> Each section contains links to more in-depth resources.

<!-- Table of Contents Section -->
* [Distributed System](#distributed-system)
  * [Why Distributed System ?](#why-distributed-system)
  * [Fallacies of Distributed System](#fallacies-of-distributed-system)
  * [Why Distributed System are hard ?](#why-distributed-system-are-hard)
  * [Correctness in Distributed System](#correctness-in-distributed-system)
  * [System Models ?](#system-models)
    * [Timing Model](#timing-model)
    * [Fault Model](#fault-model)
    * [Network Model](#network-model)
  * [Tale of exactly one semantics](#tale-of-exactly-one-semantics)
  * [Failure in the world of distributed system](#failure-in-the-world-of-distributed-system)
  * [Stateful & stateless system ?](#stateful--stateless-system)

---

## Distributed System
Components are located on different networked computer which communicate & co-ordinate thier action by passing messages to one another.

**Why It Matters in Interviews ?**
* Sets vocabulary — interviewers expect you to frame solutions in terms of nodes, messages, coordination 


## Why Distributed System ?
1. Performance - hardware limitations of a single computer (better performance at lower cost using) & using multiple commodity hardware.
2. Scalability - we can split, store and distributed data and traffice among multiple nodes.
3. Avaiability - redudancy (using multiple nodes)

**Why It Matters in Interviews ?**
* Directly answers "why not just use one big server?" — foundational justification for every design decision 

## Fallacies of Distributed System
1. N/w is reliable - even TCP gurantees, N/w and its hardware can fail.
2. N/w is homogenous - all machines are not same.
3. Topology doesn't change - no fixed IP (nodes come and go)
4. Latency is zero - n/w call not same as local database call.
5. Bandwidth is infinite - concurrent request can congest bandwidth.
6. N/w is secure
7. Global Clock
8. There is one admin controls everything

**Why It Matters in Interviews ?**
Shows maturity — you know the network is unreliable, latency exists, topology changes; prevents naive designs

## Why Distributed System are hard ?
* Hard to design, build and reason thus increasing the risk of error.
* Main properties which makes distributed system hard?
  1. Network asynchrony
  2. Partial failures
  3. Concurrency

## Correctness in Distributed System
* correctness - propeties must be satisfied by system.
* safety properties - must never happen (money shouldn't be deducted twice)
* liveness properties - should happen eventually.(Once n/w back; make payment success or fail clearly and refund)
* Trade off between both or cannot be achieved both.

**Why It Matters in Interviews ?**
Comes up when justifying tradeoffs — "this system guarantees it won't double-charge (safety) but may delay confirmation (liveness)"

## System Models
* Formal set of assumpations about how the system's process, network and time behaves.
* Distributed system are too diverse to reason about directly, so we define model with clear assumption.
* Algorthim proven correct in a mode, work for all system that satisy those assumptions without reasoning from scartch.
* There are 3 Dimension/Properties of a system.
  1. How nodes communicate with each other (timing model)
  2. How nodes fail(fault model)
  3. How messages behave (network model)
  4. Clock Assumptions. 

**Why It Matters in Interviews ?**
Lets you reason about algorithm choices — "I'm using quorum because we assume crash faults and unreliable links"

### Timing Model
| **Model** | **Description** |
|---|---|
| **Synchronous** | Known bounds on message delay and processing time |
| **Asynchronous** | No timing bounds — you can't distinguish slow from dead |
| **Partially synchronous** | Usually synchronous, but occasionally asynchronous (real world) |

### Fault Model
| **Model** | **Description** | **Node** |
|---|---|---|
| **Fail-stop** | Permantely gone & other nodes cna detects its gone| Crash + Detectable
| **Crash Nodes(fail-silent)** | dead but others cannot detect its gone. | Crash + not detectable
| **Ommision** | Works but unable to communicate | Drops message
| **Byzantine** | Random problem | Arbitrary/malicious behavior.
### Network Model
| **Model** | **Description** |
|---|---|
| **Reliable links** | Messages are never lost, duplicated, or reordered |
| **Fair-loss links** | Messages may be lost but will eventually get through if retried |
| **Arbitrary links** | Messages can be lost, duplicated, reordered, or corrupted |

## Tale of exactly one semantics
* Issue with processing message or not processing at all.  
E.g. deducting money twice or not deducting at all.
* Solutions
  1. Idempotent operation
  2. De-duplication
* Notion of delivery & Processing
  1. Delivery - we don't control over because of n/w reliability and stability
  2. Processing - we have control and we can do some solutioning.
* Even the message delivered multiple times, we can ensure **the effect happens only one**

**Why It Matters in Interviews ?**
Very common interview topic — design a payment system, message queue, or order processor without duplicate processing. Practical solutions interviewers expect: idempotency & de-duplication (Stripe, AWS all use this)

## Failure in the world of distributed system
* Very difficult to identify failure due to async nature and very hard to find node dead or slow 
* Main mechanism to detect 
  1. Time out - otherwise node will be waiting eternally.
      * Small number – early declare dead or even split brain 
      * Large number – response will be slower due to waiting 
  2. Failure detectors – component of a node which identifies other failed nodes 
      * 2 category of detectors (trade off b/w completeness & accuracy) 
      1. Based on completeness(short timeout) – ever gets stuck waiting on a dead node (may falsely accuse a slow node)
      2. Based on accuracy (long timeout) – never wrongly accuses a live node (may wait too long on a dead one)

| **Detector** | **Used In** | **Tuning** |
|---|---|---|
| **Heartbeat + timeout** | Kafka, ZooKeeper, etcd | `session.timeout.ms`, `tickTime` — lower = faster detection, more false positives |
| **Phi Accrual detector** | Akka, Cassandra | `phi` threshold — higher phi = more accurate, slower to declare failure |

**Why It Matters in Interviews ?**
"How do you know a node is dead?" — timeout tradeoffs show you understand split-brain risk. Underpins leader election, health checks, load balancer design

## Stateful & stateless system
* Stateless - maintains no states/ all nodes are identical/ easy to scale
* Stateful - more complex as different node store different data, need proper routing.

**Why It Matters in Interviews ?**
Core scaling question — "can I add more instances?" depends entirely on whether your service is stateful
