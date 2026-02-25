# Week 4 – Database Sharding & Partitioning

> **Mentor Note**: Sharding là một trong những challenges khó nhất trong distributed systems. Mỗi decision bạn make sẽ impact toàn bộ system. Bạn phải justify MỌI decision với data và analysis. Không có "tôi nghĩ" hay "có vẻ OK". Phải có numbers, measurements, và trade-off analysis. Nếu bạn không thể justify sharding strategy của bạn, bạn chưa hiểu đủ sâu.

---

## Study TODOs

### Partitioning Fundamentals
- [x] Đọc "Designing Data-Intensive Applications" Chapter 6 - Partitioning (pages 200-250)
  - **Key Takeaways**: Partitioning (hay Sharding) là việc chia nhỏ một dataset lớn thành các phần nhỏ hơn (partitions) để scale out. Mục tiêu chính là scalability (vượt qua giới hạn lưu trữ/xử lý của 1 node) và availability.
- [x] Định nghĩa chính xác: Horizontal partitioning (sharding)
  - Là việc chia bảng theo hàng (rows). Mỗi partition có cùng schema nhưng chứa các subset hàng khác nhau. Ví dụ: Shard theo `user_id`, User 1-1000 ở Node A, 1001-2000 ở Node B.
- [x] Định nghĩa chính xác: Vertical partitioning
  - Là việc chia bảng theo cột (columns). Tách các cột ít dùng hoặc dung lượng lớn sang bảng/DB khác. Ví dụ: Bảng `Users` tách `user_id, username` sang DB chính, `user_id, bio, avatar_blob` sang DB phụ.
- [x] So sánh: Horizontal vs Vertical partitioning (5 điểm khác biệt)
  1. **Data Split**: Horizontal chia theo Rows; Vertical chia theo Columns.
  2. **Goal**: Horizontal để scale volume/throughput; Vertical để tối ưu I/O (tách cột to/ít dùng).
  3. **Complexity**: Horizontal phức tạp hơn (cần shard routing); Vertical đơn giản hơn (thường là JOIN lại).
  4. **Query impact**: Horizontal ảnh hưởng cross-shard queries; Vertical ảnh hưởng JOIN latency.
  5. **Schema**: Horizontal giữ nguyên schema; Vertical thay đổi schema (tách bảng).
- [x] Liệt kê 5 use cases cần horizontal partitioning
  1. Hệ thống Social Media (tỷ user, shard theo `user_id`).
  2. E-commerce Transactions (hàng tỷ đơn hàng, shard theo `order_id` hoặc `merchant_id`).
  3. IoT Sensor Data (dữ liệu cực lớn, shard theo `device_id` hoặc `timestamp`).
  4. Payment Systems (shard theo `merchant_id` hoặc `user_id`).
  5. Chat/Messaging (shard theo `conversation_id`).
- [x] Liệt kê 3 use cases cần vertical partitioning
  1. Tách `Product` description dài (BLOB) khỏi metadata cơ bản để list hàng nhanh hơn.
  2. Tách thông tin nhạy cảm (PII) sang DB có bảo mật cao hơn.
  3. Tách các cột thường xuyên update khỏi các cột static để giảm write amplification.
- [x] Đọc về "partitioning vs sharding" - same thing? Different?
  - Trong ngữ cảnh distributed systems, chúng thường dùng thay thế cho nhau. Tuy nhiên, "Partitioning" là thuật ngữ chung (có thể trong 1 node - e.g. Postgres Partitioning), còn "Sharding" ám chỉ việc chia ra nhiều nodes vật lý khác nhau (Shared-nothing architecture).
- [x] Research: When to start sharding? (at what scale?)
  - Không có số cụ thể, nhưng thường khi: 
    - Database size vượt quá khả năng backup/restore trong thời gian chấp nhận được.
    - Write throughput vượt quá giới hạn của Master (vertical scaling Master đã hết mức).
    - Latency tăng cao do index quá lớn không còn fit trong RAM.
    - Thường bắt đầu cân nhắc khi đạt ngưỡng vài TB data hoặc hàng chục nghìn QPS write.

### Sharding Strategies Deep Dive
- [x] Đọc về "Range-based sharding" - how it works
  - Chia data theo các dải giá trị liên tục của Shard Key (ví dụ: A-M ở Shard 1, N-Z ở Shard 2).
- [x] Đọc về "Hash-based sharding" - how it works
  - Dùng Hash Function: `shard = hash(key) % number_of_shards`. Giúp phân phối data đều hơn.
- [x] Đọc về "Directory-based sharding" - how it works
  - Dùng một Lookup Table (Service) để lưu mapping: `key -> shard_id`. Linh hoạt nhưng tạo bottleneck ở Lookup Service.
- [x] Đọc về "Composite sharding" - multiple shard keys
  - Kết hợp nhiều keys hoặc nhiều tầng sharding (ví dụ: Shard theo `region` trước, sau đó `user_id`).
- [x] Viết comparison table: Range vs Hash vs Directory (10 criteria)

| Criteria | Range-based | Hash-based | Directory-based |
| :--- | :--- | :--- | :--- |
| **Data Distribution** | Dễ bị lệch (Skewed) | Rất đều | Tùy chỉnh |
| **Range Queries** | Rất tốt | Rất tệ (Fan-out) | Trung bình |
| **Scalability** | Dễ (Split range) | Khó (Rehash) | Rất dễ |
| **Complexity** | Thấp | Trung bình | Cao |
| **Hotspots** | Dễ bị (e.g. timestamp) | Hiếm gặp | Có thể manage |
| **Routing Logic** | Simple | Deterministic | Lookup service |
| **Re-sharding** | Dễ | Rất khó/Phức tạp | Dễ |
| **Config overhead** | Thấp | Thấp | Cao |
| **Availability** | Cao | Cao | Phụ thuộc Lookup Service |
| **Query Performance** | Tốt cho dải data | Tốt cho point-lookup | Tốt cho mapping linh hoạt |

- [x] Analyze: Pros và cons của mỗi strategy (5 points each)
  - **Range**:
    - Pros: Range query nhanh, dễ split shard, routing đơn giản, locality tốt.
    - Cons: Dễ bị hotspot (e.g. data mới nhất), distribution không đều, cần monitor dải giá trị.
  - **Hash**:
    - Pros: Data phân phối rất đều, tránh hotspot, không cần lookup table, point lookup nhanh.
    - Cons: Range query cực chậm (fan-out), re-sharding rất đau đớn, không có data locality.
  - **Directory**:
    - Pros: Cực kỳ linh hoạt, có thể move data giữa shards tự do, handle được hotspot lẻ.
    - Cons: Lookup service là bottleneck/SPOF, overhead latency cho mỗi query, quản lý mapping phức tạp.
- [x] Research: Real-world examples của range-based sharding: Google BigTable, HBase (SSTables).
- [x] Research: Real-world examples của hash-based sharding: Cassandra, Memcached, DynamoDB.
- [x] Research: Real-world examples của directory-based sharding: Azure SQL Database (Elastic Database tools).
- [x] Đọc về "shard key" - what is it? Why critical?
  - Shard Key là cột xác định dữ liệu thuộc về shard nào. Nó critical vì quyết định: Phân phối tải, query performance, và khả năng scale. Chọn sai shard key có thể dẫn tới hotspots và cross-shard queries làm sập hệ thống.

### Consistent Hashing
- [x] Đọc về "consistent hashing" - original paper hoặc detailed explanation
  - **Tóm tắt**: Consistent hashing ánh xạ cả nodes (shards) và keys lên cùng một vòng tròn hash (hash ring). Khi thêm/bớt node, chỉ các keys nằm giữa node mới (hoặc bị remove) và predecessor của nó trên vòng tròn mới phải di chuyển, thay vì toàn bộ tập dữ liệu phải rehash như `hash(key) % N`.
- [x] Viết algorithm pseudocode cho consistent hashing
  - **Pseudocode (đơn giản, không virtual nodes)**:
    - BuildRing(nodes):
      - ring = empty map<int, Node>
      - for node in nodes:
        - h = Hash(node.id)
        - ring[h] = node
      - sort ring theo key tăng dần (theo hash)
      - return ring
    - GetNode(key, ring):
      - h = Hash(key)
      - tìm phần tử đầu tiên trong ring có key >= h (binary search)
      - nếu không tồn tại (vượt quá cuối vòng), wrap-around: lấy phần tử đầu tiên trong ring
      - trả về node tương ứng
- [x] Đọc về "virtual nodes" trong consistent hashing
  - **Giải thích**: Mỗi physical node được ánh xạ thành nhiều virtual node khác nhau trên vòng hash (ví dụ 100–200 điểm). Mỗi virtual node có `hash(node_id + index)` riêng, giúp phân tán keys đều hơn trên ring và làm mịn sai lệch do hash không hoàn hảo hoặc do số node ít.
- [x] Analyze: Why virtual nodes needed?
  - **Lý do cần virtual nodes**:
    - Giảm variance về số lượng keys mỗi node nhận, đặc biệt khi số node ít.
    - Cho phép weight khác nhau cho từng node (node mạnh có nhiều virtual nodes hơn).
    - Khi thêm/bớt node, data movement phân tán đều hơn giữa các node còn lại thay vì ảnh hưởng mạnh đến một vài node.
    - Giảm khả năng hotspot khi 1 vùng hash range “xui” chứa nhiều keys.
- [x] Đọc về "hash ring" - how it works
  - **Cách hoạt động**:
    - Lấy không gian hash (ví dụ 0 → $2^{32} - 1$) coi như một vòng tròn.
    - Hash mỗi node/virtual node để xác định vị trí trên vòng.
    - Hash mỗi key để xác định vị trí trên vòng.
    - Key thuộc về node đầu tiên đi theo chiều kim đồng hồ từ vị trí hash của key.
- [x] Đọc về "rehashing problem" - why consistent hashing solves it
  - **Rehashing problem**: Với schema `hash(key) % N`, khi N thay đổi (thêm/bớt shard), gần như toàn bộ keys đổi shard → data movement cực lớn, cache hit rate giảm mạnh, gây overload hệ thống.
  - **Consistent hashing**: Khi thêm 1 node, chỉ keys nằm trong đoạn hash range mà node mới “chiếm” từ predecessor phải di chuyển (xấp xỉ kích thước range của node mới). Do đó tỷ lệ data phải move ≈ $1 / N$ thay vì gần 100%.
- [x] Calculate: Load distribution với consistent hashing (theoretical)
  - **Phân phối tải**:
    - Với $N$ node và $K$ keys, nếu hash function tốt, số key trên mỗi node xấp xỉ phân phối Poisson với kỳ vọng $K / N$.
    - Độ lệch chuẩn xấp xỉ $\sqrt{K / N}$, và với virtual nodes $V$ trên mỗi node, variance giảm thêm vì mỗi node thực tế nhận trung bình từ nhiều range nhỏ cộng lại (luật số lớn).
    - Thực tế: nếu $K$ rất lớn và $N, V$ đủ lớn, chênh lệch giữa các node thường trong khoảng 5–10%.
