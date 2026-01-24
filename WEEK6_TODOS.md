# Week 6 – Load Balancing & API Design

> **Mentor Note**: API design và load balancing là foundation của mọi distributed system. Một API design sai sẽ cost bạn years of technical debt. Một load balancing config sai sẽ cause cascading failures. Bạn phải think như principal engineer - challenge mọi assumption, question mọi decision. Không có "it works" - phải có "it works correctly under all conditions". Production APIs need to be robust, scalable, và maintainable.

---

## Study TODOs

### Load Balancing Fundamentals
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 5 - Replication (load balancing sections)
- [ ] Định nghĩa chính xác: Load balancing
- [ ] Định nghĩa chính xác: Reverse proxy
- [ ] Định nghĩa chính xác: API Gateway
- [ ] So sánh: Load balancer vs Reverse proxy vs API Gateway (5 differences)
- [ ] Đọc về "Layer 4 (L4) load balancing" - how it works
- [ ] Đọc về "Layer 7 (L7) load balancing" - how it works
- [ ] So sánh: L4 vs L7 load balancing (10 criteria)
- [ ] Analyze: When to use L4? When to use L7? (5 use cases each)
- [ ] Research: Popular load balancers (Nginx, HAProxy, AWS ALB, F5)

### Load Balancing Algorithms
- [ ] Đọc về "Round-Robin" algorithm
- [ ] Viết algorithm pseudocode cho Round-Robin
- [ ] Analyze: Pros và cons (5 points each)
- [ ] Đọc về "Weighted Round-Robin" algorithm
- [ ] Viết algorithm pseudocode cho Weighted Round-Robin
- [ ] Analyze: Pros và cons (5 points each)
- [ ] Đọc về "Least Connections" algorithm
- [ ] Viết algorithm pseudocode cho Least Connections
- [ ] Analyze: Pros và cons (5 points each)
- [ ] Đọc về "IP Hash" algorithm
- [ ] Viết algorithm pseudocode cho IP Hash
- [ ] Analyze: Pros và cons (5 points each)
- [ ] Đọc về "Least Response Time" algorithm
- [ ] Compare: All algorithms (10 criteria)
- [ ] Research: Real-world usage của mỗi algorithm

### Health Checks & Failover
- [ ] Đọc về "health checks" trong load balancers
- [ ] Đọc về "active health checks" - how it works
- [ ] Đọc về "passive health checks" - how it works
- [ ] So sánh: Active vs Passive health checks (5 criteria)
- [ ] Đọc về "health check intervals" - how often?
- [ ] Đọc về "health check timeout" - how long?
- [ ] Đọc về "health check thresholds" - success/failure counts
- [ ] Đọc về "graceful degradation" - remove unhealthy backends
- [ ] Đọc về "failover time" - how fast?
- [ ] Research: Health check best practices

### Sticky Sessions
- [ ] Đọc về "sticky sessions" (session affinity)
- [ ] Đọc về "cookie-based sticky sessions"
- [ ] Đọc về "IP-based sticky sessions"
- [ ] Analyze: When sticky sessions needed? (5 use cases)
- [ ] Analyze: Trade-offs của sticky sessions (5 points)
- [ ] Đọc về "session persistence" - how it works
- [ ] Đọc về "sticky session problems" - what can go wrong?
- [ ] Research: Alternatives to sticky sessions (stateless design)

### API Design Principles
- [ ] Đọc về "RESTful API design" principles
- [ ] Đọc về "REST constraints" - 6 constraints
- [ ] Đọc về "HTTP methods" - GET, POST, PUT, PATCH, DELETE semantics
- [ ] Đọc về "idempotency" - what is it? Why important?
- [ ] Đọc về "safety" - safe HTTP methods
- [ ] Đọc về "stateless" - REST stateless constraint
- [ ] Đọc về "resource-based URLs" - design principles
- [ ] Đọc về "HTTP status codes" - proper usage
- [ ] Research: RESTful API best practices

