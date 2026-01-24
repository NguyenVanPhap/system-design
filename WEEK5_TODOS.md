# Week 5 – Caching & Performance Optimization

> **Mentor Note**: Caching là một trong những optimizations quan trọng nhất. Nhưng cache sai cách sẽ làm system chậm hơn, không nhanh hơn. Bạn phải MEASURE mọi thứ. Cache hit rate, latency improvement, memory usage - tất cả phải có numbers. Không có "có vẻ nhanh hơn". Phải có benchmarks, metrics, và proof. Production systems cần performance, không cần assumptions.

---

## Study TODOs

### Caching Fundamentals
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 3 - Storage and Retrieval (caching sections)
- [ ] Định nghĩa chính xác: Cache hit, cache miss, cache hit rate
- [ ] Viết công thức: Cache hit rate = ?
- [ ] Đọc về "cache hierarchy" - L1, L2, L3 caches
- [ ] Đọc về "locality of reference" - spatial và temporal
- [ ] Đọc về "cache coherence" - consistency across caches
- [ ] Research: When caching helps? When it doesn't?
- [ ] Đọc về "cache size" calculation - how much to cache?

### Cache Patterns Deep Dive
- [ ] Đọc về "Cache-Aside" pattern (Lazy Loading)
- [ ] Viết algorithm pseudocode cho Cache-Aside
- [ ] Analyze: Pros và cons của Cache-Aside (5 points each)
- [ ] Đọc về "Write-Through" pattern
- [ ] Viết algorithm pseudocode cho Write-Through
- [ ] Analyze: Pros và cons của Write-Through (5 points each)
- [ ] Đọc về "Write-Behind" (Write-Back) pattern
- [ ] Viết algorithm pseudocode cho Write-Behind
- [ ] Analyze: Pros và cons của Write-Behind (5 points each)
- [ ] Compare: Cache-Aside vs Write-Through vs Write-Behind (10 criteria)
- [ ] Research: Real-world examples của mỗi pattern
- [ ] Analyze: When to use which pattern? (5 use cases each)

### Cache Invalidation Strategies
- [ ] Đọc về "TTL" (Time-To-Live) invalidation
- [ ] Analyze: Pros và cons của TTL (5 points each)
- [ ] Đọc về "event-driven invalidation"
- [ ] Analyze: Pros và cons của event-driven (5 points each)
- [ ] Đọc về "explicit invalidation" (manual delete)
- [ ] Đọc về "version-based invalidation"
- [ ] Đọc về "cache stampede" problem
- [ ] Research: Strategies to prevent cache stampede
- [ ] Đọc về "thundering herd" problem
- [ ] Research: Solutions cho thundering herd
- [ ] Compare: Invalidation strategies (5 criteria)

### Redis Data Structures
- [ ] Đọc về Redis String - use cases
- [ ] Đọc về Redis Hash - use cases
- [ ] Đọc về Redis List - use cases
- [ ] Đọc về Redis Set - use cases
- [ ] Đọc về Redis Sorted Set - use cases
- [ ] Đọc về Redis Bitmap - use cases
- [ ] Đọc về Redis HyperLogLog - use cases
- [ ] Đọc về Redis Streams - use cases
- [ ] Compare: Data structures (when to use which?)
- [ ] Research: Memory efficiency của mỗi structure
- [ ] Research: Performance characteristics của mỗi structure

### Cache Problems & Solutions
- [ ] Đọc về "cache breakdown" (cache miss storm)
- [ ] Analyze: What causes cache breakdown?
- [ ] Research: Solutions cho cache breakdown
- [ ] Đọc về "cache penetration" (query non-existent data)
- [ ] Analyze: What causes cache penetration?
- [ ] Research: Solutions cho cache penetration (bloom filter, etc.)
- [ ] Đọc về "hot key" problem
- [ ] Analyze: What causes hot keys?
- [ ] Research: Solutions cho hot keys (local cache, sharding, etc.)
- [ ] Đọc về "big key" problem
- [ ] Analyze: What causes big keys?
- [ ] Research: Solutions cho big keys (compression, splitting, etc.)

