# Week 9 – Distributed Systems Core Principles

> **Mentor Note**: Distributed systems là về trade-offs, không phải perfect solutions. CAP theorem forces bạn to choose. Bạn không thể have it all. Mỗi consistency choice bạn make có cost. Bạn phải justify MỌI choice với deep understanding của trade-offs. Systems researcher không accept "I think it's better". Phải có mathematical reasoning, proofs, và real-world evidence. Production systems need justified trade-offs, not wishful thinking.

---

## Study TODOs

### CAP Theorem Deep Dive
- [ ] Đọc "Brewer's CAP Theorem" original paper hoặc detailed explanation
- [ ] Định nghĩa chính xác: Consistency (linearizability)
- [ ] Định nghĩa chính xác: Availability (every request gets response)
- [ ] Định nghĩa chính xác: Partition tolerance (network partition)
- [ ] Viết proof: Why can only choose 2 of 3? (mathematical reasoning)
- [ ] Đọc về "CAP theorem misconceptions" - common misunderstandings
- [ ] Đọc về "CAP theorem in practice" - real-world implications
- [ ] Analyze: CP systems (Consistency + Partition tolerance)
- [ ] Analyze: AP systems (Availability + Partition tolerance)
- [ ] Analyze: CA systems (Consistency + Availability) - why impossible in distributed?
- [ ] Research: Real-world CP systems (examples)
- [ ] Research: Real-world AP systems (examples)
- [ ] Đọc về "CAP theorem refinements" - PACELC theorem

### Consistency Models
- [ ] Đọc về "linearizability" (strong consistency)
- [ ] Định nghĩa chính xác: Linearizability
- [ ] Đọc về "sequential consistency"
- [ ] So sánh: Linearizability vs Sequential consistency (5 differences)
- [ ] Đọc về "causal consistency"
- [ ] Định nghĩa chính xác: Causal consistency
- [ ] Đọc về "eventual consistency"
- [ ] Định nghĩa chính xác: Eventual consistency
- [ ] Đọc về "strong eventual consistency" (SEC)
- [ ] Đọc về "read-your-writes consistency"
- [ ] Đọc về "monotonic reads consistency"
- [ ] Đọc về "monotonic writes consistency"
- [ ] Đọc về "consistent prefix reads"
- [ ] Compare: All consistency models (10 criteria)
- [ ] Research: Which systems use which consistency model?

### ACID vs BASE
- [ ] Đọc về "ACID properties" (review from Week 3)
- [ ] Đọc về "BASE properties" - Basically Available, Soft state, Eventual consistency
- [ ] So sánh: ACID vs BASE (10 criteria)
- [ ] Analyze: When to use ACID? (5 use cases)
- [ ] Analyze: When to use BASE? (5 use cases)
- [ ] Đọc về "ACID limitations" trong distributed systems
- [ ] Đọc về "BASE trade-offs" - what you sacrifice
- [ ] Research: Real-world ACID systems
- [ ] Research: Real-world BASE systems

### Leader Election
- [ ] Đọc về "leader election" - why needed?
- [ ] Đọc về "single leader" pattern
- [ ] Đọc về "leader election algorithms"
- [ ] Đọc về "bully algorithm" - how it works
- [ ] Đọc về "ring algorithm" - how it works
- [ ] Đọc về "leader lease" - lease-based leadership
- [ ] Đọc về "split-brain" problem - multiple leaders
- [ ] Đọc về "fencing tokens" - prevent split-brain
- [ ] Research: Leader election implementations (Zookeeper, etcd)

### Consensus Algorithms - Raft
- [ ] Đọc "In Search of an Understandable Consensus Algorithm" (Raft paper)
- [ ] Đọc về "Raft algorithm" - overview
- [ ] Đọc về "Raft states" - Leader, Follower, Candidate
- [ ] Đọc về "Raft leader election" - how it works
- [ ] Đọc về "Raft log replication" - how it works
- [ ] Đọc về "Raft safety properties" - what it guarantees
- [ ] Đọc về "Raft liveness properties" - what it guarantees
- [ ] Đọc về "Raft vs Paxos" - comparison
- [ ] Research: Raft implementations

