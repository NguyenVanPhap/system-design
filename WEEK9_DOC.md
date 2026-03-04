# Week 9 – Distributed Systems Core Principles

> **Mentor Note**: Distributed systems là về trade-offs, không phải perfect solutions. CAP theorem forces bạn to choose. Bạn không thể have it all. Mỗi consistency choice bạn make có cost. Bạn phải justify MỌI choice với deep understanding của trade-offs. Systems researcher không accept "I think it's better". Phải có mathematical reasoning, proofs, và real-world evidence. Production systems need justified trade-offs, not wishful thinking.

---

## Study TODOs

### CAP Theorem Deep Dive
- [x] Đọc "Brewer's CAP Theorem" original paper hoặc detailed explanation  
    - **CAP Theorem (Brewer / Gilbert & Lynch)**: trong một hệ thống phân tán có khả năng xảy ra **network partition** (P), bạn **không thể đồng thời** đảm bảo hoàn hảo cả **Consistency (C)** và **Availability (A)**. Khi partition xảy ra, phải **chọn ưu tiên C hoặc A** – đây là trade-off bắt buộc, không phải vấn đề “engineering giỏi hơn”.
- [x] Định nghĩa chính xác: Consistency (linearizability)  
    - **Consistency trong CAP = Linearizability**: mọi thao tác đọc/ghi lên một object phân tán **hành xử như trên một bản sao đơn lẻ** được thực thi tuần tự.  
    - Nghĩa là:  
        - Mỗi operation có một **thời điểm duy nhất** $t$ mà tại đó nó “có hiệu lực”.  
        - Tất cả clients quan sát thứ tự các operations **giống nhau** và **phù hợp với thời gian thực** (real-time order).  
        - Nếu một write $W$ hoàn thành trước khi một read $R$ bắt đầu, thì $R$ **phải** thấy kết quả của $W$.  
    - Linearizability là **mô hình nhất quán mạnh nhất** mà ta thường nhắc đến trong CAP.
- [x] Định nghĩa chính xác: Availability (every request gets response)  
    - **Availability trong CAP**: **mỗi request** từ client gửi đến một node **không bị lỗi** đều phải nhận được **response trong thời gian hữu hạn**, **không được trả lỗi “timeout vì chờ node khác”**.  
    - Quan trọng:  
        - Availability **không yêu cầu response phải là mới nhất**, chỉ cần là response hợp lệ.  
        - Nếu một node còn sống và được client gọi, nó **không được im lặng hoặc từ chối**, trừ khi chính nó crash.  
        - Vì vậy, trong AP system, node ở partition thiểu số vẫn phải **trả lời** dựa trên state hiện tại của nó.
- [x] Định nghĩa chính xác: Partition tolerance (network partition)  
    - **Partition tolerance (P)**: hệ thống vẫn tiếp tục hoạt động (theo nghĩa nào đó) ngay cả khi mạng bị chia thành **nhiều partition** không giao tiếp được với nhau.  
    - Trong môi trường thực tế (WAN, datacenter, cloud), **network partition là không tránh khỏi**, nên hầu hết hệ thống thực tế đều **phải** chấp nhận P.  
    - CAP đặt giả thiết: **partition có thể và sẽ xảy ra** → câu hỏi là **khi partition xảy ra, bạn chọn ưu tiên C hay A**.
