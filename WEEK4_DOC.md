# Week 4 – Database Sharding & Partitioning

> **Mentor Note**: Sharding là một trong những challenges khó nhất trong distributed systems. Mỗi decision bạn make sẽ impact toàn bộ system. Bạn phải justify MỌI decision với data và analysis. Không có "tôi nghĩ" hay "có vẻ OK". Phải có numbers, measurements, và trade-off analysis. Nếu bạn không thể justify sharding strategy của bạn, bạn chưa hiểu đủ sâu.

---

## Study TODOs

### Partitioning Fundamentals
- [x] Đọc "Designing Data-Intensive Applications" Chapter 6 - Partitioning (pages 200-250)
  - **Key Takeaways**: Partitioning (hay Sharding) là việc chia nhỏ một dataset lớn thành các phần nhỏ hơn (partitions) để scale out. Mục tiêu chính là scalability (vượt qua giới hạn lưu trữ/xử lý của 1 node) và availability.
- [x] Định nghĩa chính xác: Horizontal partitioning (sharding)
  - Là việc chia bảng theo hàng (rows). Mỗi partition có cùng schema nhưng chứa các subset hàng khác nhau. Ví dụ: Shard theo `user_id`, User 1-1000 ở Node A, 1001-2000 ở Node B.
- [x] Định nghĩa chính xác: Vertical partitioning
  - Là việc chia bảng theo cột (columns). Tách các cột ít dùng hoặc dung lượng lớn sang bảng/DB khác. Ví dụ: Bảng `Users` tách `user_id, username` sang DB chính, `user_id, bio, avatar_blob` sang DB phụ.
- [x] So sánh: Horizontal vs Vertical partitioning (5 điểm khác biệt)
  1. **Data Split**: Horizontal chia theo Rows; Vertical chia theo Columns.
  2. **Goal**: Horizontal để scale volume/throughput; Vertical để tối ưu I/O (tách cột to/ít dùng).
  3. **Complexity**: Horizontal phức tạp hơn (cần shard routing); Vertical đơn giản hơn (thường là JOIN lại).
  4. **Query impact**: Horizontal ảnh hưởng cross-shard queries; Vertical ảnh hưởng JOIN latency.
  5. **Schema**: Horizontal giữ nguyên schema; Vertical thay đổi schema (tách bảng).
- [x] Liệt kê 5 use cases cần horizontal partitioning
  1. Hệ thống Social Media (tỷ user, shard theo `user_id`).
  2. E-commerce Transactions (hàng tỷ đơn hàng, shard theo `order_id` hoặc `merchant_id`).
  3. IoT Sensor Data (dữ liệu cực lớn, shard theo `device_id` hoặc `timestamp`).
  4. Payment Systems (shard theo `merchant_id` hoặc `user_id`).
  5. Chat/Messaging (shard theo `conversation_id`).
- [x] Liệt kê 3 use cases cần vertical partitioning
  1. Tách `Product` description dài (BLOB) khỏi metadata cơ bản để list hàng nhanh hơn.
  2. Tách thông tin nhạy cảm (PII) sang DB có bảo mật cao hơn.
  3. Tách các cột thường xuyên update khỏi các cột static để giảm write amplification.
- [x] Đọc về "partitioning vs sharding" - same thing? Different?
  - Trong ngữ cảnh distributed systems, chúng thường dùng thay thế cho nhau. Tuy nhiên, "Partitioning" là thuật ngữ chung (có thể trong 1 node - e.g. Postgres Partitioning), còn "Sharding" ám chỉ việc chia ra nhiều nodes vật lý khác nhau (Shared-nothing architecture).
- [x] Research: When to start sharding? (at what scale?)
  - Không có số cụ thể, nhưng thường khi: 
    - Database size vượt quá khả năng backup/restore trong thời gian chấp nhận được.
    - Write throughput vượt quá giới hạn của Master (vertical scaling Master đã hết mức).
    - Latency tăng cao do index quá lớn không còn fit trong RAM.
    - Thường bắt đầu cân nhắc khi đạt ngưỡng vài TB data hoặc hàng chục nghìn QPS write.

