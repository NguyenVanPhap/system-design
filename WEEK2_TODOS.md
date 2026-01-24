# Week 2 – Scaling Architecture & High Availability

> **Mentor Note**: Tuần 1 bạn đã học fundamentals. Tuần 2 này bạn phải THỰC HÀNH scaling và high availability. Không có lý thuyết suông. Mỗi concept phải được implement và test. Nếu bạn không code, bạn không học được gì.

---

## Study TODOs

### Horizontal vs Vertical Scaling Deep Dive
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 1 - Scaling approaches (pages 30-50)
- [ ] Viết comparison table: Horizontal vs Vertical (10 điểm so sánh)
- [ ] Liệt kê 5 limitations của vertical scaling
- [ ] Liệt kê 5 challenges của horizontal scaling
- [ ] Đọc về "scaling laws" - khi nào vertical scaling fails?
- [ ] Research: Maximum server size available (CPU cores, RAM) - current limits
- [ ] Research: Cost comparison - 1 large server vs 10 small servers (same capacity)
- [ ] Đọc về "scaling out" vs "scaling up" terminology
- [ ] Tìm 3 systems that MUST use horizontal scaling (và tại sao)
- [ ] Tìm 2 systems that CAN use vertical scaling (và tại sao họ chọn)

### Load Balancer Strategies
- [ ] Đọc về 5 load balancing algorithms: Round-robin, Weighted Round-robin, Least Connections, IP Hash, Least Response Time
- [ ] Viết algorithm pseudocode cho mỗi algorithm
- [ ] So sánh: Round-robin vs Least Connections (3 use cases mỗi cái)
- [ ] Đọc về "sticky sessions" (session affinity)
- [ ] Analyze: Khi nào cần sticky sessions? Trade-offs?
- [ ] Đọc về Layer 4 (L4) vs Layer 7 (L7) load balancing
- [ ] So sánh: L4 vs L7 (performance, features, use cases)
- [ ] Research: Popular load balancers (Nginx, HAProxy, AWS ELB, F5)
- [ ] Compare: Nginx vs HAProxy (features, performance, use cases)
- [ ] Đọc về "health checks" trong load balancers
- [ ] Đọc về "graceful degradation" khi backend fails

### Stateless vs Stateful Services
- [ ] Định nghĩa chính xác: Stateless service
- [ ] Định nghĩa chính xác: Stateful service
- [ ] Liệt kê 5 characteristics của stateless service
- [ ] Liệt kê 5 characteristics của stateful service
- [ ] Analyze: REST API - stateless hay stateful? Tại sao?
- [ ] Analyze: WebSocket connection - stateless hay stateful? Tại sao?
- [ ] Đọc về "session state" - where to store?
- [ ] Compare: In-memory session vs Redis session vs Database session (3 pros/cons mỗi cái)
- [ ] Đọc về "stateless authentication" (JWT tokens)
- [ ] Đọc về "stateful authentication" (server-side sessions)
- [ ] Analyze: Khi nào cần stateful service? (3 use cases)
- [ ] Analyze: Khi nào MUST use stateless? (3 use cases)
- [ ] Đọc về "shared nothing architecture"

### Redundancy & Failover Patterns
- [ ] Đọc về "Active-Active" redundancy pattern (deep dive)
- [ ] Đọc về "Active-Passive" redundancy pattern (deep dive)
- [ ] Đọc về "Active-Passive Hot Standby" vs "Active-Passive Cold Standby"
- [ ] Compare: Active-Active vs Active-Passive (cost, complexity, failover time, use cases)
- [ ] Đọc về "failover mechanisms": Automatic vs Manual
- [ ] Đọc về "failover time" - RTO (Recovery Time Objective)
- [ ] Đọc về "data loss" - RPO (Recovery Point Objective)
- [ ] Calculate: Nếu RTO = 5 minutes, có nghĩa là gì?
- [ ] Calculate: Nếu RPO = 1 minute, có nghĩa là gì?
- [ ] Đọc về "split-brain problem" trong Active-Active
- [ ] Đọc về "quorum" và "consensus" trong distributed systems
- [ ] Research: How does database replication handle failover? (Master-Slave)
- [ ] Research: How does application-level failover work?

