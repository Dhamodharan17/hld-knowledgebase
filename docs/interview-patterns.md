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
**Proactive Detection**
* The goal of proactive detection is to identify hot keys before they cause problems.
* Monitor key access frequency: Most cache systems provide ways to sample access patterns or export per-node statistics
    * In Redis, MONITOR Command
    * Redis has a --hotkeys option
*  In production, teams often combine cache telemetry, client-side top-K counters, request logs, and node-level imbalance alerts.
**Reactive Detection**
* Reactive detection means identifying hot keys when they're actively causing problems.
* Watch for load imbalance: Node CPU, Request Latency, Network I/O, Cache Evictions.
* Set up alerts for imbalance : fire on skew rather than on healthy cluster-wide load.
**Predictive Detection**
* For scheduled events like flash sales or product launches, you can predict hot keys in advance and prepare for them.
**Pre-warming strategy:**
* Identify the keys that will be hot
* Pre-replicate read-heavy keys to multiple cache nodes
* Configure special handling by data type.
* Prepare the operations plan.
### Solution Patterns
No single solution handles every hot key. Choose a strategy based on whether the key is **read-heavy, write-heavy, approximate, or strongly consistent.**
### Local Caching
* A common first mitigation is to cache hot data in application server memory.
* The cost is staleness - With a 5-second TTL, data might be up to 5 seconds out of date.
* Best for: Read-heavy hot keys where slight staleness is acceptable, such as post content, product descriptions, public profiles, article content, and metadata.
### Key Replication
* For read-heavy keys, you can **replicate the hot key across multiple cache nodes** and load-balance reads across them.
* This costs consistency and adds write amplification - Every update must write to or invalidate all replicas.
* Best for: Hot keys with high read-to-write ratios where bounded staleness is acceptable or versioning can hide inconsistencies, such as public profiles, product details, configuration snapshots, and static metadata
### Key Splitting (Shared Counters) (same key in multiple nodes having ratio of values)
* Replicating the same key does not help if every write must update the same logical value. You need to distribute writes.
* The cost moves to the read path - Getting the total requires reading all 100 shards and summing them
Sharding / Sub-keying (Splitting Data)What it means: The total count is broken into distinct, independent pieces across nodes. 
No node has the complete data.Example:Node A holds likes:post:123:0 = 20Node B holds likes:post:123:1 = 15Node C holds likes:post:123:2 = 15Why it solves write heavy load: A incoming write request only goes to one randomly picked node (e.g., Node B gets +1). Node A and Node C know nothing about this write.Total count is only known when you sum them up ($20 + 16 + 15 = 51$).
* Best for: Write-heavy counters where exact real-time reads are not required, such as like counts, view counts, impressions, and approximate rate accounting.
### Request Coalescing
* Request coalescing ensures that only one request fetches the data; others wait and share the result:
* When a cache key expires or is missing, multiple concurrent requests might all try to fetch and populate it simultaneously.
* The limitation is scope - In-process request coalescing only helps with concurrent requests on the same server.
* Best for: Cache stampedes, cold cache warming, bursty traffic patterns.
### Stale-While-Revalidate
* For many read-heavy hot keys, a good user experience is to **serve a slightly stale value while one background refresh updates the cache.**   
* This costs freshness in a bounded way
* Best for: Public content, product pages, profiles, live metadata, and anything where slightly stale is better than an outage.

| **Feature** | **Stale-While-Revalidate** | **Request Coalescing** |
|---|---|---|
| **Response Speed** | Instant (0 ms latency) | Slower (waits for DB query to complete) |
| **Data Freshness** | Slightly stale during revalidation | Guaranteed fresh |
| **Request Handling** | Serves old data, triggers 1 async job | Blocks concurrent requests, shares 1 sync job |
| **Use Case** | Feed items, product details, news articles | Stock availability, bank balances, strict real-time data |


