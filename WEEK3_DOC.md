# Week 3 – Database Design & SQL Scaling

> **Mentor Note**: Database là backbone của mọi hệ thống. Nếu database chậm, toàn bộ system chậm. Tuần này bạn phải
> MASTER database performance. Không có "query chạy được là OK". Mỗi query phải được optimize, measure, và verify.
> Production systems không chấp nhận slow queries.

---

## Study TODOs

### Database Schema Design Fundamentals

- [x] Đọc "Designing Data-Intensive Applications" Chapter 2 - Data Models (pages 30-80)
    - **Key takeaways (DDIA Ch2 – Data Models):**
        - **Relational model**: mạnh về ad-hoc queries, JOIN, consistency; hợp cho OLTP/transactional.
        - **Document model**: schema linh hoạt, phù hợp dữ liệu dạng *aggregate* (vd: user + settings + addresses), giảm
          JOIN; trade-off là khó JOIN/constraint như relational.
        - **Graph model**: tối ưu cho quan hệ phức tạp, nhiều hops (social graph, recommendation).
        - **Insight**: chọn data model theo **access patterns**; không có model “tốt nhất”, chỉ có “phù hợp nhất”.
- [x] Đọc về "Normalization" - 1NF, 2NF, 3NF, BCNF
    - **1NF**: giá trị **nguyên tử** (không list/array trong 1 ô), không lặp group cột.
    - **2NF**: đã 1NF và mọi non-key column **phụ thuộc toàn bộ** vào khóa chính (đặc biệt với composite key).
    - **3NF**: đã 2NF và không có **transitive dependency** (A → B → C).
    - **BCNF**: mọi determinant phải là **candidate key**; chặt hơn 3NF, dùng khi integrity cực quan trọng.
    - **Thực tế**: đa số OLTP dừng ở **3NF**; BCNF chỉ khi thấy anomaly rõ ràng.
- [x] Viết notes: Khi nào normalize? Khi nào denormalize?
    - **Normalize khi:**
        - ✅ Cần consistency/integrity (tránh duplicate data)
        - ✅ Write-heavy (update 1 chỗ, tránh update nhiều bảng/record trùng)
        - ✅ Compliance/financial/healthcare cần ràng buộc chặt
    - **Denormalize khi:**
        - ✅ Read-heavy, cần latency thấp
        - ✅ Muốn giảm JOINs cho hot paths
        - ✅ Analytics/reporting cần query nhanh (pre-aggregations, summary)
- [x] Đọc về "denormalization" - trade-offs với normalization
    - **Normalization**: consistency cao, schema sạch; trade-off là query có thể chậm vì nhiều JOIN.
    - **Denormalization**: query nhanh, ít JOIN; trade-off là **dup data**, risk **inconsistency**, write/update phức
      tạp hơn, cần strategy sync.
- [x] Liệt kê 5 use cases cần normalization
    1. **User management**: users/roles/permissions cần consistency
    2. **E-commerce catalog**: products/categories/attributes tránh trùng lặp
    3. **Financial accounting**: transactions/ledgers integrity critical
    4. **Healthcare records**: compliance, auditability
    5. **CMS**: articles/authors/categories, schema linh hoạt
- [x] Liệt kê 5 use cases cần denormalization
    1. **Analytics dashboard**: pre-aggregated metrics
    2. **Real-time reporting**: fact tables phục vụ query nhanh
    3. **Caching/profile read model**: gộp user + related data để đọc 1 hit
    4. **Time-series**: summary theo window/time bucket
    5. **Search index**: denormalized docs để search nhanh (hoặc sync sang search engine)
- [x] Đọc về "star schema" vs "snowflake schema"
    - **Star schema**:
        - 1 **fact** table lớn (events/transactions) + nhiều **dimension** tables “dày” thông tin.
        - Query đơn giản, ít JOIN sâu → phổ biến trong BI/warehouse.
    - **Snowflake schema**:
        - Dimension được **normalize tiếp** (dimension tách nhỏ nhiều bảng).
        - Giảm trùng lặp dimension nhưng JOIN nhiều hơn, query phức tạp hơn.
    - **Rule of thumb**: bắt đầu **star**, chỉ snowflake khi có lý do rõ ràng (data quality, dimension quá lớn/chuẩn
      hóa).
- [x] Đọc về "entity-relationship modeling" best practices
    - **Model theo business**, không theo UI/screen.
    - **Đặt tên nhất quán** (users/orders/order_items), tránh viết tắt mơ hồ.
    - **Xác định cardinality** rõ ràng (1-1, 1-n, n-n); n-n luôn có **junction table**.
    - Ưu tiên **surrogate key** (INT/BIGINT/UUID) cho bảng core; natural key để UNIQUE constraint.
    - **Tách entity theo lifecycle/tần suất update** (profile/settings/logs/history).
    - Thiết kế dựa trên **access patterns** (liệt kê query quan trọng trước khi chốt ERD).
- [x] Đọc về "foreign key constraints" - performance impact
    - **INSERT/UPDATE/DELETE**: FK check tạo overhead + cần index trên FK columns để tránh scan.
    - **DELETE parent**: có thể tăng lock contention/cascade cost, dễ nghẽn nếu workload write cao.
    - **Operational**: bulk import/ETL đôi khi phải disable tạm để nhanh, rồi validate/sync lại.
- [x] Research: Foreign keys trong production - enable hay disable? Tại sao?
    - **Enable FK khi:**
        - Integrity là ưu tiên (financial/healthcare), schema trong 1 DB, write rate vừa phải
    - **Disable/avoid FK khi:**
        - Hệ thống cực high-write, latency critical (gaming/betting), hoặc microservices (FK cross-service không khả
          thi)
    - **Best practice thực dụng**:
        - Nếu disable FK: bắt buộc có **application-level validation + background consistency checks** (job reconcile),
          và design để tránh orphan records.

### Indexing Deep Dive

- [x] Đọc về "B-tree index" - how it works internally
    - Cấu trúc **cây cân bằng nhiều nhánh**, keys được sort trong nodes; lookup/insert/delete đều ~O(log n) vì chiều cao
      cây thấp và được tự re-balance.
- [x] Đọc về "Hash index" - use cases và limitations
    - Dùng **hash(key) → bucket** nên rất tốt cho **equality lookup** (=), nhưng không hỗ trợ range scan, ORDER BY hay
      prefix search; phù hợp key-value exact match.
- [x] Đọc về "Composite index" - column order matters
    - Index `(A,B,C)` hữu dụng cho điều kiện bắt đầu từ trái: `A`, `A,B`, `A,B,C`; nếu chỉ filter bằng `B` hoặc `C` (bỏ
      A) thì thường không tận dụng được index tốt.