### Consensus Algorithms - Paxos
- [ ] Đọc về "Paxos algorithm" - overview (simplified)
- [ ] Đọc về "Paxos phases" - Prepare, Accept
- [ ] Đọc về "Paxos safety" - what it guarantees
- [ ] Đọc về "Multi-Paxos" - optimization
- [ ] Đọc về "Paxos complexity" - why hard to understand
- [ ] Đọc về "Raft vs Paxos" - why Raft easier?
- [ ] Research: Paxos use cases

### Time Synchronization
- [ ] Đọc về "distributed time" - why hard?
- [ ] Đọc về "physical clocks" - wall clock time
- [ ] Đọc về "logical clocks" - Lamport clocks
- [ ] Đọc về "vector clocks" - causality tracking
- [ ] Đọc về "NTP" (Network Time Protocol) - clock synchronization
- [ ] Đọc về "clock skew" - time differences
- [ ] Đọc về "happens-before relationship" - causality
- [ ] Đọc về "Lamport timestamps" - logical time
- [ ] Đọc về "vector clocks" - detailed explanation
- [ ] Research: Time synchronization in distributed systems

### Network Partitions
- [ ] Đọc về "network partition" - what is it?
- [ ] Đọc về "partition detection" - how to detect?
- [ ] Đọc về "partition handling" - what to do?
- [ ] Đọc về "minority partition" - what happens?
- [ ] Đọc về "majority partition" - what happens?
- [ ] Đọc về "quorum" - majority voting
- [ ] Đọc về "split-brain" - multiple partitions think they're primary
- [ ] Đọc về "fencing" - prevent split-brain
- [ ] Research: Network partition handling strategies

### Split-Brain Scenarios
- [ ] Đọc về "split-brain" problem - detailed explanation
- [ ] Analyze: What causes split-brain? (5 causes)
- [ ] Analyze: Consequences của split-brain (5 consequences)
- [ ] Đọc về "fencing tokens" - prevent split-brain
- [ ] Đọc về "quorum" - prevent split-brain
- [ ] Đọc về "lease-based leadership" - prevent split-brain
- [ ] Research: Real-world split-brain incidents
- [ ] Research: Split-brain prevention strategies

### Distributed Locking Deep Dive
- [ ] Đọc về "distributed locks" (review from Week 5, 7)
- [ ] Đọc về "lock correctness" - safety và liveness
- [ ] Đọc về "lock expiration" - why critical?
- [ ] Đọc về "lock renewal" - keep lock alive
- [ ] Đọc về "lock granularity" - fine-grained vs coarse-grained
- [ ] Đọc về "lock contention" - performance impact
- [ ] Đọc về "deadlock" trong distributed systems
- [ ] Đọc về "livelock" - different from deadlock
- [ ] Research: Distributed lock implementations (Redis, Zookeeper, etcd)

### Distributed Transactions
- [ ] Đọc về "distributed transactions" (review from Week 4, 7)
- [ ] Đọc về "2PC" (Two-Phase Commit) - detailed
- [ ] Đọc về "2PC phases" - Prepare, Commit
- [ ] Đọc về "2PC blocking problem" - coordinator fails
- [ ] Đọc về "3PC" (Three-Phase Commit) - improvement?
- [ ] Đọc về "Saga pattern" (review from Week 7)
- [ ] Compare: 2PC vs 3PC vs Saga (10 criteria)
- [ ] Research: Distributed transaction use cases

### Vector Clocks
- [ ] Đọc về "vector clocks" - detailed explanation
- [ ] Đọc về "causality tracking" với vector clocks
- [ ] Đọc về "vector clock algorithm" - how it works
- [ ] Đọc về "vector clock comparison" - happens-before
- [ ] Đọc về "concurrent events" - detected by vector clocks
- [ ] Research: Vector clock implementations