### Sharding Strategies Deep Dive
- [x] Đọc về "Range-based sharding" - how it works
  - Chia data theo các dải giá trị liên tục của Shard Key (ví dụ: A-M ở Shard 1, N-Z ở Shard 2).
- [x] Đọc về "Hash-based sharding" - how it works
  - Dùng Hash Function: `shard = hash(key) % number_of_shards`. Giúp phân phối data đều hơn.
- [x] Đọc về "Directory-based sharding" - how it works
  - Dùng một Lookup Table (Service) để lưu mapping: `key -> shard_id`. Linh hoạt nhưng tạo bottleneck ở Lookup Service.
- [x] Đọc về "Composite sharding" - multiple shard keys
  - Kết hợp nhiều keys hoặc nhiều tầng sharding (ví dụ: Shard theo `region` trước, sau đó `user_id`).
- [x] Viết comparison table: Range vs Hash vs Directory (10 criteria)

| Criteria | Range-based | Hash-based | Directory-based |
| :--- | :--- | :--- | :--- |
| **Data Distribution** | Dễ bị lệch (Skewed) | Rất đều | Tùy chỉnh |
| **Range Queries** | Rất tốt | Rất tệ (Fan-out) | Trung bình |
| **Scalability** | Dễ (Split range) | Khó (Rehash) | Rất dễ |
| **Complexity** | Thấp | Trung bình | Cao |
| **Hotspots** | Dễ bị (e.g. timestamp) | Hiếm gặp | Có thể manage |
| **Routing Logic** | Simple | Deterministic | Lookup service |
| **Re-sharding** | Dễ | Rất khó/Phức tạp | Dễ |
| **Config overhead** | Thấp | Thấp | Cao |
| **Availability** | Cao | Cao | Phụ thuộc Lookup Service |
| **Query Performance** | Tốt cho dải data | Tốt cho point-lookup | Tốt cho mapping linh hoạt |

- [x] Analyze: Pros và cons của mỗi strategy (5 points each)
  - **Range**:
    - Pros: Range query nhanh, dễ split shard, routing đơn giản, locality tốt.
    - Cons: Dễ bị hotspot (e.g. data mới nhất), distribution không đều, cần monitor dải giá trị.
  - **Hash**:
    - Pros: Data phân phối rất đều, tránh hotspot, không cần lookup table, point lookup nhanh.
    - Cons: Range query cực chậm (fan-out), re-sharding rất đau đớn, không có data locality.
  - **Directory**:
    - Pros: Cực kỳ linh hoạt, có thể move data giữa shards tự do, handle được hotspot lẻ.
    - Cons: Lookup service là bottleneck/SPOF, overhead latency cho mỗi query, quản lý mapping phức tạp.
- [x] Research: Real-world examples của range-based sharding: Google BigTable, HBase (SSTables).
- [x] Research: Real-world examples của hash-based sharding: Cassandra, Memcached, DynamoDB.
- [x] Research: Real-world examples của directory-based sharding: Azure SQL Database (Elastic Database tools).
- [x] Đọc về "shard key" - what is it? Why critical?
  - Shard Key là cột xác định dữ liệu thuộc về shard nào. Nó critical vì quyết định: Phân phối tải, query performance, và khả năng scale. Chọn sai shard key có thể dẫn tới hotspots và cross-shard queries làm sập hệ thống.

### Consistent Hashing
- [ ] Đọc về "consistent hashing" - original paper hoặc detailed explanation
- [ ] Viết algorithm pseudocode cho consistent hashing
- [ ] Đọc về "virtual nodes" trong consistent hashing
- [ ] Analyze: Why virtual nodes needed?
- [ ] Đọc về "hash ring" - how it works
- [ ] Đọc về "rehashing problem" - why consistent hashing solves it
- [ ] Calculate: Load distribution với consistent hashing (theoretical)
- [ ] Research: Consistent hashing implementations (Ketama, etc.)