- [x] Viết notes: Index selectivity - what is it? Why important?
    - **Selectivity = distinct_values / total_rows**; càng gần 1 càng tốt.
    - High selectivity → index filter được nhiều rows → query nhanh hơn; low selectivity (vd: giới tính) thường không
      đáng để index một mình.
      Bạn hiểu đúng rồi 👍 Mình giải thích kỹ hơn một chút theo góc nhìn thực tế (đúng kiểu phỏng vấn + production luôn):

    ---

  ## 1️⃣ Selectivity là gì?

  **Selectivity = distinct_values / total_rows**

  Ví dụ bảng `users` có 1,000,000 rows:

  | Column    | Distinct values | Selectivity |
        |-----------|-----------------|-------------|
  | `user_id` | 1,000,000       | 1.0         |
  | `email`   | 1,000,000       | 1.0         |
  | `gender`  | 2               | 0.000002    |
  | `status`  | 3               | 0.000003    |
    
  ---

  ## 2️⃣ Vì sao selectivity cao thì index hiệu quả?

  Giả sử:

    ```sql
    SELECT *
    FROM users
    WHERE email = 'abc@gmail.com';
    ```

    * email gần như unique
    * index lookup → trả về 1 row
    * rất ít I/O
    * cực nhanh

  Nhưng:

    ```sql
    SELECT *
    FROM users
    WHERE gender = 'MALE';
    ```

    * có thể 500,000 rows
    * index scan → rồi vẫn phải đọc rất nhiều page
    * lúc này optimizer có thể chọn **full table scan** còn nhanh hơn

  👉 Vì index không giúp “lọc” đủ mạnh.
    
  ---

  ## 3️⃣ High selectivity vs Low selectivity

  ### 🔥 High selectivity (nên index)

    * id
    * email
    * phone
    * order_code
    * transaction_id
    * UUID

  ### ❌ Low selectivity (thường không nên index riêng lẻ)

    * gender
    * is_active (true/false)
    * status (PENDING, SUCCESS, FAILED)
    * country (nếu chỉ vài nước)

    ---

  ## 4️⃣ Nhưng low selectivity có thật sự vô dụng?

  Không hẳn ❗

  Nó hữu ích khi:

  ### ✅ Làm composite index

    ```sql
    INDEX
    (status, created_at)
    ```

  Query:

    ```sql
    WHERE status = 'PENDING'
    AND created_at >= '2026-01-01'
    ```

  → Lúc này index rất hiệu quả vì:

    * status lọc subset
    * created_at tiếp tục lọc nhỏ hơn nữa

    ---

  ## 5️⃣ Selectivity và Unique Index

  Unique index có selectivity gần 1 → rất mạnh.

  Trong:

    * MySQL
    * PostgreSQL

  Unique index còn giúp optimizer:

    * Biết chắc chắn chỉ trả về tối đa 1 row
    * Có thể chọn plan tốt hơn

    ---

  ## 6️⃣ Thực tế production (rất quan trọng)

  Trong hệ thống lớn (như hệ affiliate bạn từng nói có dữ liệu cực lớn):

    * Index sai → write chậm
    * Index dư → tốn RAM
    * Index low selectivity → không được dùng nhưng vẫn phải maintain

  Nên khi design index:

    1. Liệt kê query quan trọng trước (access pattern)
    2. Kiểm tra cardinality
    3. Dùng `EXPLAIN`
    4. Đo thực tế

    ---

  ## 7️⃣ Một insight nâng cao (interview thích hỏi)

  Selectivity chỉ là 1 yếu tố.

  Optimizer còn xem:

    * Distribution skew
        * Histogram statistics
        * Correlation giữa columns
        * Cost model (I/O vs CPU)

  Ví dụ:

    * `status = 'PENDING'` chiếm 1% thôi → dù distinct = 3 nhưng vẫn rất selective.
      Đoạn này là phần “nâng cao” của selectivity — và rất hay bị hỏi trong interview senior 😄
      Mình giải thích từng ý theo hướng thực tế DB engine (MySQL / PostgreSQL).

    ---

  # 1️⃣ Vấn đề của công thức selectivity cơ bản

  Công thức bạn đọc:

    ```
    selectivity = distinct_values / total_rows
    ```

  Công thức này **giả định dữ liệu phân bố đều (uniform distribution)**.

  Nhưng ngoài đời gần như KHÔNG BAO GIỜ đều.
    
  ---

  # 2️⃣ Distribution Skew (Phân bố lệch)

  Ví dụ bảng 1,000,000 rows:

  | status  | count   |
        | ------- | ------- |
  | SUCCESS | 980,000 |
  | FAILED  | 10,000  |
  | PENDING | 10,000  |

  Distinct = 3
  Selectivity theo công thức = 3 / 1,000,000 = rất thấp ❌

  Nhưng query:

    ```sql
    WHERE status = 'PENDING'
    ```

  → chỉ trả 1% rows
  → thực tế rất selective
  → index có giá trị

  💡 Đây gọi là **skewed distribution**.
    
  ---

  # 3️⃣ Histogram Statistics là gì?

  DB hiện đại (MySQL 8, PostgreSQL) không chỉ lưu:

    * number of distinct
        * total rows

  Mà còn lưu **histogram**:

  → Phân bố tần suất giá trị

  Ví dụ histogram sẽ biết:

    * SUCCESS = 98%
        * FAILED = 1%
        * PENDING = 1%

  Nên optimizer tính được:

    ```
    selectivity(status = 'PENDING') ≈ 0.01
    ```

  Chứ không dùng 1/3.
    
  ---

  # 4️⃣ Correlation giữa columns

  Giả sử:

    ```sql
    WHERE country = 'VN'
    AND city = 'HCM'
    ```

  Nếu optimizer nghĩ:

    * country selectivity = 10%
        * city selectivity = 5%

  Nó có thể tính sai:

    ```
    0.1 × 0.05 = 0.005 (0.5%)
    ```

  Nhưng thực tế:

    * HCM gần như luôn thuộc VN

  → 2 điều kiện có correlation cao
  → không độc lập

  Nếu DB không biết correlation → estimate sai → chọn plan sai.

  PostgreSQL hỗ trợ extended statistics để xử lý correlation tốt hơn.
    
  ---

  # 5️⃣ Cost Model (I/O vs CPU)

  Optimizer không hỏi:

  > Index có selective không?

  Mà hỏi:

  > Dùng index có rẻ hơn full scan không?

  Nó tính:

    ```
    Total cost = I/O cost + CPU cost
    ```

  Ví dụ:

    * status = SUCCESS (98%)
      → index scan phải đọc 980k rows
      → random I/O nhiều
      → full table scan có thể rẻ hơn

  → optimizer bỏ index
    
  ---

  # 6️⃣ Vì sao `distinct=3` nhưng vẫn selective?

  Quay lại ví dụ:

  status có 3 giá trị
  distinct thấp
  → theo công thức basic → low selectivity

  Nhưng nếu phân bố lệch:

    ```
    PENDING = 1%
    ```

  Thì:

    ```
    selectivity(status='PENDING') = 0.01
    ```

  Không còn là 1/3 nữa.

  💡 Selectivity thực sự là theo VALUE cụ thể, không phải theo column chung chung.
    
  ---

  # 7️⃣ Tổng hợp lại

  Optimizer hiện đại xem:

    1. Table row count
    2. Distinct values
    3. Histogram (frequency distribution)
    4. Correlation giữa columns
    5. Cost model (I/O vs CPU)
    6. Index structure (B-tree depth, clustering…)

  Chứ không chỉ mỗi distinct/rows.
    
  ---

  # 8️⃣ Thực tế production (quan trọng)

  Trong hệ thống dữ liệu lớn như affiliate/reporting:

    * Data skew rất phổ biến
    * 1 vài status chiếm gần hết
    * 1 vài status hiếm

  Nếu statistics outdated:

    * Optimizer estimate sai
    * Query plan tệ
    * Query đột nhiên chậm

  Nên phải:
    
    ```
    ANALYZE TABLE
    ```

  hoặc auto-vacuum / auto-analyze (Postgres).
    
  ---