### CRDTs (Conflict-Free Replicated Data Types)
- [ ] Đọc về "CRDTs" - what are they?
- [ ] Đọc về "operation-based CRDTs"
- [ ] Đọc về "state-based CRDTs"
- [ ] Đọc về "CRDT properties" - commutativity, associativity
- [ ] Đọc về "CRDT examples" - counters, sets, maps
- [ ] Research: CRDT use cases

---

## Coordination Design TODOs

### Design Exercise 1: CAP Theorem Application
- [ ] Scenario: Payment system, network partition occurs
- [ ] Design: CP system (choose Consistency + Partition tolerance)
- [ ] Justify: Why CP? What you sacrifice? (write 500 words)
- [ ] Design: AP system (choose Availability + Partition tolerance)
- [ ] Justify: Why AP? What you sacrifice? (write 500 words)
- [ ] Compare: CP vs AP cho payment system
- [ ] Recommend: Which choice? Justify với trade-offs
- [ ] Document: CAP theorem application analysis

### Design Exercise 2: Consistency Model Selection
- [ ] Scenario: Wallet system với multiple replicas
- [ ] Design: Strong consistency (linearizability) strategy
- [ ] Analyze: Performance impact
- [ ] Analyze: Availability impact
- [ ] Design: Eventual consistency strategy
- [ ] Analyze: Consistency delay
- [ ] Analyze: Conflict resolution
- [ ] Compare: Strong vs Eventual (10 criteria)
- [ ] Recommend: Which consistency model? Justify
- [ ] Document: Consistency model selection

### Design Exercise 3: Leader Election Design
- [ ] Design: Leader election cho database cluster
- [ ] Choose: Algorithm (Bully? Ring? Raft?)
- [ ] Justify: Algorithm choice
- [ ] Design: Leader election process
- [ ] Design: Split-brain prevention
- [ ] Design: Leader failure handling
- [ ] Design: Leader lease mechanism
- [ ] Test: Leader election với node failures
- [ ] Document: Leader election design

### Design Exercise 4: Consensus Design
- [ ] Design: Consensus mechanism cho configuration updates
- [ ] Choose: Raft or Paxos? Justify
- [ ] Design: Consensus process
- [ ] Design: Failure handling
- [ ] Design: Performance optimization
- [ ] Analyze: Consensus overhead
- [ ] Document: Consensus design

### Design Exercise 5: Network Partition Handling
- [ ] Scenario: 5-node cluster, network partition (3 nodes vs 2 nodes)
- [ ] Design: Partition handling strategy
- [ ] Design: Quorum mechanism
- [ ] Design: Minority partition behavior
- [ ] Design: Split-brain prevention
- [ ] Design: Partition recovery
- [ ] Test: Partition scenarios
- [ ] Document: Partition handling design

### Design Exercise 6: Split-Brain Prevention
- [ ] Design: Split-brain prevention cho primary-secondary setup
- [ ] Design: Fencing mechanism
- [ ] Design: Quorum voting
- [ ] Design: Lease-based leadership
- [ ] Test: Split-brain scenarios
- [ ] Verify: Split-brain prevented
- [ ] Document: Split-brain prevention design

---

## Distributed Lock TODOs

### Task 1: Redis Distributed Lock Implementation
- [ ] Implement: Distributed lock với Redis SET NX (review from Week 5)
- [ ] Add: Lock expiration (prevent deadlock)
- [ ] Add: Lock renewal mechanism
- [ ] Add: Lock release verification (only owner can release)
- [ ] Test: Lock acquisition
- [ ] Test: Lock expiration
- [ ] Test: Lock renewal
- [ ] Test: Concurrent lock attempts
- [ ] Test: Lock release by owner
- [ ] Test: Lock release by non-owner (should fail)
- [ ] Document: Distributed lock implementation

