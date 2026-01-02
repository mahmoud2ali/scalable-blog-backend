# Bottleneck Analysis

## Read Traffic   
- 115 peak read QPS
- most users hit: 
    - home feed
    - post details    
    - comments list
- Poor p95 / p99 response time

**Fix:**
- cache:
    - feed pages
    - post metadata
    - comment list

## Database Read Amplification

* One feed request = multiple queries:
    - posts
    - users
    - likes
    - comments count

**Fix:**
- denormalized read models
- precomputed counters

## Cache Invalidation Complexity
- Writes are rare
- Reads are frequent
- But when post/comment is created:
    - Feed cache becomes stale

**Fix:** 
- cache-aside strategy
    

## Comments Fan-Out
- popular posts -> many comments
- fetching comments every time is expensive

**Fix:** 
- paginated comments
- cache first N comments
- lazy loading
