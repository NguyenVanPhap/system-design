# Week 10 – Advanced Database & Data Engineering

> **Mentor Note**: Data là asset quan trọng nhất. Mất data = mất business. Bạn phải protect data at all costs. Backup không phải optional. Disaster recovery không phải "nice to have". Mỗi database decision phải consider data safety first. Principal database engineer không accept "we'll add backup later". Production systems need data protection from day one, not as afterthought.

---

## Study TODOs

### Advanced Sharding Strategies
- [ ] Đọc về "database sharding" (review from Week 4)
- [ ] Đọc về "dynamic sharding" - add/remove shards
- [ ] Đọc về "resharding strategies" - migrate data
- [ ] Đọc về "cross-shard transactions" - challenges
- [ ] Đọc về "shard rebalancing" - load distribution
- [ ] Đọc về "shard monitoring" - health checks
- [ ] Research: Sharding best practices
- [ ] Research: Sharding anti-patterns

### Read/Write Splitting
- [ ] Đọc về "read/write splitting" - master-slave pattern
- [ ] Đọc về "read replicas" - scale reads
- [ ] Đọc về "write scaling" - read replicas don't help
- [ ] Đọc về "replication lag" - consistency impact
- [ ] Đọc về "read-after-write consistency" problem
- [ ] Đọc về "sticky sessions" cho read consistency
- [ ] Đọc về "read preference" - primary vs secondary
- [ ] Research: Read/write splitting implementations

### Multi-Region Replication
- [ ] Đọc về "multi-region replication" - global distribution
- [ ] Đọc về "active-active replication" - multiple primaries
- [ ] Đọc về "active-passive replication" - single primary
- [ ] Đọc về "cross-region latency" - impact
- [ ] Đọc về "conflict resolution" - multi-region writes
- [ ] Đọc về "eventual consistency" trong multi-region
- [ ] Đọc về "region failover" - disaster recovery
- [ ] Research: Multi-region replication strategies

### OLTP vs OLAP
- [ ] Đọc về "OLTP" (Online Transaction Processing)
- [ ] Định nghĩa chính xác: OLTP characteristics
- [ ] Đọc về "OLAP" (Online Analytical Processing)
- [ ] Định nghĩa chính xác: OLAP characteristics
- [ ] So sánh: OLTP vs OLAP (15 criteria)
- [ ] Đọc về "ETL" (Extract, Transform, Load)
- [ ] Đọc về "data warehouse" - OLAP system
- [ ] Đọc về "data lake" - raw data storage
- [ ] Research: OLTP vs OLAP use cases

### Data Warehouse Concepts
- [ ] Đọc về "data warehouse" - what is it?
- [ ] Đọc về "star schema" - fact và dimension tables
- [ ] Đọc về "snowflake schema" - normalized dimensions
- [ ] Đọc về "data mart" - subset of data warehouse
- [ ] Đọc về "ETL pipelines" - data loading
- [ ] Đọc về "batch processing" - scheduled ETL
- [ ] Đọc về "incremental loading" - delta updates
- [ ] Đọc về "data modeling" cho warehouse
- [ ] Research: Data warehouse architectures

### Change Data Capture (CDC)
- [ ] Đọc về "CDC" (Change Data Capture) - what is it?
- [ ] Đọc về "log-based CDC" - read database logs
- [ ] Đọc về "trigger-based CDC" - database triggers
- [ ] Đọc về "polling-based CDC" - query changes
- [ ] So sánh: CDC approaches (10 criteria)
- [ ] Đọc về "CDC use cases" - replication, analytics
- [ ] Đọc về "Debezium" - CDC tool
- [ ] Research: CDC implementations

### Data Consistency Across Services
- [ ] Đọc về "distributed data consistency" (review from Week 9)
- [ ] Đọc về "eventual consistency" trong microservices
- [ ] Đọc về "saga pattern" cho consistency (review from Week 7)
- [ ] Đọc về "outbox pattern" - ensure delivery
- [ ] Đọc về "inbox pattern" - idempotent processing
- [ ] Đọc về "two-phase commit" (review from Week 9)
- [ ] Research: Data consistency patterns