### Read-Through Cache with Locking
* Building on request coalescing, you can use distributed locking to coordinate across servers. 
* **When the cache is empty, only one server acquires the lock and populates it; others wait or serve stale data.**
* The compare-and-delete step is not a single Redis command. Redis has no atomic check-token-then-delete, so it is implemented as a short Lua script that reads the key, compares the token, and deletes only on a match. Without it, a server whose lock already expired could delete a lock another server now holds.
* This costs latency and demands careful lock handling - Servers that do not acquire the lock must wait or serve stale data. Locks need expiry, ownership tokens, and backoff so a crashed refresher does not block everyone or delete another server's lock.

### Rate Limiting Per Key
* Sometimes the best option is to limit the damage. If a hot key is overwhelming your system, per-key admission control prevents it from taking everything down.
* The cost falls on user experience
Some users may get stale data, queued responses, or controlled errors. This is usually better than taking down the entire system.
* Best for: Last line of defense, combined with other strategies.

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
User frustation on reupload after 90% and Server suffers since buffer too much data, tie up worker threads, hit load balancer timeouts, or proxy every byte through application servers before writing to storage.  

Large-file handling is mostly about avoiding that all-or-nothing path. The interview patterns are straightforward: **split files into parts, make retries safe, move bytes directly to object storage when possible, and keep metadata separate from raw file data**.

### Understanding Naive File Handling
The simplest approach to file uploads is accepting the entire file in a single HTTP request. The client opens a connection, sends all the bytes, and waits for one final response.

The bigger issue is that the operation is still long-lived, hard to resume, and often passes through infrastructure with request-size and timeout limits  
* Memory exhaustion  - buffering data in server before storing 
* Timeout failures   - HTTP timeouts, load balancer limits, and proxy configurations often kill these long-running requests before they complete.
* No resume capability  - wastes bandwidth, frustrates users, and puts unnecessary load on your infrastructure. 
* Single point of failure - only copy of in-progress data is lost because of server restart or deployment.
* Resource blocking - server resources like threads, connections, and CPU cycles are occupied which affects other users loading simple webpage

The fundamental problem is treating a large file as a single atomic operation.  
The solution is to break it into smaller, independent pieces that can be uploaded, verified, and resumed individually.
### Chunked Uploads
Instead of uploading a file as a single blob, **split it into fixed-size chunks and upload each chunk as an independent request.** If any chunk fails, you retry only that chunk rather than the entire file.

This transforms a single high-stakes operation into many small, low-risk operations. Each chunk is typically 4-64 MB, meaning a 2 GB file becomes 32-512 independent uploads. If one fails, you have lost minutes of progress, not hours.



**How It Works :**  
The upload happens in three phases: **initialization, chunk uploads, and completion.**  
**Step 1: Initialize Upload**  
The client starts by requesting an upload session  
```
POST /uploads/init
{
    "file_name": "video.mp4",
    "file_size": 1073741824,    // 1 GB
    "content_type": "video/mp4",
    "checksum": "sha256:abc123..."
}

Response:
{
    "upload_id": "up_xyz789",
    "chunk_size": 67108864,     // 64 MB
    "total_chunks": 16
}
```
The server creates a record tracking this upload session.  
**Step 2: Upload Chunks**  
Client uploads each chunk independently: using upload_id from above response  
```
PUT /uploads/up_xyz789/chunks/0
Content-Length: 67108864
X-Chunk-SHA256: <chunk_checksum>

[64 MB of binary data]

Response:
{
    "chunk_index": 0,
    "received": true,
    "checksum_valid": true
}
```
Each chunk upload is a separate HTTP request. Chunks can be uploaded in **parallel (with a concurrency limit) or sequentially**.  
**Step 3: Complete Upload**  
After all chunks are uploaded:  
```
POST /uploads/up_xyz789/complete

Response:
{
    "file_id": "file_abc123",
    "status": "complete",
    "url": "https://storage.example.com/files/file_abc123"
}
```
The server verifies all chunks are present, commits the final object, and creates the file record in the metadata database.   
If chunks were uploaded through the application, the server may compose them.   
If chunks were uploaded using a cloud multipart API, the storage service performs the final assembly.

