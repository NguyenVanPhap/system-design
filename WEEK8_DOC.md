# Week 8 – Microservices Architecture & Service Governance

> **Mentor Note**: Microservices không phải là silver bullet. Nhiều teams split services mà không có lý do rõ ràng, tạo ra complexity không cần thiết. Bạn phải justify MỌI service split. Nếu bạn không thể explain tại sao service A phải tách khỏi service B, có thể chúng không nên tách. Platform architect phải challenge assumptions, not follow trends. Production systems need justified architecture, not microservices for the sake of microservices.

---

## Study TODOs

### Microservices vs Monolith
- [ ] Đọc "Building Microservices" by Sam Newman - Chapter 1-2
- [ ] Định nghĩa chính xác: Monolith
- [ ] Định nghĩa chính xác: Microservices
- [ ] Đọc về "monolith advantages" - 10 advantages
- [ ] Đọc về "monolith disadvantages" - 10 disadvantages
- [ ] Đọc về "microservices advantages" - 10 advantages
- [ ] Đọc về "microservices disadvantages" - 10 disadvantages
- [ ] Compare: Monolith vs Microservices (15 criteria)
- [ ] Research: When to start with monolith? (5 indicators)
- [ ] Research: When to split to microservices? (5 indicators)
- [ ] Đọc về "microservices prerequisites" - what you need first

### Service Decomposition Strategies
- [ ] Đọc về "domain-driven design" (DDD) - bounded contexts
- [ ] Đọc về "bounded context" - service boundaries
- [ ] Đọc về "decomposition by business capability"
- [ ] Đọc về "decomposition by subdomain"
- [ ] Đọc về "decomposition by data"
- [ ] Compare: Decomposition strategies (10 criteria)
- [ ] Đọc về "shared database anti-pattern"
- [ ] Đọc về "service coupling" - tight vs loose coupling
- [ ] Đọc về "cohesion" - high cohesion principle
- [ ] Research: Service decomposition best practices

### Service Boundaries
- [ ] Đọc về "service boundaries" - how to define?
- [ ] Đọc về "single responsibility principle" - applied to services
- [ ] Đọc về "data ownership" - each service owns its data
- [ ] Đọc về "shared data anti-pattern"
- [ ] Đọc về "service autonomy" - independent deployment
- [ ] Đọc về "service size" - how big? how small?
- [ ] Research: Service size guidelines (team size, code size)
- [ ] Đọc về "nanoservices anti-pattern" - too small services

### API Gateway Patterns
- [ ] Đọc về "API Gateway" pattern (review from Week 6)
- [ ] Đọc về "BFF" (Backend for Frontend) pattern
- [ ] So sánh: API Gateway vs BFF (10 criteria)
- [ ] Đọc về "service mesh" - alternative to API Gateway?
- [ ] Đọc về "API Gateway responsibilities" - routing, auth, rate limiting
- [ ] Đọc về "API Gateway scalability" - single point of failure?
- [ ] Research: API Gateway solutions (Kong, AWS API Gateway, Spring Cloud Gateway)

### Service Discovery
- [ ] Đọc về "service discovery" - why needed?
- [ ] Đọc về "client-side discovery" - how it works
- [ ] Đọc về "server-side discovery" - how it works
- [ ] So sánh: Client-side vs Server-side discovery (10 criteria)
- [ ] Đọc về "service registry" - centralized registry
- [ ] Đọc về "self-registration" - services register themselves
- [ ] Đọc về "third-party registration" - registry discovers services
- [ ] Research: Service discovery solutions (Eureka, Consul, Kubernetes)

### Configuration Management
- [ ] Đọc về "configuration management" - why important?
- [ ] Đọc về "externalized configuration" - 12-factor app
- [ ] Đọc về "configuration server" - centralized config
- [ ] Đọc về "configuration versioning" - track config changes
- [ ] Đọc về "configuration refresh" - hot reload
- [ ] Đọc về "environment-specific configuration"
- [ ] Đọc về "secrets management" - sensitive config
- [ ] Research: Configuration management solutions (Spring Cloud Config, Consul, Vault)