### Task 2: Redlock Algorithm Implementation
- [ ] Implement: Redlock algorithm (review from Week 5)
- [ ] Setup: Multiple Redis instances (3-5)
- [ ] Implement: Lock acquisition across instances
- [ ] Implement: Quorum-based locking (majority)
- [ ] Implement: Lock expiration calculation
- [ ] Test: Lock với majority of instances
- [ ] Test: Failure scenarios (some instances down)
- [ ] Test: Clock skew scenarios
- [ ] Measure: Lock acquisition time
- [ ] Document: Redlock implementation

### Task 3: Lease-Based Locking
- [ ] Implement: Lease-based locking
- [ ] Add: Lock lease duration
- [ ] Add: Lock renewal (heartbeat)
- [ ] Add: Automatic lock expiration
- [ ] Test: Lock renewal
- [ ] Test: Lock expiration if renewal fails
- [ ] Test: Concurrent renewal attempts
- [ ] Compare: Lease-based vs expiration-based
- [ ] Document: Lease-based locking

### Task 4: Lock Granularity Analysis
- [ ] Implement: Coarse-grained lock (lock entire resource)
- [ ] Measure: Lock contention
- [ ] Measure: Throughput
- [ ] Implement: Fine-grained lock (lock specific parts)
- [ ] Measure: Lock contention
- [ ] Measure: Throughput
- [ ] Compare: Coarse vs Fine-grained (5 criteria)
- [ ] Recommend: Optimal granularity? Justify
- [ ] Document: Lock granularity analysis

### Task 5: Deadlock Detection
- [ ] Simulate: Deadlock scenario (circular wait)
- [ ] Implement: Deadlock detection
- [ ] Test: Deadlock detection works
- [ ] Implement: Deadlock prevention (lock ordering)
- [ ] Test: Deadlock prevented
- [ ] Document: Deadlock handling

---

## Failure Simulation TODOs

### Failure Scenario 1: Network Partition
- [ ] Simulate: Network partition (split cluster)
- [ ] Test: CP system behavior (unavailable?)
- [ ] Test: AP system behavior (available but inconsistent?)
- [ ] Measure: System behavior during partition
- [ ] Test: Partition recovery
- [ ] Measure: Recovery time
- [ ] Document: Network partition handling

### Failure Scenario 2: Split-Brain
- [ ] Simulate: Split-brain scenario (multiple leaders)
- [ ] Test: System behavior (data corruption?)
- [ ] Test: Fencing mechanism (if implemented)
- [ ] Test: Quorum mechanism (if implemented)
- [ ] Verify: Split-brain prevented
- [ ] Document: Split-brain prevention

### Failure Scenario 3: Leader Failure
- [ ] Simulate: Leader node failure
- [ ] Test: Leader election process
- [ ] Measure: Election time
- [ ] Test: Service availability during election
- [ ] Test: Data consistency during election
- [ ] Document: Leader failure handling

### Failure Scenario 4: Clock Skew
- [ ] Simulate: Clock skew between nodes
- [ ] Test: Distributed lock behavior
- [ ] Test: Timestamp-based operations
- [ ] Test: Lease expiration behavior
- [ ] Analyze: Impact of clock skew
- [ ] Design: Clock skew mitigation
- [ ] Document: Clock skew handling

### Failure Scenario 5: Consensus Failure
- [ ] Simulate: Consensus algorithm failure (not enough nodes)
- [ ] Test: System behavior (blocked?)
- [ ] Test: Quorum requirements
- [ ] Test: Recovery when nodes come back
- [ ] Document: Consensus failure handling

### Failure Scenario 6: Distributed Transaction Failure
- [ ] Simulate: 2PC coordinator failure
- [ ] Test: Blocking problem (participants blocked?)
- [ ] Test: Recovery mechanism
- [ ] Compare: 2PC vs Saga failure handling
- [ ] Document: Transaction failure handling

---

## Trade-off Analysis TODOs