### Backup Strategies
- [ ] Đọc về "backup types" - full, incremental, differential
- [ ] Đọc về "backup frequency" - how often?
- [ ] Đọc về "backup retention" - how long to keep?
- [ ] Đọc về "backup storage" - where to store?
- [ ] Đọc về "backup encryption" - security
- [ ] Đọc về "backup verification" - test restores
- [ ] Đọc về "point-in-time recovery" (PITR)
- [ ] Đọc về "continuous backup" - real-time
- [ ] Research: Backup best practices

### Disaster Recovery
- [ ] Đọc về "disaster recovery" (DR) - what is it?
- [ ] Đọc về "RTO" (Recovery Time Objective) - max downtime
- [ ] Đọc về "RPO" (Recovery Point Objective) - max data loss
- [ ] Đọc về "DR strategies" - backup, replication, failover
- [ ] Đọc về "hot standby" - ready to failover
- [ ] Đọc về "cold standby" - manual recovery
- [ ] Đọc về "DR testing" - verify recovery
- [ ] Đọc về "failover procedures" - step-by-step
- [ ] Research: DR best practices

### Database High Availability
- [ ] Đọc về "database HA" - high availability
- [ ] Đọc về "failover mechanisms" - automatic vs manual
- [ ] Đọc về "failover time" - how fast?
- [ ] Đọc về "data loss prevention" - zero data loss
- [ ] Đọc về "synchronous replication" - strong consistency
- [ ] Đọc về "asynchronous replication" - eventual consistency
- [ ] Đọc về "quorum-based failover" - prevent split-brain
- [ ] Research: Database HA architectures

### Data Protection
- [ ] Đọc về "data encryption" - at rest và in transit
- [ ] Đọc về "access control" - who can access?
- [ ] Đọc về "audit logging" - track all access
- [ ] Đọc về "data retention policies" - how long?
- [ ] Đọc về "data deletion" - secure deletion
- [ ] Đọc về "compliance" - GDPR, PCI-DSS
- [ ] Research: Data protection regulations

---

## Architecture TODOs

### Design Exercise 1: Multi-Region Database Architecture
- [ ] Design: Multi-region database architecture cho Payment System
- [ ] Requirement: 3 regions (US, EU, Asia), low latency, data consistency
- [ ] Design: Replication strategy (active-active? active-passive?)
- [ ] Design: Write strategy (single region? multi-region?)
- [ ] Design: Read strategy (local reads? global reads?)
- [ ] Design: Conflict resolution (if multi-region writes)
- [ ] Design: Failover strategy
- [ ] Calculate: Latency impact
- [ ] Calculate: Consistency trade-offs
- [ ] Document: Multi-region architecture (1000 words)

### Design Exercise 2: Read/Write Splitting Architecture
- [ ] Design: Read/write splitting cho Wallet System
- [ ] Requirement: 1 master, 3 read replicas, read scaling
- [ ] Design: Write routing (always to master)
- [ ] Design: Read routing (round-robin? least lag?)
- [ ] Design: Replication lag handling
- [ ] Design: Read-after-write consistency
- [ ] Design: Failover strategy (master failure)
- [ ] Calculate: Read capacity improvement
- [ ] Document: Read/write splitting design

### Design Exercise 3: OLTP + OLAP Architecture
- [ ] Design: OLTP + OLAP architecture cho E-commerce
- [ ] Requirement: Transaction processing + Analytics
- [ ] Design: OLTP database (transactional)
- [ ] Design: OLAP database (data warehouse)
- [ ] Design: ETL pipeline (OLTP → OLAP)
- [ ] Design: Data synchronization strategy
- [ ] Design: Query routing (OLTP vs OLAP)
- [ ] Calculate: Data latency (OLTP to OLAP)
- [ ] Document: OLTP/OLAP architecture

### Design Exercise 4: Sharding với Replication
- [ ] Design: Sharded database với replication
- [ ] Requirement: 10 shards, each với 2 replicas
- [ ] Design: Sharding strategy (hash-based? range-based?)
- [ ] Design: Replication per shard
- [ ] Design: Read routing (shard + replica selection)
- [ ] Design: Write routing (shard selection)
- [ ] Design: Failover (shard replica failure)
- [ ] Design: Rebalancing (add new shard)
- [ ] Document: Sharding + replication design

