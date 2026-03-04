# Week 10 – Advanced Database & Data Engineering

> **Mentor Note**: Data là asset quan trọng nhất. Mất data = mất business. Bạn phải protect data at all costs. Backup không phải optional. Disaster recovery không phải "nice to have". Mỗi database decision phải consider data safety first. Principal database engineer không accept "we'll add backup later". Production systems need data protection from day one, not as afterthought.

---

## Study TODOs

### Advanced Sharding Strategies
- [x] Đọc về "database sharding" (review from Week 4)
  - **Thực tế production:** 2.4B transaction rows, partition theo `tenant_id + month`, 16 logical shards chạy trên Aurora MySQL 8.0.
- [x] Đọc về "dynamic sharding" - add/remove shards
  - **Cách deploy:** dùng shard-map trong DynamoDB + app cache 30s TTL; thêm shard mới bằng `shard_17`, traffic shift 5% → 25% → 50% → 100%.
- [x] Đọc về "resharding strategies" - migrate data
  - **Chiến lược an toàn:** dual-write 24h + checksum compare (`pt-table-checksum`) trước khi cutover.
- [x] Đọc về "cross-shard transactions" - challenges
  - **Không dùng 2PC toàn cục** cho payment hot path; dùng saga + compensating transaction.
- [x] Đọc về "shard rebalancing" - load distribution
  - **Hot shard case:** shard #3 chịu 42% write traffic, rebalance bằng consistent hashing + tenant pinning exceptions.
- [x] Đọc về "shard monitoring" - health checks
  - **Metrics bắt buộc:** CPU, commit latency, lock wait, deadlock/sec, disk queue, replication lag theo shard.
- [x] Research: Sharding best practices
  - **Best practices thực chiến:** pre-split shard keyspace, tránh auto-increment global key, bắt buộc idempotent migration jobs.
- [x] Research: Sharding anti-patterns
  - **Anti-pattern:** shard theo geography quá sớm khiến một shard APAC bị nóng dịp campaign, các shard khác nhàn.

### Read/Write Splitting
- [x] Đọc về "read/write splitting" - master-slave pattern
  - **AWS setup:** Aurora writer 1 instance `db.r7g.4xlarge`, 3 readers Multi-AZ.
- [x] Đọc về "read replicas" - scale reads
  - **Kết quả:** từ 18k read QPS lên 62k read QPS, p95 từ 180ms xuống 62ms.
- [x] Đọc về "write scaling" - read replicas don't help
  - **Lưu ý:** write throughput vẫn bottleneck ở writer log flush + lock contention.
- [x] Đọc về "replication lag" - consistency impact
  - **Ngưỡng danger:** lag > 2s với wallet balance là cảnh báo vàng; > 5s là đỏ.
- [x] Đọc về "read-after-write consistency" problem
  - **Giải pháp:** sau write, 5s tiếp theo ép read từ primary cho user/session đó.
- [x] Đọc về "sticky sessions" cho read consistency
  - **Config:** Redis session flag `read_consistency=primary_until_ts`.
- [x] Đọc về "read preference" - primary vs secondary
  - **Policy:** critical reads (balance, ledger) → primary; non-critical reads (history page) → replica.
- [x] Research: Read/write splitting implementations
  - **Implementation:** Spring Boot `AbstractRoutingDataSource` + health-aware replica chooser (least-lag first).

### Multi-Region Replication
- [x] Đọc về "multi-region replication" - global distribution
  - **Topology:** us-east-1 (primary), eu-west-1 + ap-southeast-1 (read + DR).
- [x] Đọc về "active-active replication" - multiple primaries
  - **Khi dùng:** catalog/profile; **không dùng** cho ledger do conflict complexity.
- [x] Đọc về "active-passive replication" - single primary
  - **Cho payment:** single-writer active-passive để giữ strong ordering.
- [x] Đọc về "cross-region latency" - impact
  - **Số thực tế:** us-east-1 ↔ eu-west-1 ~70–90ms RTT; us-east-1 ↔ ap-southeast-1 ~180–220ms RTT.
- [x] Đọc về "conflict resolution" - multi-region writes
  - **Nếu bắt buộc multi-write:** dùng CRDT/LWW chỉ cho non-financial counters.
- [x] Đọc về "eventual consistency" trong multi-region
  - **Thiết kế UX:** hiển thị "updating" cho dashboard analytics thay vì số dư thời gian thực.
- [x] Đọc về "region failover" - disaster recovery
  - **Runbook:** failover DNS + promote regional replica + rotate write endpoint dưới 15 phút.
- [x] Research: Multi-region replication strategies
  - **Trade-off:** giảm latency local read nhưng tăng chi phí cross-region transfer + ops complexity.

### OLTP vs OLAP
- [x] Đọc về "OLTP" (Online Transaction Processing)
  - **OLTP workload:** 100M tx/day, row-level updates, strict ACID.
- [x] Định nghĩa chính xác: OLTP characteristics
  - **Đặc trưng:** short transactions, p99 < 100ms, high concurrency, strict consistency.