### API Versioning Strategies
- [ ] Đọc về "URL versioning" - /v1/, /v2/
- [ ] Analyze: Pros và cons của URL versioning (5 points each)
- [ ] Đọc về "Header versioning" - Accept: application/vnd.api+json;version=1
- [ ] Analyze: Pros và cons của Header versioning (5 points each)
- [ ] Đọc về "Query parameter versioning" - ?version=1
- [ ] Analyze: Pros và cons của Query versioning (5 points each)
- [ ] Đọc về "content negotiation" - versioning via content type
- [ ] Compare: All versioning strategies (10 criteria)
- [ ] Research: Real-world examples của mỗi strategy
- [ ] Analyze: When to use which strategy? (5 use cases each)

### Idempotency
- [ ] Đọc về "idempotency" - mathematical definition
- [ ] Đọc về "HTTP idempotency" - which methods are idempotent?
- [ ] Đọc về "idempotency keys" - how to implement
- [ ] Đọc về "idempotency storage" - where to store keys?
- [ ] Đọc về "idempotency TTL" - how long to store?
- [ ] Đọc về "idempotency scope" - per user? per request?
- [ ] Research: Idempotency implementation patterns
- [ ] Analyze: Idempotency trade-offs (performance, storage)

### Rate Limiting Deep Dive
- [ ] Đọc về "rate limiting" - why needed?
- [ ] Đọc về "Token Bucket" algorithm (review from Week 5)
- [ ] Đọc về "Sliding Window" algorithm (review from Week 5)
- [ ] Đọc về "Fixed Window" algorithm
- [ ] Đọc về "Leaky Bucket" algorithm
- [ ] Compare: All rate limiting algorithms (5 criteria)
- [ ] Đọc về "rate limiting headers" - X-RateLimit-*
- [ ] Đọc về "rate limiting strategies" - per user, per IP, per API key
- [ ] Đọc về "rate limiting tiers" - different limits for different users
- [ ] Research: Rate limiting best practices

### Backward Compatibility
- [ ] Đọc về "backward compatibility" - what is it?
- [ ] Đọc về "breaking changes" - what breaks compatibility?
- [ ] Đọc về "additive changes" - safe changes
- [ ] Đọc về "field deprecation" - how to deprecate safely
- [ ] Đọc về "API evolution" - how to evolve APIs
- [ ] Đọc về "versioning strategy" for backward compatibility
- [ ] Research: Backward compatibility best practices
- [ ] Analyze: Trade-offs của backward compatibility (maintenance cost)

### API Gateway Patterns
- [ ] Đọc về "API Gateway" pattern
- [ ] Đọc về "API Gateway responsibilities" - routing, auth, rate limiting, etc.
- [ ] Đọc về "BFF" (Backend for Frontend) pattern
- [ ] So sánh: API Gateway vs BFF (5 differences)
- [ ] Đọc về "service mesh" - alternative to API Gateway?
- [ ] Research: Popular API Gateway solutions (Kong, AWS API Gateway, Spring Cloud Gateway)

---

## Traffic Routing TODOs

### Design Exercise 1: Load Balancer Architecture
- [ ] Design: Load balancer architecture cho Payment API
- [ ] Requirement: 10K QPS, 5 backend servers, 99.9% availability
- [ ] Choose: L4 or L7 load balancer? Justify
- [ ] Choose: Load balancing algorithm? Justify
- [ ] Design: Health check strategy (interval, timeout, thresholds)
- [ ] Design: Failover strategy (automatic removal, re-addition)
- [ ] Design: Sticky sessions? If yes, justify. If no, justify.
- [ ] Calculate: Load per backend server
- [ ] Design: Monitoring strategy
- [ ] Document: Complete load balancer design (500 words)

### Design Exercise 2: Multi-Tier Load Balancing
- [ ] Design: Multi-tier load balancing architecture
- [ ] Tier 1: External load balancer (ALB/Nginx)
- [ ] Tier 2: Internal load balancer (application layer)
- [ ] Design: Traffic flow through tiers
- [ ] Design: Health check at each tier
- [ ] Analyze: Single point of failure at each tier
- [ ] Design: Redundancy strategy
- [ ] Document: Multi-tier architecture (500 words)

