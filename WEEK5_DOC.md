# Week 5 – Caching & Performance Optimization

> **Mentor Note**: Caching là một trong những optimizations quan trọng nhất. Nhưng cache sai cách sẽ làm system chậm hơn, không nhanh hơn. Bạn phải MEASURE mọi thứ. Cache hit rate, latency improvement, memory usage - tất cả phải có numbers. Không có "có vẻ nhanh hơn". Phải có benchmarks, metrics, và proof. Production systems cần performance, không cần assumptions.

---

## Study TODOs

### Caching Fundamentals
- [x] Đọc "Designing Data-Intensive Applications" Chapter 3 - Storage and Retrieval (caching sections)
    - **Key takeaways (DDIA Ch3 – Caching):**
        - **Caching layers**: CPU cache (L1/L2/L3) → Memory → Disk → Network; mỗi layer chậm hơn 10-100x nhưng rẻ hơn.
        - **Cache hit vs miss**: Hit = data có trong cache (nhanh), Miss = phải fetch từ storage chậm hơn.
        - **Cache replacement policies**: LRU (Least Recently Used), LFU (Least Frequently Used), FIFO, Random.
        - **Write policies**: Write-through (write cache + storage), Write-back (write cache, flush sau), Write-around (skip cache).
        - **Cache invalidation**: TTL, event-driven, explicit delete, version-based.
        - **Insight**: Cache chỉ hiệu quả khi **hit rate cao** (>80%) và **access pattern có locality** (temporal/spatial).
- [x] Định nghĩa chính xác: Cache hit, cache miss, cache hit rate
    - **Cache hit**: Request tìm data trong cache và **tìm thấy** → trả về từ cache (nhanh, latency thấp).
    - **Cache miss**: Request tìm data trong cache nhưng **không tìm thấy** → phải fetch từ storage chậm hơn (database, disk, API).
    - **Cache hit rate**: Tỷ lệ phần trăm requests được phục vụ từ cache.
        - **Công thức**: `Hit Rate = (Cache Hits / Total Requests) × 100%`
        - **Target**: Production systems thường target >80% hit rate để có hiệu quả rõ rệt.
        - **Ví dụ**: 1000 requests, 850 hits → hit rate = 85%.
- [x] Viết công thức: Cache hit rate = ?
    - **Công thức cơ bản:**
        ```
        Cache Hit Rate = (Cache Hits / Total Requests) × 100%
        ```
    - **Công thức chi tiết:**
        ```
        Hit Rate = Hits / (Hits + Misses) × 100%
        ```
    - **Cache miss rate** (bổ sung):
        ```
        Miss Rate = (Cache Misses / Total Requests) × 100% = 100% - Hit Rate
        ```
    - **Average latency calculation** (quan trọng cho performance):
        ```
        Avg Latency = (Hit Rate × Cache Latency) + (Miss Rate × Storage Latency)
        ```
        - Ví dụ: Hit rate = 80%, cache latency = 1ms, DB latency = 100ms
        - Avg Latency = (0.8 × 1) + (0.2 × 100) = 0.8 + 20 = **20.8ms**
        - So với không cache: 100ms → improvement = **79.2%**
- [x] Đọc về "cache hierarchy" - L1, L2, L3 caches
    - **CPU Cache Hierarchy** (từ nhanh → chậm, nhỏ → lớn):
        - **L1 Cache**: Nhanh nhất (~1ns), nhỏ nhất (32-64KB), nằm trong CPU core, chia thành L1i (instruction) và L1d (data).
        - **L2 Cache**: Nhanh (~3ns), trung bình (256KB-1MB), có thể shared giữa cores hoặc per-core.
        - **L3 Cache**: Chậm hơn (~10ns), lớn hơn (8-32MB), shared giữa tất cả cores trong CPU.
        - **RAM**: Chậm hơn nhiều (~100ns), lớn (GB), ngoài CPU.
    - **Application Cache Hierarchy** (tương tự):
        - **L1**: In-memory cache trong application (Caffeine, Guava Cache) - ~1-10μs
        - **L2**: Distributed cache (Redis, Memcached) - ~1-5ms
        - **L3**: Database với query cache - ~10-100ms
        - **L4**: Disk storage - ~1-10ms
    - **Principle**: Data được access nhiều nhất nên ở cache level cao nhất (L1), ít access hơn ở level thấp hơn.
- [x] Đọc về "locality of reference" - spatial và temporal
    - **Temporal Locality**: Data được access gần đây có khả năng được access lại sớm.
        - **Ví dụ**: User profile được load → có thể được access lại trong vài phút.
        - **Cache strategy**: LRU (Least Recently Used) tận dụng temporal locality.
    - **Spatial Locality**: Data gần nhau về mặt địa chỉ có khả năng được access cùng lúc.
        - **Ví dụ**: Đọc 1 page từ disk → load cả block 4KB (có thể chứa data liên quan).
        - **Cache strategy**: Prefetching, block-based caching.
    - **Tại sao quan trọng?**
        - Cache chỉ hiệu quả khi access pattern có **locality cao**.
        - Random access pattern → cache hit rate thấp → cache không giúp gì, thậm chí làm chậm hơn (overhead).
    - **Real-world examples:**
        - **High temporal locality**: User session, hot products, trending content.
        - **High spatial locality**: Sequential file reads, array traversal.
        - **Low locality**: Random database queries, full table scans → không nên cache.
- [x] Đọc về "cache coherence" - consistency across caches
    - **Cache Coherence**: Đảm bảo tất cả caches có **consistent view** của data.
    - **Vấn đề**: Khi data được update ở một nơi, các cache khác có thể vẫn giữ **stale data**.
    - **Solutions:**
        - **Write-through**: Write vào cache + storage → đảm bảo consistency nhưng chậm.
        - **Write-invalidate**: Khi write, invalidate tất cả copies trong cache → force reload.
        - **Write-update**: Broadcast update tới tất cả caches → giữ consistency nhưng tốn bandwidth.
        - **TTL-based**: Accept eventual consistency, data expire sau một thời gian.
    - **Distributed cache coherence** (Redis, Memcached):
        - **Challenge**: Multiple servers, network latency, partition tolerance.
        - **Common approach**: Event-driven invalidation (pub/sub), hoặc accept eventual consistency với TTL.
    - **Trade-off**: Consistency vs Performance
        - **Strong consistency**: Chậm hơn, phức tạp hơn (distributed locks, quorum).
        - **Eventual consistency**: Nhanh hơn, đơn giản hơn, nhưng có thể đọc stale data.
- [x] Research: When caching helps? When it doesn't?
    - **Caching HELPS khi:**
        1. **Read-heavy workload**: Tỷ lệ read >> write (80/20 hoặc 90/10).
        2. **High hit rate potential**: Access pattern có locality, data được access lặp lại.
        3. **Expensive operations**: Database queries chậm, API calls tốn thời gian, complex calculations.
        4. **Data changes infrequently**: User profiles, product catalogs, configuration.
        5. **Latency critical**: Real-time systems, user-facing applications cần response <100ms.
    - **Caching DOESN'T HELP (thậm chí làm chậm) khi:**
        1. **Write-heavy workload**: Tỷ lệ write cao → cache invalidation overhead lớn.
        2. **Low hit rate**: Access pattern random, không có locality → cache miss nhiều, tốn memory vô ích.
        3. **Data changes frequently**: Real-time data, stock prices, live scores → cache nhanh chóng stale.
        4. **Small dataset**: Dataset nhỏ, fit trong memory → không cần cache, đọc trực tiếp cũng nhanh.
        5. **Simple/fast operations**: Operations đã nhanh (<1ms) → cache overhead có thể lớn hơn benefit.
    - **Rule of thumb**: Cache khi **hit rate > 50%** và **latency improvement > 30%**, ngược lại không nên cache.