- [x] Viết proof: Why can only choose 2 of 3? (mathematical reasoning)  
    - Giả sử:  
        - Có 2 replicas $R_1$ và $R_2$ chứa cùng một object.  
        - Xảy ra network partition: $R_1$ và $R_2$ **không thể giao tiếp**.  
    - Nếu bạn muốn **Consistency (C)**:  
        - Mọi read phải thấy **giá trị mới nhất toàn hệ thống**.  
        - Khi client gửi write đến $R_1$, để đảm bảo C, $R_1$ **phải đồng bộ** với $R_2$ trước khi commit.  
        - Nhưng partition khiến việc đồng bộ **bất khả thi** → $R_1$ **buộc phải từ chối** hoặc **block** request → **mất Availability (A)**.  
    - Nếu bạn muốn **Availability (A)**:  
        - Mọi request tới node còn sống **phải được trả lời** trong thời gian hữu hạn.  
        - Khi partition, $R_1$ nhận write → **phải commit local** và trả kết quả, **không thể chờ $R_2$** (vì sẽ vi phạm A).  
        - $R_2$ có thể tiếp tục phục vụ reads/writes độc lập.  
        - Khi đó, hai replica có thể có **state khác nhau** → **mất Consistency (C)**.  
    - Do đó, **trong điều kiện có P**, bạn **bắt buộc** phải hi sinh C hoặc A → không thể có cả C, A, P cùng lúc.
- [x] Đọc về "CAP theorem misconceptions" - common misunderstandings  
    - **Hiểu sai phổ biến**:  
        1. “Hệ thống này là CA” → Sai, vì trong môi trường thực tế **không thể loại trừ P**, nên **mọi hệ thống thực tế đều phải là CP hoặc AP** khi partition xảy ra.  
        2. “CAP là chọn 2/3 mọi lúc” → Thật ra, **chỉ khi partition xảy ra** mới phải chọn; bình thường có thể có cả C và A.  
        3. “CP = luôn unavailable” → Sai, CP chỉ **từ chối một số requests** trong partition để bảo vệ consistency.  
        4. “AP = inconsistent, không dùng được” → AP chấp nhận **eventual consistency**, nhưng vẫn có nhiều kỹ thuật để hạn chế inconsistency.  
        5. “CAP giải thích mọi thứ về distributed systems” → CAP chỉ là **một lát cắt** (C/A dưới P), không nói gì về **latency, throughput, durability…**.
- [x] Đọc về "CAP theorem in practice" - real-world implications  
    - Trong thực tế:  
        - **CP systems** thường là những hệ thống cần **độ chính xác tuyệt đối**: metadata cluster (Zookeeper, etcd), distributed databases dùng consensus cho writes.  
        - **AP systems** ưu tiên **availability**: nhiều key-value stores như Dynamo, Cassandra (có cấu hình theo hướng AP), các cache cluster.  
        - Phần lớn hệ thống **trộn nhiều vùng C/AP khác nhau**: core ledger có thể CP, reporting / search có thể AP.  
    - Kỹ sư phải **map yêu cầu business** → **mức độ chấp nhận mất tính sẵn sàng vs mất tính nhất quán**.
- [x] Analyze: CP systems (Consistency + Partition tolerance)  
    - **Đặc điểm CP**:  
        - Khi partition, hệ thống **chấp nhận từ chối** hoặc **block** một số requests để giữ C.  
        - Thường dùng **quorum** và **consensus algorithms** (Raft, Paxos) để đảm bảo mọi writes đi qua **đa số nodes**.  
    - **Ưu điểm**:  
        - Đảm bảo **strong consistency** → logic tài chính, quyền hạn, cấu hình hệ thống an toàn.  
        - Dễ **reason** về state của hệ thống, tránh những bug khó chịu do eventual consistency.  
    - **Nhược điểm**:  
        - Trong partition, một số operations sẽ **unavailable** (ví dụ: không cho phép ghi ở partition thiểu số).  
        - Latency thường **cao hơn** vì cần coordination giữa nhiều nodes.
- [x] Analyze: AP systems (Availability + Partition tolerance)  
    - **Đặc điểm AP**:  
        - Khi partition, các partition **vẫn nhận và xử lý requests** (đặc biệt là writes).  
        - Chấp nhận state tạm thời **không nhất quán** giữa các replicas; sau partition sẽ cần **reconciliation**.  
    - **Ưu điểm**:  
        - **High availability** kể cả khi network không ổn định.  
        - Thường có **latency thấp hơn** do tránh coordination chặt chẽ cho mọi operation.  
    - **Nhược điểm**:  
        - Cần cơ chế **conflict resolution** (last-write-wins, vector clocks, CRDTs...).  
        - Business logic phải sống chung với **stale reads**, **anomalies tạm thời** (vd: số dư hiển thị chậm).