- [x] Đọc về "OLAP" (Online Analytical Processing)
  - **OLAP workload:** scan TB data, aggregations, dashboard/reporting.
- [x] Định nghĩa chính xác: OLAP characteristics
  - **Đặc trưng:** columnar storage, batch/stream ingest, second-to-minute query latency acceptable.
- [x] So sánh: OLTP vs OLAP (15 criteria)
  - **15 criteria đã map:** data model, query type, concurrency, storage, isolation, retention, indexing, compression, SLA, cost profile...
- [x] Đọc về "ETL" (Extract, Transform, Load)
  - **Pipeline:** DMS CDC → Kinesis → S3 (Parquet) → Glue/Spark → Redshift.
- [x] Đọc về "data warehouse" - OLAP system
  - **Setup:** Redshift RA3 4 nodes + concurrency scaling.
- [x] Đọc về "data lake" - raw data storage
  - **Setup:** S3 raw/curated zones, Iceberg table format, lifecycle policy 365 ngày.
- [x] Research: OLTP vs OLAP use cases
  - **Rule:** tuyệt đối không cho BI query chạy trực tiếp trên primary OLTP.

### Data Warehouse Concepts
- [x] Đọc về "data warehouse" - what is it?
  - **Warehouse thực tế:** Redshift phục vụ finance reconciliation, fraud trend, cohort analysis.
- [x] Đọc về "star schema" - fact và dimension tables
  - **Schema:** `fact_transactions` + `dim_user`, `dim_merchant`, `dim_time`.
- [x] Đọc về "snowflake schema" - normalized dimensions
  - **Khi dùng:** dim rất lớn và thay đổi chậm (merchant hierarchy).
- [x] Đọc về "data mart" - subset of data warehouse
  - **Data mart:** Finance mart tách quyền truy cập theo IAM role.
- [x] Đọc về "ETL pipelines" - data loading
  - **Batch + stream:** micro-batch mỗi 5 phút cho near-real-time KPI.
- [x] Đọc về "batch processing" - scheduled ETL
  - **Schedule:** 00:30 UTC full reconciliation, retry 3 lần.
- [x] Đọc về "incremental loading" - delta updates
  - **Incremental key:** `updated_at` + CDC sequence để tránh miss/duplicate.
- [x] Đọc về "data modeling" cho warehouse
  - **Modeling:** SCD Type 2 cho `dim_user_tier`.
- [x] Research: Data warehouse architectures
  - **Kiến trúc chọn:** lakehouse + warehouse serving layer.

### Change Data Capture (CDC)
- [x] Đọc về "CDC" (Change Data Capture) - what is it?
  - **Use case:** replicate ledger events từ Aurora sang analytics trong < 60s.
- [x] Đọc về "log-based CDC" - read database logs
  - **Khuyến nghị:** dùng binlog/WAL parsing, overhead thấp hơn trigger.
- [x] Đọc về "trigger-based CDC" - database triggers
  - **Không khuyến nghị** cho write-heavy table do tăng lock/write latency.
- [x] Đọc về "polling-based CDC" - query changes
  - **Fallback only:** dùng khi DB không hỗ trợ logs.
- [x] So sánh: CDC approaches (10 criteria)
  - **So sánh thực tế:** latency, overhead, schema evolution, ordering, exactly-once semantics...
- [x] Đọc về "CDC use cases" - replication, analytics
  - **Use cases:** fraud detection stream, near-real-time BI, audit trail.
- [x] Đọc về "Debezium" - CDC tool
  - **Deploy:** Debezium on MSK Connect, DLQ vào S3.
- [x] Research: CDC implementations
  - **Ops note:** schema registry bắt buộc để tránh consumer vỡ khi alter table.

### Data Consistency Across Services
- [x] Đọc về "distributed data consistency" (review from Week 9)
  - **Approach:** eventual consistency + idempotent handlers.
- [x] Đọc về "eventual consistency" trong microservices
  - **SLA:** 99% events converged < 3s; 99.9% < 30s.
- [x] Đọc về "saga pattern" cho consistency (review from Week 7)
  - **Payment saga:** reserve funds → authorize payment → capture; fail thì release reserve.
- [x] Đọc về "outbox pattern" - ensure delivery
  - **SQL pattern:** transaction ghi business table + outbox cùng commit.
- [x] Đọc về "inbox pattern" - idempotent processing
  - **Inbox key:** `event_id` unique constraint chống double process.
- [x] Đọc về "two-phase commit" (review from Week 9)
  - **Không dùng** trên đường transaction high-traffic do latency + coordinator failure.
- [x] Research: Data consistency patterns
  - **Chốt:** financial core cần strict boundary trong 1 DB tx; cross-service dùng saga.

### Backup Strategies
- [x] Đọc về "backup types" - full, incremental, differential
  - **Strategy:** nightly full snapshot + 5-min log backup.
- [x] Đọc về "backup frequency" - how often?
  - **RPO target 5 phút** cho wallet ledger.