### Design Exercise 5: Data Warehouse Design
- [ ] Design: Data warehouse cho Analytics
- [ ] Design: Star schema (fact tables, dimension tables)
- [ ] Design: Fact table schema
- [ ] Design: Dimension table schemas
- [ ] Design: ETL pipeline design
- [ ] Design: Data loading strategy (full load? incremental?)
- [ ] Design: Data retention policy
- [ ] Document: Data warehouse design

### Design Exercise 6: CDC Architecture
- [ ] Design: CDC architecture cho data replication
- [ ] Requirement: Replicate transactions từ OLTP to OLAP
- [ ] Choose: CDC approach (log-based? trigger-based?)
- [ ] Justify: CDC approach choice
- [ ] Design: CDC pipeline
- [ ] Design: Error handling (CDC failures)
- [ ] Design: Latency requirements
- [ ] Document: CDC architecture

---

## Data Pipeline TODOs

### Task 1: Read/Write Splitting Implementation
- [ ] Setup: MySQL master-slave replication
- [ ] Configure: Master database
- [ ] Configure: Slave database (read replica)
- [ ] Test: Replication works (write to master, read from slave)
- [ ] Measure: Replication lag
- [ ] Implement: Spring Boot với multiple data sources
- [ ] Configure: Write to master, read from slave
- [ ] Test: Read/write splitting works
- [ ] Handle: Read-after-write consistency
- [ ] Document: Read/write splitting implementation

### Task 2: Multi-Region Replication Setup
- [ ] Setup: Database replication across 2 regions (simulated)
- [ ] Configure: Primary region (master)
- [ ] Configure: Secondary region (replica)
- [ ] Test: Cross-region replication
- [ ] Measure: Replication lag
- [ ] Test: Region failover
- [ ] Document: Multi-region replication

### Task 3: ETL Pipeline Implementation
- [ ] Design: ETL pipeline (OLTP → Data Warehouse)
- [ ] Implement: Extract step (read from OLTP)
- [ ] Implement: Transform step (data transformation)
- [ ] Implement: Load step (write to warehouse)
- [ ] Test: ETL pipeline end-to-end
- [ ] Implement: Incremental loading (delta updates)
- [ ] Test: Incremental loading
- [ ] Schedule: ETL jobs (cron hoặc scheduler)
- [ ] Document: ETL pipeline implementation

### Task 4: CDC Implementation
- [ ] Setup: CDC tool (Debezium hoặc custom)
- [ ] Configure: Read database change logs
- [ ] Implement: CDC consumer
- [ ] Process: Change events
- [ ] Test: CDC captures changes
- [ ] Test: CDC handles failures
- [ ] Measure: CDC latency
- [ ] Document: CDC implementation

### Task 5: Data Synchronization
- [ ] Design: Data sync strategy between services
- [ ] Implement: Outbox pattern (ensure delivery)
- [ ] Implement: Inbox pattern (idempotent processing)
- [ ] Test: Data synchronization
- [ ] Test: Failure scenarios
- [ ] Test: Idempotency
- [ ] Document: Data synchronization

### Task 6: Batch Processing
- [ ] Implement: Batch processing job
- [ ] Process: Large dataset in batches
- [ ] Implement: Batch error handling
- [ ] Implement: Batch retry mechanism
- [ ] Test: Batch processing
- [ ] Measure: Batch performance
- [ ] Document: Batch processing

---

## Optimization TODOs

### Optimization Task 1: Query Performance
- [ ] Analyze: Slow queries trong production-like dataset
- [ ] Identify: Top 5 slowest queries
- [ ] Optimize: Each query (indexes, query rewrite)
- [ ] Measure: Performance improvement
- [ ] Verify: Query results correctness
- [ ] Document: Query optimizations

### Optimization Task 2: Connection Pool Tuning
- [ ] Analyze: Current connection pool usage
- [ ] Measure: Connection pool metrics
- [ ] Identify: Pool size issues (too small? too large?)
- [ ] Tune: Connection pool size
- [ ] Tune: Connection timeout
- [ ] Test: Under load
- [ ] Measure: Performance improvement
- [ ] Document: Connection pool tuning

### Optimization Task 3: Replication Lag Optimization
- [ ] Measure: Current replication lag
- [ ] Identify: Lag causes (network? load?)
- [ ] Optimize: Replication configuration
- [ ] Optimize: Network (if applicable)
- [ ] Test: Lag reduction
- [ ] Measure: Lag improvement
- [ ] Document: Replication lag optimization