### Analysis 1: Consistency vs Availability
- [ ] Scenario: Payment system, network partition
- [ ] Analyze: CP choice (consistency, sacrifice availability)
- [ ] Calculate: Availability impact (downtime percentage)
- [ ] Analyze: AP choice (availability, sacrifice consistency)
- [ ] Calculate: Consistency impact (stale reads percentage)
- [ ] Compare: CP vs AP (10 criteria)
- [ ] Recommend: Which trade-off? Justify với numbers
- [ ] Document: Consistency vs Availability analysis

### Analysis 2: Strong vs Eventual Consistency
- [ ] Scenario: Wallet balance updates
- [ ] Analyze: Strong consistency (linearizability)
- [ ] Measure: Latency impact
- [ ] Measure: Throughput impact
- [ ] Analyze: Eventual consistency
- [ ] Measure: Consistency delay
- [ ] Measure: Conflict rate
- [ ] Compare: Strong vs Eventual (10 criteria)
- [ ] Recommend: Which consistency? Justify
- [ ] Document: Consistency trade-off analysis

### Analysis 3: ACID vs BASE
- [ ] Scenario: E-commerce order processing
- [ ] Analyze: ACID approach
- [ ] Calculate: Performance cost
- [ ] Calculate: Scalability limitations
- [ ] Analyze: BASE approach
- [ ] Calculate: Consistency cost
- [ ] Calculate: Complexity cost
- [ ] Compare: ACID vs BASE (10 criteria)
- [ ] Recommend: Which approach? Justify
- [ ] Document: ACID vs BASE analysis

### Analysis 4: 2PC vs Saga
- [ ] Scenario: Distributed payment transaction
- [ ] Analyze: 2PC approach
- [ ] Calculate: Performance overhead
- [ ] Calculate: Blocking risk
- [ ] Analyze: Saga approach
- [ ] Calculate: Complexity overhead
- [ ] Calculate: Compensation cost
- [ ] Compare: 2PC vs Saga (10 criteria)
- [ ] Recommend: Which approach? Justify
- [ ] Document: Transaction approach analysis

### Analysis 5: Leader Election Overhead
- [ ] Analyze: Leader election overhead
- [ ] Calculate: Election time
- [ ] Calculate: Service unavailability during election
- [ ] Analyze: Leader-based vs leaderless
- [ ] Compare: Leader-based vs leaderless (10 criteria)
- [ ] Recommend: Which approach? Justify
- [ ] Document: Leader election analysis

### Analysis 6: Distributed Lock Performance
- [ ] Analyze: Distributed lock overhead
- [ ] Measure: Lock acquisition time
- [ ] Measure: Lock contention impact
- [ ] Measure: Throughput degradation
- [ ] Analyze: Alternatives (optimistic locking?)
- [ ] Compare: Pessimistic vs Optimistic locking
- [ ] Recommend: Which approach? Justify
- [ ] Document: Lock performance analysis

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Coordination Design exercises hoàn thành?
- [ ] Review: Tất cả Distributed Lock tasks hoàn thành?
- [ ] Review: Tất cả Failure scenarios simulated?
- [ ] Review: Tất cả Trade-off analyses hoàn thành?
- [ ] Rate: CAP theorem understanding (1-10)
- [ ] Rate: Consistency models understanding (1-10)
- [ ] Rate: Consensus algorithms understanding (1-10)
- [ ] Rate: Trade-off reasoning skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Theory Review
- [ ] Review: CAP theorem understanding
- [ ] Verify: Can explain why only 2 of 3?
- [ ] Verify: Can apply CAP to real systems?
- [ ] Review: Consistency models
- [ ] Verify: Can distinguish different consistency levels?
- [ ] Review: Consensus algorithms
- [ ] Verify: Can explain Raft basics?
- [ ] Document: Theory review findings

