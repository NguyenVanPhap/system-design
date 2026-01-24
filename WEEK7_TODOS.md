# Week 7 – Message Queues & Event-Driven Architecture

> **Mentor Note**: Message queues là backbone của distributed systems. Nhưng messages có thể lost, duplicated, hoặc out-of-order. Bạn phải reason về MỌI failure scenario. Không có "message sent, done". Phải có "message sent, acknowledged, processed, verified". Production systems need message reliability, not hope. Mỗi message delivery guarantee phải được understood, tested, và verified.

---

## Study TODOs

### Message Queue Fundamentals
- [ ] Đọc "Designing Data-Intensive Applications" Chapter 11 - Stream Processing
- [ ] Định nghĩa chính xác: Message queue
- [ ] Định nghĩa chính xác: Message broker
- [ ] Định nghĩa chính xác: Producer, Consumer, Broker
- [ ] Đọc về "point-to-point" messaging pattern
- [ ] Đọc về "publish-subscribe" (pub/sub) pattern
- [ ] So sánh: Point-to-point vs Pub/Sub (5 differences)
- [ ] Analyze: When to use point-to-point? When pub/sub? (5 use cases each)
- [ ] Đọc về "message durability" - what is it?
- [ ] Đọc về "message persistence" - how it works

### Kafka Fundamentals
- [ ] Đọc về "Apache Kafka" - what is it?
- [ ] Đọc về "Kafka topics" - how they work
- [ ] Đọc về "Kafka partitions" - why partitions?
- [ ] Đọc về "partition keys" - how they work
- [ ] Đọc về "consumer groups" - how they work
- [ ] Đọc về "offset" - what is it? Why important?
- [ ] Đọc về "Kafka brokers" - cluster architecture
- [ ] Đọc về "replication factor" - how replication works
- [ ] Đọc về "leader và followers" trong partitions
- [ ] Đọc về "Kafka retention" - how long messages kept?
- [ ] Research: Kafka use cases và limitations

### RabbitMQ Fundamentals
- [ ] Đọc về "RabbitMQ" - what is it?
- [ ] Đọc về "exchanges" - types (direct, topic, fanout, headers)
- [ ] Đọc về "queues" - how they work
- [ ] Đọc về "bindings" - routing messages
- [ ] Đọc về "routing keys" - how routing works
- [ ] Đọc về "durable queues" - persistence
- [ ] Đọc về "message acknowledgment" - ack vs nack
- [ ] Đọc về "prefetch count" - consumer prefetching
- [ ] Research: RabbitMQ vs Kafka - when to use which?

### Message Ordering
- [ ] Đọc về "message ordering" - why important?
- [ ] Đọc về "FIFO ordering" - first-in-first-out
- [ ] Đọc về "partition-based ordering" trong Kafka
- [ ] Analyze: How to ensure ordering trong Kafka?
- [ ] Analyze: How to ensure ordering trong RabbitMQ?
- [ ] Đọc về "out-of-order messages" - causes và solutions
- [ ] Đọc về "idempotent processing" - handle duplicates
- [ ] Research: Ordering guarantees của different MQs

### Message Delivery Guarantees
- [ ] Đọc về "at-most-once" delivery - what is it?
- [ ] Analyze: Pros và cons của at-most-once (5 points each)
- [ ] Đọc về "at-least-once" delivery - what is it?
- [ ] Analyze: Pros và cons của at-least-once (5 points each)
- [ ] Đọc về "exactly-once" delivery - what is it?
- [ ] Analyze: Pros và cons của exactly-once (5 points each)
- [ ] Đọc về "idempotency" - how to achieve exactly-once semantics
- [ ] Compare: All delivery guarantees (10 criteria)
- [ ] Research: Which MQs support which guarantees?

