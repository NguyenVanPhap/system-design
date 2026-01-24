# 🎯 System Design Learning Roadmap - 12 Tuần
## Lộ trình từ Backend Developer → Senior/Lead System Designer

> **Mentor Note**: Đây không phải là khóa học lý thuyết. Mỗi tuần bạn sẽ phải THỰC HÀNH, CODE, và DESIGN. Nếu bạn chỉ đọc mà không làm, bạn sẽ thất bại. Hãy nghiêm túc với bản thân.

---

## 📋 Tổng quan

**Mục tiêu**: Đạt level Senior/Lead System Designer trong 3-5 tháng  
**Thời gian**: 2 sessions/tuần × 12 tuần = 24 sessions  
**Phong cách**: Practical-first, minimal theory, real-world backend systems

**Hệ thống thực tế sẽ thiết kế**:
- Payment Processing System
- Betting Platform
- Digital Wallet
- Affiliate Marketing System
- Trading Platform

---

## 📅 Tuần 1: Fundamentals - Scalability & Performance

### 🎯 Main Topic
Hiểu sâu về Scalability, Performance Metrics, và cách đo lường hệ thống

### 📚 Key Concepts
- **Vertical vs Horizontal Scaling**: Khi nào dùng cái nào?
- **Performance Metrics**: Latency, Throughput, QPS, TPS
- **Bottleneck Identification**: CPU, Memory, I/O, Network
- **Load Testing**: Cách test và measure performance
- **Capacity Planning**: Tính toán resource cần thiết

### 🌍 Real-world Examples
- **Payment Gateway**: Xử lý 10,000 TPS trong Black Friday
- **Betting Platform**: Xử lý 100,000 concurrent bets trong World Cup
- **Wallet System**: Xử lý 1M transactions/ngày

### 🛠️ Mini Project
**Design: Payment Gateway - High Throughput**

Thiết kế hệ thống payment gateway có thể xử lý:
- 10,000 transactions/second
- 99.9% uptime
- Average latency < 100ms
- Handle peak traffic (10x normal)

**Deliverables**:
1. Architecture diagram (draw.io hoặc Excalidraw)
2. Component breakdown với responsibilities
3. Performance estimates (QPS mỗi component)
4. Bottleneck analysis

### 💻 Hands-on Coding Tasks
1. **Load Testing với JMeter/Gatling**
   - Tạo test plan cho một REST API đơn giản
   - Measure: QPS, latency (p50, p95, p99), error rate
   - Identify bottleneck

2. **Performance Profiling với Java**
   - Sử dụng JProfiler hoặc VisualVM
   - Profile một Spring Boot app đơn giản
   - Identify memory leaks, CPU hotspots

3. **Build Simple Load Balancer (Optional)**
   - Round-robin algorithm
   - Health check mechanism
   - Basic implementation trong Java

### ✅ TODO Checklist

- [ ] Đọc và hiểu: "Designing Data-Intensive Applications" - Chapter 1
- [ ] Xem video: "System Design Interview" - Scalability basics
- [ ] Setup JMeter/Gatling và chạy load test đầu tiên
- [ ] Profile một Spring Boot application
- [ ] Thiết kế Payment Gateway architecture
- [ ] Viết document giải thích design decisions
- [ ] Review design với mentor/peer (nếu có)
- [ ] **Reflection**: Ghi lại 3 điều học được, 2 điều còn confuse

### 🎓 Expected Outcome
- Hiểu rõ sự khác biệt giữa vertical và horizontal scaling
- Biết cách measure và identify bottlenecks
- Có thể estimate capacity cho một hệ thống đơn giản
- Thiết kế được một hệ thống high-throughput cơ bản

---

## 📅 Tuần 2: Availability & Reliability

### 🎯 Main Topic
High Availability, Fault Tolerance, và Disaster Recovery