- [x] Analyze: CA systems (Consistency + Availability) - why impossible in distributed?  
    - **CA “thuần”** chỉ có thể tồn tại nếu **không có P** (single node, hoặc mạng được giả định là perfect).  
    - Trong môi trường phân tán thực tế:  
        - Partition **không thể tránh** → nếu bạn cố giữ C và A cùng lúc, khi partition xảy ra bạn sẽ **vi phạm định nghĩa C hoặc A**.  
        - Vì vậy, “CA hệ phân tán” thực chất là **CP hoặc AP** khi partition xảy ra; “CA” chỉ mô tả **trạng thái khi mạng bình thường**.  
    - Thực tế, khi người ta nói “CA system” thường là **single-node database** hoặc **cluster trong đó coi P là cực kỳ hiếm và chấp nhận ignore CAP** (nhưng về mặt lý thuyết vẫn không escape được).
- [x] Research: Real-world CP systems (examples)  
    - Ví dụ CP:  
        - **Zookeeper, etcd, Consul (consensus mode)**: dùng cho configuration, service discovery – ưu tiên **consistency**.  
        - **Spanner (ở mức nào đó)**: dùng TrueTime, consensus cho writes để cung cấp external consistency.  
        - **HDFS NameNode HA**: metadata được quản lý theo kiểu CP (chỉ một active NameNode).  
    - Các hệ thống này sẵn sàng **từ chối writes** nếu không đảm bảo quorum.
- [x] Research: Real-world AP systems (examples)  
    - Ví dụ AP (theo cấu hình phổ biến):  
        - **Amazon Dynamo, DynamoDB (kiểu Dynamo)** với eventual consistency cho nhiều chế độ đọc.  
        - **Cassandra, Riak** khi cấu hình quorum nhỏ, ưu tiên availability.  
        - **Many cache clusters** (Redis Cluster, Memcached) khi chấp nhận mất/hỏng một phần dữ liệu tạm thời.  
    - Pattern chung: **ưu tiên uptime**, chấp nhận **reconciliation** sau partition.
- [x] Đọc về "CAP theorem refinements" - PACELC theorem  
    - **PACELC** mở rộng CAP:  
        - Nếu có **Partition (P)**: phải trade-off giữa **Availability (A)** và **Consistency (C)** → **PA/PC**.  
        - **Else (E)** – khi **không có partition**: vẫn phải trade-off giữa **Latency (L)** và **Consistency (C)** → **EL/EC**.  
    - Nhiều hệ thống mô tả theo PACELC:  
        - **Dynamo-style**: PA/EL (ưu tiên Availability khi partition, ưu tiên Latency khi bình thường).  
        - **Spanner-style**: PC/EC (ưu tiên Consistency cả khi partition và khi bình thường, chấp nhận latency cao).  
    - PACELC nhấn mạnh rằng **ngay cả khi không partition**, bạn vẫn luôn trade-off giữa **latency** và **mức độ consistency**.

### Consistency Models
- [x] Đọc về "linearizability" (strong consistency)  
- [x] Định nghĩa chính xác: Linearizability  
    - **Linearizability**: mọi operation trên một object phân tán có thể được coi như thực thi **tức thời** tại một điểm thời gian duy nhất giữa lúc nó bắt đầu và kết thúc.  
    - Tính chất:  
        - Tôn trọng **real-time order**: nếu operation A hoàn thành trước khi B bắt đầu, mọi node đều thấy A trước B.  
        - Mọi client thấy **cùng một thứ tự** operations.  
        - Dễ reason nhất cho developer: “giống như có một biến toàn cục an toàn thread-safe”.  
    - Đây là mô hình **mạnh nhất** thường được nhắc tới cho single object.
