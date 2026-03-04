# Week 7 – Message Queues & Event-Driven Architecture

> **Mentor Note**: Message queues là backbone của distributed systems. Nhưng messages có thể lost, duplicated, hoặc out-of-order. Bạn phải reason về MỌI failure scenario. Không có "message sent, done". Phải có "message sent, acknowledged, processed, verified". Production systems need message reliability, not hope. Mỗi message delivery guarantee phải được understood, tested, và verified.

---

## Study TODOs

### Message Queue Fundamentals
- [x] Đọc "Designing Data-Intensive Applications" Chapter 11 - Stream Processing
    - **Key takeaways (DDIA Ch11 – Stream Processing):**
        - **Stream processing**: xử lý dữ liệu liên tục, real-time, khác với batch processing (xử lý theo lô).
        - **Event streams**: sequence of events, append-only, immutable; phù hợp cho event sourcing, audit logs.
        - **Message brokers**: trung gian giữa producers và consumers, đảm bảo delivery, ordering, persistence.
        - **Stream processing patterns**: windowing (time-based, count-based), joins (stream-stream, stream-table), aggregations.
        - **Fault tolerance**: cần checkpointing, exactly-once semantics, idempotency để handle failures.
        - **Insight**: stream processing cho phép real-time analytics, event-driven architecture, nhưng phức tạp hơn batch về consistency và error handling.
- [x] Định nghĩa chính xác: Message queue
    - **Message queue**: cấu trúc dữ liệu FIFO (First-In-First-Out) lưu trữ messages tạm thời, cho phép producers gửi messages và consumers nhận messages theo thứ tự. Messages được lưu trong queue cho đến khi được consumer xử lý và acknowledge.
- [x] Định nghĩa chính xác: Message broker
    - **Message broker**: middleware component quản lý message queues, đảm bảo routing, persistence, delivery guarantees giữa producers và consumers. Broker nhận messages từ producers, lưu trữ, và phân phối đến consumers theo các pattern (point-to-point, pub/sub).
- [x] Định nghĩa chính xác: Producer, Consumer, Broker
    - **Producer**: service/application gửi messages vào message queue/broker.
    - **Consumer**: service/application nhận và xử lý messages từ message queue/broker.
    - **Broker**: trung gian quản lý queues/topics, đảm bảo delivery, persistence, routing messages từ producers đến consumers.
- [x] Đọc về "point-to-point" messaging pattern
    - **Point-to-point**: mỗi message chỉ được deliver đến **một consumer duy nhất**. Nhiều consumers có thể subscribe cùng queue, nhưng mỗi message chỉ được một consumer xử lý. Phù hợp cho task distribution, load balancing.
- [x] Đọc về "publish-subscribe" (pub/sub) pattern
    - **Pub/Sub**: một message được publish đến **nhiều subscribers** (consumers). Mỗi subscriber nhận bản copy của message. Phù hợp cho event broadcasting, notifications, real-time updates.
- [x] So sánh: Point-to-point vs Pub/Sub (5 differences)
    1. **Message delivery**: Point-to-point → 1 consumer; Pub/Sub → nhiều consumers.
    2. **Use case**: Point-to-point → task distribution, work queues; Pub/Sub → event broadcasting, notifications.
    3. **Message ownership**: Point-to-point → message bị remove sau khi consumed; Pub/Sub → message tồn tại cho đến khi tất cả subscribers nhận.
    4. **Scalability**: Point-to-point → scale bằng cách thêm consumers (load balancing); Pub/Sub → scale bằng cách thêm subscribers (fan-out).
    5. **Coupling**: Point-to-point → producer biết có consumer; Pub/Sub → producer không biết có bao nhiêu subscribers (loose coupling).
- [x] Analyze: When to use point-to-point? When pub/sub? (5 use cases each)
    - **Point-to-point use cases:**
        1. **Task queues**: phân phối jobs cho workers (image processing, email sending).
        2. **Order processing**: mỗi order chỉ được xử lý bởi một service.
        3. **Payment processing**: mỗi payment transaction chỉ được process một lần.
        4. **Data synchronization**: sync data từ source đến một destination cụ thể.
        5. **Load balancing**: phân phối requests đồng đều giữa multiple workers.
    - **Pub/Sub use cases:**
        1. **Event notifications**: user registration event → email service, analytics service, audit service đều nhận.
        2. **Real-time updates**: price changes → multiple services cần update (inventory, pricing, recommendations).
        3. **Log aggregation**: application logs → multiple consumers (monitoring, analytics, archiving).
        4. **Cache invalidation**: product update → invalidate cache ở multiple services.
        5. **Microservices communication**: service A publishes event → services B, C, D subscribe và react.
- [x] Đọc về "message durability" - what is it?
    - **Message durability**: đảm bảo messages không bị mất khi broker restart/crash. Durable messages được lưu vào disk (persistent storage) thay vì chỉ trong memory. Cần cả durable queue và durable message để đảm bảo hoàn toàn.
- [x] Đọc về "message persistence" - how it works
    - **Message persistence**: lưu messages vào disk (non-volatile storage) thay vì chỉ RAM. Khi broker restart, messages vẫn còn. Thường dùng write-ahead log (WAL) hoặc append-only log để persist nhanh. Trade-off: latency cao hơn (disk I/O) nhưng đảm bảo reliability.

### Kafka Fundamentals
- [x] Đọc về "Apache Kafka" - what is it?
    - **Apache Kafka**: distributed streaming platform, hoạt động như distributed commit log. Cho phép publish/subscribe streams of records, lưu trữ messages theo thời gian (retention period), và xử lý streams real-time. Được thiết kế cho high throughput, horizontal scaling, và durability.
- [x] Đọc về "Kafka topics" - how they work
    - **Kafka topics**: categories/feeds mà messages được publish vào. Topic là logical grouping của messages. Mỗi topic có thể có nhiều partitions để scale. Producers gửi messages đến topic, consumers đọc từ topic.
- [x] Đọc về "Kafka partitions" - why partitions?
    - **Partitions**: chia topic thành nhiều segments để:
        1. **Scale throughput**: nhiều partitions = nhiều producers/consumers có thể làm việc song song.
        2. **Parallel processing**: mỗi partition có thể được consume bởi một consumer riêng.
        3. **Ordering guarantee**: messages trong cùng partition được đảm bảo thứ tự (FIFO trong partition).
        4. **Distribution**: partitions được distribute trên nhiều brokers để load balancing.
- [x] Đọc về "partition keys" - how they work
    - **Partition key**: giá trị producer chỉ định khi gửi message. Kafka hash key để quyết định message đi vào partition nào: `partition = hash(key) % num_partitions`. Messages cùng key → cùng partition → đảm bảo ordering. Nếu không có key → round-robin distribution.
- [x] Đọc về "consumer groups" - how they work
    - **Consumer groups**: nhóm consumers cùng làm việc để consume một topic. Mỗi partition chỉ được assign cho một consumer trong group (load balancing). Nếu số consumers > số partitions → một số consumers idle. Nếu consumer fails → partitions được reassign cho consumers còn lại (rebalancing).
- [x] Đọc về "offset" - what is it? Why important?
    - **Offset**: số thứ tự (sequence number) của message trong partition. Mỗi consumer group lưu offset hiện tại để biết đã đọc đến đâu. Quan trọng vì:
        1. **Resume processing**: consumer restart có thể tiếp tục từ offset cuối.
        2. **Replay messages**: có thể đọc lại từ offset cũ.
        3. **Exactly-once semantics**: offset commit atomic với processing để tránh duplicate.
