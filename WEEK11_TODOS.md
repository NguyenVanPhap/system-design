# Week 11 – Monitoring, Security & Reliability Engineering

> **Mentor Note**: Systems WILL fail. It's not "if", it's "when". SRE lead assumes failure, designs for failure, và practices failure. Nếu bạn không have observability, bạn are flying blind. Nếu bạn không have tested incident response, bạn will panic when incident happens. Nếu bạn không have security hardening, bạn are vulnerable. Production systems need observability, security, và reliability from day one. Assume systems will fail, design accordingly.

---

## Observability TODOs

### Metrics Fundamentals
- [ ] Đọc "Site Reliability Engineering" - Monitoring Distributed Systems
- [ ] Định nghĩa chính xác: Metrics
- [ ] Đọc về "Four Golden Signals" - Latency, Traffic, Errors, Saturation
- [ ] Đọc về "RED method" - Rate, Errors, Duration
- [ ] Đọc về "USE method" - Utilization, Saturation, Errors
- [ ] Đọc về "counter" metrics - cumulative counts
- [ ] Đọc về "gauge" metrics - current value
- [ ] Đọc về "histogram" metrics - distribution
- [ ] Đọc về "summary" metrics - percentiles
- [ ] Research: Metrics best practices

### Logging Fundamentals
- [ ] Đọc về "structured logging" - JSON format
- [ ] Đọc về "log levels" - DEBUG, INFO, WARN, ERROR
- [ ] Đọc về "correlation IDs" - trace requests
- [ ] Đọc về "log aggregation" - centralized logging
- [ ] Đọc về "log retention" - how long to keep?
- [ ] Đọc về "log sampling" - reduce volume
- [ ] Đọc về "log parsing" - extract structured data
- [ ] Research: Logging best practices

### Distributed Tracing
- [ ] Đọc về "distributed tracing" (review from Week 8)
- [ ] Đọc về "OpenTelemetry" - observability standard
- [ ] Đọc về "trace" - request journey
- [ ] Đọc về "span" - single operation
- [ ] Đọc về "trace context" - propagation
- [ ] Đọc về "sampling" - reduce trace volume
- [ ] Đọc về "trace analysis" - find bottlenecks
- [ ] Research: OpenTelemetry implementations

### SLI, SLO, SLA
- [ ] Đọc về "SLI" (Service Level Indicator) - what to measure
- [ ] Đọc về "SLO" (Service Level Objective) - target value
- [ ] Đọc về "SLA" (Service Level Agreement) - contract
- [ ] Đọc về "error budget" - allowed failures
- [ ] Đọc về "error budget policy" - what to do when exhausted
- [ ] Đọc về "SLO design" - how to set SLOs
- [ ] Đọc về "SLO types" - availability, latency, throughput
- [ ] Research: SLO best practices

### Alerting Strategies
- [ ] Đọc về "alerting" - when to alert?
- [ ] Đọc về "alert fatigue" - too many alerts
- [ ] Đọc về "alert levels" - P1, P2, P3, P4
- [ ] Đọc về "alert routing" - who gets notified?
- [ ] Đọc về "alert grouping" - reduce noise
- [ ] Đọc về "alert suppression" - prevent duplicates
- [ ] Đọc về "runbook" - incident response procedures
- [ ] Research: Alerting best practices

### Prometheus & Grafana
- [ ] Đọc về "Prometheus" - metrics collection
- [ ] Đọc về "PromQL" - query language
- [ ] Đọc về "Grafana" - visualization
- [ ] Đọc về "recording rules" - pre-compute queries
- [ ] Đọc về "alerting rules" - alert conditions
- [ ] Research: Prometheus best practices

### OpenTelemetry Setup
- [ ] Đọc về "OpenTelemetry" - observability framework
- [ ] Đọc về "OTEL collectors" - data collection
- [ ] Đọc về "OTEL exporters" - send to backends
- [ ] Đọc về "OTEL instrumentation" - auto-instrumentation
- [ ] Research: OpenTelemetry architecture

---

## SRE TODOs