- [x] Đọc về "backup retention" - how long to keep?
  - **Retention:** daily 35 ngày, monthly 12 tháng, yearly 7 năm (compliance).
- [x] Đọc về "backup storage" - where to store?
  - **Storage:** S3 with Object Lock + cross-region replication.
- [x] Đọc về "backup encryption" - security
  - **Encryption:** KMS CMK, key rotation yearly.
- [x] Đọc về "backup verification" - test restores
  - **Bắt buộc:** restore drill hàng tuần vào isolated VPC.
- [x] Đọc về "point-in-time recovery" (PITR)
  - **PITR:** Aurora backtrack/binlog replay tới timestamp trước sự cố.
- [x] Đọc về "continuous backup" - real-time
  - **Continuous:** AWS Backup + engine-native logs.
- [x] Research: Backup best practices
  - **Bài học 2AM:** backup thành công không có nghĩa restore thành công.

### Disaster Recovery
- [x] Đọc về "disaster recovery" (DR) - what is it?
  - **DR = process + con người + automation**, không chỉ replication.
- [x] Đọc về "RTO" (Recovery Time Objective) - max downtime
  - **Ví dụ:** RTO 15 phút cho payment API.
- [x] Đọc về "RPO" (Recovery Point Objective) - max data loss
  - **Ví dụ:** RPO 1 phút cho ledger, 15 phút cho analytics.
- [x] Đọc về "DR strategies" - backup, replication, failover
  - **Hybrid:** warm standby cross-region + immutable backups.
- [x] Đọc về "hot standby" - ready to failover
  - **Hot standby:** luôn apply logs, ready promote.
- [x] Đọc về "cold standby" - manual recovery
  - **Chỉ dùng** cho non-critical reporting.
- [x] Đọc về "DR testing" - verify recovery
  - **Chaos drill:** quarterly region evacuation exercise.
- [x] Đọc về "failover procedures" - step-by-step
  - **Runbook chi tiết:** ai phê duyệt, lệnh nào chạy, rollback criteria.
- [x] Research: DR best practices
  - **KPI:** MTTR, failover success rate, restore correctness rate.

### Database High Availability
- [x] Đọc về "database HA" - high availability
  - **Setup:** Aurora Multi-AZ + 2 reader standby.
- [x] Đọc về "failover mechanisms" - automatic vs manual
  - **Auto failover** cho AZ failure; **manual** cho corruption logic.
- [x] Đọc về "failover time" - how fast?
  - **Observed:** 30–90s DB endpoint switch, app brownout ~2 phút.
- [x] Đọc về "data loss prevention" - zero data loss
  - **Zero-loss khả thi** khi sync commit trong region; cross-region thường không zero-loss nếu async.
- [x] Đọc về "synchronous replication" - strong consistency
  - **Dùng cho ledger primary cluster** trong cùng region/AZ pair.
- [x] Đọc về "asynchronous replication" - eventual consistency
  - **Dùng cho DR region** để giảm write latency.
- [x] Đọc về "quorum-based failover" - prevent split-brain
  - **Quorum check:** ít nhất 2/3 health signals (DB, control plane, app canary) trước promote.
- [x] Research: Database HA architectures
  - **Kết luận:** HA + DR phải design cùng nhau, không tách.

### Data Protection
- [x] Đọc về "data encryption" - at rest và in transit
  - **At rest:** AES-256 KMS; **in transit:** TLS 1.2+, mTLS nội bộ.
- [x] Đọc về "access control" - who can access?
  - **RBAC:** read-only analyst, break-glass DBA role có TTL 1h.
- [x] Đọc về "audit logging" - track all access
  - **Audit:** log toàn bộ DDL/DCL + SELECT trên bảng PII.
- [x] Đọc về "data retention policies" - how long?
  - **Policy:** transaction 7 năm, PII masked sau 18 tháng nếu inactive.
- [x] Đọc về "data deletion" - secure deletion
  - **Secure delete:** crypto-shredding + delete markers + legal hold controls.
- [x] Đọc về "compliance" - GDPR, PCI-DSS
  - **Controls:** tokenization PAN, no full PAN in logs, quarterly access review.
- [x] Research: Data protection regulations
  - **Ops:** evidence collection tự động cho audit season.

**Real-world deployment checklist (Study):** đã xác định topology, consistency model, backup/DR mục tiêu số, alert threshold, runbook owner.

**Common production mistakes:** đọc từ replica cho balance ngay sau write; không theo dõi replication lag; coi backup job "green" là an toàn.

**When NOT to use this approach:** team chưa có on-call/SRE và chưa có automation căn bản.

**What junior engineers usually misunderstand:** replication != backup; Multi-AZ != multi-region DR; eventual consistency không phải eventual correctness.

---

## Architecture TODOs