- [x] Đọc về "covering index" - index-only scans
    - Index chứa **tất cả columns** mà query cần → engine chỉ đọc index, không phải quay lại table (**index-only scan
      **), giảm I/O và latency.
- [x] Đọc về "partial index" - conditional indexes
    - Index chỉ tạo trên subset rows với `WHERE` (vd: `status = 'PENDING'`) → index nhỏ, tập trung cho hot subset, giảm
      overhead write.
- [x] Đọc về "unique index" vs "non-unique index" - performance difference
    - Unique index enforce uniqueness và cho phép optimizer giả định tối đa 1 row match, đôi khi cho plan tốt hơn; nhưng
      mọi write phải check unique constraint → thêm chút overhead.
- [x] Đọc về "index maintenance" - INSERT/UPDATE/DELETE overhead
    - Mỗi thay đổi dữ liệu phải update **tất cả indexes liên quan**; nhiều index → write cost tăng (CPU + I/O), đặc biệt
      trên bảng lớn.
- [x] Research: How many indexes is too many? (general rule)
    - Rule of thumb: small table: 3–5 index; medium: 5–7; large: 5–10 là đã phải rất cân nhắc; luôn dựa vào **read/write
      profile + index usage stats** để quyết.
- [x] Đọc về "index fragmentation" - when và how to rebuild
    - Nhiều INSERT/DELETE/UPDATE gây pages rỗng/không liên tục → fragmentation, làm scan chậm.
    - Rebuild/reorganize index khi fragmentation cao (vd > ~30%) hoặc sau bulk operations lớn.
- [x] Đọc về "clustered index" vs "non-clustered index" (SQL Server) hoặc "primary key" vs "secondary index" (MySQL)
    - **Clustered/primary**: data physically sắp theo key, tốt cho range scan; chỉ có 1.
    - **Non-clustered/secondary**: cấu trúc riêng chứa key + pointer tới row/PK; có thể có nhiều.
- [x] Đọc về "full-text index" - when to use
    - Dùng cho **search text** (title/content), support tokenization, ranking; không thay thế được search engine chuyên
      dụng nhưng tốt cho full-text search đơn giản trong DB.

### Query Optimization

- [x] Đọc về "EXPLAIN" / "EXPLAIN PLAN" - how to read output
    - Tập trung vào: **loại scan/join**, index nào dùng, rows ước tính/actual, và flags Extra (vd: `Using where`,
      `Using filesort`, `Using temporary`) để tìm chỗ full scan hoặc sort tốn kém.
- [x] Đọc về "query execution plan" - steps database takes
    - Plan cho biết thứ tự đọc bảng, cách JOIN, cách apply WHERE/GROUP BY/ORDER BY; hiểu plan giúp biết nên thêm/sửa
      index, đổi query thế nào.
- [x] Đọc về "table scan" vs "index scan" vs "index seek"
    - **Table scan**: đọc hết bảng → O(n), tệ cho bảng lớn.
    - **Index scan**: scan toàn index (thường cho range).
    - **Index seek**: nhảy thẳng vào đoạn cần trong index → nhanh nhất.
- [x] Đọc về "cost-based optimizer" - how it works
    - Optimizer sinh nhiều plan, estimate cost dựa trên **statistics**, rồi chọn plan rẻ nhất; stats xấu/outdated → chọn
      plan tệ.
- [x] Đọc về "query hints" - when to use (rarely)
    - Hints ép engine dùng index/plan cụ thể; chỉ nên dùng khi đã hiểu rõ và không thể fix bằng schema/index/stats, vì
      hints có thể lỗi thời khi data thay đổi.
- [x] Đọc về "statistics" - why database needs them
    - Stats (row count, distribution, distinct values) là input cho optimizer để estimate **cardinality** và cost; cần
      được update định kỳ.
- [x] Đọc về "cardinality estimation" - why it matters
    - Ước lượng sai số rows → chọn sai join order/algorithm, có thể dẫn tới plan cực chậm (hash join thay vì nested
      loop, hoặc ngược lại).
- [x] Đọc về "JOIN algorithms" - nested loop, hash join, merge join
    - **Nested loop**: tốt khi 1 bảng nhỏ + index trên bảng kia.
    - **Hash join**: build hash trên bảng nhỏ, tốt cho large scans không index.
    - **Merge join**: hiệu quả khi cả 2 input đã sort theo join key.
- [x] Đọc về "subquery" vs "JOIN" - performance comparison
    - Nhiều subquery có thể được rewrite thành JOIN, thường dễ optimize và rõ ràng hơn; đa số engine cũng transform
      internally, nhưng viết JOIN rõ ràng thường an toàn.
- [x] Đọc về "EXISTS" vs "IN" vs "JOIN" - when to use which
    - **EXISTS**: check tồn tại, dừng sớm; tốt cho existence check.
    - **IN**: tốt với list nhỏ; subquery lớn có thể kém.
    - **JOIN**: khi cần dữ liệu từ nhiều bảng; thường nhanh nhất cho large sets.
- [x] Đọc về "LIMIT/OFFSET" performance issues
    - `LIMIT ... OFFSET big` buộc DB **skip** rất nhiều rows → chậm dần theo page; tệ cho deep pagination.
- [x] Đọc về "cursor-based pagination" - better alternative
    - Dùng **key của row cuối** làm cursor (`WHERE id > last_id`) thay vì OFFSET, tận dụng index tốt hơn và không bị
      skip/lặp khi có dữ liệu mới.

### Connection Pooling

- [x] Đọc về "connection pooling" - why needed?
    - Tạo/đóng DB connection rất đắt (network handshake, auth); pool reuse connections, giới hạn max concurrent
      connections, cải thiện latency và stability.
- [x] Đọc về "HikariCP" - default Spring Boot pool
    - HikariCP là default pool trong Spring Boot, tối ưu performance, ít config nhưng đủ hooks cho tuning, support
      metrics tốt.