- [x] Đọc về "cache size" calculation - how much to cache?
    - **Công thức cơ bản:**
        ```
        Cache Size = (Working Set Size) × (Safety Factor)
        ```
        - **Working Set**: Tập data được access thường xuyên trong một khoảng thời gian.
        - **Safety Factor**: 1.2-1.5 để tránh eviction quá nhiều.
    - **Cách tính Working Set:**
        1. **Analyze access patterns**: Log requests, identify hot data.
        2. **80/20 rule**: 20% data thường chiếm 80% requests.
        3. **Time window**: Data được access trong 24h/7 days gần đây.
    - **Ví dụ thực tế:**
        - **E-commerce**: 1M products, nhưng 100K hot products → cache 100K × 10KB = **1GB**.
        - **User profiles**: 10M users, nhưng 1M active users → cache 1M × 5KB = **5GB**.
    - **Memory constraints:**
        - **Redis maxmemory**: Set theo available RAM, thường 50-70% total RAM.
        - **Eviction policy**: LRU, LFU, TTL-based để tự động evict khi full.
    - **Cost-benefit analysis:**
        - **Cost**: Memory cost, maintenance overhead.
        - **Benefit**: Latency reduction, database load reduction.
        - **ROI**: Nếu cache giảm 50% DB load → tiết kiệm DB costs > memory costs → worth it.

### Cache Patterns Deep Dive
- [x] Đọc về "Cache-Aside" pattern (Lazy Loading)
    - **Cache-Aside (Lazy Loading)**: Application tự quản lý cache, không có sự tham gia của cache trong write flow.
    - **Flow**:
        1. **Read**: Check cache → nếu có (hit) → return; nếu không (miss) → query DB → store vào cache → return.
        2. **Write**: Update DB → invalidate/delete cache key.
    - **Đặc điểm**: Cache là "aside" (bên cạnh), không nằm giữa application và storage.
- [x] Viết algorithm pseudocode cho Cache-Aside
    ```pseudocode
    function getData(key):
        // Read path
        value = cache.get(key)
        if value != null:
            return value  // Cache hit
        
        // Cache miss - load from DB
        value = database.get(key)
        if value != null:
            cache.set(key, value, ttl)  // Store in cache for next time
        return value
    
    function updateData(key, newValue):
        // Write path
        database.update(key, newValue)
        cache.delete(key)  // Invalidate cache
        // Note: Could also do cache.set(key, newValue) to update cache
    
    function deleteData(key):
        database.delete(key)
        cache.delete(key)  // Invalidate cache
    ```
- [x] Analyze: Pros và cons của Cache-Aside (5 points each)
    - **Pros:**
        1. **Đơn giản**: Dễ implement, không cần cache support write operations.
        2. **Flexible**: Application kiểm soát hoàn toàn cache logic, có thể customize.
        3. **Resilient**: Nếu cache down, application vẫn hoạt động (fallback to DB).
        4. **Cache miss handling**: Có thể implement logic phức tạp khi miss (retry, fallback).
        5. **No cache pollution**: Chỉ cache data thực sự được access (lazy loading).
    - **Cons:**
        1. **Cache miss penalty**: Mỗi miss = 2 round trips (cache + DB), latency cao.
        2. **Race condition risk**: Nếu 2 requests cùng miss → cả 2 query DB, có thể cache stampede.
        3. **Stale data risk**: Nếu update DB nhưng cache delete fail → stale data.
        4. **Application complexity**: Application phải quản lý cache logic ở nhiều nơi.
        5. **No write optimization**: Write vẫn phải hit DB, không tận dụng cache cho write.
- [x] Đọc về "Write-Through" pattern
    - **Write-Through**: Mọi write đều được ghi vào **cả cache và storage** đồng thời.
    - **Flow**:
        1. **Write**: Application write → cache write → storage write (cả 2 phải succeed).
        2. **Read**: Check cache → nếu có return, nếu không load từ storage.
    - **Đặc điểm**: Cache luôn có data mới nhất (strong consistency), nhưng write chậm hơn (phải chờ cả 2).
- [x] Viết algorithm pseudocode cho Write-Through
    ```pseudocode
    function getData(key):
        value = cache.get(key)
        if value != null:
            return value  // Cache hit
        
        // Cache miss - load from storage
        value = storage.get(key)
        if value != null:
            cache.set(key, value)  // Populate cache
        return value
    
    function writeData(key, value):
        // Write to both cache and storage
        try:
            cache.set(key, value)      // Write to cache first
            storage.write(key, value)   // Then write to storage
            // Both must succeed
        catch error:
            // Rollback: delete from cache if storage fails
            cache.delete(key)
            throw error
    
    function updateData(key, newValue):
        // Same as writeData - update both
        cache.set(key, newValue)
        storage.update(key, newValue)
    ```
- [x] Analyze: Pros và cons của Write-Through (5 points each)
    - **Pros:**
        1. **Strong consistency**: Cache luôn sync với storage, không có stale data.
        2. **Read optimization**: Write đã populate cache → read sau đó sẽ hit cache.
        3. **Data safety**: Data được ghi vào storage ngay → không mất data nếu cache crash.
        4. **Simple read logic**: Read chỉ cần check cache, không cần handle miss phức tạp.
        5. **No invalidation needed**: Không cần invalidate vì cache luôn up-to-date.
    - **Cons:**
        1. **Write latency cao**: Phải chờ cả cache + storage → latency = max(cache_latency, storage_latency).
        2. **Write failure handling**: Nếu 1 trong 2 fail → phải rollback, phức tạp.
        3. **Cache pollution**: Cache cả data ít được đọc (write-only data).
        4. **No write optimization**: Không tận dụng cache để buffer writes.
        5. **Storage bottleneck**: Mọi write đều hit storage → không giảm load cho storage.
- [x] Đọc về "Write-Behind" (Write-Back) pattern
    - **Write-Behind (Write-Back)**: Write vào cache ngay (fast), sau đó **async write** vào storage.
    - **Flow**:
        1. **Write**: Application write → cache write (return ngay) → async queue → storage write (background).
        2. **Read**: Check cache → nếu có return, nếu không load từ storage.
    - **Đặc điểm**: Write nhanh nhất (chỉ chờ cache), nhưng có risk mất data nếu cache crash trước khi flush.
- [x] Viết algorithm pseudocode cho Write-Behind
    ```pseudocode
    queue = AsyncWriteQueue()
    
    function getData(key):
        value = cache.get(key)
        if value != null:
            return value  // Cache hit
        
        // Cache miss - load from storage
        value = storage.get(key)
        if value != null:
            cache.set(key, value)
        return value
    
    function writeData(key, value):
        // Write to cache immediately (fast)
        cache.set(key, value)
        
        // Queue async write to storage
        queue.enqueue({key: key, value: value, timestamp: now()})
        return  // Return immediately, don't wait for storage
    
    // Background worker
    function backgroundWriter():
        while true:
            writeTask = queue.dequeue()
            try:
                storage.write(writeTask.key, writeTask.value)
                // Mark as completed
            catch error:
                // Retry logic
                if writeTask.retryCount < MAX_RETRIES:
                    queue.enqueue(writeTask, retryCount++)
                else:
                    // Alert: data loss risk
                    alert("Failed to write to storage after retries")
    
    function flushCache():
        // Flush all pending writes before shutdown
        while queue.notEmpty():
            writeTask = queue.dequeue()
            storage.write(writeTask.key, writeTask.value)
    ```
- [x] Analyze: Pros và cons của Write-Behind (5 points each)
    - **Pros:**
        1. **Write latency thấp nhất**: Chỉ chờ cache write (~1ms) vs storage write (~100ms).
        2. **Write throughput cao**: Có thể batch nhiều writes → giảm I/O cho storage.
        3. **Storage load reduction**: Writes được buffer → giảm số lượng writes tới storage.
        4. **Better for write-heavy**: Phù hợp với workload write nhiều.
        5. **Read optimization**: Write đã populate cache → read sau đó sẽ hit.
    - **Cons:**
        1. **Data loss risk**: Nếu cache crash trước khi flush → mất data chưa được ghi vào storage.
        2. **Complexity cao**: Cần async queue, retry logic, crash recovery.
        3. **Consistency issues**: Storage có thể lag behind cache → eventual consistency.
        4. **Crash recovery**: Cần mechanism để recover pending writes sau khi restart.
        5. **Monitoring required**: Phải monitor queue size, flush rate để tránh backlog.