**Resumable Uploads**  
Chunked uploads make resumability possible. When a connection drops, the client does not have to guess where it left off. It asks the server what chunks are missing using upload_id:
```
GET /uploads/up_xyz789/status

Response:
{
    "upload_id": "up_xyz789",
    "total_chunks": 16,
    "received_chunks": [0, 1, 2, 3, 4],  // Chunks 5-15 missing
    "bytes_received": 335544320          // 320 MB
}
```
**Idempotent Chunk Uploads**  
Retries introduce a subtle problem: what if the chunk made it to the server, but the acknowledgment was lost? The client thinks it failed, but the server already has the data. Uploading again would waste bandwidth and could create duplicate data.

The solution is to make each chunk addressable and idempotent. A chunk is identified by (upload_id, chunk_index) and verified with its size and checksum.  

**On retry** The server reads the headers first as the bytes stream in.checks its database/cache, sees (12345, Chunk 3, MD5) is already complete, immediately closes the connection or discards the incoming body stream, and instantly returns.   

Same upload_id, same chunk_index, same checksum: return success because the chunk is already stored.
Same upload_id, same chunk_index, different checksum: reject it because the client is trying to overwrite a different byte range.(can happen because client error)
The checksum protects integrity; the upload ID and chunk index provide idempotency.

**Choosing Chunk Size**
Chunk size is a trade-off between several factors:
| **Chunk Size** | **Pros** | **Cons** |
|---|---|---|
| **Small (1–4 MB)** | Fine-grained resume, low memory | More HTTP overhead, more round trips |
| **Medium (16–64 MB)** | Good balance for most use cases | Moderate retry cost |
| **Large (100+ MB)** | Fewer requests, less coordination | Higher retry cost, more memory |

* 4-8 MB for mobile clients where connections are unstable and failures are common.
* 16-64 MB for desktop and server clients with stable connections. This reduces HTTP overhead
* Adaptive chunk sizing is even better if you can implement it. Start with smaller chunks and increase the size if uploads are succeeding consistently.
* Provider constraints matter. For example, S3 multipart uploads require parts of at least 5 MB except the final part, allow up to 10,000 parts, and cap an object at 5 TB.

### Direct Upload with Pre-Signed URLs
Chunked uploads solve the resumability problem, but a server-proxied implementation still has a bottleneck: all data flows through your application servers.  
You may also pay twice for bandwidth: once to receive the data and once to forward it to storage.  
A better approach avoids proxying the data. Have clients upload directly to object storage like S3, GCS, or Azure Blob. Your application server handles authentication, policy, and metadata, while the file bytes move between the client and storage service.

**How Pre-Signed URLs Work**  
A pre-signed URL is a regular URL with embedded authorization. Your server signs it using storage credentials, but the URL itself can be used by anyone who has it, for a limited time and purpose.
```
https://storage.example.com/bucket/file.mp4
  ?X-Signature=abc123...
  &X-Expires=1702900000
  &X-Algorithm=AWS4-HMAC-SHA256
  ```
When the client uses this URL, the storage service verifies the signature and any signed constraints. If everything matches, the operation proceeds. The client never sees your credentials and cannot access objects beyond the signed scope.  
**Why This Matters**  
Direct upload has several benefits:
* Reduced server load - bypass file upload
* Lower bandwidth costs - don't pay for data transfer in your infra
* Better performance - cloud providers optimize & serve from near by s3.
* Independent scalability - s3 can be scaled   
**Pre-Signed URL Example**  
The function below uses the AWS SDK to generate one time-limited upload URL per chunk, **keying each object by upload ID and chunk index** so the server can track and assemble them independently:

**The Problem** 
If you upload **chunks directly as separate, independent temporary objects** in S3 (e.g., file_part1.bin, file_part2.bin), S3 treats them as completely unrelated files. S3 cannot instantly "merge" or "stitch" them together into your final file (file.mp4).