### Zero-Downtime Deployment
- [ ] Đọc về "blue-green deployment" strategy
- [ ] Đọc về "canary deployment" strategy
- [ ] Đọc về "rolling deployment" strategy
- [ ] Compare: Blue-Green vs Canary vs Rolling (cost, risk, complexity, downtime)
- [ ] Đọc về "feature flags" trong zero-downtime deployments
- [ ] Đọc về "database migrations" trong zero-downtime deployments
- [ ] Analyze: Backward compatibility requirements
- [ ] Đọc về "traffic shifting" strategies
- [ ] Đọc về "rollback strategies" - how to rollback quickly?
- [ ] Research: Kubernetes deployment strategies
- [ ] Research: Spring Boot zero-downtime deployment patterns

### Capacity Planning Basics
- [ ] Đọc về "capacity planning" process (5 steps)
- [ ] Đọc về "baseline measurement" - current capacity
- [ ] Đọc về "growth projection" - future capacity needs
- [ ] Đọc về "headroom" - buffer capacity
- [ ] Calculate: Nếu current load = 1K QPS, growth = 20%/month, cần capacity sau 6 tháng?
- [ ] Đọc về "capacity units" - how to measure?
- [ ] Đọc về "scaling triggers" - when to scale?
- [ ] Đọc về "auto-scaling" vs "manual scaling"
- [ ] Research: AWS Auto Scaling groups
- [ ] Research: Kubernetes Horizontal Pod Autoscaler (HPA)

---

## Design TODOs

### Design Exercise 1: Payment Gateway - Horizontal Scaling
- [ ] Thiết kế Payment Gateway với horizontal scaling requirement
- [ ] Requirement: Scale from 1K QPS to 50K QPS
- [ ] Design: Load balancer architecture (type, algorithm, placement)
- [ ] Design: Application server scaling strategy
- [ ] Design: Database scaling strategy (read replicas, sharding consideration)
- [ ] Design: Stateless service architecture (no session state)
- [ ] Identify: All stateful components (nếu có)
- [ ] Design: Solution để make stateful components stateless
- [ ] Calculate: Number of servers needed (assume 5K QPS per server)
- [ ] Calculate: Cost comparison - vertical vs horizontal scaling
- [ ] Design: Auto-scaling rules (when to scale up/down)
- [ ] Document: Complete architecture diagram với scaling paths
- [ ] Document: Design decisions và trade-offs (500 words)

### Design Exercise 2: Betting Platform - High Availability
- [ ] Thiết kế Betting Platform với 99.99% availability requirement
- [ ] Design: Multi-layer redundancy (application, database, load balancer)
- [ ] Design: Active-Active setup cho application servers
- [ ] Design: Active-Passive setup cho database (Master-Slave)
- [ ] Design: Failover mechanism (automatic, < 30 seconds)
- [ ] Design: Health check strategy (what to check, how often)
- [ ] Design: Monitoring và alerting (what to monitor, alert thresholds)
- [ ] Identify: All SPOFs trong design
- [ ] Eliminate: All SPOFs (redesign nếu cần)
- [ ] Design: Disaster recovery plan (RTO = 5 minutes, RPO = 1 minute)
- [ ] Design: Multi-region deployment (optional, nhưng nên consider)
- [ ] Calculate: Availability của toàn bộ system (series và parallel components)
- [ ] Document: Failure scenarios (ít nhất 5 scenarios) và mitigation
- [ ] Document: Complete HA architecture (500 words)

### Design Exercise 3: Wallet System - Stateless Architecture
- [ ] Thiết kế Wallet System với stateless requirement
- [ ] Requirement: Must be horizontally scalable, no session state
- [ ] Design: Authentication strategy (stateless - JWT)
- [ ] Design: Session management (nếu cần, dùng external store)
- [ ] Design: State storage (Redis cho shared state)
- [ ] Identify: All stateful operations
- [ ] Design: Solution để make operations stateless
- [ ] Design: Load balancer configuration (algorithm, sticky sessions?)
- [ ] Design: Cache strategy (shared cache, not in-memory)
- [ ] Design: Database connection strategy (connection pooling, no local state)
- [ ] Verify: System có thể scale horizontally không? (checklist)
- [ ] Document: Stateless architecture design (500 words)