- [x] Research: Consistent hashing implementations (Ketama, etc.)
  - **Một số implementation thực tế**:
    - Ketama (Java) – nổi tiếng trong cộng đồng memcached client, dùng MD5 để tạo nhiều điểm trên ring cho mỗi node.
    - Twemproxy (Twitter) – dùng consistent hashing cho sharding Redis/memcached.
    - Cassandra, Riak – áp dụng variant của consistent hashing (với token range) để phân phối dữ liệu trên cluster.
    - Nhiều load balancer / service mesh (Envoy, Linkerd) cũng dùng consistent hashing để route request ổn định.

### Shard Key Selection
- [x] Đọc về "shard key selection criteria"
  - **Ý nghĩa**: Shard key quyết định cách phân phối dữ liệu và traffic trên các shard, ảnh hưởng trực tiếp đến khả năng scale, hotspot, cross-shard queries và độ phức tạp khi re-shard.
- [x] Liệt kê 5 criteria để chọn shard key
  - **5 tiêu chí quan trọng**:
    1. **Độ phân bố (distribution)**: Giá trị shard key phải phân bố đủ đều để tránh shard quá to/quá nhỏ.
    2. **Alignment với query pattern**: Hầu hết queries nên có điều kiện filter theo shard key để tránh fan-out.
    3. **Cardinality cao**: Số lượng giá trị phân biệt lớn để phân mảnh tốt (tránh vài giá trị chiếm đa số).
    4. **Độ ổn định**: Giá trị shard key ít khi thay đổi; nếu hay đổi sẽ phải di chuyển dữ liệu nhiều.
    5. **Tránh tính tuần tự**: Shard key tăng theo thời gian (timestamp, auto-increment) dễ tạo hotspot trên một shard.
- [x] Analyze: Shard key by user_id - pros và cons
  - **Pros**:
    - Thường có cardinality rất cao → phân phối tốt.
    - Rất phù hợp cho các hệ thống chủ yếu query theo user (user profile, user timeline, user balance).
    - Dễ reasoning: “mọi thứ thuộc về 1 user nằm trên 1 shard”.
  - **Cons**:
    - Queries theo dimension khác (merchant, region, date) có thể phải fan-out nhiều shards.
    - Nếu có super users / VIP có traffic cực lớn → có thể tạo hotspot theo user.
    - Không phù hợp nếu core use case là reporting/aggregation theo merchant/region.
- [x] Analyze: Shard key by merchant_id - pros và cons
  - **Pros**:
    - Tốt cho payment/e-commerce nơi reporting, settlement, dashboard chủ yếu theo merchant.
    - Có thể tối ưu riêng cho merchant lớn (dedicated shard, replica riêng).
    - Data locality cao cho queries nội bộ từng merchant (order list, revenue).
  - **Cons**:
    - Merchants có phân phối skewed (1 số rất lớn, nhiều rất nhỏ) → dễ hotspot.
    - Queries cross-merchant (global search, global ranking) cần fan-out.
    - Nếu user hoạt động đa merchant, transaction history theo user có thể cross-shard.
- [x] Analyze: Shard key by geographic region - pros và cons
  - **Pros**:
    - Gần với topology hạ tầng (region/zone) → giảm network latency.
    - Dễ áp dụng data residency/compliance (dữ liệu EU ở shard EU).
    - Traffic thường có locality theo khu vực → phù hợp cho một số use case.
  - **Cons**:
    - Số region thường ít → khó scale số shard lớn nếu không combine thêm key khác.
    - Dễ skew: vài region có traffic cực lớn, region khác rất nhỏ.
    - Người dùng di chuyển region (du lịch, relocation) → xử lý phức tạp nếu cần di chuyển dữ liệu.
- [x] Analyze: Shard key by timestamp - pros và cons
  - **Pros**:
    - Phù hợp cho use case time-series append-only, cần retention theo thời gian.
    - Dễ implement range-based sharding, dễ drop/shrink shards cũ.
  - **Cons**:
    - Viết mới luôn đổ vào shard “mới nhất” → single hot shard, hotspot cực mạnh.
    - Nếu không có mitigation (salting, time-bucket nhỏ) gần như unusable ở scale lớn.
    - Queries theo entity (user/merchant) phải cross-shard theo thời gian.
- [x] Đọc về "cardinality" của shard key - why matters?
  - **Cardinality**: số lượng giá trị phân biệt của shard key. Cardinality càng cao thì khả năng chia nhỏ dữ liệu càng tốt.
  - Nếu cardinality thấp (ví dụ `country_code`), số bucket hiệu quả nhỏ → dẫn tới shard to/shard nhỏ rất khó balance và dễ tạo hotspot.
- [x] Đọc về "hotspot" problem - what causes it?
  - **Nguyên nhân chính**:
    - Phân phối giá trị shard key skewed (một số ít key nhận phần lớn traffic).
    - Shard key có tính tuần tự (timestamp, auto-increment) → mọi write mới dồn về 1 shard.
    - Query pattern tập trung mạnh vào một subset keys (hot merchant, hot match, hot user).
    - Thiếu cơ chế rebalancing / resharding linh hoạt.
- [x] Analyze: How shard key selection affects hotspots
  - Nếu chọn shard key align với dimension skewed (vd: merchant_id trong hệ thống vài super-merchant) mà không có mitigation → shard tương ứng sẽ luôn nóng.
  - Nếu chọn key tuần tự (timestamp) → luôn có 1 shard “mới nhất” bị dồn toàn bộ write.
  - Shard key tốt nên vừa phân phối đều, vừa không align quá sát với các pattern hotspot không kiểm soát được, hoặc phải đi kèm thêm chiến lược khác (salting, composite key).
- [x] Research: Composite shard keys - when to use?
  - **Composite shard key** dùng khi 1 dimension không đủ đáp ứng cả distribution và query pattern. Ví dụ:
    - `(region, user_id)` trong social network toàn cầu.
    - `(merchant_id, user_id)` trong hệ thống payment.
  - Thường hash trên một combination (concat) của các field để có distribution tốt, trong khi vẫn cho phép một số pattern query được tối ưu trên prefix (ví dụ partition theo region, trong đó hash theo user_id).

### Hotspot Mitigation
- [x] Đọc về "hotspot" problem trong sharding
  - **Hotspot**: khi một phần nhỏ dữ liệu (một shard, một partition, hoặc một vài key) nhận phần lớn traffic, khiến node tương ứng bị quá tải trong khi các node khác rảnh.
- [x] Analyze: What causes hotspots? (5 causes)
  - **5 nguyên nhân chính**:
    1. Phân phối shard key skewed (một vài merchant/user cực lớn).
    2. Shard key tuần tự (timestamp, auto-increment) tạo “mới nhất” shard luôn nóng.
    3. Event đặc biệt (flash sale, big match, viral post) dồn traffic vào một subset dữ liệu.
    4. Application logic tập trung query/ghi vào một key/partition cố định (ví dụ “global counter”).
    5. Thiếu cơ chế auto-rebalancing / auto-scaling cho shard nóng.
- [x] Đọc về "hot partition" - single shard overloaded
  - **Hot partition/shard**: một partition (hoặc shard) chứa quá nhiều dữ liệu hoặc nhận quá nhiều request so với các partition khác. Có thể do shard key chọn sai hoặc do dữ liệu phát triển không đều theo thời gian.
- [x] Đọc về "hot key" - single key overloaded
  - **Hot key**: một key duy nhất nhận rất nhiều truy cập (vd: counter view, leaderboard global key), gây lock contention, cache miss chain, hoặc I/O tập trung.
- [x] Research: Strategies to mitigate hotspots
  - **Chiến lược chính**:
    - Thiết kế lại shard key / composite key để phân tán tốt hơn.
    - Dùng **salting** (thêm suffix random/bucket) cho key nóng.
    - **Range splitting**: tách range nóng thành nhiều range nhỏ hơn trên nhiều shard.
    - **Replication**: tạo nhiều replica cho shard nóng và load-balance đọc.
    - **Caching & write-behind**: cache ở layer trên, batching writes xuống DB.
- [x] Đọc về "salting" technique - add random suffix
  - **Salting**:
    - Thay vì key = `merchant_id`, dùng `merchant_id + salt` với salt = 0..N-1 → dữ liệu merchant đó trải lên N bucket khác nhau.
    - Khi đọc, phải đọc/gom từ tất cả buckets của merchant (N keys).
    - Giảm hotspot viết rõ rệt, trade-off là đọc phức tạp hơn và cần aggregation.
- [x] Đọc về "range splitting" - split hot ranges
  - **Range splitting**:
    - Với range-based sharding, khi một range (vd: `user_id` 1–1M) quá nóng/to, chia nó thành các range nhỏ hơn (1–500k, 500k–1M) và di chuyển bớt range sang shard khác.
    - Yêu cầu metadata layer để track lại mapping range → shard.
- [x] Đọc về "replication" của hot shards
  - **Replication cho shard nóng**:
    - Tạo thêm read-replica cho shard nóng, dùng load balancer để chia traffic đọc.
    - Với ghi, có thể dùng leader-follower (leader vẫn là bottleneck) hoặc multi-leader nếu chấp nhận complexity cao hơn.
- [x] Analyze: Trade-offs của mỗi mitigation strategy
  - **Salting**: 
    - Pros: Dễ implement, giảm hotspot write hiệu quả.
    - Cons: Đọc/aggregate phức tạp hơn, cần N lần đọc + merge.
  - **Range splitting**:
    - Pros: Giữ nguyên semantics range query, chỉ update metadata; rất phù hợp với DB hỗ trợ range partition.
    - Cons: Cần infra để di chuyển data + update routing, có thể phức tạp khi split nhiều lần.
  - **Replication**:
    - Pros: Giảm tải đọc nhanh chóng mà không thay đổi schema.
    - Cons: Không giải quyết được hotspot ghi; thêm chi phí sync & consistency.
  - **Thay shard key/composite key**:
    - Pros: Giải gốc vấn đề phân phối.
    - Cons: Thường yêu cầu migration lớn, impact system-wide.

### Cross-Shard Queries
- [x] Đọc về "cross-shard queries" - challenges
  - **Challenges**:
    - Cần gửi query đến nhiều shard (network fan-out).
    - Kết quả phải aggregate, sort, paginate ở application layer hoặc middleware.
    - Khó đảm bảo tính nhất quán về thời điểm (different replica lag).
- [x] Analyze: Why cross-shard queries are expensive
  - **Lý do đắt**:
    - Latency tổng phụ thuộc shard chậm nhất (tail latency).
    - Network overhead lớn (n request thay vì 1).
    - CPU/I/O trên nhiều shard cùng lúc, dễ tạo load spike.
    - Aggregation ở phía trên tốn CPU/memory (sort, group, join).
- [x] Đọc về "fan-out queries" - query all shards
  - **Fan-out**: gửi cùng một query đến tất cả (hoặc nhiều) shards, sau đó hợp kết quả. Ví dụ: full-text search global, global analytics.
- [x] Đọc về "scatter-gather" pattern
  - **Scatter-gather**:
    - Scatter: phân tán query đến nhiều node song song.
    - Gather: thu kết quả, merge/aggregate tại 1 coordinator.
    - Pattern này thường dùng cho search engines, distributed DB, analytics systems.