### SLO Design Exercise
- [ ] Design: SLIs cho Payment API
- [ ] SLI 1: Availability (uptime percentage)
- [ ] SLI 2: Latency (p95 response time)
- [ ] SLI 3: Error rate (4xx, 5xx errors)
- [ ] Design: SLOs cho each SLI
- [ ] SLO 1: 99.9% availability
- [ ] SLO 2: p95 < 200ms
- [ ] SLO 3: Error rate < 0.1%
- [ ] Calculate: Error budget (allowed downtime)
- [ ] Design: Error budget policy (what to do when exhausted)
- [ ] Document: SLO design (500 words)

### Monitoring Architecture Design
- [ ] Design: Complete monitoring architecture
- [ ] Design: Metrics collection (Prometheus)
- [ ] Design: Log aggregation (ELK hoặc Loki)
- [ ] Design: Distributed tracing (Jaeger/Zipkin)
- [ ] Design: Alerting system (Alertmanager)
- [ ] Design: Dashboard strategy (Grafana)
- [ ] Design: Data retention policies
- [ ] Document: Monitoring architecture

### Alerting Rules Design
- [ ] Design: Alerting rules cho Payment System
- [ ] Alert 1: High error rate (> 1%)
- [ ] Alert 2: High latency (p95 > 500ms)
- [ ] Alert 3: Low availability (< 99%)
- [ ] Alert 4: High CPU usage (> 80%)
- [ ] Alert 5: High memory usage (> 90%)
- [ ] Design: Alert severity levels
- [ ] Design: Alert routing (who gets notified)
- [ ] Design: Alert runbooks (response procedures)
- [ ] Document: Alerting strategy

### Runbook Creation
- [ ] Create: Runbook cho high error rate alert
- [ ] Steps: 1) Check error logs, 2) Identify error pattern, 3) Check recent deployments, 4) Rollback if needed
- [ ] Create: Runbook cho high latency alert
- [ ] Steps: 1) Check slow queries, 2) Check database load, 3) Check cache hit rate, 4) Scale if needed
- [ ] Create: Runbook cho service down alert
- [ ] Steps: 1) Check health endpoints, 2) Check dependencies, 3) Restart service, 4) Escalate if needed
- [ ] Test: Runbooks với simulated incidents
- [ ] Document: All runbooks

### Error Budget Tracking
- [ ] Implement: Error budget calculation
- [ ] Track: Error budget consumption
- [ ] Alert: When error budget < 20%
- [ ] Alert: When error budget exhausted
- [ ] Design: Error budget policy (freeze deployments?)
- [ ] Document: Error budget tracking

---

## Security TODOs

### Zero Trust Security
- [ ] Đọc về "Zero Trust" security model
- [ ] Đọc về "never trust, always verify"
- [ ] Đọc về "least privilege" principle
- [ ] Đọc về "defense in depth" - multiple layers
- [ ] Đọc về "network segmentation"
- [ ] Đọc về "identity-based access"
- [ ] Research: Zero Trust implementations

### OAuth2 & JWT Deep Dive
- [ ] Đọc về "OAuth2" - authorization framework
- [ ] Đọc về "OAuth2 flows" - Authorization Code, Client Credentials, etc.
- [ ] Đọc về "JWT" (JSON Web Token) - token format
- [ ] Đọc về "JWT structure" - header, payload, signature
- [ ] Đọc về "JWT security" - token validation
- [ ] Đọc về "refresh tokens" - token renewal
- [ ] Đọc về "token expiration" - security
- [ ] Research: OAuth2/JWT best practices

### Authentication Implementation
- [ ] Implement: OAuth2 Authorization Server
- [ ] Implement: JWT token generation
- [ ] Implement: JWT token validation
- [ ] Implement: Refresh token mechanism
- [ ] Implement: Token revocation
- [ ] Test: Authentication flow
- [ ] Test: Token expiration
- [ ] Test: Token refresh
- [ ] Document: Authentication implementation