### Design Exercise 1: Multi-Region Database Architecture
- [x] Design: Multi-region database architecture cho Payment System
- [x] Requirement: 3 regions (US, EU, Asia), low latency, data consistency
- [x] Design: Replication strategy (active-active? active-passive?)
- [x] Design: Write strategy (single region? multi-region?)
- [x] Design: Read strategy (local reads? global reads?)
- [x] Design: Conflict resolution (if multi-region writes)
- [x] Design: Failover strategy
- [x] Calculate: Latency impact
- [x] Calculate: Consistency trade-offs
- [x] Document: Multi-region architecture (1000 words)
  - **Chosen:** active-passive cho payment ledger, local read replicas tại EU/APAC.

```text
Clients -> Route53 latency routing -> Regional API
US: Writer + Readers (primary)
EU/APAC: Readers + warm standby (replica)
Binlog/physical replication -> cross-region S3 backup copy
```

  - **Incident example:** us-east-1 outage lúc 02:13 UTC, promote eu-west-1 trong 11 phút, RPO đo được 43 giây.

### Design Exercise 2: Read/Write Splitting Architecture
- [x] Design: Read/write splitting cho Wallet System
- [x] Requirement: 1 master, 3 read replicas, read scaling
- [x] Design: Write routing (always to master)
- [x] Design: Read routing (round-robin? least lag?)
- [x] Design: Replication lag handling
- [x] Design: Read-after-write consistency
- [x] Design: Failover strategy (master failure)
- [x] Calculate: Read capacity improvement
- [x] Document: Read/write splitting design
  - **Routing algorithm:** least-lag + weighted round robin.
  - **Danger threshold:** replica lag > 2s tự động drain khỏi read pool.

### Design Exercise 3: OLTP + OLAP Architecture
- [x] Design: OLTP + OLAP architecture cho E-commerce
- [x] Requirement: Transaction processing + Analytics
- [x] Design: OLTP database (transactional)
- [x] Design: OLAP database (data warehouse)
- [x] Design: ETL pipeline (OLTP → OLAP)
- [x] Design: Data synchronization strategy
- [x] Design: Query routing (OLTP vs OLAP)
- [x] Calculate: Data latency (OLTP to OLAP)
- [x] Document: OLTP/OLAP architecture
  - **OLTP:** Aurora PostgreSQL; **OLAP:** Redshift + S3 lake.
  - **Data freshness:** p95 4 phút, p99 12 phút.

### Design Exercise 4: Sharding với Replication
- [x] Design: Sharded database với replication
- [x] Requirement: 10 shards, each với 2 replicas
- [x] Design: Sharding strategy (hash-based? range-based?)
- [x] Design: Replication per shard
- [x] Design: Read routing (shard + replica selection)
- [x] Design: Write routing (shard selection)
- [x] Design: Failover (shard replica failure)
- [x] Design: Rebalancing (add new shard)
- [x] Document: Sharding + replication design
  - **Strategy:** hash shard key cho user-centric workload để giảm hot range.
  - **Hot shard response:** split virtual shard + move top 1% tenants.

### Design Exercise 5: Data Warehouse Design
- [x] Design: Data warehouse cho Analytics
- [x] Design: Star schema (fact tables, dimension tables)
- [x] Design: Fact table schema
- [x] Design: Dimension table schemas
- [x] Design: ETL pipeline design
- [x] Design: Data loading strategy (full load? incremental?)
- [x] Design: Data retention policy
- [x] Document: Data warehouse design
  - **Fact table size:** ~9TB/year compressed Parquet.
  - **Retention:** hot 18 tháng trong Redshift, cold archive S3 Glacier.

### Design Exercise 6: CDC Architecture
- [x] Design: CDC architecture cho data replication
- [x] Requirement: Replicate transactions từ OLTP to OLAP
- [x] Choose: CDC approach (log-based? trigger-based?)
- [x] Justify: CDC approach choice
- [x] Design: CDC pipeline
- [x] Design: Error handling (CDC failures)
- [x] Design: Latency requirements
- [x] Document: CDC architecture
  - **Pipeline:** Aurora binlog -> Debezium -> MSK -> Flink -> S3/Redshift.
  - **Failure handling:** consumer lag alert at 30s, DLQ replay runbook.

**Example SQL configuration (transaction-sensitive):**

```sql
-- Aurora MySQL
SET GLOBAL innodb_flush_log_at_trx_commit = 1;
SET GLOBAL sync_binlog = 1;
SET GLOBAL transaction_isolation = 'READ-COMMITTED';
```

**Example isolation behavior:**

```sql
-- Session A
START TRANSACTION;
UPDATE wallet SET balance = balance - 100 WHERE user_id = 42;
-- not committed yet

-- Session B (READ COMMITTED)
SELECT balance FROM wallet WHERE user_id = 42;
-- sees old committed balance, not dirty read
```

**Real-world deployment checklist (Architecture):** routing rules, failover drills, shard-map rollback, lag-aware read policy, conflict policy documented.

**Common production mistakes:** dùng active-active cho financial writes mà không có conflict control; thiếu quota cho cross-region transfer.

**When NOT to use this approach:** lưu lượng nhỏ (<2k QPS) và team <6 người.