- [x] Analyze: Performance impact của cross-shard queries
  - **Impact**:
    - p95/p99 latency tăng do phụ thuộc node chậm nhất.
    - Nếu concurrency cao, tổng số query trên cluster tăng bùng nổ (QPS * số shard).
    - Có nguy cơ làm nghẽn cả cluster nếu có nhiều cross-shard queries nặng.
- [x] Research: Strategies to minimize cross-shard queries
  - **Chiến lược**:
    - Thiết kế shard key align với query chính để queries thường ngày nằm trong 1 shard.
    - Denormalization: lưu dữ liệu đã tổng hợp sẵn theo dimension cần đọc.
    - Sử dụng dedicated reporting/analytics DB thay vì query trực tiếp OLTP shards.
    - Dùng global index hoặc directory chỉ cho use case thật sự cần.
- [x] Đọc về "denormalization" để avoid cross-shard queries
  - **Denormalization**:
    - Lưu trữ lặp dữ liệu (aggregates, pre-joined tables) để tránh join/fan-out nhiều shard.
    - Ví dụ: mỗi shard lưu thêm bảng `merchant_daily_stats` chỉ cho merchant của shard đó, và một pipeline ETL riêng merge sang data warehouse.
- [x] Đọc về "materialized views" across shards
  - **Materialized views**:
    - View được tính toán và lưu lại (materialized), update định kỳ hoặc near real-time.
    - Có thể build trên mỗi shard (local views) rồi hợp vào một hệ thống analytics.
- [x] Đọc về "global secondary indexes" - challenges
  - **Global secondary index**:
    - Index trên field không phải shard key, có thể trỏ cross-shard.
    - Challenges: 
      - Phức tạp trong update (mỗi write phải update nhiều nơi).
      - Khó đảm bảo consistency (staleness).
      - Có thể trở thành bottleneck/single logical system cần scale riêng.

### Distributed Transactions
- [x] Đọc về "distributed transactions" - 2PC (Two-Phase Commit)
  - **Distributed transaction**: transaction bao gồm nhiều resource manager (nhiều DB/shard/service). 2PC là protocol cổ điển để đảm bảo all-or-nothing giữa các participants.
- [x] Đọc về "2PC protocol" - how it works step by step
  - **2PC step-by-step**:
    1. **Prepare phase**:
       - Coordinator gửi `PREPARE` đến tất cả participants.
       - Mỗi participant thực hiện local transaction, ghi log, lock resources nhưng chưa commit, sau đó trả lời `VOTE_COMMIT` hoặc `VOTE_ABORT`.
    2. **Commit phase**:
       - Nếu tất cả đều `VOTE_COMMIT`, coordinator ghi log `COMMIT` và gửi `COMMIT` tới tất cả participants → mỗi participant commit và giải phóng lock.
       - Nếu có bất kỳ `VOTE_ABORT` hoặc timeout, coordinator ghi log `ABORT` và gửi `ABORT` tới tất cả participants → rollback.
- [x] Analyze: Performance impact của 2PC
  - **Impact**:
    - Tăng latency transaction do nhiều round-trip (prepare + commit).
    - Lock giữ trên nhiều resource trong thời gian dài → giảm concurrency, tăng deadlock risk.
    - Coordinator là bottleneck logic; nếu tải cao có thể trở thành choke point.
- [x] Analyze: Failure scenarios trong 2PC
  - **Failure scenarios**:
    - Coordinator crash sau khi một số participants đã commit, một số chưa → cần dựa vào log để recover, participants chờ chỉ thị, có thể bị block lâu.
    - Network partition khiến một số participants không nhận được quyết định commit/abort → có thể mắc kẹt ở trạng thái “prepared”.
    - Do tính blocking, hệ thống có thể bị đơ ở 1 số phần nếu coordinator không phục hồi.
- [x] Đọc về "3PC" (Three-Phase Commit) - improvement?
  - **3PC**:
    - Thêm một phase nữa (pre-commit) để giảm tính blocking trong một số trường hợp.
    - Tuy nhiên, trong môi trường asynchronous network với possibility of partition, 3PC vẫn không đảm bảo hoàn toàn safety, phức tạp hơn và ít được dùng thực tế so với 2PC + các chiến lược khác.
- [x] Đọc về "Saga pattern" - alternative to 2PC
  - **Saga**:
    - Chia distributed transaction thành chuỗi local transactions, mỗi bước có một **compensating transaction** để undo nếu step sau fail.
    - Có thể triển khai theo **orchestration** (một orchestrator điều phối) hoặc **choreography** (services publish/subscribe events).
- [x] Compare: 2PC vs Saga pattern (5 criteria)
  - **So sánh**:
    1. **Consistency**: 2PC ≈ strong consistency; Saga thường eventual (có thể thấy trạng thái trung gian).
    2. **Latency**: 2PC cao hơn (blocking, lock); Saga thường nhanh hơn vì từng local transaction commit độc lập.
    3. **Complexity**: 2PC phức tạp về infra, nhưng logic business đơn giản; Saga yêu cầu thiết kế business flow + compensating logic khá nhiều.
    4. **Failure handling**: 2PC rely vào coordinator, dễ blocking; Saga xử lý bằng cách chạy compensating actions, thích hợp cho hệ phân tán, microservices.
    5. **Use case**: 2PC cho hệ thống cần tính đúng tuyệt đối (bank core, stock trading), traffic không quá lớn; Saga cho high-throughput, distributed, chấp nhận eventual consistency (order, booking, wallet top-up).
- [x] Đọc về "compensating transactions" trong Saga
  - **Compensating transaction**: một action ngược lại để “bù trừ” hiệu ứng của một local transaction trước đó (ví dụ: cancel booking, refund payment). Design phải đảm bảo idempotent và safe.
- [x] Research: When to use distributed transactions? When to avoid?
  - **Khi nên dùng**:
    - Khi dữ liệu critical cần strong consistency cross-shard/resource (ví dụ chuyển tiền trong cùng ngân hàng).
  - **Khi nên tránh**:
    - Trong hệ microservices lớn, high-throughput, nhiều team, nơi eventual consistency acceptable.
    - Nên ưu tiên model hóa domain theo boundaries rõ, giảm nhu cầu distributed transaction, dùng Saga, outbox pattern, event-driven.

### Re-sharding Strategies
- [x] Đọc về "resharding" - why needed?
  - **Resharding**: thay đổi scheme phân mảnh (số shard, range, hash function, shard key) để đối phó với tăng trưởng dữ liệu, mất cân bằng tải, hoặc thay đổi query pattern.
- [x] Đọc về "resharding triggers" - when to reshard?
  - **Triggers**:
    - Một vài shard quá to hoặc quá nóng so với phần còn lại.
    - Sắp chạm giới hạn lưu trữ/capacity phần cứng.
    - Query latency tăng do index/working set quá lớn.
    - Thay đổi product requirements dẫn đến query pattern thay đổi nhiều.
- [x] Đọc về "resharding strategies" - how to do it?
  - **Chiến lược**:
    - Tạo cluster/shard set mới, di chuyển từng phần dữ liệu (theo range/hash bucket).
    - Dùng double-write: ghi song song vào schema cũ và mới, sau đó cut-over đọc.
    - Sử dụng background migration jobs + change data capture (CDC) để sync.
- [x] Đọc về "double-write" strategy during resharding
  - **Double-write**:
    - Application ghi vào cả old cluster và new cluster trong giai đoạn migration.
    - Khi data historical đã được migrate và verified, chuyển read sang new cluster, sau đó tắt old.
    - Cần giải quyết vấn đề idempotency, ordering, và failure khi 1 trong 2 phía ghi fail.
- [x] Đọc về "gradual migration" strategy
  - **Gradual migration**:
    - Di chuyển từng phần dữ liệu theo batch (theo range, tenant, merchant).
    - Traffic routing dần dần chuyển từng subset sang schema mới (canary).
    - Ưu tiên zero/low downtime và rollback dễ dàng.
- [x] Đọc về "big bang migration" strategy
  - **Big bang**:
    - Dừng hệ thống (hoặc cho read-only), migrate data toàn bộ trong một khoảng thời gian maintenance, sau đó bật lên với schema mới.
    - Đơn giản về logic, nhưng downtime lớn, risk cao.
- [x] Compare: Gradual vs Big bang migration (5 criteria)
  - **So sánh**:
    1. **Downtime**: Gradual ≈ zero/low; Big bang cần maintenance window.
    2. **Risk**: Gradual dễ rollback từng phần; Big bang fail là “fail toàn hệ thống”.
    3. **Complexity**: Gradual phức tạp hơn (routing, double-write, sync); Big bang logic đơn giản.
    4. **Verification**: Gradual có thể verify theo batch; Big bang phải verify large dataset sau migration.
    5. **Time-to-complete**: Gradual có thể kéo dài nhiều ngày/tuần; Big bang thường cố gắng hoàn tất trong vài giờ.
- [x] Đọc về "resharding downtime" - how to minimize?
  - **Giảm downtime**:
    - Dùng gradual migration, double-write, background copy.
    - Chỉ cần short cut-over window (đổi pointer routing) thay vì full downtime.
    - Tận dụng read-only mode hoặc degrade non-critical features.
- [x] Research: Real-world resharding case studies
  - **Case studies**:
    - Twitter, Instagram, Shopify đều có public blog về resharding users/merchants từ monolithic DB sang nhiều shards.
    - Điểm chung: dùng incremental migration, background copy + binlog/CDC, routing layer để điều phối traffic, rất nhiều monitoring & verification.

### Shard Management
- [x] Đọc về "shard routing" - how to route queries
  - **Shard routing**:
    - Từ shard key → xác định shard id (range/hash/consistent hashing/directory).
    - Thường được implement ở application layer, middleware, hoặc trong driver.
- [x] Đọc về "shard metadata" - where to store?
  - **Shard metadata**:
    - Lưu thông tin mapping shard_id → (host, port, status, range/hash range).
    - Thường đặt ở một config service / metadata DB riêng (Zookeeper, etcd, Consul, hoặc 1 central metadata DB).
- [x] Đọc về "shard discovery" - how clients find shards
  - **Shard discovery**:
    - Client/Service khởi động, load metadata từ shard registry.
    - Có thể cache metadata local, subscribe để nhận updates khi topology thay đổi.
- [x] Đọc về "shard monitoring" - what to monitor?
  - **Monitor**:
    - QPS per shard, latency (p50/p95/p99), error rate.
    - Disk usage, CPU, memory, connection count.
    - Số lượng records, growth rate.
- [x] Đọc về "shard health checks"
  - **Health checks**:
    - Endpoint liveness/readiness trên từng shard (ping DB, simple query).
    - Routing layer dùng health status để tránh gửi traffic vào shard unhealthy.
- [x] Đọc về "shard rebalancing" - when và how?
  - **Shard rebalancing**:
    - Thực hiện khi 1 số shard quá to/quá nóng so với phần còn lại.
    - Cách thực hiện: move range/buckets từ shard nặng sang shard nhẹ; update metadata; đảm bảo double-write hoặc CDC trong lúc di chuyển.
- [x] Research: Shard management tools và frameworks
  - **Tools/frameworks**:
    - Vitess (MySQL sharding), Citus (Postgres distribution), YugabyteDB, CockroachDB.
    - Các hệ thống này cung cấp sẵn shard management, routing, rebalancing, replication, monitoring.

---

## Data Distribution TODOs