### Design Review
- [ ] Review: All coordination designs
- [ ] Check: CAP theorem applied correctly?
- [ ] Check: Consistency choices justified?
- [ ] Check: Trade-offs analyzed?
- [ ] Identify: 3 design improvements
- [ ] Propose: Design optimizations
- [ ] Document: Design review findings

### Code Review
- [ ] Review: Distributed lock implementation
- [ ] Check: Lock correctness (safety, liveness)?
- [ ] Check: Deadlock prevention?
- [ ] Check: Performance acceptable?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Trade-off Review
- [ ] Review: All trade-off analyses
- [ ] Verify: Trade-offs quantified (numbers)?
- [ ] Verify: Justifications clear?
- [ ] Verify: Real-world evidence?
- [ ] Identify: Missing trade-offs
- [ ] Document: Trade-off review

### Knowledge Check
- [ ] Explain: CAP theorem - why only 2 of 3? (viết proof, không xem notes)
- [ ] Explain: Linearizability vs Eventual consistency (viết comparison, không xem notes)
- [ ] Explain: Raft leader election - how it works (viết explanation, không xem notes)
- [ ] Explain: Split-brain problem - causes và prevention (viết analysis, không xem notes)
- [ ] Explain: 2PC blocking problem - why? (viết explanation, không xem notes)
- [ ] Solve: 5-node cluster, quorum = 3. Network partition: 3 nodes vs 2 nodes. Which partition can serve writes?
- [ ] Solve: System có 99.9% availability, network partition 1% of time. CP system availability? AP system availability?
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 distributed systems concepts còn confuse
- [ ] Write: 1 consistency choice bạn made và deep rationale
- [ ] Write: Confidence level cho Week 10 (1-10)
- [ ] Compare: Week 8 vs Week 9 progress
- [ ] Plan: Preparation cho Week 10 (Advanced Database Patterns)
- [ ] Set: Goals cho Week 10
- [ ] Document: Week 9 reflection (500 words)

### Mentor Questions (Answer these - justify với proofs)
- [ ] Q1: Payment system, network partition. Bạn chọn CP. Justify với mathematical reasoning. What's availability cost? (viết analysis với calculations)
- [ ] Q2: Wallet balance update. Bạn chọn eventual consistency. Justify. What's consistency delay? Conflict rate? (viết analysis với measurements)
- [ ] Q3: 5-node cluster, quorum = 3. 2 nodes fail. System available? Can serve writes? Prove với quorum reasoning. (viết proof)
- [ ] Q4: Split-brain scenario - 2 partitions both think they're primary. Làm sao prevent? Justify solution với safety properties. (viết prevention strategy với proof)
- [ ] Q5: 2PC coordinator fails after Prepare phase. Participants blocked? Why? Làm sao unblock? (viết analysis với solution)
- [ ] Q6: Strong consistency latency = 100ms, eventual consistency = 10ms. 1M requests/day. Total latency difference? Worth strong consistency? Justify. (viết cost-benefit analysis với numbers)
- [ ] Review: Answers có mathematical reasoning? Có proofs? Có real-world evidence? Có justify trade-offs?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Coordination Design TODOs: ✅ Complete với justifications
- [ ] Tất cả Distributed Lock TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Failure Simulation TODOs: ✅ Complete với test results
- [ ] Tất cả Trade-off Analysis TODOs: ✅ Complete với numbers và justifications
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Design documents saved: ✅ Yes
- [ ] Trade-off analyses saved: ✅ Yes
- [ ] Ready for Week 10: ✅ Yes

---

> **Mentor Final Check**: Distributed systems là về trade-offs, không phải perfect solutions. Nếu bạn không thể explain với mathematical reasoning tại sao bạn chọn consistency level này, bạn chưa understand đủ. Systems researcher requires proofs, not opinions. Hãy honest: Bạn có thể prove CAP theorem chưa? Bạn có thể justify consistency choices với numbers chưa? Bạn có thể reason về trade-offs mathematically chưa? Nếu chưa, làm lại. Production systems need justified trade-offs, not wishful thinking.