- [x] Đọc về "sequential consistency"  
    - **Sequential consistency**: tồn tại một thứ tự tuần tự các operations sao cho:  
        - Thứ tự đó **tôn trọng order chương trình** của từng client.  
        - Nhưng **không bắt buộc** tôn trọng real-time order giữa các client.  
    - Nói cách khác, hệ thống hành xử như thể tất cả operations được execute tuần tự, nhưng thứ tự đó không nhất thiết khớp với thời gian thực.
- [x] So sánh: Linearizability vs Sequential consistency (5 differences)  
    1. **Real-time order**: Linearizability tôn trọng real-time; Sequential không bắt buộc.  
    2. **Độ mạnh**: Linearizability **mạnh hơn** Sequential (Linearizable ⇒ Sequential, chiều ngược lại không đúng).  
    3. **Trực quan cho dev**: Linearizability dễ hiểu hơn vì khớp với trực giác “write xong là mọi người thấy ngay”.  
    4. **Implementation cost**: Linearizability thường **tốn latency hơn** (cần đồng bộ chặt), Sequential có thể tối ưu hơn.  
    5. **Use cases**: Linearizability dùng cho **metadata quan trọng, locks, tiền**; Sequential có thể dùng cho **shared memory multiprocessor** hoặc một số hệ thống replication tối ưu.
- [x] Đọc về "causal consistency"  
- [x] Định nghĩa chính xác: Causal consistency  
    - **Causal consistency**: nếu một operation $A$ **gây ra** (causes) operation $B$ (vd: client đọc kết quả A rồi quyết định ghi B), thì **mọi node** phải thấy A **trước** B.  
    - Các operations **không có quan hệ nhân quả** (concurrent) có thể được thấy theo thứ tự khác nhau ở các replicas.  
    - Thường được implement bằng **vector clocks** hoặc cơ chế metadata để track “happens-before”.
- [x] Đọc về "eventual consistency"  
- [x] Định nghĩa chính xác: Eventual consistency  
    - **Eventual consistency**: nếu **không có thêm updates mới** lên một object, thì **tất cả replicas cuối cùng sẽ hội tụ** về cùng một giá trị.  
    - Trong thời gian ngắn: reads có thể thấy **stale values**, **out-of-order updates**.  
    - Không nói gì về **thời gian hội tụ** cụ thể, chỉ đảm bảo “cuối cùng sẽ nhất quán”.
- [x] Đọc về "strong eventual consistency" (SEC)  
    - **Strong eventual consistency (SEC)**:  
        - Nếu không có thêm updates, mọi replicas hội tụ.  
        - Với cùng một tập updates (không cần cùng thứ tự), mọi replicas sẽ hội tụ về **cùng một state** **mà không cần coordination**.  
    - CRDTs (Conflict-free Replicated Data Types) thường được thiết kế để đạt SEC nhờ các properties như **commutativity, associativity, idempotency**.
- [x] Đọc về "read-your-writes consistency"  
    - **Read-your-writes**: một client **sau khi ghi** một giá trị, **tất cả các lần đọc sau đó của chính client đó** sẽ **ít nhất** thấy giá trị đó (hoặc mới hơn).  
    - Đây là một dạng **session consistency**; không yêu cầu mọi client khác cũng thấy giá trị mới ngay lập tức.
- [x] Đọc về "monotonic reads consistency"  
    - **Monotonic reads**: một client **không bao giờ đọc được giá trị cũ hơn** so với lần đọc trước đó của chính nó.  
    - Nếu lần 1 đọc version v3, lần 2 **ít nhất** phải là v3, không thể quay lại v2.
- [x] Đọc về "monotonic writes consistency"  
    - **Monotonic writes**: các writes từ **một client** được apply theo **đúng thứ tự** mà client đó gửi.  
    - Tránh tình huống update B đến trước update A của cùng một client do routing / replication.
- [x] Đọc về "consistent prefix reads"  
    - **Consistent prefix**: mọi read luôn thấy một **prefix hợp lệ** của chuỗi writes toàn hệ thống.  
    - Ví dụ: nếu chuỗi writes logic là `A → B → C`, thì read có thể thấy `A`, hoặc `A,B`, hoặc `A,B,C`, nhưng **không bao giờ** thấy `A,C` mà không có `B`.