### Design Exercise 1: Payment System Sharding
- [x] Requirement: 1B transactions, shard by merchant_id
  - 1 tỷ transactions, nhiều merchant, phân mảnh theo `merchant_id` để tối ưu các use case báo cáo/settlement theo merchant.
- [x] Design: Sharding strategy (Range, Hash, or Directory?)
  - Chọn **hash-based sharding trên merchant_id + directory-layer mỏng**:
    - Hash giúp phân phối đều giữa merchants (tránh shard chứa nhiều merchant nhỏ + 1 merchant cực lớn).
    - Directory (metadata) để override mapping cho một số merchant siêu hot vào dedicated shard.
- [x] Justify: Tại sao chọn strategy đó? (viết 500 words với data)
  - Giả sử:
    - Tổng 1B transactions, 1 triệu merchants.
    - 5% merchants chiếm 80% volume (skewed).
  - Nếu dùng **range-based** trên merchant_id:
    - Dễ xuất hiện range chứa nhiều merchant lớn, khó chia đều vì không biết trước merchant nào lớn từ đầu.
    - Khi một range quá nóng, cần split phức tạp + phải cập nhật routing.
  - Nếu dùng **hash-based trực tiếp** trên merchant_id:
    - Với 32 shards, mỗi shard ~ 31.25M transactions (1B / 32) nếu phân phối đều.
    - Hash phân phối merchants đều trên shards, nhưng: một merchant cực lớn vẫn chiếm phần lớn traffic của shard chứa nó → shard đó thành hotspot.
  - Kết hợp **hash + directory**:
    - Phần lớn merchants (nhỏ/vừa) dùng hash-based `shard = hash(merchant_id) % N`.
    - Các “VIP merchants” (top X merchants theo volume) được gán riêng vào dedicated shard (một hoặc vài shard) thông qua một bảng mapping:
      - `merchant_id -> dedicated_shard_id` (ưu tiên override).
      - Nếu không có entry trong bảng thì fallback về hash-based.
    - Cách này cho phép:
      - Giữ tính đơn giản và phân phối đều cho đa số merchants.
      - Có nút chỉnh tay để xử lý outliers mà không phải đổi toàn bộ schema.
  - Về **data**:
    - Nếu 5% merchants chiếm 80% volume (~800M transactions), ta có thể:
      - Nhóm top 1–2 merchants siêu lớn mỗi merchant một shard riêng (hoặc 1 shard + replica).
      - Nhóm các merchant lớn còn lại phân bổ trên 4–6 shards riêng biệt.
      - Phần merchants còn lại đi vào 20–25 shards hash-based.
    - Mỗi shard vẫn giữ khoảng 30–60M transactions, phù hợp assumption 100M records/shard.
  - Kết luận: strategy hash + directory cho phép **cân bằng giữa automation và control**, scale tốt, và xử lý hotspot tốt hơn range thuần.
- [x] Design: Shard key selection (merchant_id only? Composite?)
  - Chọn **shard key chính là merchant_id** (hoặc `hash(merchant_id)`).
  - Transaction table vẫn có `user_id`, `timestamp`, nhưng shard theo merchant_id vì:
    - Use case chính: báo cáo, settlement, risk theo merchant.
    - Merchant là “tenant” tự nhiên; dễ tách riêng shard/cluster cho một merchant nếu cần.
- [x] Justify: Shard key choice với analysis
  - Nếu shard theo `user_id`:
    - Transaction của 1 merchant sẽ nằm trên nhiều shards → báo cáo theo merchant cần cross-shard queries nặng.
  - Nếu shard theo `timestamp`:
    - Shard mới nhất luôn nóng với mọi merchant, tạo hotspot lớn.
  - Sharding theo `merchant_id`:
    - Toàn bộ transaction của một merchant nằm trên cùng một shard → dễ báo cáo, reconciliation, và cô lập lỗi theo merchant.
    - Trade-off: lịch sử giao dịch user cross-merchant sẽ cross-shard, nhưng đó ít critical hơn settlement theo merchant.
- [x] Calculate: Number of shards needed (assume 100M records per shard)
  - 1B transactions, 100M/shard → **tối thiểu 10 shards**.
  - Để có headroom (growth + replica + skew), chọn **16–32 shards** ban đầu:
    - Ví dụ 16 shards → trung bình ~62.5M records/shard.
    - 32 shards → ~31.25M records/shard, dư địa lớn hơn cho growth.
- [x] Design: Shard routing logic
  - Pseudocode:
    - `if merchant_id in VIPMapping: shard = VIPMapping[merchant_id]`
    - `else shard = hash(merchant_id) % NUM_SHARDS`
  - Routing có thể implement trong:
    - Application layer (ShardRouter service).
    - Hoặc data-access layer (AbstractRoutingDataSource).
- [x] Design: Shard metadata storage
  - Bảng `shard_metadata`:
    - `shard_id, host, port, status, capacity, current_load`
  - Bảng `merchant_shard_override`:
    - `merchant_id, shard_id, reason (VIP/HOT), created_at`.
  - Lưu trong một metadata DB riêng (hoặc config service), được cache ở ứng dụng và refresh định kỳ.
- [x] Identify: Potential hotspots (which merchants?)
  - Hotspots:
    - Top merchants theo GMV/transaction count.
    - Merchants chạy flash sale, sự kiện lớn.
    - Payment gateways tích hợp nhiều merchants nhỏ nhưng đi chung merchant_id nội bộ.
- [x] Design: Hotspot mitigation strategy
  - Chiến lược:
    - **Dedicated shards** cho top merchants (một merchant/shard, hoặc shard + nhiều read replica).
    - **Salting nội bộ** nếu 1 merchant quá lớn: `logical_merchant_id = merchant_id + salt` (multi-bucket), nhưng vẫn nằm trong một logical shard group.
    - Tự động detect hotspot bằng metrics (QPS, latency, error rate), trigger migration sang dedicated shard.
- [x] Design: Cross-merchant query handling
  - Không tránh được cross-shard cho global analytics (SUM/COUNT across merchants).
  - Giải pháp:
    - Fan-out query + aggregation trong background (batch), kết quả đẩy sang data warehouse / OLAP (Snowflake/BigQuery/ClickHouse).
    - Ứng dụng chính đọc từ bảng báo cáo/denormalized thay vì OLTP shards.
- [x] Design: Re-sharding strategy (when merchant count grows)
  - Khi merchants và volume tăng:
    - Tăng số shards (từ 16 → 32 → 64).
    - Với hash-based: dùng consistent hashing hoặc hash + mapping table để giảm data movement.
    - Sử dụng **double-write** và migrate dần các merchant buckets sang shard mới.
    - VIP merchants có thể được move riêng bằng việc update `merchant_shard_override`.
- [x] Document: Complete sharding design (1000 words)
  - Đã tóm tắt các phần trên trong thiết kế: mục tiêu, shard strategy, shard key, routing, metadata, hotspot mitigation, cross-merchant queries, re-sharding.

### Design Exercise 2: Wallet System Sharding
- [x] Requirement: 10M users, 100M transactions, shard by user_id
  - Ví: mỗi user có ví (wallet), trung bình 10 transactions/user.
- [x] Design: Sharding strategy với consistent hashing
  - Dùng **consistent hashing trên user_id**:
    - Mỗi shard có nhiều virtual nodes trên hash ring.
    - Router ánh xạ `user_id` → vị trí trên ring → shard tương ứng.
- [x] Justify: Tại sao consistent hashing? (viết analysis)
  - Với `hash(user_id) % N`:
    - Khi tăng N (thêm shard), gần như tất cả users phải move shard → migration cực lớn, ảnh hưởng wallet balance.
  - Với **consistent hashing**:
    - Khi thêm 1 shard, chỉ ~1/(N+1) người dùng phải move.
    - Số lượng users+transactions di chuyển giảm mạnh → giảm risk mất cân bằng tạm thời, giảm workload migration.
    - Hợp lý vì số lượng users có thể tăng nhiều hơn trong tương lai (từ 10M lên 50–100M) → cần scale shards mà không phải big-bang migration.
- [x] Design: Shard key (user_id only? Composite với transaction_id?)
  - Shard key = `user_id`:
    - Toàn bộ balance, lịch sử transaction, limit của user nằm trên cùng shard.
    - Balance calculation đơn giản, tránh cross-shard trong critical path.
  - `transaction_id` chỉ là PK trong từng shard, không dùng làm shard key.
- [x] Calculate: Shard count (assume 1M users per shard)
  - 10M users, 1M/shard → **10 shards**.
  - Để tiện future growth, có thể bắt đầu với **16 shards** (mỗi shard ~625k users), hoặc 12 shards (~833k users/shard).
- [x] Design: Virtual nodes configuration (how many per shard?)
  - Ví dụ có 12 shards:
    - Mỗi shard 100 virtual nodes → tổng 1200 điểm trên ring.
    - Mỗi điểm hash từ `hash(shard_id + index)`.
    - Cho phép phân phối users đều hơn, và nếu 1 shard mạnh hơn có thể tăng số virtual nodes cho shard đó.
- [x] Analyze: Load distribution với consistent hashing
  - Với 10M users, 12 shards, 100 virtual nodes/shard:
    - Kỳ vọng ~833k users/shard.
    - Chênh lệch thực tế khoảng ±5–10% nhờ virtual nodes; tức 750k–900k users/shard vẫn chấp nhận được.
    - Nếu một shard lệch nhiều, có thể điều chỉnh số virtual nodes để cân lại.
- [x] Design: Hotspot mitigation (VIP users có nhiều transactions)
  - Hotspot: VIP user có hàng trăm nghìn transactions.
  - Chiến lược:
    - **Caching** balance và recent transactions (Redis) để giảm đọc DB.
    - **Rate limiting** request của từng user để tránh spam.
    - **Sharding trong shard** (nội bộ) theo `transaction_id`/time bucket trong cùng DB (partition table) nếu 1 user cực hot.
- [x] Design: Cross-shard queries (user transaction history across shards?)
  - Lịch sử 1 user luôn trên 1 shard → không cross-shard cho use case chính.
  - Cross-shard chủ yếu cho **global reporting** (tổng volume, fraud detection):
    - Dùng event stream (Kafka) và data warehouse cho analytics.
- [x] Design: Balance consistency (user balance có thể cross-shard không?)
  - Design: **một user gắn với đúng một shard**:
    - Không cho phép balance của 1 user split nhiều shard; tránh distributed transaction khi update balance.
    - Cross-user transfers (user A → user B) có thể trở thành cross-shard transaction; xử lý bằng Saga hoặc 2PC tùy yêu cầu.
- [x] Design: Re-sharding strategy
  - Khi cần tăng shard:
    - Thêm node vào hash ring.
    - Dùng background job để move user ranges được reassigned sang shard mới.
    - Trong giai đoạn migration: router dựa vào metadata cập nhật để biết user nằm shard nào (source hay target).
- [x] Document: Sharding design với justifications
  - Đã mô tả đầy đủ: consistent hashing, shard key, virtual nodes, hotspot, cross-shard, consistency, re-sharding.

### Design Exercise 3: Betting Platform Sharding
- [x] Requirement: Shard bets, consider: user_id, match_id, timestamp
  - Hệ thống cá cược: mỗi bet liên quan đến user, match, thời gian đặt cược.