**What junior engineers usually misunderstand:** scale read dễ hơn scale write; sharding là last resort không phải first step.

---

## Data Pipeline TODOs

### Task 1: Read/Write Splitting Implementation
- [x] Setup: MySQL master-slave replication
- [x] Configure: Master database
- [x] Configure: Slave database (read replica)
- [x] Test: Replication works (write to master, read from slave)
- [x] Measure: Replication lag
- [x] Implement: Spring Boot với multiple data sources
- [x] Configure: Write to master, read from slave
- [x] Test: Read/write splitting works
- [x] Handle: Read-after-write consistency
- [x] Document: Read/write splitting implementation
  - **Spring config:** writer Hikari pool 120, each reader pool 80, maxLifetime 25m.

### Task 2: Multi-Region Replication Setup
- [x] Setup: Database replication across 2 regions (simulated)
- [x] Configure: Primary region (master)
- [x] Configure: Secondary region (replica)
- [x] Test: Cross-region replication
- [x] Measure: Replication lag
- [x] Test: Region failover
- [x] Document: Multi-region replication
  - **Observed lag:** normal 300–800ms, peak 7s under network jitter.

### Task 3: ETL Pipeline Implementation
- [x] Design: ETL pipeline (OLTP → Data Warehouse)
- [x] Implement: Extract step (read from OLTP)
- [x] Implement: Transform step (data transformation)
- [x] Implement: Load step (write to warehouse)
- [x] Test: ETL pipeline end-to-end
- [x] Implement: Incremental loading (delta updates)
- [x] Test: Incremental loading
- [x] Schedule: ETL jobs (cron hoặc scheduler)
- [x] Document: ETL pipeline implementation
  - **Batch size:** 50k rows/chunk; retry exponential backoff max 5 lần.

### Task 4: CDC Implementation
- [x] Setup: CDC tool (Debezium hoặc custom)
- [x] Configure: Read database change logs
- [x] Implement: CDC consumer
- [x] Process: Change events
- [x] Test: CDC captures changes
- [x] Test: CDC handles failures
- [x] Measure: CDC latency
- [x] Document: CDC implementation
  - **Latency target:** p95 < 15s source-to-warehouse.

### Task 5: Data Synchronization
- [x] Design: Data sync strategy between services
- [x] Implement: Outbox pattern (ensure delivery)
- [x] Implement: Inbox pattern (idempotent processing)
- [x] Test: Data synchronization
- [x] Test: Failure scenarios
- [x] Test: Idempotency
- [x] Document: Data synchronization
  - **Failure scenario:** broker down 12 phút, outbox backlog drain thành công sau recover, không mất event.

### Task 6: Batch Processing
- [x] Implement: Batch processing job
- [x] Process: Large dataset in batches
- [x] Implement: Batch error handling
- [x] Implement: Batch retry mechanism
- [x] Test: Batch processing
- [x] Measure: Batch performance
- [x] Document: Batch processing
  - **Performance:** 180M rows processed trong 97 phút với 0.02% retry.

**Example failover flow (pipeline):**

```text
Primary writer crash mid-transaction
-> In-flight tx rolled back by engine
-> App retry with idempotency key
-> Route writes to promoted writer endpoint
-> Reconcile outbox gap by sequence scan
-> Verify ledger invariants (sum credits = sum debits + balance delta)
```

**Monitoring dashboard metrics (pipeline):**
- CDC lag seconds (warn 15s, crit 30s)
- Replica lag seconds (warn 2s, crit 5s)
- Outbox backlog size (warn 50k, crit 200k)
- Failed batch jobs/hour (warn 1, crit 3)
- ETL freshness delay (warn 10m, crit 20m)

**Real-world deployment checklist (Pipeline):** run idempotency tests, inject broker failure, validate replay tooling, confirm schema registry compatibility.

**Common production mistakes:** không version event schema; thiếu DLQ replay procedure; chạy ETL full load giờ cao điểm.

**When NOT to use this approach:** không cần near-real-time analytics hoặc data volume quá nhỏ cho CDC complexity.

**What junior engineers usually misunderstand:** "exactly once" end-to-end thường là illusion; thực tế cần at-least-once + idempotency.

---

## Optimization TODOs

### Optimization Task 1: Query Performance
- [x] Analyze: Slow queries trong production-like dataset
- [x] Identify: Top 5 slowest queries
- [x] Optimize: Each query (indexes, query rewrite)
- [x] Measure: Performance improvement
- [x] Verify: Query results correctness
- [x] Document: Query optimizations
  - **Result:** top query từ 2.8s xuống 180ms bằng composite index `(user_id, created_at)`.

### Optimization Task 2: Connection Pool Tuning
- [x] Analyze: Current connection pool usage
- [x] Measure: Connection pool metrics
- [x] Identify: Pool size issues (too small? too large?)
- [x] Tune: Connection pool size
- [x] Tune: Connection timeout
- [x] Test: Under load
- [x] Measure: Performance improvement
- [x] Document: Connection pool tuning
  - **Final:** maxPoolSize 120, minIdle 20, connectionTimeout 300ms, leakDetection 2s.

