# Read Path


## Load Home Feed

1. Client requests '/feed'
2. API checks cache
3. Cache hit -> return data
4. cache miss:
    - Query read replica
    - Aggregate posts + users + counters
    - Store result in cache
5. Return response to client


## Optimizations
- Cache feed pages
- Precomputed counters
- Pagination
