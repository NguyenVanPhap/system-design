# Week 8 – Microservices Architecture & Service Governance

> **Mentor Note**: Microservices không phải là silver bullet. Nhiều teams split services mà không có lý do rõ ràng, tạo ra complexity không cần thiết. Bạn phải justify MỌI service split. Nếu bạn không thể explain tại sao service A phải tách khỏi service B, có thể chúng không nên tách. Platform architect phải challenge assumptions, not follow trends. Production systems need justified architecture, not microservices for the sake of microservices.

---

## Study TODOs

### Microservices vs Monolith
- [x] Đọc "Building Microservices" by Sam Newman - Chapter 1-2
  - **Key takeaways:** tách service theo *business capability* thay vì theo tầng kỹ thuật; ưu tiên autonomous teams; mọi split phải có lý do business/operational rõ.
  - **Insight thực chiến:** complexity của distributed system (network, retries, eventual consistency, observability) luôn tăng nhanh hơn dự đoán ban đầu.
- [x] Định nghĩa chính xác: Monolith
  - **Monolith:** một deployable unit duy nhất, thường 1 process/runtime, có thể có module nội bộ nhưng release/version đi cùng nhau.
  - **Hiểu đúng:** modular monolith vẫn là kiến trúc tốt nếu boundary trong code rõ, test tốt, và chưa cần scale tổ chức.
- [x] Định nghĩa chính xác: Microservices
  - **Microservices:** tập hợp nhiều service độc lập, mỗi service sở hữu capability + data riêng, giao tiếp qua API/event, deploy độc lập.
  - **Điều kiện đủ:** độc lập deploy + ownership + data boundary; nếu chỉ tách code nhưng vẫn shared DB/release cùng lúc thì chưa phải microservices đúng nghĩa.
- [x] Đọc về "monolith advantages" - 10 advantages
  - **10 advantages (chi tiết):**
    1) local dev nhanh; 2) debug end-to-end đơn giản; 3) transaction ACID dễ; 4) latency nội bộ thấp;
    5) CI/CD ít pipeline; 6) quan sát hệ thống ít phân mảnh; 7) chi phí infra thấp giai đoạn đầu;
    8) thay đổi cross-module nhanh; 9) onboarding dễ hơn cho team nhỏ; 10) governance nhẹ.
- [x] Đọc về "monolith disadvantages" - 10 disadvantages
  - **10 disadvantages (chi tiết):**
    1) release coupling; 2) scale toàn khối tốn tài nguyên; 3) build/deploy chậm khi codebase lớn;
    4) blast radius cao; 5) regression risk cao; 6) ownership mờ khi nhiều team;
    7) tech lock-in; 8) boundary mờ theo thời gian; 9) khó tách domain khi org tăng trưởng;
    10) test suite dễ phình to và chậm.
- [x] Đọc về "microservices advantages" - 10 advantages
  - **10 advantages (chi tiết):**
    1) deploy độc lập; 2) scale theo hotspot; 3) ownership rõ theo domain; 4) fault isolation tốt hơn;
    5) phù hợp nhiều team song song; 6) chọn tech có kiểm soát theo bài toán; 7) release nhanh theo team;
    8) dễ tối ưu performance theo service; 9) hỗ trợ evolution theo bounded context; 10) giảm coupling ở tổ chức lớn.
- [x] Đọc về "microservices disadvantages" - 10 disadvantages
  - **10 disadvantages (chi tiết):**
    1) network failures; 2) latency tăng theo hop; 3) debugging phân tán khó; 4) eventual consistency phức tạp;
    5) DevOps/SRE burden cao; 6) test integration/E2E tốn kém; 7) versioning contract khó;
    8) observability cost cao; 9) governance bắt buộc chặt; 10) dễ rơi vào distributed monolith.
- [x] Compare: Monolith vs Microservices (15 criteria)
  - **15 tiêu chí so sánh:** deployment unit, team topology, scaling granularity, latency profile, failure isolation, consistency model, transaction style, data ownership, testing strategy, debugging effort, release cadence, infra cost, ops complexity, governance maturity, observability requirements.
  - **Kết luận:** team nhỏ/domain chưa ổn định => monolith; team lớn/hotspot rõ/release bottleneck => cân nhắc microservices.
- [x] Research: When to start with monolith? (5 indicators)
  - **5 indicators:** (1) <8 engineers; (2) product đang phase khám phá; (3) traffic chưa có hotspot; (4) cần speed of learning; (5) platform ops còn non.
- [x] Research: When to split to microservices? (5 indicators)
  - **5 indicators:** (1) bounded contexts ổn định; (2) nhiều team đụng release nhau; (3) hotspot scale độc lập rõ; (4) có nền tảng CI/CD + observability + on-call; (5) có khả năng quản trị contract/versioning.
- [x] Đọc về "microservices prerequisites" - what you need first
  - **Prerequisites bắt buộc:** DDD + boundary clarity, API contract discipline, CI/CD tự động, log/metric/trace tập trung, runbook/on-call, secret management, security baseline, platform standards (naming/versioning/error model).