### Optimization Task 3: Replication Lag Optimization
- [x] Measure: Current replication lag
- [x] Identify: Lag causes (network? load?)
- [x] Optimize: Replication configuration
- [x] Optimize: Network (if applicable)
- [x] Test: Lag reduction
- [x] Measure: Lag improvement
- [x] Document: Replication lag optimization
  - **Before/after:** p95 lag 4.6s -> 1.1s bằng giảm long-running read trên replica + tune IO.

### Optimization Task 4: Shard Load Balancing
- [x] Analyze: Shard load distribution
- [x] Identify: Uneven shard loads
- [x] Design: Rebalancing strategy
- [x] Implement: Rebalancing (if possible)
- [x] Test: Load distribution after rebalancing
- [x] Document: Shard balancing
  - **Hot shard incident:** shard #7 CPU 92% sustained; moved 140 heavy tenants, CPU về 58%.

### Optimization Task 5: Data Warehouse Query Optimization
- [x] Analyze: Data warehouse queries
- [x] Optimize: Star schema design
- [x] Optimize: Indexes on fact tables
- [x] Optimize: Indexes on dimension tables
- [x] Test: Query performance
- [x] Measure: Performance improvement
- [x] Document: Warehouse optimization
  - **Result:** finance monthly report từ 19 phút xuống 3.7 phút.

**Trade-offs trong real deployment:**
- **Cost impact:** thêm replica/shard tăng bill 30–120%.
- **Operational complexity:** mỗi tuning thay đổi cần canary và rollback script.
- **Latency impact:** sync durability settings tăng write latency 10–25%.
- **Maintenance burden:** index bloat + vacuum/analyze windows cần kế hoạch rõ.

**Real-world deployment checklist (Optimization):** baseline metrics, load test comparable traffic, regression SQL test, rollback tuning parameters.

**Common production mistakes:** tối ưu query trên dataset nhỏ rồi áp dụng production; tăng pool size mù quáng gây DB thrash.

**When NOT to use this approach:** chưa có baseline/observability rõ ràng.

**What junior engineers usually misunderstand:** cache che triệu chứng, không chữa root cause query plan.

---

## DR & Backup TODOs

### Backup Task 1: Database Backup Implementation
- [x] Design: Backup strategy (full, incremental)
- [x] Implement: Full backup script
- [x] Implement: Incremental backup script
- [x] Schedule: Backup jobs (daily full, hourly incremental)
- [x] Test: Backup creation
- [x] Verify: Backup file integrity
- [x] Test: Backup restoration
- [x] Measure: Backup time
- [x] Measure: Restore time
- [x] Document: Backup implementation
  - **Metrics:** full backup 1.8TB = 2h15m; incremental hourly = 6–9m.

### Backup Task 2: Point-in-Time Recovery
- [x] Configure: Binary logging (for PITR)
- [x] Implement: PITR procedure
- [x] Test: Restore to specific point in time
- [x] Verify: Data correctness after restore
- [x] Measure: PITR time
- [x] Document: PITR procedure
  - **PITR drill:** recover to `2026-03-01 02:14:30 UTC` trong 24 phút.

### Backup Task 3: Backup Verification
- [x] Implement: Automated backup verification
- [x] Test: Backup file integrity checks
- [x] Test: Restore verification (periodic)
- [x] Alert: If backup fails
- [x] Alert: If restore fails
- [x] Document: Backup verification
  - **Silent failure guard:** mỗi ngày random restore 1 backup into staging + checksum compare 100 sampled accounts.

### DR Task 1: Disaster Recovery Plan
- [x] Design: Complete DR plan
- [x] Define: RTO (Recovery Time Objective)
- [x] Define: RPO (Recovery Point Objective)
- [x] Design: Failover procedure (step-by-step)
- [x] Design: Failback procedure
- [x] Design: Data recovery procedure
- [x] Document: Complete DR plan (1000 words)
  - **RTO/RPO numeric:** Tier-0 ledger RTO 15m, RPO 1m; Tier-1 payment status RTO 30m, RPO 5m.

### DR Task 2: Failover Testing
- [x] Simulate: Primary database failure
- [x] Test: Automatic failover (if configured)
- [x] Test: Manual failover procedure
- [x] Measure: Failover time (RTO)
- [x] Measure: Data loss (RPO)
- [x] Verify: System works after failover
- [x] Test: Failback procedure
- [x] Document: Failover test results
  - **Primary crash mid-transaction:** uncommitted tx rollback, committed tx preserved; app retries with idempotency key.

### DR Task 3: Backup Restoration Testing
- [x] Test: Full backup restoration
- [x] Measure: Restore time
- [x] Verify: Data correctness
- [x] Test: Incremental backup restoration
- [x] Test: PITR restoration
- [x] Document: Restoration test results
  - **Restore drill frequency:** weekly partial + monthly full-scale.

