# Estimations

## Assumptions
- 1M MAU  
- 100K DAU

## Behavioral Assumptions
- Reads per user/day = 20
- Writes (posts) 2% of DAU 
- comments 10% of DAU

## Post Content
- Each post contains
    - Text content (required)
    - Optional media
- Media is stored outside the core database
- backend stores only media URLs

### Query per second estimate: 
**writes:**
- posts/day = 100,000 * 0.02 = 2,000 posts/day
- comments/day = 100,000 * 0.1 = 10,000

    * 2,000 posts/day ÷ 86,400 = 0.02 QPS
    * 10,000 comments/day ÷ 86,400 = 0.12 QPS
        
- Total QPS = 0.14 QPS
- Peak (*5) = 1  write QPS

**Reads:**
- 20 reads/user/day  23 QPS
- peak (*5) = 23 * 5 = 115 QPS

### Storage:
    | Type     | Avg Size | Daily | Yearly |
    |----------|----------|-------|--------|
    | Posts    | 1 KB     | 2 MB  | 730 MB |
    | Comments | 0.3 KB   | 3 MB  | 1.1 GB |
    | **Total**|          | 5 MB  | 2 GB   |


### Conclusion: 
- Read-heavy system
- writes are low -> simple primary DB is good
- reads need: 
    - cache
    - CDN (for media)
    - possibly read replicas
