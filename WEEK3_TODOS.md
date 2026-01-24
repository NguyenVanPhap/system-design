# Week 3 – Database Design & SQL Scaling

> **Mentor Note**: Database là backbone của mọi hệ thống. Nếu database chậm, toàn bộ system chậm. Tuần này bạn phải MASTER database performance. Không có "query chạy được là OK". Mỗi query phải được optimize, measure, và verify. Production systems không chấp nhận slow queries.

---

## Study TODOs

### Database Schema Design Fundamentals
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 2 - Data Models (pages 30-80)
- [ ] Đọc về "Normalization" - 1NF, 2NF, 3NF, BCNF
- [ ] Viết notes: Khi nào normalize? Khi nào denormalize?
- [ ] Đọc về "denormalization" - trade-offs với normalization
- [ ] Liệt kê 5 use cases cần normalization
- [ ] Liệt kê 5 use cases cần denormalization
- [ ] Đọc về "star schema" vs "snowflake schema"
- [ ] Đọc về "entity-relationship modeling" best practices
- [ ] Đọc về "foreign key constraints" - performance impact
- [ ] Research: Foreign keys trong production - enable hay disable? Tại sao?

### Indexing Deep Dive
- [ ] Đọc về "B-tree index" - how it works internally
- [ ] Đọc về "Hash index" - use cases và limitations
- [ ] Đọc về "Composite index" - column order matters
- [ ] Viết notes: Index selectivity - what is it? Why important?
- [ ] Đọc về "covering index" - index-only scans
- [ ] Đọc về "partial index" - conditional indexes
- [ ] Đọc về "unique index" vs "non-unique index" - performance difference
- [ ] Đọc về "index maintenance" - INSERT/UPDATE/DELETE overhead
- [ ] Research: How many indexes is too many? (general rule)
- [ ] Đọc về "index fragmentation" - when và how to rebuild
- [ ] Đọc về "clustered index" vs "non-clustered index" (SQL Server) hoặc "primary key" vs "secondary index" (MySQL)
- [ ] Đọc về "full-text index" - when to use

### Query Optimization
- [ ] Đọc về "EXPLAIN" / "EXPLAIN PLAN" - how to read output
- [ ] Đọc về "query execution plan" - steps database takes
- [ ] Đọc về "table scan" vs "index scan" vs "index seek"
- [ ] Đọc về "cost-based optimizer" - how it works
- [ ] Đọc về "query hints" - when to use (rarely)
- [ ] Đọc về "statistics" - why database needs them
- [ ] Đọc về "cardinality estimation" - why it matters
- [ ] Đọc về "JOIN algorithms" - nested loop, hash join, merge join
- [ ] Đọc về "subquery" vs "JOIN" - performance comparison
- [ ] Đọc về "EXISTS" vs "IN" vs "JOIN" - when to use which
- [ ] Đọc về "LIMIT/OFFSET" performance issues
- [ ] Đọc về "cursor-based pagination" - better alternative

### Connection Pooling
- [ ] Đọc về "connection pooling" - why needed?
- [ ] Đọc về "HikariCP" - default Spring Boot pool
- [ ] Đọc về pool size calculation: connections = ((core_count * 2) + effective_spindle_count)
- [ ] Đọc về "connection pool tuning" - min, max, idle timeout
- [ ] Đọc về "connection leak detection"
- [ ] Đọc về "connection timeout" vs "query timeout"
- [ ] Research: HikariCP configuration best practices
- [ ] Đọc về "prepared statements" - performance và security
- [ ] Đọc về "statement caching" - reduce parsing overhead

### Read Replicas & Replication
- [ ] Đọc về "master-slave replication" - how it works
- [ ] Đọc về "replication lag" - what causes it?
- [ ] Đọc về "eventual consistency" trong read replicas
- [ ] Đọc về "read-after-write consistency" problem
- [ ] Đọc về "read scaling" - when read replicas help
- [ ] Đọc về "write scaling" - read replicas DON'T help
- [ ] Research: MySQL replication setup
- [ ] Research: PostgreSQL replication setup
- [ ] Đọc về "replication topologies" - chain, star, etc.
- [ ] Đọc về "failover" trong master-slave setup