### Design Exercise 4: Zero-Downtime Deployment Strategy
- [ ] Design: Deployment strategy cho Payment Gateway
- [ ] Requirement: Zero downtime, support rollback
- [ ] Choose: Blue-Green, Canary, or Rolling deployment
- [ ] Justify: Tại sao chọn strategy đó?
- [ ] Design: Traffic routing during deployment
- [ ] Design: Database migration strategy (backward compatible)
- [ ] Design: Feature flag strategy (nếu cần)
- [ ] Design: Rollback procedure (step-by-step)
- [ ] Design: Health check during deployment
- [ ] Design: Monitoring during deployment (what to watch)
- [ ] Estimate: Deployment time và risk
- [ ] Document: Complete deployment strategy (500 words)

### Design Exercise 5: Capacity Planning
- [ ] Scenario: E-commerce platform, current 10K users, expected 1M users in 1 year
- [ ] Measure: Current capacity (QPS, storage, bandwidth) - estimate nếu không có data
- [ ] Project: Future capacity needs (1M users)
- [ ] Calculate: Growth rate (monthly)
- [ ] Calculate: Peak capacity needed (assume 10x average)
- [ ] Design: Scaling plan (when to add servers)
- [ ] Calculate: Server count needed (assume 5K QPS per server)
- [ ] Calculate: Storage needed (assume 1GB per 1K users)
- [ ] Calculate: Bandwidth needed (assume 5KB per request)
- [ ] Design: Auto-scaling rules (triggers, min/max instances)
- [ ] Estimate: Cost projection (monthly, yearly)
- [ ] Design: Monitoring để track capacity usage
- [ ] Document: Complete capacity planning document

---

## Coding TODOs

### Task 1: Stateless Spring Boot Application
- [ ] Tạo Spring Boot project: Stateless Payment API
- [ ] Implement: REST API endpoints (no session state)
- [ ] Implement: JWT-based authentication (stateless)
- [ ] Verify: No in-memory session storage
- [ ] Verify: No server-side session state
- [ ] Test: Deploy 2 instances, verify both can handle same user
- [ ] Test: Kill one instance, verify other instance continues working
- [ ] Document: Stateless design decisions

### Task 2: Load Balancer Implementation
- [ ] Implement: Simple load balancer trong Java
- [ ] Algorithm: Round-robin
- [ ] Algorithm: Weighted round-robin (add weights)
- [ ] Algorithm: Least connections (track connection count)
- [ ] Add: Health check mechanism (ping backend)
- [ ] Add: Remove unhealthy backends automatically
- [ ] Add: Add back healthy backends automatically
- [ ] Test: With 3 backend servers
- [ ] Test: Mark one server unhealthy, verify traffic stops
- [ ] Test: Mark server healthy again, verify traffic resumes
- [ ] Measure: Load balancer overhead (latency added)
- [ ] Document: Code và test results

### Task 3: Health Check Implementation
- [ ] Implement: `/health` endpoint (basic)
- [ ] Implement: `/health/readiness` endpoint
- [ ] Implement: `/health/liveness` endpoint
- [ ] Add: Database connection check trong readiness
- [ ] Add: External service check trong readiness
- [ ] Add: Memory check trong liveness (fail if > 90%)
- [ ] Add: Disk space check trong liveness (fail if > 90%)
- [ ] Add: Thread pool check (fail if all threads busy)
- [ ] Test: All health endpoints
- [ ] Test: Simulate DB down, verify readiness fails
- [ ] Test: Simulate high memory, verify liveness fails
- [ ] Integrate: Health checks với load balancer
- [ ] Document: Health check strategy

### Task 4: Circuit Breaker với Resilience4j
- [ ] Add: Resilience4j dependency
- [ ] Implement: Circuit breaker cho external API call
- [ ] Configure: Failure threshold (50% failures)
- [ ] Configure: Wait duration (60 seconds)
- [ ] Configure: Half-open state (allow 3 requests)
- [ ] Add: Fallback mechanism
- [ ] Test: Normal operation (circuit closed)
- [ ] Test: Simulate failures, verify circuit opens
- [ ] Test: Verify fallback được called khi circuit open
- [ ] Test: Wait duration, verify circuit goes half-open
- [ ] Test: Successful requests in half-open, verify circuit closes
- [ ] Monitor: Circuit breaker metrics
- [ ] Document: Circuit breaker configuration và behavior