### Inter-Service Communication
- [ ] Đọc về "synchronous communication" - REST, gRPC
- [ ] Đọc về "asynchronous communication" - message queues
- [ ] So sánh: Sync vs Async communication (10 criteria)
- [ ] Đọc về "REST over HTTP" - pros và cons
- [ ] Đọc về "gRPC" - when to use?
- [ ] Đọc về "message queues" - async communication (review from Week 7)
- [ ] Đọc về "service mesh" - service-to-service communication
- [ ] Research: Communication patterns best practices

### Circuit Breaker Pattern
- [ ] Đọc về "Circuit Breaker" pattern (review from Week 2)
- [ ] Đọc về "circuit states" - closed, open, half-open
- [ ] Đọc về "failure threshold" - when to open circuit
- [ ] Đọc về "timeout" - wait before retry
- [ ] Đọc về "fallback mechanism" - degraded response
- [ ] Đọc về "circuit breaker vs retry" - when to use which?
- [ ] Research: Circuit breaker implementations (Hystrix, Resilience4j, Sentinel)

### Distributed Tracing
- [ ] Đọc về "distributed tracing" - why needed?
- [ ] Đọc về "trace" - request journey across services
- [ ] Đọc về "span" - single operation in trace
- [ ] Đọc về "trace context" - correlation ID
- [ ] Đọc về "sampling" - reduce trace volume
- [ ] Đọc về "trace propagation" - pass context between services
- [ ] Research: Distributed tracing solutions (Zipkin, Jaeger, OpenTelemetry)

### Service Versioning
- [ ] Đọc về "service versioning" - why needed?
- [ ] Đọc về "backward compatibility" - version compatibility
- [ ] Đọc về "versioning strategies" - URL, header, content negotiation
- [ ] Đọc về "deprecation strategy" - how to deprecate versions
- [ ] Đọc về "coexistence" - multiple versions running
- [ ] Đọc về "version routing" - route to correct version
- [ ] Research: Service versioning best practices

### Service Mesh
- [ ] Đọc về "service mesh" - what is it?
- [ ] Đọc về "sidecar pattern" - proxy alongside service
- [ ] Đọc về "service mesh benefits" - traffic management, security
- [ ] Đọc về "Istio" - popular service mesh
- [ ] Đọc về "Linkerd" - alternative service mesh
- [ ] So sánh: Service mesh vs API Gateway (10 criteria)
- [ ] Research: When to use service mesh?

---

## Service Decomposition TODOs

### Decomposition Exercise 1: Monolith Analysis
- [ ] Scenario: Existing monolith application
- [ ] Analyze: Current monolith structure
- [ ] Identify: 5 reasons to keep as monolith
- [ ] Identify: 5 reasons to split to microservices
- [ ] Evaluate: Is split justified? Justify answer
- [ ] Document: Monolith analysis (500 words)

### Decomposition Exercise 2: Payment System Decomposition
- [ ] Design: Decompose Payment System into microservices
- [ ] Requirement: Payment processing, notifications, reporting
- [ ] Identify: Service boundaries (use DDD bounded contexts)
- [ ] Design: Service responsibilities
- [ ] Design: Data ownership (each service owns its data)
- [ ] Justify: Each service split (why separate services?)
- [ ] Challenge: Can any services be merged? Justify
- [ ] Document: Service decomposition design (1000 words)

### Decomposition Exercise 3: Betting Platform Decomposition
- [ ] Design: Decompose Betting Platform into microservices
- [ ] Requirement: Bet management, match management, settlement
- [ ] Apply: Domain-driven design principles
- [ ] Define: Bounded contexts
- [ ] Design: Service boundaries
- [ ] Design: Service communication (sync/async)
- [ ] Justify: Service decomposition decisions
- [ ] Document: Complete microservices architecture

### Decomposition Exercise 4: Service Size Analysis
- [ ] Scenario: Team wants to create 20 microservices
- [ ] Analyze: Is 20 services justified?
- [ ] Calculate: Team size needed (assume 2 developers per service)
- [ ] Calculate: Operational overhead (monitoring, deployment)
- [ ] Analyze: Communication complexity (N services = N*(N-1)/2 connections)
- [ ] Recommend: Optimal number of services? Justify
- [ ] Document: Service size analysis