### Optimization Task 4: Shard Load Balancing
- [ ] Analyze: Shard load distribution
- [ ] Identify: Uneven shard loads
- [ ] Design: Rebalancing strategy
- [ ] Implement: Rebalancing (if possible)
- [ ] Test: Load distribution after rebalancing
- [ ] Document: Shard balancing

### Optimization Task 5: Data Warehouse Query Optimization
- [ ] Analyze: Data warehouse queries
- [ ] Optimize: Star schema design
- [ ] Optimize: Indexes on fact tables
- [ ] Optimize: Indexes on dimension tables
- [ ] Test: Query performance
- [ ] Measure: Performance improvement
- [ ] Document: Warehouse optimization

---

## DR & Backup TODOs

### Backup Task 1: Database Backup Implementation
- [ ] Design: Backup strategy (full, incremental)
- [ ] Implement: Full backup script
- [ ] Implement: Incremental backup script
- [ ] Schedule: Backup jobs (daily full, hourly incremental)
- [ ] Test: Backup creation
- [ ] Verify: Backup file integrity
- [ ] Test: Backup restoration
- [ ] Measure: Backup time
- [ ] Measure: Restore time
- [ ] Document: Backup implementation

### Backup Task 2: Point-in-Time Recovery
- [ ] Configure: Binary logging (for PITR)
- [ ] Implement: PITR procedure
- [ ] Test: Restore to specific point in time
- [ ] Verify: Data correctness after restore
- [ ] Measure: PITR time
- [ ] Document: PITR procedure

### Backup Task 3: Backup Verification
- [ ] Implement: Automated backup verification
- [ ] Test: Backup file integrity checks
- [ ] Test: Restore verification (periodic)
- [ ] Alert: If backup fails
- [ ] Alert: If restore fails
- [ ] Document: Backup verification

### DR Task 1: Disaster Recovery Plan
- [ ] Design: Complete DR plan
- [ ] Define: RTO (Recovery Time Objective)
- [ ] Define: RPO (Recovery Point Objective)
- [ ] Design: Failover procedure (step-by-step)
- [ ] Design: Failback procedure
- [ ] Design: Data recovery procedure
- [ ] Document: Complete DR plan (1000 words)

### DR Task 2: Failover Testing
- [ ] Simulate: Primary database failure
- [ ] Test: Automatic failover (if configured)
- [ ] Test: Manual failover procedure
- [ ] Measure: Failover time (RTO)
- [ ] Measure: Data loss (RPO)
- [ ] Verify: System works after failover
- [ ] Test: Failback procedure
- [ ] Document: Failover test results

### DR Task 3: Backup Restoration Testing
- [ ] Test: Full backup restoration
- [ ] Measure: Restore time
- [ ] Verify: Data correctness
- [ ] Test: Incremental backup restoration
- [ ] Test: PITR restoration
- [ ] Document: Restoration test results

### DR Task 4: Multi-Region Failover
- [ ] Design: Multi-region failover strategy
- [ ] Test: Region failover
- [ ] Measure: Failover time
- [ ] Test: Data consistency after failover
- [ ] Test: Failback to primary region
- [ ] Document: Multi-region failover

### DR Task 5: Data Loss Prevention
- [ ] Design: Zero data loss strategy
- [ ] Implement: Synchronous replication (if needed)
- [ ] Test: Data loss scenarios
- [ ] Verify: Zero data loss
- [ ] Measure: Performance impact
- [ ] Document: Data loss prevention

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Architecture exercises hoàn thành?
- [ ] Review: Tất cả Data Pipeline tasks hoàn thành?
- [ ] Review: Tất cả Optimization tasks hoàn thành?
- [ ] Review: Tất cả DR & Backup tasks hoàn thành?
- [ ] Rate: Database architecture skills (1-10)
- [ ] Rate: Data pipeline skills (1-10)
- [ ] Rate: Backup/DR skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Architecture Review
- [ ] Review: All database architectures designed
- [ ] Check: Data protection considered?
- [ ] Check: Backup strategy included?
- [ ] Check: DR plan included?
- [ ] Check: High availability designed?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Document: Architecture review findings