### 📚 Key Concepts
- **Availability Metrics**: 99.9% vs 99.99% (downtime calculation)
- **Fault Tolerance**: Single Point of Failure (SPOF)
- **Redundancy**: Active-Active, Active-Passive
- **Disaster Recovery**: RTO, RPO
- **Circuit Breaker Pattern**: Hystrix, Resilience4j
- **Health Checks**: Liveness vs Readiness probes

### 🌍 Real-world Examples
- **Payment System**: Không thể down trong giờ cao điểm
- **Betting Platform**: Mất 1 phút downtime = mất hàng triệu đô
- **Wallet System**: Phải available 24/7, global

### 🛠️ Mini Project
**Design: Betting Platform - High Availability**

Thiết kế hệ thống betting với:
- 99.99% availability (52 phút downtime/năm)
- Zero data loss
- Auto-failover trong < 30 giây
- Multi-region deployment

**Deliverables**:
1. High-level architecture với redundancy
2. Failure scenarios và mitigation strategies
3. Disaster recovery plan
4. Monitoring và alerting strategy

### 💻 Hands-on Coding Tasks
1. **Implement Circuit Breaker**
   - Sử dụng Resilience4j trong Spring Boot
   - Implement cho external API calls
   - Test failure scenarios

2. **Health Check Endpoints**
   - Implement `/health`, `/readiness`, `/liveness`
   - Database connection check
   - External service dependency check

3. **Retry Mechanism với Exponential Backoff**
   - Implement retry logic cho failed requests
   - Exponential backoff strategy
   - Max retry limits

### ✅ TODO Checklist

- [ ] Tính toán downtime cho các SLA levels (99%, 99.9%, 99.99%)
- [ ] Đọc về Circuit Breaker pattern
- [ ] Implement Circuit Breaker trong Spring Boot project
- [ ] Implement health check endpoints
- [ ] Thiết kế Betting Platform với HA requirements
- [ ] Document failure scenarios (ít nhất 5 scenarios)
- [ ] Design disaster recovery plan
- [ ] **Reflection**: Liệt kê SPOF trong design của bạn, cách fix?

### 🎓 Expected Outcome
- Hiểu sâu về availability và cách tính toán
- Biết cách design hệ thống fault-tolerant
- Implement được circuit breaker và health checks
- Thiết kế được hệ thống với 99.99% availability

---

## 📅 Tuần 3: Database Design - SQL Optimization & Indexing

### 🎯 Main Topic
Database Performance, Query Optimization, và Indexing Strategies

### 📚 Key Concepts
- **Index Types**: B-tree, Hash, Composite indexes
- **Query Optimization**: EXPLAIN PLAN, query tuning
- **Normalization vs Denormalization**: Trade-offs
- **Partitioning**: Horizontal, Vertical partitioning
- **Connection Pooling**: HikariCP configuration
- **Read Replicas**: Master-Slave replication

### 🌍 Real-world Examples
- **Payment System**: Query 1M transactions/second
- **Betting Platform**: Complex queries với JOINs trên bảng lớn
- **Wallet System**: Balance queries phải < 10ms

### 🛠️ Mini Project
**Design: Wallet System Database**

Thiết kế database schema cho wallet system:
- 10M users
- 100M transactions
- Balance queries < 10ms
- Transaction history queries
- Support concurrent balance updates

**Deliverables**:
1. Complete database schema (ERD)
2. Index strategy cho mỗi bảng
3. Query optimization plan
4. Partitioning strategy (nếu cần)

### 💻 Hands-on Coding Tasks
1. **Query Optimization Exercise**
   - Tạo bảng với 1M+ records
   - Write slow queries (JOIN, GROUP BY, ORDER BY)
   - Optimize bằng indexes
   - Measure performance improvement

2. **Connection Pooling Tuning**
   - Setup HikariCP trong Spring Boot
   - Tune pool size dựa trên load
   - Monitor connection pool metrics

3. **Read Replica Setup (Optional)**
   - Setup MySQL master-slave replication
   - Configure Spring Boot để route read queries
   - Test failover scenario