### Decomposition Exercise 5: Shared Data Analysis
- [ ] Scenario: 3 services need same user data
- [ ] Analyze: Options (shared database? data duplication? user service?)
- [ ] Compare: All options (5 criteria each)
- [ ] Design: Best approach với justification
- [ ] Design: Data synchronization strategy (if needed)
- [ ] Document: Shared data strategy

### Decomposition Exercise 6: Anti-Patterns
- [ ] Identify: 5 microservices anti-patterns
- [ ] Analyze: Shared database anti-pattern
- [ ] Analyze: Distributed monolith anti-pattern
- [ ] Analyze: Nanoservices anti-pattern
- [ ] Analyze: Chatty services anti-pattern
- [ ] Design: Solutions cho each anti-pattern
- [ ] Document: Anti-patterns và solutions

---

## Spring Cloud TODOs

### Task 1: Service Discovery với Eureka
- [ ] Setup: Eureka Server (Spring Cloud Netflix)
- [ ] Create: Eureka Server application
- [ ] Configure: Eureka Server properties
- [ ] Create: 2 microservices (Payment Service, Notification Service)
- [ ] Configure: Services register với Eureka
- [ ] Test: Service registration
- [ ] Test: Service discovery (lookup service by name)
- [ ] Test: Service deregistration (when service stops)
- [ ] Document: Eureka setup

### Task 2: Inter-Service Communication
- [ ] Implement: REST communication between services
- [ ] Use: RestTemplate hoặc WebClient
- [ ] Use: Service discovery (lookup via Eureka)
- [ ] Test: Service-to-service calls
- [ ] Implement: Error handling (service unavailable)
- [ ] Implement: Timeout configuration
- [ ] Test: Failure scenarios
- [ ] Document: Inter-service communication

### Task 3: Configuration Server
- [ ] Setup: Spring Cloud Config Server
- [ ] Create: Config Server application
- [ ] Configure: Git backend (hoặc file system)
- [ ] Create: Configuration files cho services
- [ ] Configure: Services connect to Config Server
- [ ] Test: Configuration retrieval
- [ ] Test: Configuration refresh (hot reload)
- [ ] Test: Environment-specific configuration
- [ ] Document: Config Server setup

### Task 4: API Gateway với Spring Cloud Gateway
- [ ] Setup: Spring Cloud Gateway (review from Week 6)
- [ ] Configure: Routes to microservices
- [ ] Configure: Service discovery integration
- [ ] Test: Request routing through Gateway
- [ ] Configure: Load balancing trong Gateway
- [ ] Test: Load balancing works
- [ ] Document: API Gateway setup

### Task 5: Circuit Breaker với Resilience4j
- [ ] Add: Resilience4j dependency
- [ ] Configure: Circuit breaker cho service calls
- [ ] Configure: Failure threshold, timeout
- [ ] Implement: Fallback mechanism
- [ ] Test: Circuit opens on failures
- [ ] Test: Fallback responses
- [ ] Test: Circuit closes after recovery
- [ ] Monitor: Circuit breaker metrics
- [ ] Document: Circuit breaker implementation

### Task 6: Retry với Resilience4j
- [ ] Configure: Retry cho service calls
- [ ] Configure: Max retries, backoff strategy
- [ ] Test: Retry on transient failures
- [ ] Test: No retry on permanent failures
- [ ] Measure: Retry performance impact
- [ ] Document: Retry configuration

### Task 7: Rate Limiting trong Gateway
- [ ] Implement: Rate limiting filter trong Gateway
- [ ] Configure: Rate limits per service
- [ ] Configure: Rate limits per user
- [ ] Test: Rate limit enforcement
- [ ] Test: Rate limit headers
- [ ] Document: Rate limiting setup

### Task 8: Distributed Tracing với Zipkin
- [ ] Setup: Zipkin server
- [ ] Add: Spring Cloud Sleuth dependency
- [ ] Configure: Tracing trong all services
- [ ] Configure: Trace sampling
- [ ] Test: Trace requests across services
- [ ] View: Traces trong Zipkin UI
- [ ] Analyze: Request flow across services
- [ ] Document: Distributed tracing setup

### Task 9: Service Health Checks
- [ ] Implement: Health check endpoints trong all services
- [ ] Configure: Eureka health checks
- [ ] Test: Health check integration
- [ ] Test: Unhealthy service removed from registry
- [ ] Document: Health check implementation