**The 2 Complications**  
* Extra Work & Delay (UploadPartCopy):  
To combine them, **you must start a formal S3 Multipart Upload** and tell S3 to copy each temporary object into the final destination using UploadPartCopy. S3 has to perform a copy operation for every single chunk behind the scenes, taking extra time and API calls.

* The 5 MB Limit Rule:  
S3 Multipart Upload enforces a strict 5 MB minimum size per part (except for the very last part). If any of your temporary chunk objects are smaller than 5 MB, UploadPartCopy will fail, and **you won't be able to assemble the final file.**  

**Security Mitigations**
* Short expiration
* Content validation - Sign required headers such as content type and checksum when supported.
* IP restrictions
* Monitor usage

### Multipart Upload Protocol
* If you are using cloud storage like S3, GCS, or Azure Blob, you usually do not need to build chunk assembly from scratch. These **services provide multipart or resumable upload protocols designed for large objects.**  
* Multipart upload combines the ideas we have discussed: parts, direct-to-object-storage transfers, retryable uploads, and a final completion step.  
* You still need to track upload state, retries, checksums, and cleanup, but object storage handles the final object assembly.

**S3 Multipart Upload Flow**
1. Initiate:
```
POST /bucket/object?uploads

Response:
<UploadId>xyz789</UploadId>
```
2. Upload Parts using uploadId:
```
PUT /bucket/object?partNumber=1&uploadId=xyz789
[Part 1 data]

Response:
ETag: "abc123"
```
Parts can be 5 MB to 5 GB. Maximum 10,000 parts, allowing objects up to 5 TB.

3. Complete using Etag:
```
POST /bucket/object?uploadId=xyz789
<CompleteMultipartUpload>
  <Part>
    <PartNumber>1</PartNumber>
    <ETag>"abc123"</ETag>
  </Part>
  ...
</CompleteMultipartUpload>
```
4. Abort (if needed):
```
DELETE /bucket/object?uploadId=xyz789
```
**Parallel Part Uploads (Default & Configurable)**  
One useful property of multipart upload is that parts can be uploaded in parallel. Since each part is independent, multiple threads, or even multiple machines, can upload different parts simultaneously
In practice, most upload clients default to 4-8 parallel streams. Going higher rarely helps because you hit other limits like network bandwidth, disk read speed, or connection overhead.

**Handling Incomplete Uploads**
Multipart uploads have a hidden cost: incomplete uploads consume storage but are invisible to normal listing. If a user starts an upload and never completes it(user cancels, application crash before calling complete API), those parts sit in storage, accumulating charges.

You need a cleanup strategy:  
* Lifecycle policies are the simplest solution. Configure S3 to automatically abort uploads older than a certain age
* Explicit tracking gives you more control. Track active uploads in your database with timestamps. 
* A background job periodically scans for uploads that have been pending too long and aborts them.


### Streaming Uploads
Everything we have discussed assumes you know the file size upfront. But what about live video streams, dynamically generated data, or files being compressed on the fly? You can **still send data in pieces, but you cannot precompute the total part count or a whole-file checksum before the stream finishes**.  
Streaming uploads handle this by not requiring Content-Length. Data flows to the server as it becomes available.

**Chunked Transfer Encoding**
* HTTP/1.1 supports chunked transfer encoding, which lets you send data in pieces without knowing the total size:
* Each transfer chunk is prefixed with its size in hexadecimal. A zero-length chunk signals the end. 
* These HTTP transfer chunks are not the same as application-level upload chunks; they are a wire format that lets the server process bytes as they arrive.

**Server-Side Handling**
* The server can process data as it streams in, avoiding the need to buffer everything in memory:
Client (Sends Stream) 
   ──[ TCP Bytes (~64KB) ]──> Server Memory (Holds 64KB) 
                                 │
                                 └──> Disk/Storage (Appends & Frees RAM)
* The entire file streams directly through a tiny memory buffer to permanent storage without consuming huge RAM.
* This works well for live data where you want to minimize latency between data creation and storage.