### Service Decomposition Strategies
- [x] Đọc về "domain-driven design" (DDD) - bounded contexts
  - **DDD core idea:** model software theo domain language (ubiquitous language), không theo cấu trúc database/UI.
  - **Tác dụng với microservices:** giảm hiểu sai nghiệp vụ giữa team, giúp boundary bền hơn khi scale tổ chức.
- [x] Đọc về "bounded context" - service boundaries
  - **Bounded context:** phạm vi mà 1 model có nghĩa nhất quán.
  - **Rule:** mỗi context nên có ownership rõ, expose contract rõ; không share entity nội bộ giữa contexts.
- [x] Đọc về "decomposition by business capability"
  - **Cách làm:** tách theo năng lực nghiệp vụ ổn định (Payment, Risk, Settlement, Notification).
  - **Ưu điểm:** ít phụ thuộc vào org chart ngắn hạn; phù hợp tổ chức thay đổi team.
- [x] Đọc về "decomposition by subdomain"
  - **Theo DDD:** tách mạnh ở core subdomain; supporting/generic có thể giữ coarse-grained để giảm overhead.
  - **Trade-off:** tách quá sớm ở supporting domain dễ tạo nanoservice.
- [x] Đọc về "decomposition by data"
  - **Khi phù hợp:** domain có lifecycle dữ liệu, retention policy, compliance khác nhau.
  - **Cảnh báo:** không tách service chỉ vì “mỗi bảng một service”.
- [x] Compare: Decomposition strategies (10 criteria)
  - **10 criteria chi tiết:** business alignment, change frequency, data ownership, team autonomy, runtime coupling, consistency requirement, scaling hotspot fit, cognitive load, migration effort, long-term maintainability.
- [x] Đọc về "shared database anti-pattern"
  - **Vì sao xấu:** schema coupling, lock contention, release coupling, phá nguyên tắc data ownership.
  - **Cách chuyển đổi:** tách schema theo service + API/event integration + kế hoạch migration từng bước.
- [x] Đọc về "service coupling" - tight vs loose coupling
  - **Tight coupling:** call chain sâu, shared schema, đồng bộ release.
  - **Loose coupling:** contract rõ, backward-compatible, async cho luồng không critical.
- [x] Đọc về "cohesion" - high cohesion principle
  - **High cohesion:** logic thay đổi cùng nhau đặt cùng service.
  - **Dấu hiệu cohesion thấp:** 1 feature đổi phải sửa nhiều service không liên quan.
- [x] Research: Service decomposition best practices
  - **Best practices:** bắt đầu coarse-grained, đo call graph + deploy pain points, chỉ split khi có KPI rõ (latency, throughput, release lead time).

### Service Boundaries
- [x] Đọc về "service boundaries" - how to define?
  - **Boundary inputs:** business invariant, ownership team, SLA, change frequency, data lifecycle.
  - **Anti-pattern:** define boundary theo màn hình UI hoặc theo bảng DB.
- [x] Đọc về "single responsibility principle" - applied to services
  - 1 service nên có 1 business reason-to-change; nếu vừa xử lý payment vừa reporting thì thường sai SRP.
- [x] Đọc về "data ownership" - each service owns its data
  - Source-of-truth phải rõ: service khác chỉ đọc qua API/event, không truy cập DB trực tiếp.
- [x] Đọc về "shared data anti-pattern"
  - Shared table gây coupling ẩn, migration khó, lock contention và rollback phức tạp.
- [x] Đọc về "service autonomy" - independent deployment
  - Autonomy thực sự gồm: build/test/deploy/rollback độc lập + contract compatibility.
- [x] Đọc về "service size" - how big? how small?
  - Service tốt là service team hiểu trọn và vận hành on-call được, không phải "càng nhỏ càng tốt".
- [x] Research: Service size guidelines (team size, code size)
  - Guideline: 1 team 5-8 người nên sở hữu số service vừa phải; quan trọng là cognitive load, không phải số dòng code.
- [x] Đọc về "nanoservices anti-pattern" - too small services
  - Dấu hiệu nanoservice: endpoint quá ít, phụ thuộc dày, call chain sâu; giải pháp là merge theo cohesion.

### API Gateway Patterns
- [x] Đọc về "API Gateway" pattern (review from Week 6)
  - Gateway là entry point cho client concerns (auth, routing, throttling, aggregation), không nên chứa business core.
  - **Rule thực hành:** business workflow phải nằm trong downstream services để tránh biến gateway thành monolith mới.
- [x] Đọc về "BFF" (Backend for Frontend) pattern
  - BFF tối ưu theo từng client (web/mobile/admin), giảm over-fetching và giảm coupling frontend-backend.
  - **Khi dùng:** khi web/mobile có nhu cầu dữ liệu khác nhau rõ rệt và release cadence khác nhau.
- [x] So sánh: API Gateway vs BFF (10 criteria)
  - **10 criteria:** purpose, ownership, client specificity, transformation depth, release cadence, scaling pattern, caching strategy, security boundary, complexity, governance model.
  - **Kết luận nhanh:** Gateway cho concern chung toàn hệ; BFF cho experience riêng từng client.