### Message Acknowledgment
- [ ] Đọc về "message acknowledgment" - why needed?
- [ ] Đọc về "auto-ack" - automatic acknowledgment
- [ ] Analyze: Pros và cons của auto-ack (5 points each)
- [ ] Đọc về "manual ack" - explicit acknowledgment
- [ ] Analyze: Pros và cons của manual ack (5 points each)
- [ ] Đọc về "ack timeout" - what happens if timeout?
- [ ] Đọc về "negative acknowledgment" (nack) - when to use?
- [ ] Đọc về "requeue" - retry failed messages
- [ ] Research: Acknowledgment best practices

### Dead Letter Queues
- [ ] Đọc về "dead letter queue" (DLQ) - what is it?
- [ ] Đọc về "DLQ use cases" - when messages go to DLQ?
- [ ] Đọc về "message TTL" - time-to-live expiration
- [ ] Đọc về "max retry count" - retry limit
- [ ] Đọc về "DLQ processing" - how to handle DLQ messages?
- [ ] Đọc về "DLQ monitoring" - alert on DLQ messages
- [ ] Research: DLQ patterns và best practices
- [ ] Analyze: When to use DLQ? When to retry?

### Event-Driven Architecture
- [ ] Đọc về "event-driven architecture" (EDA) - what is it?
- [ ] Đọc về "events" vs "commands" - differences
- [ ] Đọc về "event sourcing" - storing events
- [ ] Đọc về "CQRS" (Command Query Responsibility Segregation)
- [ ] Đọc về "eventual consistency" - in EDA context
- [ ] Đọc về "event replay" - rebuilding state from events
- [ ] Đọc về "event versioning" - how to version events?
- [ ] Research: EDA patterns và anti-patterns

### Event Sourcing
- [ ] Đọc về "event sourcing" - detailed explanation
- [ ] Đọc về "event store" - where events stored?
- [ ] Đọc về "event stream" - sequence of events
- [ ] Đọc về "snapshots" - performance optimization
- [ ] Đọc về "event replay" - rebuilding aggregates
- [ ] Analyze: Pros và cons của event sourcing (5 points each)
- [ ] Research: Event sourcing use cases
- [ ] Đọc về "event versioning" - schema evolution

### Saga Pattern
- [ ] Đọc về "Saga pattern" - distributed transactions
- [ ] Đọc về "choreography" saga - decentralized
- [ ] Đọc về "orchestration" saga - centralized
- [ ] Compare: Choreography vs Orchestration (10 criteria)
- [ ] Đọc về "compensating transactions" - rollback
- [ ] Đọc về "saga failure handling" - what if step fails?
- [ ] Research: Saga pattern implementations

### Message Retry Strategies
- [ ] Đọc về "retry strategies" - why needed?
- [ ] Đọc về "exponential backoff" - retry with delay
- [ ] Đọc về "fixed delay" - constant retry interval
- [ ] Đọc về "max retry attempts" - retry limit
- [ ] Đọc về "retry jitter" - randomize retry time
- [ ] Compare: Retry strategies (5 criteria)
- [ ] Research: Retry best practices

### Message Idempotency
- [ ] Đọc về "idempotent consumers" - handle duplicates
- [ ] Đọc về "idempotency keys" - unique message IDs
- [ ] Đọc về "idempotency storage" - where to store?
- [ ] Đọc về "idempotency TTL" - how long to remember?
- [ ] Đọc về "idempotency scope" - per message? per user?
- [ ] Research: Idempotency implementation patterns

---

## Event Design TODOs

### Design Exercise 1: Payment Event Schema
- [ ] Design: Event schema cho payment creation
- [ ] Fields: event_id, timestamp, payment_id, amount, merchant_id, status
- [ ] Design: Event versioning strategy
- [ ] Design: Event schema evolution (backward compatibility)
- [ ] Design: Event validation rules
- [ ] Document: Complete event schema