### Design Exercise 3: Health Check Strategy
- [ ] Design: Health check strategy cho microservices
- [ ] Requirement: Fast failure detection (< 5 seconds)
- [ ] Design: Health check endpoint (/health)
- [ ] Design: Liveness probe vs Readiness probe
- [ ] Design: Health check interval và timeout
- [ ] Design: Success/failure thresholds
- [ ] Design: Graceful shutdown integration
- [ ] Test: Health check với various failure scenarios
- [ ] Document: Health check strategy

### Design Exercise 4: Sticky Session Strategy
- [ ] Scenario: E-commerce cart, need sticky sessions
- [ ] Design: Sticky session implementation
- [ ] Choose: Cookie-based or IP-based? Justify
- [ ] Design: Session persistence configuration
- [ ] Design: Fallback strategy (if sticky session fails)
- [ ] Analyze: Trade-offs của sticky sessions
- [ ] Design: Alternative - stateless cart (if possible)
- [ ] Compare: Sticky sessions vs stateless design
- [ ] Document: Session strategy với justification

### Design Exercise 5: Traffic Splitting
- [ ] Design: Traffic splitting strategy (A/B testing, canary)
- [ ] Scenario: Deploy new API version, route 10% traffic
- [ ] Design: Traffic routing rules
- [ ] Design: Monitoring cho each traffic split
- [ ] Design: Rollback strategy
- [ ] Design: Gradual rollout plan (10% → 50% → 100%)
- [ ] Document: Traffic splitting strategy

---

## API Design TODOs

### Design Exercise 1: RESTful Payment API
- [ ] Design: Complete RESTful API cho Payment System
- [ ] Design: Resource URLs (payments, merchants, transactions)
- [ ] Design: HTTP methods cho each operation
- [ ] Design: Request/response schemas
- [ ] Design: HTTP status codes cho each scenario
- [ ] Design: Error response format
- [ ] Design: Pagination strategy
- [ ] Design: Filtering và sorting
- [ ] Verify: RESTful principles compliance
- [ ] Document: Complete API specification

### Design Exercise 2: API Versioning Strategy
- [ ] Design: API versioning strategy cho Payment API
- [ ] Choose: Versioning approach (URL, Header, Query)? Justify
- [ ] Design: Version lifecycle (deprecation, sunset)
- [ ] Design: Backward compatibility policy
- [ ] Design: Breaking change policy
- [ ] Design: Migration path for clients
- [ ] Design: Version documentation strategy
- [ ] Document: Versioning strategy (500 words)

### Design Exercise 3: Idempotent API Design
- [ ] Design: Idempotent payment creation API
- [ ] Design: Idempotency key mechanism
- [ ] Design: Idempotency key format
- [ ] Design: Idempotency key storage (Redis? Database?)
- [ ] Design: Idempotency key TTL
- [ ] Design: Duplicate request handling
- [ ] Design: Response for duplicate requests
- [ ] Test: Idempotency với concurrent requests
- [ ] Document: Idempotency implementation

### Design Exercise 4: Rate Limiting Strategy
- [ ] Design: Rate limiting strategy cho Payment API
- [ ] Design: Rate limits per merchant tier (Basic: 100/min, Premium: 1000/min)
- [ ] Design: Rate limiting algorithm (Token Bucket? Sliding Window?)
- [ ] Design: Rate limit headers (X-RateLimit-*)
- [ ] Design: Rate limit error response (429 Too Many Requests)
- [ ] Design: Rate limit storage (Redis?)
- [ ] Design: Distributed rate limiting (across multiple gateways)
- [ ] Document: Rate limiting strategy

### Design Exercise 5: Pagination Design
- [ ] Design: Pagination strategy cho transaction list API
- [ ] Requirement: Support 1M+ transactions
- [ ] Compare: Offset-based vs Cursor-based pagination
- [ ] Choose: Pagination approach? Justify
- [ ] Design: Pagination parameters
- [ ] Design: Pagination response format
- [ ] Design: Performance optimization (indexes, query optimization)
- [ ] Document: Pagination design

### Design Exercise 6: Error Handling Design
- [ ] Design: Error response format
- [ ] Design: Error codes và messages
- [ ] Design: HTTP status code mapping
- [ ] Design: Validation error format
- [ ] Design: Business logic error format
- [ ] Design: System error format (500 errors)
- [ ] Design: Error logging strategy
- [ ] Document: Error handling design