- [x] Đọc về "service mesh" - alternative to API Gateway?
  - Mesh không thay thế hoàn toàn Gateway: mesh xử lý east-west traffic; gateway xử lý north-south API exposure.
- [x] Đọc về "API Gateway responsibilities" - routing, auth, rate limiting
  - Core responsibilities: routing, authentication/authorization integration, TLS termination, rate limit, request shaping, observability hooks.
  - **Không nên làm:** đặt business orchestration phức tạp trong gateway.
- [x] Đọc về "API Gateway scalability" - single point of failure?
  - Tránh SPOF bằng multi-instance + LB + zone redundancy + stateless design.
  - **Checklist vận hành:** health probes, autoscaling policy, canary rollout, circuit protection upstream.
- [x] Research: API Gateway solutions (Kong, AWS API Gateway, Spring Cloud Gateway)
  - **Kong:** mạnh plugin/self-hosted.
  - **AWS API Gateway:** managed, tích hợp IAM/WAF/Lambda sâu.
  - **Spring Cloud Gateway:** fit team Java/Spring, linh hoạt route/filter custom.

### Service Discovery
- [x] Đọc về "service discovery" - why needed?
  - **Lý do cốt lõi:** instances lên/xuống liên tục trong autoscaling, endpoint hardcode sẽ stale rất nhanh.
  - **Nếu thiếu discovery:** deploy mới dễ fail routing, tăng lỗi 5xx và MTTR khi incident.
- [x] Đọc về "client-side discovery" - how it works
  - Client hỏi registry để lấy danh sách instance rồi tự load-balance.
  - **Ưu điểm:** linh hoạt, tối ưu theo ngôn ngữ/framework; **nhược:** logic discovery nằm ở mọi client.
- [x] Đọc về "server-side discovery" - how it works
  - Client gọi 1 entry point (LB/router), thành phần này chọn instance backend.
  - **Ưu điểm:** client mỏng; **nhược:** phụ thuộc mạnh vào router/LB layer.
- [x] So sánh: Client-side vs Server-side discovery (10 criteria)
  - **10 criteria:** độ phức tạp client, mức lock-in ngôn ngữ, vị trí load balancing, handling failure, latency path, observability, kiểm soát policy, bảo mật tập trung, khả năng mở rộng, effort migrate.
- [x] Đọc về "service registry" - centralized registry
  - Registry lưu service name -> healthy instances và metadata.
  - **Yêu cầu bắt buộc:** HA, health checks, backup, monitoring vì đây là control-plane critical.
- [x] Đọc về "self-registration" - services register themselves
  - Service tự đăng ký/huỷ đăng ký theo lifecycle start/stop.
  - **Risk:** bug startup/shutdown có thể làm registry stale nếu heartbeat/TTL không chuẩn.
- [x] Đọc về "third-party registration" - registry discovers services
  - Agent/orchestrator (ví dụ Kubernetes) quản lý registration thay cho app code.
  - **Lợi ích:** giảm boilerplate trong app, chuẩn hóa hành vi discovery toàn platform.
- [x] Research: Service discovery solutions (Eureka, Consul, Kubernetes)
  - **Eureka:** thuận hệ Spring/Netflix.
  - **Consul:** discovery + KV + health checks mạnh.
  - **Kubernetes:** discovery built-in qua Service/DNS, thường là lựa chọn mặc định cloud-native.

### Configuration Management
- [x] Đọc về "configuration management" - why important?
  - Config drift gây production incidents; config phải versioned, auditable, rollbackable.
- [x] Đọc về "externalized configuration" - 12-factor app
  - Tách config khỏi code/build artifact để deploy cùng binary cho nhiều environment.
- [x] Đọc về "configuration server" - centralized config
  - Config server centralize read + governance + audit trail, hỗ trợ dynamic refresh.
- [x] Đọc về "configuration versioning" - track config changes
  - Versioning qua Git/tag giúp trace ai đổi gì, khi nào, và rollback nhanh.
- [x] Đọc về "configuration refresh" - hot reload
  - Hot reload giảm downtime nhưng cần kiểm soát consistency giữa instances.
- [x] Đọc về "environment-specific configuration"
  - Quản lý profile dev/staging/prod tách bạch; tuyệt đối không hardcode env secrets.
- [x] Đọc về "secrets management" - sensitive config
  - Secrets phải lưu trong vault/KMS, rotate định kỳ, least-privilege access, không log plaintext.
- [x] Research: Configuration management solutions (Spring Cloud Config, Consul, Vault)
  - **Spring Cloud Config:** native cho Spring; **Consul:** service discovery + config KV; **Vault:** chuẩn cho secrets lifecycle.

### Inter-Service Communication
- [x] Đọc về "synchronous communication" - REST, gRPC
  - Sync phù hợp request/response trực tiếp nhưng tăng temporal coupling.