### Partitioning Basics
- [ ] Đọc về "horizontal partitioning" (sharding)
- [ ] Đọc về "vertical partitioning" (column splitting)
- [ ] Đọc về "range partitioning" - partition by date range
- [ ] Đọc về "hash partitioning" - partition by hash
- [ ] Đọc về "list partitioning" - partition by category
- [ ] Đọc về "partition pruning" - query optimization
- [ ] Đọc về "cross-partition queries" - performance impact
- [ ] Research: MySQL partitioning
- [ ] Research: PostgreSQL partitioning

### Transaction Isolation Levels
- [ ] Đọc về "ACID properties" - Atomicity, Consistency, Isolation, Durability
- [ ] Đọc về "transaction isolation levels" - READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
- [ ] Đọc về "dirty read" - what is it?
- [ ] Đọc về "non-repeatable read" - what is it?
- [ ] Đọc về "phantom read" - what is it?
- [ ] Viết table: Isolation level → Prevents which anomalies?
- [ ] Đọc về "MVCC" (Multi-Version Concurrency Control) - how it works
- [ ] Đọc về "locking" - shared locks, exclusive locks
- [ ] Đọc về "deadlock" - what causes it? How to prevent?
- [ ] Đọc về "optimistic locking" vs "pessimistic locking"
- [ ] Research: Default isolation level trong MySQL vs PostgreSQL

### Data Consistency vs Performance
- [ ] Đọc về "strong consistency" - trade-offs
- [ ] Đọc về "eventual consistency" - trade-offs
- [ ] Đọc về "read consistency" - snapshot isolation
- [ ] Đọc về "write consistency" - how to ensure
- [ ] Analyze: When to sacrifice consistency for performance?
- [ ] Analyze: When MUST have strong consistency?
- [ ] Đọc về "distributed transactions" - 2PC, performance impact

---

## Schema & Modeling TODOs

### Schema Design Exercise 1: Wallet System
- [ ] Design: Complete database schema cho Wallet System
- [ ] Requirement: 10M users, 100M transactions, balance queries < 10ms
- [ ] Design: Users table (columns, data types, constraints)
- [ ] Design: Wallets table (columns, data types, constraints)
- [ ] Design: Transactions table (columns, data types, constraints)
- [ ] Design: Transaction types (enum hoặc lookup table)
- [ ] Design: Indexes cho mỗi table (list all indexes với rationale)
- [ ] Design: Foreign keys (which ones? why?)
- [ ] Verify: Normalization level (1NF, 2NF, 3NF?)
- [ ] Consider: Denormalization opportunities (nếu có)
- [ ] Create: ERD diagram (draw.io hoặc tool)
- [ ] Document: Schema design decisions (500 words)

### Schema Design Exercise 2: Payment Gateway
- [ ] Design: Database schema cho Payment Gateway
- [ ] Requirement: 1M transactions/day, complex reporting queries
- [ ] Design: Merchants table
- [ ] Design: Payments table (with all required fields)
- [ ] Design: Payment status tracking (state machine)
- [ ] Design: Refunds table
- [ ] Design: Payment methods table
- [ ] Design: Indexes strategy (consider query patterns)
- [ ] Consider: Partitioning strategy (by date? by merchant?)
- [ ] Design: Archive strategy (old transactions)
- [ ] Create: Complete schema với all tables
- [ ] Document: Design decisions và trade-offs

### Schema Design Exercise 3: Betting Platform
- [ ] Design: Database schema cho Betting Platform
- [ ] Requirement: Real-time odds updates, bet history, settlement
- [ ] Design: Matches table
- [ ] Design: Odds table (frequently updated)
- [ ] Design: Bets table
- [ ] Design: Bet outcomes table
- [ ] Design: User accounts table
- [ ] Design: Indexes (consider update frequency)
- [ ] Consider: How to handle high-frequency odds updates?
- [ ] Consider: Denormalization cho read-heavy queries
- [ ] Design: Data retention policy
- [ ] Create: Schema diagram
- [ ] Document: Design rationale