- [x] Compare: All consistency models (10 criteria)  
    1. **Độ mạnh**: Linearizability > Sequential > Causal > Eventual.  
    2. **Real-time awareness**: chỉ Linearizability yêu cầu tuân thủ real-time order.  
    3. **Causality**: Causal + SEC bảo toàn quan hệ nhân quả; Eventual đơn thuần thì không.  
    4. **Latency**: mô hình mạnh hơn thường **latency cao hơn** (cần coordination).  
    5. **Implementation complexity**: Linearizability / strong consistency khó hơn Eventual / Causal.  
    6. **Scalability**: Eventual / Causal thường **scale tốt hơn** trên WAN.  
    7. **Developer reasoning**: Linearizability dễ reason nhất, Eventual khó reason nhất (cần nghĩ về conflicts, convergence).  
    8. **Use case**: tiền, quyền hạn, metadata quan trọng → strong/linearizable; timeline, news feed, counters → eventual/causal/SEC.  
    9. **Conflict handling**: strong consistency tránh nhiều conflicts; eventual cần **conflict resolution** phức tạp hơn.  
    10. **Tolerance với partition**: mô hình yếu hơn thường chấp nhận **AP trade-offs** dễ hơn.
- [x] Research: Which systems use which consistency model?  
    - **Linearizable / Strong consistency**: Zookeeper, etcd, nhiều NewSQL DB (Spanner, CockroachDB cho các thao tác nhất định).  
    - **Sequential / Causal-ish**: một số shared-memory multiprocessor, key-value stores có causal consistency (COPS, Eiger).  
    - **Eventual / SEC**: Dynamo-style systems, Cassandra (default), Riak, CRDT-based systems (AntidoteDB, Redis CRDT modules).

### ACID vs BASE
- [x] Đọc về "ACID properties" (review from Week 3)  
    - **ACID**:  
        - **Atomicity**: transaction là **tất cả hoặc không gì cả**.  
        - **Consistency**: transaction đưa database từ **một state hợp lệ** sang **state hợp lệ khác**, tôn trọng invariants.  
        - **Isolation**: các transaction song song **hành xử như chạy tuần tự** (tuỳ isolation level).  
        - **Durability**: khi commit, dữ liệu **không bị mất** dù crash (được persist).  
    - Hướng tới **tính đúng đắn mạnh** cho từng transaction.
- [x] Đọc về "BASE properties" - Basically Available, Soft state, Eventual consistency  
    - **BASE**:  
        - **Basically Available**: hệ thống **luôn cố gắng trả lời**, chấp nhận trả state cũ hoặc tạm thời.  
        - **Soft state**: state có thể **thay đổi theo thời gian** ngay cả khi không có inputs mới (do background replication, repair).  
        - **Eventual consistency**: cuối cùng các replicas sẽ hội tụ nếu không có updates mới.  
    - BASE không phải “chống ACID” mà là **một style khác** phù hợp cho hệ phân tán lớn, high availability.
- [x] So sánh: ACID vs BASE (10 criteria)  
    1. **Mục tiêu chính**: ACID ưu tiên **correctness**, BASE ưu tiên **availability & scalability**.  
    2. **Consistency**: ACID → strong consistency; BASE → **eventual / weaker consistency**.  
    3. **Isolation**: ACID định nghĩa rõ isolation levels; BASE thường **không có isolation mạnh**, chấp nhận anomalies.  
    4. **Latency**: ACID thường **latency cao hơn** do locking, coordination; BASE hướng đến **low latency**.  
    5. **Scalability**: ACID truyền thống khó scale ngang; BASE sinh ra cho **scale-out** trên nhiều nodes/regions.  
    6. **Error handling**: ACID: transaction fail → rollback; BASE: chấp nhận **inconsistency** và **compensation** sau đó.  
    7. **Use cases**: ACID cho core business invariants; BASE cho analytics, feed, search, caching.  
    8. **Complexity**: ACID push complexity về hạ tầng DB; BASE thường push complexity lên **application layer** (saga, compensating actions).  
    9. **User expectations**: ACID phù hợp nơi user không chấp nhận sai lệch; BASE chấp nhận “đôi khi số hiển thị chậm vài giây”.  
    10. **Failure model**: ACID → thường CP-ish; BASE → thường AP-ish với mechanisms để reconcile.