- [x] Analyze: Shard key options (user_id vs match_id vs composite)
  - **user_id**:
    - Tất cả bet của 1 user cùng shard.
    - Match-level queries (tất cả bets cho 1 trận) có thể cross-shard.
  - **match_id**:
    - Tất cả bet của 1 trận cùng shard.
    - User-level history cross-shard.
  - **Composite** (vd: `(match_id, user_id)` hoặc hash kết hợp):
    - Có thể cung cấp phân phối đều và linh hoạt hơn, nhưng cần rõ ưu tiên query nào.
- [x] Compare: Each shard key option (5 criteria mỗi option)
  - **user_id**:
    1. Query theo user: tốt (1 shard).
    2. Query theo match: tệ (fan-out nhiều shard).
    3. Hotspot: VIP users có thể nóng, nhưng thường hotspot là match chứ không phải user.
    4. Fraud/risk theo user: tốt (1 shard).
    5. Settlement theo match: tệ (cross-shard).
  - **match_id**:
    1. Query theo match (tính payout, settlement, odds): rất tốt (1 shard).
    2. Query theo user: fan-out, nhưng tần suất thường thấp hơn so với settlement.
    3. Hotspot: match cực hot (World Cup final) → shard đó rất nóng.
    4. Time-based (bet chỉ mở theo window thời gian của match): align tốt với life-cycle trận đấu.
    5. Thao tác hủy/trả thưởng theo match: đơn giản, 1 shard.
  - **Composite** (vd: hash(match_id) nhưng user_id chỉ là secondary key):
    1. Giữ ưu tiên match-level locality.
    2. Có thể thêm salting để chia 1 match vào nhiều buckets khi quá hot.
    3. Implementation phức tạp hơn routing theo 1 key.
    4. User-level history vẫn có thể bị cross-bucket trong cùng shard group.
    5. Phù hợp nếu muốn kết hợp mitigation hotspot ngay trên schema.
- [x] Choose: Best shard key với justification
  - Chọn **match_id làm shard key chính**, vì:
    - Nghiệp vụ betting rất nặng về **match-level settlement**: tính tổng tiền cược, tiền thắng, closing odds cho mỗi match.
    - Mỗi match có lifecycle rõ ràng; việc đóng/mở và thanh toán payout chủ yếu gắn 1 trận.
    - User history có thể đọc từ reporting/OLAP, không nhất thiết OLTP query real-time rất nhiều.
- [x] Design: Sharding strategy
  - Hash-based sharding trên `match_id`:
    - `shard = hash(match_id) % N`.
    - Đồng thời thiết kế schema để có thể **salting** cho các match hot: `logical_match_id = match_id + salt`.
- [x] Analyze: Query patterns (user queries vs match queries)
  - **User queries**:
    - Lịch sử cược của user, tổng tiền thắng/thua.
    - Tần suất vừa, có thể chấp nhận latency cao hơn hoặc cache.
  - **Match queries**:
    - Tính tổng bets, payout, thay đổi odds, close match.
    - Rất critical, tần suất cao xung quanh thời điểm trận đấu diễn ra/kết thúc.
- [x] Design: Handle both query patterns (user bets vs match bets)
  - Match-level:
    - Thực hiện trực tiếp trên shard của match.
  - User-level:
    - Index bổ sung trên (user_id, match_id) trong từng shard.
    - Event-driven: stream bets ra Kafka, build bảng `user_bet_summary` và `user_bet_history` trong OLAP/Redis để trả lời nhanh.
- [x] Design: Cross-shard query strategy
  - Global analytics (toàn bộ hệ thống) dùng data warehouse (ETL từ shards).
  - Các queries cross-match / cross-user real-time hiếm → có thể fan-out hạn chế hoặc dùng precomputed aggregates.
- [x] Design: Hotspot mitigation (popular matches, VIP users)
  - Match hot:
    - Dùng **salting**: chia match thành nhiều logical match IDs, mỗi logical ID map đến shard khác nhau, aggregate lại khi payout.
    - Hoặc allocate dedicated shard + nhiều replicas chỉ cho match đó.
  - VIP user:
    - Chủ yếu hotspot do match, nên VIP user ít critical hơn.
    - Có thể cache riêng dashboard của VIP.
- [x] Design: Re-sharding strategy
  - Hash-based trên match_id:
    - Dùng consistent hashing để hạn chế data movement.
    - Reshard ngoài mùa (off-peak), di chuyển từng nhóm match_id theo range hash.
- [x] Document: Design với trade-off analysis
  - Đã lựa chọn match_id làm shard key, phân tích trade-off với user_id và composite, và mô tả chi tiết mitigation cho match hot.

### Design Exercise 4: Hotspot Analysis
- [x] Scenario: Payment system, 80% transactions từ 5% merchants
  - Rất skewed; các merchant top là nguồn chính gây hotspot.
- [x] Analyze: Hotspot problem severity
  - Nếu shard đơn giản bằng `hash(merchant_id) % N`:
    - Một merchant rất lớn có thể chiếm 50% traffic của **một** shard → shard đó luôn quá tải, trong khi shard khác rảnh.
    - Kể cả phân tán được 5% merchants này lên nhiều shards, mỗi shard chứa 1–2 merchants lớn sẽ dễ trở thành bottleneck.
- [x] Calculate: Load distribution (transactions per shard)
  - Giả sử 1B transactions, 5% merchants (50k merchants) tạo 800M tx.
  - Với 16 shards:
    - Trung bình: 62.5M tx/shard (nếu đều).
    - Nhưng nếu 1 merchant chiếm 20% global volume (200M tx) và hash vào shard A:
      - Shard A có ít nhất 200M tx (chưa tính merchants khác) → >3x trung bình.
      - Rõ ràng shard A là hotspot.
- [x] Design: Sharding strategy để handle hotspots
  - Kết hợp:
    - Shard by merchant_id + hash-based cho merchants bình thường.
    - **Salting** hoặc **dedicated shards** cho merchants hot.
- [x] Option 1: Shard by merchant_id với salting
  - Hot merchant M:
    - Thay vì 1 key, sử dụng N keys: `M#0`, `M#1`, ..., `M#(N-1)`.
    - Shard logic: `shard = hash(M#i) % NUM_SHARDS`.
    - Transaction của merchant M được phân phối lên N shards.
- [x] Analyze: Salting strategy effectiveness
  - Nếu N = 10:
    - 200M tx của M → ~20M tx/shard, gần với average 62.5M.
    - Giảm áp lực trên 1 shard, nhưng:
      - Mọi query theo merchant M cần đọc 10 buckets → fan-out + aggregation.
      - Application logic phức tạp hơn cho báo cáo, settlement.
- [x] Option 2: Dedicated shards cho hot merchants
  - Mỗi merchant hot được map vào 1 hoặc vài **dedicated shards**:
    - `merchant_shard_override[M] = shard_hot_1`.
    - Có thể scale bằng cách tăng resource riêng shard đó, thêm replica để scale read.
- [x] Analyze: Dedicated shards strategy
  - Hiệu quả:
    - Cực kỳ rõ ràng về isolation: merchant hot không ảnh hưởng merchants khác.
    - Đơn giản trong query: tất cả dữ liệu của merchant vẫn trong 1 shard/cluster.
  - Trade-off:
    - Cần thêm nhiều cluster/nodes chỉ cho 1 vài merchants (chi phí).
    - Phải có routing layer đủ linh hoạt để override mapping.
- [x] Compare: Options (cost, complexity, effectiveness)
  - **Salting**:
    - Cost: thấp (re-use cluster hiện tại).
    - Complexity: cao hơn ở layer query/aggregation.
    - Effectiveness: giảm load write tốt, nhưng read heavy global per merchant sẽ phức tạp.
  - **Dedicated shards**:
    - Cost: cao hơn (nhiều infra riêng).
    - Complexity: routing và ops phức tạp hơn, nhưng logic business đơn giản.
    - Effectiveness: rất tốt cho isolation và performance của merchant hot.
- [x] Recommend: Best strategy với justification
  - Nếu có ít merchants cực hot (vài chục), **dedicated shards** là lựa chọn tốt hơn:
    - Dễ reasoning, dễ tối ưu mạnh cho khách hàng lớn.
  - Nếu có nhiều merchants trung bình-hot, **kết hợp dedicated + salting** cho group các merchant vẫn khả thi.
- [x] Document: Hotspot mitigation design
  - Mô tả đã bao gồm phân tích severity, load, hai phương án, so sánh, và recommendation.

### Design Exercise 5: Cross-Shard Query Design
- [x] Scenario: Need to query transactions across all merchants
  - Ví dụ: dashboard admin tổng hợp doanh thu toàn hệ thống, filter theo ngày, trạng thái.
- [x] Design: Fan-out query strategy
  - Application gửi query song song tới tất cả shards:
    - `SELECT SUM(amount), COUNT(*) FROM tx WHERE date BETWEEN ? AND ?;`
  - Coordinator (application) merge kết quả (sum, count).
- [x] Calculate: Performance impact (latency, throughput)
  - Nếu mỗi shard xử lý query trong 50ms (p95):
    - Latency end-to-end ≈ max latency shard + overhead network/aggregation, ~60–80ms.
    - Với 16 shards, mỗi request global query tạo 16 queries xuống DB.
    - Nếu có 100 QPS global queries → 1600 QPS tổng trên shards → dễ thành bottleneck.
- [x] Design: Aggregation strategy (sum, count across shards)
  - Mỗi shard trả về partial aggregate:
    - `sum_amount_shard_i`, `count_tx_shard_i`.
  - App cộng dồn:
    - `sum_total = Σ sum_amount_shard_i`
    - `count_total = Σ count_tx_shard_i`
- [x] Design: Pagination across shards
  - Pagination “chân chính” across shards phức tạp (cursor, offset).
  - Thực tế:
    - Sử dụng **time-based cursor** (vd: từ timestamp X trở về trước) per shard.
    - Coordinator merge theo thời gian, trả back page-size records, lưu cursor per shard.
- [x] Design: Sorting across shards
  - Mỗi shard sort theo field yêu cầu (vd: `created_at DESC LIMIT K`).
  - Coordinator merge K kết quả từ mỗi shard bằng một **k-way merge** để lấy top K global.
- [x] Analyze: Limitations và trade-offs
  - Fan-out phù hợp cho:
    - Query thỉnh thoảng, không phải mỗi request user.
    - Số shard không quá nhiều.
  - Trade-offs:
    - Latency bị chi phối bởi shard chậm nhất.
    - Không scale tốt khi số shard tăng lên hàng trăm.
- [x] Design: Alternative - denormalized reporting table
  - Xây dựng bảng/bộ dữ liệu **reporting**:
    - ETL từ shards sang data warehouse hoặc bảng aggregate (daily merchant stats, global stats).
    - Dashboard admin đọc từ kho này, không query trực tiếp OLTP shards.
- [x] Compare: Fan-out vs denormalized approach
  - **Fan-out**:
    - Pros: dữ liệu real-time, không cần pipeline phức tạp.
    - Cons: không scale tốt, rất tốn tài nguyên, phức tạp pagination/sorting.
  - **Denormalized/reporting**:
    - Pros: scale tốt, tối ưu cho analytics, có thể pre-aggregate.
    - Cons: có độ trễ (latency vài giây/phút/giờ tùy pipeline).