### ✅ TODO Checklist

- [ ] Đọc về database indexing (B-tree, composite indexes)
- [ ] Practice EXPLAIN PLAN với slow queries
- [ ] Tạo database với 1M+ records và optimize queries
- [ ] Setup và tune HikariCP connection pool
- [ ] Thiết kế Wallet System database schema
- [ ] Design index strategy cho tất cả tables
- [ ] Write optimized queries cho common use cases
- [ ] **Reflection**: 3 queries bạn optimize, performance improvement?

### 🎓 Expected Outcome
- Hiểu sâu về indexing và query optimization
- Biết cách design database schema cho high-performance
- Optimize được slow queries
- Thiết kế được database cho hệ thống lớn

---

## 📅 Tuần 4: Database Design - Sharding & Replication

### 🎯 Main Topic
Database Scaling: Sharding Strategies và Advanced Replication

### 📚 Key Concepts
- **Sharding Strategies**: Range-based, Hash-based, Directory-based
- **Shard Key Selection**: Criteria và trade-offs
- **Cross-shard Queries**: Challenges và solutions
- **Consistent Hashing**: Load distribution
- **Replication Patterns**: Master-Slave, Master-Master, Multi-master
- **Eventual Consistency**: CAP theorem trong database context

### 🌍 Real-world Examples
- **Payment System**: Shard transactions theo user_id hoặc merchant_id
- **Betting Platform**: Shard bets theo match_id hoặc user_id
- **Wallet System**: Shard wallets theo user_id với consistent hashing

### 🛠️ Mini Project
**Design: Payment System - Database Sharding**

Thiết kế sharding strategy cho payment system:
- 1B transactions
- Shard theo merchant_id
- Support cross-merchant queries
- Handle shard rebalancing

**Deliverables**:
1. Sharding strategy document
2. Shard key selection rationale
3. Cross-shard query handling approach
4. Rebalancing strategy

### 💻 Hands-on Coding Tasks
1. **Implement Sharding Logic (Simplified)**
   - Implement hash-based sharding
   - Route queries đến correct shard
   - Handle shard lookup

2. **Consistent Hashing Implementation**
   - Implement consistent hashing algorithm
   - Add/remove nodes dynamically
   - Test load distribution

3. **Shard Routing Service**
   - Create service để route queries
   - Implement shard selection logic
   - Add caching cho shard mappings

### ✅ TODO Checklist

- [ ] Đọc về database sharding strategies
- [ ] Hiểu consistent hashing algorithm
- [ ] Implement simplified sharding logic
- [ ] Implement consistent hashing
- [ ] Thiết kế Payment System sharding strategy
- [ ] Design shard key selection
- [ ] Plan cross-shard query handling
- [ ] **Reflection**: Trade-offs của sharding strategy bạn chọn?

### 🎓 Expected Outcome
- Hiểu sâu về sharding và khi nào cần shard
- Biết cách chọn shard key phù hợp
- Implement được basic sharding logic
- Thiết kế được sharding strategy cho hệ thống lớn

---

## 📅 Tuần 5: Caching Strategies

### 🎯 Main Topic
Caching Layers, Cache Patterns, và Cache Invalidation

### 📚 Key Concepts
- **Cache Types**: In-memory (Redis), Application-level, CDN
- **Cache Patterns**: Cache-Aside, Write-Through, Write-Behind
- **Cache Invalidation**: TTL, Event-driven invalidation
- **Cache Warming**: Pre-loading strategies
- **Distributed Caching**: Redis Cluster, Memcached
- **Cache Coherence**: Handling stale data

### 🌍 Real-world Examples
- **Payment System**: Cache merchant info, user balances
- **Betting Platform**: Cache match odds, user bets
- **Wallet System**: Cache user balances (với careful invalidation)

### 🛠️ Mini Project
**Design: Betting Platform - Multi-layer Caching**

Thiết kế caching strategy cho betting platform:
- Cache match odds (update mỗi giây)
- Cache user bets (read-heavy)
- Cache match results (write-once, read-many)
- Handle cache invalidation