### Authorization Implementation
- [ ] Implement: Role-based access control (RBAC)
- [ ] Implement: Permission checks
- [ ] Implement: Resource-level authorization
- [ ] Test: Authorization enforcement
- [ ] Test: Unauthorized access blocked
- [ ] Document: Authorization implementation

### Secrets Management
- [ ] Đọc về "secrets management" - why critical?
- [ ] Đọc về "HashiCorp Vault" - secrets manager
- [ ] Đọc về "AWS Secrets Manager" - cloud solution
- [ ] Đọc về "secrets rotation" - automatic rotation
- [ ] Đọc về "secrets encryption" - at rest và in transit
- [ ] Implement: Secrets management integration
- [ ] Test: Secrets retrieval
- [ ] Test: Secrets rotation
- [ ] Document: Secrets management

### API Security
- [ ] Implement: Rate limiting (review from Week 6)
- [ ] Implement: Input validation
- [ ] Implement: SQL injection prevention
- [ ] Implement: XSS prevention
- [ ] Implement: CSRF protection
- [ ] Implement: API key authentication
- [ ] Test: Security vulnerabilities
- [ ] Document: API security measures

### WAF (Web Application Firewall)
- [ ] Đọc về "WAF" - what is it?
- [ ] Đọc về "WAF rules" - attack patterns
- [ ] Đọc về "WAF deployment" - inline vs out-of-band
- [ ] Research: WAF solutions (Cloudflare, AWS WAF)

### Encryption
- [ ] Đọc về "encryption at rest" - data encryption
- [ ] Đọc về "encryption in transit" - TLS/SSL
- [ ] Đọc về "key management" - encryption keys
- [ ] Implement: TLS/SSL configuration
- [ ] Test: Encryption verification
- [ ] Document: Encryption strategy

### Audit Logging
- [ ] Design: Audit logging strategy
- [ ] Log: All authentication attempts
- [ ] Log: All authorization decisions
- [ ] Log: All data access
- [ ] Log: All configuration changes
- [ ] Implement: Audit logging
- [ ] Test: Audit log generation
- [ ] Document: Audit logging

---

## Incident Response TODOs

### Incident Response Plan
- [ ] Design: Complete incident response plan
- [ ] Define: Incident severity levels (P1-P4)
- [ ] Define: On-call rotation
- [ ] Define: Escalation procedures
- [ ] Define: Communication channels
- [ ] Define: Post-incident review process
- [ ] Document: Incident response plan (1000 words)

### Incident Simulation 1: Service Outage
- [ ] Simulate: Payment service completely down
- [ ] Test: Alert triggers correctly
- [ ] Test: On-call engineer notified
- [ ] Test: Incident response procedure
- [ ] Test: Communication (status page, internal)
- [ ] Test: Root cause analysis
- [ ] Test: Service recovery
- [ ] Test: Post-incident review
- [ ] Document: Incident simulation results

### Incident Simulation 2: High Error Rate
- [ ] Simulate: Error rate spikes to 10%
- [ ] Test: Alert triggers
- [ ] Test: Investigation procedure
- [ ] Test: Rollback procedure (if deployment issue)
- [ ] Test: Service recovery
- [ ] Document: Incident handling

### Incident Simulation 3: Database Failure
- [ ] Simulate: Primary database failure
- [ ] Test: Failover procedure
- [ ] Test: Data consistency verification
- [ ] Test: Service recovery
- [ ] Measure: RTO (Recovery Time Objective)
- [ ] Measure: RPO (Recovery Point Objective)
- [ ] Document: Database failure incident

### Incident Simulation 4: Security Breach
- [ ] Simulate: Unauthorized access detected
- [ ] Test: Security alert triggers
- [ ] Test: Incident response (contain, investigate, remediate)
- [ ] Test: Communication (internal, external if needed)
- [ ] Test: Forensic analysis
- [ ] Document: Security incident response

### Post-Incident Review
- [ ] Create: Post-incident review template
- [ ] Sections: Timeline, Root cause, Impact, Actions taken, Lessons learned, Action items
- [ ] Practice: Write post-incident review cho simulated incident
- [ ] Document: Post-incident review process