- [x] Compare: Cache-Aside vs Write-Through vs Write-Behind (10 criteria)
    | Criteria | Cache-Aside | Write-Through | Write-Behind |
    |----------|-------------|---------------|--------------|
    | **Write Latency** | Medium (DB only) | High (Cache + DB) | Low (Cache only) |
    | **Read Latency** | Medium (cache miss = 2 trips) | Low (cache always populated) | Low (cache always populated) |
    | **Consistency** | Eventual (stale possible) | Strong (always sync) | Eventual (async write) |
    | **Complexity** | Low | Medium | High |
    | **Data Safety** | High (DB is source of truth) | High (both written) | Medium (risk if cache crash) |
    | **Cache Hit Rate** | Depends on access pattern | High (writes populate cache) | High (writes populate cache) |
    | **Storage Load** | High (every read miss hits DB) | High (every write hits DB) | Low (writes batched) |
    | **Failure Handling** | Simple (cache down = fallback) | Complex (both must succeed) | Complex (async queue recovery) |
    | **Use Case** | Read-heavy, infrequent writes | Strong consistency required | Write-heavy, can accept eventual consistency |
    | **Implementation** | Easy | Medium | Hard (needs queue, retry) |
- [x] Research: Real-world examples của mỗi pattern
    - **Cache-Aside Examples:**
        1. **Facebook/Meta**: User profiles, posts - cache-aside với Memcached.
        2. **Amazon**: Product catalog - cache-aside với Redis.
        3. **Twitter**: Timeline cache - cache-aside pattern.
        4. **Netflix**: Movie metadata - cache-aside với EVCache.
    - **Write-Through Examples:**
        1. **Financial systems**: Transaction logs - cần strong consistency.
        2. **Configuration management**: System configs - cần sync ngay.
        3. **User sessions**: Session data - cần persist ngay để tránh mất.
    - **Write-Behind Examples:**
        1. **Analytics systems**: Event logging - có thể mất một số events.
        2. **Metrics collection**: System metrics - write nhiều, có thể batch.
        3. **Audit logs**: Activity logs - write-heavy, eventual consistency OK.
        4. **Gaming leaderboards**: Score updates - write nhiều, read sau.
- [x] Analyze: When to use which pattern? (5 use cases each)
    - **Cache-Aside - Use khi:**
        1. **Read-heavy workload**: 80%+ reads, ít writes.
        2. **Data changes infrequently**: User profiles, product info, static content.
        3. **Simple requirements**: Không cần strong consistency, dễ implement.
        4. **Cache as optimization layer**: Cache là optional, system vẫn chạy được không có cache.
        5. **Different cache strategies per data type**: Một số data cache-aside, một số không cache.
    - **Write-Through - Use khi:**
        1. **Strong consistency required**: Financial data, critical transactions.
        2. **Read-after-write common**: User vừa update → ngay lập tức đọc lại.
        3. **Data safety critical**: Không thể mất data, mọi write phải persist ngay.
        4. **Simple cache logic**: Không muốn quản lý invalidation phức tạp.
        5. **Write rate low**: Writes không quá nhiều → latency overhead acceptable.
    - **Write-Behind - Use khi:**
        1. **Write-heavy workload**: 50%+ writes, cần optimize write performance.
        2. **Write latency critical**: Real-time systems, gaming, high-frequency trading.
        3. **Can accept eventual consistency**: Analytics, logs, metrics.
        4. **Batch writes beneficial**: Có thể batch nhiều writes → giảm I/O.
        5. **Temporary data loss acceptable**: Có thể mất một số writes nếu cache crash (non-critical data).

### Cache Invalidation Strategies
- [x] Đọc về "TTL" (Time-To-Live) invalidation
    - **TTL (Time-To-Live)**: Cache key tự động expire sau một khoảng thời gian cố định.
    - **How it works**: Khi set cache, specify TTL (vd: 1 hour, 24 hours). Sau TTL, key tự động bị xóa.
    - **Implementation**: Redis `SET key value EX 3600` (expire sau 3600 giây).
    - **Use cases**: Data có thể stale một chút, không cần real-time (user profiles, product info, static content).
- [x] Analyze: Pros và cons của TTL (5 points each)
    - **Pros:**
        1. **Đơn giản**: Không cần logic phức tạp, chỉ set TTL khi cache.
        2. **Automatic**: Tự động expire, không cần manual management.
        3. **Memory management**: Tự động giải phóng memory khi expire.
        4. **No coordination needed**: Không cần event system, pub/sub.
        5. **Predictable**: Biết chắc data sẽ fresh sau TTL.
    - **Cons:**
        1. **Stale data risk**: Data có thể stale trong TTL period (có thể vài giờ).
        2. **Cache breakdown risk**: Nếu nhiều keys expire cùng lúc → cache miss storm.
        3. **No immediate update**: Update ở DB không invalidate cache ngay → phải đợi TTL.
        4. **Waste memory**: Data có thể không cần nữa nhưng vẫn tồn tại đến khi expire.
        5. **TTL tuning**: Phải chọn TTL phù hợp - quá ngắn → miss nhiều, quá dài → stale data.
- [x] Đọc về "event-driven invalidation"
    - **Event-driven invalidation**: Khi data được update ở source (DB), publish event → cache subscribe và invalidate.
    - **How it works**: 
        1. Application update DB → publish "data updated" event.
        2. Cache service subscribe event → delete cache key.
    - **Implementation**: Redis Pub/Sub, Message Queue (Kafka, RabbitMQ), Event Bus.
    - **Use cases**: Data cần real-time, update không thường xuyên nhưng cần invalidate ngay.
- [x] Analyze: Pros và cons của event-driven (5 points each)
    - **Pros:**
        1. **Real-time invalidation**: Cache được invalidate ngay khi data update → không stale.
        2. **Efficient**: Chỉ invalidate khi có update, không waste memory.
        3. **Scalable**: Event system có thể scale, nhiều cache instances subscribe.
        4. **Decoupled**: Cache và application không tightly coupled.
        5. **Flexible**: Có thể invalidate nhiều keys, hoặc pattern-based invalidation.
    - **Cons:**
        1. **Complexity**: Cần event infrastructure (pub/sub, message queue).
        2. **Event delivery guarantee**: Phải đảm bảo event được deliver (at-least-once, exactly-once).
        3. **Latency**: Event có thể có delay → cache vẫn stale trong thời gian ngắn.
        4. **Failure handling**: Nếu event lost → cache không được invalidate → stale data.
        5. **Cost**: Event infrastructure có overhead (network, processing).
- [x] Đọc về "explicit invalidation" (manual delete)
    - **Explicit invalidation**: Application manually delete cache key khi cần.
    - **How it works**: Sau khi update DB, application gọi `cache.delete(key)`.
    - **Implementation**: Redis `DEL key`, hoặc `cache.evict(key)` trong Spring Cache.
    - **Use cases**: Update không thường xuyên, cần control chính xác khi nào invalidate.
- [x] Đọc về "version-based invalidation"
    - **Version-based invalidation**: Mỗi data có version number, cache key include version.
    - **How it works**:
        1. Data có version field (vd: `user:123:v5`).
        2. Khi update → version tăng → old cache key (`user:123:v4`) không match → tự động stale.
        3. New reads sẽ tạo cache key mới với version mới.
    - **Implementation**: Cache key format `{entity}:{id}:v{version}`, version stored in DB hoặc metadata store.
    - **Use cases**: Data có versioning, cần support multiple versions, A/B testing.