### Design Exercise 2: Wallet Event-Driven Architecture
- [ ] Design: Event-driven wallet system architecture
- [ ] Requirement: Async balance updates, transaction events
- [ ] Design: Event types (TransactionCreated, BalanceUpdated, etc.)
- [ ] Design: Event producers (which services publish?)
- [ ] Design: Event consumers (which services subscribe?)
- [ ] Design: Event flow diagram
- [ ] Design: Eventual consistency handling
- [ ] Document: Complete EDA architecture (1000 words)

### Design Exercise 3: Event Sourcing Design
- [ ] Design: Event sourcing cho transaction history
- [ ] Design: Event store schema
- [ ] Design: Event types và schemas
- [ ] Design: Snapshot strategy (when to snapshot?)
- [ ] Design: Event replay mechanism
- [ ] Design: Event versioning và migration
- [ ] Document: Event sourcing design

### Design Exercise 4: Saga Pattern Design
- [ ] Design: Saga cho payment processing
- [ ] Steps: Validate payment → Charge card → Update balance → Send notification
- [ ] Choose: Choreography or Orchestration? Justify
- [ ] Design: Compensating transactions cho each step
- [ ] Design: Failure handling (what if step fails?)
- [ ] Design: Saga state management
- [ ] Document: Saga design

### Design Exercise 5: Message Queue Topology
- [ ] Design: Message queue topology cho payment system
- [ ] Design: Topics/Queues needed
- [ ] Design: Producer services
- [ ] Design: Consumer services
- [ ] Design: Message routing
- [ ] Design: Dead letter queues
- [ ] Document: Queue topology diagram

### Design Exercise 6: Event Ordering Strategy
- [ ] Scenario: Payment events must be processed in order
- [ ] Design: Ordering strategy (partition key? single consumer?)
- [ ] Analyze: Trade-offs của ordering strategy
- [ ] Design: Out-of-order detection
- [ ] Design: Handling out-of-order messages
- [ ] Document: Ordering strategy

---

## MQ Integration TODOs

### Task 1: RabbitMQ Setup & Basic Operations
- [ ] Install: RabbitMQ server (Docker hoặc local)
- [ ] Setup: Spring Boot với RabbitMQ dependency
- [ ] Configure: RabbitMQ connection (host, port, credentials)
- [ ] Create: Exchange (direct type)
- [ ] Create: Queue
- [ ] Create: Binding (exchange to queue)
- [ ] Test: Basic publish và consume
- [ ] Document: RabbitMQ setup

### Task 2: RabbitMQ Producer Implementation
- [ ] Implement: Message producer service
- [ ] Method: sendMessage(exchange, routingKey, message)
- [ ] Configure: Message durability
- [ ] Configure: Message persistence
- [ ] Add: Message headers (correlation ID, timestamp)
- [ ] Add: Error handling (connection failures)
- [ ] Test: Message publishing
- [ ] Document: Producer implementation

### Task 3: RabbitMQ Consumer Implementation
- [ ] Implement: Message consumer với @RabbitListener
- [ ] Configure: Manual acknowledgment
- [ ] Implement: Message processing logic
- [ ] Implement: Acknowledgment after successful processing
- [ ] Implement: Negative acknowledgment (nack) on failure
- [ ] Configure: Prefetch count
- [ ] Test: Message consumption
- [ ] Test: Acknowledgment behavior
- [ ] Document: Consumer implementation

### Task 4: Dead Letter Queue Implementation
- [ ] Create: Dead letter exchange
- [ ] Create: Dead letter queue
- [ ] Configure: Queue với DLX (dead letter exchange)
- [ ] Configure: Message TTL (time-to-live)
- [ ] Configure: Max retry count
- [ ] Implement: DLQ message handler
- [ ] Test: Messages go to DLQ after max retries
- [ ] Test: DLQ message processing
- [ ] Document: DLQ implementation

### Task 5: Kafka Setup & Basic Operations
- [ ] Install: Kafka và Zookeeper (Docker hoặc local)
- [ ] Setup: Spring Boot với Kafka dependency
- [ ] Configure: Kafka producer properties
- [ ] Configure: Kafka consumer properties
- [ ] Create: Kafka topic với partitions
- [ ] Test: Basic produce và consume
- [ ] Document: Kafka setup