### On-Call Setup
- [ ] Design: On-call rotation schedule
- [ ] Design: On-call handoff procedure
- [ ] Design: On-call escalation path
- [ ] Design: On-call compensation/benefits
- [ ] Document: On-call setup

---

## Chaos TODOs

### Chaos Engineering Fundamentals
- [ ] Đọc "Chaos Engineering" by Casey Rosenthal
- [ ] Đọc về "chaos engineering" - what is it?
- [ ] Đọc về "chaos experiments" - controlled failures
- [ ] Đọc về "chaos principles" - start small, automate, learn
- [ ] Đọc về "chaos tools" - Chaos Monkey, Gremlin, etc.
- [ ] Research: Chaos engineering best practices

### Chaos Experiment 1: Service Failure
- [ ] Design: Chaos experiment - kill random service
- [ ] Hypothesis: System degrades gracefully
- [ ] Run: Experiment (kill service)
- [ ] Observe: System behavior
- [ ] Measure: Impact (errors, latency)
- [ ] Verify: Hypothesis (graceful degradation?)
- [ ] Document: Experiment results

### Chaos Experiment 2: Network Latency
- [ ] Design: Chaos experiment - add network latency
- [ ] Hypothesis: System handles latency gracefully
- [ ] Run: Experiment (add 500ms latency)
- [ ] Observe: System behavior
- [ ] Measure: Impact (timeouts, errors)
- [ ] Verify: Hypothesis
- [ ] Document: Experiment results

### Chaos Experiment 3: Database Slowdown
- [ ] Design: Chaos experiment - slow database queries
- [ ] Hypothesis: Circuit breaker opens, fallback works
- [ ] Run: Experiment (slow queries)
- [ ] Observe: Circuit breaker behavior
- [ ] Observe: Fallback mechanism
- [ ] Verify: Hypothesis
- [ ] Document: Experiment results

### Chaos Experiment 4: Memory Leak
- [ ] Design: Chaos experiment - simulate memory leak
- [ ] Hypothesis: System detects và handles memory pressure
- [ ] Run: Experiment (consume memory)
- [ ] Observe: Memory monitoring
- [ ] Observe: System behavior (OOM? graceful degradation?)
- [ ] Verify: Hypothesis
- [ ] Document: Experiment results

### Chaos Experiment 5: Network Partition
- [ ] Design: Chaos experiment - network partition
- [ ] Hypothesis: System handles partition correctly
- [ ] Run: Experiment (partition network)
- [ ] Observe: System behavior (CP? AP?)
- [ ] Measure: Impact
- [ ] Verify: Hypothesis
- [ ] Document: Experiment results

### Chaos Experiment 6: Cascading Failure
- [ ] Design: Chaos experiment - trigger cascading failure
- [ ] Hypothesis: Circuit breakers prevent cascading
- [ ] Run: Experiment (fail service, observe cascade)
- [ ] Observe: Cascading behavior
- [ ] Observe: Circuit breaker effectiveness
- [ ] Verify: Hypothesis
- [ ] Document: Experiment results

### Chaos Engineering Automation
- [ ] Design: Automated chaos experiments
- [ ] Schedule: Regular chaos experiments (weekly?)
- [ ] Automate: Experiment execution
- [ ] Automate: Result collection
- [ ] Automate: Reporting
- [ ] Document: Chaos automation

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Observability TODOs hoàn thành?
- [ ] Review: Tất cả SRE exercises hoàn thành?
- [ ] Review: Tất cả Security tasks hoàn thành?
- [ ] Review: Tất cả Incident simulations tested?
- [ ] Review: Tất cả Chaos experiments run?
- [ ] Rate: Observability skills (1-10)
- [ ] Rate: SRE skills (1-10)
- [ ] Rate: Security skills (1-10)
- [ ] Rate: Incident response skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Observability Review
- [ ] Review: Monitoring setup
- [ ] Check: All critical metrics collected?
- [ ] Check: Dashboards useful?
- [ ] Check: Alerting rules appropriate?
- [ ] Check: Logging comprehensive?
- [ ] Check: Tracing implemented?
- [ ] Identify: Observability gaps
- [ ] Document: Observability review