### Backup/DR Review
- [ ] Review: Backup strategy
- [ ] Verify: Backup frequency adequate?
- [ ] Verify: Backup retention adequate?
- [ ] Verify: Backup tested?
- [ ] Review: DR plan
- [ ] Verify: RTO achievable?
- [ ] Verify: RPO acceptable?
- [ ] Verify: DR plan tested?
- [ ] Identify: Backup/DR improvements
- [ ] Document: Backup/DR review

### Performance Review
- [ ] Review: Query performance optimizations
- [ ] Review: Replication lag optimizations
- [ ] Review: Connection pool tuning
- [ ] Measure: Overall performance improvements
- [ ] Identify: Remaining bottlenecks
- [ ] Document: Performance review

### Data Safety Review
- [ ] Review: Data protection measures
- [ ] Check: Encryption implemented?
- [ ] Check: Access control adequate?
- [ ] Check: Audit logging enabled?
- [ ] Check: Backup verified?
- [ ] Check: DR plan tested?
- [ ] Identify: Data safety gaps
- [ ] Document: Data safety review

### Knowledge Check
- [ ] Explain: OLTP vs OLAP (viết comparison, không xem notes)
- [ ] Explain: Read/write splitting - pros và cons (viết analysis, không xem notes)
- [ ] Explain: CDC - approaches và use cases (viết explanation, không xem notes)
- [ ] Explain: RTO vs RPO (viết comparison, không xem notes)
- [ ] Explain: Synchronous vs Asynchronous replication (viết comparison, không xem notes)
- [ ] Solve: Full backup = 1 hour, incremental = 10 min. Daily full, hourly incremental. Total backup time per day?
- [ ] Solve: RTO = 1 hour, RPO = 15 min. What does this mean? (viết explanation)
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 data engineering concepts còn confuse
- [ ] Write: 1 backup/DR lesson learned
- [ ] Write: Confidence level cho Week 11 (1-10)
- [ ] Compare: Week 9 vs Week 10 progress
- [ ] Plan: Preparation cho Week 11 (Security & Monitoring)
- [ ] Set: Goals cho Week 11
- [ ] Document: Week 10 reflection (500 words)

### Mentor Questions (Answer these - protect data)
- [ ] Q1: Database có 1TB data. Full backup takes 2 hours. Incremental = 10 min. RPO = 1 hour. Backup strategy? Justify. (viết backup strategy với calculations)
- [ ] Q2: Multi-region replication, async. Region A fails. Data loss? RPO? Làm sao achieve zero data loss? Trade-offs? (viết analysis với RPO calculation)
- [ ] Q3: Backup created daily, never tested. Production database corrupts. Restore fails. What went wrong? Làm sao prevent? (viết analysis với prevention strategy)
- [ ] Q4: Read replica lag = 5 seconds. User updates balance, immediately reads. Stale read? Làm sao fix? Trade-offs? (viết solution với trade-off analysis)
- [ ] Q5: OLTP database, 100M transactions/day. ETL to warehouse takes 6 hours. Analytics queries stale data. Làm sao improve? (viết optimization strategy)
- [ ] Q6: RTO = 30 min, RPO = 5 min. Current failover takes 2 hours, backup every 24 hours. Gap? Làm sao close gap? (viết analysis với improvement plan)
- [ ] Review: Answers có prioritize data protection? Có consider all failure scenarios? Có have tested procedures?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Architecture TODOs: ✅ Complete với designs
- [ ] Tất cả Data Pipeline TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Optimization TODOs: ✅ Complete với performance measurements
- [ ] Tất cả DR & Backup TODOs: ✅ Complete với tested procedures
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Architecture diagrams saved: ✅ Yes
- [ ] Backup/DR procedures saved: ✅ Yes
- [ ] Backup restoration tested: ✅ Yes
- [ ] DR failover tested: ✅ Yes
- [ ] Ready for Week 11: ✅ Yes

---

> **Mentor Final Check**: Data protection is non-negotiable. Nếu bạn không có tested backup và DR plan, bạn are one failure away from disaster. Principal database engineer không accept "we'll add backup later" hoặc "DR plan is on roadmap". Hãy honest: Bạn có tested backup restoration chưa? Bạn có tested DR failover chưa? Bạn có verified zero data loss chưa? Nếu chưa, làm lại. Production systems need data protection from day one, not as afterthought. Data loss is unacceptable.