However, plain streaming uploads have significant limitations:

* **No resumability by default.** HTTP chunked transfer encoding does not give you an upload session, part numbers, or a resume protocol. **You can add checkpoints yourself**, but then you are effectively designing a resumable upload protocol.   
* **Late size verification.** You cannot validate the final size or whole-file checksum until after the stream ends. You can still validate per-block checksums if your protocol includes them.
* **Proxy issues.** Some proxies and load balancers buffer chunked requests before forwarding, defeating the purpose.

### After the Upload: Completion and Processing
**Reliable Completion**  
Every flow so far ends with the client calling POST /uploads/complete. That works when the client behaves, but it is a weak guarantee. The client can crash after the last chunk lands, lose its network before sending the completion call, or never come back.

If your system only marks a file ready when the client says so, those uploads sit in storage as orphaned bytes that no record points to.

Object stores solve this with event notifications. When an object is written, **the storage service emits an event** (for example, S3 sends an ObjectCreated event to SNS, SQS, or a function). The backend listens for these events and finalizes the file record itself, so completion does not depend on the client at all.

This is describing Single-Request Uploads (Direct-to-S3 single PUT requests) or Streaming Uploads, where S3 receives the exact file size upfront in the HTTP request headers.

** S3 Multipart Uploads (The Exception)** 
If you are using S3 Multipart Upload, S3 cannot trigger an ObjectCreated event automatically per object because:

S3 treats each chunk as an isolated part.

S3 has no idea how many total parts you plan to upload.

For Multipart Uploads, someone must send the CompleteMultipartUpload API request to tell S3 to stitch the parts together. To make this reliable without trusting the client, companies do one of two things:

Client sends completion to S3 directly: The client calls S3's CompleteMultipartUpload. S3 stitches the file and then fires the s3:ObjectCreated:CompleteMultipartUpload event to your backend.

Backend handles completion: The client notifies your backend server when all chunks are done, and your reliable backend server calls CompleteMultipartUpload on S3.

**The Processing Pipeline**
* For many systems, a raw uploaded file is not directly usable
* . A video needs transcoding into multiple resolutions and bitrates. An image needs thumbnails. A document needs text extraction and indexing for search. Almost any user upload should be scanned for malware before it is served to other users.
* The standard approach for processing file is to run it asynchronously: the upload finalizes quickly, then a message goes onto a queue, and a pool of workers processes the file in the background.
* This is why file records usually carry a status such as uploaded, processing, and ready. 

### Download Optimizations
* A naive download suffers from the same problems as a naive upload: no resumability, slow transfer on single connections, and poor user experience when something goes wrong.
* Fortunately, HTTP has built-in features that address these issues.
### Range Request
* HTTP range requests allow downloading specific byte ranges rather than the entire file
```
GET /files/video.mp4
Range: bytes=0-1048575

Response:
HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1048575/104857600
Content-Length: 1048576

[First 1 MB of data]
```
* The 206 Partial Content status code indicates that the server is returning only part of the file
* This enables three important capabilities: Resumable downloads/Parallel downloads/Video seeking

**Parallel Downloads**
* Parallel downloads work the same way as parallel uploads. Split the file into ranges and fetch them concurrently
* The client reassembles the chunks into the complete file. This saturates available bandwidth better than a single connection, especially on high-latency links where TCP congestion control limits individual connection speed.

### CDN Distribution
* A Content Delivery Network solves this by caching files at edge servers distributed around the world

For large files, CDNs offer several advantages:

Lower latency. Users connect to nearby edge servers rather than distant origin servers.
Higher throughput. Edge servers are optimized for high-bandwidth transfers.
Reduced origin load. Popular files are served from cache, protecting your origin from traffic spikes.
Geographic redundancy. If one edge goes down, traffic routes to another.