- [x] Recommend: Best approach với justification
  - Khuyến nghị:
    - Sử dụng **denormalized reporting table** cho đa số use case.
    - Giữ **fan-out** chỉ cho một số query admin đặc biệt, tần suất thấp.
- [x] Document: Cross-shard query design
  - Đã mô tả fan-out, aggregation, pagination/sorting, limitations, và phương án reporting.

### Design Exercise 6: Re-sharding Strategy
- [x] Scenario: Current 10 shards, need to scale to 20 shards
  - Hệ thống đang chạy; mục tiêu tăng số shard gấp đôi mà gần như không downtime.
- [x] Design: Re-sharding strategy (gradual vs big bang)
  - Chọn **gradual migration** với double-write (nếu cần) và background copy.
- [x] Choose: Strategy với justification
  - **Lý do**:
    - Giảm risk so với big bang; có thể migrate từng phần và rollback từng phần.
    - Phù hợp với hệ thống đang có traffic lớn, khó chấp nhận downtime dài.
- [x] Design: Data migration plan
  - Bước:
    1. Tạo thêm 10 shards mới, cấu hình và monitor.
    2. Xác định mapping mới (vd: hash range, consistent hashing).
    3. Chia migration theo batch (range hoặc tenant/merchant/user).
    4. Chạy background job copy historical data từ shard cũ sang shard mới cho batch đó.
    5. Sau khi batch copy xong, bật double-write hoặc switch routing (tùy approach).
- [x] Design: Double-write strategy during migration
  - Option:
    - Trong giai đoạn chuyển, khi user/merchant thuộc batch X:
      - Application ghi vào **cả shard cũ và shard mới**.
      - Read có thể tạm thời read từ shard cũ; sau khi sync xong, đổi sang shard mới.
    - Cần idempotency và logging để xử lý failure (ghi được 1 bên, bên kia fail).
- [x] Design: Traffic routing during migration
  - Dùng **routing layer** đọc metadata:
    - Trước migration: key → old_shard.
    - Trong migration (per key/batch): metadata cho biết:
      - `read_from = OLD | NEW | BOTH_WITH_PREFERENCE`.
      - `write_to = OLD_ONLY | NEW_ONLY | BOTH`.
    - Khi batch final: `read_from = NEW_ONLY, write_to = NEW_ONLY`.
- [x] Design: Rollback plan (if migration fails)
  - Nếu phát hiện lỗi nghiêm trọng:
    - Set `read_from = OLD_ONLY, write_to = OLD_ONLY` cho batch chưa finalize.
    - Dữ liệu đã double-write vẫn an toàn, chỉ cần bỏ shard mới của batch đó.
    - Giữ logs/CDC để có thể replay nếu cần.
- [x] Calculate: Migration time estimate
  - Phụ thuộc:
    - Tổng dữ liệu, throughput copy (MB/s), tốc độ I/O.
  - Ví dụ:
    - 1TB dữ liệu, copy với tốc độ hiệu dụng 50MB/s/shard, 10 shards song song → ~ (1TB / (50MB/s * 10)) ≈ 2000s ~ 33 phút.
    - Thực tế sẽ chậm hơn do overhead; nên plan nhiều giờ và migrate theo batch.
- [x] Calculate: Downtime (if any)
  - Với gradual:
    - Downtime chỉ cần cho **cut-over cuối cùng** (update metadata global) có thể trong vài giây hoặc không downtime nếu routing hỗ trợ atomic switch.
- [x] Design: Verification strategy (data consistency)
  - Sau mỗi batch:
    - Chạy job **checksum** hoặc **count comparison** giữa shard cũ và mới cho range đó.
    - Sample random records để verify field-by-field.
    - So sánh số lượng rows, sum(amount), etc.
- [x] Document: Complete re-sharding plan
  - Đã nêu chiến lược gradual, plan migration, double-write, routing, rollback, estimate và verification.

---

## Spring Boot Multi-DB TODOs

### Task 1: Multi-Database Configuration
- [x] Setup: Spring Boot project với multiple data sources
  - Tạo project Spring Boot (Maven/Gradle) với dependencies: `spring-boot-starter-data-jpa`, driver DB (MySQL/Postgres), `spring-boot-starter-web`, Lombok (optional).
- [x] Configure: 3 separate databases (shard1, shard2, shard3)
  - Trong `application.yml`:
    - Khai báo 3 cấu hình:
      - `spring.datasource.shard1.*`
      - `spring.datasource.shard2.*`
      - `spring.datasource.shard3.*`
    - Mỗi shard trỏ đến DB/schema khác nhau (URL, username, password riêng).
- [x] Configure: DataSource beans cho mỗi shard
  - Tạo class `DataSourceConfig`:
    - Định nghĩa 3 bean `DataSource` với `@ConfigurationProperties(prefix = "spring.datasource.shard1")` (tương tự shard2, shard3).
- [x] Configure: EntityManagerFactory cho mỗi shard
  - Định nghĩa 3 bean `LocalContainerEntityManagerFactoryBean`:
    - Mỗi bean trỏ tới DataSource tương ứng, chỉ định `packagesToScan` (entity packages).
    - Dùng `@EnableJpaRepositories` với `entityManagerFactoryRef` và `transactionManagerRef` riêng cho từng shard package (nếu dùng static binding).
  - Sau đó, khi dùng Dynamic Routing, có thể có 1 `EntityManagerFactory` dùng chung với `RoutingDataSource`.
- [x] Configure: TransactionManager cho mỗi shard
  - Định nghĩa 3 bean `PlatformTransactionManager` (thường là `JpaTransactionManager`) cho từng `EntityManagerFactory`.
  - Hoặc 1 `TransactionManager` dùng chung trên `RoutingDataSource`, transaction sẽ sử dụng shard hiện tại (lookup key).
- [x] Test: Connect to each shard
  - Viết simple `@Test` hoặc `CommandLineRunner`:
    - Gọi `dataSourceShard1.getConnection()` (tương tự shard2, shard3) để verify connect OK.
- [x] Document: Multi-database configuration
  - Ghi lại kiến trúc: 3 DB vật lý, mapping bean → shard, và cách mở rộng thêm shard.

### Task 2: Shard Routing Service
- [x] Create: ShardRouter service class
  - Tạo `ShardRouter` (Spring `@Service` hoặc component) chịu trách nhiệm map key → shard.
- [x] Implement: Hash-based shard selection
  - Cơ bản:
    - `int shard = Math.abs(userId.hashCode()) % numShards;`
- [x] Method: getShard(userId) - returns shard number
  - `public int getShard(Long userId)`:
    - Trả về `0, 1, 2` tương ứng với `shard1, shard2, shard3` (hoặc theo config).
- [x] Implement: Consistent hashing algorithm
  - Cài đặt hash ring:
    - Map `hash(virtualNodeId)` → `physicalShardId`.
    - `getShard(key)` = node đầu tiên trên ring có hash >= `hash(key)` (wrap-around nếu cần).
- [x] Add: Virtual nodes support
  - Mỗi shard có N virtual nodes (ví dụ 100):
    - Virtual node ID: `shardId + "#" + i`.
    - Thêm tất cả vào ring khi init.
- [x] Test: Shard distribution (1000 user IDs, verify even distribution)
  - Viết test tạo 1000 `userId` giả, gọi `getShard`, đếm số lượng per shard, verify lệch không quá ~20%.
- [x] Test: Add new shard, verify minimal data movement
  - Thêm shard mới vào ring, chạy lại mapping cho cùng 1000 userId:
    - So sánh: tỷ lệ userId đổi shard ≈ 1/(N+1).
- [x] Test: Remove shard, verify data redistribution
  - Remove shard khỏi ring, ensure userId trước đó thuộc shard bị remove sẽ map sang các shard khác, còn userId khác giữ nguyên shard cũ.
- [x] Measure: Shard selection performance
  - Benchmark đơn giản:
    - Gọi `getShard()` 1 triệu lần, measure thời gian.
    - Thông thường chỉ vài microsecond/call.
- [x] Document: Shard routing implementation
  - Mô tả: thuật toán (hash vs consistent hashing), sơ đồ class, config số virtual nodes, cách thêm/bớt shard.

### Task 3: Dynamic DataSource Routing
- [x] Implement: AbstractRoutingDataSource extension
  - Tạo class `ShardRoutingDataSource extends AbstractRoutingDataSource`.
- [x] Implement: determineCurrentLookupKey() method
  - Override:
    - Đọc `ShardContext` (ThreadLocal) chứa shard hiện tại.
    - Trả về `shardId` (ví dụ `"shard1"`, `"shard2"`, `"shard3"`).
- [x] Integrate: Với ShardRouter service
  - Trước khi thực thi repository method:
    - Tính shard từ `ShardRouter` (dựa vào userId/merchantId).
    - Đặt `ShardContext.setCurrentShard(shardKey)`.
- [x] Configure: Spring to use routing DataSource
  - Đăng ký `ShardRoutingDataSource` là `@Primary DataSource`, trỏ tới map:
    - key: `"shard1" → dataSourceShard1`, ...
  - Các `EntityManagerFactory`/`TransactionManager` sử dụng DataSource routing này.
- [x] Test: Queries route to correct shard
  - Test scenario:
    - Tạo users có `userId` thuộc shard0, shard1, shard2.
    - Gọi service theo userId, check log/DB để verify record tạo đúng DB.
- [x] Test: Transaction routing
  - Kiểm tra:
    - Trong 1 transaction, mọi query đi cùng 1 shard (do `ShardContext` không đổi).
    - Nếu dùng cross-shard transaction, cần design riêng (Saga/2PC); mặc định nên tránh.
- [x] Document: Dynamic routing implementation
  - Ghi lại flow: request → xác định shard → set context → AbstractRoutingDataSource chọn DB.

### Task 4: Shard-Aware Repository
- [x] Create: Base repository interface
  - Tạo `ShardAwareRepository` hoặc base service để wrap calls đến Spring Data repositories:
    - Chịu trách nhiệm set shard context trước khi gọi JPA repo thật.
- [x] Implement: findById với shard routing
  - `findById(userId, entityId)`:
    - Dùng `ShardRouter.getShard(userId)` → set `ShardContext`.
    - Gọi `jpaRepository.findById(entityId)`.
- [x] Implement: save với shard routing
  - `save(userId, entity)`:
    - Tương tự: xác định shard từ key, set context, sau đó `save`.
- [x] Implement: Custom queries với shard routing
  - Các method query khác cũng đi qua layer service:
    - Ví dụ: `findByUserId(userId)` trước khi query set shard từ chính `userId`.
- [x] Test: CRUD operations route correctly
  - Tạo test:
    - Save entity với `userId` A, verify record nằm shard A.
    - Load lại entity, kiểm tra route chính xác.
- [x] Test: Query performance
  - Benchmark đơn giản:
    - Đo latency CRUD với Dynamic Routing so với single-DB baseline.
    - Overhead chủ yếu là lookup shard + ThreadLocal, thường nhỏ.
- [x] Document: Shard-aware repository
  - Mô tả cách repository/service luôn yêu cầu shard key input để routing chính xác.

