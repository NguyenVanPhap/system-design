# Week 1 – Fundamentals: Scalability & Availability

> **Mentor Note**: Đây là TODO list nghiêm khắc. Mỗi item phải được hoàn thành 100%. Không có "gần như xong" hay "hiểu đại khái". Bạn phải CODE, MEASURE, và DOCUMENT.

---

## Study TODOs

### Scalability Concepts
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 1 - Scalability (pages 1-30)
- [ ] Viết notes: Định nghĩa chính xác của "scalability" trong 2 câu
- [ ] Liệt kê 5 điểm khác biệt giữa vertical và horizontal scaling
- [ ] Tìm 3 real-world examples của vertical scaling (và tại sao họ chọn)
- [ ] Tìm 3 real-world examples của horizontal scaling (và tại sao họ chọn)
- [ ] Đọc về "Amdahl's Law" và viết công thức + giải thích ý nghĩa
- [ ] Đọc về "Gustafson's Law" và so sánh với Amdahl's Law

### Performance Metrics
- [ ] Định nghĩa chính xác: Latency, Throughput, QPS, TPS, RPS
- [ ] Viết công thức tính: QPS = ?
- [ ] Viết công thức tính: Throughput = ?
- [ ] Đọc về percentile metrics (p50, p95, p99, p999)
- [ ] Tính toán: Nếu p95 latency = 200ms, có nghĩa là gì? (viết câu trả lời)
- [ ] Tìm hiểu: Tại sao p99 quan trọng hơn average latency?
- [ ] Đọc về "tail latency" và "latency SLOs"

### Availability Concepts
- [ ] Tính toán downtime cho: 99%, 99.9%, 99.99%, 99.999% (theo năm, tháng, tuần, ngày)
- [ ] Viết bảng so sánh: Availability % → Downtime/year → Downtime/month
- [ ] Đọc về "nines" trong availability (3 nines, 4 nines, 5 nines)
- [ ] Định nghĩa: Single Point of Failure (SPOF)
- [ ] Định nghĩa: Mean Time Between Failures (MTBF)
- [ ] Định nghĩa: Mean Time To Recovery (MTTR)
- [ ] Công thức: Availability = MTBF / (MTBF + MTTR) - verify và hiểu

### Redundancy Patterns
- [ ] Đọc về Active-Active redundancy pattern
- [ ] Đọc về Active-Passive (Hot Standby) redundancy pattern
- [ ] Đọc về Active-Passive (Cold Standby) redundancy pattern
- [ ] So sánh: Active-Active vs Active-Passive (3 điểm khác biệt)
- [ ] Tìm 2 real-world examples của Active-Active
- [ ] Tìm 2 real-world examples của Active-Passive

### Bottleneck Identification
- [ ] Liệt kê 4 loại bottlenecks chính: CPU, Memory, I/O, Network
- [ ] Với mỗi bottleneck, viết 2 cách identify
- [ ] Với mỗi bottleneck, viết 2 cách resolve
- [ ] Đọc về "Amdahl's Law" trong context của bottlenecks

### Capacity Planning
- [ ] Đọc về "back-of-envelope calculations"
- [ ] Học cách estimate: Storage requirements
- [ ] Học cách estimate: Bandwidth requirements
- [ ] Học cách estimate: Compute requirements
- [ ] Practice: Estimate storage cho 1M users, mỗi user 10MB data
- [ ] Practice: Estimate bandwidth cho 10K QPS, mỗi request 2KB response

---

## Design TODOs

### Design Exercise 1: Payment Gateway
- [ ] Thiết kế architecture diagram cho Payment Gateway (10K TPS, 99.9% uptime)
- [ ] Vẽ diagram với các components: API Gateway, Payment Service, Database, Cache
- [ ] Label mỗi component với estimated QPS capacity
- [ ] Identify ít nhất 3 SPOF trong design đầu tiên
- [ ] Redesign để eliminate tất cả SPOF
- [ ] Tính toán: Cần bao nhiêu instances cho mỗi service? (show calculations)
- [ ] Design redundancy strategy cho database
- [ ] Design redundancy strategy cho application servers
- [ ] Estimate: Total cost (rough) cho infrastructure
- [ ] Viết document (500 words) giải thích design decisions

