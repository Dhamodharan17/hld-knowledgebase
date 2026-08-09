# Index of system design topics

> Summaries of various system design topics, including pros and cons. **Everything is a trade-off.**
>
> Each section contains links to more in-depth resources.

# Distributed System
<!-- Table of Contents Section -->
* [Introduction to Distributed System](#distributed-system)
  * [Why Distributed System ?](#why-distributed-system)
  * [Fallacies of Distributed System](#fallacies-of-distributed-system)
  * [Why Distributed System are hard ?](#why-distributed-system-are-hard)
  * [Correctness in Distributed System](#correctness-in-distributed-system)
  * [System Models](#system-models)
    * [Timing Model](#timing-model)
    * [Fault Model](#fault-model)
    * [Network Model](#network-model)
  * [Tale of exactly one semantics](#tale-of-exactly-one-semantics)
  * [Failure in the world of distributed system](#failure-in-the-world-of-distributed-system)
  * [Stateful & stateless system ?](#stateful--stateless-system)
* [Basic Concepts & Theorems](#basic-concepts--theorems)
  * [Partitioning](#partitioning)
    * [Vertical partitioning](#vertical-partitioning)
    * [Horizontal partitioning](#horizontal-partitioning)
      * [Range based partitioning](#range-based-partitioning)
      * [Hash based partitioning](#hash-based-partitioning)
      * [Consistent hashing](#consistent-hashing)
        * [Virtual nodes](#virtual-nodes)
  * [Replication](#replication)
    * [Single master replication](#single-master-replication)
    * [Multi master replication](#multi-master-replication)
    * [Quorums in distributed system](#quorums-in-distributed-system)
  * [Safety Guarantees](#safety-guarantees)
  * [Consistency Models](#consistency-models)
    * [ACID Transactions](#acid-transactions)
    * [CAP Theorem](#cap-theorem)
    * [PACELC Theorem](#pacelc-theorem)
  * [Isolation Level & Anomalies](#isolation-level--anomalies)
    * [Isolation Levels](#isolation-levels)
      * [Serializability](#serializability)
      * [Repeatable Read](#repeatable-read)
      * [Snapshot Isolation](#snapshot-isolation)
      * [Read Committed](#read-committed)
      * [Read Uncommitted](#read-uncommitted)
    * [Anomalies](#anomalies)
      * [Dirty Write](#dirty-write)
      * [Dirty Read](#dirty-read)
      * [Fuzzy or non-repeatable read](#fuzzy-or-non-repeatable-read)
      * [Phantom reads](#phantom-reads)
      * [Read Skew](#read-skew)
      * [Write Skew](#write-skew)
* [Distributed Transactions](#distributed-system)
  * [Types of Distributed Transactions](#why-distributed-system)
  * [Achieving Isolation](#fallacies-of-distributed-system)
    * [PCC / 2PL ](#why-distributed-system-are-hard)
    * [OCC / MVCC ](#correctness-in-distributed-system)
  * [Achieving Atomicity](#system-models)
    * [2PC](#timing-model)
    * [3PC](#fault-model)
    * [Quorum-based Commit](#network-model)
  * [Achieving consistency in long lived transaction](#tale-of-exactly-one-semantics)
    * [Sagas](#timing-model)
* [Consensus](#consensus)
  * [FLP Impossibility](#flp-impossibility)
  * [Paxos](#paxos)
  * [Replicated State Machine via Consensus](#replicated-state-machine-via-consensus)
  * [Distributed Transaction via Consensus](#distributed-transaction-via-consensus)
  * [Raft](#raft)
* [Time & Clock](#time--clock)
  * [Time](#time)
  * [Logical Clocks](#logical-clocks)
  * [Orders](#orders)
  * [Total Order and Partial Order](#total-order-and-partial-order)
  * [Concept of Causality](#concept-of-causality)
  * [Lamport Clock](#lamport-clock)
  * [Vector Clock](#vector-clock)
  * [Version Vectors & Dotted Vectors](#version-vectors--dotted-vectors)
  * [Distributed Snapshot](#distributed-snapshot)
  * [Physical & Logical Time](#physical--logical-time)

# Interview Patterns

* [Real-time Updates](#distributed-system)
  * [Short Polling](#why-distributed-system)
  * [Long Polling](#fallacies-of-distributed-system)
  * [Server-Sent-Events(SSE)](#why-distributed-system-are-hard)
  * [Web Sockets](#correctness-in-distributed-system)
  * [Core Challenges](#system-models)
    * [Connection Managements](#system-models)
    * [Hearbeats for connection health](#system-models)
    * [Message out of order](#system-models)
    * [Back Pressure & Flow Control](#system-models)
* [Fan-Out Pattern](#distributed-system)
  * [Fan-Out](#why-distributed-system)
  * [FanOut-On-Write (Push Model)](#fallacies-of-distributed-system)
  * [FanOut-On-Read (Pull Model)](#why-distributed-system-are-hard)
  * [Hybrid Approach](#correctness-in-distributed-system)
  * [Handling Edgecases](#system-models)
  * [Performance Optimization](#system-models)
    * [Batching](#system-models)
    * [Multi-layer Caching](#system-models)
    * [Feed Cache Sizing](#system-models)
* [Handling High Read Traffic](#handling-high-read-traffic)
  * [Understanding the Problem](#caching)
  * [Caching](#caching)
  * [Read Replica](#read-replica)
  * [CDN / Edge Servers](#cdn--edge-servers)
  * [Database Optimization (Indexing / Partitioning)](#database-optimization-indexing--partitioning)
  * [Pre-computation](#pre-computation)
  * [Load Balancing](#load-balancing)
* [Handling High Write Traffic](#handling-high-read-traffic)
  * [Understanding the Problem](#caching)
  * [Batching](#caching)
  * [Async Processing](#read-replica)
  * [Sharding](#cdn--edge-servers)
  * [Write Optimized Storage](#database-optimization-indexing--partitioning)
  * [Event Sourcing/ CQRS](#pre-computation)
  * [Back Pressure](#load-balancing)
* [Handling Hot Keys](#handling-high-read-traffic)
  * [Understanding Hot Keys](#caching)
  * [Hot Key Detection](#caching)
  * [Local Caching](#read-replica)
  * [Key Replication](#cdn--edge-servers)
  * [Key Splitting(Shared Counters)](#database-optimization-indexing--partitioning)
  * [Request Coalescing](#pre-computation)
  * [Read-Through Cache with Locking](#load-balancing)
  * [Rate Limiting Per Key](#load-balancing)
* [Handling Traffic Spikes](#handling-high-read-traffic)
  * [Understanding Traffic Spikes](#caching)
  * [AutoScaling](#autoscaling)
  * [Load Shedding](#load-shedding)
  * [Rate Limiting](#rate-limiting)
  * [Caching](#caching)
  * [Queue-Based Load Leveling](#queue-based-load-leveling)
  * [Graceful Degradation](#graceful-degradation)
  * [Database Protection](#database-protection)
  * [Pre-Warming and Capacity Planning](#pre-warming-and-capacity-planning)
* [Handling Large Files](#handling-high-read-traffic)
  * [Understanding Naive File Handling](#caching)
  * [Chunked Uploads](#autoscaling)
  * [Direct Upload with Pre-Signed URLs](#load-shedding)
  * [Multipart Upload Protocol](#rate-limiting)
  * [Download Optimizations](#caching)
  * [Range Request](#queue-based-load-leveling)
  * [CDN Distribution](#graceful-degradation)
  * [Storage Architecture](#database-protection)
    * [Choosing the Right Storage](#database-protection)
    * [Separating Metadata from Data](#database-protection)
    * [Content-Addressable Storage](#database-protection)
  * [Compression and Deduplication](#pre-warming-and-capacity-planning)
      * [Block Level Deduplication](#database-protection)
      * [Content-Defined Chunking](#database-protection)
* [Media Streaming](#media-streaming)
  * [Why Is Streaming Challenging?](#why-is-streaming-challenging)
  * [Video Encoding Fundamentals](#video-encoding-fundamentals)
    * [How Video Compression Works](#how-video-compression-works)
    * [Codecs](#codecs)
    * [Encoding Profiles](#encoding-profiles)
  * [Streaming Protocols](#streaming-protocols)
  * [Architecture of Streaming System](#architecture-of-streaming-system)
    * [High-Level Architecture](#high-level-architecture)
  * [Live Streaming vs Video-on-Demand](#live-streaming-vs-video-on-demand)
  * [Adaptive Bitrate Streaming (ABR)](#adaptive-bitrate-streaming-abr)
    * [How ABR Works](#how-abr-works)
    * [ABR Algorithms](#abr-algorithms)
    * [Quality Switches](#quality-switches)
  * [Low-Latency Streaming](#low-latency-streaming)
    * [Sources of Latency](#sources-of-latency)
    * [Techniques to Reduce Latency](#techniques-to-reduce-latency)
    * [Latency vs Scale Trade-off](#latency-vs-scale-trade-off)
  * [Scaling Strategies](#scaling-strategies)
  * [Content Protection](#content-protection)
  * [Measuring Quality of Experience](#measuring-quality-of-experience)
  * [Real-World Architecture: Live Streaming Platform](#real-world-architecture-live-streaming-platform)
  * [Common Mistakes to Avoid](#common-mistakes-to-avoid)
* [Handline Location Data](#media-streaming)
* [Generate Unique Ids](#media-streaming)
* [Distributed Counter](#media-streaming)
* [Handlig Failures](#media-streaming)
* [Distributed Counter](#media-streaming)
---

## Media Streaming

## Why Is Streaming Challenging?
Demanding workload:
1. Video is Massive – raw video, enormous egress
2. Timing is critical – frame shouldn't arrive late
3. Networks are unreliable – should adapt n/w condition
4. Scale multiplies everything – all problems will be multiplied

## Video Encoding Fundamentals

### How Video Compression Works
Removing redundancy:
1. Spatial redundancy – within frame (sky, wall)
2. Temporal redundancy – between frames
3. I-Frames (intra-frames) – like photo / no dependency / anchor seek points
4. P-Frames (Predicted) – predict from previous P or I frame
5. B-Frames (Bidirectional) – predict current from left and right

### Codecs
Implements compression algorithms using above frames (H.264 / VP9 / AV1)

### Encoding Profiles
1. Key frame – how often to insert I frame
2. FPS – frame per second (30 fps + Keyframe interval = 2 seconds → every 60 frames, one I-frame)
3. Bitrate – how much data is sent every second

## Streaming Protocols
1. **RTMP** (Real-Time Messaging Protocol) – TCP based / upload use cases
2. **HLS** (HTTP Live Streaming) – chop into segments / maintain text manifest
3. **DASH** (Dynamic Adaptive Streaming over HTTP) – same as HLS but XML manifest
4. **WebRTC** – UDP based for real-time communication (video calls) / cannot scale much

## Architecture of Streaming System

### High-Level Architecture
1. **Ingest Layer** – streamers inject
2. **Transcoding Layer** – transcode to multi-format / package into segments and manifest
3. **Storage Layer** – hot (live) / warm (VOD) / cold (archive)
4. **Origin Server** – serves segments and manifest
5. **CDN** (Content Delivery Network) – redundant / protect origin
6. **Player Layer** – downloads segment and manifest, decides next quality segment to download

## Live Streaming vs Video-on-Demand
1. Live Streaming
2. Video-on-Demand

## Adaptive Bitrate Streaming (ABR)

### How ABR Works

### ABR Algorithms
1. Throughput-based
2. Buffer-based (BBA)
3. Hybrid approaches

### Quality Switches

## Low-Latency Streaming

### Sources of Latency

### Techniques to Reduce Latency
1. Shorter segments
2. Chunked transfer (CMAF)
3. Low-latency HLS (LL-HLS)
4. Reduced client buffer
5. WebRTC for interactive delivery

### Latency vs Scale Trade-off

## Scaling Strategies
1. CDN Edge caching
2. Multi-CDN
3. Regional Origin Server
4. Transcoding Distribution
5. Predictive Scaling

## Content Protection
1. Tokenized URLs
2. DRM (Digital Rights Management)

## Measuring Quality of Experience

## Real-World Architecture: Live Streaming Platform
1. Requirement
2. Architecture
3. Component Details
   1. Ingest Layer
   2. Transcoding Pipeline
   3. Manifest Generation
   4. Low-Latency Mode
4. Scaling Considerations
   1. Ingest scaling
   2. Transcoding scaling
   3. Storage scaling
   4. CDN scaling

## Common Mistakes to Avoid
1. Ignoring client diversity
2. Underestimating transcoding costs
3. Not testing network conditions
4. Hardcoding URLs
5. Ignoring the last mile

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