- [x] Đọc về "cache stampede" problem
    - **Cache Stampede (Cache Miss Storm)**: Khi cache key expire, nhiều requests cùng lúc miss → tất cả query DB → DB overload.
    - **Scenario**: 
        - Popular key expire (vd: trending products).
        - 1000 requests cùng lúc → tất cả miss cache.
        - 1000 requests cùng query DB → DB overload, latency spike.
    - **Impact**: Database overload, application latency spike, có thể cause cascading failure.
- [x] Research: Strategies to prevent cache stampede
    1. **Staggered TTL (TTL jitter)**:
        - Thay vì TTL cố định, thêm random jitter: `TTL = base_ttl + random(0, jitter)`.
        - Ví dụ: Base TTL = 1 hour, jitter = 10 minutes → TTL = 50-70 minutes.
        - **Effect**: Keys expire ở thời điểm khác nhau → tránh đồng loạt expire.
    2. **Mutex Lock (Single Request Load)**:
        - Khi cache miss, chỉ 1 request được phép load từ DB, các request khác wait.
        - **Implementation**: Distributed lock (Redis SET NX) → chỉ 1 request acquire lock → load DB → populate cache → release lock.
    3. **Cache Warming**:
        - Pre-load hot data vào cache trước khi expire.
        - **Implementation**: Background job refresh cache trước khi TTL hết (vd: refresh khi còn 10% TTL).
    4. **Probabilistic Early Expiration**:
        - Random expire sớm một chút (vd: expire ở 90% TTL với probability 10%).
        - **Effect**: Spread expiration time, reduce stampede risk.
    5. **Background Refresh**:
        - Khi cache hit nhưng gần expire → trigger background refresh.
        - **Implementation**: Return stale data ngay, refresh async → không block request.
- [x] Đọc về "thundering herd" problem
    - **Thundering Herd**: Khi cache miss, nhiều requests cùng lúc query DB → database connection pool exhaustion, DB overload.
    - **Difference với Cache Stampede**: 
        - **Cache Stampede**: Nhiều keys expire cùng lúc.
        - **Thundering Herd**: Một key miss, nhiều requests cùng query DB.
    - **Scenario**: 
        - Popular endpoint, cache miss (hoặc cache down).
        - 1000 concurrent requests → tất cả query DB.
        - Database connection pool = 100 → 900 requests block/wait → timeout.
- [x] Research: Solutions cho thundering herd
    1. **Request Coalescing**:
        - Khi nhiều requests cùng query một key → chỉ 1 request query DB, các request khác wait và share result.
        - **Implementation**: In-memory lock per key, first request load, others wait.
    2. **Connection Pooling + Queue**:
        - Limit concurrent DB queries per key.
        - **Implementation**: Semaphore per key, chỉ cho phép N concurrent queries.
    3. **Circuit Breaker**:
        - Nếu DB overload → circuit open → return cached data (stale OK) hoặc error.
        - **Implementation**: Monitor DB latency, nếu > threshold → open circuit.
    4. **Rate Limiting**:
        - Limit số requests có thể query DB per second.
        - **Implementation**: Token bucket, sliding window rate limiter.
    5. **Graceful Degradation**:
        - Return stale data hoặc default value thay vì query DB khi overload.
        - **Implementation**: Check cache age, nếu < max_stale_time → return stale data.
- [x] Compare: Invalidation strategies (5 criteria)
    | Criteria | TTL | Event-driven | Explicit | Version-based |
    |----------|-----|--------------|----------|---------------|
    | **Consistency** | Eventual (stale trong TTL) | Strong (invalidate ngay) | Strong (manual control) | Strong (version check) |
    | **Complexity** | Low | High (need event infra) | Medium | Medium-High |
    | **Latency** | Low (no coordination) | Medium (event delay) | Low (direct delete) | Low (version check) |
    | **Memory Efficiency** | Medium (waste until expire) | High (invalidate when needed) | High (invalidate when needed) | Medium (old versions exist) |
    | **Use Case** | Static/semi-static data | Real-time updates | Manual control needed | Versioned data |

### Redis Data Structures
- [x] Đọc về Redis String - use cases
    - **Redis String**: Basic key-value store, value có thể là string, number, hoặc binary data (max 512MB).
    - **Commands**: `SET`, `GET`, `INCR`, `DECR`, `APPEND`, `STRLEN`.
    - **Use cases**:
        1. **Simple caching**: User profiles, product info, configuration.
        2. **Counters**: Page views, likes, shares (dùng `INCR`).
        3. **Sessions**: User session data, authentication tokens.
        4. **Flags**: Feature flags, boolean values.
        5. **Serialized objects**: JSON, binary data.
- [x] Đọc về Redis Hash - use cases
    - **Redis Hash**: Key-value pairs trong một key, như object/dictionary.
    - **Commands**: `HSET`, `HGET`, `HGETALL`, `HINCRBY`, `HDEL`.
    - **Use cases**:
        1. **User profiles**: `user:123` → `{name, email, age, ...}`.
        2. **Shopping cart**: `cart:user:123` → `{product_id: quantity, ...}`.
        3. **Object caching**: Cache entire objects, update individual fields.
        4. **Configuration**: System configs với nhiều fields.
        5. **Memory efficient**: Hash nhỏ hơn nhiều String keys riêng lẻ.