### Design Exercise 7: Backward Compatibility Plan
- [ ] Scenario: Need to add new field to Payment response
- [ ] Design: Backward compatible change
- [ ] Scenario: Need to remove deprecated field
- [ ] Design: Deprecation strategy
- [ ] Scenario: Need to change field type
- [ ] Design: Breaking change migration plan
- [ ] Design: Client communication strategy
- [ ] Document: Backward compatibility strategy

---

## Gateway & Spring Boot TODOs

### Task 1: Spring Cloud Gateway Setup
- [ ] Create: Spring Boot project với Spring Cloud Gateway
- [ ] Configure: Gateway routes
- [ ] Configure: Route to multiple backend services
- [ ] Test: Basic routing functionality
- [ ] Configure: Gateway port và context path
- [ ] Document: Gateway setup

### Task 2: Load Balancing với Gateway
- [ ] Configure: Multiple instances của backend service
- [ ] Configure: Load balancing trong Gateway (Round-robin)
- [ ] Test: Requests distributed across instances
- [ ] Configure: Health checks cho backend services
- [ ] Test: Unhealthy instance removed from pool
- [ ] Test: Healthy instance re-added to pool
- [ ] Measure: Load distribution
- [ ] Document: Load balancing configuration

### Task 3: Rate Limiting Filter
- [ ] Implement: Rate limiting filter trong Gateway
- [ ] Algorithm: Token Bucket hoặc Sliding Window
- [ ] Configure: Rate limits per route
- [ ] Configure: Rate limits per user/IP
- [ ] Test: Rate limit enforcement
- [ ] Test: Rate limit headers (X-RateLimit-*)
- [ ] Test: 429 response khi limit exceeded
- [ ] Measure: Rate limiting overhead
- [ ] Document: Rate limiting implementation

### Task 4: Authentication & Authorization Filter
- [ ] Implement: JWT authentication filter
- [ ] Validate: JWT token trong Gateway
- [ ] Extract: User information từ token
- [ ] Add: User info to request headers
- [ ] Implement: Authorization checks (role-based)
- [ ] Test: Unauthenticated requests rejected
- [ ] Test: Unauthorized requests rejected
- [ ] Document: Auth filter implementation

### Task 5: Request/Response Transformation
- [ ] Implement: Request transformation filter
- [ ] Transform: Request headers
- [ ] Transform: Request body (if needed)
- [ ] Implement: Response transformation filter
- [ ] Transform: Response headers
- [ ] Transform: Response body (if needed)
- [ ] Test: Transformations work correctly
- [ ] Document: Transformation filters

### Task 6: Circuit Breaker Integration
- [ ] Add: Resilience4j circuit breaker dependency
- [ ] Configure: Circuit breaker cho backend routes
- [ ] Configure: Failure threshold và timeout
- [ ] Configure: Fallback responses
- [ ] Test: Circuit opens on failures
- [ ] Test: Fallback responses returned
- [ ] Test: Circuit closes after recovery
- [ ] Document: Circuit breaker configuration

### Task 7: API Versioning với Gateway
- [ ] Design: API versioning strategy
- [ ] Implement: Version routing (URL-based: /v1/, /v2/)
- [ ] Configure: Routes cho each version
- [ ] Test: Version routing works
- [ ] Implement: Header-based versioning (alternative)
- [ ] Compare: URL vs Header versioning implementation
- [ ] Document: Versioning implementation

### Task 8: Request Logging & Monitoring
- [ ] Implement: Request logging filter
- [ ] Log: Request method, URL, headers, body
- [ ] Log: Response status, latency
- [ ] Add: Correlation ID to requests
- [ ] Integrate: Với Micrometer metrics
- [ ] Expose: Gateway metrics (request count, latency, errors)
- [ ] Create: Dashboard với key metrics
- [ ] Document: Logging và monitoring setup