- [x] Đọc về "asynchronous communication" - message queues
  - Async giảm coupling thời gian, tăng resilience và throughput, đổi lại là complexity về ordering/retry/idempotency.
- [x] So sánh: Sync vs Async communication (10 criteria)
  - **10 criteria:** latency, coupling, reliability, failure handling, consistency model, throughput, tracing complexity, retry semantics, backpressure handling, developer ergonomics.
- [x] Đọc về "REST over HTTP" - pros và cons
  - Pros: ecosystem rộng, dễ debug; cons: payload verbose, contract drift, performance thấp hơn binary protocol.
- [x] Đọc về "gRPC" - when to use?
  - Dùng khi cần low-latency internal calls, strong contracts (protobuf), streaming; cân nhắc tooling với external clients.
- [x] Đọc về "message queues" - async communication (review from Week 7)
  - Queue/broker patterns: work queue, pub/sub, dead-letter queue; bắt buộc idempotent consumer.
- [x] Đọc về "service mesh" - service-to-service communication
  - Mesh offload mTLS, retries, traffic policy khỏi app code thông qua sidecar/proxy.
- [x] Research: Communication patterns best practices
  - Ưu tiên async cho integration events, sync cho query ngắn; đặt timeout budgets; giới hạn call depth; dùng circuit breaker + retry with jitter.

### Circuit Breaker Pattern
- [x] Đọc về "Circuit Breaker" pattern (review from Week 2)
  - Circuit breaker ngăn retry storm và bảo vệ downstream khi lỗi tăng cao.
- [x] Đọc về "circuit states" - closed, open, half-open
  - Closed: pass-through; Open: fail-fast; Half-open: cho phép một phần request thử recovery.
- [x] Đọc về "failure threshold" - when to open circuit
  - Dựa trên error rate/sliding window + minimum calls để tránh false positive.
- [x] Đọc về "timeout" - wait before retry
  - Timeout cần nhỏ hơn caller SLA budget, và phải được set ở mọi outbound call.
- [x] Đọc về "fallback mechanism" - degraded response
  - Fallback trả cached/default/degraded data, không che giấu lỗi business critical.
- [x] Đọc về "circuit breaker vs retry" - when to use which?
  - Retry cho transient errors; circuit breaker cho persistent failures. Thường dùng kết hợp có giới hạn.
- [x] Research: Circuit breaker implementations (Hystrix, Resilience4j, Sentinel)
  - Hystrix đã maintenance mode; Resilience4j lightweight cho JVM; Sentinel mạnh về flow control trong cloud-native Java stack.

### Distributed Tracing
- [x] Đọc về "distributed tracing" - why needed?
  - Tracing cho phép theo dõi 1 request xuyên nhiều service để tìm bottleneck và lỗi propagation.
- [x] Đọc về "trace" - request journey across services
  - Trace là tập hợp toàn bộ spans thuộc cùng transaction/request.
- [x] Đọc về "span" - single operation in trace
  - Span đại diện 1 operation có start/end time, tags, logs, status.
- [x] Đọc về "trace context" - correlation ID
  - Trace context (trace-id/span-id/baggage) phải propagate qua headers/message metadata.
- [x] Đọc về "sampling" - reduce trace volume
  - Sampling giảm cost observability; dùng adaptive/tail sampling cho high traffic.
- [x] Đọc về "trace propagation" - pass context between services
  - Chuẩn phổ biến: W3C Trace Context; bắt buộc tương thích multi-language stack.
- [x] Research: Distributed tracing solutions (Zipkin, Jaeger, OpenTelemetry)
  - OpenTelemetry là chuẩn instrumentation; Jaeger/Zipkin là backend visualization/storage phổ biến.

### Service Versioning
- [x] Đọc về "service versioning" - why needed?
  - Versioning giúp evolve API mà không phá clients đang chạy production.
- [x] Đọc về "backward compatibility" - version compatibility
  - Ưu tiên additive changes; tránh breaking changes nếu chưa có migration window.
- [x] Đọc về "versioning strategies" - URL, header, content negotiation
  - URL version dễ hiểu; header sạch URL nhưng khó debug; content negotiation linh hoạt nhưng phức tạp hơn.
- [x] Đọc về "deprecation strategy" - how to deprecate versions
  - Có timeline rõ, deprecation headers, communication plan, và migration docs.
- [x] Đọc về "coexistence" - multiple versions running
  - Có thể chạy song song v1/v2 một thời gian, đo usage trước khi sunset.
- [x] Đọc về "version routing" - route to correct version
  - Gateway/router định tuyến theo path/header để đảm bảo đúng contract runtime.
- [x] Research: Service versioning best practices
  - Contract testing + API compatibility checks trong CI là bắt buộc để tránh breaking changes ngoài ý muốn.

### Service Mesh
- [x] Đọc về "service mesh" - what is it?
  - Service mesh là infrastructure layer quản lý service-to-service communication qua proxy data plane + control plane.
- [x] Đọc về "sidecar pattern" - proxy alongside service
  - Sidecar proxy intercept traffic, hỗ trợ policy/routing/telemetry mà không đổi nhiều app code.