**Deliverables**:
1. Caching architecture (multiple layers)
2. Cache pattern cho mỗi data type
3. Invalidation strategy
4. Cache sizing và eviction policies

### 💻 Hands-on Coding Tasks
1. **Redis Integration với Spring Boot**
   - Setup Redis
   - Implement cache-aside pattern
   - Implement cache với Spring Cache abstraction

2. **Cache Invalidation Logic**
   - Implement TTL-based invalidation
   - Implement event-driven invalidation
   - Test cache coherence

3. **Cache Warming Strategy**
   - Implement pre-loading cho hot data
   - Schedule cache warming jobs
   - Monitor cache hit rates

### ✅ TODO Checklist

- [ ] Đọc về caching patterns (Cache-Aside, Write-Through, etc.)
- [ ] Setup Redis và integrate với Spring Boot
- [ ] Implement cache-aside pattern
- [ ] Implement cache invalidation logic
- [ ] Thiết kế Betting Platform caching strategy
- [ ] Design multi-layer caching architecture
- [ ] Implement cache warming
- [ ] **Reflection**: Cache hit rate target? Làm sao measure?

### 🎓 Expected Outcome
- Hiểu sâu về các caching patterns
- Biết cách design multi-layer caching
- Implement được caching với Redis
- Thiết kế được caching strategy phù hợp

---

## 📅 Tuần 6: Load Balancing & API Design

### 🎯 Main Topic
Load Balancing Algorithms, API Design Patterns, và Rate Limiting

### 📚 Key Concepts
- **Load Balancing**: Round-robin, Weighted, Least connections, IP hash
- **API Design**: RESTful principles, versioning, pagination
- **Rate Limiting**: Token bucket, Sliding window, Fixed window
- **API Gateway**: Routing, authentication, rate limiting
- **Idempotency**: Ensuring safe retries
- **API Versioning**: URL, Header, Query parameter

### 🌍 Real-world Examples
- **Payment API**: Rate limit theo merchant tier
- **Betting API**: Rate limit để prevent abuse
- **Wallet API**: Idempotent operations cho transactions

### 🛠️ Mini Project
**Design: Payment API Gateway**

Thiết kế API Gateway cho payment system:
- Route requests đến multiple services
- Rate limiting theo merchant tier
- Authentication và authorization
- Request/response transformation
- API versioning

**Deliverables**:
1. API Gateway architecture
2. Rate limiting strategy
3. API versioning approach
4. Authentication flow

### 💻 Hands-on Coding Tasks
1. **Implement Rate Limiter**
   - Token bucket algorithm
   - Sliding window algorithm
   - Integrate với Spring Boot

2. **API Gateway với Spring Cloud Gateway**
   - Setup Spring Cloud Gateway
   - Implement routing rules
   - Add rate limiting filters

3. **Idempotent API Endpoints**
   - Design idempotent payment API
   - Implement idempotency keys
   - Handle duplicate requests

### ✅ TODO Checklist

- [ ] Đọc về load balancing algorithms
- [ ] Đọc về rate limiting algorithms
- [ ] Implement rate limiter (token bucket hoặc sliding window)
- [ ] Setup Spring Cloud Gateway
- [ ] Implement routing và rate limiting
- [ ] Thiết kế Payment API Gateway
- [ ] Design idempotent API endpoints
- [ ] **Reflection**: Trade-offs của rate limiting algorithms?

### 🎓 Expected Outcome
- Hiểu sâu về load balancing và rate limiting
- Biết cách design API Gateway
- Implement được rate limiting
- Thiết kế được scalable API architecture

---

## 📅 Tuần 7: Message Queues & Async Processing

### 🎯 Main Topic
Message Queues, Async Processing, và Event-Driven Architecture