### Shard Key Selection
- [ ] Đọc về "shard key selection criteria"
- [ ] Liệt kê 5 criteria để chọn shard key
- [ ] Analyze: Shard key by user_id - pros và cons
- [ ] Analyze: Shard key by merchant_id - pros và cons
- [ ] Analyze: Shard key by geographic region - pros và cons
- [ ] Analyze: Shard key by timestamp - pros và cons
- [ ] Đọc về "cardinality" của shard key - why matters?
- [ ] Đọc về "hotspot" problem - what causes it?
- [ ] Analyze: How shard key selection affects hotspots
- [ ] Research: Composite shard keys - when to use?

### Hotspot Mitigation
- [ ] Đọc về "hotspot" problem trong sharding
- [ ] Analyze: What causes hotspots? (5 causes)
- [ ] Đọc về "hot partition" - single shard overloaded
- [ ] Đọc về "hot key" - single key overloaded
- [ ] Research: Strategies to mitigate hotspots
- [ ] Đọc về "salting" technique - add random suffix
- [ ] Đọc về "range splitting" - split hot ranges
- [ ] Đọc về "replication" của hot shards
- [ ] Analyze: Trade-offs của mỗi mitigation strategy

### Cross-Shard Queries
- [ ] Đọc về "cross-shard queries" - challenges
- [ ] Analyze: Why cross-shard queries are expensive
- [ ] Đọc về "fan-out queries" - query all shards
- [ ] Đọc về "scatter-gather" pattern
- [ ] Analyze: Performance impact của cross-shard queries
- [ ] Research: Strategies to minimize cross-shard queries
- [ ] Đọc về "denormalization" để avoid cross-shard queries
- [ ] Đọc về "materialized views" across shards
- [ ] Đọc về "global secondary indexes" - challenges

### Distributed Transactions
- [ ] Đọc về "distributed transactions" - 2PC (Two-Phase Commit)
- [ ] Đọc về "2PC protocol" - how it works step by step
- [ ] Analyze: Performance impact của 2PC
- [ ] Analyze: Failure scenarios trong 2PC
- [ ] Đọc về "3PC" (Three-Phase Commit) - improvement?
- [ ] Đọc về "Saga pattern" - alternative to 2PC
- [ ] Compare: 2PC vs Saga pattern (5 criteria)
- [ ] Đọc về "compensating transactions" trong Saga
- [ ] Research: When to use distributed transactions? When to avoid?

### Re-sharding Strategies
- [ ] Đọc về "resharding" - why needed?
- [ ] Đọc về "resharding triggers" - when to reshard?
- [ ] Đọc về "resharding strategies" - how to do it?
- [ ] Đọc về "double-write" strategy during resharding
- [ ] Đọc về "gradual migration" strategy
- [ ] Đọc về "big bang migration" strategy
- [ ] Compare: Gradual vs Big bang migration (5 criteria)
- [ ] Đọc về "resharding downtime" - how to minimize?
- [ ] Research: Real-world resharding case studies

### Shard Management
- [ ] Đọc về "shard routing" - how to route queries
- [ ] Đọc về "shard metadata" - where to store?
- [ ] Đọc về "shard discovery" - how clients find shards
- [ ] Đọc về "shard monitoring" - what to monitor?
- [ ] Đọc về "shard health checks"
- [ ] Đọc về "shard rebalancing" - when và how?
- [ ] Research: Shard management tools và frameworks

---

## Data Distribution TODOs

### Design Exercise 1: Payment System Sharding
- [ ] Requirement: 1B transactions, shard by merchant_id
- [ ] Design: Sharding strategy (Range, Hash, or Directory?)
- [ ] Justify: Tại sao chọn strategy đó? (viết 500 words với data)
- [ ] Design: Shard key selection (merchant_id only? Composite?)
- [ ] Justify: Shard key choice với analysis
- [ ] Calculate: Number of shards needed (assume 100M records per shard)
- [ ] Design: Shard routing logic
- [ ] Design: Shard metadata storage
- [ ] Identify: Potential hotspots (which merchants?)
- [ ] Design: Hotspot mitigation strategy
- [ ] Design: Cross-merchant query handling
- [ ] Design: Re-sharding strategy (when merchant count grows)
- [ ] Document: Complete sharding design (1000 words)