### Task 10: Service Metrics
- [ ] Add: Micrometer dependency
- [ ] Expose: Metrics endpoints
- [ ] Configure: Custom metrics (request count, latency)
- [ ] Integrate: Với Prometheus (optional)
- [ ] Create: Dashboard với key metrics
- [ ] Document: Metrics setup

---

## Resilience TODOs

### Failure Scenario 1: Service Failure
- [ ] Simulate: Payment Service failure
- [ ] Test: Circuit breaker behavior
- [ ] Test: Fallback mechanism
- [ ] Test: Error propagation
- [ ] Measure: Failure impact on other services
- [ ] Document: Service failure handling

### Failure Scenario 2: Service Slow Response
- [ ] Simulate: Service slow response (high latency)
- [ ] Test: Timeout behavior
- [ ] Test: Circuit breaker opens
- [ ] Test: Retry behavior
- [ ] Measure: Impact on overall latency
- [ ] Document: Slow response handling

### Failure Scenario 3: Cascading Failures
- [ ] Simulate: Cascading failure (service A fails, causes B to fail, causes C to fail)
- [ ] Test: Circuit breaker prevents cascading
- [ ] Test: Bulkhead pattern (isolate failures)
- [ ] Design: Failure isolation strategy
- [ ] Document: Cascading failure prevention

### Failure Scenario 4: Service Discovery Failure
- [ ] Simulate: Eureka server failure
- [ ] Test: Service behavior (cached service list?)
- [ ] Test: Service re-registration
- [ ] Design: Service discovery redundancy
- [ ] Document: Discovery failure handling

### Failure Scenario 5: Configuration Server Failure
- [ ] Simulate: Config Server failure
- [ ] Test: Service behavior (cached config?)
- [ ] Test: Fallback configuration
- [ ] Design: Config Server redundancy
- [ ] Document: Config failure handling

### Resilience Testing
- [ ] Design: Chaos testing scenarios
- [ ] Test: Random service failures
- [ ] Test: Network partitions
- [ ] Test: High latency
- [ ] Measure: System resilience
- [ ] Identify: Weak points
- [ ] Document: Resilience test results

---

## Observability TODOs

### Observability Setup
- [ ] Design: Observability strategy
- [ ] Implement: Centralized logging
- [ ] Implement: Distributed tracing
- [ ] Implement: Metrics collection
- [ ] Implement: Alerting
- [ ] Create: Observability dashboard
- [ ] Document: Observability setup

### Distributed Tracing Analysis
- [ ] Trace: Request flow across 3+ services
- [ ] Analyze: Trace spans
- [ ] Identify: Slowest service
- [ ] Identify: Bottlenecks
- [ ] Measure: End-to-end latency
- [ ] Optimize: Slow services
- [ ] Document: Tracing analysis

### Logging Strategy
- [ ] Design: Logging strategy
- [ ] Implement: Structured logging (JSON)
- [ ] Add: Correlation IDs to logs
- [ ] Configure: Log levels per environment
- [ ] Implement: Centralized log aggregation
- [ ] Test: Log search và filtering
- [ ] Document: Logging strategy

### Metrics Collection
- [ ] Define: Key metrics (QPS, latency, error rate)
- [ ] Implement: Metrics collection
- [ ] Expose: Metrics endpoints
- [ ] Aggregate: Metrics across services
- [ ] Create: Metrics dashboard
- [ ] Configure: Alerting thresholds
- [ ] Document: Metrics strategy

### Service Dependency Mapping
- [ ] Map: Service dependencies
- [ ] Create: Dependency graph
- [ ] Identify: Critical dependencies
- [ ] Identify: Single points of failure
- [ ] Design: Dependency reduction
- [ ] Document: Dependency map

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Service Decomposition exercises hoàn thành?
- [ ] Review: Tất cả Spring Cloud tasks hoàn thành?
- [ ] Review: Tất cả Resilience scenarios tested?
- [ ] Review: Tất cả Observability tasks hoàn thành?
- [ ] Rate: Microservices architecture understanding (1-10)
- [ ] Rate: Service decomposition skills (1-10)
- [ ] Rate: Spring Cloud implementation skills (1-10)
- [ ] Rate: Resilience engineering skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Architecture Review
- [ ] Review: Microservices architecture design
- [ ] Challenge: Is each service justified? (justify each)
- [ ] Check: Service boundaries clear?
- [ ] Check: Data ownership clear?
- [ ] Check: Communication patterns appropriate?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Compare: Design vs industry best practices
- [ ] Document: Architecture review findings