- [x] Đọc về pool size calculation: connections = ((core_count * 2) + effective_spindle_count)
    - Công thức này cho **starting point**; sau đó phải dựa trên metrics (active vs idle, wait time) và capacity DB thực
      tế để tune.
- [x] Đọc về "connection pool tuning" - min, max, idle timeout
    - `minimumIdle`, `maximumPoolSize`, `idleTimeout`, `maxLifetime`, `connectionTimeout` cần align với pattern traffic;
      tránh max quá lớn gây overload DB, cũng tránh quá nhỏ gây wait nhiều.
- [x] Đọc về "connection leak detection"
    - Leak xảy ra khi mượn connection mà không trả; HikariCP có `leakDetectionThreshold` để log cảnh báo khi 1
      connection bị giữ quá lâu → giúp bắt bug.
- [x] Đọc về "connection timeout" vs "query timeout"
    - **Connection timeout**: thời gian chờ lấy connection từ pool.
    - **Query timeout**: thời gian tối đa cho một query chạy; bảo vệ khỏi query treo/hỏng plan.
- [x] Research: HikariCP configuration best practices
    - Giữ pool size **vừa đủ**, bật metrics, set `maxLifetime` < lifetime thực của DB connections, và test dưới load
      thật để tune; không set timeout quá cao khiến request treo lâu.
- [x] Đọc về "prepared statements" - performance và security
    - Prepared statements tách query template và params → tránh SQL injection và reuse plan, giảm chi phí parse/optimize
      lặp lại.
- [x] Đọc về "statement caching" - reduce parsing overhead
    - Cache prepared statements (client side hoặc server side) giúp tránh parse/plan lại; đặc biệt hiệu quả với query
      lặp nhiều.

### Read Replicas & Replication

- [x] Đọc về "master-slave replication" - how it works
    - Master ghi data, log lại (binlog/WAL); slaves đọc log và apply theo thứ tự → dữ liệu trên slave **trễ** so với
      master.
- [x] Đọc về "replication lag" - what causes it?
    - Network latency, load cao trên slave, disk chậm, hoặc burst write lớn trên master khiến slave apply không kịp.
- [x] Đọc về "eventual consistency" trong read replicas
    - Slave chỉ đảm bảo **eventual** sync; tại một thời điểm có thể outdated vài giây (hoặc hơn) so với master.
- [x] Đọc về "read-after-write consistency" problem
    - Sau khi user ghi (update balance) vào master, nếu ngay lập tức đọc từ slave có thể không thấy update → cần route
      các read-critical về master hoặc có strategy đặc biệt.
- [x] Đọc về "read scaling" - when read replicas help
    - Đọc nhiều hơn ghi, query chủ yếu là read-only/analytics → có thể offload sang nhiều slaves để scale read.
- [x] Đọc về "write scaling" - read replicas DON'T help
    - Việc ghi vẫn bottleneck ở master; replicas chỉ giúp đọc, không tăng write throughput (trừ khi sharding).
- [x] Research: MySQL replication setup
    - MySQL dùng binlog, slave `CHANGE MASTER TO ...` để subscribe; có nhiều mode (async, semi-sync) với trade-off
      latency vs durability.
- [x] Research: PostgreSQL replication setup
    - PostgreSQL dùng WAL shipping/streaming replication; slaves thường là hot-standby có thể phục vụ read-only.
- [x] Đọc về "replication topologies" - chain, star, etc.
    - Chain (master → slave1 → slave2...), fan-out/star từ master, multi-tier; mỗi kiểu có trade-off về **lag**, độ phức
      tạp và single points of failure.
- [x] Đọc về "failover" trong master-slave setup
    - Cần cơ chế promote slave lên master + re-point các node khác; phải xử lý split-brain, mất dữ liệu chưa replicate,
      và đảm bảo app biết master mới.

### Partitioning Basics

- [x] Đọc về "horizontal partitioning" (sharding)
    - Chia rows theo key (vd: user_id) ra nhiều shard/partition; mỗi partition chứa subset của rows nhưng full columns.
- [x] Đọc về "vertical partitioning" (column splitting)
    - Chia columns của cùng 1 logical entity ra nhiều bảng (hot columns vs cold/large columns) để tối ưu I/O và width
      của row.
- [x] Đọc về "range partitioning" - partition by date range
    - Mỗi partition là 1 khoảng (vd: theo tháng/năm); cực hợp cho time-series và queries theo date range.
- [x] Đọc về "hash partitioning" - partition by hash
    - Hash(key) mod N → chia đều data, giúp balance load; nhưng cross-key range scan kém tự nhiên hơn range partition.
- [x] Đọc về "list partitioning" - partition by category
    - Phân theo set giá trị cụ thể (region = US/EU/APAC); hữu ích khi workload khác nhau theo nhóm giá trị.
- [x] Đọc về "partition pruning" - query optimization
    - Optimizer chỉ scan partitions có liên quan dựa trên điều kiện WHERE (ví dụ theo date) → giảm I/O rất nhiều nếu
      partition design đúng.
- [x] Đọc về "cross-partition queries" - performance impact
    - Query phải chạm nhiều partition/shard sẽ tốn chi phí tổng hợp/merge, có thể mất lợi thế partition; cần design
      access pattern để đa số query chỉ chạm ít partition.
- [x] Research: MySQL partitioning
    - MySQL hỗ trợ range/list/hash partition ở mức table; cần cẩn thận với constraints và index (index thường
      per-partition).
- [x] Research: PostgreSQL partitioning
    - PostgreSQL có declarative partitioning (range/list/hash); query planner làm pruning khá tốt nếu WHERE align với
      partition key.

### Transaction Isolation Levels

- [x] Đọc về "ACID properties" - Atomicity, Consistency, Isolation, Durability
    - **Atomicity**: all-or-nothing; **Consistency**: không phá vỡ invariants; **Isolation**: transaction như chạy tuần
      tự; **Durability**: đã commit thì không mất (dù crash).
- [x] Đọc về "transaction isolation levels" - READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
    - Mỗi level cấm thêm một số anomaly, đổi lại cost concurrency/performance cao hơn.
- [x] Đọc về "dirty read" - what is it?
    - Đọc dữ liệu từ transaction khác **chưa commit**; nếu bị rollback thì read đó là “bẩn”.
- [x] Đọc về "non-repeatable read" - what is it?
    - Cùng 1 query trong 1 transaction nhưng chạy 2 lần cho ra **giá trị khác nhau** vì transaction khác đã commit
      update.
- [x] Đọc về "phantom read" - what is it?
    - Lần sau xuất hiện **thêm/hết rows** thỏa điều kiện, không chỉ đổi giá trị, do transaction khác insert/delete giữa
      chừng.
- [x] Viết table: Isolation level → Prevents which anomalies?
    - READ UNCOMMITTED: không ngăn gì.
    - READ COMMITTED: ngăn **dirty read**.
    - REPEATABLE READ: ngăn **dirty + non-repeatable read** (phantom tùy engine).
    - SERIALIZABLE: ngăn cả **phantom** (gần như serialize).