To make your files CDN-friendly, set appropriate cache headers:
```
Cache-Control: public, max-age=31536000, immutable
ETag: "abc123"
Accept-Ranges: bytes
```
The Accept-Ranges: bytes header tells clients that range requests are supported.
Use long max-age values only for immutable, versioned object names. 

### Authorizing Downloads
* The same direct-path idea from uploads applies in reverse: rather than streaming private bytes through your application, the API server authorizes the request and then hands back a short-lived signed URL (or signed cookie) that points at object storage or the CDN edge.
* The client asks the API server for a file. The server checks that the caller is allowed to read it, then returns a pre-signed GET URL scoped to that one object with a short expiry, typically 15 to 60 minutes. The client downloads directly from storage or the CDN using that URL.

### Storage Architecture
So far we have focused on how to transfer large files. But where should they live once they arrive? The answer depends on your access patterns, scale, and consistency requirements.

#### Choosing the Right Storage
| Storage Type | Best For | Avoid When |
| :--- | :--- | :--- |
| **Object Storage** (S3, GCS, Azure Blob) | Large immutable objects, static assets, backups | Low-latency random writes inside files |
| **Distributed File System** (HDFS, GlusterFS) | Analytics workloads, shared access across nodes | Small files, web serving |
| **Database** (BLOB columns) | Small files (<1 MB), tight consistency with related data | Large files (performance degrades badly) |
* For most web applications, object storage is the right choice. It is designed for large objects, scales well, and integrates with CDNs and direct upload patterns
#### Separating Metadata from Data
* A common mistake is treating file storage as a single problem. **In practice, you have two distinct concerns.** The **metadata** is the file name, size, owner, timestamps, permissions, and the location of the actual data. The **data is the actual file bytes**.

* Storing these together in a traditional database works for small files, but breaks down quickly as files grow. Instead, **store metadata in a database optimized for queries, and store file data in object storage optimized for large objects**
* This separation provides several benefits:
1. Independent scaling
2. Efficient queries
3. Clear consistency boundaries - Keep the metadata database as the source of truth.
4. Different retention policies - You might keep metadata forever for auditing, but delete the actual files after a retention period

#### Content-Addressable Storage
* Instead of storing files at arbitrary paths, compute a hash of the file content and use that hash as the storage key:
```
File: vacation.mp4
SHA-256: 3b4c5d6e7f8a9b0c...
Storage path: /blobs/3b/4c/3b4c5d6e7f8a9b0c...
```
* The path is derived from the hash, typically using the first few characters as directory prefixes
This approach has several useful properties:

Whole-file deduplication
If two users upload the same file, it has the same hash and can be stored once. The metadata records point to the same object. For systems where users often share common files like popular PDFs or media, this can save significant storage.
Immutability
The content cannot change without changing the hash. This makes integrity verification trivial: re-hash the content and compare. If they match, the file is intact.
Safe concurrent uploads ??

### Compression and Deduplication
#### Compression Trade-offs
| Approach | Pros | Cons |
| :--- | :--- | :--- |
| **Client-side compression** | Reduces upload bandwidth | CPU cost on client, requires client support |
| **Server-side compression** | Transparent to client | Server CPU cost, delayed storage |
| **Storage-level compression** | Automatic, transparent | May not work for already-compressed formats |
#### Block Level Deduplication
* Block-level deduplication solves this by splitting files into blocks and deduplicating at the block level:
* File B shares blocks 1 and 3 with File A. Only block 2 is different, so only one new block is stored.
* Backup and sync systems use this idea to upload only changed blocks when a small part of a large file changes.

#### Content-Defined Chunking
* All three fixed-size chunks changed even though we only inserted one character. In real systems, this boundary-shift problem can greatly reduce deduplication.

* Content-Defined Chunking (CDC) solves this by using the content itself to find chunk boundaries. Instead of cutting at fixed intervals, it looks for specific patterns in the data (typically using a rolling hash like Rabin fingerprinting). When the pattern appears, that becomes a chunk boundary.

* With CDC, many original chunks are preserved because boundaries are based on content patterns rather than fixed offsets

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