### Service Decomposition Review
- [ ] Review: Service decomposition decisions
- [ ] Challenge: Can any services be merged? Justify
- [ ] Challenge: Are any services too small? (nanoservices)
- [ ] Challenge: Are any services too large? (distributed monolith)
- [ ] Verify: Each service has clear responsibility
- [ ] Verify: No shared database anti-pattern
- [ ] Document: Decomposition review

### Code Review
- [ ] Review: Microservices implementation code
- [ ] Check: Service discovery integration correct?
- [ ] Check: Circuit breaker configured?
- [ ] Check: Error handling adequate?
- [ ] Check: Logging và tracing implemented?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Inter-service communication performance
- [ ] Measure: Service-to-service latency
- [ ] Measure: Gateway overhead
- [ ] Identify: Performance bottlenecks
- [ ] Optimize: Slow services
- [ ] Re-test: Performance after optimization
- [ ] Document: Performance review

### Knowledge Check
- [ ] Explain: Monolith vs Microservices (viết comparison, không xem notes)
- [ ] Explain: Service decomposition strategies (viết explanation, không xem notes)
- [ ] Explain: Circuit breaker pattern (viết explanation, không xem notes)
- [ ] Explain: Service discovery - client-side vs server-side (viết comparison, không xem notes)
- [ ] Explain: Distributed tracing - why needed? (viết analysis, không xem notes)
- [ ] Solve: 5 services, each calls 2 others. Total service-to-service calls for 1 user request?
- [ ] Solve: Service A calls B, B calls C. A timeout = 5s, B timeout = 3s, C takes 4s. What happens?
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 microservices concepts còn confuse
- [ ] Write: 1 service decomposition decision bạn made và rationale
- [ ] Write: Confidence level cho Week 9 (1-10)
- [ ] Compare: Week 7 vs Week 8 progress
- [ ] Plan: Preparation cho Week 9 (Distributed Systems - Consistency)
- [ ] Set: Goals cho Week 9
- [ ] Document: Week 8 reflection (500 words)

### Mentor Questions (Answer these - challenge microservices)
- [ ] Q1: Team có 3 developers, muốn tạo 10 microservices. Justified? Tại sao hoặc tại sao không? (viết analysis với team size, operational overhead)
- [ ] Q2: Service A và B share same database. Microservices? Distributed monolith? Làm sao fix? (viết analysis với solution)
- [ ] Q3: Service A calls B, B calls C, C calls D. 1 user request = 4 service calls. Latency? Làm sao optimize? (viết analysis với optimization strategies)
- [ ] Q4: Monolith handles 10K QPS. Split to 5 microservices, each handles 2K QPS. Better? Justify. (viết analysis với trade-offs)
- [ ] Q5: Service needs to call 10 other services để complete 1 request. Problem? Làm sao fix? (viết analysis với solutions)
- [ ] Q6: Microservices có 99.9% availability each. 5 services in sequence. Overall availability? (calculate và analyze)
- [ ] Review: Answers có challenge microservices assumptions? Có justify decisions? Có consider trade-offs?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Service Decomposition TODOs: ✅ Complete với justifications
- [ ] Tất cả Spring Cloud TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Resilience TODOs: ✅ Complete với test results
- [ ] Tất cả Observability TODOs: ✅ Complete với dashboards
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Architecture diagrams saved: ✅ Yes
- [ ] Service decomposition justifications saved: ✅ Yes
- [ ] Ready for Week 9: ✅ Yes

---

> **Mentor Final Check**: Microservices là tool, không phải goal. Nếu bạn không thể justify tại sao service A phải tách khỏi service B, có thể chúng không nên tách. Platform architect phải challenge assumptions, not follow trends. Hãy honest: Bạn có thể justify mọi service split chưa? Bạn có thể explain trade-offs chưa? Bạn có thể identify khi nào monolith is better chưa? Nếu chưa, làm lại. Production systems need justified architecture, not microservices for the sake of microservices.