### Design Exercise 2: Wallet System Sharding
- [ ] Requirement: 10M users, 100M transactions, shard by user_id
- [ ] Design: Sharding strategy với consistent hashing
- [ ] Justify: Tại sao consistent hashing? (viết analysis)
- [ ] Design: Shard key (user_id only? Composite với transaction_id?)
- [ ] Calculate: Shard count (assume 1M users per shard)
- [ ] Design: Virtual nodes configuration (how many per shard?)
- [ ] Analyze: Load distribution với consistent hashing
- [ ] Design: Hotspot mitigation (VIP users có nhiều transactions)
- [ ] Design: Cross-shard queries (user transaction history across shards?)
- [ ] Design: Balance consistency (user balance có thể cross-shard không?)
- [ ] Design: Re-sharding strategy
- [ ] Document: Sharding design với justifications

### Design Exercise 3: Betting Platform Sharding
- [ ] Requirement: Shard bets, consider: user_id, match_id, timestamp
- [ ] Analyze: Shard key options (user_id vs match_id vs composite)
- [ ] Compare: Each shard key option (5 criteria mỗi option)
- [ ] Choose: Best shard key với justification
- [ ] Design: Sharding strategy
- [ ] Analyze: Query patterns (user queries vs match queries)
- [ ] Design: Handle both query patterns (user bets vs match bets)
- [ ] Design: Cross-shard query strategy
- [ ] Design: Hotspot mitigation (popular matches, VIP users)
- [ ] Design: Re-sharding strategy
- [ ] Document: Design với trade-off analysis

### Design Exercise 4: Hotspot Analysis
- [ ] Scenario: Payment system, 80% transactions từ 5% merchants
- [ ] Analyze: Hotspot problem severity
- [ ] Calculate: Load distribution (transactions per shard)
- [ ] Design: Sharding strategy để handle hotspots
- [ ] Option 1: Shard by merchant_id với salting
- [ ] Analyze: Salting strategy effectiveness
- [ ] Option 2: Dedicated shards cho hot merchants
- [ ] Analyze: Dedicated shards strategy
- [ ] Compare: Options (cost, complexity, effectiveness)
- [ ] Recommend: Best strategy với justification
- [ ] Document: Hotspot mitigation design

### Design Exercise 5: Cross-Shard Query Design
- [ ] Scenario: Need to query transactions across all merchants
- [ ] Design: Fan-out query strategy
- [ ] Calculate: Performance impact (latency, throughput)
- [ ] Design: Aggregation strategy (sum, count across shards)
- [ ] Design: Pagination across shards
- [ ] Design: Sorting across shards
- [ ] Analyze: Limitations và trade-offs
- [ ] Design: Alternative - denormalized reporting table
- [ ] Compare: Fan-out vs denormalized approach
- [ ] Recommend: Best approach với justification
- [ ] Document: Cross-shard query design

### Design Exercise 6: Re-sharding Strategy
- [ ] Scenario: Current 10 shards, need to scale to 20 shards
- [ ] Design: Re-sharding strategy (gradual vs big bang)
- [ ] Choose: Strategy với justification
- [ ] Design: Data migration plan
- [ ] Design: Double-write strategy during migration
- [ ] Design: Traffic routing during migration
- [ ] Design: Rollback plan (if migration fails)
- [ ] Calculate: Migration time estimate
- [ ] Calculate: Downtime (if any)
- [ ] Design: Verification strategy (data consistency)
- [ ] Document: Complete re-sharding plan

---

## Spring Boot Multi-DB TODOs

### Task 1: Multi-Database Configuration
- [ ] Setup: Spring Boot project với multiple data sources
- [ ] Configure: 3 separate databases (shard1, shard2, shard3)
- [ ] Configure: DataSource beans cho mỗi shard
- [ ] Configure: EntityManagerFactory cho mỗi shard
- [ ] Configure: TransactionManager cho mỗi shard
- [ ] Test: Connect to each shard
- [ ] Document: Multi-database configuration