- [x] Đọc về "MVCC" (Multi-Version Concurrency Control) - how it works
    - Lưu nhiều version row, mỗi transaction đọc snapshot phù hợp với timestamp của nó; giảm cần lock read, cho phép
      read không block write (và ngược lại) trong đa số case.
- [x] Đọc về "locking" - shared locks, exclusive locks
    - **Shared (read)**: nhiều reader cùng nắm được, block writer.
    - **Exclusive (write)**: 1 writer, block reader khác; lock scope có thể là row/page/table tùy engine và query.
- [x] Đọc về "deadlock" - what causes it? How to prevent?
    - Xảy ra khi 2+ transactions giữ lock A, chờ lock B của nhau; prevention: thứ tự truy cập tài nguyên nhất quán, giữ
      transaction ngắn, index tốt để lock ít rows hơn.
- [x] Đọc về "optimistic locking" vs "pessimistic locking"
    - **Optimistic**: đọc version, khi update check version không đổi; tốt cho conflict ít.
    - **Pessimistic**: lock row ngay khi đọc để chặn concurrent writers; tốt khi conflict cao nhưng giảm concurrency.
- [x] Research: Default isolation level trong MySQL vs PostgreSQL
    - MySQL InnoDB: mặc định **REPEATABLE READ**; PostgreSQL: mặc định **READ COMMITTED**.

### Data Consistency vs Performance

- [x] Đọc về "strong consistency" - trade-offs
    - Mọi read sau write đều thấy dữ liệu mới (theo 1 model rõ ràng); thường cần coordination mạnh, lock nhiều hơn,
      latency cao hơn, khó scale.
- [x] Đọc về "eventual consistency" - trade-offs
    - Cho phép node/replica khác nhau tạm thời thấy state khác nhau, miễn là rồi sẽ converge; dễ scale, latency thấp
      hơn, nhưng app phải chấp nhận và xử lý **stale reads**.
- [x] Đọc về "read consistency" - snapshot isolation
    - Snapshot isolation dùng MVCC để mỗi transaction đọc 1 snapshot nhất quán của DB, không thấy half-committed state,
      giảm lock giữa readers và writers.
- [x] Đọc về "write consistency" - how to ensure
    - Dùng transaction đúng chỗ, constraints ở DB (FK, UNIQUE, CHECK), idempotent operations, và nếu distributed thì cần
      protocol đảm bảo commit đồng bộ.
- [x] Analyze: When to sacrifice consistency for performance?
    - Khi dữ liệu **không critical** (likes, views, feed ordering), chấp nhận đọc hơi cũ nhưng đổi lại
      throughput/latency tốt hơn, hoặc trong hệ thống phân tán multi-region.
- [x] Analyze: When MUST have strong consistency?
    - Số dư wallet, chuyển tiền, đơn hàng thanh toán, quyền truy cập/security, hoặc bất kỳ thứ gì mà state sai 1 lần là
      gây mất tiền/tổn hại lớn.
- [x] Đọc về "distributed transactions" - 2PC, performance impact
    - **2-phase commit** đảm bảo atomic commit giữa nhiều resource managers, nhưng thêm coordinator, nhiều round-trip,
      dễ thành bottleneck và có risk blocking nếu coordinator chết; thường tránh cho high-throughput path, ưu tiên
      design **saga/eventual consistency** nếu phù hợp.

---

## Schema & Modeling TODOs

### Schema Design Exercise 1: Wallet System

- [ ] Design: Complete database schema cho Wallet System
- [ ] Requirement: 10M users, 100M transactions, balance queries < 10ms
- [ ] Design: Users table (columns, data types, constraints)
- [ ] Design: Wallets table (columns, data types, constraints)
- [ ] Design: Transactions table (columns, data types, constraints)
- [ ] Design: Transaction types (enum hoặc lookup table)
- [ ] Design: Indexes cho mỗi table (list all indexes với rationale)
- [ ] Design: Foreign keys (which ones? why?)
- [ ] Verify: Normalization level (1NF, 2NF, 3NF?)
- [ ] Consider: Denormalization opportunities (nếu có)
- [ ] Create: ERD diagram (draw.io hoặc tool)
- [ ] Document: Schema design decisions (500 words)

### Schema Design Exercise 2: Payment Gateway

- [ ] Design: Database schema cho Payment Gateway
- [ ] Requirement: 1M transactions/day, complex reporting queries
- [ ] Design: Merchants table
- [ ] Design: Payments table (with all required fields)
- [ ] Design: Payment status tracking (state machine)
- [ ] Design: Refunds table
- [ ] Design: Payment methods table
- [ ] Design: Indexes strategy (consider query patterns)
- [ ] Consider: Partitioning strategy (by date? by merchant?)
- [ ] Design: Archive strategy (old transactions)
- [ ] Create: Complete schema với all tables
- [ ] Document: Design decisions và trade-offs

### Schema Design Exercise 3: Betting Platform

- [ ] Design: Database schema cho Betting Platform
- [ ] Requirement: Real-time odds updates, bet history, settlement
- [ ] Design: Matches table
- [ ] Design: Odds table (frequently updated)
- [ ] Design: Bets table
- [ ] Design: Bet outcomes table
- [ ] Design: User accounts table
- [ ] Design: Indexes (consider update frequency)
- [ ] Consider: How to handle high-frequency odds updates?
- [ ] Consider: Denormalization cho read-heavy queries
- [ ] Design: Data retention policy
- [ ] Create: Schema diagram
- [ ] Document: Design rationale

### Schema Review & Optimization

- [ ] Review: Wallet System schema
- [ ] Check: All indexes có necessary không?
- [ ] Check: Missing indexes? (identify potential)
- [ ] Check: Over-indexing? (too many indexes)
- [ ] Check: Data types optimal? (VARCHAR size, INT vs BIGINT)
- [ ] Check: Constraints appropriate? (NOT NULL, UNIQUE, CHECK)
- [ ] Optimize: Schema based on review
- [ ] Document: Schema review findings

---

## SQL Optimization TODOs

### Query Optimization Exercise 1: Slow Query Identification

- [ ] Setup: Database với 1M+ records (use sample data generator)
- [ ] Create: Users table với 1M records
- [ ] Create: Orders table với 10M records
- [ ] Create: OrderItems table với 50M records
- [ ] Write: Query to find user orders (JOIN Users và Orders)
- [ ] Run: EXPLAIN PLAN cho query
- [ ] Identify: Table scan? Index scan? Index seek?
- [ ] Measure: Query execution time
- [ ] Document: Baseline performance

### Query Optimization Exercise 2: Index Optimization

- [ ] Query: SELECT * FROM orders WHERE user_id = ? AND status = 'PENDING'
- [ ] Run: EXPLAIN PLAN (before optimization)
- [ ] Measure: Execution time (before)
- [ ] Analyze: Missing index? Wrong index?
- [ ] Create: Appropriate index (single hoặc composite)
- [ ] Run: EXPLAIN PLAN (after optimization)
- [ ] Measure: Execution time (after)
- [ ] Calculate: Performance improvement (%)
- [ ] Verify: Index được sử dụng (check EXPLAIN output)
- [ ] Document: Optimization results