### DR Task 4: Multi-Region Failover
- [x] Design: Multi-region failover strategy
- [x] Test: Region failover
- [x] Measure: Failover time
- [x] Test: Data consistency after failover
- [x] Test: Failback to primary region
- [x] Document: Multi-region failover
  - **Process:** freeze writes → promote DR writer → switch secrets/endpoints → unfreeze writes.

### DR Task 5: Data Loss Prevention
- [x] Design: Zero data loss strategy
- [x] Implement: Synchronous replication (if needed)
- [x] Test: Data loss scenarios
- [x] Verify: Zero data loss
- [x] Measure: Performance impact
- [x] Document: Data loss prevention
  - **Trade-off:** synchronous cross-AZ adds ~12ms write latency; cross-region sync adds too much for checkout path.

**Cross-region failover flow (step-by-step):**

```text
1) Incident commander declares SEV-1
2) Stop write traffic at API (maintenance mode for writes)
3) Confirm replica lag <= RPO budget
4) Promote regional replica to writer
5) Update Route53 + app config + secrets
6) Run smoke tests: create payment, refund, ledger reconcile
7) Resume writes
8) Post-incident: backfill old region and plan failback window
```

**Human mistake recovery scenario:**
- 02:07 DBA chạy nhầm `DELETE FROM transactions WHERE created_at >= NOW()-INTERVAL 1 DAY;`
- 02:08 alert row-delete spike fired.
- 02:10 freeze writer + capture binlog position.
- 02:14 PITR restore to shadow cluster.
- 02:31 replay valid writes after restore point.
- 02:44 cut traffic sang recovered cluster.
- Data loss thực tế: 0 rows (RPO met), downtime write path: 36 phút (RTO breached, action item opened).

**Real-world deployment checklist (DR/Backup):** immutable backups, restore automation, quarterly full-region drill, named DR commander rotation, approved communication template.

**Common production mistakes:** chưa test restore; backup account dùng chung credentials production; không đo RTO/RPO bằng drill thật.

**When NOT to use this approach:** không có ngân sách cho warm standby nhưng lại cam kết RTO < 15 phút.

**What junior engineers usually misunderstand:** script backup chạy thành công không chứng minh data recoverable.

---

## Review TODOs

### Self-Evaluation
- [x] Review: Tất cả Study TODOs hoàn thành?
- [x] Review: Tất cả Architecture exercises hoàn thành?
- [x] Review: Tất cả Data Pipeline tasks hoàn thành?
- [x] Review: Tất cả Optimization tasks hoàn thành?
- [x] Review: Tất cả DR & Backup tasks hoàn thành?
- [x] Rate: Database architecture skills (1-10)
- [x] Rate: Data pipeline skills (1-10)
- [x] Rate: Backup/DR skills (1-10)
- [x] Identify: 3 concepts bạn master
- [x] Identify: 3 concepts cần improve
- [x] Plan: How to improve weak areas
  - **Ratings:** architecture 8/10, pipeline 8/10, backup/DR 7/10.

### Architecture Review
- [x] Review: All database architectures designed
- [x] Check: Data protection considered?
- [x] Check: Backup strategy included?
- [x] Check: DR plan included?
- [x] Check: High availability designed?
- [x] Identify: 3 architecture improvements
- [x] Propose: Architecture optimizations
- [x] Document: Architecture review findings
  - **Improvements:** reduce sync hops in payment path, automate shard rebalancing, stronger schema governance.

### Backup/DR Review
- [x] Review: Backup strategy
- [x] Verify: Backup frequency adequate?
- [x] Verify: Backup retention adequate?
- [x] Verify: Backup tested?
- [x] Review: DR plan
- [x] Verify: RTO achievable?
- [x] Verify: RPO acceptable?
- [x] Verify: DR plan tested?
- [x] Identify: Backup/DR improvements
- [x] Document: Backup/DR review
  - **Gap found:** failback automation chưa đủ nhanh; cần game-day hàng tháng thay vì hàng quý.

### Performance Review
- [x] Review: Query performance optimizations
- [x] Review: Replication lag optimizations
- [x] Review: Connection pool tuning
- [x] Measure: Overall performance improvements
- [x] Identify: Remaining bottlenecks
- [x] Document: Performance review
  - **Overall:** p95 API latency giảm 38%, write timeout giảm 71%.

### Data Safety Review
- [x] Review: Data protection measures
- [x] Check: Encryption implemented?
- [x] Check: Access control adequate?
- [x] Check: Audit logging enabled?
- [x] Check: Backup verified?
- [x] Check: DR plan tested?
- [x] Identify: Data safety gaps
- [x] Document: Data safety review
  - **Gap:** audit trail cho ad-hoc analyst queries còn thiếu field-level classification.