### Task 2: Shard Routing Service
- [ ] Create: ShardRouter service class
- [ ] Implement: Hash-based shard selection
- [ ] Method: getShard(userId) - returns shard number
- [ ] Implement: Consistent hashing algorithm
- [ ] Add: Virtual nodes support
- [ ] Test: Shard distribution (1000 user IDs, verify even distribution)
- [ ] Test: Add new shard, verify minimal data movement
- [ ] Test: Remove shard, verify data redistribution
- [ ] Measure: Shard selection performance
- [ ] Document: Shard routing implementation

### Task 3: Dynamic DataSource Routing
- [ ] Implement: AbstractRoutingDataSource extension
- [ ] Implement: determineCurrentLookupKey() method
- [ ] Integrate: Với ShardRouter service
- [ ] Configure: Spring to use routing DataSource
- [ ] Test: Queries route to correct shard
- [ ] Test: Transaction routing
- [ ] Document: Dynamic routing implementation

### Task 4: Shard-Aware Repository
- [ ] Create: Base repository interface
- [ ] Implement: findById với shard routing
- [ ] Implement: save với shard routing
- [ ] Implement: Custom queries với shard routing
- [ ] Test: CRUD operations route correctly
- [ ] Test: Query performance
- [ ] Document: Shard-aware repository

### Task 5: Cross-Shard Query Implementation
- [ ] Implement: Fan-out query service
- [ ] Method: queryAllShards(query) - execute on all shards
- [ ] Implement: Result aggregation (merge results)
- [ ] Implement: Pagination across shards
- [ ] Implement: Sorting across shards
- [ ] Test: Query all shards
- [ ] Measure: Performance (latency, throughput)
- [ ] Optimize: Parallel execution
- [ ] Document: Cross-shard query implementation

### Task 6: Shard Metadata Management
- [ ] Create: ShardMetadata entity/table
- [ ] Fields: shard_id, shard_host, shard_status, shard_range
- [ ] Implement: ShardMetadataService
- [ ] Methods: registerShard(), updateShardStatus(), getShardInfo()
- [ ] Implement: Shard health check
- [ ] Implement: Automatic shard discovery
- [ ] Test: Shard registration
- [ ] Test: Shard status updates
- [ ] Document: Shard metadata management

### Task 7: Connection Pool per Shard
- [ ] Configure: Separate HikariCP pool cho mỗi shard
- [ ] Configure: Pool size per shard
- [ ] Monitor: Connection pool metrics per shard
- [ ] Test: Pool không exhausted under load
- [ ] Document: Connection pool configuration

### Task 8: Shard Monitoring
- [ ] Add: Metrics cho mỗi shard (query count, latency, errors)
- [ ] Add: Shard health endpoints
- [ ] Implement: Shard load monitoring
- [ ] Implement: Alert khi shard overloaded
- [ ] Create: Dashboard với shard metrics
- [ ] Document: Shard monitoring setup

---

## Cross-Shard & Transaction TODOs

### Cross-Shard Query Analysis
- [ ] Scenario: Query user transactions across all shards
- [ ] Implement: Fan-out query
- [ ] Measure: Latency (p50, p95, p99)
- [ ] Measure: Throughput (QPS)
- [ ] Analyze: Bottleneck (network? database? aggregation?)
- [ ] Optimize: Parallel execution
- [ ] Measure: Performance improvement
- [ ] Compare: Fan-out vs single-shard query
- [ ] Document: Cross-shard query performance analysis

### Distributed Transaction Implementation
- [ ] Scenario: Transfer money between users on different shards
- [ ] Design: Transaction strategy (2PC vs Saga)
- [ ] Choose: Strategy với justification
- [ ] Implement: Distributed transaction
- [ ] If 2PC: Implement coordinator
- [ ] If Saga: Implement compensating transactions
- [ ] Test: Successful transaction
- [ ] Test: Failure scenarios (rollback)
- [ ] Measure: Performance impact
- [ ] Document: Distributed transaction implementation