### Schema Review & Optimization
- [ ] Review: Wallet System schema
- [ ] Check: All indexes có necessary không?
- [ ] Check: Missing indexes? (identify potential)
- [ ] Check: Over-indexing? (too many indexes)
- [ ] Check: Data types optimal? (VARCHAR size, INT vs BIGINT)
- [ ] Check: Constraints appropriate? (NOT NULL, UNIQUE, CHECK)
- [ ] Optimize: Schema based on review
- [ ] Document: Schema review findings

---

## SQL Optimization TODOs

### Query Optimization Exercise 1: Slow Query Identification
- [ ] Setup: Database với 1M+ records (use sample data generator)
- [ ] Create: Users table với 1M records
- [ ] Create: Orders table với 10M records
- [ ] Create: OrderItems table với 50M records
- [ ] Write: Query to find user orders (JOIN Users và Orders)
- [ ] Run: EXPLAIN PLAN cho query
- [ ] Identify: Table scan? Index scan? Index seek?
- [ ] Measure: Query execution time
- [ ] Document: Baseline performance

### Query Optimization Exercise 2: Index Optimization
- [ ] Query: SELECT * FROM orders WHERE user_id = ? AND status = 'PENDING'
- [ ] Run: EXPLAIN PLAN (before optimization)
- [ ] Measure: Execution time (before)
- [ ] Analyze: Missing index? Wrong index?
- [ ] Create: Appropriate index (single hoặc composite)
- [ ] Run: EXPLAIN PLAN (after optimization)
- [ ] Measure: Execution time (after)
- [ ] Calculate: Performance improvement (%)
- [ ] Verify: Index được sử dụng (check EXPLAIN output)
- [ ] Document: Optimization results

### Query Optimization Exercise 3: JOIN Optimization
- [ ] Write: Complex query với 3-table JOIN
- [ ] Query: Users JOIN Orders JOIN OrderItems
- [ ] Run: EXPLAIN PLAN
- [ ] Identify: JOIN order (does it matter?)
- [ ] Identify: JOIN algorithm used (nested loop? hash join?)
- [ ] Optimize: JOIN order (if needed)
- [ ] Add: Indexes to support JOINs
- [ ] Measure: Performance before và after
- [ ] Document: JOIN optimization

### Query Optimization Exercise 4: Aggregation Optimization
- [ ] Write: Query với GROUP BY và aggregate functions
- [ ] Query: SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Analyze: Can index help GROUP BY?
- [ ] Create: Index to optimize GROUP BY
- [ ] Measure: Performance improvement
- [ ] Consider: Materialized view? (if applicable)
- [ ] Document: Aggregation optimization

### Query Optimization Exercise 5: Subquery vs JOIN
- [ ] Write: Query using subquery
- [ ] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000)
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Rewrite: Same query using JOIN
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Compare: Subquery vs JOIN performance
- [ ] Document: Which is better? Why?

### Query Optimization Exercise 6: EXISTS vs IN vs JOIN
- [ ] Write: Query using IN clause
- [ ] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders)
- [ ] Measure: Execution time
- [ ] Rewrite: Using EXISTS
- [ ] Measure: Execution time
- [ ] Rewrite: Using JOIN
- [ ] Measure: Execution time
- [ ] Compare: IN vs EXISTS vs JOIN
- [ ] Analyze: Which performs best? Why?
- [ ] Document: Best practice recommendation

### Query Optimization Exercise 7: Pagination Optimization
- [ ] Write: Query với LIMIT và OFFSET
- [ ] Example: SELECT * FROM orders ORDER BY created_at LIMIT 10 OFFSET 1000000
- [ ] Measure: Execution time (slow với large OFFSET)
- [ ] Analyze: Why is it slow?
- [ ] Rewrite: Using cursor-based pagination
- [ ] Example: SELECT * FROM orders WHERE id > ? ORDER BY id LIMIT 10
- [ ] Measure: Execution time
- [ ] Compare: OFFSET vs cursor-based
- [ ] Document: Pagination best practices