### Distributed Caching
- [ ] Đọc về "Redis Cluster" - how it works
- [ ] Đọc về "Redis Sentinel" - high availability
- [ ] Đọc về "consistent hashing" trong Redis Cluster
- [ ] Đọc về "replication" trong Redis
- [ ] Đọc về "persistence" - RDB vs AOF
- [ ] Compare: RDB vs AOF (5 criteria)
- [ ] Đọc về "Redis Pub/Sub" - use cases
- [ ] Research: Redis performance tuning

### Distributed Locking
- [ ] Đọc về "distributed locks" - why needed?
- [ ] Đọc về "Redis SET NX" - basic locking
- [ ] Đọc về "deadlock" trong distributed locks
- [ ] Đọc về "lock expiration" - why critical?
- [ ] Đọc về "Redlock algorithm" - distributed lock algorithm
- [ ] Analyze: Pros và cons của Redlock
- [ ] Đọc về "lease-based locking"
- [ ] Research: Distributed lock implementations
- [ ] Đọc về "lock contention" - performance impact

### Rate Limiting
- [ ] Đọc về "rate limiting" - why needed?
- [ ] Đọc về "Token Bucket" algorithm
- [ ] Viết algorithm pseudocode cho Token Bucket
- [ ] Đọc về "Leaky Bucket" algorithm
- [ ] Viết algorithm pseudocode cho Leaky Bucket
- [ ] Đọc về "Sliding Window" algorithm
- [ ] Viết algorithm pseudocode cho Sliding Window
- [ ] Đọc về "Fixed Window" algorithm
- [ ] Compare: Rate limiting algorithms (5 criteria)
- [ ] Research: Redis-based rate limiting implementations

### Performance Optimization
- [ ] Đọc về "latency" vs "throughput" trade-offs
- [ ] Đọc về "pipelining" trong Redis
- [ ] Đọc về "batch operations" - performance improvement
- [ ] Đọc về "connection pooling" cho Redis
- [ ] Đọc về "serialization" - JSON vs MessagePack vs Protobuf
- [ ] Compare: Serialization formats (performance, size)
- [ ] Đọc về "compression" - when to compress cache?
- [ ] Research: Cache performance best practices

---

## Redis Integration TODOs

### Task 1: Redis Setup & Basic Operations
- [ ] Install: Redis server (local hoặc Docker)
- [ ] Setup: Spring Boot với Redis dependency
- [ ] Configure: Redis connection (host, port, password)
- [ ] Configure: Connection pool (Lettuce hoặc Jedis)
- [ ] Test: Basic SET/GET operations
- [ ] Test: Connection pooling
- [ ] Measure: Basic operation latency
- [ ] Document: Redis setup

### Task 2: Cache-Aside Pattern Implementation
- [ ] Create: Service class với cache-aside pattern
- [ ] Implement: getData(key) - check cache first, then DB
- [ ] Implement: updateData(key, value) - update DB, invalidate cache
- [ ] Add: Cache miss logging
- [ ] Add: Cache hit logging
- [ ] Test: Cache hit scenario
- [ ] Test: Cache miss scenario
- [ ] Measure: Latency improvement với cache
- [ ] Measure: Cache hit rate
- [ ] Document: Cache-Aside implementation

### Task 3: Write-Through Pattern Implementation
- [ ] Implement: Write-Through pattern
- [ ] Method: writeData(key, value) - write to cache và DB
- [ ] Ensure: Atomicity (both succeed or both fail)
- [ ] Test: Write success scenario
- [ ] Test: Write failure scenario (DB fails)
- [ ] Test: Write failure scenario (Cache fails)
- [ ] Measure: Write latency
- [ ] Compare: Write-Through vs Cache-Aside write performance
- [ ] Document: Write-Through implementation

### Task 4: Write-Behind Pattern Implementation
- [ ] Implement: Write-Behind pattern
- [ ] Method: writeData(key, value) - write to cache, async to DB
- [ ] Implement: Async DB write queue
- [ ] Implement: Retry mechanism cho failed writes
- [ ] Test: Write success scenario
- [ ] Test: DB write failure (retry)
- [ ] Test: Data loss prevention
- [ ] Measure: Write latency improvement
- [ ] Document: Write-Behind implementation