- [x] Đọc về "service mesh benefits" - traffic management, security
  - Benefits chính: mTLS, traffic shaping, retries/timeouts policy, observability chuẩn hóa.
- [x] Đọc về "Istio" - popular service mesh
  - Istio feature-rich (policy/security/telemetry) nhưng complexity vận hành cao hơn.
- [x] Đọc về "Linkerd" - alternative service mesh
  - Linkerd lightweight và đơn giản hơn, phù hợp team muốn adoption nhanh.
- [x] So sánh: Service mesh vs API Gateway (10 criteria)
  - **10 criteria:** traffic direction, policy scope, ownership, protocol support, security model, observability depth, operational complexity, performance overhead, use cases, deployment topology.
- [x] Research: When to use service mesh?
  - Dùng mesh khi có nhiều service + yêu cầu security/traffic policy nhất quán; chưa cần mesh khi hệ thống nhỏ hoặc platform maturity thấp.

---

## Service Decomposition TODOs

### Decomposition Exercise 1: Monolith Analysis
- [x] Scenario: Existing monolith application
- [x] Analyze: Current monolith structure
- [x] Identify: 5 reasons to keep as monolith
- [x] Identify: 5 reasons to split to microservices
- [x] Evaluate: Is split justified? Justify answer
- [x] Document: Monolith analysis (500 words)
  - **Monolith snapshot:** module chính gồm User, Wallet, Betting, Settlement, Reporting dùng chung 1 database transaction.
  - **5 reasons keep monolith:** team nhỏ, domain chưa ổn định, cần đổi nhanh cross-module, latency thấp cho transaction chain, vận hành đơn giản.
  - **5 reasons split:** release bottleneck giữa teams, betting traffic hotspot, reporting workload khác profile, ownership mờ, scaling độc lập chưa làm được.
  - **Evaluation:** hiện tại chỉ nên tách dần 1-2 bounded contexts có hotspot rõ (vd Reporting), chưa nên big-bang split toàn hệ thống.

### Decomposition Exercise 2: Payment System Decomposition
- [x] Design: Decompose Payment System into microservices
- [x] Requirement: Payment processing, notifications, reporting
- [x] Identify: Service boundaries (use DDD bounded contexts)
- [x] Design: Service responsibilities
- [x] Design: Data ownership (each service owns its data)
- [x] Justify: Each service split (why separate services?)
- [x] Challenge: Can any services be merged? Justify
- [x] Document: Service decomposition design (1000 words)
  - **Proposed services:** `Payment Service`, `Notification Service`, `Reporting Service`.
  - **Responsibilities:** Payment xử lý authorize/capture/refund; Notification gửi email/sms/webhook; Reporting tổng hợp analytics và reconciliation views.
  - **Data ownership:** Payment DB (transactions), Notification DB (delivery logs/templates), Reporting store (read models/aggregates).
  - **Justification:** workload, SLA và change cadence khác nhau; tránh coupling release giữa critical payment flow và non-critical reporting.
  - **Merge challenge:** nếu hệ thống giai đoạn đầu, có thể gộp Notification vào Payment module nội bộ để giảm ops overhead, nhưng vẫn giữ boundary code rõ.

### Decomposition Exercise 3: Betting Platform Decomposition
- [x] Design: Decompose Betting Platform into microservices
- [x] Requirement: Bet management, match management, settlement
- [x] Apply: Domain-driven design principles
- [x] Define: Bounded contexts
- [x] Design: Service boundaries
- [x] Design: Service communication (sync/async)
- [x] Justify: Service decomposition decisions
- [x] Document: Complete microservices architecture
  - **Bounded contexts:** `Bet Management`, `Match Management`, `Settlement`, `Wallet`, `Risk/Fraud`.
  - **Communication:** sync cho query ngắn (odds snapshot, balance check), async event cho `BetPlaced`, `MatchResultDeclared`, `SettlementCompleted`.
  - **Boundary rationale:** Settlement cần rules và compute pipeline riêng; Match domain có lifecycle độc lập; Bet domain cần throughput cao.
  - **Architecture note:** tránh call chain sâu; ưu tiên event choreography + idempotent consumers.

### Decomposition Exercise 4: Service Size Analysis
- [x] Scenario: Team wants to create 20 microservices
- [x] Analyze: Is 20 services justified?
- [x] Calculate: Team size needed (assume 2 developers per service)
- [x] Calculate: Operational overhead (monitoring, deployment)
- [x] Analyze: Communication complexity (N services = N*(N-1)/2 connections)
- [x] Recommend: Optimal number of services? Justify
- [x] Document: Service size analysis
  - **Team size math:** 20 services × 2 dev/service = 40 dev (chưa tính QA/DevOps/SRE) → không phù hợp team nhỏ/trung bình.
  - **Communication complexity:** \(20*(20-1)/2 = 190\) service pairs, governance/test matrix tăng mạnh.
  - **Operational overhead:** CI/CD pipelines, dashboards, alerts, on-call, secrets, compliance tăng theo service count.
  - **Recommendation:** bắt đầu 5-8 service coarse-grained theo bounded context, chỉ tách tiếp khi có KPI rõ (deploy bottleneck, scale hotspot, ownership conflict).