### Task 5: Retry với Exponential Backoff
- [ ] Implement: Retry mechanism cho failed requests
- [ ] Algorithm: Exponential backoff (1s, 2s, 4s, 8s)
- [ ] Add: Max retry attempts (3)
- [ ] Add: Jitter (random delay to prevent thundering herd)
- [ ] Test: Transient failure, verify retry works
- [ ] Test: Permanent failure, verify stops after max retries
- [ ] Test: Success after retry, verify no more retries
- [ ] Measure: Total time với retries
- [ ] Integrate: Retry với circuit breaker
- [ ] Document: Retry strategy

### Task 6: Multi-Instance Deployment
- [ ] Build: Spring Boot application JAR
- [ ] Deploy: 2 instances trên local (different ports)
- [ ] Setup: Nginx như load balancer (hoặc use Spring Cloud Gateway)
- [ ] Configure: Round-robin load balancing
- [ ] Configure: Health checks
- [ ] Test: Send 100 requests, verify distributed across instances
- [ ] Test: Kill one instance, verify traffic goes to other
- [ ] Test: Restart killed instance, verify traffic resumes
- [ ] Monitor: Request distribution (logs)
- [ ] Document: Deployment setup

### Task 7: State Externalization (Redis)
- [ ] Setup: Redis server
- [ ] Implement: Session storage trong Redis (thay vì in-memory)
- [ ] Implement: Cache trong Redis (thay vì local cache)
- [ ] Test: Deploy 2 instances, verify shared state (Redis)
- [ ] Test: User login on instance 1, verify session available on instance 2
- [ ] Test: Kill instance 1, verify user still logged in on instance 2
- [ ] Measure: Redis latency impact
- [ ] Document: State externalization strategy

### Task 8: Graceful Shutdown
- [ ] Implement: Graceful shutdown trong Spring Boot
- [ ] Add: Shutdown hook để stop accepting new requests
- [ ] Add: Wait for in-flight requests to complete
- [ ] Add: Timeout (30 seconds max wait)
- [ ] Test: Send requests, then shutdown app
- [ ] Verify: In-flight requests complete before shutdown
- [ ] Verify: New requests rejected during shutdown
- [ ] Test: With load balancer (remove from pool during shutdown)
- [ ] Document: Graceful shutdown implementation

---

## Failure & Resilience TODOs

### Failure Simulation 1: Server Failure
- [ ] Setup: 3 application instances behind load balancer
- [ ] Test: Normal operation (all healthy)
- [ ] Simulate: Kill one instance abruptly
- [ ] Measure: Time for load balancer to detect failure
- [ ] Measure: Time for traffic to stop going to dead instance
- [ ] Measure: Impact on users (errors, latency)
- [ ] Verify: Other instances handle increased load
- [ ] Restore: Dead instance
- [ ] Measure: Time for instance to rejoin pool
- [ ] Document: Failure scenario và recovery

### Failure Simulation 2: Database Failure
- [ ] Setup: Application với database
- [ ] Test: Normal operation
- [ ] Simulate: Database connection failure
- [ ] Verify: Readiness check fails
- [ ] Verify: Load balancer removes instance from pool
- [ ] Verify: Circuit breaker opens (nếu có)
- [ ] Restore: Database connection
- [ ] Verify: Readiness check passes
- [ ] Verify: Instance rejoins pool
- [ ] Measure: Total downtime
- [ ] Document: Database failure handling

### Failure Simulation 3: Partial Failure
- [ ] Setup: Multi-instance application
- [ ] Simulate: One instance slow (add artificial delay)
- [ ] Verify: Load balancer behavior (least connections?)
- [ ] Measure: Impact on overall latency
- [ ] Simulate: One instance high error rate (50%)
- [ ] Verify: Circuit breaker behavior
- [ ] Verify: Load balancer removes unhealthy instance
- [ ] Document: Partial failure handling