### Design Exercise 2: Betting Platform
- [ ] Thiết kế architecture cho Betting Platform (100K concurrent users)
- [ ] Identify bottleneck trong design (CPU, Memory, I/O, Network - chọn 1)
- [ ] Design solution để resolve bottleneck đó
- [ ] Tính toán: Peak traffic = 10x normal, design để handle
- [ ] Design horizontal scaling strategy
- [ ] Design vertical scaling strategy (nếu cần)
- [ ] So sánh: Horizontal vs Vertical cho use case này (viết 3 điểm)
- [ ] Estimate: Latency cho mỗi component (p50, p95, p99)
- [ ] Identify: Component nào sẽ là bottleneck? Tại sao?
- [ ] Viết document (500 words) về scaling strategy

### Design Exercise 3: Load Estimation
- [ ] Scenario: E-commerce site, 1M daily active users
- [ ] Estimate: Peak QPS (assume 20% traffic trong 1 hour)
- [ ] Estimate: Average request size (assume 5KB)
- [ ] Estimate: Average response size (assume 10KB)
- [ ] Calculate: Total bandwidth requirement (Mbps)
- [ ] Estimate: Database size (assume 1GB per 10K users)
- [ ] Calculate: Storage growth rate (per month)
- [ ] Estimate: Cache size needed (assume 20% of DB size)
- [ ] Create spreadsheet với tất cả calculations
- [ ] Verify: Tất cả numbers có hợp lý không? (write validation)

---

## Coding TODOs

### Task 1: Spring Boot Performance App
- [ ] Tạo Spring Boot project mới
- [ ] Tạo REST API endpoint: `GET /api/users/{id}` (return mock user data)
- [ ] Tạo REST API endpoint: `POST /api/users` (create user, save to in-memory list)
- [ ] Add logging: Log request time cho mỗi endpoint
- [ ] Add metrics: Count requests, measure latency
- [ ] Run app và test với 100 requests (manual hoặc script)
- [ ] Measure: Average latency, p95 latency
- [ ] Document: Performance baseline

### Task 2: Load Testing Setup
- [ ] Install JMeter hoặc Gatling
- [ ] Tạo test plan cho `/api/users/{id}` endpoint
- [ ] Configure: 100 concurrent users, 1000 total requests
- [ ] Run load test
- [ ] Export results: QPS, latency (p50, p95, p99), error rate
- [ ] Identify: At what load does latency spike?
- [ ] Identify: At what load do errors start?
- [ ] Document: Performance limits của app hiện tại
- [ ] Tạo report với charts (response time, throughput)

### Task 3: Performance Profiling
- [ ] Install VisualVM hoặc JProfiler
- [ ] Attach profiler to Spring Boot app
- [ ] Run load test while profiling
- [ ] Identify: Top 5 methods by CPU time
- [ ] Identify: Memory allocation hotspots
- [ ] Check: Memory leaks (heap growth over time)
- [ ] Check: Thread contention
- [ ] Document: 3 performance issues found
- [ ] Propose: 3 optimizations (không cần implement, chỉ propose)

### Task 4: Simple Load Balancer
- [ ] Tạo Java class: `SimpleLoadBalancer`
- [ ] Implement: Round-robin algorithm
- [ ] Add: List of backend servers (hardcoded URLs)
- [ ] Add: `getNextServer()` method
- [ ] Add: Health check mechanism (ping endpoint)
- [ ] Add: Skip unhealthy servers
- [ ] Test: With 3 mock backend servers
- [ ] Test: Mark one server unhealthy, verify it's skipped
- [ ] Measure: Overhead của load balancer (latency added)
- [ ] Document: Code và test results