### Security Review
- [ ] Review: Security implementation
- [ ] Check: Authentication secure?
- [ ] Check: Authorization enforced?
- [ ] Check: Secrets managed securely?
- [ ] Check: Encryption implemented?
- [ ] Check: Audit logging enabled?
- [ ] Identify: Security vulnerabilities
- [ ] Document: Security review

### Incident Response Review
- [ ] Review: Incident response plan
- [ ] Check: Plan complete?
- [ ] Check: Procedures tested?
- [ ] Check: Runbooks up-to-date?
- [ ] Check: On-call setup ready?
- [ ] Identify: Incident response improvements
- [ ] Document: Incident response review

### Chaos Engineering Review
- [ ] Review: Chaos experiments
- [ ] Check: Experiments cover critical failures?
- [ ] Check: Experiments automated?
- [ ] Check: Results documented?
- [ ] Identify: Additional chaos experiments needed
- [ ] Document: Chaos engineering review

### Knowledge Check
- [ ] Explain: SLI vs SLO vs SLA (viết comparison, không xem notes)
- [ ] Explain: Four Golden Signals (viết explanation, không xem notes)
- [ ] Explain: OAuth2 Authorization Code flow (viết flow, không xem notes)
- [ ] Explain: Zero Trust security model (viết explanation, không xem notes)
- [ ] Explain: Chaos engineering principles (viết explanation, không xem notes)
- [ ] Solve: SLO = 99.9% availability. Error budget for 30 days? (calculate)
- [ ] Solve: p95 latency SLO = 200ms. Current p95 = 250ms. SLO violated? (analyze)
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 SRE concepts còn confuse
- [ ] Write: 1 incident response lesson learned
- [ ] Write: Confidence level cho Week 12 (1-10)
- [ ] Compare: Week 10 vs Week 11 progress
- [ ] Plan: Preparation cho Week 12 (Final Case Studies)
- [ ] Set: Goals cho Week 12
- [ ] Document: Week 11 reflection (500 words)

### Mentor Questions (Answer these - assume failure)
- [ ] Q1: System có 99.9% SLO. Error budget = 43 minutes/month. Current downtime = 50 minutes. What to do? (viết error budget policy với actions)
- [ ] Q2: Payment service down. No alerts fired. Why? Làm sao prevent? (viết analysis với monitoring improvements)
- [ ] Q3: Security breach detected - unauthorized API access. Incident response? Step-by-step. (viết incident response procedure)
- [ ] Q4: Chaos experiment - kill database. System completely down. No graceful degradation. Làm sao fix? (viết resilience improvements)
- [ ] Q5: p95 latency SLO = 200ms. Current = 300ms. No alerts. Why? Làm sao fix alerting? (viết alerting improvements)
- [ ] Q6: On-call engineer gets 50 alerts/night. 90% false positives. Alert fatigue. Làm sao fix? (viết alerting optimization strategy)
- [ ] Review: Answers có assume systems will fail? Có have tested procedures? Có prioritize reliability?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Observability TODOs: ✅ Complete
- [ ] Tất cả SRE TODOs: ✅ Complete với SLOs và runbooks
- [ ] Tất cả Security TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Incident Response TODOs: ✅ Complete với tested procedures
- [ ] Tất cả Chaos TODOs: ✅ Complete với experiment results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Monitoring dashboards created: ✅ Yes
- [ ] Incident response plan tested: ✅ Yes
- [ ] Chaos experiments documented: ✅ Yes
- [ ] Ready for Week 12: ✅ Yes

---

> **Mentor Final Check**: Systems WILL fail. Nếu bạn không have observability, bạn are blind. Nếu bạn không have tested incident response, bạn will panic. Nếu bạn không have security hardening, bạn are vulnerable. SRE lead assumes failure và designs for it. Hãy honest: Bạn có tested incident response chưa? Bạn có run chaos experiments chưa? Bạn có verified security chưa? Nếu chưa, làm lại. Production systems need observability, security, và reliability from day one. Assume systems will fail, design accordingly.