### Query Optimization Exercise 8: Full Table Scan Prevention
- [ ] Write: Query that causes full table scan
- [ ] Run: EXPLAIN PLAN - verify table scan
- [ ] Identify: Missing index
- [ ] Create: Index to prevent table scan
- [ ] Run: EXPLAIN PLAN - verify index used
- [ ] Measure: Performance improvement
- [ ] Document: Table scan elimination

### Query Performance Testing
- [ ] Create: Test suite với 10 different queries
- [ ] Run: All queries và measure execution time
- [ ] Identify: 3 slowest queries
- [ ] Optimize: 3 slowest queries
- [ ] Measure: Performance improvement cho mỗi query
- [ ] Set: Performance targets (p95 < 100ms)
- [ ] Verify: All queries meet targets
- [ ] Document: Performance test results

---

## Spring Boot + DB TODOs

### Task 1: HikariCP Configuration
- [ ] Create: Spring Boot project với database
- [ ] Add: HikariCP dependency (default trong Spring Boot)
- [ ] Configure: Connection pool size (calculate based on formula)
- [ ] Configure: Minimum idle connections
- [ ] Configure: Maximum pool size
- [ ] Configure: Connection timeout
- [ ] Configure: Idle timeout
- [ ] Configure: Max lifetime
- [ ] Add: Connection pool monitoring (metrics)
- [ ] Test: Under load, verify pool không exhausted
- [ ] Document: HikariCP configuration

### Task 2: JPA/Hibernate Optimization
- [ ] Setup: Spring Data JPA với Hibernate
- [ ] Configure: HikariCP connection pool
- [ ] Configure: Hibernate batch size
- [ ] Configure: Hibernate fetch size
- [ ] Configure: Hibernate second-level cache (optional)
- [ ] Write: Entity classes với proper annotations
- [ ] Write: Repository với custom queries
- [ ] Test: Query performance
- [ ] Monitor: Hibernate SQL logs (verify no N+1 queries)
- [ ] Optimize: N+1 queries (if found)
- [ ] Document: JPA optimization

### Task 3: Query Performance Monitoring
- [ ] Add: Micrometer metrics
- [ ] Expose: Database query metrics
- [ ] Monitor: Query execution time
- [ ] Monitor: Connection pool usage
- [ ] Monitor: Slow queries (log queries > 1 second)
- [ ] Setup: Alert cho slow queries
- [ ] Create: Dashboard với key metrics
- [ ] Document: Monitoring setup

### Task 4: Prepared Statements
- [ ] Verify: Spring JPA uses prepared statements (check logs)
- [ ] Write: Native query với prepared statement
- [ ] Test: SQL injection prevention
- [ ] Measure: Performance vs regular statements
- [ ] Document: Prepared statement usage

### Task 5: Transaction Management
- [ ] Implement: Service method với @Transactional
- [ ] Test: Transaction rollback on exception
- [ ] Configure: Transaction isolation level
- [ ] Test: Different isolation levels (READ COMMITTED, REPEATABLE READ)
- [ ] Measure: Performance impact của isolation levels
- [ ] Document: Transaction configuration

### Task 6: Database Migration
- [ ] Setup: Flyway hoặc Liquibase
- [ ] Create: Initial schema migration
- [ ] Create: Index migration
- [ ] Create: Data migration (if needed)
- [ ] Test: Migration rollback
- [ ] Document: Migration strategy