### Task 5: Health Check Endpoints
- [ ] Add endpoint: `GET /health` (basic health check)
- [ ] Add endpoint: `GET /health/readiness` (readiness probe)
- [ ] Add endpoint: `GET /health/liveness` (liveness probe)
- [ ] Implement: Database connection check trong readiness
- [ ] Implement: Memory check trong liveness (fail if memory > 90%)
- [ ] Test: All health endpoints return correct status
- [ ] Test: Simulate DB down, verify readiness fails
- [ ] Test: Simulate high memory, verify liveness fails
- [ ] Document: Khi nào dùng readiness vs liveness

### Task 6: Metrics Collection
- [ ] Add Micrometer dependency
- [ ] Expose metrics endpoint: `GET /actuator/metrics`
- [ ] Add custom metric: `requests.total` (counter)
- [ ] Add custom metric: `request.duration` (timer)
- [ ] Instrument: All API endpoints với metrics
- [ ] Verify: Metrics được update correctly
- [ ] Export: Metrics to Prometheus format (optional)
- [ ] Document: Metrics available và ý nghĩa

---

## Analysis TODOs

### Analysis Task 1: Bottleneck Analysis
- [ ] Chọn một Spring Boot app hiện tại (hoặc tạo simple one)
- [ ] Run load test với increasing load: 10, 50, 100, 500, 1000 concurrent users
- [ ] Measure: Latency, throughput, error rate cho mỗi load level
- [ ] Plot graph: Latency vs Load
- [ ] Plot graph: Throughput vs Load
- [ ] Identify: Breaking point (khi latency spikes)
- [ ] Identify: Type of bottleneck (CPU-bound, I/O-bound, Memory-bound)
- [ ] Analyze: Tại sao bottleneck xảy ra ở điểm đó?
- [ ] Propose: 3 solutions để resolve bottleneck
- [ ] Estimate: Improvement expected từ mỗi solution

### Analysis Task 2: Capacity Planning
- [ ] Scenario: Design system cho 10M users
- [ ] Estimate: Peak concurrent users (assume 10% of total)
- [ ] Estimate: Peak QPS (assume 5 requests/user/minute)
- [ ] Calculate: Database size (assume 1KB per user)
- [ ] Calculate: Cache size (assume 10% of DB)
- [ ] Calculate: Bandwidth (assume 5KB per request)
- [ ] Estimate: Server count (assume 1 server = 10K QPS)
- [ ] Estimate: Cost (rough, assume $100/server/month)
- [ ] Create: Capacity planning spreadsheet
- [ ] Validate: Tất cả assumptions có realistic không?

### Analysis Task 3: Availability Calculation
- [ ] Calculate: Downtime budget cho 99.9% availability (per year)
- [ ] Calculate: Downtime budget cho 99.99% availability (per year)
- [ ] Scenario: System có 5 components, mỗi component có 99.9% availability
- [ ] Calculate: Overall system availability (series)
- [ ] Scenario: System có 2 redundant components (parallel), mỗi 99.9%
- [ ] Calculate: Overall system availability (parallel)
- [ ] Analyze: Cần bao nhiêu nines để có < 1 hour downtime/year?
- [ ] Analyze: Nếu MTTR = 1 hour, cần MTBF = ? để đạt 99.99%?
- [ ] Create: Availability calculator spreadsheet
- [ ] Document: Findings và insights

### Analysis Task 4: Scaling Strategy Comparison
- [ ] Scenario: Payment API, current load = 1K QPS, expected = 10K QPS
- [ ] Option 1: Vertical scaling (bigger server)
- [ ] Calculate: Cost của vertical scaling
- [ ] Calculate: Limitations (max server size)
- [ ] Option 2: Horizontal scaling (more servers)
- [ ] Calculate: Cost của horizontal scaling
- [ ] Calculate: Complexity added
- [ ] Compare: Vertical vs Horizontal (cost, complexity, limits)
- [ ] Recommend: Which strategy? Tại sao?
- [ ] Document: Decision matrix