### Query Optimization Exercise 3: JOIN Optimization

- [ ] Write: Complex query với 3-table JOIN
- [ ] Query: Users JOIN Orders JOIN OrderItems
- [ ] Run: EXPLAIN PLAN
- [ ] Identify: JOIN order (does it matter?)
- [ ] Identify: JOIN algorithm used (nested loop? hash join?)
- [ ] Optimize: JOIN order (if needed)
- [ ] Add: Indexes to support JOINs
- [ ] Measure: Performance before và after
- [ ] Document: JOIN optimization

### Query Optimization Exercise 4: Aggregation Optimization

- [ ] Write: Query với GROUP BY và aggregate functions
- [ ] Query: SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Analyze: Can index help GROUP BY?
- [ ] Create: Index to optimize GROUP BY
- [ ] Measure: Performance improvement
- [ ] Consider: Materialized view? (if applicable)
- [ ] Document: Aggregation optimization

### Query Optimization Exercise 5: Subquery vs JOIN

- [ ] Write: Query using subquery
- [ ] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000)
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Rewrite: Same query using JOIN
- [ ] Run: EXPLAIN PLAN
- [ ] Measure: Execution time
- [ ] Compare: Subquery vs JOIN performance
- [ ] Document: Which is better? Why?

### Query Optimization Exercise 6: EXISTS vs IN vs JOIN

- [ ] Write: Query using IN clause
- [ ] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders)
- [ ] Measure: Execution time
- [ ] Rewrite: Using EXISTS
- [ ] Measure: Execution time
- [ ] Rewrite: Using JOIN
- [ ] Measure: Execution time
- [ ] Compare: IN vs EXISTS vs JOIN
- [ ] Analyze: Which performs best? Why?
- [ ] Document: Best practice recommendation

### Query Optimization Exercise 7: Pagination Optimization

- [ ] Write: Query với LIMIT và OFFSET
- [ ] Example: SELECT * FROM orders ORDER BY created_at LIMIT 10 OFFSET 1000000
- [ ] Measure: Execution time (slow với large OFFSET)
- [ ] Analyze: Why is it slow?
- [ ] Rewrite: Using cursor-based pagination
- [ ] Example: SELECT * FROM orders WHERE id > ? ORDER BY id LIMIT 10
- [ ] Measure: Execution time
- [ ] Compare: OFFSET vs cursor-based
- [ ] Document: Pagination best practices

### Query Optimization Exercise 8: Full Table Scan Prevention

- [ ] Write: Query that causes full table scan
- [ ] Run: EXPLAIN PLAN - verify table scan
- [ ] Identify: Missing index
- [ ] Create: Index to prevent table scan
- [ ] Run: EXPLAIN PLAN - verify index used
- [ ] Measure: Performance improvement
- [ ] Document: Table scan elimination

### Query Performance Testing

- [ ] Create: Test suite với 10 different queries
- [ ] Run: All queries và measure execution time
- [ ] Identify: 3 slowest queries
- [ ] Optimize: 3 slowest queries
- [ ] Measure: Performance improvement cho mỗi query
- [ ] Set: Performance targets (p95 < 100ms)
- [ ] Verify: All queries meet targets
- [ ] Document: Performance test results

---

## Spring Boot + DB TODOs

### Task 1: HikariCP Configuration

- [ ] Create: Spring Boot project với database
- [ ] Add: HikariCP dependency (default trong Spring Boot)
- [ ] Configure: Connection pool size (calculate based on formula)
- [ ] Configure: Minimum idle connections
- [ ] Configure: Maximum pool size
- [ ] Configure: Connection timeout
- [ ] Configure: Idle timeout
- [ ] Configure: Max lifetime
- [ ] Add: Connection pool monitoring (metrics)
- [ ] Test: Under load, verify pool không exhausted
- [ ] Document: HikariCP configuration

### Task 2: JPA/Hibernate Optimization

- [ ] Setup: Spring Data JPA với Hibernate
- [ ] Configure: HikariCP connection pool
- [ ] Configure: Hibernate batch size
- [ ] Configure: Hibernate fetch size
- [ ] Configure: Hibernate second-level cache (optional)
- [ ] Write: Entity classes với proper annotations
- [ ] Write: Repository với custom queries
- [ ] Test: Query performance
- [ ] Monitor: Hibernate SQL logs (verify no N+1 queries)
- [ ] Optimize: N+1 queries (if found)
- [ ] Document: JPA optimization

### Task 3: Query Performance Monitoring

- [ ] Add: Micrometer metrics
- [ ] Expose: Database query metrics
- [ ] Monitor: Query execution time
- [ ] Monitor: Connection pool usage
- [ ] Monitor: Slow queries (log queries > 1 second)
- [ ] Setup: Alert cho slow queries
- [ ] Create: Dashboard với key metrics
- [ ] Document: Monitoring setup

### Task 4: Prepared Statements

- [ ] Verify: Spring JPA uses prepared statements (check logs)
- [ ] Write: Native query với prepared statement
- [ ] Test: SQL injection prevention
- [ ] Measure: Performance vs regular statements
- [ ] Document: Prepared statement usage

### Task 5: Transaction Management

- [ ] Implement: Service method với @Transactional
- [ ] Test: Transaction rollback on exception
- [ ] Configure: Transaction isolation level
- [ ] Test: Different isolation levels (READ COMMITTED, REPEATABLE READ)
- [ ] Measure: Performance impact của isolation levels
- [ ] Document: Transaction configuration

### Task 6: Database Migration

- [ ] Setup: Flyway hoặc Liquibase
- [ ] Create: Initial schema migration
- [ ] Create: Index migration
- [ ] Create: Data migration (if needed)
- [ ] Test: Migration rollback
- [ ] Document: Migration strategy

### Task 7: Connection Leak Detection