### Task 6: Kafka Producer Implementation
- [ ] Implement: Kafka producer service
- [ ] Method: sendMessage(topic, key, message)
- [ ] Configure: Producer acks (0, 1, all)
- [ ] Configure: Retries và retry backoff
- [ ] Configure: Idempotent producer
- [ ] Add: Partition key selection
- [ ] Test: Message publishing
- [ ] Test: Partitioning behavior
- [ ] Document: Producer implementation

### Task 7: Kafka Consumer Implementation
- [ ] Implement: Kafka consumer với @KafkaListener
- [ ] Configure: Consumer group
- [ ] Configure: Auto offset commit (enable/disable)
- [ ] Implement: Manual offset commit
- [ ] Implement: Message processing logic
- [ ] Configure: Consumer parallelism (multiple consumers)
- [ ] Test: Message consumption
- [ ] Test: Consumer group behavior
- [ ] Test: Offset management
- [ ] Document: Consumer implementation

### Task 8: Kafka Consumer Groups
- [ ] Setup: Multiple consumers trong same group
- [ ] Test: Partition assignment (each consumer gets partitions)
- [ ] Test: Consumer failure (partitions reassigned)
- [ ] Test: New consumer joins (rebalancing)
- [ ] Measure: Rebalancing time
- [ ] Document: Consumer group behavior

### Task 9: Message Ordering với Kafka
- [ ] Setup: Topic với multiple partitions
- [ ] Test: Messages với same partition key stay ordered
- [ ] Test: Messages với different keys can be out-of-order
- [ ] Implement: Single partition strategy (ensure ordering)
- [ ] Test: Ordering guarantee
- [ ] Analyze: Trade-offs (throughput vs ordering)
- [ ] Document: Ordering strategy

### Task 10: Event Sourcing Implementation
- [ ] Design: Event store (database table hoặc Kafka topic)
- [ ] Implement: Event publishing
- [ ] Implement: Event storage
- [ ] Implement: Event replay mechanism
- [ ] Implement: Snapshot creation
- [ ] Implement: State rebuilding từ events
- [ ] Test: Event replay
- [ ] Test: Snapshot usage
- [ ] Document: Event sourcing implementation

---

## Failure & Retry TODOs

### Failure Scenario 1: Message Loss
- [ ] Simulate: Message loss scenario (producer fails before ack)
- [ ] Test: Message not delivered
- [ ] Implement: Solution (producer retry, idempotent consumer)
- [ ] Test: Solution effectiveness
- [ ] Measure: Message loss rate
- [ ] Document: Message loss prevention

### Failure Scenario 2: Duplicate Messages
- [ ] Simulate: Duplicate message delivery
- [ ] Test: Consumer receives duplicate
- [ ] Implement: Idempotent consumer
- [ ] Test: Duplicate handling (no side effects)
- [ ] Measure: Duplicate detection accuracy
- [ ] Document: Duplicate handling

### Failure Scenario 3: Consumer Failure
- [ ] Simulate: Consumer crashes during processing
- [ ] Test: Message acknowledgment behavior
- [ ] Test: Message requeue (if not acked)
- [ ] Test: Consumer recovery (reprocess messages)
- [ ] Measure: Message reprocessing time
- [ ] Document: Consumer failure handling

### Failure Scenario 4: Broker Failure
- [ ] Simulate: Message broker failure
- [ ] Test: Producer behavior (connection lost)
- [ ] Test: Consumer behavior (connection lost)
- [ ] Implement: Retry logic với exponential backoff
- [ ] Test: Automatic reconnection
- [ ] Test: Message recovery after broker restart
- [ ] Document: Broker failure handling