### Failure Simulation 4: Network Partition
- [ ] Setup: 2 instances, shared Redis
- [ ] Simulate: Network partition (instance 1 can't reach Redis)
- [ ] Verify: Instance 1 behavior (fail fast? retry?)
- [ ] Verify: Instance 2 continues working
- [ ] Verify: Load balancer removes instance 1
- [ ] Restore: Network connectivity
- [ ] Verify: Instance 1 recovers
- [ ] Document: Network partition handling

### Resilience Testing
- [ ] Implement: Chaos testing scenario
- [ ] Test: Random instance failures (kill random instance every 30s)
- [ ] Measure: System behavior under chaos
- [ ] Verify: System continues serving requests
- [ ] Measure: Error rate during chaos
- [ ] Measure: Latency during chaos
- [ ] Document: Resilience test results
- [ ] Identify: Weak points trong system
- [ ] Propose: Improvements để handle chaos better

---

## Analysis TODOs

### Analysis Task 1: Scaling Strategy Decision
- [ ] Scenario: Current system handles 1K QPS, need to handle 10K QPS
- [ ] Option 1: Vertical scaling (bigger server)
- [ ] Calculate: Cost của vertical scaling
- [ ] Identify: Limitations (max server size)
- [ ] Identify: Single point of failure risk
- [ ] Option 2: Horizontal scaling (more servers)
- [ ] Calculate: Cost của horizontal scaling
- [ ] Identify: Complexity added
- [ ] Identify: Load balancer requirement
- [ ] Compare: Vertical vs Horizontal (cost, complexity, risk, scalability)
- [ ] Recommend: Which strategy? Justify với numbers
- [ ] Document: Decision analysis (500 words)

### Analysis Task 2: Load Balancer Algorithm Selection
- [ ] Scenario: Payment API, different servers have different capacities
- [ ] Analyze: Round-robin suitability
- [ ] Analyze: Weighted round-robin suitability
- [ ] Analyze: Least connections suitability
- [ ] Analyze: IP hash suitability (sticky sessions needed?)
- [ ] Test: Each algorithm với realistic load
- [ ] Measure: Load distribution (even? uneven?)
- [ ] Measure: Latency impact
- [ ] Recommend: Best algorithm cho scenario
- [ ] Justify: Recommendation với data
- [ ] Document: Algorithm selection analysis

### Analysis Task 3: Stateless vs Stateful Trade-off
- [ ] Scenario: E-commerce cart system
- [ ] Option 1: Stateless (cart in Redis)
- [ ] Analyze: Pros (3 points)
- [ ] Analyze: Cons (3 points)
- [ ] Analyze: Complexity
- [ ] Option 2: Stateful (cart in server memory)
- [ ] Analyze: Pros (3 points)
- [ ] Analyze: Cons (3 points)
- [ ] Analyze: Scalability limitations
- [ ] Compare: Stateless vs Stateful
- [ ] Recommend: Which approach? Justify
- [ ] Document: Trade-off analysis (500 words)

### Analysis Task 4: Availability Calculation
- [ ] System có 5 components: Load Balancer (99.9%), App Server (99.9%), Database (99.95%), Cache (99.9%), External API (99%)
- [ ] Calculate: Overall availability (all in series)
- [ ] Scenario: App Servers có 3 instances (parallel, each 99.9%)
- [ ] Calculate: App Server cluster availability
- [ ] Scenario: Database có Master-Slave (active-passive)
- [ ] Calculate: Database availability
- [ ] Recalculate: Overall system availability với redundancy
- [ ] Analyze: Which component is weakest link?
- [ ] Propose: How to improve weakest component?
- [ ] Calculate: New overall availability after improvement
- [ ] Document: Availability analysis và improvements

### Analysis Task 5: Capacity Planning
- [ ] Current: 1K QPS, 2 servers (500 QPS each)
- [ ] Growth: 50% per month
- [ ] Calculate: QPS after 3 months
- [ ] Calculate: QPS after 6 months
- [ ] Calculate: QPS after 12 months
- [ ] Plan: When to add servers? (trigger points)
- [ ] Calculate: Server count needed each month
- [ ] Calculate: Cost projection (monthly)
- [ ] Design: Auto-scaling rules (min, max, scale-up threshold, scale-down threshold)
- [ ] Estimate: Headroom needed (20% buffer)
- [ ] Document: Capacity planning và scaling plan

### Analysis Task 6: Deployment Strategy Comparison
- [ ] Compare: Blue-Green vs Canary vs Rolling deployment
- [ ] Criteria: Downtime, Risk, Cost, Complexity, Rollback speed
- [ ] Score: Each strategy on each criteria (1-10)
- [ ] Analyze: Best use case cho Blue-Green
- [ ] Analyze: Best use case cho Canary
- [ ] Analyze: Best use case cho Rolling
- [ ] Scenario: Critical payment system
- [ ] Recommend: Deployment strategy
- [ ] Justify: Recommendation
- [ ] Document: Deployment strategy analysis

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành chưa?
- [ ] Review: Tất cả Design exercises hoàn thành chưa?
- [ ] Review: Tất cả Coding tasks đã code và test chưa?
- [ ] Review: Tất cả Failure simulations đã run chưa?
- [ ] Review: Tất cả Analysis tasks đã complete chưa?
- [ ] Rate: Understanding của horizontal scaling (1-10)
- [ ] Rate: Understanding của load balancing (1-10)
- [ ] Rate: Understanding của stateless design (1-10)
- [ ] Rate: Understanding của high availability (1-10)
- [ ] Rate: Practical skills (implementation) (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve 3 weak areas

### Architecture Review
- [ ] Review: Payment Gateway design
- [ ] Check: Horizontal scaling strategy có realistic không?
- [ ] Check: Load balancer configuration đúng chưa?
- [ ] Check: Stateless design đúng chưa? (no session state?)
- [ ] Check: SPOFs eliminated chưa?
- [ ] Identify: 3 weaknesses trong design
- [ ] Propose: Improvements cho weaknesses
- [ ] Compare: Design vs industry best practices
- [ ] Document: Architecture review findings

### Code Review
- [ ] Review: Stateless application code
- [ ] Check: No session state trong code?
- [ ] Check: JWT implementation correct?
- [ ] Review: Load balancer implementation
- [ ] Check: Algorithm implementation correct?
- [ ] Check: Health check logic correct?
- [ ] Review: Circuit breaker implementation
- [ ] Check: Configuration correct?
- [ ] Check: Fallback mechanism works?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Load test results
- [ ] Review: Failure simulation results
- [ ] Measure: System performance under normal load
- [ ] Measure: System performance under failure scenarios
- [ ] Identify: Performance bottlenecks
- [ ] Verify: Scaling works as expected?
- [ ] Document: Performance analysis
- [ ] Create: Performance improvement plan

### Knowledge Check
- [ ] Explain: Horizontal vs Vertical scaling (viết 1 paragraph, không xem notes)
- [ ] Explain: Load balancer algorithms (viết comparison, không xem notes)
- [ ] Explain: Stateless vs Stateful (viết 1 paragraph, không xem notes)
- [ ] Explain: Active-Active vs Active-Passive (viết comparison, không xem notes)
- [ ] Explain: Zero-downtime deployment strategies (viết 1 paragraph, không xem notes)
- [ ] Solve: System có 3 components (99%, 99.9%, 99.99%), 2 app servers parallel (99.9% each), tính overall availability
- [ ] Solve: Current 1K QPS, growth 30%/month, cần capacity sau 6 tháng? (với 20% headroom)
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 concepts còn confuse
- [ ] Write: 1 mistake đã làm và lesson learned
- [ ] Write: Confidence level cho Week 3 (1-10)
- [ ] Compare: Week 1 vs Week 2 progress
- [ ] Plan: Preparation cho Week 3 (Database Design)
- [ ] Set: Goals cho Week 3
- [ ] Document: Week 2 reflection (500 words)

### Mentor Questions (Answer these - be honest)
- [ ] Q1: Bạn có thể scale một stateful service horizontally không? Tại sao? (viết analysis)
- [ ] Q2: Load balancer với sticky sessions - có phải stateless không? Giải thích. (viết answer)
- [ ] Q3: Active-Active setup có eliminate SPOF không? Tại sao? (viết analysis)
- [ ] Q4: Zero-downtime deployment có nghĩa là zero risk không? Giải thích. (viết answer)
- [ ] Q5: Nếu bạn có 99.9% availability nhưng vẫn bị complain, có thể do gì? (viết analysis)
- [ ] Q6: Horizontal scaling có giới hạn không? Giới hạn là gì? (viết answer)
- [ ] Review: Answers có đủ depth và critical thinking chưa?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Design TODOs: ✅ Complete với diagrams
- [ ] Tất cả Coding TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Failure & Resilience TODOs: ✅ Complete với results
- [ ] Tất cả Analysis TODOs: ✅ Complete với documentation
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Design diagrams saved: ✅ Yes
- [ ] Ready for Week 3: ✅ Yes

---

> **Mentor Final Check**: Tuần 2 này bạn phải MASTER scaling và high availability. Nếu bạn chỉ hiểu lý thuyết mà không implement được, bạn chưa sẵn sàng. Hãy honest: Bạn có thể design và implement một hệ thống scalable, highly available chưa? Nếu chưa, làm lại.