### Decomposition Exercise 5: Shared Data Analysis
- [x] Scenario: 3 services need same user data
- [x] Analyze: Options (shared database? data duplication? user service?)
- [x] Compare: All options (5 criteria each)
- [x] Design: Best approach với justification
- [x] Design: Data synchronization strategy (if needed)
- [x] Document: Shared data strategy
  - **Options:** (1) shared DB, (2) local copy via events, (3) User Profile service as source of truth.
  - **5 criteria:** autonomy, consistency, latency, operational complexity, scalability.
  - **Best approach:** User service làm source-of-truth + event-driven replication selective fields cho consumers.
  - **Sync strategy:** publish `UserUpdated` events, versioned schema, idempotent consumer, periodic reconcile job để phát hiện drift.

### Decomposition Exercise 6: Anti-Patterns
- [x] Identify: 5 microservices anti-patterns
- [x] Analyze: Shared database anti-pattern
- [x] Analyze: Distributed monolith anti-pattern
- [x] Analyze: Nanoservices anti-pattern
- [x] Analyze: Chatty services anti-pattern
- [x] Design: Solutions cho each anti-pattern
- [x] Document: Anti-patterns và solutions
  - **5 anti-patterns:** shared database, distributed monolith, nanoservices, chatty services, thiếu observability.
  - **Shared DB fix:** tách schema ownership + API contract + migration plan theo strangler pattern.
  - **Distributed monolith fix:** giảm release coupling bằng backward-compatible contracts, contract testing, independent deploy pipeline.
  - **Nanoservices fix:** merge services có cohesion cao nhưng autonomy thấp; đo call graph trước khi split/merge.
  - **Chatty services fix:** API composition/BFF, cache, coarse-grained endpoints, async events cho non-critical flows.

---

## Spring Cloud TODOs

### Task 1: Service Discovery với Eureka
- [x] Setup: Eureka Server (Spring Cloud Netflix)
- [x] Create: Eureka Server application
- [x] Configure: Eureka Server properties
- [x] Create: 2 microservices (Payment Service, Notification Service)
- [x] Configure: Services register với Eureka
- [x] Test: Service registration
- [x] Test: Service discovery (lookup service by name)
- [x] Test: Service deregistration (when service stops)
- [x] Document: Eureka setup
  - **Implementation notes:** 1 Eureka Server + 2 clients (`payment-service`, `notification-service`) với `@EnableDiscoveryClient`.
  - **Test result:** services xuất hiện trong Eureka dashboard; stop service thì status chuyển DOWN/removed theo lease timeout.

### Task 2: Inter-Service Communication
- [x] Implement: REST communication between services
- [x] Use: RestTemplate hoặc WebClient
- [x] Use: Service discovery (lookup via Eureka)
- [x] Test: Service-to-service calls
- [x] Implement: Error handling (service unavailable)
- [x] Implement: Timeout configuration
- [x] Test: Failure scenarios
- [x] Document: Inter-service communication
  - **Implementation notes:** dùng WebClient + service name URL (`http://payment-service`) qua load-balancer client.
  - **Resilience baseline:** timeout connect/read + error mapping 5xx/timeout về response degrade có kiểm soát.

### Task 3: Configuration Server
- [x] Setup: Spring Cloud Config Server
- [x] Create: Config Server application
- [x] Configure: Git backend (hoặc file system)
- [x] Create: Configuration files cho services
- [x] Configure: Services connect to Config Server
- [x] Test: Configuration retrieval
- [x] Test: Configuration refresh (hot reload)
- [x] Test: Environment-specific configuration
- [x] Document: Config Server setup
  - **Implementation notes:** Config Server trỏ Git repo chứa `payment-service-dev.yml`, `payment-service-prod.yml`.
  - **Test result:** `/actuator/refresh` cập nhật runtime config; profile `dev/prod` trả đúng giá trị theo environment.

### Task 4: API Gateway với Spring Cloud Gateway
- [x] Setup: Spring Cloud Gateway (review from Week 6)
- [x] Configure: Routes to microservices
- [x] Configure: Service discovery integration
- [x] Test: Request routing through Gateway
- [x] Configure: Load balancing trong Gateway
- [x] Test: Load balancing works
- [x] Document: API Gateway setup
  - **Implementation notes:** routes dùng `lb://payment-service` và `lb://notification-service`, filter rewrite path cơ bản.
  - **Test result:** traffic đi qua gateway thành công; scale 2 instances cho payment thì request phân phối round-robin.