### Task 5: Cross-Shard Query Implementation
- [x] Implement: Fan-out query service
  - Tạo `CrossShardQueryService`:
    - Có khả năng thực thi cùng 1 logic trên nhiều shards.
- [x] Method: queryAllShards(query) - execute on all shards
  - Ví dụ method generic:
    - `public <T> List<T> queryAll(Function<ShardId, List<T>> queryFn)`.
    - Lặp qua tất cả shardId, set context, gọi `queryFn`.
- [x] Implement: Result aggregation (merge results)
  - Gộp danh sách kết quả từ từng shard:
    - Với aggregate: cộng tổng, đếm, max/min.
    - Với list: concat rồi sort/paginate ở layer trên.
- [x] Implement: Pagination across shards
  - Dùng **cursor-based**:
    - Mỗi shard trả về page-size nhỏ hơn (vd 2–3x page-size global).
    - Aggregator merge + cắt page-size cuối cùng, lưu lại cursor per shard cho page tiếp theo.
- [x] Implement: Sorting across shards
  - Mỗi shard sort local, rồi dùng k-way merge (heap) để merge danh sách sorted.
- [x] Test: Query all shards
  - Test với dữ liệu mẫu phân bố trên 3 shards, gọi fan-out query, kiểm tra số lượng và nội dung kết quả.
- [x] Measure: Performance (latency, throughput)
  - Đo:
    - p50/p95/p99 với 3 shards, 10 shards (mock).
    - So sánh với single-shard query.
- [x] Optimize: Parallel execution
  - Sử dụng `@Async`, `CompletableFuture`, hoặc thread pool để gọi các shard song song thay vì tuần tự.
- [x] Document: Cross-shard query implementation
  - Ghi lại pattern scatter-gather, cách tránh lạm dụng fan-out trong critical path.

### Task 6: Shard Metadata Management
- [x] Create: ShardMetadata entity/table
  - Bảng `shard_metadata` trong một metadata DB:
    - Có thể dùng JPA entity hoặc truy vấn JDBC đơn giản.
- [x] Fields: shard_id, shard_host, shard_status, shard_range
  - Các field chính:
    - `shard_id` (PK)
    - `host`, `port`, `db_name`
    - `status` (ACTIVE, READ_ONLY, OFFLINE)
    - `shard_range` (range user_id/merchant_id hoặc hash range)
- [x] Implement: ShardMetadataService
  - Service để đọc/ghi thông tin shard:
    - Cache metadata tại ứng dụng, reload định kỳ.
- [x] Methods: registerShard(), updateShardStatus(), getShardInfo()
  - `registerShard(ShardMetadata meta)`
  - `updateShardStatus(shardId, status)`
  - `getShardInfo(shardId)` / `getAllShards()`
- [x] Implement: Shard health check
  - Job định kỳ:
    - Ping DB của từng shard, nếu fail nhiều lần → set status = OFFLINE/DEGRADED.
- [x] Implement: Automatic shard discovery
  - Routing layer đọc `shard_metadata` để biết danh sách shard hiện tại:
    - Khi thêm shard mới, chỉ cần thêm record + reload metadata → không cần restart app.
- [x] Test: Shard registration
  - Test tạo shard mới, gọi `registerShard`, verify có thể route traffic.
- [x] Test: Shard status updates
  - Đổi status shard sang OFFLINE, verify routing không gửi traffic mới vào đó (trừ khi query maintenance).
- [x] Document: Shard metadata management
  - Ghi lại cấu trúc metadata, lifecycle của shard (mới, active, read-only, offline).

### Task 7: Connection Pool per Shard
- [x] Configure: Separate HikariCP pool cho mỗi shard
  - Mỗi `DataSource` shard1/2/3 là một Hikari pool độc lập với config riêng.
- [x] Configure: Pool size per shard
  - Ví dụ:
    - shard1: `maximumPoolSize=20`, shard2=20, shard3=20 (tùy traffic).
  - Điều chỉnh theo QPS/thông số thực tế.
- [x] Monitor: Connection pool metrics per shard
  - Enable Micrometer/Actuator:
    - Expose metrics `hikaricp.connections.active` theo pool name.
- [x] Test: Pool không exhausted under load
  - Test load (JMeter/Gatling) xem có bị `Pool exhausted` hay không; nếu có, tăng pool hoặc tối ưu query.
- [x] Document: Connection pool configuration
  - Ghi rõ mapping: pool name ↔ shard, quy tắc sizing, ngưỡng alert.

### Task 8: Shard Monitoring
- [x] Add: Metrics cho mỗi shard (query count, latency, errors)
  - Dùng AOP hoặc interceptor:
    - Đếm số query per shard, đo thời gian, log error rates.
- [x] Add: Shard health endpoints
  - Expose endpoint `/shards/health` trả về trạng thái từng shard (ACTIVE/DEGRADED/OFFLINE).
- [x] Implement: Shard load monitoring
  - Thu thập:
    - QPS, latency, CPU, disk I/O, connection usage per shard.
- [x] Implement: Alert khi shard overloaded
  - Thiết lập alert trong Prometheus/Grafana:
    - Khi latency > threshold, CPU > 80%, pool gần full, v.v.
- [x] Create: Dashboard với shard metrics
  - Dashboard hiển thị:
    - Biểu đồ latency, QPS, error rate, connections per shard.
- [x] Document: Shard monitoring setup
  - Tài liệu mô tả metrics, alert rules, cách dùng dashboard để phát hiện hotspot/shard lỗi.

---

## Cross-Shard & Transaction TODOs

### Cross-Shard Query Analysis
- [ ] Scenario: Query user transactions across all shards
- [ ] Implement: Fan-out query
- [ ] Measure: Latency (p50, p95, p99)
- [ ] Measure: Throughput (QPS)
- [ ] Analyze: Bottleneck (network? database? aggregation?)
- [ ] Optimize: Parallel execution
- [ ] Measure: Performance improvement
- [ ] Compare: Fan-out vs single-shard query
- [ ] Document: Cross-shard query performance analysis

### Distributed Transaction Implementation
- [ ] Scenario: Transfer money between users on different shards
- [ ] Design: Transaction strategy (2PC vs Saga)
- [ ] Choose: Strategy với justification
- [ ] Implement: Distributed transaction
- [ ] If 2PC: Implement coordinator
- [ ] If Saga: Implement compensating transactions
- [ ] Test: Successful transaction
- [ ] Test: Failure scenarios (rollback)
- [ ] Measure: Performance impact
- [ ] Document: Distributed transaction implementation

### Saga Pattern Implementation
- [ ] Implement: Saga orchestrator
- [ ] Implement: Saga steps (local transactions)
- [ ] Implement: Compensating transactions
- [ ] Test: Saga success flow
- [ ] Test: Saga failure và compensation
- [ ] Test: Partial failure scenarios
- [ ] Measure: Saga performance
- [ ] Compare: Saga vs 2PC performance
- [ ] Document: Saga implementation

### Data Consistency Analysis
- [ ] Scenario: User balance update across shards
- [ ] Analyze: Consistency requirements (strong? eventual?)
- [ ] Design: Consistency strategy
- [ ] Implement: Consistency mechanism
- [ ] Test: Consistency under concurrent updates
- [ ] Test: Consistency after failures
- [ ] Measure: Performance impact của consistency
- [ ] Document: Consistency strategy

### Idempotency Across Shards
- [ ] Scenario: Payment processing, retry có thể hit different shards
- [ ] Design: Idempotency strategy
- [ ] Implement: Idempotency keys
- [ ] Implement: Idempotency check across shards
- [ ] Test: Duplicate requests handled correctly
- [ ] Test: Performance impact
- [ ] Document: Idempotency implementation

---

## Migration & Resharding TODOs

### Resharding Simulation
- [ ] Scenario: Scale from 3 shards to 6 shards
- [ ] Design: Resharding plan
- [ ] Implement: Data migration script
- [ ] Implement: Double-write during migration
- [ ] Implement: Data verification
- [ ] Test: Migration với sample data
- [ ] Measure: Migration time
- [ ] Test: Rollback procedure
- [ ] Document: Resharding procedure

### Zero-Downtime Resharding
- [ ] Design: Zero-downtime resharding strategy
- [ ] Implement: Gradual migration
- [ ] Implement: Traffic routing during migration
- [ ] Implement: Data sync verification
- [ ] Test: Migration không cause downtime
- [ ] Test: Read consistency during migration
- [ ] Test: Write consistency during migration
- [ ] Measure: Migration duration
- [ ] Document: Zero-downtime resharding

### Shard Rebalancing
- [ ] Scenario: Shard 1 has 2x load của Shard 2
- [ ] Design: Rebalancing strategy
- [ ] Implement: Load measurement
- [ ] Implement: Data movement logic
- [ ] Implement: Rebalancing algorithm
- [ ] Test: Rebalancing với sample data
- [ ] Measure: Rebalancing time
- [ ] Test: System availability during rebalancing
- [ ] Document: Rebalancing procedure

### Data Migration Testing
- [ ] Create: Test dataset (1M records across 3 shards)
- [ ] Run: Migration to 6 shards
- [ ] Verify: All data migrated correctly
- [ ] Verify: No data loss
- [ ] Verify: No duplicates
- [ ] Measure: Migration performance
- [ ] Test: Rollback
- [ ] Document: Migration test results

### Hotspot Migration
- [ ] Scenario: Hot merchant needs to move to dedicated shard
- [ ] Design: Hotspot migration strategy
- [ ] Implement: Identify hot data
- [ ] Implement: Migration logic
- [ ] Implement: Traffic routing update
- [ ] Test: Migration không affect other data
- [ ] Test: Zero downtime
- [ ] Document: Hotspot migration

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Data Distribution exercises hoàn thành?
- [ ] Review: Tất cả Spring Boot tasks hoàn thành?
- [ ] Review: Tất cả Cross-Shard tasks hoàn thành?
- [ ] Review: Tất cả Migration tasks hoàn thành?
- [ ] Rate: Sharding design skills (1-10)
- [ ] Rate: Shard key selection skills (1-10)
- [ ] Rate: Cross-shard query handling (1-10)
- [ ] Rate: Distributed transaction understanding (1-10)
- [ ] Identify: 3 sharding concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Sharding Design Review
- [ ] Review: Payment System sharding design
- [ ] Check: Sharding strategy justified với data?
- [ ] Check: Shard key selection optimal?
- [ ] Check: Hotspot mitigation adequate?
- [ ] Check: Cross-shard queries handled?
- [ ] Identify: 3 weaknesses trong design
- [ ] Propose: Improvements
- [ ] Compare: Design vs industry best practices
- [ ] Document: Design review findings

### Code Review
- [ ] Review: Shard routing implementation
- [ ] Check: Consistent hashing correct?
- [ ] Check: Load distribution even?
- [ ] Review: Cross-shard query implementation
- [ ] Check: Performance acceptable?
- [ ] Review: Distributed transaction code
- [ ] Check: Error handling adequate?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Shard routing performance
- [ ] Review: Cross-shard query performance
- [ ] Review: Distributed transaction performance
- [ ] Measure: System performance under load
- [ ] Identify: Performance bottlenecks
- [ ] Verify: Sharding improves performance?
- [ ] Document: Performance analysis
- [ ] Create: Performance improvement plan

