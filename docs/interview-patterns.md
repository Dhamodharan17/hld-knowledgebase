# Interview Patterns

## Real-time Updates
### Short Polling
### Long Polling
### Server-Sent-Events (SSE)
### Web Sockets
### Core Challenges
#### Connection Managements
#### Heartbeats for Connection Health
#### Message out of Order
#### Back Pressure & Flow Control

## Fan-Out Pattern
### Fan-Out
### FanOut-On-Write (Push Model)
### FanOut-On-Read (Pull Model)
### Hybrid Approach
### Handling Edgecases
### Performance Optimization
#### Batching
#### Multi-layer Caching
#### Feed Cache Sizing

## Handling High Read Traffic
### Understanding the Problem
### Caching
### Read Replica
### CDN / Edge Servers
### Database Optimization (Indexing / Partitioning)
### Pre-computation
### Load Balancing

## Handling High Write Traffic
### Understanding the Problem
### Batching
### Async Processing
### Sharding
### Write Optimized Storage
### Event Sourcing / CQRS
### Back Pressure

## Handling Hot Keys
### Understanding Hot Keys
### Hot Key Detection
### Local Caching
### Key Replication
### Key Splitting (Shared Counters)
### Request Coalescing
### Read-Through Cache with Locking
### Rate Limiting Per Key

## Handling Traffic Spikes
### Understanding Traffic Spikes
### AutoScaling
### Load Shedding
### Rate Limiting
### Caching
### Queue-Based Load Leveling
### Graceful Degradation
### Database Protection
### Pre-Warming and Capacity Planning

## Handling Large Files
### Understanding Naive File Handling
### Chunked Uploads
### Direct Upload with Pre-Signed URLs
### Multipart Upload Protocol
### Download Optimizations
### Range Request
### CDN Distribution
### Storage Architecture
#### Choosing the Right Storage
#### Separating Metadata from Data
#### Content-Addressable Storage
### Compression and Deduplication
#### Block Level Deduplication
#### Content-Defined Chunking

## Media Streaming
→ See [media-streaming.md](./media-streaming.md)

## Handling Location Data
### Why is Location Data Challenging?
### Spatial Indexing Techniques
#### Quadtree
#### R-tree
#### Geo-Hash
### Database Options for Geo-Spatial Data
#### PostgreSQL + PostGIS
#### Redis Geo
#### MongoDB (2dsphere Index)
#### Elasticsearch
### Common Query Patterns
#### Nearby Search (K-Nearest Neighbors)
#### Radius Search
#### Bounding Box Query
#### Polygon Search
#### Distance Calculation
### Scaling Strategies
#### Geographic Sharding
#### Geohash-Based Partitioning
#### Hybrid Architecture
#### Caching Hot Regions
#### In-Memory Quadtree
### Privacy and Data Sensitivity
#### Collect only what you need
#### Reduce precision where full precision is not needed
#### Separate live data from history
#### Apply access controls and retention policies

## Generate Unique IDs
### Why Is This Problem Hard?
### Database Auto-Increment
### UUID (Universally Unique Identifier)
### Snowflake IDs
### ULID (Universally Unique Lexicographically Sortable Identifier)
### MongoDB ObjectID
### Ticket Servers
### Other Notable Schemes
#### KSUID (K-Sortable Unique Identifier)
#### Sonyflake
#### NanoID

## Distributed Counter
### Why Is Counting Hard at Scale?
### The Concurrency Problem
### Single Counter with Atomic Updates
### Sharded Counters
### Write-Behind (Async Aggregation)
### Count-Min Sketch (Approximate Counting)
### HyperLogLog (Cardinality Estimation)
### Counting Over Time Windows

## Handling Failures
### Why Failures Are Inevitable
### Core Failure Handling Patterns
#### Retries with Exponential Backoff
#### Circuit Breakers
#### Timeouts
#### Fallbacks
#### Bulkheads
#### Idempotency
#### Graceful Degradation
#### Load Shedding and Backpressure
#### Failover
#### Data Replication
### Implementation Checklist
#### Design for Failure
#### Fail Fast
#### Make Failures Visible
#### Test Failure Handling
#### Automate Recovery

## Removing Single Point of Failure
### Understanding SPOFs
### Strategies to Remove SPOFs
#### Redundancy
#### Load Balancing
#### Redundant Load Balancers
#### Database Replication
#### Multi-Zone Deployment
#### Multi-Region Deployment
#### Eliminate Human SPOFs
### Layer-by-Layer SPOF Elimination
#### DNS Layer
#### CDN Layer
#### Application Layer
#### Cache Layer
#### Database Layer
#### Message Queue Layer
#### Storage Layer
### Testing for SPOFs
#### Chaos Engineering
#### Game Days
#### Automated Testing
### Cost vs. Resilience Trade-offs
#### Risk Assessment Matrix
#### Calculating Acceptable Risk