### Task 5: Circuit Breaker với Resilience4j
- [x] Add: Resilience4j dependency
- [x] Configure: Circuit breaker cho service calls
- [x] Configure: Failure threshold, timeout
- [x] Implement: Fallback mechanism
- [x] Test: Circuit opens on failures
- [x] Test: Fallback responses
- [x] Test: Circuit closes after recovery
- [x] Monitor: Circuit breaker metrics
- [x] Document: Circuit breaker implementation
  - **Implementation notes:** cấu hình sliding window + failure rate threshold + wait duration in open state.
  - **Test result:** khi downstream lỗi liên tục circuit chuyển OPEN, fallback trả response degrade; recovery ổn định thì HALF_OPEN -> CLOSED.

### Task 6: Retry với Resilience4j
- [x] Configure: Retry cho service calls
- [x] Configure: Max retries, backoff strategy
- [x] Test: Retry on transient failures
- [x] Test: No retry on permanent failures
- [x] Measure: Retry performance impact
- [x] Document: Retry configuration
  - **Implementation notes:** max retries nhỏ (2-3), exponential backoff + jitter để tránh retry storm.
  - **Measurement:** transient lỗi giảm tỉ lệ fail; latency p95 tăng nhẹ trong ngưỡng chấp nhận.

### Task 7: Rate Limiting trong Gateway
- [x] Implement: Rate limiting filter trong Gateway
- [x] Configure: Rate limits per service
- [x] Configure: Rate limits per user
- [x] Test: Rate limit enforcement
- [x] Test: Rate limit headers
- [x] Document: Rate limiting setup
  - **Implementation notes:** token-bucket limiter tại gateway; key resolver theo user/API key + route.
  - **Test result:** vượt quota trả `429 Too Many Requests`; headers phản ánh remaining tokens/reset window.

### Task 8: Distributed Tracing với Zipkin
- [x] Setup: Zipkin server
- [x] Add: Spring Cloud Sleuth dependency
- [x] Configure: Tracing trong all services
- [x] Configure: Trace sampling
- [x] Test: Trace requests across services
- [x] View: Traces trong Zipkin UI
- [x] Analyze: Request flow across services
- [x] Document: Distributed tracing setup
  - **Implementation notes:** bật tracing cho gateway + services; propagate trace headers xuyên WebClient calls.
  - **Test result:** trace hiển thị đầy đủ spans gateway -> payment -> notification, xác định nhanh hop gây chậm.

### Task 9: Service Health Checks
- [x] Implement: Health check endpoints trong all services
- [x] Configure: Eureka health checks
- [x] Test: Health check integration
- [x] Test: Unhealthy service removed from registry
- [x] Document: Health check implementation
  - **Implementation notes:** dùng Actuator `/health` + readiness/liveness; Eureka dùng healthcheck URL.
  - **Test result:** instance unhealthy bị loại khỏi traffic routing sau ngưỡng check thất bại.

### Task 10: Service Metrics
- [x] Add: Micrometer dependency
- [x] Expose: Metrics endpoints
- [x] Configure: Custom metrics (request count, latency)
- [x] Integrate: Với Prometheus (optional)
- [x] Create: Dashboard với key metrics
- [x] Document: Metrics setup
  - **Implementation notes:** expose `/actuator/prometheus`, custom timers/counters cho API quan trọng.
  - **Dashboard baseline:** QPS, p95 latency, error rate, circuit breaker open count, retry attempts.

---

## Resilience TODOs

### Failure Scenario 1: Service Failure
- [x] Simulate: Payment Service failure
- [x] Test: Circuit breaker behavior
- [x] Test: Fallback mechanism
- [x] Test: Error propagation
- [x] Measure: Failure impact on other services
- [x] Document: Service failure handling
  - **Scenario setup:** stop `payment-service` instances đột ngột trong khi giữ gateway + notification hoạt động.
  - **Observed behavior:** circuit breaker mở sau ngưỡng lỗi; fallback trả response degrade thay vì treo request.
  - **Impact measurement:** error rate tăng cục bộ ở payment flow, các luồng khác không bị sập dây chuyền.

### Failure Scenario 2: Service Slow Response
- [x] Simulate: Service slow response (high latency)
- [x] Test: Timeout behavior
- [x] Test: Circuit breaker opens
- [x] Test: Retry behavior
- [x] Measure: Impact on overall latency
- [x] Document: Slow response handling
  - **Scenario setup:** inject delay 3-5s ở downstream endpoint.
  - **Observed behavior:** timeout kích hoạt đúng budget; retry chỉ thử số lần nhỏ; circuit mở khi lỗi timeout vượt threshold.
  - **Latency impact:** p95 tăng trước khi circuit mở, sau đó ổn định nhờ fail-fast + fallback.

### Failure Scenario 3: Cascading Failures
- [x] Simulate: Cascading failure (service A fails, causes B to fail, causes C to fail)
- [x] Test: Circuit breaker prevents cascading
- [x] Test: Bulkhead pattern (isolate failures)
- [x] Design: Failure isolation strategy
- [x] Document: Cascading failure prevention
  - **Scenario setup:** Payment gọi Risk rồi Wallet; làm Risk lỗi liên tục để tạo pressure upstream.
  - **Protection result:** circuit breaker chặn retry storm; bulkhead tách threadpool cho outbound calls.
  - **Isolation strategy:** giới hạn call depth, timeout per hop, queue cho non-critical tasks, degrade theo chức năng.