### Failure Scenario 5: Out-of-Order Messages
- [ ] Simulate: Out-of-order message delivery
- [ ] Test: Consumer receives messages out of order
- [ ] Implement: Ordering mechanism (buffering, sequencing)
- [ ] Test: Messages processed in order
- [ ] Measure: Ordering overhead
- [ ] Document: Ordering solution

### Failure Scenario 6: Message Poisoning
- [ ] Simulate: Poison message (always fails processing)
- [ ] Test: Message retries infinitely
- [ ] Implement: Max retry count
- [ ] Test: Message goes to DLQ after max retries
- [ ] Implement: DLQ monitoring và alerting
- [ ] Document: Poison message handling

### Retry Strategy Implementation
- [ ] Implement: Exponential backoff retry
- [ ] Configure: Initial delay, max delay, multiplier
- [ ] Add: Retry jitter (randomization)
- [ ] Test: Retry behavior
- [ ] Measure: Retry performance impact
- [ ] Compare: Different retry strategies
- [ ] Document: Retry strategy

### Dead Letter Queue Testing
- [ ] Generate: Messages that will fail processing
- [ ] Test: Messages retry up to max count
- [ ] Test: Messages move to DLQ
- [ ] Test: DLQ message inspection
- [ ] Test: DLQ message reprocessing (after fix)
- [ ] Implement: DLQ monitoring
- [ ] Document: DLQ testing results

---

## Consistency TODOs

### Consistency Analysis 1: Eventual Consistency
- [ ] Scenario: Payment event published, balance update async
- [ ] Analyze: Consistency model (eventual)
- [ ] Test: Read balance immediately after payment (might be stale)
- [ ] Test: Read balance after delay (eventually consistent)
- [ ] Measure: Consistency delay (time to consistency)
- [ ] Design: Handling stale reads
- [ ] Document: Eventual consistency analysis

### Consistency Analysis 2: Message Ordering & Consistency
- [ ] Scenario: Multiple events for same entity (balance updates)
- [ ] Test: Events processed out of order
- [ ] Analyze: Consistency impact
- [ ] Design: Ordering strategy để ensure consistency
- [ ] Test: Ordered processing
- [ ] Verify: Consistency maintained
- [ ] Document: Ordering và consistency

### Consistency Analysis 3: Idempotency & Consistency
- [ ] Scenario: Duplicate payment events
- [ ] Test: Process duplicate events
- [ ] Analyze: Consistency impact (double charge?)
- [ ] Implement: Idempotent processing
- [ ] Test: Duplicate events processed once
- [ ] Verify: Consistency maintained
- [ ] Document: Idempotency và consistency

### Consistency Analysis 4: Saga & Consistency
- [ ] Scenario: Saga với multiple steps
- [ ] Test: Saga success (all steps complete)
- [ ] Test: Saga failure (one step fails)
- [ ] Test: Compensating transactions
- [ ] Verify: Consistency after compensation
- [ ] Measure: Compensation time
- [ ] Document: Saga consistency

### Consistency Testing
- [ ] Design: Consistency test scenarios
- [ ] Test: Eventual consistency với various delays
- [ ] Test: Consistency under failures
- [ ] Test: Consistency với concurrent events
- [ ] Measure: Consistency metrics
- [ ] Document: Consistency test results

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Event Design exercises hoàn thành?
- [ ] Review: Tất cả MQ Integration tasks hoàn thành?
- [ ] Review: Tất cả Failure scenarios tested?
- [ ] Review: Tất cả Consistency analyses hoàn thành?
- [ ] Rate: Message queue understanding (1-10)
- [ ] Rate: Event-driven architecture skills (1-10)
- [ ] Rate: Failure handling skills (1-10)
- [ ] Rate: Consistency reasoning skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Architecture Review
- [ ] Review: Event-driven architecture design
- [ ] Check: Event schemas well-designed?
- [ ] Check: Message flow correct?
- [ ] Check: Failure handling adequate?
- [ ] Check: Consistency strategy appropriate?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Compare: Design vs industry best practices
- [ ] Document: Architecture review findings