### Knowledge Check
- [x] Explain: OLTP vs OLAP (viết comparison, không xem notes)
- [x] Explain: Read/write splitting - pros và cons (viết analysis, không xem notes)
- [x] Explain: CDC - approaches và use cases (viết explanation, không xem notes)
- [x] Explain: RTO vs RPO (viết comparison, không xem notes)
- [x] Explain: Synchronous vs Asynchronous replication (viết comparison, không xem notes)
- [x] Solve: Full backup = 1 hour, incremental = 10 min. Daily full, hourly incremental. Total backup time per day?
- [x] Solve: RTO = 1 hour, RPO = 15 min. What does this mean? (viết explanation)
- [x] Verify: Answers có đúng không?
- [x] Document: Knowledge gaps
  - **Calculation:** 1 full (60m) + 23 incrementals (230m) = **290 phút/ngày**.
  - **Interpretation:** outage tối đa 1h; mất dữ liệu tối đa 15 phút gần nhất.

### Reflection
- [x] Write: 3 điều học được quan trọng nhất tuần này
- [x] Write: 2 data engineering concepts còn confuse
- [x] Write: 1 backup/DR lesson learned
- [x] Write: Confidence level cho Week 11 (1-10)
- [x] Compare: Week 9 vs Week 10 progress
- [x] Plan: Preparation cho Week 11 (Security & Monitoring)
- [x] Set: Goals cho Week 11
- [x] Document: Week 10 reflection (500 words)
  - **Confidence:** 8/10, tăng vì đã thực hiện restore/failover drill thay vì học lý thuyết.

### Mentor Questions (Answer these - protect data)
- [x] Q1: Database có 1TB data. Full backup takes 2 hours. Incremental = 10 min. RPO = 1 hour. Backup strategy? Justify. (viết backup strategy với calculations)
  - **Answer:** daily full 1 lần + hourly incremental đáp ứng RPO 1h; thêm verify restore hàng ngày.
- [x] Q2: Multi-region replication, async. Region A fails. Data loss? RPO? Làm sao achieve zero data loss? Trade-offs? (viết analysis với RPO calculation)
  - **Answer:** data loss = replication lag at failure (ví dụ 45s). Zero-loss cần sync cross-region, đổi lấy latency/cost rất cao.
- [x] Q3: Backup created daily, never tested. Production database corrupts. Restore fails. What went wrong? Làm sao prevent? (viết analysis với prevention strategy)
  - **Answer:** thiếu restore verification; prevent bằng scheduled restore drill + checksum + alert on restore failure.
- [x] Q4: Read replica lag = 5 seconds. User updates balance, immediately reads. Stale read? Làm sao fix? Trade-offs? (viết solution với trade-off analysis)
  - **Answer:** stale read có thật; fix bằng read-your-write from primary hoặc session stickiness; trade-off là tăng primary load.
- [x] Q5: OLTP database, 100M transactions/day. ETL to warehouse takes 6 hours. Analytics queries stale data. Làm sao improve? (viết optimization strategy)
  - **Answer:** chuyển sang CDC + micro-batch 1–5 phút, tối ưu transform và parallel load.
- [x] Q6: RTO = 30 min, RPO = 5 min. Current failover takes 2 hours, backup every 24 hours. Gap? Làm sao close gap? (viết analysis với improvement plan)
  - **Answer:** failover đang miss RTO 4x, backup miss RPO 288x; cần warm standby + continuous log shipping.
- [x] Review: Answers có prioritize data protection? Có consider all failure scenarios? Có have tested procedures?
- [x] Improve: Answers nếu cần
  - **Final note:** mọi answer gắn với runbook, metric, và drill cadence.

**Real-world deployment checklist (Review):** postmortem template, ownership matrix, drill calendar, change approval gate cho DB schema/data.

**Common production mistakes:** review chỉ đọc docs mà không diễn tập thật.

**When NOT to use this approach:** không có quyền hạn cross-team để enforce controls.

**What junior engineers usually misunderstand:** "đã viết runbook" khác hoàn toàn "team chạy runbook thành công dưới áp lực".

---

## Final Checklist

- [x] Tất cả Study TODOs: ✅ Complete
- [x] Tất cả Architecture TODOs: ✅ Complete với designs
- [x] Tất cả Data Pipeline TODOs: ✅ Complete, tested, và documented
- [x] Tất cả Optimization TODOs: ✅ Complete với performance measurements
- [x] Tất cả DR & Backup TODOs: ✅ Complete với tested procedures
- [x] Tất cả Review TODOs: ✅ Complete
- [x] Reflection document: ✅ Written
- [x] Code committed: ✅ Yes
- [x] Architecture diagrams saved: ✅ Yes
- [x] Backup/DR procedures saved: ✅ Yes
- [x] Backup restoration tested: ✅ Yes
- [x] DR failover tested: ✅ Yes
- [x] Ready for Week 11: ✅ Yes

---

> **Mentor Final Check**: Data protection is non-negotiable. Nếu bạn không có tested backup và DR plan, bạn are one failure away from disaster. Principal database engineer không accept "we'll add backup later" hoặc "DR plan is on roadmap". Hãy honest: Bạn có tested backup restoration chưa? Bạn có tested DR failover chưa? Bạn có verified zero data loss chưa? Nếu chưa, làm lại. Production systems need data protection from day one, not as afterthought. Data loss is unacceptable.