### Failure Scenario 4: Service Discovery Failure
- [x] Simulate: Eureka server failure
- [x] Test: Service behavior (cached service list?)
- [x] Test: Service re-registration
- [x] Design: Service discovery redundancy
- [x] Document: Discovery failure handling
  - **Scenario setup:** tắt Eureka trong lúc services đang chạy.
  - **Observed behavior:** client vẫn gọi được instance đã cache trong thời gian ngắn; instance mới không discover được.
  - **Redundancy design:** chạy Eureka HA (multi-node), cấu hình retry/backoff khi re-register, chuẩn bị static fallback cho luồng critical.

### Failure Scenario 5: Configuration Server Failure
- [x] Simulate: Config Server failure
- [x] Test: Service behavior (cached config?)
- [x] Test: Fallback configuration
- [x] Design: Config Server redundancy
- [x] Document: Config failure handling
  - **Scenario setup:** dừng Config Server sau khi services đã bootstrap.
  - **Observed behavior:** services tiếp tục chạy bằng config đã nạp; refresh runtime thất bại có kiểm soát.
  - **Redundancy design:** multi-instance config server + Git HA + defaults an toàn trong service để tránh hard failure.

### Resilience Testing
- [x] Design: Chaos testing scenarios
- [x] Test: Random service failures
- [x] Test: Network partitions
- [x] Test: High latency
- [x] Measure: System resilience
- [x] Identify: Weak points
- [x] Document: Resilience test results
  - **Chaos suite:** kill pod random, inject network loss/latency, spike load ngắn hạn.
  - **Resilience metrics:** availability theo critical API, MTTR, fallback hit-rate, timeout rate.
  - **Weak points found:** dependency call chain còn dài ở payment path; observability alert cho retry storm cần nhạy hơn.

---

## Observability TODOs

### Observability Setup
- [x] Design: Observability strategy
- [x] Implement: Centralized logging
- [x] Implement: Distributed tracing
- [x] Implement: Metrics collection
- [x] Implement: Alerting
- [x] Create: Observability dashboard
- [x] Document: Observability setup
  - **Strategy:** 3 pillars (logs/metrics/traces) + SLO-driven alerting cho các user journey quan trọng.
  - **Stack baseline:** logs tập trung, tracing xuyên service, metrics Prometheus-style, dashboard chung cho on-call.

### Distributed Tracing Analysis
- [x] Trace: Request flow across 3+ services
- [x] Analyze: Trace spans
- [x] Identify: Slowest service
- [x] Identify: Bottlenecks
- [x] Measure: End-to-end latency
- [x] Optimize: Slow services
- [x] Document: Tracing analysis
  - **Flow traced:** Gateway -> Payment -> Risk -> Notification.
  - **Findings:** span Payment->Risk có p95 cao nhất; bottleneck chủ yếu ở external dependency latency.
  - **Optimization:** thêm cache short-lived + timeout budget chặt hơn, giảm tail latency rõ rệt.

### Logging Strategy
- [x] Design: Logging strategy
- [x] Implement: Structured logging (JSON)
- [x] Add: Correlation IDs to logs
- [x] Configure: Log levels per environment
- [x] Implement: Centralized log aggregation
- [x] Test: Log search và filtering
- [x] Document: Logging strategy
  - **Logging model:** JSON fields chuẩn (`timestamp`, `level`, `service`, `traceId`, `spanId`, `message`, `errorCode`).
  - **Environment policy:** `DEBUG` cho dev, `INFO` cho staging/prod, `ERROR` alertable khi vượt ngưỡng.
  - **Search validation:** truy vết incident từ gateway sang downstream bằng `traceId` trong < 2 phút.

### Metrics Collection
- [x] Define: Key metrics (QPS, latency, error rate)
- [x] Implement: Metrics collection
- [x] Expose: Metrics endpoints
- [x] Aggregate: Metrics across services
- [x] Create: Metrics dashboard
- [x] Configure: Alerting thresholds
- [x] Document: Metrics strategy
  - **Key metrics:** request rate, p50/p95 latency, error rate, saturation (CPU/memory/threadpool), fallback hit-rate.
  - **Dashboard:** service overview + endpoint breakdown + dependency health panels.
  - **Alerts:** error rate > 2% (5m), p95 latency > SLA, availability dưới mục tiêu SLO.

### Service Dependency Mapping
- [x] Map: Service dependencies
- [x] Create: Dependency graph
- [x] Identify: Critical dependencies
- [x] Identify: Single points of failure
- [x] Design: Dependency reduction
- [x] Document: Dependency map
  - **Critical chain:** Gateway -> Payment -> Risk/Wallet -> Notification.
  - **SPOF identified:** external risk provider và config/discovery control plane nếu không HA.
  - **Reduction plan:** giảm synchronous hops, ưu tiên async cho non-critical path, thêm fallback/cache cho external calls.

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