- [x] Đọc về "Kafka brokers" - cluster architecture
    - **Kafka brokers**: servers trong Kafka cluster, mỗi broker lưu một số partitions. Cluster có thể có nhiều brokers (thường 3-5 cho production). Mỗi broker có unique ID. Brokers replicate partitions để đảm bảo fault tolerance.
- [x] Đọc về "replication factor" - how replication works
    - **Replication factor**: số bản copy của mỗi partition (thường 3). Một partition có 1 leader (handle reads/writes) và N-1 followers (replicate từ leader). Nếu leader fails → follower được promote thành leader. Đảm bảo durability và availability.
- [x] Đọc về "leader và followers" trong partitions
    - **Leader**: broker chịu trách nhiệm handle tất cả read/write requests cho partition. **Followers**: replicate data từ leader, không serve requests. Nếu leader fails → controller broker chọn follower mới làm leader (leader election). Followers fetch data từ leader và maintain replica.
- [x] Đọc về "Kafka retention" - how long messages kept?
    - **Kafka retention**: thời gian hoặc kích thước messages được giữ lại. Có 2 policies:
        1. **Time-based**: `retention.ms` (default 7 days) - xóa messages cũ hơn.
        2. **Size-based**: `retention.bytes` - xóa messages khi đạt giới hạn.
    - Messages cũ hơn retention bị xóa tự động. Cho phép replay events trong retention window.
- [x] Research: Kafka use cases và limitations
    - **Use cases:**
        1. **Event streaming**: real-time event processing (user actions, IoT data).
        2. **Log aggregation**: collect logs từ nhiều services.
        3. **Metrics collection**: time-series data, monitoring.
        4. **Event sourcing**: store events để rebuild state.
        5. **Message queue**: async communication giữa microservices.
    - **Limitations:**
        1. **Complexity**: setup và maintain phức tạp hơn RabbitMQ.
        2. **Ordering**: chỉ đảm bảo ordering trong partition, không cross-partition.
        3. **Latency**: không phù hợp cho real-time low-latency (< 10ms).
        4. **Overkill cho simple use cases**: nếu chỉ cần simple queue → RabbitMQ đơn giản hơn.
        5. **Resource intensive**: cần nhiều disk I/O, memory cho high throughput.

### RabbitMQ Fundamentals
- [x] Đọc về "RabbitMQ" - what is it?
    - **RabbitMQ**: message broker open-source, implement AMQP (Advanced Message Queuing Protocol). Hỗ trợ nhiều messaging patterns (point-to-point, pub/sub), routing linh hoạt qua exchanges, và đảm bảo message delivery. Phù hợp cho task queues, RPC, event distribution.