### Code Review
- [ ] Review: MQ integration code
- [ ] Check: Error handling adequate?
- [ ] Check: Retry logic correct?
- [ ] Check: Idempotency implemented?
- [ ] Check: Acknowledgment handling correct?
- [ ] Review: Event sourcing implementation
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Reliability Review
- [ ] Review: All failure scenarios tested
- [ ] Verify: All failure scenarios handled?
- [ ] Identify: Remaining failure modes
- [ ] Design: Handling cho remaining failures
- [ ] Test: New failure handling
- [ ] Document: Reliability review

### Performance Review
- [ ] Review: Message throughput
- [ ] Review: Message latency
- [ ] Review: Consumer processing time
- [ ] Identify: Performance bottlenecks
- [ ] Optimize: Bottlenecks
- [ ] Re-test: Performance after optimization
- [ ] Document: Performance review

### Knowledge Check
- [ ] Explain: At-least-once vs Exactly-once delivery (viết comparison, không xem notes)
- [ ] Explain: Kafka partitions và consumer groups (viết explanation, không xem notes)
- [ ] Explain: Dead letter queue - when và why? (viết analysis, không xem notes)
- [ ] Explain: Event sourcing - pros và cons (viết analysis, không xem notes)
- [ ] Explain: Saga pattern - choreography vs orchestration (viết comparison, không xem notes)
- [ ] Solve: 1000 messages/second, consumer processes 100 messages/second. How many consumers needed?
- [ ] Solve: Message TTL = 60s, max retries = 3, retry delay = 10s. Max time in queue?
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 message reliability concepts còn confuse
- [ ] Write: 1 failure scenario bạn handled và solution
- [ ] Write: Confidence level cho Week 8 (1-10)
- [ ] Compare: Week 6 vs Week 7 progress
- [ ] Plan: Preparation cho Week 8 (Microservices)
- [ ] Set: Goals cho Week 8
- [ ] Document: Week 7 reflection (500 words)

### Mentor Questions (Answer these - reason about reliability)
- [ ] Q1: Producer sends message, gets ack, nhưng message lost trong broker. Possible? Làm sao prevent? (viết analysis với delivery guarantee reasoning)
- [ ] Q2: Consumer processes message, crashes before ack. Message reprocessed. Làm sao ensure idempotency? (viết idempotency strategy)
- [ ] Q3: 3 events: CreatePayment, UpdateBalance, SendNotification. Event 2 fails. Làm sao handle? Saga? Event ordering? (viết solution với trade-offs)
- [ ] Q4: Kafka topic có 10 partitions, 5 consumers trong group. Partition assignment? What if 1 consumer fails? (viết analysis với rebalancing)
- [ ] Q5: Message processed successfully, nhưng consumer crashes before ack. Message reprocessed. Side effects? Làm sao prevent? (viết analysis với idempotency)
- [ ] Q6: Exactly-once delivery - possible? How? Trade-offs? (viết analysis với implementation approach)
- [ ] Review: Answers có reason deeply về reliability? Có consider all failure scenarios? Có justify solutions?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Event Design TODOs: ✅ Complete với designs
- [ ] Tất cả MQ Integration TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Failure & Retry TODOs: ✅ Complete với test results
- [ ] Tất cả Consistency TODOs: ✅ Complete với analysis
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Event schemas saved: ✅ Yes
- [ ] Failure test results saved: ✅ Yes
- [ ] Ready for Week 8: ✅ Yes

---

> **Mentor Final Check**: Message queues require deep reasoning về reliability. Nếu bạn không understand delivery guarantees, idempotency, và failure scenarios, bạn will lose messages, duplicate processing, hoặc break consistency. Hãy honest: Bạn có thể reason về message reliability chưa? Bạn có thể handle all failure scenarios chưa? Bạn có thể ensure exactly-once semantics chưa? Nếu chưa, làm lại. Production systems need message reliability, not "hope it works".