### 📚 Key Concepts
- **Message Queue Patterns**: Point-to-point, Pub/Sub
- **Queue Types**: RabbitMQ, Kafka, AWS SQS
- **Message Ordering**: FIFO queues, partitioning
- **Dead Letter Queues**: Handling failed messages
- **Event Sourcing**: Storing events instead of state
- **Saga Pattern**: Distributed transactions

### 🌍 Real-world Examples
- **Payment System**: Async payment processing, notification queue
- **Betting Platform**: Event-driven bet processing, settlement queue
- **Wallet System**: Async balance updates, transaction logs

### 🛠️ Mini Project
**Design: Wallet System - Event-Driven Architecture**

Thiết kế event-driven wallet system:
- Async balance updates
- Transaction event streaming
- Event sourcing cho transaction history
- Handle eventual consistency

**Deliverables**:
1. Event-driven architecture diagram
2. Event schema design
3. Message queue topology
4. Event sourcing strategy

### 💻 Hands-on Coding Tasks
1. **RabbitMQ Integration**
   - Setup RabbitMQ
   - Implement producer và consumer
   - Handle message acknowledgment
   - Implement dead letter queue

2. **Kafka Integration (Optional)**
   - Setup Kafka
   - Implement producer và consumer
   - Handle partitioning và ordering

3. **Event Sourcing Implementation**
   - Design event store
   - Implement event sourcing cho một domain
   - Replay events để rebuild state

### ✅ TODO Checklist

- [ ] Đọc về message queue patterns
- [ ] Setup RabbitMQ và implement producer/consumer
- [ ] Implement dead letter queue handling
- [ ] Thiết kế Wallet System event-driven architecture
- [ ] Design event schemas
- [ ] Implement event sourcing (basic)
- [ ] **Reflection**: Khi nào dùng queue vs direct call?

### 🎓 Expected Outcome
- Hiểu sâu về message queues và async processing
- Biết cách design event-driven architecture
- Implement được message queue integration
- Thiết kế được hệ thống với async processing

---

## 📅 Tuần 8: Microservices Architecture

### 🎯 Main Topic
Microservices Design, Service Communication, và Distributed Systems Challenges

### 📚 Key Concepts
- **Microservices vs Monolith**: Trade-offs
- **Service Communication**: Synchronous (REST, gRPC) vs Asynchronous (MQ)
- **Service Discovery**: Eureka, Consul, Kubernetes
- **API Gateway**: Centralized entry point
- **Distributed Tracing**: Zipkin, Jaeger
- **Service Mesh**: Istio, Linkerd

### 🌍 Real-world Examples
- **Payment System**: Payment service, Notification service, Reporting service
- **Betting Platform**: Bet service, Match service, Settlement service
- **Wallet System**: Wallet service, Transaction service, Notification service

### 🛠️ Mini Project
**Design: Betting Platform - Microservices Architecture**

Thiết kế microservices cho betting platform:
- Service boundaries và responsibilities
- Inter-service communication
- Data consistency across services
- Service discovery và load balancing

**Deliverables**:
1. Microservices architecture diagram
2. Service boundaries definition
3. Communication patterns (sync/async)
4. Data consistency strategy

### 💻 Hands-on Coding Tasks
1. **Build Microservices với Spring Cloud**
   - Create 2-3 microservices
   - Setup service discovery (Eureka)
   - Implement inter-service communication

2. **Distributed Tracing**
   - Setup Zipkin hoặc Jaeger
   - Add tracing vào microservices
   - Trace requests across services

3. **API Gateway Integration**
   - Integrate API Gateway với microservices
   - Route requests đến correct services
   - Add authentication middleware

### ✅ TODO Checklist

- [ ] Đọc về microservices patterns và anti-patterns
- [ ] Build 2-3 microservices với Spring Cloud
- [ ] Setup service discovery
- [ ] Implement distributed tracing
- [ ] Thiết kế Betting Platform microservices
- [ ] Define service boundaries
- [ ] Design inter-service communication
- [ ] **Reflection**: Khi nào nên split service? Khi nào không?

