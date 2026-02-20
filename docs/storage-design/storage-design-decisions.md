# Storage Design Decisions — Blog Backend

This document connects storage engine concepts to the blog backend design.

The goal is to understand how internal database structures affect system behavior under load and at scale.

---

# 1. Storage Engine Choice

The blog backend uses a relational database that relies primarily on **B-Tree indexes**.

B-Trees are optimized for:

- Read-heavy workloads
- Efficient point lookups
- Efficient range queries
- Predictable latency

Our blog system is:

- Read-heavy (feed + comments)
- Moderately write-heavy (posts + comments)
- Requires ordered results (recent posts first)

Therefore, a B-Tree-based engine is appropriate at current scale.

---

# 2. Mapping Functional Requirements to Storage Engine Behavior

## 2.1 User Signup / Login

### Operations
- INSERT user
- SELECT user WHERE email = ?

### Storage Implication

A B-Tree index on `email`:

- Performs lookup in O(log n)
- Navigates tree levels using page reads
- Upper tree levels are typically cached in memory (buffer pool)

Without an index:
- Full table scan
- O(n) disk reads
- High read amplification

### Scaling Risk

If user table grows beyond available memory:

- More disk seeks
- Increased latency due to random I/O
- Reduced cache hit rate

This is directly tied to how B-Trees access disk pages.

---

## 2.2 Create Post

### Operation
- INSERT into Posts

### Storage Behavior in B-Tree

- Insert into sorted structure
- May trigger page split
- Causes random disk writes

This creates **write amplification**:

- One logical write
- Multiple physical page writes

B-Trees trade higher write cost for efficient reads.

### When It Becomes a Problem

If write rate becomes extremely high:

- Frequent page splits
- Disk IOPS becomes bottleneck
- Latency becomes unstable

An LSM-tree engine could improve write throughput by:

- Writing sequentially to SSTables
- Performing compaction later

---

## 2.3 Read Posts (Feed)

### Query

SELECT * FROM Posts
ORDER BY CreatedAt DESC
LIMIT 20;

### Why This Is a Range Query

This query:

- Reads newest keys
- Scans a small sorted range
- Depends on ordering

B-Trees store keys in sorted order across leaf pages.

This enables:

- Efficient range scan
- Sequential page reads
- Low read amplification

Without index on `CreatedAt`:

- Full table scan
- Large disk I/O
- Slow feed generation

This is where B-Trees perform better than hash-based indexing.

---

## 2.4 Comment on Post

### Operations
- INSERT comment
- SELECT WHERE PostId = ?

This creates a secondary index on `PostId`.

### Risk

A viral post creates:

- Heavy reads on same key range
- “Hot” pages in B-Tree
- Buffer pool contention
- Disk page contention

This is a hotspot problem.

### Mitigation

- Cache popular comment threads
- Paginate comments
- Eventually shard by PostId

---

# 3. Write Amplification

In B-Tree systems, write amplification occurs due to:

- Page splits
- Rebalancing
- Updating indexed columns

Effect:

- Multiple disk writes per logical write
- Increased latency under heavy write load

In blog system:

- High comment creation rate increases write amplification
- Large indexes increase maintenance cost

Current risk level: Moderate.

---

# 4. Read Amplification

Read amplification occurs when:

- Query requires reading many disk pages
- Index is missing or not selective
- Rows are large
- Data locality is poor

Examples:

- Fetching posts without CreatedAt index
- Fetching comments without PostId index
- Large comment threads with poor pagination

Mitigation:

- Proper indexing
- Query optimization
- Caching layer
- Read replicas

---

# 5. Memory and Disk Interaction

B-Trees rely heavily on:

- Buffer pool caching
- Keeping upper tree levels in memory

As dataset grows:

- Tree height increases
- More disk page reads required
- Cache miss rate increases
- Latency becomes less predictable

Scalability is directly tied to:

- RAM size
- Disk IOPS
- Index design

---

# 6. When This Architecture Breaks

This storage model may struggle when:

1. Write traffic becomes extremely high → page split pressure
2. Dataset greatly exceeds RAM → disk seeks dominate
3. Feed traffic spikes → buffer pool thrashing
4. Viral posts create read hotspots
5. Complex queries require scanning large ranges

At that stage, possible solutions include:

- Read replicas
- Caching layer
- Sharding
- Switching heavy-write workloads to LSM-based engine

---

# 7. Why This Design Is Appropriate (Current Scale)

The blog system:

- Has predictable read-heavy workload
- Requires ordered queries
- Does not have extreme write throughput
- Benefits from strong consistency

Therefore:

B-Tree-based relational storage provides:

- Efficient range scans
- Stable latency
- Mature ecosystem
- Simpler operational complexity

---

# 8. Key Engineering Insight

Storage engine internals determine:

- Latency profile
- Scalability ceiling
- Infrastructure cost
- Failure modes under load

Understanding B-Tree vs LSM tradeoffs allows intentional scaling decisions instead of reactive fixes.

This document ensures that storage decisions in the blog backend are intentional, measurable, and scalable.