# Caching

## Why Caching Matters

## The Caching Hierarchy
### Client-Side Caching (Browser)
### CDN Caching
### Load Balancer / API Gateway Caching
### Application-Level Caching
### Database Caching

## Cache Key Design

## Caching Strategies for Reads
### Cache-Aside (Lazy Loading)
### Read-Through
### Refresh-Ahead

## Caching Strategies for Writes
### Write-Through
### Write-Behind (Write-Back)
### Write-Around
### Cache Invalidation on Write

## Cache Eviction Policies
### LRU (Least Recently Used)
### LFU (Least Frequently Used)
### FIFO (First In, First Out)
### TTL (Time To Live)
### Random Replacement
### Advanced Policies

## Distributed Caching
### Distributed Cache Architecture
### Data Partitioning
### Replication
### Two-Tier Caching
### Hot Keys

## Cache Consistency and Invalidation
### Invalidation Strategies
#### TTL-Based Expiration
#### Event-Based Invalidation
#### Write-Through
#### Publish-Subscribe Invalidation
### Dealing with Race Conditions
#### Delayed Double Deletion
#### Cache Versioning
#### Distributed Locks
### Cache Stampede Prevention
#### Locking (Single Flight)
#### Probabilistic Early Expiration
#### Stale-While-Revalidate
### Cache Penetration and Avalanche
#### Negative Caching
#### Bloom Filter
### Consistency Levels
#### Strong
#### Eventual
#### Weak
### Cache Failures and Resilience
#### Failure Modes
#### Handling Cache Unavailability
#### Preventing Cascading Failures
#### Cache Warming
#### Cache Observability

## Caching Patterns in Practice
### User Session Caching
### Feed / Timeline Caching
### Leaderboard Caching
### Rate Limiting with Cache
### URL Shortener Cache
### E-commerce Product Cache