### Analysis Task 5: Performance Baseline
- [ ] Measure: Current app performance (baseline)
- [ ] Metrics: QPS, latency (p50, p95, p99), error rate
- [ ] Document: Baseline performance
- [ ] Set: Performance goals (improve by 2x)
- [ ] Identify: Bottleneck preventing goal achievement
- [ ] Propose: Optimization plan
- [ ] Estimate: Expected improvement từ mỗi optimization
- [ ] Prioritize: Optimizations by impact/effort
- [ ] Create: Performance improvement roadmap
- [ ] Document: Analysis và recommendations

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả study TODOs đã hoàn thành chưa?
- [ ] Review: Tất cả design exercises đã làm chưa?
- [ ] Review: Tất cả coding tasks đã code và test chưa?
- [ ] Review: Tất cả analysis tasks đã complete chưa?
- [ ] Rate yourself: Understanding của scalability (1-10)
- [ ] Rate yourself: Understanding của availability (1-10)
- [ ] Rate yourself: Practical skills (load testing, profiling) (1-10)
- [ ] Identify: 3 concepts bạn hiểu rõ nhất
- [ ] Identify: 3 concepts bạn còn confuse
- [ ] Plan: Làm sao để clarify 3 concepts còn confuse?

### Design Review
- [ ] Review Payment Gateway design
- [ ] Check: Có SPOF không?
- [ ] Check: Scaling strategy có realistic không?
- [ ] Check: Capacity estimates có hợp lý không?
- [ ] Identify: 3 weaknesses trong design
- [ ] Propose: Improvements cho 3 weaknesses
- [ ] Compare: Design của bạn vs best practices
- [ ] Document: Lessons learned

### Code Review
- [ ] Review: Code quality (clean code principles)
- [ ] Review: Error handling
- [ ] Review: Logging và monitoring
- [ ] Review: Performance optimizations
- [ ] Identify: 3 code improvements needed
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Load test results
- [ ] Review: Profiling results
- [ ] Identify: Top 3 performance issues
- [ ] Verify: Performance goals đã đạt chưa?
- [ ] Document: Performance analysis
- [ ] Create: Performance improvement plan

### Knowledge Check
- [ ] Explain: Vertical vs Horizontal scaling (viết 1 paragraph, không xem notes)
- [ ] Explain: Availability calculation (viết công thức và example)
- [ ] Explain: Bottleneck identification process (viết 5 steps)
- [ ] Explain: Capacity planning approach (viết 5 steps)
- [ ] Solve: System có 3 components (99%, 99.9%, 99.99%), tính overall availability
- [ ] Solve: Estimate QPS cho 1M users, 10% online, 2 requests/user/minute
- [ ] Verify: Answers của bạn có đúng không?
- [ ] Document: Knowledge gaps found

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 điều còn confuse hoặc cần học thêm
- [ ] Write: 1 mistake bạn đã làm và lesson learned
- [ ] Write: Confidence level cho Week 2 (1-10)
- [ ] Plan: Preparation cho Week 2 (Availability & Reliability)
- [ ] Set: Goals cho Week 2
- [ ] Document: Week 1 reflection (500 words)

### Mentor Questions (Answer these)
- [ ] Q1: Nếu bạn phải scale từ 1K QPS lên 100K QPS, bạn sẽ làm gì? (viết 5 steps)
- [ ] Q2: System có 99.9% availability nhưng vẫn bị complain về downtime. Tại sao? (viết analysis)
- [ ] Q3: Làm sao bạn identify bottleneck trong production system? (viết process)
- [ ] Q4: Vertical scaling có giới hạn không? Giới hạn là gì? (viết answer)
- [ ] Q5: Tại sao p99 latency quan trọng hơn average latency? (viết explanation)
- [ ] Review: Answers của bạn có đủ depth chưa?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Design TODOs: ✅ Complete
- [ ] Tất cả Coding TODOs: ✅ Complete và tested
- [ ] Tất cả Analysis TODOs: ✅ Complete với documentation
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed to repo: ✅ Yes
- [ ] Design diagrams saved: ✅ Yes
- [ ] Ready for Week 2: ✅ Yes

---

> **Mentor Final Check**: Nếu bạn check tất cả items trên, bạn đã sẵn sàng cho Week 2. Nếu không, bạn chưa sẵn sàng. Hãy honest với bản thân.