### 🎓 Expected Outcome
- Hiểu sâu về microservices architecture
- Biết cách design service boundaries
- Implement được microservices với Spring Cloud
- Thiết kế được microservices architecture

---

## 📅 Tuần 9: Distributed Systems - Consistency & Transactions

### 🎯 Main Topic
CAP Theorem, Consistency Models, và Distributed Transactions

### 📚 Key Concepts
- **CAP Theorem**: Consistency, Availability, Partition tolerance
- **Consistency Models**: Strong, Eventual, Causal
- **ACID vs BASE**: Trade-offs
- **Distributed Transactions**: 2PC, 3PC, Saga pattern
- **Vector Clocks**: Causality tracking
- **CRDTs**: Conflict-free Replicated Data Types

### 🌍 Real-world Examples
- **Payment System**: Strong consistency cho balance, eventual cho history
- **Betting Platform**: Eventual consistency cho odds updates
- **Wallet System**: Strong consistency cho balance, eventual cho notifications

### 🛠️ Mini Project
**Design: Payment System - Consistency Strategy**

Thiết kế consistency strategy cho payment system:
- Strong consistency cho critical operations (balance updates)
- Eventual consistency cho non-critical (notifications, reports)
- Handle distributed transactions
- Conflict resolution

**Deliverables**:
1. Consistency model cho mỗi operation
2. Distributed transaction strategy
3. Conflict resolution approach
4. Trade-off analysis (consistency vs availability)

### 💻 Hands-on Coding Tasks
1. **Implement Saga Pattern**
   - Design saga cho payment flow
   - Implement compensating transactions
   - Handle saga failures

2. **Eventual Consistency Demo**
   - Implement eventually consistent system
   - Show how data converges
   - Handle conflicts

3. **Distributed Lock Implementation**
   - Implement distributed lock với Redis
   - Use cho critical sections
   - Handle lock expiration

### ✅ TODO Checklist

- [ ] Đọc và hiểu CAP theorem sâu sắc
- [ ] Đọc về consistency models
- [ ] Implement Saga pattern
- [ ] Implement distributed lock
- [ ] Thiết kế Payment System consistency strategy
- [ ] Define consistency requirements cho mỗi operation
- [ ] Design conflict resolution
- [ ] **Reflection**: Khi nào chọn strong vs eventual consistency?

### 🎓 Expected Outcome
- Hiểu sâu về CAP theorem và consistency models
- Biết cách chọn consistency level phù hợp
- Implement được distributed transactions
- Thiết kế được consistency strategy

---

## 📅 Tuần 10: Advanced Database Patterns

### 🎯 Main Topic
NoSQL Databases, Time-Series Data, và Specialized Storage

### 📚 Key Concepts
- **NoSQL Types**: Document (MongoDB), Key-Value (Redis), Column (Cassandra), Graph (Neo4j)
- **When to Use NoSQL**: Use cases và trade-offs
- **Time-Series Databases**: InfluxDB, TimescaleDB
- **Search Engines**: Elasticsearch cho full-text search
- **Polyglot Persistence**: Using multiple databases
- **CQRS**: Command Query Responsibility Segregation

### 🌍 Real-world Examples
- **Payment System**: MongoDB cho transaction logs, Elasticsearch cho search
- **Betting Platform**: Time-series DB cho odds history, Redis cho real-time data
- **Wallet System**: PostgreSQL cho transactions, Redis cho balances

### 🛠️ Mini Project
**Design: Affiliate System - Multi-Database Architecture**

Thiết kế database architecture cho affiliate system:
- User data (PostgreSQL)
- Click tracking (Time-series DB)
- Commission calculations (PostgreSQL)
- Analytics queries (Elasticsearch)
- Real-time leaderboard (Redis)

**Deliverables**:
1. Database selection cho mỗi use case
2. Data flow giữa databases
3. CQRS implementation plan
4. Data synchronization strategy