- [x] Analyze: When to use ACID? (5 use cases)  
    1. **Core payment / banking ledger**: số dư tài khoản, chuyển tiền.  
    2. **Order state machine**: từ CREATED → PAID → SHIPPED → COMPLETED.  
    3. **Inventory deduction chính xác** khi stock thấp.  
    4. **Permission / access control**: thay đổi quyền, roles.  
    5. **Metadata hệ thống quan trọng** (config cluster, leader election metadata).  
    - Các use case này **không chấp nhận** anomalies như double-spend, mất order, state impossible.
- [x] Analyze: When to use BASE? (5 use cases)  
    1. **News feed / timeline**: hiển thị post có thể chậm vài giây.  
    2. **Analytics / reporting**: biểu đồ có thể trễ so với thực tế.  
    3. **Product recommendations**: gợi ý không cần chính xác tuyệt đối tại mọi thời điểm.  
    4. **Search index**: kết quả search có thể thiếu item mới tạo trong vài giây.  
    5. **Caching layers**: cache có thể stale nhưng chấp nhận được.  
    - Điểm chung: **đọc nhiều**, chấp nhận **stale**, ưu tiên **throughput & availability**.
- [x] Đọc về "ACID limitations" trong distributed systems  
    - Trong môi trường phân tán:  
        - Đạt **full ACID** trên nhiều nodes/regions thường yêu cầu **distributed transactions (2PC, Paxos/Raft)** → latency cao, failure modes phức tạp.  
        - **Network partitions** khiến việc giữ strong consistency + availability trở nên **bất khả thi** (CAP).  
        - Isolation mạnh (serializable) trên phạm vi rộng gây **contention**, **deadlock**, **throughput thấp**.  
    - Do đó nhiều hệ thống chọn **weaker isolation**, hoặc **ACID trong boundary nhỏ** (một service) + BASE giữa services.
- [x] Đọc về "BASE trade-offs" - what you sacrifice  
    - BASE **đánh đổi**:  
        - **Immediate consistency** → lấy **eventual consistency**.  
        - **Logic đơn giản ở DB** → đẩy complexity lên **application** (saga, retries, conflict resolution).  
        - **Guarantees mạnh** → lấy **high availability & scalability**.  
    - Hệ thống BASE cần **rất nhiều test** cho failure scenarios: message lost, duplicated, out-of-order, partial failures.
- [x] Research: Real-world ACID systems  
    - **PostgreSQL, MySQL/InnoDB, Oracle, SQL Server**: ACID trong single-node hoặc cluster với replication đồng bộ.  
    - **NewSQL DBs** như **Spanner, CockroachDB** cố gắng cung cấp **distributed ACID transactions** trên nhiều node/region.  
    - Nhiều systems cho phép **transaction ACID trong một shard** nhưng cross-shard thì phải dùng patterns khác.
- [x] Research: Real-world BASE systems  
    - **Amazon Dynamo, Cassandra, Riak**: thiết kế theo hướng **Dynamo-style, eventual consistency**.  
    - **MongoDB (ở nhiều chế độ)**: default từng giai đoạn là eventual, về sau bổ sung nhiều guarantees mạnh hơn nhưng vẫn cho phép cấu hình BASE-ish.  
    - **Search / analytics**: Elasticsearch, Solr, ClickHouse – thường eventual giữa replicas, ưu tiên throughput.  
    - **CDN, cache systems**: chấp nhận stale content để đổi lấy performance & availability.

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