### Task 5: Redis Data Structures Usage
- [ ] Implement: String operations (user profile cache)
- [ ] Implement: Hash operations (user session cache)
- [ ] Implement: List operations (recent activities)
- [ ] Implement: Set operations (user tags)
- [ ] Implement: Sorted Set operations (leaderboard)
- [ ] Test: Each data structure với realistic data
- [ ] Measure: Memory usage của mỗi structure
- [ ] Measure: Operation performance
- [ ] Document: Data structure selection guide

### Task 6: Cache Invalidation Implementation
- [ ] Implement: TTL-based invalidation
- [ ] Configure: TTL cho different data types
- [ ] Test: TTL expiration
- [ ] Implement: Event-driven invalidation
- [ ] Publish: Invalidation events
- [ ] Subscribe: Invalidation events
- [ ] Test: Event-driven invalidation
- [ ] Implement: Explicit invalidation (delete)
- [ ] Test: All invalidation methods
- [ ] Document: Invalidation strategy

### Task 7: Cache Warming Implementation
- [ ] Identify: Hot data (frequently accessed)
- [ ] Implement: Cache warming service
- [ ] Load: Hot data vào cache on startup
- [ ] Schedule: Periodic cache warming
- [ ] Monitor: Cache hit rate before và after warming
- [ ] Measure: Performance improvement
- [ ] Document: Cache warming strategy

### Task 8: Spring Cache Abstraction
- [ ] Configure: Spring Cache với Redis
- [ ] Annotate: Methods với @Cacheable
- [ ] Annotate: Methods với @CacheEvict
- [ ] Annotate: Methods với @CachePut
- [ ] Configure: Cache names và TTL
- [ ] Test: Cache annotations work correctly
- [ ] Measure: Performance với Spring Cache
- [ ] Document: Spring Cache configuration

---

## Performance Testing TODOs

### Benchmark 1: Cache Hit Rate Measurement
- [ ] Setup: Application với cache
- [ ] Generate: Realistic load (1000 requests)
- [ ] Measure: Cache hit rate
- [ ] Target: Cache hit rate > 80%
- [ ] Analyze: If hit rate < 80%, identify reasons
- [ ] Optimize: Cache strategy to improve hit rate
- [ ] Re-measure: Cache hit rate after optimization
- [ ] Document: Hit rate analysis và improvements

### Benchmark 2: Latency Improvement
- [ ] Measure: Baseline latency (without cache)
- [ ] Measure: Latency với cache (p50, p95, p99)
- [ ] Calculate: Latency improvement (%)
- [ ] Target: Latency improvement > 50%
- [ ] Analyze: If improvement < 50%, identify bottlenecks
- [ ] Optimize: Cache configuration
- [ ] Re-measure: Latency after optimization
- [ ] Document: Latency improvement analysis