- [x] Đọc về Redis List - use cases
    - **Redis List**: Ordered collection, có thể push/pop từ đầu hoặc cuối.
    - **Commands**: `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `LRANGE`, `LLEN`.
    - **Use cases**:
        1. **Message queues**: Simple queue, producer push, consumer pop.
        2. **Activity feeds**: Recent activities, timeline.
        3. **Task queues**: Background job queues.
        4. **Recent items**: Last N items viewed, last N searches.
        5. **Leaderboard (simple)**: List of top N (nhưng Sorted Set tốt hơn).
- [x] Đọc về Redis Set - use cases
    - **Redis Set**: Unordered collection of unique strings.
    - **Commands**: `SADD`, `SREM`, `SMEMBERS`, `SISMEMBER`, `SINTER`, `SUNION`.
    - **Use cases**:
        1. **Tags**: Product tags, user interests.
        2. **Unique tracking**: Unique visitors, unique IPs.
        3. **Relationships**: Friends, followers, memberships.
        4. **Set operations**: Intersection (common friends), union, difference.
        5. **Blacklist/Whitelist**: Blocked users, allowed IPs.
- [x] Đọc về Redis Sorted Set - use cases
    - **Redis Sorted Set**: Set với score, tự động sort theo score.
    - **Commands**: `ZADD`, `ZRANGE`, `ZREVRANGE`, `ZRANK`, `ZSCORE`, `ZINCRBY`.
    - **Use cases**:
        1. **Leaderboards**: Game scores, rankings (top N).
        2. **Time-series data**: Events với timestamp as score.
        3. **Priority queues**: Tasks với priority.
        4. **Rate limiting**: Sliding window với timestamp.
        5. **Top N queries**: Top products, top users by metric.
- [x] Đọc về Redis Bitmap - use cases
    - **Redis Bitmap**: String được treat như array of bits, operations trên bits.
    - **Commands**: `SETBIT`, `GETBIT`, `BITCOUNT`, `BITOP`, `BITPOS`.
    - **Use cases**:
        1. **User activity tracking**: Daily active users, weekly active users.
        2. **Feature flags per user**: Enable/disable features.
        3. **Analytics**: Track events, conversions.
        4. **Bloom filters**: Implement bloom filter để check existence.
        5. **Memory efficient**: 1 bit per user → 1M users = 125KB.
- [x] Đọc về Redis HyperLogLog - use cases
    - **Redis HyperLogLog**: Probabilistic data structure để count unique elements.
    - **Commands**: `PFADD`, `PFCOUNT`, `PFMERGE`.
    - **Use cases**:
        1. **Unique visitors**: Count unique IPs, unique users (approximate).
        2. **Cardinality estimation**: Count distinct values (vd: distinct products viewed).
        3. **Analytics**: Daily unique users, unique events.
        4. **Memory efficient**: 12KB cho 2^64 unique elements (với error ~0.81%).
        5. **Merge operations**: Combine multiple HyperLogLogs.
- [x] Đọc về Redis Streams - use cases
    - **Redis Streams**: Log-like data structure, append-only, support consumer groups.
    - **Commands**: `XADD`, `XREAD`, `XREADGROUP`, `XRANGE`, `XACK`.
    - **Use cases**:
        1. **Message queues**: Reliable message queue với consumer groups.
        2. **Event sourcing**: Store events, replay history.
        3. **Activity logs**: User activity logs, audit trails.
        4. **Real-time analytics**: Stream processing, event streams.
        5. **Pub/Sub alternative**: More reliable than Pub/Sub (persistent messages).
- [x] Compare: Data structures (when to use which?)
    | Structure | When to Use | Example |
    |-----------|-------------|---------|
    | **String** | Simple key-value, counters, sessions | `user:123`, `page:views:100` |
    | **Hash** | Object với nhiều fields, partial updates | `user:123` → `{name, email, age}` |
    | **List** | Ordered collection, queues, feeds | Activity feed, message queue |
    | **Set** | Unique items, tags, relationships | User tags, friends list |
    | **Sorted Set** | Ranked data, leaderboards, time-series | Game leaderboard, events by time |
    | **Bitmap** | Boolean flags, activity tracking | Daily active users, feature flags |
    | **HyperLogLog** | Approximate unique count | Unique visitors (approximate) |
    | **Streams** | Message queues, event logs | Activity logs, event sourcing |
- [x] Research: Memory efficiency của mỗi structure
    - **String**: ~40-50 bytes overhead per key + value size. **Inefficient** cho small values.
    - **Hash**: ~40 bytes overhead + (field_count × ~40 bytes) + values. **Efficient** cho objects với nhiều fields.
    - **List**: ~40 bytes overhead + (element_count × ~40 bytes) + values. **Efficient** cho sequential data.
    - **Set**: ~40 bytes overhead + (element_count × ~40 bytes) + values. **Efficient** cho unique collections.
    - **Sorted Set**: ~40 bytes overhead + (element_count × ~80 bytes) + values + scores. **Less efficient** nhưng cần cho ranking.
    - **Bitmap**: ~40 bytes overhead + (bit_count / 8) bytes. **Very efficient** cho boolean flags (1 bit per item).
    - **HyperLogLog**: ~12KB fixed size cho 2^64 elements. **Extremely efficient** cho cardinality estimation.
    - **Streams**: ~40 bytes overhead + message size. **Efficient** cho append-only logs.
- [x] Research: Performance characteristics của mỗi structure
    - **String**: O(1) GET/SET, O(1) INCR. **Fastest** cho simple operations.
    - **Hash**: O(1) HGET/HSET, O(N) HGETALL. **Fast** cho field access, slow cho full object.
    - **List**: O(1) LPUSH/RPOP, O(N) LRANGE. **Fast** cho queue operations.
    - **Set**: O(1) SADD/SISMEMBER, O(N) SMEMBERS. **Fast** cho membership check.
    - **Sorted Set**: O(log N) ZADD/ZRANK, O(log N + M) ZRANGE. **Fast** cho ranking, slower than Set.
    - **Bitmap**: O(1) SETBIT/GETBIT, O(N) BITCOUNT. **Very fast** cho bit operations.
    - **HyperLogLog**: O(1) PFADD, O(1) PFCOUNT. **Very fast** cho cardinality.
    - **Streams**: O(1) XADD, O(N) XREAD. **Fast** cho append, efficient cho streaming.

### Cache Problems & Solutions
- [x] Đọc về "cache breakdown" (cache miss storm)
    - **Cache Breakdown**: Khi cache key expire hoặc bị delete, nhiều requests cùng lúc miss → tất cả query DB → DB overload.
    - **Scenario**: Popular key expire → 1000 requests miss → 1000 DB queries → DB crash.
    - **Impact**: Database overload, application latency spike, cascading failure.
- [x] Analyze: What causes cache breakdown?
    1. **Synchronous expiration**: Nhiều keys expire cùng lúc (same TTL).
    2. **Cache restart**: Redis restart → tất cả keys mất → miss storm.
    3. **Cache eviction**: Memory full → evict nhiều keys → miss storm.
    4. **Manual invalidation**: Invalidate nhiều keys cùng lúc (vd: clear all cache).
    5. **Popular key expiration**: Key được access nhiều expire → nhiều requests cùng miss.
- [x] Research: Solutions cho cache breakdown
    1. **Staggered TTL**: `TTL = base_ttl + random(0, jitter)` → keys expire ở thời điểm khác nhau.
    2. **Mutex lock**: Khi miss, chỉ 1 request load DB, others wait → tránh duplicate queries.
    3. **Cache warming**: Pre-load hot data trước khi expire.
    4. **Never expire hot keys**: Hot keys không set TTL, chỉ invalidate khi update.
    5. **Graceful degradation**: Return stale data hoặc default value khi DB overload.
- [x] Đọc về "cache penetration" (query non-existent data)
    - **Cache Penetration**: Query data không tồn tại trong DB → cache miss → query DB → không có data → không cache → lần sau lại miss → loop.
    - **Scenario**: Attacker query random user IDs không tồn tại → mỗi request miss cache + query DB → DB overload.
    - **Impact**: Wasted DB queries, cache không giúp gì, DB load cao.
- [x] Analyze: What causes cache penetration?
    1. **Non-existent data queries**: Query IDs không tồn tại (random IDs, invalid input).
    2. **No caching of empty results**: Không cache "not found" results → mỗi lần query lại miss.
    3. **Attack**: Malicious users query random IDs để overload system.
    4. **Data deletion**: Data bị delete nhưng cache không biết → query lại miss.
    5. **Invalid input**: User input validation không đủ → query invalid data.
- [x] Research: Solutions cho cache penetration (bloom filter, etc.)
    1. **Cache empty results**: Cache "not found" với short TTL (vd: 5 minutes) → tránh query DB lặp lại.
    2. **Bloom Filter**: Check existence trước khi query DB → nếu không có trong bloom filter → skip DB query.
    3. **Input validation**: Validate input format trước khi query (vd: user ID phải là số).
    4. **Rate limiting**: Limit số requests per IP/user → tránh attack.
    5. **Whitelist/Blacklist**: Maintain list valid IDs → reject invalid IDs ngay.
- [x] Đọc về "hot key" problem
    - **Hot Key**: Một key được access quá nhiều → Redis instance handling key đó bị overload (CPU, network).
    - **Scenario**: Popular product page → single key `product:123` nhận 10K requests/second → Redis CPU 90% → latency spike.
    - **Impact**: Redis CPU bottleneck, network bottleneck, latency spike cho tất cả requests.
- [x] Analyze: What causes hot keys?
    1. **Popular content**: Trending products, viral posts, breaking news.
    2. **Single key design**: Tất cả requests dùng cùng một key (vd: `trending:products`).
    3. **Cache key không được shard**: Key không được distribute across instances.
    4. **Real-time data**: Live scores, stock prices → nhiều users query cùng lúc.
    5. **Application bug**: Infinite loop query same key.
- [x] Research: Solutions cho hot keys (local cache, sharding, etc.)
    1. **Local cache (L1 cache)**: Application-level cache (Caffeine, Guava) → giảm requests tới Redis.
    2. **Key sharding**: Split hot key thành nhiều keys (vd: `product:123:shard0`, `product:123:shard1`) → distribute load.
    3. **Read replica**: Route reads tới replica → giảm load master.
    4. **Pre-computation**: Pre-compute và cache result → tránh expensive computation mỗi request.
    5. **Rate limiting**: Limit requests per key → tránh overload.
- [x] Đọc về "big key" problem
    - **Big Key**: Key chứa quá nhiều data (vd: >10MB) → slow operations, memory issues, network transfer slow.
    - **Scenario**: Cache entire user list trong 1 key → key size = 50MB → GET operation mất 100ms → block Redis.
    - **Impact**: Slow operations, memory pressure, network bottleneck, Redis blocking.
- [x] Analyze: What causes big keys?
    1. **Large collections**: Cache entire list/set trong 1 key (vd: all products, all users).
    2. **No pagination**: Load all data thay vì paginate.
    3. **Serialization overhead**: Serialize large objects → big values.
    4. **Accumulation**: Key accumulate data over time (vd: append logs).
    5. **Wrong data structure**: Dùng String thay vì Hash/List → không thể partial access.
- [x] Research: Solutions cho big keys (compression, splitting, etc.)
    1. **Split key**: Split thành nhiều keys nhỏ hơn (vd: `products:page:1`, `products:page:2`).
    2. **Compression**: Compress value trước khi cache (gzip, snappy) → giảm size.
    3. **Use appropriate data structure**: Dùng Hash/List thay vì String → partial access.
    4. **Pagination**: Cache paginated data thay vì all data.
    5. **Lazy loading**: Load data on-demand, không cache all at once.

### Distributed Caching
- [x] Đọc về "Redis Cluster" - how it works
    - **Redis Cluster**: Distributed Redis với automatic sharding, high availability.
    - **How it works**:
        1. **Sharding**: Data được shard across multiple nodes (16384 hash slots).
        2. **Hash slots**: Key được hash → map tới 1 trong 16384 slots → slot assigned to node.
        3. **Gossip protocol**: Nodes communicate để maintain cluster state.
        4. **Failover**: Nếu master down → replica tự động promote thành master.
    - **Features**: Automatic failover, data sharding, no single point of failure.
- [x] Đọc về "Redis Sentinel" - high availability
    - **Redis Sentinel**: High availability solution cho Redis, monitor và auto-failover.
    - **How it works**:
        1. **Monitoring**: Sentinel monitors master và replicas.
        2. **Failure detection**: Nếu master không respond → Sentinel detect failure.
        3. **Failover**: Sentinel promote replica thành master.
        4. **Configuration update**: Sentinel notify clients về new master.
    - **Use cases**: High availability cho single Redis instance hoặc master-replica setup.
- [x] Đọc về "consistent hashing" trong Redis Cluster
    - **Consistent Hashing**: Hash algorithm để distribute keys across nodes, minimize rehashing khi nodes add/remove.
    - **How it works**:
        1. **Hash ring**: Nodes và keys được hash → map tới hash ring (0-2^32-1).
        2. **Key assignment**: Key được assign tới node đầu tiên clockwise từ hash position.
        3. **Node addition**: Khi add node → chỉ keys gần node mới được move → minimal rehashing.
        4. **Node removal**: Khi remove node → keys được reassign tới next node.
    - **Redis Cluster**: Dùng hash slots (16384 slots) thay vì consistent hashing → simpler, faster.
- [x] Đọc về "replication" trong Redis
    - **Redis Replication**: Master-replica setup, master handles writes, replicas sync data từ master.
    - **How it works**:
        1. **Master writes**: All writes go to master.
        2. **Replication stream**: Master streams write commands tới replicas.
        3. **Replica sync**: Replicas execute commands → stay in sync.
        4. **Read scaling**: Replicas handle reads → scale read capacity.
    - **Features**: Async replication, partial resync, read scaling, high availability.
- [x] Đọc về "persistence" - RDB vs AOF
    - **RDB (Redis Database Backup)**: Snapshot của database tại một thời điểm.
        - **How**: Fork process → save memory snapshot to disk.
        - **Format**: Binary file, compact.
        - **Frequency**: Configurable (vd: save every 5 minutes nếu có 100+ changes).
    - **AOF (Append-Only File)**: Log mọi write command.
        - **How**: Append write commands to file.
        - **Format**: Text file, human-readable.
        - **Frequency**: Every write (fsync có thể config: always, everysec, no).
- [x] Compare: RDB vs AOF (5 criteria)
    | Criteria | RDB | AOF |
    |----------|-----|-----|
    | **Data Safety** | Medium (có thể mất data giữa snapshots) | High (mọi write được log) |
    | **File Size** | Small (compressed snapshot) | Large (mọi command được log) |
    | **Recovery Speed** | Fast (load snapshot) | Slow (replay commands) |
    | **Performance Impact** | Low (fork process, background) | Medium (fsync overhead) |
    | **Use Case** | Backup, disaster recovery | High data safety requirement |
- [x] Đọc về "Redis Pub/Sub" - use cases
    - **Redis Pub/Sub**: Publish-subscribe messaging pattern.
    - **How it works**: Publishers send messages tới channels, subscribers receive messages từ channels.
    - **Commands**: `PUBLISH`, `SUBSCRIBE`, `UNSUBSCRIBE`, `PSUBSCRIBE` (pattern subscribe).
    - **Use cases**:
        1. **Real-time notifications**: Chat, notifications, alerts.
        2. **Event broadcasting**: Cache invalidation events, system events.
        3. **Microservices communication**: Service-to-service messaging.
        4. **Live updates**: Stock prices, live scores, real-time dashboards.
        5. **Log aggregation**: Collect logs từ multiple sources.
- [x] Research: Redis performance tuning
    1. **Memory optimization**:
        - Set `maxmemory` và `maxmemory-policy` (LRU, LFU).
        - Use appropriate data structures (Hash thay vì multiple Strings).
        - Enable compression cho large values.
    2. **Persistence tuning**:
        - RDB: Tune `save` frequency based on data change rate.
        - AOF: Use `everysec` fsync (balance safety và performance).
    3. **Network optimization**:
        - Enable TCP keepalive.
        - Tune `tcp-backlog` cho high connection count.
    4. **Connection pooling**:
        - Use connection pool (Lettuce, Jedis) → reuse connections.
        - Tune pool size based on workload.
    5. **Pipelining**:
        - Batch multiple commands → reduce round trips.
    6. **Lua scripts**:
        - Use Lua scripts cho complex operations → reduce round trips.
    7. **Slow log**:
        - Monitor slow commands → optimize queries.

### Distributed Locking
- [x] Đọc về "distributed locks" - why needed?
    - **Distributed Locks**: Đảm bảo chỉ 1 process có thể execute critical section tại một thời điểm trong distributed system.
    - **Why needed**:
        1. **Prevent race conditions**: Multiple processes update same resource → cần lock.
        2. **Resource access control**: Chỉ 1 process access resource (vd: file, database row).
        3. **Idempotency**: Prevent duplicate processing (vd: payment processing).
        4. **Leader election**: Chọn leader trong cluster.
        5. **Rate limiting**: Limit concurrent operations.
- [x] Đọc về "Redis SET NX" - basic locking
    - **SET NX (SET if Not eXists)**: Atomic operation để acquire lock.
    - **How it works**: `SET key value NX EX timeout` → set key nếu không tồn tại, với expiration.
    - **Example**: `SET lock:resource:123 "process-id" NX EX 10` → acquire lock với 10s expiration.
    - **Release**: `DEL lock:resource:123` → release lock.
    - **Limitation**: Không đảm bảo lock được release bởi owner (process có thể crash sau khi acquire).
- [x] Đọc về "deadlock" trong distributed locks
    - **Deadlock**: Process A hold lock X, wait lock Y; Process B hold lock Y, wait lock X → cả 2 block forever.
    - **In distributed locks**: Ít phổ biến hơn single-machine locks vì locks có expiration.
    - **Prevention**:
        1. **Lock expiration**: Locks tự động expire → tránh deadlock.
        2. **Lock ordering**: Always acquire locks theo cùng thứ tự.
        3. **Timeout**: Set timeout khi acquire lock → fail nếu không acquire được.
- [x] Đọc về "lock expiration" - why critical?
    - **Lock Expiration**: Lock tự động expire sau một thời gian → prevent deadlock nếu process crash.
    - **Why critical**:
        1. **Process crash**: Nếu process crash sau khi acquire lock → lock không được release → deadlock.
        2. **Network partition**: Process bị partition → không thể release lock → expiration giải quyết.
        3. **Long operations**: Nếu operation lâu hơn expiration → lock expire → another process có thể acquire → race condition.
    - **Solution**: Lock renewal (heartbeat) → extend expiration nếu operation chưa xong.
- [x] Đọc về "Redlock algorithm" - distributed lock algorithm
    - **Redlock**: Distributed lock algorithm cho Redis, đảm bảo lock được acquire trên majority of nodes.
    - **How it works**:
        1. **Acquire lock**: Try acquire lock trên N Redis instances (N thường = 5).
        2. **Quorum**: Lock được acquire nếu acquire được trên majority (N/2 + 1).
        3. **Lock validity**: Lock valid trong thời gian = `TTL - (current_time - acquire_time) - clock_drift`.
        4. **Release**: Release lock trên tất cả instances.
    - **Purpose**: Đảm bảo lock reliable ngay cả khi một số Redis instances fail.
- [x] Analyze: Pros và cons của Redlock
    - **Pros:**
        1. **High availability**: Lock reliable ngay cả khi một số nodes fail.
        2. **Fault tolerance**: Tolerate minority node failures.
        3. **Distributed consensus**: Dựa trên quorum → strong guarantee.
    - **Cons:**
        1. **Complexity**: Phức tạp hơn single Redis lock.
        2. **Performance**: Phải communicate với nhiều nodes → slower.
        3. **Clock dependency**: Phụ thuộc vào clock synchronization → có thể có issues.
        4. **Controversy**: Một số experts (vd: Martin Kleppmann) argue Redlock không đảm bảo safety trong một số edge cases.
- [x] Đọc về "lease-based locking"
    - **Lease-based Locking**: Lock có "lease" (thời gian sở hữu), có thể renew lease để extend.
    - **How it works**:
        1. **Acquire lock**: Acquire lock với lease time (vd: 10 seconds).
        2. **Heartbeat**: Process gửi heartbeat để renew lease trước khi expire.
        3. **Auto-expire**: Nếu không có heartbeat → lock expire → another process có thể acquire.
    - **Benefits**: 
        - Cho phép long operations (renew lease).
        - Auto-release nếu process crash (no heartbeat).
    - **Implementation**: Redis với Lua script để atomic renew.
- [x] Research: Distributed lock implementations
    1. **Redis SET NX**: Simple, fast, nhưng không đảm bảo strong consistency.
    2. **Redlock**: Strong consistency, nhưng phức tạp và có controversy.
    3. **Zookeeper**: Strong consistency, nhưng slower và phức tạp hơn.
    4. **etcd**: Distributed lock với strong consistency, dùng cho Kubernetes.
    5. **Database locks**: Use database (PostgreSQL advisory locks) → simple nhưng có thể bottleneck.
- [x] Đọc về "lock contention" - performance impact
    - **Lock Contention**: Nhiều processes compete cho cùng một lock → chỉ 1 process acquire được, others wait.
    - **Performance Impact**:
        1. **Latency increase**: Processes wait → latency tăng.
        2. **Throughput decrease**: Chỉ 1 process execute tại một thời điểm → throughput giảm.
        3. **Resource waste**: Processes block → waste CPU, memory.
        4. **Cascading delays**: Lock contention → delays → more contention.
    - **Solutions**:
        1. **Lock sharding**: Split lock thành nhiều locks → reduce contention.
        2. **Optimistic locking**: Dùng version numbers thay vì locks.
        3. **Reduce lock scope**: Lock nhỏ nhất có thể, release sớm nhất có thể.
        4. **Lock-free algorithms**: Dùng lock-free data structures nếu có thể.

### Rate Limiting
- [x] Đọc về "rate limiting" - why needed?
    - **Rate Limiting**: Giới hạn số requests một client có thể make trong một khoảng thời gian.
    - **Why needed**:
        1. **Prevent abuse**: Tránh DDoS attacks, API abuse.
        2. **Fair resource sharing**: Đảm bảo tất cả users có fair access.
        3. **Cost control**: Limit usage để control costs (vd: API costs).
        4. **System stability**: Prevent overload → maintain system stability.
        5. **Business logic**: Enforce business rules (vd: free tier limits).
- [x] Đọc về "Token Bucket" algorithm
    - **Token Bucket**: Bucket chứa tokens, mỗi request consume 1 token, tokens được refill theo rate.
    - **How it works**:
        1. **Bucket capacity**: Bucket có capacity (vd: 100 tokens).
        2. **Refill rate**: Tokens được refill theo rate (vd: 10 tokens/second).
        3. **Request**: Mỗi request consume 1 token → nếu có token → allow, nếu không → reject.
    - **Features**: Cho phép burst (nếu bucket full, có thể consume nhiều tokens cùng lúc).
- [x] Viết algorithm pseudocode cho Token Bucket
    ```pseudocode
    class TokenBucket:
        capacity = 100        // Max tokens
        tokens = 100         // Current tokens
        refillRate = 10      // Tokens per second
        lastRefill = now()   // Last refill time
    
    function allowRequest(bucket, tokensNeeded = 1):
        currentTime = now()
        timePassed = currentTime - bucket.lastRefill
        
        // Refill tokens
        tokensToAdd = timePassed * bucket.refillRate
        bucket.tokens = min(bucket.capacity, bucket.tokens + tokensToAdd)
        bucket.lastRefill = currentTime
        
        // Check if enough tokens
        if bucket.tokens >= tokensNeeded:
            bucket.tokens -= tokensNeeded
            return true  // Allow request
        else:
            return false  // Reject request
    ```
- [x] Đọc về "Leaky Bucket" algorithm
    - **Leaky Bucket**: Bucket có fixed capacity, requests được add vào bucket, bucket "leak" (process) requests theo rate.
    - **How it works**:
        1. **Bucket capacity**: Bucket có capacity (vd: 100 requests).
        2. **Leak rate**: Requests được process theo rate (vd: 10 requests/second).
        3. **Request**: Nếu bucket không full → add request, nếu full → reject.
    - **Features**: Smooth rate (không cho phép burst như Token Bucket), requests được process đều đều.
- [x] Viết algorithm pseudocode cho Leaky Bucket
    ```pseudocode
    class LeakyBucket:
        capacity = 100        // Max requests
        queue = []           // Request queue
        leakRate = 10        // Requests per second
        lastLeak = now()     // Last leak time
    
    function allowRequest(bucket):
        currentTime = now()
        timePassed = currentTime - bucket.lastLeak
        
        // Process requests (leak)
        requestsToProcess = timePassed * bucket.leakRate
        for i = 0 to min(requestsToProcess, bucket.queue.length):
            request = bucket.queue.dequeue()
            processRequest(request)
        bucket.lastLeak = currentTime
        
        // Check if bucket has space
        if bucket.queue.length < bucket.capacity:
            bucket.queue.enqueue(currentRequest)
            return true  // Allow request
        else:
            return false  // Reject request (bucket full)
    ```
- [x] Đọc về "Sliding Window" algorithm
    - **Sliding Window**: Track requests trong sliding time window, limit requests trong window.
    - **How it works**:
        1. **Time window**: Window size (vd: 1 minute).
        2. **Request tracking**: Track timestamps của requests trong window.
        3. **Limit check**: Count requests trong window → nếu < limit → allow, nếu không → reject.
        4. **Sliding**: Window slide theo thời gian → old requests tự động expire.
    - **Features**: Accurate rate limiting, smooth distribution.
- [x] Viết algorithm pseudocode cho Sliding Window
    ```pseudocode
    class SlidingWindow:
        limit = 100          // Max requests
        windowSize = 60      // Window size in seconds
        requests = []        // Request timestamps
    
    function allowRequest(window):
        currentTime = now()
        
        // Remove old requests outside window
        cutoffTime = currentTime - window.windowSize
        window.requests = filter(window.requests, timestamp > cutoffTime)
        
        // Check limit
        if window.requests.length < window.limit:
            window.requests.append(currentTime)
            return true  // Allow request
        else:
            return false  // Reject request
    ```
- [x] Đọc về "Fixed Window" algorithm
    - **Fixed Window**: Limit requests trong fixed time window (vd: 1 minute), reset window mỗi period.
    - **How it works**:
        1. **Time window**: Fixed window (vd: minute 0-59, 60-119, ...).
        2. **Counter**: Count requests trong current window.
        3. **Limit check**: Nếu counter < limit → allow, nếu không → reject.
        4. **Reset**: Khi window expires → counter reset về 0.
    - **Features**: Simple, nhưng có thể có burst ở boundary (vd: 100 requests ở cuối window + 100 requests ở đầu window mới = 200 requests trong 2 giây).
- [x] Compare: Rate limiting algorithms (5 criteria)
    | Criteria | Token Bucket | Leaky Bucket | Sliding Window | Fixed Window |
    |----------|-------------|--------------|----------------|--------------|
    | **Burst Allowed** | Yes | No | Depends | Yes (at boundary) |
    | **Smooth Rate** | No | Yes | Yes | No |
    | **Accuracy** | Medium | High | High | Low (boundary issue) |
    | **Complexity** | Low | Medium | Medium | Low |
    | **Memory Usage** | Low | Medium | High (store timestamps) | Low |
- [x] Research: Redis-based rate limiting implementations
    1. **Fixed Window với Redis INCR**:
        - Key: `rate_limit:{user_id}:{window}`.
        - `INCR key` → check count → set TTL.
        - Simple, nhưng có boundary issue.
    2. **Sliding Window với Redis Sorted Set**:
        - Key: `rate_limit:{user_id}`.
        - Add request timestamp với `ZADD`, remove old với `ZREMRANGEBYSCORE`, count với `ZCARD`.
        - Accurate, nhưng tốn memory.
    3. **Token Bucket với Redis**:
        - Key: `token_bucket:{user_id}`.
        - Store `{tokens, lastRefill}` trong Hash.
        - Lua script để atomic refill và consume.
    4. **Leaky Bucket với Redis List**:
        - Key: `leaky_bucket:{user_id}`.
        - Use Redis List như queue, process với background worker.

### Performance Optimization
- [x] Đọc về "latency" vs "throughput" trade-offs
    - **Latency**: Thời gian để complete một request (vd: 10ms).
    - **Throughput**: Số requests có thể process trong một đơn vị thời gian (vd: 1000 QPS).
    - **Trade-offs**:
        1. **Batching**: Tăng throughput (process nhiều requests cùng lúc) nhưng tăng latency (wait để batch).
        2. **Connection pooling**: Tăng throughput (reuse connections) nhưng có thể tăng latency nếu pool exhausted.
        3. **Caching**: Giảm latency (fast reads) nhưng có thể giảm throughput nếu cache miss nhiều.
        4. **Async processing**: Tăng throughput (non-blocking) nhưng có thể tăng latency (queue wait).
    - **Rule of thumb**: Optimize cho metric quan trọng nhất (user-facing → latency, batch processing → throughput).
- [x] Đọc về "pipelining" trong Redis
    - **Pipelining**: Batch multiple commands → send cùng lúc → reduce round trips.
    - **How it works**:
        1. Client queue multiple commands.
        2. Send tất cả commands trong 1 batch.
        3. Redis process và return results.
    - **Performance**: Giảm latency từ N round trips → 1 round trip.
        - Ví dụ: 100 commands → không pipeline: 100 × 1ms = 100ms; pipeline: 1ms → **100x improvement**.
    - **Use cases**: Bulk operations, loading data, batch updates.
- [x] Đọc về "batch operations" - performance improvement
    - **Batch Operations**: Group multiple operations → execute cùng lúc → reduce overhead.
    - **Benefits**:
        1. **Reduce round trips**: Network overhead giảm.
        2. **Reduce context switching**: OS overhead giảm.
        3. **Better resource utilization**: CPU, I/O được sử dụng hiệu quả hơn.
    - **Examples**:
        - **Redis MGET**: Get multiple keys cùng lúc.
        - **Redis MSET**: Set multiple keys cùng lúc.
        - **Database batch insert**: Insert nhiều rows trong 1 query.
    - **Trade-off**: Tăng latency (wait để batch) nhưng tăng throughput.
- [x] Đọc về "connection pooling" cho Redis
    - **Connection Pooling**: Reuse connections thay vì create new connection mỗi request.
    - **How it works**:
        1. Pool maintain một số connections sẵn sàng.
        2. Request borrow connection từ pool.
        3. Sau khi dùng xong → return connection về pool.
    - **Benefits**:
        1. **Reduce connection overhead**: TCP handshake, authentication chỉ làm 1 lần.
        2. **Better resource utilization**: Reuse connections → giảm memory, CPU.
        3. **Higher throughput**: Không phải wait để create connection.
    - **Configuration**: Pool size = `(core_count × 2) + effective_spindle_count`, tune based on workload.
- [x] Đọc về "serialization" - JSON vs MessagePack vs Protobuf
    - **Serialization**: Convert objects thành bytes để store trong cache.
    - **JSON**: Human-readable, widely supported, nhưng lớn và chậm.
    - **MessagePack**: Binary format, nhỏ hơn JSON ~30%, nhanh hơn ~2x.
    - **Protobuf**: Google's binary format, nhỏ nhất, nhanh nhất, nhưng cần schema definition.
- [x] Compare: Serialization formats (performance, size)
    | Format | Size | Serialization Speed | Deserialization Speed | Human Readable |
    |--------|------|---------------------|----------------------|----------------|
    | **JSON** | 100% (baseline) | 1x (baseline) | 1x (baseline) | Yes |
    | **MessagePack** | ~70% | ~2x faster | ~2x faster | No |
    | **Protobuf** | ~50-60% | ~3-5x faster | ~3-5x faster | No |
    | **Java Serialization** | ~150% | Slower | Slower | No |
- [x] Đọc về "compression" - when to compress cache?
    - **Compression**: Compress data trước khi cache → giảm memory usage, network transfer time.
    - **When to compress**:
        1. **Large values**: Values > 1KB → compression có benefit.
        2. **Text data**: JSON, XML, text → compress tốt (gzip, snappy).
        3. **Memory constrained**: Khi memory là bottleneck.
        4. **Network bandwidth limited**: Khi network transfer là bottleneck.
    - **When NOT to compress**:
        1. **Small values**: Values < 100 bytes → compression overhead > benefit.
        2. **Already compressed**: Images, videos → không compress thêm.
        3. **CPU constrained**: Compression tốn CPU → có thể làm chậm hơn.
    - **Algorithms**: gzip (best compression), snappy (fast), lz4 (balanced).
- [x] Research: Cache performance best practices
    1. **Measure everything**: Cache hit rate, latency, memory usage → optimize based on data.
    2. **Right data structure**: Chọn data structure phù hợp (Hash thay vì multiple Strings).
    3. **TTL strategy**: Staggered TTL để tránh cache breakdown.
    4. **Connection pooling**: Reuse connections → reduce overhead.
    5. **Pipelining**: Batch operations → reduce round trips.
    6. **Serialization**: Chọn format phù hợp (MessagePack/Protobuf cho performance, JSON cho debugging).
    7. **Compression**: Compress large values → save memory.
    8. **Monitor hot keys**: Identify và optimize hot keys.
    9. **Avoid big keys**: Split large keys → reduce blocking.
    10. **Cache warming**: Pre-load hot data → improve hit rate.

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