- [x] Đọc về "exchanges" - types (direct, topic, fanout, headers)
    - **Exchanges**: nhận messages từ producers và route đến queues dựa trên routing rules.
        - **Direct**: route messages đến queue có routing key khớp chính xác.
        - **Topic**: route dựa trên pattern matching (wildcards: *, #) của routing key.
        - **Fanout**: broadcast messages đến tất cả queues được bind (ignore routing key).
        - **Headers**: route dựa trên message headers thay vì routing key.
- [x] Đọc về "queues" - how they work
    - **Queues**: lưu trữ messages cho đến khi consumers xử lý. Queue có thể durable (survive broker restart) hoặc transient. Messages được deliver theo FIFO (trong cùng queue). Queue có thể có TTL, max length, dead letter exchange.
- [x] Đọc về "bindings" - routing messages
    - **Bindings**: kết nối exchange với queue, định nghĩa routing rules. Binding có routing key (hoặc pattern) để filter messages. Khi message đến exchange, exchange kiểm tra bindings và route đến queues phù hợp.
- [x] Đọc về "routing keys" - how routing works
    - **Routing key**: string producer gửi kèm message. Exchange dùng routing key để quyết định route message đến queue nào:
        - **Direct exchange**: routing key phải khớp chính xác với binding key.
        - **Topic exchange**: routing key match pattern (vd: "user.*.created" match "user.123.created").
        - **Fanout**: ignore routing key.
- [x] Đọc về "durable queues" - persistence
    - **Durable queues**: queues tồn tại sau khi broker restart. Cần declare queue với `durable=true`. Kết hợp với durable messages để đảm bảo messages không bị mất. Non-durable queues bị xóa khi broker restart.
- [x] Đọc về "message acknowledgment" - ack vs nack
    - **Acknowledgment (ack)**: consumer báo cho broker đã xử lý message thành công. Broker xóa message khỏi queue sau khi nhận ack.
    - **Negative acknowledgment (nack)**: consumer báo message xử lý thất bại. Broker có thể requeue message (retry) hoặc gửi đến dead letter queue.
    - **Auto-ack**: broker tự động ack khi deliver message (risk mất message nếu consumer crash).
- [x] Đọc về "prefetch count" - consumer prefetching
    - **Prefetch count**: số messages broker gửi đến consumer trước khi chờ ack. Giúp tăng throughput (consumer không phải chờ từng message). Nhưng nếu prefetch quá cao → một consumer giữ nhiều messages, các consumers khác thiếu work. Thường set 1-10 tùy processing time.
- [x] Research: RabbitMQ vs Kafka - when to use which?
    - **RabbitMQ khi:**
        1. Cần simple message queue, task distribution.
        2. Cần routing linh hoạt (exchanges, routing keys).
        3. Cần message TTL, priority queues.
        4. Workload vừa phải (< 1M messages/day).
        5. Cần setup đơn giản, dễ maintain.
    - **Kafka khi:**
        1. Cần high throughput (> 1M messages/second).
        2. Cần event streaming, log aggregation.
        3. Cần replay events, event sourcing.
        4. Cần long retention (days/weeks).
        5. Cần horizontal scaling lớn.
    - **Rule of thumb**: RabbitMQ cho task queues, Kafka cho event streaming.

### Message Ordering
- [x] Đọc về "message ordering" - why important?
    - **Message ordering**: đảm bảo messages được xử lý theo thứ tự gửi. Quan trọng vì:
        1. **Business logic**: payment events phải xử lý theo thứ tự (create → process → complete).
        2. **State consistency**: balance updates phải theo thứ tự để tránh sai số.
        3. **User experience**: notifications, updates phải hiển thị đúng thứ tự.
- [x] Đọc về "FIFO ordering" - first-in-first-out
    - **FIFO ordering**: messages được xử lý theo thứ tự gửi (First-In-First-Out). Đảm bảo message A gửi trước message B thì A được xử lý trước B. Khó đạt được trong distributed systems vì parallel processing, network delays, failures.
- [x] Đọc về "partition-based ordering" trong Kafka
    - **Partition-based ordering**: Kafka đảm bảo ordering **chỉ trong cùng partition**. Messages trong partition được xử lý theo thứ tự. Nhưng messages khác partitions có thể out-of-order. Để đảm bảo ordering cho một entity → dùng partition key để đưa tất cả messages của entity vào cùng partition.
- [x] Analyze: How to ensure ordering trong Kafka?
    1. **Single partition**: đảm bảo ordering hoàn toàn nhưng giảm throughput (không scale).
    2. **Partition key**: dùng key (vd: user_id) để đưa messages cùng entity vào cùng partition → ordering per entity.
    3. **Single consumer per partition**: mỗi partition chỉ có 1 consumer để tránh parallel processing gây out-of-order.
    4. **Idempotent producer**: tránh duplicate messages gây confusion về ordering.
- [x] Analyze: How to ensure ordering trong RabbitMQ?
    1. **Single queue, single consumer**: đảm bảo ordering nhưng không scale.
    2. **Consumer prefetch = 1**: chỉ process 1 message tại một thời điểm, đợi ack trước khi nhận message tiếp.
    3. **Message priority**: dùng priority queues nếu cần ordering theo priority.
    4. **Sequencing**: thêm sequence number vào message, consumer buffer và reorder nếu cần.
- [x] Đọc về "out-of-order messages" - causes và solutions
    - **Causes:**
        1. **Parallel processing**: nhiều consumers xử lý messages song song.
        2. **Network delays**: messages đến consumer với thứ tự khác.
        3. **Retries**: message retry có thể đến sau messages gửi sau.
        4. **Multiple partitions/queues**: messages phân tán trên nhiều partitions.
    - **Solutions:**
        1. **Partition key strategy**: đảm bảo related messages vào cùng partition.
        2. **Buffering và sequencing**: consumer buffer messages, reorder theo sequence number.
        3. **Single consumer per entity**: dùng consistent hashing để route messages cùng entity đến cùng consumer.
- [x] Đọc về "idempotent processing" - handle duplicates
    - **Idempotent processing**: xử lý cùng message nhiều lần cho cùng kết quả. Quan trọng vì:
        1. **Retries**: message có thể được deliver nhiều lần.
        2. **Exactly-once semantics**: cần idempotency để đạt exactly-once.
        3. **Consistency**: tránh duplicate side effects (double charge, duplicate records).
    - **Implementation**: dùng idempotency key (message ID), lưu processed IDs, check trước khi process.
- [x] Research: Ordering guarantees của different MQs
    - **Kafka**: ordering trong partition, không cross-partition. Cần partition key strategy.
    - **RabbitMQ**: ordering trong queue nếu single consumer, prefetch=1. Không đảm bảo cross-queue.
    - **Amazon SQS FIFO**: đảm bảo FIFO ordering trong queue, hỗ trợ message groups.
    - **Azure Service Bus**: ordering trong session (session ID), messages cùng session được xử lý tuần tự.
    - **Redis Streams**: ordering trong stream, consumer groups đảm bảo processing order.

### Message Delivery Guarantees
- [x] Đọc về "at-most-once" delivery - what is it?
    - **At-most-once**: message được deliver tối đa 1 lần, có thể bị mất (lost) nhưng không duplicate. Producer gửi message, không đợi ack, không retry. Consumer không ack. Phù hợp cho use cases chấp nhận mất message (metrics, logs không critical).
- [x] Analyze: Pros và cons của at-most-once (5 points each)
    - **Pros:**
        1. **Low latency**: không đợi ack, không retry → nhanh.
        2. **Simple**: không cần idempotency, retry logic.
        3. **Low overhead**: ít network traffic, ít storage.
        4. **High throughput**: không block trên failures.
        5. **Suitable cho non-critical data**: metrics, logs, analytics.
    - **Cons:**
        1. **Message loss**: messages có thể bị mất → không đảm bảo delivery.
        2. **No retry**: không tự động retry khi fail.
        3. **Not suitable cho critical operations**: payments, orders không thể mất.
        4. **No guarantee**: không đảm bảo message được process.
        5. **Data inconsistency risk**: mất message → state không đồng bộ.
- [x] Đọc về "at-least-once" delivery - what is it?
    - **At-least-once**: message được deliver ít nhất 1 lần, có thể duplicate nhưng không mất. Producer retry nếu không nhận ack. Consumer ack sau khi process. Nếu consumer crash trước ack → message được redeliver → duplicate. Phổ biến nhất trong production.
- [x] Analyze: Pros và cons của at-least-once (5 points each)
    - **Pros:**
        1. **No message loss**: đảm bảo message được deliver (có thể nhiều lần).
        2. **Reliable**: phù hợp cho critical operations.
        3. **Simple implementation**: chỉ cần retry + ack.
        4. **High availability**: messages không bị mất khi failures.
        5. **Industry standard**: hầu hết MQs hỗ trợ.
    - **Cons:**
        1. **Duplicates**: messages có thể được process nhiều lần.
        2. **Need idempotency**: consumer phải handle duplicates.
        3. **Potential side effects**: duplicate processing → double charge, duplicate records nếu không idempotent.
        4. **Overhead**: retries tăng network traffic, processing time.
        5. **Complexity**: cần idempotency logic ở consumer.
- [x] Đọc về "exactly-once" delivery - what is it?
    - **Exactly-once**: message được deliver đúng 1 lần, không mất, không duplicate. Khó đạt được, cần idempotency + transactional semantics. Kafka hỗ trợ với idempotent producer + transactional API. Phức tạp, có overhead.
- [x] Analyze: Pros và cons của exactly-once (5 points each)
    - **Pros:**
        1. **No duplicates, no loss**: đảm bảo perfect delivery.
        2. **Simpler consumer logic**: không cần handle duplicates (theo lý thuyết).
        3. **Data consistency**: đảm bảo state consistency.
        4. **Ideal cho financial systems**: payments, transactions cần chính xác.
        5. **No side effects**: tránh duplicate processing.
    - **Cons:**
        1. **Complex implementation**: cần idempotency + transactions.
        2. **Performance overhead**: transactional semantics chậm hơn.
        3. **Limited support**: không phải MQ nào cũng hỗ trợ tốt.
        4. **Trade-offs**: có thể không đạt được trong mọi scenario (network partitions).
        5. **Cost**: cần storage cho idempotency keys, coordination overhead.
- [x] Đọc về "idempotency" - how to achieve exactly-once semantics
    - **Idempotency**: xử lý cùng message nhiều lần cho cùng kết quả. Để đạt exactly-once:
        1. **Idempotency key**: mỗi message có unique ID (producer-generated hoặc business key).
        2. **Idempotency store**: lưu processed message IDs (Redis, DB) với TTL.
        3. **Check before process**: consumer check ID đã process chưa trước khi xử lý.
        4. **Atomic check-and-process**: check và mark processed trong transaction để tránh race condition.
        5. **TTL management**: xóa old IDs sau TTL để tránh storage bloat.
- [x] Compare: All delivery guarantees (10 criteria)
    1. **Message loss**: At-most-once (có thể mất), At-least-once (không mất), Exactly-once (không mất).
    2. **Duplicates**: At-most-once (không), At-least-once (có thể), Exactly-once (không).
    3. **Latency**: At-most-once (thấp nhất), At-least-once (trung bình), Exactly-once (cao nhất).
    4. **Throughput**: At-most-once (cao nhất), At-least-once (trung bình), Exactly-once (thấp nhất).
    5. **Complexity**: At-most-once (đơn giản), At-least-once (trung bình), Exactly-once (phức tạp).
    6. **Use cases**: At-most-once (metrics, logs), At-least-once (hầu hết cases), Exactly-once (financial).
    7. **Idempotency needed**: At-most-once (không), At-least-once (có), Exactly-once (có).
    8. **Retry logic**: At-most-once (không), At-least-once (có), Exactly-once (có + transactions).
    9. **Storage overhead**: At-most-once (thấp), At-least-once (trung bình), Exactly-once (cao).
    10. **Reliability**: At-most-once (thấp), At-least-once (cao), Exactly-once (cao nhất).
- [x] Research: Which MQs support which guarantees?
    - **Kafka**: At-least-once (default), Exactly-once (với idempotent producer + transactions).
    - **RabbitMQ**: At-least-once (với manual ack), At-most-once (với auto-ack).
    - **Amazon SQS**: At-least-once (standard), Exactly-once (FIFO queues).
    - **Azure Service Bus**: At-least-once, Exactly-once (với sessions).
    - **Redis Streams**: At-least-once (default), có thể đạt exactly-once với idempotency.
    - **Google Pub/Sub**: At-least-once, không hỗ trợ exactly-once native.

### Message Acknowledgment
- [x] Đọc về "message acknowledgment" - why needed?
    - **Acknowledgment**: consumer báo cho broker đã xử lý message thành công. Cần thiết để:
        1. **Reliability**: broker biết message đã được process, có thể xóa khỏi queue.
        2. **At-least-once guarantee**: nếu không ack → broker redeliver message.
        3. **Failure handling**: nếu consumer crash → broker biết message chưa process, redeliver.
        4. **Flow control**: broker không gửi quá nhiều messages nếu consumer chậm.
- [x] Đọc về "auto-ack" - automatic acknowledgment
    - **Auto-ack**: broker tự động ack message ngay khi deliver đến consumer, không đợi consumer xử lý xong. Nếu consumer crash sau khi nhận message nhưng trước khi process → message bị mất (at-most-once semantics).
- [x] Analyze: Pros và cons của auto-ack (5 points each)
    - **Pros:**
        1. **Simple**: không cần code ack logic.
        2. **Low latency**: không đợi processing xong.
        3. **High throughput**: messages được xóa nhanh khỏi queue.
        4. **Suitable cho fire-and-forget**: metrics, logs không critical.
        5. **Less code**: giảm complexity ở consumer.
    - **Cons:**
        1. **Message loss**: nếu consumer crash → message bị mất.
        2. **No retry**: không tự động retry failed messages.
        3. **Not reliable**: không phù hợp cho critical operations.
        4. **At-most-once**: chỉ đảm bảo at-most-once delivery.
        5. **No error handling**: không biết message có được process thành công không.
- [x] Đọc về "manual ack" - explicit acknowledgment
    - **Manual ack**: consumer phải explicitly gọi `ack()` sau khi xử lý message thành công. Broker chỉ xóa message sau khi nhận ack. Nếu consumer crash trước ack → broker redeliver message (at-least-once semantics).
- [x] Analyze: Pros và cons của manual ack (5 points each)
    - **Pros:**
        1. **Reliability**: đảm bảo message được process trước khi ack.
        2. **At-least-once**: đảm bảo không mất message.
        3. **Error handling**: có thể nack nếu process fail.
        4. **Retry support**: có thể requeue failed messages.
        5. **Production-ready**: standard cho critical systems.
    - **Cons:**
        1. **Complexity**: cần code ack/nack logic.
        2. **Potential duplicates**: nếu crash sau process nhưng trước ack → duplicate.
        3. **Need idempotency**: phải handle duplicates.
        4. **Slower**: phải đợi processing xong mới ack.
        5. **Memory usage**: messages giữ trong queue lâu hơn nếu processing chậm.
- [x] Đọc về "ack timeout" - what happens if timeout?
    - **Ack timeout**: thời gian broker đợi ack từ consumer. Nếu timeout:
        1. Broker coi như consumer failed.
        2. Message được redeliver (có thể đến consumer khác).
        3. Consumer có thể nhận duplicate nếu đã process nhưng chưa kịp ack.
        4. Cần set timeout phù hợp với processing time (không quá ngắn → false redelivery, không quá dài → chậm recovery).
- [x] Đọc về "negative acknowledgment" (nack) - when to use?
    - **Negative acknowledgment (nack)**: consumer báo message xử lý thất bại. Broker có thể:
        1. **Requeue**: gửi lại message vào queue (retry).
        2. **Dead letter**: gửi đến dead letter queue nếu đã retry quá nhiều.
        3. **Discard**: xóa message nếu không quan trọng.
    - **When to use**: khi processing fail (validation error, business logic error, temporary failure).
- [x] Đọc về "requeue" - retry failed messages
    - **Requeue**: đưa message trở lại queue để retry. Có thể:
        1. **Immediate requeue**: gửi lại ngay (risk infinite loop nếu message luôn fail).
        2. **Delayed requeue**: đợi một khoảng thời gian trước khi retry (exponential backoff).
        3. **Max retries**: giới hạn số lần retry, sau đó gửi đến DLQ.
- [x] Research: Acknowledgment best practices
    1. **Always use manual ack** cho critical operations.
    2. **Ack after successful processing**, không ack trước khi process xong.
    3. **Set appropriate timeout** dựa trên processing time.
    4. **Handle nack properly**: requeue với delay, có max retry limit.
    5. **Idempotent processing**: handle duplicates nếu dùng manual ack.
    6. **Monitor ack rate**: track số messages ack/nack để detect issues.
    7. **Batch ack**: nếu có thể, ack nhiều messages cùng lúc để tăng throughput.

### Dead Letter Queues
- [x] Đọc về "dead letter queue" (DLQ) - what is it?
    - **Dead Letter Queue (DLQ)**: queue đặc biệt lưu messages không thể xử lý sau nhiều lần retry. Giúp:
        1. **Isolate poison messages**: messages luôn fail không làm ảnh hưởng normal processing.
        2. **Debugging**: inspect messages failed để tìm nguyên nhân.
        3. **Manual intervention**: có thể reprocess sau khi fix bug.
        4. **Monitoring**: alert khi có messages trong DLQ.
- [x] Đọc về "DLQ use cases" - when messages go to DLQ?
    1. **Max retry exceeded**: message đã retry quá số lần cho phép.
    2. **TTL expired**: message quá cũ, không còn valid.
    3. **Rejection**: consumer nack và không requeue.
    4. **Queue full**: queue đạt max length.
    5. **Invalid format**: message không parse được, validation fail.
- [x] Đọc về "message TTL" - time-to-live expiration
    - **Message TTL**: thời gian message tồn tại trong queue. Sau TTL:
        1. Message bị xóa hoặc chuyển đến DLQ.
        2. Tránh messages cũ tích tụ trong queue.
        3. Đảm bảo messages được process trong thời gian hợp lý.
        4. Có thể set TTL per message hoặc per queue.
- [x] Đọc về "max retry count" - retry limit
    - **Max retry count**: số lần tối đa message được retry trước khi chuyển đến DLQ. Quan trọng để:
        1. **Prevent infinite loops**: tránh retry mãi mãi với poison messages.
        2. **Resource management**: không waste resources cho messages không thể xử lý.
        3. **Fast failure**: fail nhanh để có thể investigate và fix.
- [x] Đọc về "DLQ processing" - how to handle DLQ messages?
    1. **Manual inspection**: xem messages trong DLQ để tìm nguyên nhân.
    2. **Fix và reprocess**: sau khi fix bug, có thể republish messages từ DLQ.
    3. **Alert và monitoring**: notify team khi có messages trong DLQ.
    4. **Analysis**: phân tích patterns của failed messages để improve system.
    5. **Archive**: lưu messages quan trọng để audit, compliance.
- [x] Đọc về "DLQ monitoring" - alert on DLQ messages
    - **DLQ monitoring**: track số messages trong DLQ, alert khi:
        1. **DLQ size > threshold**: có nhiều messages failed.
        2. **New messages**: có message mới vào DLQ.
        3. **Rate increase**: số messages vào DLQ tăng đột biến.
        4. **Pattern analysis**: detect common failure patterns.
- [x] Research: DLQ patterns và best practices
    1. **Always configure DLQ** cho production systems.
    2. **Set appropriate max retries**: không quá ít (false positives) hoặc quá nhiều (waste resources).
    3. **Monitor DLQ**: alert khi có messages, track trends.
    4. **DLQ processing workflow**: có process để investigate và fix.
    5. **DLQ retention**: giữ messages trong DLQ đủ lâu để debug nhưng không quá lâu.
    6. **Separate DLQ per queue**: mỗi queue có DLQ riêng để dễ debug.
    7. **DLQ metadata**: lưu thêm metadata (retry count, failure reason, timestamp) để debug.
- [x] Analyze: When to use DLQ? When to retry?
    - **Use DLQ khi:**
        1. Message đã retry quá max count.
        2. Message có format invalid, không thể fix.
        3. Business logic error không thể recover.
        4. Message quá cũ (TTL expired).
    - **Retry khi:**
        1. Temporary failures (network, timeout, service temporarily down).
        2. Transient errors có thể tự recover.
        3. Chưa đạt max retry count.
        4. Có khả năng success sau retry (exponential backoff).

### Event-Driven Architecture
- [x] Đọc về "event-driven architecture" (EDA) - what is it?
    - **Event-Driven Architecture (EDA)**: kiến trúc mà services communicate qua events thay vì direct calls. Services publish events khi có thay đổi, services khác subscribe và react. Đặc điểm:
        1. **Loose coupling**: services không biết về nhau.
        2. **Async communication**: không block, high throughput.
        3. **Scalability**: dễ scale từng service độc lập.
        4. **Resilience**: failures không lan truyền.
- [x] Đọc về "events" vs "commands" - differences
    - **Events**: facts đã xảy ra (past tense): `PaymentCompleted`, `UserRegistered`. Immutable, broadcast đến nhiều subscribers. Không có return value.
    - **Commands**: requests để thực hiện action (imperative): `CreatePayment`, `UpdateUser`. Gửi đến một handler cụ thể, có thể có return value, có thể fail.
    - **Key difference**: Events = "đã xảy ra", Commands = "hãy làm điều này".
- [x] Đọc về "event sourcing" - storing events
    - **Event sourcing**: lưu trữ state dưới dạng sequence of events thay vì current state. State được rebuild bằng cách replay events. Cho phép:
        1. **Complete audit trail**: biết mọi thay đổi.
        2. **Time travel**: xem state tại bất kỳ thời điểm nào.
        3. **Event replay**: rebuild state từ events.
        4. **Debugging**: trace mọi thay đổi.
- [x] Đọc về "CQRS" (Command Query Responsibility Segregation)
    - **CQRS**: tách biệt Command (write) và Query (read) models. Write model optimize cho consistency, write performance. Read model optimize cho query performance, có thể denormalized. Cho phép:
        1. **Independent scaling**: scale read/write riêng.
        2. **Optimized models**: mỗi model tối ưu cho use case riêng.
        3. **Eventual consistency**: read model sync từ write model qua events.
- [x] Đọc về "eventual consistency" - in EDA context
    - **Eventual consistency**: trong EDA, khi event được publish, subscribers xử lý async → có delay giữa event và state update. Không đảm bảo immediate consistency nhưng đảm bảo sẽ consistent sau một thời gian. Trade-off: high availability, scalability vs immediate consistency.
- [x] Đọc về "event replay" - rebuilding state from events
    - **Event replay**: process lại events từ đầu để rebuild current state. Hữu ích khi:
        1. **Recovery**: rebuild state sau failure.
        2. **New subscribers**: new service cần build state từ events.
        3. **Debugging**: replay để tìm bug.
        4. **Migration**: migrate sang new system.
- [x] Đọc về "event versioning" - how to version events?
    - **Event versioning**: quản lý schema changes của events theo thời gian. Strategies:
        1. **Version field**: thêm `version` field vào event.
        2. **Backward compatibility**: new fields optional, không remove old fields.
        3. **Upcasting**: convert old events sang new format khi replay.
        4. **Separate topics**: version mới dùng topic mới (vd: `payment.v1`, `payment.v2`).
- [x] Research: EDA patterns và anti-patterns
    - **Patterns:**
        1. **Event sourcing**: store events, rebuild state.
        2. **CQRS**: separate read/write models.
        3. **Saga**: distributed transactions qua events.
        4. **Event streaming**: continuous event processing.
    - **Anti-patterns:**
        1. **Synchronous events**: đợi response từ event (nên dùng command).
        2. **Tight coupling**: events có quá nhiều dependencies.
        3. **Event storms**: quá nhiều events, khó trace.
        4. **No idempotency**: không handle duplicate events.
        5. **No versioning**: không version events → breaking changes.

### Event Sourcing
- [x] Đọc về "event sourcing" - detailed explanation
    - **Event sourcing**: pattern lưu trữ state dưới dạng immutable sequence of events. Thay vì update current state, append events. State được tính bằng cách apply tất cả events từ đầu. Mỗi event là fact đã xảy ra, không thể thay đổi.
- [x] Đọc về "event store" - where events stored?
    - **Event store**: database/stream lưu events. Requirements:
        1. **Append-only**: chỉ thêm events, không update/delete.
        2. **Ordered**: events có thứ tự (sequence number, timestamp).
        3. **Durable**: persist events, survive failures.
        4. **Queryable**: có thể query events theo aggregate ID, time range.
    - **Options**: Kafka topics, specialized event stores (EventStore), database tables.
- [x] Đọc về "event stream" - sequence of events
    - **Event stream**: sequence of events cho một aggregate/entity. Events được append theo thứ tự, có sequence number. Stream có thể infinite (append liên tục) hoặc bounded (có end). Mỗi aggregate có stream riêng.
- [x] Đọc về "snapshots" - performance optimization
    - **Snapshots**: lưu state tại một thời điểm để tránh replay tất cả events. Khi rebuild state:
        1. Load snapshot gần nhất.
        2. Replay events sau snapshot.
        3. Giảm thời gian rebuild từ O(n) xuống O(n - snapshot_position).
    - **When to snapshot**: sau N events, hoặc sau T time, hoặc khi state đạt size threshold.
- [x] Đọc về "event replay" - rebuilding aggregates
    - **Event replay**: process events từ đầu (hoặc từ snapshot) để rebuild current state của aggregate. Algorithm:
        1. Load snapshot (nếu có).
        2. Load events sau snapshot.
        3. Apply events theo thứ tự.
        4. Return current state.
- [x] Analyze: Pros và cons của event sourcing (5 points each)
    - **Pros:**
        1. **Complete audit trail**: biết mọi thay đổi, compliance.
        2. **Time travel**: xem state tại bất kỳ thời điểm nào.
        3. **Debugging**: trace mọi thay đổi để debug.
        4. **Flexibility**: có thể rebuild read models khác nhau từ events.
        5. **Event replay**: rebuild state sau failures, migrate systems.
    - **Cons:**
        1. **Complexity**: phức tạp hơn CRUD, khó implement đúng.
        2. **Storage**: lưu nhiều events, storage tăng theo thời gian.
        3. **Performance**: replay events có thể chậm nếu không có snapshot.
        4. **Event versioning**: cần handle schema evolution.
        5. **Learning curve**: team cần học pattern mới.
- [x] Research: Event sourcing use cases
    1. **Financial systems**: audit trail, compliance, transaction history.
    2. **Gaming**: player state, game events, replay games.
    3. **Collaboration tools**: document history, version control.
    4. **IoT**: sensor data streams, device state.
    5. **E-commerce**: order history, inventory changes.
- [x] Đọc về "event versioning" - schema evolution
    - **Event versioning**: quản lý thay đổi schema của events. Strategies:
        1. **Version field**: `event_version` trong event.
        2. **Backward compatibility**: new fields optional, không remove fields.
        3. **Upcasting**: convert old events sang new format khi load.
        4. **Separate streams**: version mới dùng stream mới.
        5. **Schema registry**: quản lý schemas centrally (Confluent Schema Registry).

### Saga Pattern
- [x] Đọc về "Saga pattern" - distributed transactions
    - **Saga pattern**: pattern quản lý distributed transactions bằng cách chia thành sequence of local transactions. Mỗi local transaction có compensating transaction để rollback. Không dùng 2PC (Two-Phase Commit) vì blocking, không scale. Phù hợp cho microservices, long-running transactions.
- [x] Đọc về "choreography" saga - decentralized
    - **Choreography saga**: mỗi service tự quyết định next step dựa trên events nhận được. Không có central coordinator. Services publish events, services khác subscribe và react. Loose coupling, nhưng khó trace và debug.
- [x] Đọc về "orchestration" saga - centralized
    - **Orchestration saga**: có central orchestrator (saga coordinator) quyết định sequence of steps. Orchestrator gọi services theo thứ tự, handle failures, trigger compensating transactions. Dễ trace, debug, nhưng tight coupling với orchestrator.
- [x] Compare: Choreography vs Orchestration (10 criteria)
    1. **Coupling**: Choreography (loose), Orchestration (tight với orchestrator).
    2. **Complexity**: Choreography (phức tạp hơn, khó trace), Orchestration (đơn giản hơn, rõ ràng).
    3. **Scalability**: Choreography (scale tốt, không bottleneck), Orchestration (orchestrator có thể bottleneck).
    4. **Debugging**: Choreography (khó trace flow), Orchestration (dễ trace, có central log).
    5. **Failure handling**: Choreography (phức tạp, mỗi service tự handle), Orchestration (centralized, dễ handle).
    6. **Testability**: Choreography (khó test end-to-end), Orchestration (dễ test, có orchestrator).
    7. **Flexibility**: Choreography (linh hoạt, dễ thêm services), Orchestration (khó thay đổi flow).
    8. **Single point of failure**: Choreography (không có), Orchestration (orchestrator là SPOF).
    9. **Eventual consistency**: Choreography (eventual), Orchestration (có thể có intermediate consistency).
    10. **Use cases**: Choreography (simple flows, event-driven), Orchestration (complex flows, cần control).
- [x] Đọc về "compensating transactions" - rollback
    - **Compensating transactions**: actions để undo effects của completed transactions. Không phải rollback DB transaction (đã commit), mà là business logic để reverse effects. Ví dụ: charge payment → compensating = refund. Phải idempotent, có thể retry.
- [x] Đọc về "saga failure handling" - what if step fails?
    - **Saga failure handling**: khi một step fails:
        1. **Stop forward progress**: không tiếp tục các steps sau.
        2. **Execute compensating transactions**: rollback các steps đã complete (theo thứ tự ngược).
        3. **Handle partial failures**: một số steps đã complete, một số chưa.
        4. **Idempotent compensations**: compensating transactions phải idempotent.
        5. **Retry logic**: có thể retry failed step hoặc compensating transaction.
- [x] Research: Saga pattern implementations
    1. **Temporal**: workflow engine hỗ trợ saga.
    2. **Zeebe**: workflow engine cho microservices.
    3. **AWS Step Functions**: orchestration service.
    4. **Custom implementation**: dùng message queues + state machine.
    5. **Spring Cloud Saga**: framework cho Java.

### Message Retry Strategies
- [x] Đọc về "retry strategies" - why needed?
    - **Retry strategies**: cách retry failed messages. Cần thiết vì:
        1. **Transient failures**: network hiccups, temporary service unavailability.
        2. **Resilience**: không fail ngay lập tức, cho system cơ hội recover.
        3. **Reliability**: đảm bảo messages được process cuối cùng.
        4. **Load management**: tránh thundering herd khi service recover.
- [x] Đọc về "exponential backoff" - retry with delay
    - **Exponential backoff**: delay tăng exponentially sau mỗi retry: `delay = base_delay * (2 ^ retry_count)`. Ví dụ: 1s, 2s, 4s, 8s, 16s. Giúp:
        1. **Avoid overwhelming**: không spam service đang recover.
        2. **Self-healing**: cho service thời gian recover.
        3. **Reduce load**: giảm load khi service đang down.
- [x] Đọc về "fixed delay" - constant retry interval
    - **Fixed delay**: retry với delay cố định (vd: 5s giữa mỗi retry). Đơn giản, predictable, nhưng không adapt với service state. Phù hợp cho:
        1. **Predictable failures**: biết chắc service sẽ recover sau T time.
        2. **Simple systems**: không cần sophisticated backoff.
- [x] Đọc về "max retry attempts" - retry limit
    - **Max retry attempts**: số lần tối đa retry trước khi give up hoặc gửi đến DLQ. Quan trọng để:
        1. **Prevent infinite loops**: tránh retry mãi mãi với poison messages.
        2. **Resource management**: không waste resources.
        3. **Fast failure**: fail nhanh để có thể investigate.
- [x] Đọc về "retry jitter" - randomize retry time
    - **Retry jitter**: thêm randomness vào retry delay để tránh thundering herd. Thay vì tất cả retry cùng lúc, spread ra: `delay = base_delay + random(0, jitter)`. Giúp:
        1. **Avoid synchronized retries**: nhiều consumers không retry cùng lúc.
        2. **Load distribution**: spread load đều hơn.
        3. **Reduce contention**: giảm contention khi service recover.
- [x] Compare: Retry strategies (5 criteria)
    1. **Complexity**: Fixed delay (đơn giản), Exponential backoff (trung bình), Exponential + jitter (phức tạp hơn).
    2. **Efficiency**: Fixed delay (có thể waste time), Exponential (hiệu quả hơn), Exponential + jitter (tốt nhất).
    3. **Load management**: Fixed delay (không tốt), Exponential (tốt), Exponential + jitter (tốt nhất).
    4. **Use cases**: Fixed delay (simple, predictable), Exponential (general purpose), Exponential + jitter (high load).
    5. **Implementation**: Fixed delay (dễ), Exponential (trung bình), Exponential + jitter (cần random).
- [x] Research: Retry best practices
    1. **Use exponential backoff** cho general cases.
    2. **Add jitter** để tránh thundering herd.
    3. **Set max retries** để tránh infinite loops.
    4. **Monitor retry rate** để detect issues.
    5. **Different strategies** cho different error types (network vs business logic).
    6. **Circuit breaker**: stop retrying nếu service down quá lâu.
    7. **Dead letter queue**: gửi đến DLQ sau max retries.

### Message Idempotency
- [x] Đọc về "idempotent consumers" - handle duplicates
    - **Idempotent consumers**: consumers xử lý cùng message nhiều lần cho cùng kết quả. Quan trọng vì:
        1. **At-least-once delivery**: messages có thể được deliver nhiều lần.
        2. **Retries**: retry có thể gửi duplicate.
        3. **Exactly-once semantics**: cần idempotency để đạt exactly-once.
        4. **Consistency**: tránh duplicate side effects.
- [x] Đọc về "idempotency keys" - unique message IDs
    - **Idempotency keys**: unique identifier cho mỗi message (hoặc operation). Có thể:
        1. **Message ID**: UUID từ producer.
        2. **Business key**: combination của business fields (user_id + order_id + timestamp).
        3. **Request ID**: ID từ client request.
    - Consumer check key đã process chưa trước khi xử lý.
- [x] Đọc về "idempotency storage" - where to store?
    - **Idempotency storage**: nơi lưu processed message IDs. Options:
        1. **Redis**: fast, có TTL, phù hợp cho high throughput.
        2. **Database**: persistent, có thể query, phù hợp cho critical operations.
        3. **In-memory**: nhanh nhưng mất khi restart (không recommended cho production).
        4. **Distributed cache**: Hazelcast, Memcached cho multi-instance.
- [x] Đọc về "idempotency TTL" - how long to remember?
    - **Idempotency TTL**: thời gian lưu processed IDs. Cần đủ lâu để:
        1. **Cover retry window**: lâu hơn max retry time.
        2. **Handle network delays**: cover message delivery delays.
        3. **Avoid storage bloat**: không quá lâu để tránh storage tăng.
    - **Typical TTL**: 24-48 hours cho most cases, có thể lâu hơn cho critical operations.
- [x] Đọc về "idempotency scope" - per message? per user?
    - **Idempotency scope**: phạm vi idempotency check:
        1. **Per message**: mỗi message có ID riêng, check global.
        2. **Per user + operation**: cùng user, cùng operation chỉ process một lần (vd: user_id + payment_id).
        3. **Per entity**: cùng entity chỉ update một lần (vd: order_id).
    - **Choice depends on**: business requirements, duplicate scenarios.
- [x] Research: Idempotency implementation patterns
    1. **Check-then-process**: check ID trước, process, mark processed (cần atomic operation).
    2. **Idempotency middleware**: middleware tự động check và skip duplicates.
    3. **Database unique constraint**: dùng DB constraint để enforce idempotency.
    4. **Idempotency service**: dedicated service quản lý idempotency keys.
    5. **Optimistic locking**: dùng version/timestamp để detect duplicates.
    6. **Idempotency key in business logic**: design operations tự nhiên idempotent (upsert, set).

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
- [x] Q1: Producer sends message, gets ack, nhưng message lost trong broker. Possible? Làm sao prevent? (viết analysis với delivery guarantee reasoning)
    - **Possible?** Có thể xảy ra trong một số scenarios:
        1. **Broker crash sau ack nhưng trước persistence**: broker ack producer nhưng chưa kịp write vào disk, crash → message mất.
        2. **Disk failure**: message đã write nhưng disk corrupt → message mất.
        3. **Replication lag**: leader ack nhưng chưa replicate đến followers, leader fails → message mất nếu không có replica.
    - **Prevention strategies:**
        1. **Producer acks = "all"**: producer chỉ coi là success khi tất cả in-sync replicas đã ack (Kafka).
        2. **Durable writes**: broker phải fsync messages vào disk trước khi ack (trade-off: latency cao hơn).
        3. **Replication factor >= 3**: đảm bảo có ít nhất 2 replicas, giảm risk mất message khi 1 broker fails.
        4. **Idempotent producer**: producer retry với same message ID, broker deduplicate.
        5. **Transactional producer**: dùng transactions để đảm bảo atomicity.
    - **Delivery guarantee reasoning**: Với `acks=all` + replication + durable writes, đạt được **at-least-once** với probability mất message rất thấp (nhưng không phải zero). Để đạt exactly-once cần thêm idempotency ở consumer.
- [x] Q2: Consumer processes message, crashes before ack. Message reprocessed. Làm sao ensure idempotency? (viết idempotency strategy)
    - **Idempotency strategy:**
        1. **Idempotency key**: mỗi message có unique ID (producer-generated UUID hoặc business key như `payment_id`).
        2. **Idempotency store**: lưu processed message IDs trong Redis/DB với TTL (24-48h).
        3. **Check-before-process**: trước khi process, check `if (idempotency_store.contains(message_id)) return;`
        4. **Atomic check-and-mark**: dùng atomic operation (Redis SETNX, DB unique constraint) để check và mark processed trong một operation, tránh race condition.
        5. **Idempotent business logic**: design operations tự nhiên idempotent:
           - **Upsert**: `UPDATE ... ON DUPLICATE KEY UPDATE` thay vì INSERT.
           - **Set operations**: `SET balance = 100` thay vì `balance += 10` (nếu có thể).
           - **Conditional updates**: chỉ update nếu điều kiện đúng (vd: `UPDATE order SET status='PAID' WHERE status='PENDING'`).
        6. **Idempotency scope**: quyết định scope (per message, per user+operation) dựa trên business requirements.
    - **Implementation example:**
        ```java
        String messageId = message.getId();
        if (idempotencyStore.exists(messageId)) {
            log.info("Duplicate message, skipping: {}", messageId);
            return; // Already processed
        }
        // Atomic: check and mark
        if (idempotencyStore.tryMarkProcessed(messageId, TTL)) {
            try {
                processMessage(message);
                ack(message);
            } catch (Exception e) {
                idempotencyStore.remove(messageId); // Allow retry
                throw e;
            }
        } else {
            // Another instance already processing
            return;
        }
        ```
- [x] Q3: 3 events: CreatePayment, UpdateBalance, SendNotification. Event 2 fails. Làm sao handle? Saga? Event ordering? (viết solution với trade-offs)
    - **Problem**: Cần đảm bảo atomicity của 3 steps, nhưng là distributed system → không dùng 2PC.
    - **Solution 1: Saga Pattern (Orchestration)**
        - **Approach**: Orchestrator gọi services theo thứ tự:
            1. CreatePayment → success
            2. UpdateBalance → **fails**
            3. Orchestrator trigger compensating: CancelPayment
        - **Pros**: Centralized control, dễ trace, dễ handle failures.
        - **Cons**: Orchestrator là SPOF, tight coupling.
    - **Solution 2: Saga Pattern (Choreography)**
        - **Approach**: Services publish events, react:
            1. PaymentService publishes `PaymentCreated`
            2. BalanceService subscribes, updates balance, publishes `BalanceUpdated` hoặc `BalanceUpdateFailed`
            3. Nếu `BalanceUpdateFailed` → PaymentService publishes `PaymentCancelled`
        - **Pros**: Loose coupling, scalable.
        - **Cons**: Khó trace, phức tạp handle failures.
    - **Solution 3: Event Ordering + Compensating Events**
        - **Approach**: Đảm bảo events cùng payment_id vào cùng partition (Kafka), process tuần tự:
            1. Process `CreatePayment` → success
            2. Process `UpdateBalance` → fails → publish `BalanceUpdateFailed`
            3. Consumer của `BalanceUpdateFailed` publish `CancelPayment`
        - **Pros**: Event-driven, scalable.
        - **Cons**: Cần đảm bảo ordering, eventual consistency.
    - **Recommendation**: Dùng **Saga Orchestration** cho payment flows vì:
        1. Cần control chặt chẽ, dễ debug.
        2. Failure handling rõ ràng.
        3. Có thể monitor và alert dễ dàng.
    - **Trade-offs**: Chấp nhận orchestrator SPOF và latency cao hơn để đổi lấy reliability và traceability.
- [x] Q4: Kafka topic có 10 partitions, 5 consumers trong group. Partition assignment? What if 1 consumer fails? (viết analysis với rebalancing)
    - **Initial partition assignment:**
        - **5 consumers, 10 partitions**: mỗi consumer được assign 2 partitions.
        - **Assignment strategy** (default: Range hoặc RoundRobin):
            - **Range**: Consumer 1 → partitions 0-1, Consumer 2 → partitions 2-3, ...
            - **RoundRobin**: Consumer 1 → partitions 0,5, Consumer 2 → partitions 1,6, ...
    - **If 1 consumer fails:**
        1. **Detection**: Coordinator broker phát hiện consumer không heartbeat (session timeout, thường 10s).
        2. **Rebalancing**: Coordinator trigger rebalancing, tất cả consumers trong group tham gia.
        3. **Partition reassignment**: Partitions của failed consumer được redistribute cho 4 consumers còn lại:
           - Mỗi consumer nhận thêm ~0.5 partition (không thể chia đều) → một số consumers có 3 partitions, một số có 2.
        4. **Processing pause**: Trong rebalancing, consumers pause processing, commit offsets.
        5. **Resume**: Sau rebalancing, consumers resume với partitions mới.
    - **Rebalancing time**: Thường 1-5 giây, phụ thuộc vào:
        - Số partitions
        - Số consumers
        - Network latency
        - Offset commit time
    - **Impact:**
        - **Throughput**: Giảm tạm thời trong rebalancing.
        - **Message processing**: Messages trong partitions của failed consumer bị delay cho đến khi reassigned.
        - **Duplicate processing**: Nếu consumer fail sau khi process nhưng trước commit offset → messages có thể được reprocess.
    - **Best practices:**
        1. **Set appropriate session timeout**: không quá ngắn (false failures) hoặc quá dài (chậm recovery).
        2. **Monitor rebalancing**: alert khi rebalancing xảy ra thường xuyên.
        3. **Graceful shutdown**: consumer commit offsets trước khi shutdown để tránh rebalancing.
- [x] Q5: Message processed successfully, nhưng consumer crashes before ack. Message reprocessed. Side effects? Làm sao prevent? (viết analysis với idempotency)
    - **Side effects:**
        1. **Duplicate processing**: Message được process 2 lần → duplicate side effects:
           - **Double charge**: charge payment 2 lần.
           - **Duplicate records**: insert duplicate vào database.
           - **Duplicate notifications**: gửi email/SMS 2 lần.
           - **Incorrect state**: state bị update sai (vd: balance bị trừ 2 lần).
        2. **Data inconsistency**: State không đồng bộ giữa services.
        3. **User experience**: User thấy duplicate actions, confused.
    - **Prevention: Idempotency**
        1. **Idempotency key**: Mỗi message có unique ID (message ID hoặc business key).
        2. **Idempotency check**: Trước khi process, check `if (already_processed(message_id)) return;`
        3. **Idempotency store**: Lưu processed IDs trong Redis/DB với TTL.
        4. **Atomic operation**: Dùng atomic check-and-mark để tránh race condition:
           ```java
           if (redis.setIfNotExists("processed:" + messageId, "1", TTL)) {
               processMessage(message);
               ack(message);
           } else {
               // Already processed, skip
               ack(message); // Still ack to remove from queue
           }
           ```
        5. **Idempotent business logic**: Design operations idempotent:
           - **Upsert**: `INSERT ... ON DUPLICATE KEY UPDATE` thay vì INSERT.
           - **Conditional updates**: `UPDATE order SET status='PAID' WHERE status='PENDING' AND id=?`
           - **Idempotent APIs**: External APIs phải idempotent (dùng idempotency key).
    - **Implementation pattern:**
        ```java
        String idempotencyKey = message.getId();
        // Atomic check and mark
        if (idempotencyStore.tryMarkProcessed(idempotencyKey)) {
            try {
                // Process message
                processPayment(message);
                ack(message);
            } catch (Exception e) {
                // Remove on failure to allow retry
                idempotencyStore.remove(idempotencyKey);
                nack(message, requeue=true);
            }
        } else {
            // Already processed, skip but still ack
            log.info("Duplicate message, skipping: {}", idempotencyKey);
            ack(message);
        }
        ```
    - **Additional considerations:**
        1. **Idempotency TTL**: Đủ lâu để cover retry window (24-48h).
        2. **Storage choice**: Redis cho high throughput, DB cho persistence.
        3. **Scope**: Quyết định scope (per message, per user+operation) dựa trên requirements.
- [x] Q6: Exactly-once delivery - possible? How? Trade-offs? (viết analysis với implementation approach)
    - **Possible?** Có thể đạt được với combination của:
        1. **Idempotent producer**: Producer retry với same message ID, broker deduplicate.
        2. **Transactional producer**: Producer dùng transactions để đảm bảo atomicity.
        3. **Idempotent consumer**: Consumer check và skip duplicates.
        4. **Transactional consumer**: Consumer commit offset và process trong transaction.
    - **Kafka Exactly-Once Implementation:**
        1. **Idempotent producer**: Enable `enable.idempotence=true`, producer gửi với sequence number, broker deduplicate.
        2. **Transactional producer**: Enable transactions, producer gửi messages trong transaction, commit transaction.
        3. **Transactional consumer**: Consumer đọc trong transaction, process, commit transaction (offset + processing atomic).
        4. **Read-Process-Write pattern**: Consumer đọc từ input topic, process, write to output topic trong transaction.
    - **Implementation approach:**
        ```java
        // Producer
        producer.initTransactions();
        producer.beginTransaction();
        try {
            producer.send(record1);
            producer.send(record2);
            producer.commitTransaction(); // Atomic: all or nothing
        } catch (Exception e) {
            producer.abortTransaction();
        }
        
        // Consumer
        consumer.subscribe(topics);
        while (true) {
            ConsumerRecords records = consumer.poll();
            for (ConsumerRecord record : records) {
                // Check idempotency
                if (!idempotencyStore.exists(record.id())) {
                    process(record);
                    idempotencyStore.markProcessed(record.id());
                }
            }
            // Commit offset after processing
            consumer.commitSync(); // Atomic with processing
        }
        ```
    - **Trade-offs:**
        1. **Performance**: Overhead của transactions, idempotency checks → latency cao hơn, throughput thấp hơn.
        2. **Complexity**: Code phức tạp hơn, cần handle transactions, idempotency.
        3. **Storage**: Cần lưu idempotency keys, sequence numbers → storage overhead.
        4. **Limited scenarios**: Không đạt được trong mọi scenario (network partitions, split-brain).
        5. **Cost**: Coordination overhead, additional infrastructure (idempotency store).
    - **When to use:**
        - **Financial systems**: Payments, transactions cần chính xác.
        - **Critical operations**: Không thể chấp nhận duplicates hoặc loss.
        - **Compliance**: Audit requirements.
    - **When NOT to use:**
        - **High throughput**: Metrics, logs không cần exactly-once.
        - **Simple systems**: At-least-once + idempotency đủ.
        - **Latency-sensitive**: Real-time systems cần low latency.
    - **Recommendation**: Đa số cases, **at-least-once + idempotency** đủ và đơn giản hơn. Chỉ dùng exactly-once khi thực sự cần thiết (financial, compliance).
- [x] Review: Answers có reason deeply về reliability? Có consider all failure scenarios? Có justify solutions?
    - **Review**: Các answers đã:
        1. ✅ Reason về reliability: phân tích failure scenarios, delivery guarantees.
        2. ✅ Consider failure scenarios: broker failures, consumer crashes, network issues.
        3. ✅ Justify solutions: explain trade-offs, khi nào dùng solution nào.
        4. ✅ Provide implementation details: code examples, patterns.
        5. ✅ Cover edge cases: race conditions, rebalancing, duplicates.
- [x] Improve: Answers nếu cần
    - Answers đã đầy đủ và chi tiết, cover các aspects quan trọng về reliability, failure handling, và implementation.

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