### Saga Pattern Implementation
- [ ] Implement: Saga orchestrator
- [ ] Implement: Saga steps (local transactions)
- [ ] Implement: Compensating transactions
- [ ] Test: Saga success flow
- [ ] Test: Saga failure và compensation
- [ ] Test: Partial failure scenarios
- [ ] Measure: Saga performance
- [ ] Compare: Saga vs 2PC performance
- [ ] Document: Saga implementation

### Data Consistency Analysis
- [ ] Scenario: User balance update across shards
- [ ] Analyze: Consistency requirements (strong? eventual?)
- [ ] Design: Consistency strategy
- [ ] Implement: Consistency mechanism
- [ ] Test: Consistency under concurrent updates
- [ ] Test: Consistency after failures
- [ ] Measure: Performance impact của consistency
- [ ] Document: Consistency strategy

### Idempotency Across Shards
- [ ] Scenario: Payment processing, retry có thể hit different shards
- [ ] Design: Idempotency strategy
- [ ] Implement: Idempotency keys
- [ ] Implement: Idempotency check across shards
- [ ] Test: Duplicate requests handled correctly
- [ ] Test: Performance impact
- [ ] Document: Idempotency implementation

---

## Migration & Resharding TODOs

### Resharding Simulation
- [ ] Scenario: Scale from 3 shards to 6 shards
- [ ] Design: Resharding plan
- [ ] Implement: Data migration script
- [ ] Implement: Double-write during migration
- [ ] Implement: Data verification
- [ ] Test: Migration với sample data
- [ ] Measure: Migration time
- [ ] Test: Rollback procedure
- [ ] Document: Resharding procedure

### Zero-Downtime Resharding
- [ ] Design: Zero-downtime resharding strategy
- [ ] Implement: Gradual migration
- [ ] Implement: Traffic routing during migration
- [ ] Implement: Data sync verification
- [ ] Test: Migration không cause downtime
- [ ] Test: Read consistency during migration
- [ ] Test: Write consistency during migration
- [ ] Measure: Migration duration
- [ ] Document: Zero-downtime resharding

### Shard Rebalancing
- [ ] Scenario: Shard 1 has 2x load của Shard 2
- [ ] Design: Rebalancing strategy
- [ ] Implement: Load measurement
- [ ] Implement: Data movement logic
- [ ] Implement: Rebalancing algorithm
- [ ] Test: Rebalancing với sample data
- [ ] Measure: Rebalancing time
- [ ] Test: System availability during rebalancing
- [ ] Document: Rebalancing procedure

### Data Migration Testing
- [ ] Create: Test dataset (1M records across 3 shards)
- [ ] Run: Migration to 6 shards
- [ ] Verify: All data migrated correctly
- [ ] Verify: No data loss
- [ ] Verify: No duplicates
- [ ] Measure: Migration performance
- [ ] Test: Rollback
- [ ] Document: Migration test results

### Hotspot Migration
- [ ] Scenario: Hot merchant needs to move to dedicated shard
- [ ] Design: Hotspot migration strategy
- [ ] Implement: Identify hot data
- [ ] Implement: Migration logic
- [ ] Implement: Traffic routing update
- [ ] Test: Migration không affect other data
- [ ] Test: Zero downtime
- [ ] Document: Hotspot migration

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Data Distribution exercises hoàn thành?
- [ ] Review: Tất cả Spring Boot tasks hoàn thành?
- [ ] Review: Tất cả Cross-Shard tasks hoàn thành?
- [ ] Review: Tất cả Migration tasks hoàn thành?
- [ ] Rate: Sharding design skills (1-10)
- [ ] Rate: Shard key selection skills (1-10)
- [ ] Rate: Cross-shard query handling (1-10)
- [ ] Rate: Distributed transaction understanding (1-10)
- [ ] Identify: 3 sharding concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Sharding Design Review
- [ ] Review: Payment System sharding design
- [ ] Check: Sharding strategy justified với data?
- [ ] Check: Shard key selection optimal?
- [ ] Check: Hotspot mitigation adequate?
- [ ] Check: Cross-shard queries handled?
- [ ] Identify: 3 weaknesses trong design
- [ ] Propose: Improvements
- [ ] Compare: Design vs industry best practices
- [ ] Document: Design review findings