- [ ] Configure: HikariCP leak detection
- [ ] Set: Leak detection threshold (10 seconds)
- [ ] Test: Simulate connection leak (don't close connection)
- [ ] Verify: Leak detection logs warning
- [ ] Fix: Connection leak
- [ ] Document: Leak detection setup

### Task 8: Read-Write Splitting (Optional)

- [ ] Setup: MySQL master-slave replication
- [ ] Configure: Spring Boot với multiple data sources
- [ ] Implement: Read queries go to slave
- [ ] Implement: Write queries go to master
- [ ] Test: Read từ slave
- [ ] Test: Write to master
- [ ] Handle: Replication lag (read-after-write consistency)
- [ ] Document: Read-write splitting implementation

---

## Scaling & Replication TODOs

### Replication Setup Exercise

- [ ] Research: MySQL master-slave replication setup
- [ ] Setup: MySQL master server
- [ ] Setup: MySQL slave server
- [ ] Configure: Replication
- [ ] Test: Write to master, verify replication to slave
- [ ] Measure: Replication lag
- [ ] Test: Read from slave
- [ ] Document: Replication setup

### Read Replica Design

- [ ] Design: Read replica architecture cho Payment System
- [ ] Calculate: Number of read replicas needed (based on read load)
- [ ] Design: Read routing strategy (round-robin? least lag?)
- [ ] Design: Failover strategy (if replica fails)
- [ ] Design: Read-after-write consistency handling
- [ ] Document: Read replica design

### Partitioning Design

- [ ] Design: Partitioning strategy cho Transactions table
- [ ] Requirement: 100M transactions, query by date range
- [ ] Choose: Range partitioning by date
- [ ] Design: Partition key (created_at)
- [ ] Design: Partition pruning strategy
- [ ] Consider: Cross-partition queries impact
- [ ] Document: Partitioning design

### Database Sharding Design (Basic)

- [ ] Read: Database sharding concepts
- [ ] Design: Sharding strategy cho Users table (10M users)
- [ ] Choose: Shard key (user_id)
- [ ] Choose: Sharding algorithm (hash-based)
- [ ] Design: Shard routing logic
- [ ] Design: Cross-shard query handling
- [ ] Document: Sharding design (high-level)

---

## Consistency & Transaction TODOs

### Transaction Isolation Analysis

- [ ] Test: READ UNCOMMITTED isolation level
- [ ] Simulate: Dirty read scenario
- [ ] Verify: Dirty read occurs
- [ ] Test: READ COMMITTED isolation level
- [ ] Simulate: Non-repeatable read scenario
- [ ] Verify: Non-repeatable read occurs
- [ ] Test: REPEATABLE READ isolation level
- [ ] Simulate: Phantom read scenario
- [ ] Verify: Phantom read occurs (or not?)
- [ ] Test: SERIALIZABLE isolation level
- [ ] Measure: Performance impact của each level
- [ ] Document: Isolation level analysis

### Deadlock Analysis

- [ ] Simulate: Deadlock scenario
- [ ] Create: Two transactions that deadlock
- [ ] Verify: Deadlock occurs
- [ ] Configure: Deadlock detection
- [ ] Test: Deadlock resolution
- [ ] Analyze: How to prevent deadlocks?
- [ ] Document: Deadlock prevention strategies

### Optimistic vs Pessimistic Locking

- [ ] Implement: Optimistic locking (version field)
- [ ] Test: Concurrent updates với optimistic locking
- [ ] Verify: OptimisticLockException on conflict
- [ ] Implement: Pessimistic locking (@Lock)
- [ ] Test: Concurrent updates với pessimistic locking
- [ ] Verify: Second transaction waits
- [ ] Compare: Performance của optimistic vs pessimistic
- [ ] Document: When to use which?

### Data Consistency Scenarios

- [ ] Scenario: Update balance - deduct và add
- [ ] Implement: Transaction để ensure atomicity
- [ ] Test: Rollback on failure
- [ ] Scenario: Read balance while updating
- [ ] Test: Consistency với different isolation levels
- [ ] Document: Consistency guarantees

### ACID Properties Verification

- [ ] Test: Atomicity - transaction rollback
- [ ] Test: Consistency - constraints enforcement
- [ ] Test: Isolation - concurrent transactions
- [ ] Test: Durability - data persistence after commit
- [ ] Document: ACID verification results

---

## Review TODOs

### Self-Evaluation

- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Schema design exercises hoàn thành?
- [ ] Review: Tất cả SQL optimization exercises hoàn thành?
- [ ] Review: Tất cả Spring Boot tasks hoàn thành?
- [ ] Review: Tất cả Scaling tasks hoàn thành?
- [ ] Rate: Database schema design skills (1-10)
- [ ] Rate: SQL optimization skills (1-10)
- [ ] Rate: Query performance tuning (1-10)
- [ ] Rate: Understanding của transactions (1-10)
- [ ] Identify: 3 database concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### Schema Review

- [ ] Review: Wallet System schema
- [ ] Check: Normalization appropriate?
- [ ] Check: Indexes optimal?
- [ ] Check: Data types correct?
- [ ] Identify: 3 potential improvements
- [ ] Propose: Schema optimizations
- [ ] Compare: Schema vs industry best practices
- [ ] Document: Schema review findings

### Query Performance Review

- [ ] Review: All optimized queries
- [ ] Verify: All queries use indexes
- [ ] Verify: No full table scans
- [ ] Verify: Performance targets met
- [ ] Identify: 3 queries that could be improved further
- [ ] Document: Query performance review

### Code Review

- [ ] Review: Spring Boot database code
- [ ] Check: Connection pool configuration optimal?
- [ ] Check: No N+1 queries?
- [ ] Check: Transaction boundaries correct?
- [ ] Check: Error handling appropriate?
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Knowledge Check

- [x] Explain: B-tree index - how it works (viết 1 paragraph, không xem notes)  
  → B-tree index là một cây cân bằng, mỗi node chứa nhiều key đã được sắp xếp và pointer tới child nodes; khi tìm kiếm,
  database bắt đầu từ root, so sánh key rồi đi xuống nhánh phù hợp cho tới leaf node, nên các thao tác tìm
  kiếm/insert/delete đều ~O(log n) và tree luôn được tự cân bằng để giữ chiều cao thấp, giúp truy vấn ổn định trên tập
  dữ liệu lớn.
- [x] Explain: Composite index - column order matters (viết explanation, không xem notes)  
  → Composite index được sắp xếp theo thứ tự cột (A, B, C), nên chỉ được dùng hiệu quả nếu điều kiện WHERE match từ trái
  sang phải (A, A+B, A+B+C); nếu query chỉ filter theo B hoặc C mà bỏ A thì engine khó tận dụng index, do đó phải chọn
  order sao cho cột có selectivity cao và hay xuất hiện trong equality conditions đứng trước.
- [x] Explain: Query execution plan - how to read (viết guide, không xem notes)  
  → Khi đọc execution plan, tập trung vào: loại scan/join (table scan, index scan/seek, nested loop, hash/merge join),
  index nào đang được dùng (`key`), số rows ước tính phải đọc (`rows`), và phần `Extra/Notes` (vd: `Using where`,
  `Using filesort`, `Using temporary`); mục tiêu là tránh full table scan trên bảng lớn, giảm rows phải scan và đảm bảo
  các JOIN/WHERE đang hit đúng index.
- [x] Explain: Transaction isolation levels - differences (viết comparison, không xem notes)  
  → READ UNCOMMITTED cho phép dirty read; READ COMMITTED cấm dirty read nhưng vẫn có non-repeatable read; REPEATABLE
  READ cấm dirty read và non-repeatable read nhưng có thể còn phantom read (tùy engine/MVCC); SERIALIZABLE chặt nhất,
  loại bỏ cả phantom read bằng cách gần như serialize các transaction, đổi lại throughput giảm và lock/conflict tăng
  mạnh.
- [x] Explain: Read replica - consistency trade-offs (viết analysis, không xem notes)  
  → Read replica giúp scale đọc và giảm tải master nhưng luôn có replication lag → eventual consistency; sau khi ghi vào
  master, nếu đọc ngay từ replica có thể không thấy dữ liệu mới, nên các read critical (balance, payment) thường phải
  đọc từ master hoặc dùng chiến lược sticky session/route theo lag, chấp nhận trade-off giữa performance, availability
  và consistency.
- [x] Solve: Query SELECT * FROM orders WHERE user_id = ? AND status = ? - design index  
  → Index hợp lý: `CREATE INDEX idx_orders_user_status ON orders(user_id, status);` với `user_id` (thường selective hơn)
  đặt trước và `status` sau, cho phép index seek theo cả hai điều kiện WHERE và có thể thành covering index nếu query
  chỉ đọc một vài cột.
- [x] Solve: Calculate connection pool size for 4-core server, 1 database  
  → Dùng công thức: `connections = (core_count * 2) + effective_spindle_count` → với 4 core và 1 spindle:
  `connections = (4 * 2) + 1 = 9`; thực tế có thể cấu hình khoảng 10–15 connections rồi dựa vào metrics (active/idle,
  wait time) để tune thêm.
- [x] Verify: Answers có đúng không?
- [x] Document: Knowledge gaps

### Reflection

- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 database concepts còn confuse
- [ ] Write: 1 mistake đã làm và lesson learned
- [ ] Write: Confidence level cho Week 4 (1-10)
- [ ] Compare: Week 2 vs Week 3 progress
- [ ] Plan: Preparation cho Week 4 (Database Sharding)
- [ ] Set: Goals cho Week 4
- [ ] Document: Week 3 reflection (500 words)

### Mentor Questions (Answer these - be critical)

- [x] Q1: Bạn có 1 query chạy 5 giây. Làm sao bạn optimize? (viết step-by-step process)  
  → B1: Dùng `EXPLAIN/EXPLAIN ANALYZE` xem execution plan (full table scan? index nào đang dùng?).  
  → B2: Đo thời gian thực thi để có baseline.  
  → B3: Phân tích WHERE/JOIN/ORDER BY để tìm thiếu index, JOIN order kém, LIMIT/OFFSET lớn, subquery không cần thiết.  
  → B4: Thiết kế/tối ưu index (ưu tiên cột trong WHERE/JOIN, composite index với order đúng, covering index nếu phù
  hợp).  
  → B5: Chạy lại EXPLAIN + benchmark, verify rows scan giảm và index được dùng.  
  → B6: Nếu vẫn chậm, cân nhắc rewrite (subquery → JOIN, cursor-based pagination, materialized view, archive bớt dữ liệu
  cũ).  
  → B7: Bật slow query log, monitor trong production và tiếp tục tối ưu dựa trên số liệu thực.
- [x] Q2: Composite index (user_id, status, created_at) - query WHERE status = ? AND created_at > ? (không có user_id) -
  index có được dùng không? Tại sao? (viết analysis)  
  → Index `(user_id, status, created_at)` được sort theo `user_id` trước nên query không có điều kiện trên `user_id` sẽ
  không tận dụng tốt index này (bị skip cột đầu), đa số engine sẽ phải scan phần lớn index hoặc bỏ qua; nếu query thường
  chỉ filter theo `status` + `created_at` thì nên tạo thêm index `(status, created_at)` chuyên cho pattern đó.
- [x] Q3: Read replica có replication lag 2 giây. User vừa update balance, ngay lập tức query balance từ replica - có
  thấy update không? Làm sao fix? (viết solution)  
  → Với lag 2s thì gần như chắc chắn **không** thấy balance mới trên replica ngay sau khi update; để fix: (1) các read
  critical (balance, payment) đọc từ master, (2) dùng sticky session/flag sau khi write để route một số request tiếp
  theo sang master, (3) route theo lag (nếu lag > threshold thì đọc master), hoặc (4) dùng version/timestamp để detect
  stale read và fallback sang master.
- [x] Q4: Connection pool size = 100, nhưng có 200 concurrent requests. Chuyện gì xảy ra? Làm sao fix? (viết analysis)  
  → 100 request đầu mượn được connection, 100 request còn lại sẽ block chờ; nếu thời gian chờ vượt `connection-timeout`
  thì ném exception (timeout), gây lỗi hàng loạt và có thể làm server overload thêm. Cách xử lý: tối ưu query để giữ
  connection ngắn hơn, điều chỉnh pool size (nếu DB chịu được), giới hạn concurrency ở app layer (queue, rate limit), và
  monitor active/idle connections để tune thay vì chỉ tăng mù quáng.
- [x] Q5: Transaction isolation level = SERIALIZABLE - có phải luôn tốt nhất không? Tại sao không? (viết trade-off
  analysis)  
  → SERIALIZABLE cho isolation cao nhất (loại bỏ dirty/non‑repeatable/phantom read) nhưng đánh đổi bằng việc tăng lock,
  dễ deadlock, throughput giảm mạnh và latency cao, nên không phù hợp cho hệ thống high-concurrency; thường chỉ dùng cho
  một số luồng cực kỳ critical, còn lại dùng READ COMMITTED/REPEATABLE READ + logic ở application sẽ cân bằng hơn giữa
  correctness và performance.
- [x] Q6: Bạn có bảng 100M records, query WHERE created_at > ? - làm sao optimize? (viết optimization strategy)  
  → Chiến lược: (1) tạo index trên `created_at` (hoặc composite index nếu query còn filter thêm), (2) partition bảng
  theo range thời gian để DB chỉ scan đúng partition (partition pruning), (3) archive dữ liệu quá cũ sang bảng khác để
  bảng chính nhỏ lại, (4) dùng covering index nếu chỉ cần vài cột, (5) nếu là analytics nặng thì cân nhắc materialized
  view/OLAP store; luôn đo lại time sau mỗi bước.
- [x] Review: Answers có đủ depth và practical không?
- [x] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Schema & Modeling TODOs: ✅ Complete với diagrams
- [ ] Tất cả SQL Optimization TODOs: ✅ Complete với performance measurements
- [ ] Tất cả Spring Boot + DB TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Scaling & Replication TODOs: ✅ Complete
- [ ] Tất cả Consistency & Transaction TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Schema diagrams saved: ✅ Yes
- [ ] Query performance benchmarks saved: ✅ Yes
- [ ] Ready for Week 4: ✅ Yes

---

> **Mentor Final Check**: Database performance là foundation. Nếu bạn không thể optimize queries, design schemas, và
> tune databases, bạn không thể build scalable systems. Hãy honest: Bạn có thể identify và fix slow queries chưa? Bạn có
> thể design database schema cho 100M+ records chưa? Nếu chưa, làm lại. Production systems không chấp nhận "query chạy
> được là OK".