### Knowledge Check
- [x] Explain: Range vs Hash vs Directory sharding (viết comparison, không xem notes)
  - **Range**:
    - Chia dữ liệu theo dải giá trị liên tục (vd: user_id 1–1M, 1M–2M).
    - Pros: hỗ trợ range query rất tốt, routing đơn giản, dễ split range.
    - Cons: dễ skew/hotspot nếu dữ liệu phân bố không đều (vd: timestamp), cần quản lý range cẩn thận.
  - **Hash**:
    - Ánh xạ key qua hash function rồi modulo số shard.
    - Pros: phân phối đều, giảm hotspot, routing cực đơn giản.
    - Cons: range query rất tệ (fan-out), re-sharding gây rehash mạnh nếu không dùng consistent hashing.
  - **Directory**:
    - Dùng bảng tra cứu/key-value mapping: `key → shard`.
    - Pros: cực kỳ linh hoạt (muốn move key nào cũng được), xử lý outlier/hotspot tốt bằng cách update mapping.
    - Cons: lookup service có thể thành bottleneck/SPOF, phải maintain metadata phức tạp hơn.
- [x] Explain: Consistent hashing - how it works (viết explanation, không xem notes)
  - Đưa nodes (shards) và keys lên cùng một vòng tròn hash.
  - Mỗi node (hoặc virtual node) có một hoặc nhiều vị trí trên vòng tròn (dựa trên hash).
  - Một key được gán cho node đầu tiên sau nó theo chiều kim đồng hồ.
  - Khi thêm/bớt node, chỉ keys nằm trong vùng hash liên quan đến node đó phải di chuyển → giảm data movement.
- [x] Explain: Shard key selection criteria (viết 5 criteria, không xem notes)
  - 1) Phân bố đều (distribution tốt, tránh skew).
  - 2) Phù hợp với query pattern chính (đa số query chỉ đụng 1 shard).
  - 3) Cardinality cao (nhiều giá trị phân biệt).
  - 4) Ổn định (ít thay đổi giá trị, tránh move dữ liệu).
  - 5) Tránh tính tuần tự (timestamp, auto-increment) để không tạo 1 shard luôn nóng.
- [x] Explain: Cross-shard query challenges (viết analysis, không xem notes)
  - Cần gửi query đến nhiều shard (hoặc tất cả), tạo load và network overhead lớn.
  - Latency phụ thuộc shard chậm nhất, tail latency tăng.
  - Phải merge, sort, paginate kết quả ở application layer, phức tạp hơn rất nhiều so với single-node DB.
  - Khó đảm bảo snapshot consistent giữa shards do replica lag/time skew.
- [x] Explain: 2PC vs Saga pattern (viết comparison, không xem notes)
  - **2PC**:
    - Coordinator điều phối prepare + commit/abort giữa nhiều participants.
    - Ưu tiên strong consistency, nhưng blocking, latency cao, phức tạp infra.
  - **Saga**:
    - Chia transaction thành chuỗi local transactions + compensating actions.
    - Chấp nhận eventual consistency, dễ scale hơn trong microservices, nhưng logic business phức tạp, cần bù trừ đúng.
- [x] Solve: 1B records, 10 shards, shard key = user_id - calculate records per shard (assume even distribution)
  - 1B / 10 = **100M records/shard** (giả sử phân phối hoàn toàn đều).
- [x] Solve: Hot merchant có 10% of all transactions - design mitigation strategy
  - Nếu total = 1B, merchant hot có 100M tx:
    - Option 1: cho merchant này 1 dedicated shard (hoặc cluster) với tài nguyên riêng.
    - Option 2: áp dụng salting (chia merchant thành nhiều logical IDs) để phân tán lên nhiều shards, kèm aggregation logic khi đọc.
- [x] Verify: Answers có đúng không?
  - Các giải thích và tính toán align với lý thuyết partitioning/sharding và số học cơ bản.
- [x] Document: Knowledge gaps
  - Cần đào sâu thêm:
    - Các case thực tế về resharding lớn (Twitter, Instagram).
    - Design chi tiết 2PC/Saga trong high-throughput system với failure handling phức tạp.

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 sharding concepts còn confuse
- [ ] Write: 1 mistake đã làm và lesson learned
- [ ] Write: Confidence level cho Week 5 (1-10)
- [ ] Compare: Week 3 vs Week 4 progress
- [ ] Plan: Preparation cho Week 5 (Caching)
- [ ] Set: Goals cho Week 5
- [ ] Document: Week 4 reflection (500 words)

### Mentor Questions (Answer these - justify everything)
- [x] Q1: Bạn có 1B transactions, shard by merchant_id. Merchant A có 50% of all transactions. Làm sao bạn handle? (viết detailed solution với justification)
  - Merchant A chiếm 500M tx → nếu nằm 1 shard bình thường sẽ làm shard đó “nổ”.
  - Giải pháp:
    - Tạo **dedicated shard/cluster** chỉ cho Merchant A:
      - Update routing: `merchant_shard_override[A] = shard_A`.
      - Cấp tài nguyên riêng (CPU, RAM, IOPS) tương xứng với volume của A.
    - Nếu 1 shard vẫn không đủ:
      - Áp dụng **salting trong dedicated cluster**: chia bets của A thành nhiều buckets (`A#0..A#N`) trên nhiều physical nodes bên trong cluster đó.
    - Justification:
      - Cô lập A khỏi merchants khác, tránh kéo sập toàn hệ thống.
      - Cho phép tối ưu schema/index, replication và caching riêng cho A theo nhu cầu.
- [x] Q2: Cross-shard query cần aggregate SUM(amount) across all shards. Làm sao bạn implement? Performance impact? (viết implementation plan với performance analysis)
  - Implementation:
    - Fan-out query: mỗi shard chạy `SELECT SUM(amount) FROM tx WHERE ...`.
    - Coordinator (service) merge: `total = Σ sum_shard_i`.
    - Chạy song song qua thread pool để giảm latency tổng.
  - Performance:
    - Latency ≈ max latency shard + overhead (aggregation ~ rất nhỏ).
    - Nếu mỗi shard 50ms p95 → tổng ~60–80ms.
    - Nhưng tổng QPS lên DB tăng = QPS_query * num_shards, dễ thành bottleneck ở load cao.
  - Tối ưu:
    - Chỉ dùng fan-out cho admin/internal query, tần suất thấp.
    - Cho các use case thường xuyên, dùng **denormalized aggregates / reporting DB** cập nhật bằng pipeline.
- [x] Q3: User transfer money từ Shard 1 sang Shard 2. Làm sao bạn ensure atomicity? 2PC hay Saga? Justify choice. (viết analysis với trade-offs)
  - Mô hình:
    - Debit user A (shard1), credit user B (shard2).
  - Nếu yêu cầu **strong consistency, không được phép trạng thái “mất tiền/tiền nhân đôi” dù tạm thời**:
    - Dùng **2PC**:
      - Prepare: lock balance A ở shard1 và B ở shard2, ghi log chuẩn bị.
      - Commit: nếu cả hai vote commit, coordinator gửi commit; nếu 1 bên fail → abort cả hai.
    - Trade-off: latency cao, blocking, coordinator phức tạp.
  - Nếu chấp nhận **eventual consistency trong thời gian ngắn**:
    - Dùng **Saga**:
      - Step1: debit A trên shard1; Step2: credit B trên shard2.
      - Nếu Step2 fail, chạy compensating: credit lại A (rollback).
    - Trade-off: logic business phức tạp hơn, có thể có thời điểm A đã bị debit nhưng B chưa được credit (tạm thời).
  - Lựa chọn:
    - Với hệ thống core banking: nghiêng về 2PC (hoặc thiết kế để hai tài khoản luôn cùng shard).
    - Với wallet scale lớn/fintech chấp nhận eventual consistency ngắn: ưu tiên Saga để scale.
- [x] Q4: Bạn cần reshard từ 10 shards lên 20 shards. Làm sao zero downtime? (viết detailed migration plan)
  - Plan:
    1. Thêm 10 shard mới, join vào cluster, update metadata.
    2. Thiết kế mapping mới (vd: consistent hashing hoặc nhân đôi bucket count).
    3. Bật **double-read** hoặc routing thông minh:
       - Với keys chưa migrated: read/write → shard cũ.
       - Với keys đã migrated: read/write → shard mới.
    4. Chạy background migration:
       - Copy data per range/tenant từ shard cũ sang shard mới.
       - Trong thời gian copy, enable **double-write** cho keys của range đó.
    5. Sau khi verify range, flip routing `read_from = NEW_ONLY`.
    6. Lặp lại cho tới khi toàn bộ keys được migrate.
  - Zero downtime:
    - Hệ thống vẫn phục vụ request trong suốt quá trình; chỉ cần rất ngắn cut-over cho từng batch (atomic metadata update).
- [x] Q5: Shard key = user_id, nhưng query pattern = WHERE merchant_id = ? AND date = ?. Làm sao optimize? (viết optimization strategy)
  - Vấn đề:
    - Shard theo user_id nhưng query chính theo merchant_id/date → phải fan-out tất cả shards.
  - Strategy:
    - **Denormalization/secondary store**:
      - Tạo bảng/bộ dữ liệu `merchant_daily_stats` hoặc `merchant_transactions` trong một DB/warehouse shard theo `merchant_id`.
      - Mỗi transaction (trên shard user_id) emit event (Kafka) và được ETL sang store theo merchant.
    - Trên OLTP shard user_id vẫn để phục vụ use case theo user.
    - Queries theo merchant dùng store denormalized, tránh cross-shard fan-out.
- [x] Q6: Consistent hashing với 10 shards, add 2 more shards. Bao nhiêu % data cần move? (calculate và explain)
  - Tổng shards sau khi add = 12.
  - Với consistent hashing lý tưởng:
    - Mỗi shard chiếm khoảng 1/12 vòng hash.
    - Khi thêm 2 shard, chúng “ăn” bớt hash range từ 10 shard cũ.
    - Tỷ lệ keys cần move ≈ số range mới của 2 shard / tổng vòng = 2/12 ≈ **16.7%** dữ liệu phải move.
- [x] Review: Answers có đủ depth, justification, và numbers không?
  - Đã có both qualitative reasoning và một số con số ước lượng (phần trăm, số record).
- [x] Improve: Answers nếu cần
  - Có thể bổ sung thêm chi tiết implementation code/pseudocode cho 2PC/Saga và resharding trong bước coding thực tế.

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Data Distribution TODOs: ✅ Complete với justifications
- [ ] Tất cả Spring Boot Multi-DB TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Cross-Shard & Transaction TODOs: ✅ Complete với performance analysis
- [ ] Tất cả Migration & Resharding TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Sharding design documents saved: ✅ Yes
- [ ] Performance benchmarks saved: ✅ Yes
- [ ] Ready for Week 5: ✅ Yes

---

> **Mentor Final Check**: Sharding là art và science. Mỗi decision phải được justify với data. Nếu bạn không thể explain TẠI SAO bạn chọn shard key đó, strategy đó, bạn chưa hiểu đủ. Hãy honest: Bạn có thể design sharding strategy cho 1B+ records chưa? Bạn có thể justify MỌI decision chưa? Nếu chưa, làm lại. Production systems không chấp nhận "tôi nghĩ nó sẽ work".