### Code Review
- [ ] Review: Shard routing implementation
- [ ] Check: Consistent hashing correct?
- [ ] Check: Load distribution even?
- [ ] Review: Cross-shard query implementation
- [ ] Check: Performance acceptable?
- [ ] Review: Distributed transaction code
- [ ] Check: Error handling adequate?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Shard routing performance
- [ ] Review: Cross-shard query performance
- [ ] Review: Distributed transaction performance
- [ ] Measure: System performance under load
- [ ] Identify: Performance bottlenecks
- [ ] Verify: Sharding improves performance?
- [ ] Document: Performance analysis
- [ ] Create: Performance improvement plan

### Knowledge Check
- [ ] Explain: Range vs Hash vs Directory sharding (viết comparison, không xem notes)
- [ ] Explain: Consistent hashing - how it works (viết explanation, không xem notes)
- [ ] Explain: Shard key selection criteria (viết 5 criteria, không xem notes)
- [ ] Explain: Cross-shard query challenges (viết analysis, không xem notes)
- [ ] Explain: 2PC vs Saga pattern (viết comparison, không xem notes)
- [ ] Solve: 1B records, 10 shards, shard key = user_id - calculate records per shard (assume even distribution)
- [ ] Solve: Hot merchant có 10% of all transactions - design mitigation strategy
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 sharding concepts còn confuse
- [ ] Write: 1 mistake đã làm và lesson learned
- [ ] Write: Confidence level cho Week 5 (1-10)
- [ ] Compare: Week 3 vs Week 4 progress
- [ ] Plan: Preparation cho Week 5 (Caching)
- [ ] Set: Goals cho Week 5
- [ ] Document: Week 4 reflection (500 words)

### Mentor Questions (Answer these - justify everything)
- [ ] Q1: Bạn có 1B transactions, shard by merchant_id. Merchant A có 50% of all transactions. Làm sao bạn handle? (viết detailed solution với justification)
- [ ] Q2: Cross-shard query cần aggregate SUM(amount) across all shards. Làm sao bạn implement? Performance impact? (viết implementation plan với performance analysis)
- [ ] Q3: User transfer money từ Shard 1 sang Shard 2. Làm sao bạn ensure atomicity? 2PC hay Saga? Justify choice. (viết analysis với trade-offs)
- [ ] Q4: Bạn cần reshard từ 10 shards lên 20 shards. Làm sao zero downtime? (viết detailed migration plan)
- [ ] Q5: Shard key = user_id, nhưng query pattern = WHERE merchant_id = ? AND date = ?. Làm sao optimize? (viết optimization strategy)
- [ ] Q6: Consistent hashing với 10 shards, add 2 more shards. Bao nhiêu % data cần move? (calculate và explain)
- [ ] Review: Answers có đủ depth, justification, và numbers không?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Data Distribution TODOs: ✅ Complete với justifications
- [ ] Tất cả Spring Boot Multi-DB TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Cross-Shard & Transaction TODOs: ✅ Complete với performance analysis
- [ ] Tất cả Migration & Resharding TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Sharding design documents saved: ✅ Yes
- [ ] Performance benchmarks saved: ✅ Yes
- [ ] Ready for Week 5: ✅ Yes

---

> **Mentor Final Check**: Sharding là art và science. Mỗi decision phải được justify với data. Nếu bạn không thể explain TẠI SAO bạn chọn shard key đó, strategy đó, bạn chưa hiểu đủ. Hãy honest: Bạn có thể design sharding strategy cho 1B+ records chưa? Bạn có thể justify MỌI decision chưa? Nếu chưa, làm lại. Production systems không chấp nhận "tôi nghĩ nó sẽ work".