### Task 9: Idempotency Filter
- [ ] Implement: Idempotency filter trong Gateway
- [ ] Extract: Idempotency key từ header (Idempotency-Key)
- [ ] Check: Idempotency key trong Redis
- [ ] Store: Idempotency key với response
- [ ] Return: Cached response nếu duplicate request
- [ ] Configure: Idempotency key TTL
- [ ] Test: Duplicate requests return same response
- [ ] Document: Idempotency filter implementation

### Task 10: Health Check Endpoint
- [ ] Implement: Gateway health check endpoint
- [ ] Check: Gateway itself health
- [ ] Check: Backend services health (aggregate)
- [ ] Return: Overall health status
- [ ] Integrate: Với load balancer health checks
- [ ] Test: Health check endpoint
- [ ] Document: Health check implementation

---

## Reliability TODOs

### Failure Scenario 1: Backend Service Failure
- [ ] Simulate: Backend service failure
- [ ] Test: Load balancer removes unhealthy backend
- [ ] Measure: Failover time
- [ ] Test: Traffic redistributed to healthy backends
- [ ] Test: Circuit breaker behavior
- [ ] Test: Fallback responses
- [ ] Measure: Impact on users (errors, latency)
- [ ] Document: Failure handling

### Failure Scenario 2: Load Balancer Failure
- [ ] Simulate: Primary load balancer failure
- [ ] Design: Load balancer redundancy
- [ ] Test: Failover to secondary load balancer
- [ ] Measure: Failover time
- [ ] Test: DNS failover (if applicable)
- [ ] Test: Health check failover
- [ ] Document: Load balancer failover strategy

### Failure Scenario 3: Partial Backend Failure
- [ ] Simulate: One backend slow (high latency)
- [ ] Test: Load balancer behavior (Least Connections?)
- [ ] Measure: Impact on overall latency
- [ ] Test: Circuit breaker opens for slow backend
- [ ] Test: Traffic stops going to slow backend
- [ ] Document: Partial failure handling

### Failure Scenario 4: Rate Limit Exceeded
- [ ] Simulate: Rate limit exceeded scenario
- [ ] Test: 429 response returned
- [ ] Test: Rate limit headers included
- [ ] Test: Retry-After header (if applicable)
- [ ] Test: Client retry behavior
- [ ] Measure: Impact on legitimate users
- [ ] Document: Rate limit handling

### Failure Scenario 5: Sticky Session Failure
- [ ] Simulate: Backend với sticky session fails
- [ ] Test: Session lost? User impact?
- [ ] Design: Session recovery strategy
- [ ] Test: Alternative - stateless design
- [ ] Compare: Sticky sessions vs stateless
- [ ] Document: Session failure handling

### Failure Scenario 6: Health Check False Positive
- [ ] Simulate: Health check returns false positive (healthy but actually failing)
- [ ] Test: Impact on system
- [ ] Design: Better health check (more comprehensive)
- [ ] Test: Improved health check
- [ ] Document: Health check improvements

### Load Testing
- [ ] Setup: Load testing tool (JMeter, Gatling)
- [ ] Test: Load balancer với increasing load (1K, 5K, 10K QPS)
- [ ] Measure: Latency at each load level
- [ ] Measure: Error rate at each load level
- [ ] Identify: Breaking point
- [ ] Identify: Bottlenecks
- [ ] Optimize: Configuration
- [ ] Re-test: After optimization
- [ ] Document: Load test results

### Stress Testing
- [ ] Test: System behavior under extreme load (2x normal)
- [ ] Measure: Degradation patterns
- [ ] Test: Recovery after load spike
- [ ] Identify: Failure modes
- [ ] Document: Stress test results

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Traffic Routing exercises hoàn thành?
- [ ] Review: Tất cả API Design exercises hoàn thành?
- [ ] Review: Tất cả Gateway tasks hoàn thành?
- [ ] Review: Tất cả Reliability scenarios tested?
- [ ] Rate: Load balancing understanding (1-10)
- [ ] Rate: API design skills (1-10)
- [ ] Rate: Gateway implementation skills (1-10)
- [ ] Rate: Reliability engineering skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### API Design Review
- [ ] Review: Payment API design
- [ ] Check: RESTful principles compliance?
- [ ] Check: Versioning strategy appropriate?
- [ ] Check: Idempotency implemented?
- [ ] Check: Error handling adequate?
- [ ] Check: Backward compatibility considered?
- [ ] Identify: 3 API design improvements
- [ ] Propose: API design optimizations
- [ ] Compare: Design vs industry best practices
- [ ] Document: API design review findings