### 💻 Hands-on Coding Tasks
1. **MongoDB Integration**
   - Setup MongoDB
   - Design document schema
   - Implement CRUD operations
   - Compare với SQL queries

2. **Elasticsearch Integration**
   - Setup Elasticsearch
   - Index documents
   - Implement search queries
   - Full-text search

3. **CQRS Implementation**
   - Separate command và query models
   - Implement read và write sides
   - Sync data between sides

### ✅ TODO Checklist

- [ ] Đọc về NoSQL database types và use cases
- [ ] Setup MongoDB và implement basic operations
- [ ] Setup Elasticsearch và implement search
- [ ] Thiết kế Affiliate System multi-database architecture
- [ ] Design database selection rationale
- [ ] Implement CQRS pattern (basic)
- [ ] **Reflection**: Khi nào dùng SQL vs NoSQL? Trade-offs?

### 🎓 Expected Outcome
- Hiểu sâu về các loại NoSQL databases
- Biết cách chọn database phù hợp cho từng use case
- Implement được multi-database architecture
- Thiết kế được polyglot persistence strategy

---

## 📅 Tuần 11: Security & Monitoring

### 🎯 Main Topic
System Security, Observability, và Production Readiness

### 📚 Key Concepts
- **Security**: Authentication, Authorization, Encryption, OAuth2, JWT
- **Monitoring**: Metrics, Logging, Tracing (Three Pillars)
- **Alerting**: When và how to alert
- **Performance Monitoring**: APM tools
- **Security Best Practices**: SQL injection, XSS, CSRF prevention
- **Secrets Management**: Vault, AWS Secrets Manager

### 🌍 Real-world Examples
- **Payment System**: PCI-DSS compliance, encryption at rest và in transit
- **Betting Platform**: Fraud detection, rate limiting, audit logs
- **Wallet System**: Multi-factor authentication, transaction monitoring

### 🛠️ Mini Project
**Design: Trading Platform - Security & Monitoring**

Thiết kế security và monitoring cho trading platform:
- Authentication và authorization
- API security (rate limiting, input validation)
- Monitoring strategy (metrics, logs, traces)
- Alerting rules
- Audit logging

**Deliverables**:
1. Security architecture
2. Authentication/authorization flow
3. Monitoring và alerting strategy
4. Audit logging approach

### 💻 Hands-on Coding Tasks
1. **Implement JWT Authentication**
   - Generate và validate JWT tokens
   - Implement refresh token mechanism
   - Secure API endpoints

2. **Monitoring Setup**
   - Setup Prometheus và Grafana
   - Export custom metrics từ Spring Boot
   - Create dashboards

3. **Structured Logging**
   - Implement structured logging (JSON format)
   - Add correlation IDs
   - Centralized logging (ELK stack optional)

### ✅ TODO Checklist

- [ ] Đọc về OAuth2 và JWT
- [ ] Implement JWT authentication
- [ ] Setup Prometheus và Grafana
- [ ] Export metrics từ Spring Boot app
- [ ] Thiết kế Trading Platform security
- [ ] Design monitoring và alerting strategy
- [ ] Implement structured logging
- [ ] **Reflection**: Security vulnerabilities trong design của bạn?

### 🎓 Expected Outcome
- Hiểu sâu về security best practices
- Biết cách design secure systems
- Implement được authentication và monitoring
- Thiết kế được production-ready systems

---

## 📅 Tuần 12: Real-world Case Studies & System Design Interview

### 🎯 Main Topic
Complete System Design: End-to-end design cho real-world systems

### 📚 Key Concepts
- **System Design Interview Process**: Requirements, constraints, estimation
- **Back-of-envelope Calculations**: QPS, storage, bandwidth
- **Trade-off Analysis**: Performance vs Cost, Consistency vs Availability
- **Scalability Patterns**: Horizontal scaling, caching, sharding
- **Design Review**: Identifying weaknesses và improvements