### Benchmark 3: Throughput Testing
- [ ] Measure: Baseline throughput (QPS without cache)
- [ ] Measure: Throughput với cache (QPS)
- [ ] Calculate: Throughput improvement (%)
- [ ] Load test: With increasing load (100, 500, 1000, 5000 QPS)
- [ ] Identify: Breaking point (when cache doesn't help)
- [ ] Analyze: Bottleneck at high load
- [ ] Document: Throughput analysis

### Benchmark 4: Memory Usage Analysis
- [ ] Measure: Redis memory usage
- [ ] Calculate: Memory per cached item
- [ ] Estimate: Total memory needed cho full cache
- [ ] Configure: Max memory và eviction policy
- [ ] Test: Eviction behavior khi memory full
- [ ] Analyze: Memory efficiency
- [ ] Optimize: Reduce memory usage (compression, data structure choice)
- [ ] Document: Memory usage analysis

### Benchmark 5: Cache Pattern Comparison
- [ ] Implement: Cache-Aside pattern
- [ ] Measure: Read latency, write latency, hit rate
- [ ] Implement: Write-Through pattern
- [ ] Measure: Read latency, write latency, hit rate
- [ ] Implement: Write-Behind pattern
- [ ] Measure: Read latency, write latency, hit rate
- [ ] Compare: All patterns (5 criteria)
- [ ] Recommend: Best pattern cho use case
- [ ] Document: Pattern comparison

### Benchmark 6: Serialization Performance
- [ ] Test: JSON serialization (Jackson)
- [ ] Measure: Serialization time, deserialization time, size
- [ ] Test: MessagePack serialization
- [ ] Measure: Serialization time, deserialization time, size
- [ ] Test: Protobuf serialization (if applicable)
- [ ] Measure: Serialization time, deserialization time, size
- [ ] Compare: All serialization formats
- [ ] Recommend: Best format cho use case
- [ ] Document: Serialization comparison

### Benchmark 7: Redis Operations Performance
- [ ] Test: SET operation performance
- [ ] Test: GET operation performance
- [ ] Test: HSET/HGET operations performance
- [ ] Test: Pipeline operations performance
- [ ] Measure: Latency của mỗi operation
- [ ] Compare: Individual vs pipelined operations
- [ ] Optimize: Use pipelining where beneficial
- [ ] Document: Operations performance

---

## Cache Failure TODOs

### Failure Scenario 1: Cache Breakdown
- [ ] Simulate: Cache breakdown (all keys expire simultaneously)
- [ ] Scenario: Cache miss storm hits database
- [ ] Measure: Database load spike
- [ ] Measure: Application latency spike
- [ ] Implement: Solution (staggered TTL, cache warming)
- [ ] Test: Solution effectiveness
- [ ] Document: Cache breakdown mitigation

### Failure Scenario 2: Cache Penetration
- [ ] Simulate: Cache penetration (query non-existent keys)
- [ ] Scenario: Attacker queries random non-existent user IDs
- [ ] Measure: Cache miss rate
- [ ] Measure: Database load
- [ ] Implement: Solution (bloom filter, cache empty results)
- [ ] Test: Solution effectiveness
- [ ] Document: Cache penetration mitigation

### Failure Scenario 3: Hot Key Problem
- [ ] Simulate: Hot key (single key gets 80% of requests)
- [ ] Scenario: Popular product page, single cache key
- [ ] Measure: Redis CPU usage
- [ ] Measure: Network bandwidth
- [ ] Identify: Hot key bottleneck
- [ ] Implement: Solution (local cache, key sharding)
- [ ] Test: Solution effectiveness
- [ ] Document: Hot key mitigation

### Failure Scenario 4: Big Key Problem
- [ ] Simulate: Big key (single key stores 10MB data)
- [ ] Scenario: Large list hoặc hash trong single key
- [ ] Measure: Memory usage
- [ ] Measure: Network transfer time
- [ ] Identify: Big key bottleneck
- [ ] Implement: Solution (split key, compression)
- [ ] Test: Solution effectiveness
- [ ] Document: Big key mitigation

### Failure Scenario 5: Redis Failure
- [ ] Simulate: Redis server down
- [ ] Test: Application behavior (fail gracefully?)
- [ ] Measure: Latency impact (fallback to DB)
- [ ] Implement: Circuit breaker cho Redis
- [ ] Implement: Fallback mechanism
- [ ] Test: Circuit breaker behavior
- [ ] Test: Recovery when Redis comes back
- [ ] Document: Redis failure handling

### Failure Scenario 6: Cache Stampede
- [ ] Simulate: Cache stampede (many requests for same expired key)
- [ ] Scenario: Popular key expires, 1000 requests simultaneously
- [ ] Measure: Database load spike
- [ ] Implement: Solution (mutex lock, single request loads)
- [ ] Test: Solution effectiveness
- [ ] Document: Cache stampede mitigation

### Failure Scenario 7: Thundering Herd
- [ ] Simulate: Thundering herd (many requests after cache miss)
- [ ] Scenario: Cache miss, all requests hit database
- [ ] Measure: Database connection pool exhaustion
- [ ] Implement: Solution (request coalescing, queuing)
- [ ] Test: Solution effectiveness
- [ ] Document: Thundering herd mitigation

### Degradation Testing
- [ ] Test: System behavior với 0% cache hit rate
- [ ] Measure: Performance degradation
- [ ] Test: System behavior với 50% cache hit rate
- [ ] Measure: Performance
- [ ] Test: System behavior với 90% cache hit rate
- [ ] Measure: Performance
- [ ] Analyze: Performance vs cache hit rate correlation
- [ ] Document: Degradation analysis

---

## Concurrency & Lock TODOs

### Task 1: Distributed Lock Implementation
- [ ] Implement: Distributed lock với Redis SET NX
- [ ] Method: acquireLock(key, timeout)
- [ ] Method: releaseLock(key)
- [ ] Add: Lock expiration (prevent deadlock)
- [ ] Test: Lock acquisition
- [ ] Test: Lock release
- [ ] Test: Lock expiration
- [ ] Test: Concurrent lock attempts
- [ ] Document: Distributed lock implementation

### Task 2: Redlock Algorithm Implementation
- [ ] Implement: Redlock algorithm
- [ ] Setup: Multiple Redis instances (3-5)
- [ ] Implement: Lock acquisition across instances
- [ ] Implement: Quorum-based locking
- [ ] Test: Lock với majority of instances
- [ ] Test: Failure scenarios (some instances down)
- [ ] Measure: Lock acquisition time
- [ ] Document: Redlock implementation

### Task 3: Lease-Based Locking
- [ ] Implement: Lease-based locking
- [ ] Add: Lock renewal mechanism
- [ ] Add: Heartbeat để keep lock alive
- [ ] Test: Lock renewal
- [ ] Test: Lock expiration if renewal fails
- [ ] Test: Concurrent renewal attempts
- [ ] Document: Lease-based locking

### Task 4: Lock Contention Analysis
- [ ] Simulate: High lock contention (many threads compete)
- [ ] Measure: Lock acquisition time
- [ ] Measure: Throughput degradation
- [ ] Identify: Contention hotspots
- [ ] Implement: Solution (lock sharding, optimistic locking)
- [ ] Test: Solution effectiveness
- [ ] Document: Lock contention analysis

### Task 5: Optimistic Locking Implementation
- [ ] Implement: Optimistic locking với version field
- [ ] Add: Version check before update
- [ ] Handle: OptimisticLockException
- [ ] Test: Concurrent updates
- [ ] Test: Conflict resolution
- [ ] Compare: Optimistic vs pessimistic locking performance
- [ ] Document: Optimistic locking implementation

### Task 6: Rate Limiting Implementation
- [ ] Implement: Token Bucket algorithm với Redis
- [ ] Method: checkRateLimit(key, limit, window)
- [ ] Test: Rate limiting enforcement
- [ ] Test: Rate limit reset
- [ ] Measure: Performance overhead
- [ ] Implement: Sliding Window algorithm
- [ ] Compare: Token Bucket vs Sliding Window
- [ ] Document: Rate limiting implementation

### Task 7: Atomic Operations
- [ ] Implement: Atomic increment với Redis INCR
- [ ] Test: Concurrent increments
- [ ] Verify: No race conditions
- [ ] Implement: Atomic compare-and-set
- [ ] Test: CAS operations
- [ ] Measure: Atomic operation performance
- [ ] Document: Atomic operations usage

### Task 8: Race Condition Testing
- [ ] Identify: Potential race conditions trong cache code
- [ ] Simulate: Race conditions với concurrent requests
- [ ] Test: Cache update race condition
- [ ] Test: Cache invalidation race condition
- [ ] Fix: Race conditions với proper locking
- [ ] Re-test: After fixes
- [ ] Document: Race condition analysis và fixes

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Redis Integration tasks hoàn thành?
- [ ] Review: Tất cả Performance Testing hoàn thành?
- [ ] Review: Tất cả Cache Failure scenarios tested?
- [ ] Review: Tất cả Concurrency tasks hoàn thành?
- [ ] Rate: Caching pattern understanding (1-10)
- [ ] Rate: Redis implementation skills (1-10)
- [ ] Rate: Performance optimization skills (1-10)
- [ ] Rate: Failure handling skills (1-10)
- [ ] Identify: 3 caching concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Performance Review
- [ ] Review: All benchmark results
- [ ] Verify: Performance targets met?
- [ ] Identify: Top 3 performance improvements made
- [ ] Identify: Remaining bottlenecks
- [ ] Analyze: Cache hit rate trends
- [ ] Analyze: Latency improvement trends
- [ ] Create: Performance improvement roadmap
- [ ] Document: Performance review findings

### Code Review
- [ ] Review: Cache implementation code
- [ ] Check: Cache patterns implemented correctly?
- [ ] Check: Error handling adequate?
- [ ] Check: Logging và monitoring?
- [ ] Review: Distributed lock implementation
- [ ] Check: Deadlock prevention?
- [ ] Review: Rate limiting implementation
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Architecture Review
- [ ] Review: Caching architecture design
- [ ] Check: Multi-layer caching appropriate?
- [ ] Check: Cache invalidation strategy optimal?
- [ ] Check: Failure handling adequate?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Compare: Design vs industry best practices
- [ ] Document: Architecture review

### Knowledge Check
- [ ] Explain: Cache-Aside vs Write-Through vs Write-Behind (viết comparison, không xem notes)
- [ ] Explain: Cache breakdown vs penetration (viết differences, không xem notes)
- [ ] Explain: Hot key problem và solutions (viết analysis, không xem notes)
- [ ] Explain: Distributed locking với Redis (viết implementation approach, không xem notes)
- [ ] Explain: Token Bucket vs Sliding Window (viết comparison, không xem notes)
- [ ] Solve: Cache hit rate = 70%, DB query = 100ms, cache read = 5ms. Calculate average latency
- [ ] Solve: 1000 requests/second, cache hit rate = 80%. Calculate database load
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 caching concepts còn confuse
- [ ] Write: 1 performance optimization bạn made và impact
- [ ] Write: Confidence level cho Week 6 (1-10)
- [ ] Compare: Week 4 vs Week 5 progress
- [ ] Plan: Preparation cho Week 6 (Load Balancing & API Design)
- [ ] Set: Goals cho Week 6
- [ ] Document: Week 5 reflection (500 words)

### Mentor Questions (Answer these - with numbers)
- [ ] Q1: Cache hit rate = 60%, bạn muốn improve lên 85%. Làm sao? Measure improvement. (viết strategy với expected numbers)
- [ ] Q2: Hot key problem - single key gets 10K requests/second. Redis CPU = 90%. Làm sao fix? Measure improvement. (viết solution với performance analysis)
- [ ] Q3: Cache breakdown - 1M keys expire cùng lúc, database overloaded. Làm sao prevent? (viết prevention strategy)
- [ ] Q4: Distributed lock - 100 threads compete for same lock. Throughput drops 50%. Làm sao optimize? (viết optimization strategy với expected improvement)
- [ ] Q5: Cache memory = 10GB, hit rate = 70%. Bạn có 5GB more memory. Expected hit rate improvement? Justify với calculation. (viết analysis với numbers)
- [ ] Q6: Write-Through pattern - write latency = 50ms (cache) + 100ms (DB) = 150ms. Write-Behind có thể improve không? Expected improvement? (viết analysis với latency calculations)
- [ ] Review: Answers có đủ numbers, measurements, và proof không?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Redis Integration TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Performance Testing TODOs: ✅ Complete với benchmark results
- [ ] Tất cả Cache Failure TODOs: ✅ Complete với mitigation strategies
- [ ] Tất cả Concurrency & Lock TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Performance benchmarks saved: ✅ Yes
- [ ] Cache hit rate metrics documented: ✅ Yes
- [ ] Ready for Week 6: ✅ Yes

---

> **Mentor Final Check**: Performance engineering là về numbers, không phải feelings. Nếu bạn không thể measure improvement, bạn không improve được gì. Hãy honest: Bạn có cache hit rate > 80% chưa? Bạn có latency improvement > 50% chưa? Bạn có fix được cache breakdown chưa? Nếu chưa, làm lại. Production systems cần proof, không cần assumptions.