### Architecture Review
- [ ] Review: Load balancer architecture
- [ ] Check: Single points of failure eliminated?
- [ ] Check: Health checks adequate?
- [ ] Check: Failover strategy robust?
- [ ] Check: Monitoring comprehensive?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Document: Architecture review

### Code Review
- [ ] Review: Gateway implementation code
- [ ] Check: Error handling adequate?
- [ ] Check: Logging và monitoring?
- [ ] Check: Configuration management?
- [ ] Review: Rate limiting implementation
- [ ] Review: Idempotency implementation
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Load test results
- [ ] Review: Stress test results
- [ ] Verify: Performance targets met?
- [ ] Identify: Performance bottlenecks
- [ ] Analyze: Load balancer overhead
- [ ] Analyze: Gateway overhead
- [ ] Create: Performance improvement plan
- [ ] Document: Performance review

### Knowledge Check
- [ ] Explain: L4 vs L7 load balancing (viết comparison, không xem notes)
- [ ] Explain: Round-Robin vs Least Connections (viết comparison, không xem notes)
- [ ] Explain: URL vs Header versioning (viết comparison, không xem notes)
- [ ] Explain: Idempotency - why important? (viết explanation, không xem notes)
- [ ] Explain: Rate limiting algorithms (viết comparison, không xem notes)
- [ ] Solve: 10K QPS, 5 backends, Round-Robin. Calculate QPS per backend
- [ ] Solve: Rate limit = 100/min, 120 requests in 1 minute. How many rejected?
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 API design concepts còn confuse
- [ ] Write: 1 architecture decision bạn made và rationale
- [ ] Write: Confidence level cho Week 7 (1-10)
- [ ] Compare: Week 5 vs Week 6 progress
- [ ] Plan: Preparation cho Week 7 (Message Queues)
- [ ] Set: Goals cho Week 7
- [ ] Document: Week 6 reflection (500 words)

### Mentor Questions (Answer these - challenge assumptions)
- [ ] Q1: Bạn design API với URL versioning (/v1/, /v2/). Client cũ không update, vẫn call /v1/. Bạn cần deprecate /v1/ sau 1 năm. Làm sao communicate và enforce? (viết deprecation strategy)
- [ ] Q2: Load balancer với Round-Robin, 5 backends. Backend 1 chậm (500ms), others fast (50ms). Average latency? Làm sao fix? (viết analysis và solution)
- [ ] Q3: API có rate limit 100/min. User có 10 parallel requests, mỗi request takes 1 second. Có violate rate limit không? Tại sao? (viết analysis)
- [ ] Q4: Sticky sessions - user A always goes to Backend 1. Backend 1 có 10x load của others. Làm sao fix? Justify solution. (viết solution với trade-off analysis)
- [ ] Q5: API Gateway adds 10ms latency. Worth it? Khi nào worth it? Khi nào không? (viết cost-benefit analysis)
- [ ] Q6: Idempotency key stored trong Redis, TTL = 24 hours. 1M requests/day, 1% duplicates. Redis memory needed? (calculate và analyze)
- [ ] Review: Answers có challenge assumptions? Có consider edge cases? Có justify decisions?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Traffic Routing TODOs: ✅ Complete với designs
- [ ] Tất cả API Design TODOs: ✅ Complete với specifications
- [ ] Tất cả Gateway & Spring Boot TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Reliability TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] API specifications saved: ✅ Yes
- [ ] Load test results saved: ✅ Yes
- [ ] Ready for Week 7: ✅ Yes

---

> **Mentor Final Check**: API design và load balancing là foundation. Nếu foundation weak, everything trên đó sẽ be weak. Hãy honest: Bạn có thể design production-ready API chưa? Bạn có thể configure load balancer cho high availability chưa? Bạn có thể handle all failure scenarios chưa? Nếu chưa, làm lại. Production systems need robust foundations, not "it works on my machine".