### 🌍 Real-world Examples
- **Design: Uber/Lyft** (ride-sharing)
- **Design: Twitter** (social media feed)
- **Design: Netflix** (video streaming)
- **Design: Payment Gateway** (comprehensive)
- **Design: Stock Trading Platform** (low latency)

### 🛠️ Mini Project
**Design: Complete Payment Gateway System**

Thiết kế end-to-end payment gateway:
- Handle 100K TPS
- 99.99% availability
- Multi-region deployment
- Fraud detection
- Real-time reporting
- Merchant dashboard

**Deliverables**:
1. Complete system architecture
2. Component design (detailed)
3. Database design
4. API design
5. Scalability plan
6. Security strategy
7. Monitoring strategy
8. Cost estimation

### 💻 Hands-on Coding Tasks
1. **Build MVP Payment Gateway**
   - Core payment processing
   - Database schema
   - Basic APIs
   - Integration với payment processor (Stripe/PayPal sandbox)

2. **Load Testing**
   - Test với realistic load
   - Identify bottlenecks
   - Optimize performance

3. **System Design Presentation**
   - Prepare presentation
   - Explain design decisions
   - Answer questions về trade-offs

### ✅ TODO Checklist

- [ ] Review tất cả concepts đã học
- [ ] Practice system design interview questions (3-5 questions)
- [ ] Thiết kế complete Payment Gateway system
- [ ] Build MVP payment gateway (basic version)
- [ ] Load test MVP
- [ ] Prepare system design presentation
- [ ] **Self-assessment**: Đánh giá level hiện tại (Junior/Mid/Senior)
- [ ] **Reflection**: 3 điều mạnh nhất, 3 điều cần improve

### 🎓 Expected Outcome
- Có thể design complete systems end-to-end
- Biết cách approach system design interview
- Build được MVP của một hệ thống phức tạp
- Đạt level Senior System Designer

---

## 📊 Progress Tracker

### Tuần 1: Scalability & Performance
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 2: Availability & Reliability
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 3: Database Design - SQL Optimization
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 4: Database Design - Sharding
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 5: Caching Strategies
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 6: Load Balancing & API Design
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 7: Message Queues & Async Processing
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 8: Microservices Architecture
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 9: Distributed Systems - Consistency
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 10: Advanced Database Patterns
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 11: Security & Monitoring
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

### Tuần 12: Real-world Case Studies
- [ ] Completed
- [ ] Concepts mastered
- [ ] Project completed
- [ ] Coding tasks done
- [ ] Reflection written

---

## 🎯 Final Assessment

Sau 12 tuần, bạn nên có khả năng:

- [ ] Design hệ thống có thể scale đến hàng triệu users
- [ ] Identify và resolve bottlenecks
- [ ] Choose appropriate database và caching strategies
- [ ] Design fault-tolerant systems với high availability
- [ ] Implement distributed systems patterns
- [ ] Design secure và production-ready systems
- [ ] Pass system design interviews tại top tech companies

---

## 📚 Recommended Resources

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu
- "Microservices Patterns" by Chris Richardson

### Online Courses
- Grokking the System Design Interview
- High Scalability blog
- AWS Architecture Center

### Practice Platforms
- LeetCode System Design
- Pramp (mock interviews)
- InterviewBit System Design

---

## 💪 Mentor's Final Words

**Hãy nhớ**: System Design không phải là về biết tất cả patterns. Nó về:
1. **Understanding trade-offs**: Mọi decision đều có trade-off
2. **Thinking critically**: Question assumptions, challenge designs
3. **Practical experience**: Code, build, measure, iterate
4. **Communication**: Explain your design clearly

**Nếu bạn chỉ đọc mà không code**: Bạn sẽ không học được gì.

**Nếu bạn chỉ code mà không design**: Bạn sẽ không scale được.

**Nếu bạn chỉ design mà không measure**: Bạn sẽ không biết nó có work không.

**Hãy làm cả 3**: Design → Code → Measure → Iterate.

Chúc bạn thành công! 🚀

---

*Last Updated: January 2026*