### Task 7: Connection Leak Detection
- [ ] Configure: HikariCP leak detection
- [ ] Set: Leak detection threshold (10 seconds)
- [ ] Test: Simulate connection leak (don't close connection)
- [ ] Verify: Leak detection logs warning
- [ ] Fix: Connection leak
- [ ] Document: Leak detection setup

### Task 8: Read-Write Splitting (Optional)
- [ ] Setup: MySQL master-slave replication
- [ ] Configure: Spring Boot với multiple data sources
- [ ] Implement: Read queries go to slave
- [ ] Implement: Write queries go to master
- [ ] Test: Read từ slave
- [ ] Test: Write to master
- [ ] Handle: Replication lag (read-after-write consistency)
- [ ] Document: Read-write splitting implementation

---

## Scaling & Replication TODOs

### Replication Setup Exercise
- [ ] Research: MySQL master-slave replication setup
- [ ] Setup: MySQL master server
- [ ] Setup: MySQL slave server
- [ ] Configure: Replication
- [ ] Test: Write to master, verify replication to slave
- [ ] Measure: Replication lag
- [ ] Test: Read from slave
- [ ] Document: Replication setup

### Read Replica Design
- [ ] Design: Read replica architecture cho Payment System
- [ ] Calculate: Number of read replicas needed (based on read load)
- [ ] Design: Read routing strategy (round-robin? least lag?)
- [ ] Design: Failover strategy (if replica fails)
- [ ] Design: Read-after-write consistency handling
- [ ] Document: Read replica design

### Partitioning Design
- [ ] Design: Partitioning strategy cho Transactions table
- [ ] Requirement: 100M transactions, query by date range
- [ ] Choose: Range partitioning by date
- [ ] Design: Partition key (created_at)
- [ ] Design: Partition pruning strategy
- [ ] Consider: Cross-partition queries impact
- [ ] Document: Partitioning design

### Database Sharding Design (Basic)
- [ ] Read: Database sharding concepts
- [ ] Design: Sharding strategy cho Users table (10M users)
- [ ] Choose: Shard key (user_id)
- [ ] Choose: Sharding algorithm (hash-based)
- [ ] Design: Shard routing logic
- [ ] Design: Cross-shard query handling
- [ ] Document: Sharding design (high-level)

---

## Consistency & Transaction TODOs

### Transaction Isolation Analysis
- [ ] Test: READ UNCOMMITTED isolation level
- [ ] Simulate: Dirty read scenario
- [ ] Verify: Dirty read occurs
- [ ] Test: READ COMMITTED isolation level
- [ ] Simulate: Non-repeatable read scenario
- [ ] Verify: Non-repeatable read occurs
- [ ] Test: REPEATABLE READ isolation level
- [ ] Simulate: Phantom read scenario
- [ ] Verify: Phantom read occurs (or not?)
- [ ] Test: SERIALIZABLE isolation level
- [ ] Measure: Performance impact của each level
- [ ] Document: Isolation level analysis

### Deadlock Analysis
- [ ] Simulate: Deadlock scenario
- [ ] Create: Two transactions that deadlock
- [ ] Verify: Deadlock occurs
- [ ] Configure: Deadlock detection
- [ ] Test: Deadlock resolution
- [ ] Analyze: How to prevent deadlocks?
- [ ] Document: Deadlock prevention strategies

### Optimistic vs Pessimistic Locking
- [ ] Implement: Optimistic locking (version field)
- [ ] Test: Concurrent updates với optimistic locking
- [ ] Verify: OptimisticLockException on conflict
- [ ] Implement: Pessimistic locking (@Lock)
- [ ] Test: Concurrent updates với pessimistic locking
- [ ] Verify: Second transaction waits
- [ ] Compare: Performance của optimistic vs pessimistic
- [ ] Document: When to use which?

### Data Consistency Scenarios
- [ ] Scenario: Update balance - deduct và add
- [ ] Implement: Transaction để ensure atomicity
- [ ] Test: Rollback on failure
- [ ] Scenario: Read balance while updating
- [ ] Test: Consistency với different isolation levels
- [ ] Document: Consistency guarantees

### ACID Properties Verification
- [ ] Test: Atomicity - transaction rollback
- [ ] Test: Consistency - constraints enforcement
- [ ] Test: Isolation - concurrent transactions
- [ ] Test: Durability - data persistence after commit
- [ ] Document: ACID verification results

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Schema design exercises hoàn thành?
- [ ] Review: Tất cả SQL optimization exercises hoàn thành?
- [ ] Review: Tất cả Spring Boot tasks hoàn thành?
- [ ] Review: Tất cả Scaling tasks hoàn thành?
- [ ] Rate: Database schema design skills (1-10)
- [ ] Rate: SQL optimization skills (1-10)
- [ ] Rate: Query performance tuning (1-10)
- [ ] Rate: Understanding của transactions (1-10)
- [ ] Identify: 3 database concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Schema Review
- [ ] Review: Wallet System schema
- [ ] Check: Normalization appropriate?
- [ ] Check: Indexes optimal?
- [ ] Check: Data types correct?
- [ ] Identify: 3 potential improvements
- [ ] Propose: Schema optimizations
- [ ] Compare: Schema vs industry best practices
- [ ] Document: Schema review findings

### Query Performance Review
- [ ] Review: All optimized queries
- [ ] Verify: All queries use indexes
- [ ] Verify: No full table scans
- [ ] Verify: Performance targets met
- [ ] Identify: 3 queries that could be improved further
- [ ] Document: Query performance review

### Code Review
- [ ] Review: Spring Boot database code
- [ ] Check: Connection pool configuration optimal?
- [ ] Check: No N+1 queries?
- [ ] Check: Transaction boundaries correct?
- [ ] Check: Error handling appropriate?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Knowledge Check
- [ ] Explain: B-tree index - how it works (viết 1 paragraph, không xem notes)
- [ ] Explain: Composite index - column order matters (viết explanation, không xem notes)
- [ ] Explain: Query execution plan - how to read (viết guide, không xem notes)
- [ ] Explain: Transaction isolation levels - differences (viết comparison, không xem notes)
- [ ] Explain: Read replica - consistency trade-offs (viết analysis, không xem notes)
- [ ] Solve: Query SELECT * FROM orders WHERE user_id = ? AND status = ? - design index
- [ ] Solve: Calculate connection pool size for 4-core server, 1 database
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 database concepts còn confuse
- [ ] Write: 1 mistake đã làm và lesson learned
- [ ] Write: Confidence level cho Week 4 (1-10)
- [ ] Compare: Week 2 vs Week 3 progress
- [ ] Plan: Preparation cho Week 4 (Database Sharding)
- [ ] Set: Goals cho Week 4
- [ ] Document: Week 3 reflection (500 words)

### Mentor Questions (Answer these - be critical)
- [ ] Q1: Bạn có 1 query chạy 5 giây. Làm sao bạn optimize? (viết step-by-step process)
- [ ] Q2: Composite index (user_id, status, created_at) - query WHERE status = ? AND created_at > ? (không có user_id) - index có được dùng không? Tại sao? (viết analysis)
- [ ] Q3: Read replica có replication lag 2 giây. User vừa update balance, ngay lập tức query balance từ replica - có thấy update không? Làm sao fix? (viết solution)
- [ ] Q4: Connection pool size = 100, nhưng có 200 concurrent requests. Chuyện gì xảy ra? Làm sao fix? (viết analysis)
- [ ] Q5: Transaction isolation level = SERIALIZABLE - có phải luôn tốt nhất không? Tại sao không? (viết trade-off analysis)
- [ ] Q6: Bạn có bảng 100M records, query WHERE created_at > ? - làm sao optimize? (viết optimization strategy)
- [ ] Review: Answers có đủ depth và practical không?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Schema & Modeling TODOs: ✅ Complete với diagrams
- [ ] Tất cả SQL Optimization TODOs: ✅ Complete với performance measurements
- [ ] Tất cả Spring Boot + DB TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Scaling & Replication TODOs: ✅ Complete
- [ ] Tất cả Consistency & Transaction TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Schema diagrams saved: ✅ Yes
- [ ] Query performance benchmarks saved: ✅ Yes
- [ ] Ready for Week 4: ✅ Yes

---

> **Mentor Final Check**: Database performance là foundation. Nếu bạn không thể optimize queries, design schemas, và tune databases, bạn không thể build scalable systems. Hãy honest: Bạn có thể identify và fix slow queries chưa? Bạn có thể design database schema cho 100M+ records chưa? Nếu chưa, làm lại. Production systems không chấp nhận "query chạy được là OK".
