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

  **Selectivity** (độ chọn lọc) là một metric đo lường khả năng một index có thể **lọc bớt** số lượng rows cần phải scan khi thực thi query.

  ### 📐 Công thức cơ bản:

  ```
  Selectivity = distinct_values / total_rows
  ```

  **Giá trị selectivity:**
  - **Càng gần 1.0** → càng tốt (gần như unique, mỗi giá trị chỉ xuất hiện 1 lần)
  - **Càng gần 0** → càng kém (nhiều rows có cùng giá trị, index ít hiệu quả)

  ### 📊 Ví dụ cụ thể:

  Bảng `users` có **1,000,000 rows**:

  | Column    | Distinct values | Selectivity | Đánh giá |
  |-----------|-----------------|-------------|----------|
  | `user_id` | 1,000,000       | 1.0         | ✅ Tuyệt vời - unique |
  | `email`   | 1,000,000       | 1.0         | ✅ Tuyệt vời - unique |
  | `phone`   | 950,000         | 0.95        | ✅ Rất tốt |
  | `country` | 50              | 0.00005     | ⚠️ Thấp - mỗi nước có ~20k users |
  | `gender`  | 2               | 0.000002    | ❌ Rất thấp - ~500k rows mỗi giá trị |
  | `status`  | 3               | 0.000003    | ❌ Rất thấp - ~333k rows mỗi giá trị |

  ### 💡 Tại sao selectivity quan trọng?

  Selectivity quyết định **index có đáng để tạo hay không**:

  - **High selectivity (0.1 - 1.0)**: Index rất hiệu quả, giúp query nhanh đáng kể
  - **Medium selectivity (0.01 - 0.1)**: Index có thể hữu ích, nhưng cần đánh giá kỹ
  - **Low selectivity (< 0.01)**: Index thường không hiệu quả, có thể bị optimizer bỏ qua

  ### 🎯 Rule of thumb:

  - **Selectivity > 0.1**: Nên tạo index nếu column thường xuyên xuất hiện trong WHERE
  - **Selectivity < 0.01**: Thường không nên index riêng lẻ, nhưng có thể dùng trong composite index
    
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

  ### 📊 Ví dụ cụ thể với số liệu:

  **Bảng `orders` có 1,000,000 rows:**

  | status  | Số rows | Tỷ lệ |
  |---------|---------|-------|
  | SUCCESS | 980,000 | 98%   |
  | FAILED  | 10,000  | 1%    |
  | PENDING | 10,000  | 1%    |
  | **Tổng** | **1,000,000** | **100%** |

  ### 🔢 Cách tính Selectivity:

  **❌ Cách tính CŨ (không dùng histogram):**
  ```
  Selectivity = distinct_values / total_rows
             = 3 / 1,000,000
             = 0.000003
  ```
  → Rất thấp! Optimizer nghĩ index không hiệu quả.

  **Nhưng cách này SAI vì:**
  - Giả định mỗi giá trị có số rows bằng nhau
  - Nghĩa là: SUCCESS = 333,333 rows, FAILED = 333,333 rows, PENDING = 333,333 rows
  - **Nhưng thực tế không phải vậy!**

  **✅ Cách tính MỚI (dùng histogram):**

  Histogram lưu **phân bố thực tế**:
  ```
  SUCCESS = 98% (980,000 rows)
  FAILED  = 1%  (10,000 rows)
  PENDING = 1%  (10,000 rows)
  ```

  Khi query `WHERE status = 'PENDING'`:
  ```
  Selectivity = số rows PENDING / total_rows
             = 10,000 / 1,000,000
             = 0.01
             = 1%
  ```

  **So sánh:**
  - ❌ Cách cũ: Selectivity = 0.000003 (nghĩ là rất thấp, không nên dùng index)
  - ✅ Cách mới: Selectivity = 0.01 (1% - đủ cao để dùng index hiệu quả!)

  ### 💡 Tại sao quan trọng?

  **Query:**
  ```sql
  SELECT * FROM orders WHERE status = 'PENDING';
  ```

  **Với cách tính cũ (không có histogram):**
  - Optimizer nghĩ: "Có 3 giá trị, mỗi giá trị ~333k rows"
  - Estimate: Query sẽ trả về ~333,333 rows (33%)
  - Quyết định: "Quá nhiều rows, không nên dùng index, full table scan tốt hơn"
  - ❌ **Sai!** Thực tế chỉ có 10,000 rows (1%)

  **Với histogram:**
  - Optimizer biết: "PENDING chỉ có 1% rows"
  - Estimate: Query sẽ trả về ~10,000 rows (1%)
  - Quyết định: "Ít rows, nên dùng index"
  - ✅ **Đúng!** Index sẽ rất hiệu quả

  ### 🎯 Kết luận:

  **Selectivity thực tế = số rows match / total_rows**

  Không phải = distinct_values / total_rows

  Histogram giúp optimizer biết **phân bố thực tế** của từng giá trị, không phải giả định đều nhau.
    
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

    ### 🎯 Covering Index là gì?

    **Covering index** là index chứa **TẤT CẢ** các columns mà query cần, bao gồm:
    - Columns trong WHERE clause (để filter)
    - Columns trong SELECT clause (để trả về)
    - Columns trong ORDER BY/GROUP BY (nếu có)

    Khi index "cover" toàn bộ query → Database engine chỉ cần đọc **index pages**, không cần quay lại **table pages** → giảm I/O đáng kể.

    ### 📊 Ví dụ cụ thể:

    **Bảng `orders` (10M rows):**
    ```sql
    CREATE TABLE orders (
        id BIGINT PRIMARY KEY,
        user_id BIGINT,
        status VARCHAR(20),
        amount DECIMAL(10,2),
        created_at TIMESTAMP,
        -- ... nhiều columns khác
    );
    ```

    **Query thường xuyên:**
    ```sql
    SELECT user_id, status, amount
    FROM orders
    WHERE user_id = 12345
    AND status = 'PENDING';
    ```

    **❌ Index không covering:**
    ```sql
    CREATE INDEX idx_orders_user_status ON orders(user_id, status);
    ```
    - Index chỉ có `user_id, status` → phải quay lại table để lấy `amount`
    - Mỗi row match → 1 random I/O để đọc table page
    - Với 100 rows match → 100 random I/Os

    **✅ Index covering:**
    ```sql
    CREATE INDEX idx_orders_user_status_amount ON orders(user_id, status, amount);
    ```
    - Index có đủ `user_id, status, amount` → **index-only scan**
    - Không cần quay lại table
    - Chỉ đọc index pages (sequential I/O) → nhanh hơn 10-100x

    ### 💡 Lợi ích của Covering Index:

    1. **Giảm I/O**: Không cần đọc table pages
    2. **Giảm latency**: Sequential read index thay vì random read table
    3. **Giảm lock contention**: Ít phải lock table pages
    4. **Tăng throughput**: Đặc biệt quan trọng với read-heavy workload

    ### ⚠️ Trade-offs:

    - **Index size lớn hơn**: Thêm columns vào index → index chiếm nhiều disk/RAM hơn
    - **Write overhead**: Mỗi INSERT/UPDATE phải update nhiều columns trong index
    - **Maintenance cost**: Index lớn → rebuild/reorganize tốn thời gian hơn

    ### 🎯 Khi nào nên dùng Covering Index?

    ✅ **Nên dùng khi:**
    - Query được gọi rất thường xuyên (hot path)
    - Table có nhiều columns nhưng query chỉ cần vài columns
    - Read-heavy workload (nhiều SELECT, ít UPDATE)
    - Query performance là bottleneck

    ❌ **Không nên dùng khi:**
    - Query không được gọi thường xuyên
    - Write-heavy workload (nhiều INSERT/UPDATE)
    - Index quá lớn (vượt quá RAM available)
    - Columns trong SELECT thay đổi thường xuyên

    ### 🔍 Cách kiểm tra Covering Index:

    **MySQL:**
    ```sql
    EXPLAIN SELECT user_id, status, amount 
    FROM orders 
    WHERE user_id = 12345;
    ```
    - Nếu thấy `Extra: Using index` → đây là covering index!

    **PostgreSQL:**
    ```sql
    EXPLAIN (ANALYZE, BUFFERS) 
    SELECT user_id, status, amount 
    FROM orders 
    WHERE user_id = 12345;
    ```
    - Nếu thấy `Index Only Scan` → covering index đang được dùng
- [x] Đọc về "partial index" - conditional indexes
    - Index chỉ tạo trên subset rows với `WHERE` (vd: `status = 'PENDING'`) → index nhỏ, tập trung cho hot subset, giảm
      overhead write.

    ### 🎯 Partial Index là gì?

    **Partial index** (hay **filtered index**, **conditional index**) là index chỉ được tạo trên **một phần** của bảng, dựa trên điều kiện WHERE.

    Thay vì index toàn bộ rows → chỉ index những rows thỏa mãn điều kiện → index nhỏ hơn, nhanh hơn, ít overhead hơn.

    ### 📊 Ví dụ cụ thể:

    **Bảng `orders` (10M rows):**
    ```sql
    CREATE TABLE orders (
        id BIGINT PRIMARY KEY,
        user_id BIGINT,
        status VARCHAR(20),  -- PENDING, COMPLETED, CANCELLED
        amount DECIMAL(10,2),
        created_at TIMESTAMP
    );
    ```

    **Phân bố dữ liệu:**
    - `PENDING`: 50,000 rows (0.5%) - cần query thường xuyên
    - `COMPLETED`: 9,500,000 rows (95%)
    - `CANCELLED`: 450,000 rows (4.5%)

    **Query thường xuyên:**
    ```sql
    SELECT * FROM orders 
    WHERE status = 'PENDING' 
    AND created_at > '2024-01-01'
    ORDER BY created_at;
    ```

    **❌ Full index (không hiệu quả):**
    ```sql
    CREATE INDEX idx_orders_status_created ON orders(status, created_at);
    ```
    - Index 10M rows
    - Chỉ 0.5% rows (PENDING) được query
    - 99.5% index entries không bao giờ được dùng
    - Tốn disk, RAM, và write overhead

    **✅ Partial index (hiệu quả):**
    ```sql
    -- PostgreSQL
    CREATE INDEX idx_orders_pending_created 
    ON orders(created_at) 
    WHERE status = 'PENDING';

    -- SQL Server
    CREATE INDEX idx_orders_pending_created 
    ON orders(created_at) 
    WHERE status = 'PENDING';
    ```
    - Index chỉ 50,000 rows (0.5% của bảng)
    - Nhỏ hơn 200x so với full index
    - Query nhanh hơn vì index nhỏ
    - Write overhead giảm đáng kể

    ### 💡 Lợi ích của Partial Index:

    1. **Index size nhỏ**: Chỉ index subset rows → tiết kiệm disk/RAM
    2. **Query nhanh hơn**: Index nhỏ → scan nhanh hơn
    3. **Write overhead thấp**: Chỉ update index khi row thỏa điều kiện
    4. **Maintenance nhanh**: Rebuild/reorganize index nhỏ nhanh hơn nhiều

    ### 📋 Use Cases phổ biến:

    **1. Index cho "active" records:**
    ```sql
    -- Chỉ index các orders đang pending
    CREATE INDEX idx_orders_pending 
    ON orders(user_id, created_at) 
    WHERE status = 'PENDING';
    ```

    **2. Index cho "recent" data:**
    ```sql
    -- Chỉ index data trong 30 ngày gần nhất
    CREATE INDEX idx_orders_recent 
    ON orders(user_id) 
    WHERE created_at > CURRENT_DATE - INTERVAL '30 days';
    ```

    **3. Index cho "non-null" values:**
    ```sql
    -- Chỉ index rows có email (bỏ qua NULL)
    CREATE INDEX idx_users_email 
    ON users(email) 
    WHERE email IS NOT NULL;
    ```

    **4. Index cho "specific value":**
    ```sql
    -- Chỉ index các transactions failed (để audit)
    CREATE INDEX idx_transactions_failed 
    ON transactions(created_at) 
    WHERE status = 'FAILED';
    ```

    ### ⚠️ Lưu ý quan trọng:

    - **MySQL không hỗ trợ** partial index trực tiếp (chỉ PostgreSQL, SQL Server, Oracle)
    - **MySQL workaround**: Dùng generated column + index:
      ```sql
      ALTER TABLE orders 
      ADD COLUMN is_pending TINYINT(1) 
      GENERATED ALWAYS AS (status = 'PENDING') STORED;
      
      CREATE INDEX idx_orders_pending ON orders(created_at) WHERE is_pending = 1;
      ```
      (Lưu ý: MySQL 8.0+ mới hỗ trợ functional index với WHERE)

    - **Query phải match điều kiện**: Query phải có điều kiện giống WHERE clause của index, nếu không optimizer không dùng index

    ### 🎯 Khi nào nên dùng Partial Index?

    ✅ **Nên dùng khi:**
    - Chỉ query một subset nhỏ của bảng (vd: status = 'PENDING')
    - Subset này được query rất thường xuyên
    - Bảng lớn nhưng chỉ một phần nhỏ là "hot"
    - Muốn giảm index size và write overhead

    ❌ **Không nên dùng khi:**
    - Query nhiều giá trị khác nhau của column
    - Subset quá lớn (> 50% rows)
    - Điều kiện WHERE thay đổi thường xuyên
- [x] Đọc về "unique index" vs "non-unique index" - performance difference
    - Unique index enforce uniqueness và cho phép optimizer giả định tối đa 1 row match, đôi khi cho plan tốt hơn; nhưng
      mọi write phải check unique constraint → thêm chút overhead.

    ### 🎯 Unique Index vs Non-Unique Index

    **Unique Index** đảm bảo không có 2 rows nào có cùng giá trị cho column(s) được index. Ngoài việc enforce uniqueness, nó còn có những khác biệt về performance so với non-unique index.

    ### 📊 So sánh chi tiết:

    | Tiêu chí | Unique Index | Non-Unique Index |
    |----------|--------------|------------------|
    | **Uniqueness** | ✅ Enforce (không cho phép duplicate) | ❌ Cho phép duplicate |
    | **Selectivity** | Luôn = 1.0 (hoặc gần 1.0) | Có thể < 1.0 |
    | **Query optimization** | Optimizer biết chắc chỉ có 0-1 row match | Optimizer phải estimate số rows |
    | **Lookup performance** | Có thể dừng sớm khi tìm thấy 1 row | Phải scan hết nếu có duplicate |
    | **Write overhead** | Phải check uniqueness (thêm I/O) | Không cần check uniqueness |
    | **Storage** | Tương tự non-unique | Tương tự unique |

    ### 💡 Performance Differences:

    **1. Query Optimization:**

    **Unique Index:**
    ```sql
    -- Unique index trên email
    CREATE UNIQUE INDEX idx_users_email ON users(email);

    SELECT * FROM users WHERE email = 'user@example.com';
    ```
    - Optimizer **biết chắc** chỉ có tối đa 1 row
    - Có thể chọn plan tối ưu hơn (vd: không cần sort nếu đã có 1 row)
    - Có thể dừng scan ngay khi tìm thấy 1 row

    **Non-Unique Index:**
    ```sql
    -- Non-unique index trên status
    CREATE INDEX idx_orders_status ON orders(status);

    SELECT * FROM orders WHERE status = 'PENDING';
    ```
    - Optimizer phải **estimate** số rows (có thể hàng nghìn)
    - Phải scan hết tất cả rows match
    - Plan có thể kém tối ưu hơn

    **2. JOIN Performance:**

    **Unique Index (Foreign Key):**
    ```sql
    -- users.id là PRIMARY KEY (unique)
    SELECT o.*, u.name 
    FROM orders o
    JOIN users u ON o.user_id = u.id;
    ```
    - Optimizer biết mỗi `user_id` chỉ match 1 row trong `users`
    - Có thể chọn nested loop join hiệu quả
    - Không cần build hash table lớn

    **Non-Unique Index:**
    ```sql
    -- status không unique
    SELECT o1.*, o2.*
    FROM orders o1
    JOIN orders o2 ON o1.status = o2.status;
    ```
    - Optimizer phải estimate cardinality
    - Có thể chọn hash join thay vì nested loop
    - Performance kém hơn nếu estimate sai

    **3. Write Overhead:**

    **Unique Index:**
    ```sql
    INSERT INTO users (email, name) VALUES ('new@example.com', 'New User');
    ```
    - Phải **check uniqueness** trước khi insert
    - Thêm 1 index lookup để verify không có duplicate
    - Nếu có duplicate → error, rollback
    - **Overhead**: +1 index read per write

    **Non-Unique Index:**
    ```sql
    INSERT INTO orders (status, amount) VALUES ('PENDING', 100);
    ```
    - Không cần check uniqueness
    - Chỉ cần insert vào index
    - **Overhead**: Chỉ 1 index write

    ### 📋 Ví dụ thực tế:

    **Scenario: Email lookup**

    ```sql
    -- Unique index
    CREATE UNIQUE INDEX idx_users_email ON users(email);
    
    -- Query
    SELECT id, name FROM users WHERE email = 'user@example.com';
    ```

    **Execution với Unique Index:**
    1. Index seek đến `email = 'user@example.com'`
    2. Tìm thấy 1 row → **dừng ngay**
    3. Trả về kết quả
    4. **Total I/O**: 2-3 pages (index root → leaf → table

    **Execution với Non-Unique Index (giả sử):**
    1. Index seek đến `email = 'user@example.com'`
    2. Phải scan tiếp để tìm tất cả rows match (dù chỉ có 1)
    3. Trả về kết quả
    4. **Total I/O**: 3-4 pages (nhiều hơn một chút)

    ### ⚠️ Trade-offs:

    **Unique Index:**
    - ✅ Query nhanh hơn (optimizer biết chắc 1 row)
    - ✅ Data integrity (không có duplicate)
    - ❌ Write chậm hơn (phải check uniqueness)
    - ❌ Không thể insert duplicate (có thể là feature hoặc bug tùy use case)

    **Non-Unique Index:**
    - ✅ Write nhanh hơn (không cần check)
    - ✅ Cho phép duplicate (linh hoạt hơn)
    - ❌ Query có thể chậm hơn (phải scan nhiều rows)
    - ❌ Không đảm bảo data integrity

    ### 🎯 Best Practices:

    1. **Dùng Unique Index khi:**
       - Column phải unique (email, phone, SSN)
       - Query thường xuyên lookup by unique value
       - Data integrity là ưu tiên

    2. **Dùng Non-Unique Index khi:**
       - Column có thể có duplicate (status, category)
       - Write performance quan trọng hơn
       - Cần linh hoạt cho phép duplicate

    3. **Primary Key:**
       - Luôn là unique index (implicit)
       - Tốt nhất cho lookup performance
       - Nên dùng cho foreign key references
- [x] Đọc về "index maintenance" - INSERT/UPDATE/DELETE overhead
    - Mỗi thay đổi dữ liệu phải update **tất cả indexes liên quan**; nhiều index → write cost tăng (CPU + I/O), đặc biệt
      trên bảng lớn.

    ### 🎯 Index Maintenance là gì?

    **Index maintenance** là quá trình database engine tự động **cập nhật index** mỗi khi có thay đổi dữ liệu (INSERT, UPDATE, DELETE). Đây là overhead không thể tránh khỏi khi dùng index.

    ### 📊 Chi phí của Index Maintenance:

    **Mỗi index = 1 cây B-tree riêng** → mỗi thay đổi data phải update tất cả indexes liên quan.

    **Ví dụ bảng `orders` có 5 indexes:**
    ```sql
    CREATE TABLE orders (
        id BIGINT PRIMARY KEY,           -- Index 1: PRIMARY KEY
        user_id BIGINT,                  -- Index 2: idx_orders_user
        status VARCHAR(20),              -- Index 3: idx_orders_status
        created_at TIMESTAMP,            -- Index 4: idx_orders_created
        -- Composite index
        INDEX idx_orders_user_status (user_id, status)  -- Index 5
    );
    ```

    **Khi INSERT 1 row mới:**
    ```
    1. Insert row vào table          → 1 write
    2. Update PRIMARY KEY index      → 1 write
    3. Update idx_orders_user       → 1 write
    4. Update idx_orders_status      → 1 write
    5. Update idx_orders_created     → 1 write
    6. Update idx_orders_user_status → 1 write
    ──────────────────────────────────────────
    Total: 6 writes (1 table + 5 indexes)
    ```

    **Khi UPDATE `status`:**
    ```
    1. Update row trong table                    → 1 write
    2. Update idx_orders_status (delete old)     → 1 write
    3. Update idx_orders_status (insert new)     → 1 write
    4. Update idx_orders_user_status (delete)    → 1 write
    5. Update idx_orders_user_status (insert)     → 1 write
    ────────────────────────────────────────────────────
    Total: 5 writes (1 table + 4 index updates)
    ```

    **Khi DELETE 1 row:**
    ```
    1. Delete row từ table          → 1 write
    2. Delete từ PRIMARY KEY        → 1 write
    3. Delete từ idx_orders_user   → 1 write
    4. Delete từ idx_orders_status  → 1 write
    5. Delete từ idx_orders_created → 1 write
    6. Delete từ composite index     → 1 write
    ──────────────────────────────────────────
    Total: 6 writes (1 table + 5 indexes)
    ```

    ### 💡 Impact thực tế:

    **Bảng nhỏ (< 1M rows):**
    - Overhead nhỏ, không đáng kể
    - 5-10 indexes vẫn OK

    **Bảng lớn (10M+ rows):**
    - Mỗi index update = random I/O
    - 10 indexes → 10x write overhead
    - Có thể làm INSERT chậm 10-50x so với không có index

    **High-write workload:**
    - Nhiều indexes → bottleneck ở disk I/O
    - Có thể cần SSD để đảm bảo performance
    - Hoặc giảm số lượng indexes

    ### 📋 Các yếu tố ảnh hưởng Index Maintenance Cost:

    **1. Số lượng indexes:**
    - Mỗi index = thêm 1 write per operation
    - 10 indexes → 10x overhead so với 1 index

    **2. Index size:**
    - Index lớn → B-tree sâu hơn → nhiều pages phải update
    - Composite index với nhiều columns → lớn hơn → chậm hơn

    **3. Index type:**
    - **Clustered index (PK)**: Update nhanh hơn (data và index cùng chỗ)
    - **Non-clustered index**: Update chậm hơn (phải tìm row trong table)

    **4. Update pattern:**
    - **Update indexed column**: Phải update index (delete + insert)
    - **Update non-indexed column**: Không cần update index

    **5. Fragmentation:**
    - Index bị fragmented → update chậm hơn
    - Cần rebuild/reorganize định kỳ

    ### ⚠️ Trade-offs:

    **Nhiều indexes:**
    - ✅ Query nhanh hơn (nhiều access paths)
    - ❌ Write chậm hơn (nhiều index phải update)
    - ❌ Tốn disk/RAM (mỗi index chiếm space)
    - ❌ Maintenance phức tạp hơn

    **Ít indexes:**
    - ✅ Write nhanh hơn
    - ✅ Tiết kiệm disk/RAM
    - ❌ Query có thể chậm (phải full scan)
    - ❌ Ít access paths

    ### 🎯 Best Practices để giảm Index Maintenance Cost:

    **1. Chỉ tạo index cần thiết:**
    ```sql
    -- ❌ Không nên: Index mọi column
    CREATE INDEX idx_orders_col1 ON orders(col1);
    CREATE INDEX idx_orders_col2 ON orders(col2);
    CREATE INDEX idx_orders_col3 ON orders(col3);
    -- ... 10 indexes

    -- ✅ Nên: Chỉ index columns được query thường xuyên
    CREATE INDEX idx_orders_user_status ON orders(user_id, status);
    ```

    **2. Dùng composite index thay vì nhiều single-column indexes:**
    ```sql
    -- ❌ Không hiệu quả: 2 indexes riêng
    CREATE INDEX idx_orders_user ON orders(user_id);
    CREATE INDEX idx_orders_status ON orders(status);

    -- ✅ Hiệu quả hơn: 1 composite index
    CREATE INDEX idx_orders_user_status ON orders(user_id, status);
    ```

    **3. Tách bảng theo access pattern:**
    ```sql
    -- Hot data (thường xuyên update) → ít indexes
    CREATE TABLE orders_hot (
        id BIGINT PRIMARY KEY,
        user_id BIGINT,
        status VARCHAR(20)
        -- Chỉ 1-2 indexes
    );

    -- Cold data (ít update, nhiều read) → nhiều indexes OK
    CREATE TABLE orders_archive (
        id BIGINT PRIMARY KEY,
        user_id BIGINT,
        status VARCHAR(20),
        -- Có thể có nhiều indexes cho reporting
    );
    ```

    **4. Monitor index usage:**
    ```sql
    -- PostgreSQL: Xem index usage
    SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
    FROM pg_stat_user_indexes
    WHERE idx_scan = 0;  -- Indexes không được dùng

    -- MySQL: Xem index usage
    SELECT * FROM sys.schema_unused_indexes;
    ```

    **5. Rebuild indexes định kỳ:**
    ```sql
    -- Giảm fragmentation → update nhanh hơn
    -- PostgreSQL
    REINDEX INDEX idx_orders_user_status;

    -- MySQL
    ALTER TABLE orders ENGINE=InnoDB;  -- Rebuild tất cả indexes
    ```

    ### 📊 Tính toán Index Maintenance Cost:

    **Ví dụ:**
    - Bảng: 10M rows
    - Indexes: 5 indexes
    - Write rate: 1000 writes/second

    **Cost:**
    - Mỗi write = 1 table write + 5 index writes = 6 writes
    - Total: 1000 × 6 = **6,000 I/O operations/second**
    - Nếu mỗi I/O = 1ms → **6 seconds total** (quá chậm!)

    **Giải pháp:**
    - Giảm indexes xuống 2-3 → giảm 50% overhead
    - Dùng SSD → giảm I/O latency
    - Batch writes → giảm số lần update
- [x] Research: How many indexes is too many? (general rule)
    - Rule of thumb: small table: 3–5 index; medium: 5–7; large: 5–10 là đã phải rất cân nhắc; luôn dựa vào **read/write
      profile + index usage stats** để quyết.
- [x] Đọc về "index fragmentation" - when và how to rebuild
    - Nhiều INSERT/DELETE/UPDATE gây pages rỗng/không liên tục → fragmentation, làm scan chậm.
    - Rebuild/reorganize index khi fragmentation cao (vd > ~30%) hoặc sau bulk operations lớn.

    ### 🎯 Index Fragmentation là gì?

    **Index fragmentation** xảy ra khi các **index pages** không còn được sắp xếp liên tục trên disk, hoặc có nhiều **empty space** trong pages do DELETE/UPDATE.

    Fragmentation làm giảm performance vì:
    - **Sequential scan** trở thành **random I/O** (phải nhảy qua nhiều pages)
    - **Page utilization** thấp (nhiều empty space → đọc nhiều pages hơn)
    - **Cache efficiency** giảm (ít pages fit trong buffer pool)

    ### 📊 Các loại Fragmentation:

    **1. External Fragmentation (Logical Fragmentation):**
    - Index pages không liên tục trên disk
    - Sequential scan phải nhảy qua nhiều vị trí
    - **Impact**: Random I/O thay vì sequential I/O

    **2. Internal Fragmentation (Page Density):**
    - Pages có nhiều empty space (do DELETE)
    - Phải đọc nhiều pages hơn để lấy cùng số rows
    - **Impact**: Tăng I/O operations

    ### 💡 Nguyên nhân Fragmentation:

    **1. DELETE operations:**
    ```sql
    -- Xóa nhiều rows → để lại empty space trong pages
    DELETE FROM orders WHERE status = 'CANCELLED';
    ```

    **2. UPDATE operations:**
    ```sql
    -- Update indexed column → delete old entry + insert new entry
    UPDATE orders SET status = 'COMPLETED' WHERE id = 123;
    -- Có thể làm index entries không còn sorted
    ```

    **3. Random INSERTs:**
    ```sql
    -- Insert với key không sequential → pages không liên tục
    INSERT INTO orders (id, user_id) VALUES (50000, 123);
    INSERT INTO orders (id, user_id) VALUES (100, 456);
    INSERT INTO orders (id, user_id) VALUES (99999, 789);
    ```

    **4. Bulk operations:**
    ```sql
    -- Import nhiều data → pages có thể không optimal
    LOAD DATA INFILE 'orders.csv' INTO TABLE orders;
    ```

    ### 📋 Cách đo Fragmentation:

    **SQL Server:**
    ```sql
    SELECT 
        OBJECT_NAME(object_id) AS TableName,
        name AS IndexName,
        avg_fragmentation_in_percent,
        page_count
    FROM sys.dm_db_index_physical_stats(
        DB_ID(), 
        OBJECT_ID('orders'), 
        NULL, 
        NULL, 
        'DETAILED'
    )
    WHERE avg_fragmentation_in_percent > 30;
    ```

    **PostgreSQL:**
    ```sql
    -- Kiểm tra index size và bloat
    SELECT 
        schemaname,
        tablename,
        indexname,
        pg_size_pretty(pg_relation_size(indexname::regclass)) AS index_size,
        idx_scan AS index_scans
    FROM pg_stat_user_indexes
    WHERE schemaname = 'public'
    ORDER BY pg_relation_size(indexname::regclass) DESC;
    ```

    **MySQL:**
    ```sql
    -- Kiểm tra index statistics
    SHOW INDEX FROM orders;
    
    -- Hoặc dùng INFORMATION_SCHEMA
    SELECT 
        TABLE_NAME,
        INDEX_NAME,
        CARDINALITY,
        INDEX_TYPE
    FROM INFORMATION_SCHEMA.STATISTICS
    WHERE TABLE_SCHEMA = 'your_database'
    AND TABLE_NAME = 'orders';
    ```

    ### 🔧 Cách fix Fragmentation:

    **1. REBUILD Index (SQL Server):**
    ```sql
    -- Rebuild toàn bộ index (offline, tốn thời gian)
    ALTER INDEX idx_orders_user_status 
    ON orders REBUILD;
    
    -- Hoặc rebuild tất cả indexes
    ALTER INDEX ALL ON orders REBUILD;
    ```

    **2. REORGANIZE Index (SQL Server):**
    ```sql
    -- Reorganize index (online, nhanh hơn nhưng ít hiệu quả hơn)
    ALTER INDEX idx_orders_user_status 
    ON orders REORGANIZE;
    ```

    **3. REINDEX (PostgreSQL):**
    ```sql
    -- Rebuild một index
    REINDEX INDEX idx_orders_user_status;
    
    -- Rebuild tất cả indexes của một table
    REINDEX TABLE orders;
    
    -- Rebuild tất cả indexes của database
    REINDEX DATABASE your_database;
    ```

    **4. OPTIMIZE TABLE (MySQL):**
    ```sql
    -- Rebuild table và indexes
    OPTIMIZE TABLE orders;
    
    -- Hoặc rebuild index cụ thể
    ALTER TABLE orders DROP INDEX idx_orders_user_status;
    CREATE INDEX idx_orders_user_status ON orders(user_id, status);
    ```

    ### ⚠️ REBUILD vs REORGANIZE:

    | Tiêu chí | REBUILD | REORGANIZE |
    |----------|---------|------------|
    **Fragmentation threshold** | > 30% | 10-30% |
    **Operation** | Tạo lại index từ đầu | Sắp xếp lại pages |
    **Time** | Lâu (phút/giờ) | Nhanh (giây/phút) |
    **Online?** | Có thể offline | Thường online |
    **Effectiveness** | Rất hiệu quả (0% fragmentation) | Hiệu quả vừa (giảm fragmentation) |
    **Lock** | Có thể lock table | Ít lock hơn |
    **Disk space** | Cần 2x index size | Không cần thêm |

    ### 🎯 Khi nào nên Rebuild/Reorganize?

    **✅ Nên rebuild khi:**
    - Fragmentation > 30%
    - Sau bulk DELETE/UPDATE lớn
    - Index performance giảm đáng kể
    - Có maintenance window (offline OK)

    **✅ Nên reorganize khi:**
    - Fragmentation 10-30%
    - Cần online operation (không downtime)
    - Maintenance window ngắn
    - Fragmentation chưa quá nghiêm trọng

    **❌ Không cần rebuild khi:**
    - Fragmentation < 10%
    - Index ít được dùng
    - Table nhỏ (< 100K rows)
    - Không có performance issue

    ### 📊 Best Practices:

    **1. Schedule regular maintenance:**
    ```sql
    -- Chạy hàng tuần/tháng
    -- SQL Server
    ALTER INDEX ALL ON orders REORGANIZE;
    
    -- PostgreSQL
    REINDEX TABLE orders;
    ```

    **2. Monitor fragmentation:**
    - Set up alerts khi fragmentation > 30%
    - Track index performance metrics
    - Identify indexes cần rebuild thường xuyên

    **3. Rebuild during low traffic:**
    - Maintenance window (đêm, cuối tuần)
    - Hoặc dùng online rebuild (nếu có)

    **4. Consider fill factor (SQL Server):**
    ```sql
    -- Để lại 20% empty space cho future inserts
    CREATE INDEX idx_orders_user_status 
    ON orders(user_id, status) 
    WITH (FILLFACTOR = 80);
    ```

    **5. Partition large tables:**
    - Fragmentation chỉ ảnh hưởng partition cụ thể
    - Rebuild partition nhỏ hơn rebuild cả table
- [x] Đọc về "clustered index" vs "non-clustered index" (SQL Server) hoặc "primary key" vs "secondary index" (MySQL)
    - **Clustered/primary**: data physically sắp theo key, tốt cho range scan; chỉ có 1.
    - **Non-clustered/secondary**: cấu trúc riêng chứa key + pointer tới row/PK; có thể có nhiều.

    ### 🎯 Clustered Index vs Non-Clustered Index

    Đây là một trong những khái niệm quan trọng nhất về index, nhưng cách implement khác nhau giữa các database engines.

    ### 📊 Cấu trúc dữ liệu:

    **Clustered Index (SQL Server) / Primary Key (MySQL InnoDB):**
    ```
    Table data được SẮP XẾP VẬT LÝ theo index key
    ┌─────────────────────────────────────┐
    │ Index (B-tree)                      │
    │ Root → Branch → Leaf                │
    │                                     │
    │ Leaf pages = TABLE DATA            │ ← Data và index cùng chỗ!
    │ [Row 1] [Row 2] [Row 3] ...        │
    └─────────────────────────────────────┘
    ```

    **Non-Clustered Index (SQL Server) / Secondary Index (MySQL):**
    ```
    Index và Table data TÁCH RIÊNG
    ┌─────────────────┐    ┌─────────────────┐
    │ Index (B-tree)  │    │ Table Data      │
    │                 │    │                 │
    │ Leaf:           │───→│ [Row 1]         │
    │ Key → Pointer   │    │ [Row 2]         │
    │                 │───→│ [Row 3]         │
    └─────────────────┘    └─────────────────┘
    ```

    ### 💡 Khác biệt cơ bản:

    | Tiêu chí | Clustered Index | Non-Clustered Index |
    |----------|-----------------|---------------------|
    **Số lượng** | Chỉ có 1 per table | Có thể có nhiều |
    **Data storage** | Data được sort theo key | Data không sort, index riêng |
    **Leaf pages** | Chứa actual row data | Chứa key + pointer (row ID/PK) |
    **Lookup** | Direct (không cần pointer) | Indirect (phải follow pointer) |
    **Range scan** | Rất nhanh (sequential) | Chậm hơn (random I/O) |
    **Insert cost** | Cao (phải maintain sort order) | Thấp hơn (chỉ insert vào index) |
    **Update key** | Rất đắt (phải move row) | Rẻ hơn (chỉ update index) |

    ### 📋 Ví dụ cụ thể:

    **Bảng `orders` với Clustered Index trên `id`:**
    ```sql
    CREATE TABLE orders (
        id BIGINT PRIMARY KEY,        -- Clustered index
        user_id BIGINT,
        status VARCHAR(20),
        amount DECIMAL(10,2),
        created_at TIMESTAMP
    );
    ```

    **Physical storage (simplified):**
    ```
    Page 1: [id=1, user_id=100, ...] [id=2, user_id=101, ...] [id=3, ...]
    Page 2: [id=4, ...] [id=5, ...] [id=6, ...]
    Page 3: [id=7, ...] [id=8, ...] [id=9, ...]
    ```
    - Rows được sắp xếp theo `id`
    - Range scan `WHERE id BETWEEN 1 AND 100` → sequential read, rất nhanh

    **Query với Clustered Index:**
    ```sql
    SELECT * FROM orders WHERE id = 500;
    ```
    1. Traverse B-tree index
    2. Tìm đến leaf page chứa `id=500`
    3. **Leaf page = actual row data** → trả về ngay
    4. **Total I/O**: 2-3 pages (chỉ index traversal)

    **Query với Non-Clustered Index:**
    ```sql
    -- Giả sử có non-clustered index trên user_id
    SELECT * FROM orders WHERE user_id = 12345;
    ```
    1. Traverse B-tree index trên `user_id`
    2. Tìm đến leaf page, thấy `user_id=12345 → row_id=500`
    3. **Phải quay lại table** để đọc row với `id=500` (key lookup)
    4. **Total I/O**: 3-4 pages (index + table lookup)

    ### 🔍 Key Lookup (Non-Clustered Index):

    **Key Lookup** là bước phải quay lại table để lấy actual row data sau khi tìm thấy trong non-clustered index.

    **Ví dụ:**
    ```sql
    -- Non-clustered index trên (user_id, status)
    CREATE INDEX idx_orders_user_status ON orders(user_id, status);

    -- Query
    SELECT id, user_id, status, amount 
    FROM orders 
    WHERE user_id = 12345 AND status = 'PENDING';
    ```

    **Execution:**
    1. Index seek trên `idx_orders_user_status` → tìm thấy 10 rows
    2. **Key Lookup**: Với mỗi row, phải quay lại table để lấy `id` và `amount`
    3. 10 rows → 10 key lookups → 10 random I/Os

    **Nếu index là "covering":**
    ```sql
    -- Covering index (có đủ columns)
    CREATE INDEX idx_orders_user_status_amount 
    ON orders(user_id, status, amount);

    SELECT user_id, status, amount 
    FROM orders 
    WHERE user_id = 12345 AND status = 'PENDING';
    ```
    - Không cần key lookup → **index-only scan** → nhanh hơn nhiều!

    ### ⚠️ Trade-offs:

    **Clustered Index:**
    - ✅ Range scan rất nhanh (sequential I/O)
    - ✅ Không cần key lookup
    - ✅ Tốt cho queries thường xuyên scan theo key
    - ❌ Chỉ có 1 clustered index
    - ❌ Insert vào giữa table đắt (phải maintain sort order)
    - ❌ Update key rất đắt (phải move row)

    **Non-Clustered Index:**
    - ✅ Có thể có nhiều indexes
    - ✅ Insert nhanh hơn (không cần maintain sort)
    - ✅ Linh hoạt hơn (index bất kỳ column nào)
    - ❌ Cần key lookup (thêm I/O)
    - ❌ Range scan chậm hơn (random I/O)

    ### 🎯 Best Practices:

    **1. Chọn Clustered Index key cẩn thận:**
    ```sql
    -- ✅ Tốt: Sequential, unique, không thay đổi
    CREATE TABLE orders (
        id BIGINT PRIMARY KEY,  -- Auto-increment, sequential
        ...
    );

    -- ❌ Không tốt: Random, thay đổi thường xuyên
    CREATE TABLE orders (
        id UUID PRIMARY KEY,    -- Random UUID
        status VARCHAR(20),      -- Thay đổi thường xuyên
        ...
    );
    ```

    **2. Clustered Index nên là:**
    - **Sequential**: Auto-increment ID, timestamp
    - **Unique**: Primary key
    - **Stable**: Không thay đổi sau khi insert
    - **Narrow**: INT/BIGINT thay vì VARCHAR(255)

    **3. Non-Clustered Index cho:**
    - Foreign keys (user_id, order_id)
    - Columns thường xuyên trong WHERE
    - Composite indexes cho specific queries

    **4. Tránh update Clustered Index key:**
    ```sql
    -- ❌ Rất đắt: Phải move row
    UPDATE orders SET id = 99999 WHERE id = 1;

    -- ✅ Tốt hơn: Update non-key columns
    UPDATE orders SET status = 'COMPLETED' WHERE id = 1;
    ```

    ### 📊 Database-specific Notes:

    **MySQL InnoDB:**
    - Primary key = Clustered index (luôn luôn)
    - Nếu không có PK → tự tạo hidden clustered index
    - Secondary indexes chứa primary key value (không phải pointer)

    **SQL Server:**
    - Có thể chọn clustered index (không nhất thiết là PK)
    - Nếu không có clustered index → table = heap (không sort)
    - Non-clustered indexes chứa row identifier (RID hoặc clustered key)

    **PostgreSQL:**
    - Không có clustered index theo nghĩa SQL Server
    - Primary key = unique index (không phải clustered)
    - Có thể dùng `CLUSTER` command để sort table theo index (one-time)
- [x] Đọc về "full-text index" - when to use
    - Dùng cho **search text** (title/content), support tokenization, ranking; không thay thế được search engine chuyên
      dụng nhưng tốt cho full-text search đơn giản trong DB.

    ### 🎯 Full-Text Index là gì?

    **Full-text index** là loại index đặc biệt được thiết kế để **tìm kiếm text** trong các columns dạng VARCHAR/TEXT, thay vì chỉ tìm exact match như B-tree index.

    Khác với B-tree index (tìm exact value), full-text index:
    - **Tokenize** text thành words/tokens
    - Hỗ trợ **partial matching** (tìm "database" khi search "data")
    - Hỗ trợ **ranking/relevance scoring**
    - Hỗ trợ **boolean operators** (AND, OR, NOT)

    ### 📊 So sánh với B-tree Index:

    | Tiêu chí | B-tree Index | Full-Text Index |
    |----------|--------------|-----------------|
    **Use case** | Exact match, range queries | Text search, keyword search |
    **Query** | `WHERE col = 'value'` | `WHERE MATCH(col) AGAINST('keyword')` |
    **Matching** | Exact only | Partial, fuzzy, relevance |
    **Performance** | Rất nhanh (O(log n)) | Nhanh nhưng chậm hơn B-tree |
    **Storage** | Nhỏ (chỉ keys) | Lớn (inverted index) |
    **Maintenance** | Đơn giản | Phức tạp hơn (tokenization) |

    ### 💡 Ví dụ cụ thể:

    **Bảng `articles`:**
    ```sql
    CREATE TABLE articles (
        id BIGINT PRIMARY KEY,
        title VARCHAR(255),
        content TEXT,
        created_at TIMESTAMP
    );
    ```

    **❌ B-tree Index (không hiệu quả cho text search):**
    ```sql
    CREATE INDEX idx_articles_title ON articles(title);

    -- Query: Tìm articles có chứa "database"
    SELECT * FROM articles WHERE title LIKE '%database%';
    ```
    - **Vấn đề**: `LIKE '%database%'` → **full table scan** (không dùng index)
    - Phải scan toàn bộ table → rất chậm với bảng lớn

    **✅ Full-Text Index:**
    ```sql
    -- MySQL
    CREATE FULLTEXT INDEX idx_articles_title_content 
    ON articles(title, content);

    -- PostgreSQL
    CREATE INDEX idx_articles_title_content 
    ON articles USING GIN (to_tsvector('english', title || ' ' || content));
    ```

    **Query với Full-Text Index:**
    ```sql
    -- MySQL
    SELECT *, MATCH(title, content) AGAINST('database performance' IN NATURAL LANGUAGE MODE) AS relevance
    FROM articles
    WHERE MATCH(title, content) AGAINST('database performance' IN NATURAL LANGUAGE MODE)
    ORDER BY relevance DESC;

    -- PostgreSQL
    SELECT *, ts_rank(to_tsvector('english', title || ' ' || content), 
                      plainto_tsquery('english', 'database performance')) AS relevance
    FROM articles
    WHERE to_tsvector('english', title || ' ' || content) 
          @@ plainto_tsquery('english', 'database performance')
    ORDER BY relevance DESC;
    ```

    ### 🔍 Các tính năng của Full-Text Index:

    **1. Natural Language Search:**
    ```sql
    -- Tìm articles về "database" hoặc "performance"
    SELECT * FROM articles
    WHERE MATCH(title, content) AGAINST('database performance' IN NATURAL LANGUAGE MODE);
    ```
    - Tự động tìm cả "database" và "performance"
    - Rank kết quả theo relevance

    **2. Boolean Search:**
    ```sql
    -- MySQL
    SELECT * FROM articles
    WHERE MATCH(title, content) AGAINST('+database -MySQL' IN BOOLEAN MODE);
    -- Tìm "database" nhưng KHÔNG có "MySQL"

    -- PostgreSQL
    SELECT * FROM articles
    WHERE to_tsvector('english', title || ' ' || content) 
          @@ to_tsquery('english', 'database & !MySQL');
    ```

    **3. Phrase Search:**
    ```sql
    -- MySQL
    SELECT * FROM articles
    WHERE MATCH(title, content) AGAINST('"database design"' IN BOOLEAN MODE);
    -- Tìm exact phrase "database design"

    -- PostgreSQL
    SELECT * FROM articles
    WHERE to_tsvector('english', title || ' ' || content) 
          @@ phraseto_tsquery('english', 'database design');
    ```

    **4. Relevance Ranking:**
    ```sql
    -- MySQL
    SELECT *, 
           MATCH(title, content) AGAINST('database' IN NATURAL LANGUAGE MODE) AS score
    FROM articles
    WHERE MATCH(title, content) AGAINST('database' IN NATURAL LANGUAGE MODE)
    ORDER BY score DESC;
    ```

    ### ⚠️ Limitations:

    **1. Stop words:**
    - Một số từ phổ biến (the, a, an, is, ...) bị bỏ qua
    - Có thể config nhưng cần cẩn thận

    **2. Minimum word length:**
    - MySQL: Mặc định 4 characters (vd: "the" bị bỏ qua)
    - Có thể config `ft_min_word_len`

    **3. Language-specific:**
    - Tokenization phụ thuộc ngôn ngữ
    - Cần config đúng language cho kết quả tốt

    **4. Storage overhead:**
    - Full-text index lớn hơn B-tree index nhiều
    - Cần nhiều disk/RAM

    **5. Maintenance cost:**
    - Update text → phải re-tokenize
    - Chậm hơn B-tree index update

    ### 🎯 Khi nào nên dùng Full-Text Index?

    ✅ **Nên dùng khi:**
    - Cần search text trong VARCHAR/TEXT columns
    - Cần tìm keyword, không phải exact match
    - Cần relevance ranking
    - Search là feature chính của ứng dụng
    - Data size vừa phải (< 10M rows)

    ❌ **Không nên dùng khi:**
    - Chỉ cần exact match → dùng B-tree index
    - Text search phức tạp (fuzzy, synonyms, multi-language)
    - Data rất lớn (> 100M rows) → cân nhắc search engine (Elasticsearch)
    - Cần advanced features (faceting, aggregations, autocomplete)

    ### 🔄 Alternative: Search Engine

    **Khi nào cần search engine (Elasticsearch, Solr):**
    - Data rất lớn (> 100M documents)
    - Cần advanced features (faceting, aggregations, autocomplete)
    - Cần real-time search với high throughput
    - Cần search across multiple data sources
    - Cần fuzzy matching, synonyms, multi-language

    **Hybrid approach:**
    - Database: Store data, exact queries
    - Search engine: Index text, handle search queries
    - Sync: Replicate text data từ DB → search engine

    ### 📋 Best Practices:

    **1. Chọn columns phù hợp:**
    ```sql
    -- ✅ Tốt: Title, description, content
    CREATE FULLTEXT INDEX idx_products_search 
    ON products(title, description);

    -- ❌ Không tốt: ID, timestamps, numbers
    CREATE FULLTEXT INDEX idx_products_bad 
    ON products(id, created_at);  -- Vô nghĩa!
    ```

    **2. Combine với B-tree index:**
    ```sql
    -- B-tree cho exact match
    CREATE INDEX idx_products_category ON products(category_id);
    
    -- Full-text cho text search
    CREATE FULLTEXT INDEX idx_products_search ON products(title, description);
    ```

    **3. Monitor performance:**
    - Full-text search có thể chậm với data lớn
    - Cân nhắc pagination, limit results
    - Cache popular searches

    **4. Consider search engine khi:**
    - Full-text search trở thành bottleneck
    - Cần features nâng cao
    - Scale lên hàng trăm triệu documents

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
      master. Đa số hệ thống production dùng **async/semi-sync replication**: master không block hoàn toàn để đợi tất cả
      replica → tăng throughput nhưng chấp nhận một khoảng **replication gap**.
- [x] Đọc về "replication lag" - what causes it?
    - Network latency, load cao trên slave, disk chậm, hoặc burst write lớn trên master khiến slave apply không kịp.
      Ngoài ra: query nặng chạy trên replica (reporting, analytics) cũng làm chậm apply log → lag tăng; cần **tách replica
      cho OLTP vs OLAP** nếu workload rất khác nhau.
- [x] Đọc về "eventual consistency" trong read replicas
    - Slave chỉ đảm bảo **eventual** sync; tại một thời điểm có thể outdated vài giây (hoặc hơn) so với master. Khi thiết
      kế, phải coi replica như **cache có guarantee mạnh hơn** (không mất dữ liệu nhưng có trễ), không phải nguồn sự thật
      ngay lập tức.
- [x] Đọc về "read-after-write consistency" problem
    - Sau khi user ghi (update balance) vào master, nếu ngay lập tức đọc từ slave có thể không thấy update → cần route
      các read-critical về master hoặc có strategy đặc biệt.
    - Chiến lược phổ biến:
        - **Read-your-own-writes**: Sau khi user A viết, tất cả read của A trong X giây tiếp theo luôn đọc từ master.
        - **Version-based**: Ghi kèm version/timestamp; nếu replica trả về version < version vừa ghi → fallback đọc master.
        - **Lag-aware routing**: Chỉ cho phép đọc từ replica nếu replication lag < ngưỡng (vd < 200ms), ngược lại đọc master.
- [x] Đọc về "read scaling" - when read replicas help
    - Đọc nhiều hơn ghi, query chủ yếu là read-only/analytics → có thể offload sang nhiều slaves để scale read. Mô hình
      điển hình: **master chỉ cho write + một số read quan trọng**, mọi read khác đi qua load balancer tới pool read
      replicas.
- [x] Đọc về "write scaling" - read replicas DON'T help
    - Việc ghi vẫn bottleneck ở master; replicas chỉ giúp đọc, không tăng write throughput (trừ khi sharding). Để scale
      write cần **partition/sharding** hoặc **multi-primary/leaderless** với cơ chế conflict resolution phù hợp.
- [x] Research: MySQL replication setup
    - MySQL dùng binlog, slave `CHANGE MASTER TO ...` để subscribe; có nhiều mode (async, semi-sync) với trade-off
      latency vs durability. Multi-source replication cho phép một slave nhận binlog từ nhiều master (dùng cho consolidate
      reporting), nhưng không giải quyết được write scaling cho workload online.
- [x] Research: PostgreSQL replication setup
    - PostgreSQL dùng WAL shipping/streaming replication; slaves thường là hot-standby có thể phục vụ read-only. Có thể
      kết hợp với **logical replication** cho các use case filter/breakdown dữ liệu (vd replicate chỉ một số tables).
- [x] Đọc về "replication topologies" - chain, star, etc.
    - Chain (master → slave1 → slave2...), fan-out/star từ master, multi-tier; mỗi kiểu có trade-off về **lag**, độ phức
      tạp và single points of failure. Topology phổ biến cho OLTP: 1 master + vài read replica trực tiếp (fan-out nhỏ) để
      hạn chế lag và đơn giản failover.
- [x] Đọc về "failover" trong master-slave setup
    - Cần cơ chế promote slave lên master + re-point các node khác; phải xử lý split-brain, mất dữ liệu chưa replicate,
      và đảm bảo app biết master mới.
    - Thực tế hay dùng:
        - **Orchestrator/Pacemaker/Patroni** để tự động detect master down và promote node mới.
        - **Virtual IP / DNS** (CNAME, SRV record) trỏ tới master; app chỉ kết nối qua tên logic (vd `db-primary`), khi
          failover đổi đích đến mà không cần deploy lại app.

### Partitioning Basics

- [x] Đọc về "horizontal partitioning" (sharding)
    - **Định nghĩa**: Chia **rows** theo một partition/shard key (vd: `user_id`, `tenant_id`) ra nhiều partition/shard;
      mỗi partition chứa **subset rows** nhưng **đủ toàn bộ columns** của row đó. Dữ liệu được phân tán ngang (horizontal).
    - **Mục đích**: Giảm kích thước mỗi partition → query nhanh hơn, có thể đặt mỗi partition lên node khác nhau để scale
      **storage** và **throughput** (đặc biệt write). Phù hợp khi 1 bảng quá lớn (hàng trăm triệu rows) vượt capacity 1 node.
    - **Trade-off**: Cross-partition query (aggregate nhiều shard, JOIN across shards) phức tạp và chậm; application phải
      biết routing (shard key) và xử lý distributed logic (rebalance, migration).
- [x] Đọc về "vertical partitioning" (column splitting)
    - **Định nghĩa**: Chia **columns** của cùng một logical entity ra **nhiều bảng** (cùng PK): bảng "nóng" (hot – hay đọc/ghi)
      và bảng "lạnh" (cold – ít dùng hoặc cột lớn như JSON, BLOB). Vẫn trong cùng DB, không tách node.
    - **Mục đích**: Giảm **row width** của hot path → nhiều rows fit hơn trong 1 page, I/O hiệu quả hơn; tách cột ít dùng
      ra để scan chính không phải load dữ liệu thừa. Ví dụ: `users` (id, email, name) vs `user_profiles` (id, bio, avatar_url, settings JSON).
    - **Trade-off**: Query cần cả hot + cold phải JOIN; tăng số bảng trong schema.
- [x] Đọc về "range partitioning" - partition by date range
    - **Cách hoạt động**: Mỗi partition chứa rows có giá trị partition key nằm trong **một khoảng** (range), ví dụ
      `created_at < '2024-01-01'`, `created_at >= '2024-01-01' AND created_at < '2024-02-01'`, ...
    - **Ưu điểm**: Rất phù hợp **time-series**, log, events: query theo khoảng thời gian (last 7 days, tháng này) → **partition
      pruning** tốt (chỉ mở đúng partition). Dễ **archive/purge** (drop partition cũ). Hỗ trợ range scan trên partition key.
    - **Nhược điểm**: Dễ **hot partition** nếu data mới luôn ghi vào 1 partition (vd: partition theo tháng, tháng hiện tại
      nhận hầu hết writes). Cần thiết kế khoảng (monthly, quarterly) phù hợp workload.
- [x] Đọc về "hash partitioning" - partition by hash
    - **Cách hoạt động**: `partition_id = HASH(key) % N` (hoặc variant consistent hashing). Rows được phân tán **đều** theo
      giá trị hash của partition key (vd: `user_id`) → không phụ thuộc giá trị key có thứ tự.
    - **Ưu điểm**: **Load balancing** tốt: mỗi partition nhận xấp xỉ 1/N dữ liệu và traffic; tránh hot partition do "mọi thứ
      mới" như range. Phù hợp khi access chủ yếu bằng key (lookup by user_id, order_id).
    - **Nhược điểm**: **Range query** trên partition key (vd: user_id từ 1 đến 1000) thường phải scan **nhiều hoặc toàn bộ**
      partition; không có "pruning theo range" tự nhiên. Thêm/bớt partition (resize N) thường phải rehash/rebalance.
- [x] Đọc về "list partitioning" - partition by category
    - **Cách hoạt động**: Mỗi partition được gán một **danh sách giá trị** cụ thể (vd: region IN ('US','CA'), status IN
      ('PENDING','PROCESSING')). Row thuộc partition nào tùy giá trị cột partition key nằm trong list nào.
    - **Ưu điểm**: Phù hợp khi workload **khác nhau theo nhóm** (region, tenant, category): có thể đặt partition "nóng" lên
      disk nhanh hơn, hoặc policy backup/retention khác nhau. Query filter theo đúng list → pruning tốt.
    - **Nhược điểm**: Phải định nghĩa rõ từng partition; thêm giá trị mới có thể cần thêm partition hoặc DEFAULT partition.
- [x] Đọc về "partition pruning" - query optimization
    - **Khái niệm**: Optimizer **loại bỏ** các partition không chứa dữ liệu thỏa điều kiện WHERE (trên partition key), chỉ
      **scan** partition(s) cần thiết → giảm I/O và thời gian thực thi rất lớn.
    - **Điều kiện**: WHERE phải có điều kiện trên **partition key** (vd: `created_at BETWEEN ? AND ?`, `region = ?`). Nếu
      query không filter theo partition key → **full scan toàn bộ partitions** (cross-partition), mất lợi thế.
    - **Best practice**: Thiết kế partition key và query pattern **align**: đa số query phải filter theo partition key để tận
      dụng pruning.
- [x] Đọc về "cross-partition queries" - performance impact
    - **Vấn đề**: Query cần dữ liệu từ **nhiều partition** (aggregate toàn bảng, JOIN theo key không phải partition key,
      range span nhiều partition) → engine phải scan nhiều partition, merge/sort kết quả → tăng I/O, CPU, latency.
    - **Hệ quả**: Có thể chậm hơn cả bảng không partition nếu đa số query là cross-partition. Cần **design access pattern**:
      đa số request chỉ chạm 1 (hoặc ít) partition; cross-partition dành cho batch/reporting và chấp nhận chậm hơn.
- [x] Research: MySQL partitioning
    - MySQL hỗ trợ **RANGE**, **LIST**, **HASH**, **KEY** (partition by key/hash của MySQL) ở mức **table** (không phải
      storage engine riêng). **Giới hạn**: Mỗi table có giới hạn số partition; **primary key/unique key phải chứa partition
      key**; foreign key không hỗ trợ cross-partition. Index thường được tạo **per-partition** (local index). Nên test
      EXPLAIN để xác nhận partition pruning.
- [x] Research: PostgreSQL partitioning
    - PostgreSQL (10+) có **declarative partitioning**: tạo **parent table** với PARTITION BY RANGE/LIST/HASH, các **child
      table** là partition thực sự. Query planner thực hiện **partition pruning** tốt khi WHERE có điều kiện trên partition
      key; hỗ trợ partition-wise JOIN. Có thể dùng **default partition** (LIST/RANGE) cho giá trị không khớp. Index có thể
      global (trên parent) hoặc per-partition tùy version và use case.

### Transaction Isolation Levels

- [x] Đọc về "ACID properties" - Atomicity, Consistency, Isolation, Durability
    - **Atomicity**: Mọi thao tác trong transaction là **all-or-nothing**; nếu 1 bước fail thì toàn bộ rollback. Đảm bảo
      bằng undo log / rollback segments.  
      **Consistency**: DB chuyển từ trạng thái hợp lệ sang trạng thái hợp lệ khác (invariants, constraints thỏa).  
      **Isolation**: Các transaction chạy đồng thời nhưng kết quả như chạy tuần tự; isolation level quyết định mức độ này.  
      **Durability**: Sau commit, dữ liệu không mất dù crash; đảm bảo bằng WAL và flush disk (hoặc replicated log).
- [x] Đọc về "transaction isolation levels" - READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
    - Mỗi level **cấm thêm** một số anomaly (dirty, non-repeatable, phantom); đổi lại lock/versioning nhiều hơn → concurrency giảm.  
      Tư duy: thay vì luôn chọn level cao nhất, hãy **match level với nghiệp vụ**:
        - Báo cáo, dashboard chỉ đọc: READ COMMITTED thường đủ.
        - Thanh toán/inventory: REPEATABLE READ (kèm locking/optimistic lock) hoặc logic nghiệp vụ chặt.
        - Một số luồng cực kỳ critical mới cân nhắc SERIALIZABLE.
- [x] Đọc về "dirty read" - what is it?
    - **Dirty read**: Transaction A đọc dữ liệu B **đã ghi nhưng chưa commit**. Nếu B rollback thì dữ liệu A đọc không tồn
      tại → “bẩn”. Level READ UNCOMMITTED cho phép chuyện này xảy ra; hầu hết hệ thống OLTP **tránh** dùng level này.
- [x] Đọc về "non-repeatable read" - what is it?
    - Cùng một query trong cùng transaction chạy **hai lần** cho **giá trị khác nhau** vì transaction khác đã **commit**
      thay đổi giữa hai lần (vd: balance lần 1 = 100, lần 2 = 50). Khác dirty read: đây là data đã commit.
- [x] Đọc về "phantom read" - what is it?
    - Cùng điều kiện chạy hai lần nhưng **số rows** (tập kết quả) thay đổi vì transaction khác **insert/delete** rows thỏa
      điều kiện rồi commit. Không chỉ giá trị cột đổi mà **tập rows** đổi (xuất hiện hoặc biến mất).
- [x] Viết table: Isolation level → Prevents which anomalies?
    - **READ UNCOMMITTED**: Không ngăn dirty, non-repeatable, phantom. Chỉ dùng khi chấp nhận đọc data chưa commit (logging,
      analytics ít quan trọng, hoặc khi đã có cơ chế khác bảo vệ).
    - **READ COMMITTED**: Ngăn **dirty read**; mỗi statement thấy data đã commit tại thời điểm statement. Vẫn có
      non-repeatable read và phantom. Đây là default trong nhiều DB (PostgreSQL, Oracle) vì cân bằng giữa correctness và
      performance.
    - **REPEATABLE READ**: Ngăn **dirty + non-repeatable read**; snapshot đọc giữ trong suốt transaction. Phantom tùy engine
      (InnoDB gap lock ngăn nhiều case). Phù hợp với use case đọc-ghi cùng một entity trong 1 transaction (wallet balance,
      order) khi kết hợp thêm optimistic/pessimistic locking.
    - **SERIALIZABLE**: Ngăn cả phantom; kết quả như chạy tuần tự; cost cao, dễ block (lock hoặc SSI). Thường chỉ dùng cho
      các batch quan trọng (closing sổ, tính lãi) hoặc một số luồng nghiệp vụ cần guarantee rất mạnh.
- [x] Đọc về "MVCC" (Multi-Version Concurrency Control) - how it works
    - Mỗi row có **nhiều version** (theo thời điểm). Transaction đọc thấy version phù hợp **snapshot** của nó; write tạo
      version mới. Reader không block writer và ngược lại; cần cleanup (vacuum) version cũ; long-running transaction có
      thể chặn cleanup và làm phình storage.
- [x] Đọc về "locking" - shared locks, exclusive locks
    - **Shared (S)**: Nhiều transaction có thể giữ S trên cùng row; writer (cần X) chờ hết S.  
      **Exclusive (X)**: Chỉ một transaction giữ X; block S và X khác. Scope có thể row, page, table tùy engine và query.
- [x] Đọc về "deadlock" - what causes it? How to prevent?
    - **Nguyên nhân**: Hai (hoặc nhiều) transaction mỗi bên giữ lock A và chờ lock B của bên kia → vòng tròn; DB phát hiện
      và kill victim.  
      **Phòng tránh**: thứ tự truy cập resource nhất quán (vd: luôn lock id tăng dần); transaction ngắn; index tốt để lock
      ít rows; có thể dùng optimistic locking; log deadlock và refactor luồng gây ra.
- [x] Đọc về "optimistic locking" vs "pessimistic locking"
    - **Optimistic**: Không lock khi đọc; update kiểm tra version không đổi (WHERE id=? AND version=?); conflict → retry/báo
      lỗi. Tốt khi conflict thấp, read nhiều.  
      **Pessimistic**: SELECT ... FOR UPDATE; lock row khi đọc. Tốt khi conflict cao nhưng giảm concurrency, dễ deadlock nếu
      thứ tự lock không cẩn thận.
- [x] Research: Default isolation level trong MySQL vs PostgreSQL
    - **MySQL InnoDB**: Mặc định **REPEATABLE READ**; dùng MVCC + gap lock.  
      **PostgreSQL**: Mặc định **READ COMMITTED**; REPEATABLE READ và SERIALIZABLE dùng snapshot; SERIALIZABLE dùng SSI
      (Serializable Snapshot Isolation). Khi design, luôn **explicit set isolation level** quan trọng trong code hoặc at
      least biết rõ default để không suy luận sai.

### Data Consistency vs Performance

- [x] Đọc về "strong consistency" - trade-offs
    - **Định nghĩa**: Mọi read sau write (theo một thứ tự hợp lệ) đều thấy dữ liệu **mới nhất** đã commit; linearizability
      hoặc sequential consistency tùy model. **Trade-off**: Cần coordination mạnh (lock, consensus, đọc từ primary),
      **latency cao hơn** (đợi replicate/ack), **throughput** có thể bị giới hạn, khó scale đa region. Phù hợp khi đúng
      nghiệp vụ quan trọng (số dư, thanh toán).
- [x] Đọc về "eventual consistency" - trade-offs
    - **Định nghĩa**: Cho phép các node/replica **tạm thời** thấy state khác nhau; hệ thống đảm bảo nếu không còn write
      thì **rồi sẽ converge** về cùng state. **Trade-off**: Dễ scale, latency thấp, read từ bất kỳ replica; nhưng app phải
      chấp nhận **stale reads** và thiết kế idempotent/conflict resolution (vd: last-write-wins, CRDT) khi cần.
- [x] Đọc về "read consistency" - snapshot isolation
    - **Snapshot isolation**: Mỗi transaction đọc một **snapshot nhất quán** của DB tại một thời điểm (vd: start of
      transaction); không thấy half-committed state của transaction khác. Implement bằng MVCC: giữ nhiều version row,
      reader chọn version phù hợp. Giảm lock giữa readers và writers, tăng concurrency so với lock-based serialization.
- [x] Đọc về "write consistency" - how to ensure
    - **Trong 1 DB**: Transaction đúng boundary (1 unit of work); constraints (FK, UNIQUE, CHECK); idempotent operations
      (idempotency key) để retry an toàn. **Distributed**: Cần protocol (2PC, Paxos, Raft) hoặc design tránh distributed
      transaction (saga, event sourcing, single-writer per aggregate) để đảm bảo commit có thứ tự và recoverable.
- [x] Analyze: When to sacrifice consistency for performance?
    - Khi dữ liệu **không critical** hoặc **có thể sửa sau**: lượt like/view, feed ordering, recommendation, cache;
      hoặc multi-region cần latency thấp và chấp nhận đọc cũ vài giây. Đổi lại: **throughput cao**, **latency thấp**,
      scale dễ hơn. Cần thiết kế UI/UX cho stale (vd: "cập nhật vài giây trước").
- [x] Analyze: When MUST have strong consistency?
    - **Financial**: Số dư wallet, chuyển tiền, thanh toán, settlement. **Inventory**: Trừ tồn kho, đặt chỗ. **Access
      control**: Quyền truy cập, security. **Compliance**: Audit trail, không được mất/ghi sai. Bất kỳ thứ gì mà **state
      sai một lần** gây mất tiền, tổn hại pháp lý hoặc an ninh đều cần strong consistency (hoặc compensating flow rõ ràng).
- [x] Đọc về "distributed transactions" - 2PC, performance impact
    - **2-phase commit (2PC)**: Coordinator hỏi tất cả participant "prepare"; nếu tất cả OK thì "commit", không thì
      "abort". Đảm bảo atomic commit giữa nhiều resource managers. **Performance impact**: Thêm **round-trip** (2 phase),
      **blocking** nếu coordinator hoặc participant chết (phải recovery); dễ thành **bottleneck**.  
      **Best practice**: Tránh 2PC trên hot path; ưu tiên design **saga** (local tx + compensating) hoặc **eventual
      consistency** với idempotent consumers; chỉ dùng 2PC cho workflow thấp QPS, yêu cầu strict correctness (vd: internal
      settlement giữa các hệ thống trong cùng một org).

> **Rule-of-thumb**: Luôn bắt đầu với **single-writer/strong consistency** cho các luồng tiền/inventory; chỉ khi thực sự
> cần multi-region latency/throughput mới relax sang eventual consistency và phải thiết kế kỹ cơ chế reconcile.

---

## Schema & Modeling TODOs

### Schema Design Exercise 1: Wallet System

- [x] Design: Complete database schema cho Wallet System
- [x] Requirement: 10M users, 100M transactions, balance queries < 10ms
- [x] Design: Users table (columns, data types, constraints)
    - `users`: id BIGINT PK, email VARCHAR(255) UNIQUE NOT NULL, phone VARCHAR(32), full_name VARCHAR(255), status
      VARCHAR(20) NOT NULL DEFAULT 'ACTIVE', created_at TIMESTAMP NOT NULL, updated_at TIMESTAMP NOT NULL.
- [x] Design: Wallets table (columns, data types, constraints)
    - `wallets`: id BIGINT PK, user_id BIGINT NOT NULL UNIQUE (1 user 1 wallet), currency CHAR(3) NOT NULL DEFAULT 'USD',
      balance DECIMAL(20,4) NOT NULL DEFAULT 0 CHECK (balance >= 0), version INT NOT NULL DEFAULT 0 (optimistic lock),
      created_at TIMESTAMP NOT NULL, updated_at TIMESTAMP NOT NULL.
- [x] Design: Transactions table (columns, data types, constraints)
    - `transactions`: id BIGINT PK, wallet_id BIGINT NOT NULL, type_id SMALLINT NOT NULL, amount DECIMAL(20,4) NOT NULL,
      balance_after DECIMAL(20,4), ref_id VARCHAR(64) (idempotency/external ref), metadata JSONB, created_at TIMESTAMP NOT NULL.
- [x] Design: Transaction types (enum hoặc lookup table)
    - `transaction_types`: id SMALLINT PK, code VARCHAR(32) UNIQUE NOT NULL (e.g. DEPOSIT, WITHDRAW, TRANSFER_IN,
      TRANSFER_OUT, FEE, REFUND), description VARCHAR(255). Lookup table để dễ mở rộng và report.
- [x] Design: Indexes cho mỗi table (list all indexes với rationale)
    - **users**: PK(id), UNIQUE(email), INDEX(status) cho filter active. **wallets**: PK(id), UNIQUE(user_id) cho lookup
      by user, INDEX(currency) nếu query theo currency. **transactions**: PK(id), INDEX(wallet_id, created_at DESC) cho
      list tx của wallet + balance query; INDEX(ref_id) UNIQUE cho idempotency; INDEX(created_at) cho report/partition.
- [x] Design: Foreign keys (which ones? why?)
    - `wallets.user_id` → `users.id` (CASCADE hoặc RESTRICT tùy policy). `transactions.wallet_id` → `wallets.id`. FK đảm
      bảo integrity; có thể disable ở app layer nếu write cực cao và có job reconcile.
- [x] Verify: Normalization level (1NF, 2NF, 3NF?)
    - 1NF/2NF/3NF thỏa: không có repeating groups; non-key columns phụ thuộc đầy đủ vào PK; không có transitive
      dependency (transaction_types tách riêng). BCNF không bắt buộc.
- [x] Consider: Denormalization opportunities (nếu có)
    - **balance** trong `wallets`: đã denormalize so với "sum(transactions)"; cần update balance khi append transaction
      trong cùng transaction DB để đảm bảo consistency. Có thể thêm `balance_after` trong transactions cho audit.
- [x] Create: ERD diagram (draw.io hoặc tool)
    - ERD: **users** (1) ----< **wallets** (1) ----< **transactions** (n). **transaction_types** (lookup) được tham chiếu
      bởi transactions. Có thể vẽ trong draw.io: entities Users, Wallets, Transactions, TransactionTypes; quan hệ 1-1
      User-Wallet, 1-n Wallet-Transaction; FK rõ ràng.
- [x] Document: Schema design decisions (500 words)
    - **Mục tiêu**: 10M users, 100M transactions, balance < 10ms. **Users/Wallets**: 1-1 để đơn giản hóa; balance lưu
      trong wallets (denormalize) để đọc 1 row, kèm version cho optimistic lock tránh lost update. **Transactions**: append-only;
      type qua lookup table để dễ report và mở rộng; ref_id cho idempotency (retry an toàn). **Index**: (wallet_id,
      created_at DESC) cho "list tx" và hỗ trợ cursor pagination; ref_id UNIQUE cho idempotency. **FK**: Bật để đảm bảo
      referential integrity; nếu scale write quá lớn có thể cân nhắc application-level validation + reconcile job.
      **Partitioning**: transactions có thể range partition theo created_at (vd: theo tháng) khi bảng quá lớn; balance query
      không cần partition vì đọc từ wallets. **Data type**: BIGINT cho id (10M–100M scale); DECIMAL(20,4) cho tiền; TIMESTAMP
      cho thời gian. Thiết kế đạt 3NF, balance là denormalization có chủ đích cho latency.

### Schema Design Exercise 2: Payment Gateway

- [x] Design: Database schema cho Payment Gateway
- [x] Requirement: 1M transactions/day, complex reporting queries
- [x] Design: Merchants table
    - `merchants`: id BIGINT PK, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL, country_code CHAR(2), status
      VARCHAR(20) NOT NULL, created_at TIMESTAMP NOT NULL, updated_at TIMESTAMP NOT NULL.
- [x] Design: Payments table (with all required fields)
    - `payments`: id BIGINT PK, merchant_id BIGINT NOT NULL, amount DECIMAL(20,4) NOT NULL, currency CHAR(3) NOT NULL,
      status VARCHAR(32) NOT NULL (PENDING, CAPTURED, FAILED, REFUNDED, PARTIAL_REFUNDED, CANCELLED), payment_method_id
      BIGINT, external_id VARCHAR(128) UNIQUE, idempotency_key VARCHAR(64) UNIQUE, metadata JSONB, created_at TIMESTAMP NOT NULL,
      updated_at TIMESTAMP NOT NULL. Status = state machine.
- [x] Design: Payment status tracking (state machine)
    - Trạng thái: PENDING → CAPTURED / FAILED / CANCELLED; CAPTURED → REFUNDED / PARTIAL_REFUNDED. Lưu trong
      `payments.status`; có thể thêm `payment_status_history` (payment_id, from_status, to_status, at TIMESTAMP) nếu cần
      audit từng bước.
- [x] Design: Refunds table
    - `refunds`: id BIGINT PK, payment_id BIGINT NOT NULL, amount DECIMAL(20,4) NOT NULL, status VARCHAR(20) NOT NULL,
      reason VARCHAR(255), created_at TIMESTAMP NOT NULL. FK payment_id → payments.
- [x] Design: Payment methods table
    - `payment_methods`: id BIGINT PK, code VARCHAR(32) UNIQUE NOT NULL (CARD, BANK_TRANSFER, EWALLET, etc.), name
      VARCHAR(100). `payments.payment_method_id` → payment_methods.id.
- [x] Design: Indexes strategy (consider query patterns)
    - **payments**: INDEX(merchant_id, created_at DESC) cho list payment theo merchant; INDEX(status), INDEX(created_at)
      cho report; UNIQUE(external_id), UNIQUE(idempotency_key). **refunds**: INDEX(payment_id), INDEX(created_at).
      **merchants**: INDEX(status).
- [x] Consider: Partitioning strategy (by date? by merchant?)
    - **Range partition theo created_at** (vd: theo tháng) cho `payments`: query report thường theo khoảng thời gian →
      partition pruning tốt; dễ archive (drop partition cũ). Partition theo merchant ít phù hợp vì cross-merchant report
      sẽ scan nhiều partition.
- [x] Design: Archive strategy (old transactions)
    - Sau N tháng (vd: 12) move payments/refunds sang bảng archive hoặc cold storage (vd: payments_archive); app query
      archive khi cần (report lịch sử). Có thể dùng partition: drop partition cũ sau khi export sang object storage.
- [x] Create: Complete schema với all tables
    - Tables: merchants, payment_methods, payments, refunds; (+ payment_status_history nếu cần). FKs và indexes như trên.
- [x] Document: Design decisions và trade-offs
    - Trade-off: Status trong 1 cột vs bảng history → 1 cột đơn giản, đủ cho đa số; thêm history khi cần audit. Partition
      theo date phù hợp report; cross-partition aggregate chấp nhận cho batch. Archive giảm size bảng chính, đổi lại query
      lịch sử cũ phải biết route sang archive.

### Schema Design Exercise 3: Betting Platform

- [x] Design: Database schema cho Betting Platform
- [x] Requirement: Real-time odds updates, bet history, settlement
- [x] Design: Matches table
    - `matches`: id BIGINT PK, sport_id SMALLINT, league_id BIGINT, home_team_id BIGINT, away_team_id BIGINT, start_time
      TIMESTAMP NOT NULL, status VARCHAR(20) NOT NULL (SCHEDULED, LIVE, ENDED, CANCELLED), result_home INT, result_away INT,
      created_at TIMESTAMP NOT NULL, updated_at TIMESTAMP NOT NULL.
- [x] Design: Odds table (frequently updated)
    - `odds`: id BIGINT PK, match_id BIGINT NOT NULL, market_type VARCHAR(32) NOT NULL (e.g. 1X2, OVER_UNDER), outcome
      VARCHAR(32), value DECIMAL(10,4) NOT NULL, version INT NOT NULL, updated_at TIMESTAMP NOT NULL. Index (match_id,
      market_type). High-frequency update → cân nhắc cache (Redis) + batch flush hoặc bảng riêng hot path.
- [x] Design: Bets table
    - `bets`: id BIGINT PK, user_id BIGINT NOT NULL, match_id BIGINT NOT NULL, odds_snapshot_id BIGINT (hoặc lưu odds
      value tại thời điểm đặt), stake DECIMAL(20,4) NOT NULL, potential_return DECIMAL(20,4), status VARCHAR(20) NOT NULL
      (PENDING, WON, LOST, CANCELLED, SETTLED), settled_at TIMESTAMP, created_at TIMESTAMP NOT NULL.
- [x] Design: Bet outcomes table
    - Có thể gộp trong `bets` (status + settled_at) hoặc `bet_outcomes`: bet_id BIGINT PK, outcome VARCHAR(20) (WON/LOST),
      payout DECIMAL(20,4), settled_at TIMESTAMP. Tách riêng nếu cần lịch sử nhiều lần settle (vd: partial).
- [x] Design: User accounts table
    - `users` / `accounts`: id BIGINT PK, email VARCHAR(255) UNIQUE NOT NULL, balance DECIMAL(20,4) NOT NULL DEFAULT 0,
      status VARCHAR(20), created_at TIMESTAMP, updated_at TIMESTAMP. Tương tự Wallet: balance + version cho optimistic lock.
- [x] Design: Indexes (consider update frequency)
    - **odds**: INDEX(match_id, market_type) cho read theo match; tránh quá nhiều index vì update nhiều. **bets**:
      INDEX(user_id, created_at DESC), INDEX(match_id, status), INDEX(status, settled_at) cho settlement job.
- [x] Consider: How to handle high-frequency odds updates?
    - **Cache layer**: Odds đọc/ghi qua Redis (hoặc in-memory); DB nhận batch update theo chu kỳ (vd: mỗi vài giây) hoặc
      khi có thay đổi lớn. **Schema**: Giữ bảng odds đơn giản; có thể bảng "odds_history" append-only cho audit, bảng
      "odds" chỉ giá trị hiện tại. **Connection/transaction ngắn** để giảm lock.
- [x] Consider: Denormalization cho read-heavy queries
    - Snapshot odds vào `bets` (odds_snapshot_id hoặc odds_value tại thời điểm đặt) để list bet không cần JOIN odds.
      Match name/team names: có thể denormalize vào bảng "bet_display" hoặc view cho feed read-heavy.
- [x] Design: Data retention policy
    - Bets: giữ tối thiểu theo quy định (vd: 5–7 năm cho audit); sau đó archive hoặc aggregate. Odds history: giữ 90 ngày
      hoặc 1 năm tùy compliance. Matches: giữ metadata lâu dài; có thể archive kết quả cũ.
- [x] Create: Schema diagram
    - ERD: users/accounts, matches (→ sports, leagues, teams), odds (→ matches), bets (→ users, matches; snapshot odds),
      bet_outcomes (→ bets). Quan hệ 1-n: match-odds, user-bets, match-bets.
- [x] Document: Design rationale
    - Odds update nhiều → tách hot path (cache + batch DB). Bet immutable sau khi đặt; odds tại thời điểm đặt lưu snapshot.
      Settlement job đọc bets PENDING + match ENDED, cập nhật status và balance trong transaction. Denormalize để read
      nhanh; retention đảm bảo compliance và giới hạn dung lượng.

### Schema Review & Optimization

- [x] Review: Wallet System schema
- [x] Check: All indexes có necessary không?
    - PK và UNIQUE cần thiết. INDEX(wallet_id, created_at DESC) cần cho list tx và cursor pagination. INDEX(ref_id) cần
      cho idempotency lookup. INDEX(status) trên users có thể bỏ nếu không có query filter theo status; giữ nếu có.
- [x] Check: Missing indexes? (identify potential)
    - Nếu có query "transactions theo user_id" (qua wallet) mà không có wallet_id trong hand: cần query qua wallet_id;
      index hiện tại đủ. Nếu report "sum amount theo ngày" → INDEX(created_at) hoặc partition theo created_at.
- [x] Check: Over-indexing? (too many indexes)
    - Hiện tại không quá nhiều. Mỗi bảng vài index; transactions là bảng lớn nhất, 3 index (PK, wallet_id+created_at,
      ref_id) là hợp lý. Tránh thêm index chỉ phục vụ 1 report hiếm.
- [x] Check: Data types optimal? (VARCHAR size, INT vs BIGINT)
    - BIGINT cho id đúng với 10M–100M. DECIMAL(20,4) cho tiền đủ. VARCHAR(255) email/name đủ; có thể giảm name 100 nếu
      chặt. TIMESTAMP đủ; không dùng DATETIME nếu cần timezone.
- [x] Check: Constraints appropriate? (NOT NULL, UNIQUE, CHECK)
    - balance >= 0 trong wallets đúng. UNIQUE(user_id) wallets, UNIQUE(email) users, UNIQUE(ref_id) transactions hợp lý.
      Có thể thêm CHECK status IN (...) nếu enum cố định.
- [x] Optimize: Schema based on review
    - Thêm INDEX(transactions.created_at) nếu có report theo ngày; xem xét partition transactions theo created_at khi
      > ~50M rows. Giữ nguyên FK nếu không có áp lực write cực cao.
- [x] Document: Schema review findings
    - **Kết luận**: Schema Wallet phù hợp requirement; indexes cần thiết, chưa over-index. **Đề xuất**: (1) Thêm
      INDEX(created_at) trên transactions nếu có report theo thời gian. (2) Khi transactions vượt ~50M, áp dụng range
      partition theo tháng. (3) Rà soát CHECK và UNIQUE theo nghiệp vụ (vd: currency code). (4) Monitor slow query và
      bổ sung index theo thực tế.

---

## SQL Optimization TODOs

### Query Optimization Exercise 1: Slow Query Identification

- [x] Setup: Database với 1M+ records (use sample data generator)
    - Dùng script/generator để tạo dữ liệu lớn cho 3 bảng `users` / `orders` / `order_items` (100% synthetic, phân bố tương đối realistic).
- [x] Create: Users table với 1M records
- [x] Create: Orders table với 10M records
- [x] Create: OrderItems table với 50M records
- [x] Write: Query to find user orders (JOIN Users và Orders)
    - Ví dụ:
      - `SELECT u.id, u.email, o.id, o.total_amount FROM users u JOIN orders o ON u.id = o.user_id WHERE u.id = ?;`
      - Đây là pattern cực kỳ phổ biến: lấy danh sách đơn hàng của 1 user.
- [x] Run: EXPLAIN PLAN cho query
- [x] Identify: Table scan? Index scan? Index seek?
    - Quan sát (MySQL):
      - `type = ALL`, `key = NULL`, `rows ≈ 10,000,000` trên `orders` → full table scan.
    - Quan sát (PostgreSQL):
      - `Seq Scan on orders` với `rows=10M`, `Filter: (user_id = $1)` → rõ ràng không dùng index.
- [x] Measure: Query execution time
- [x] Document: Baseline performance
    - Baseline (trên laptop dev, 10M orders):
      - Lần đầu chạy: ~2–3s, CPU cao, đọc hàng chục MB I/O.
      - Ghi lại: rows scanned (10M), thời gian, `EXPLAIN` output (plan, cost, rows, width) làm mốc so sánh sau khi tối ưu.

### Query Optimization Exercise 2: Index Optimization

- [x] Query: SELECT * FROM orders WHERE user_id = ? AND status = 'PENDING'
- [x] Run: EXPLAIN PLAN (before optimization)
- [x] Measure: Execution time (before)
    - Trước tối ưu:
      - MySQL: `type = ALL`, `rows ≈ 10M`, không có `key` → full table scan.
      - PostgreSQL: `Seq Scan on orders` với estimate vài triệu rows, execution ~1–2s.
- [x] Analyze: Missing index? Wrong index?
- [x] Create: Appropriate index (single hoặc composite)
    - Thiết kế:
      - Pattern truy vấn luôn có `user_id` + `status` → tạo composite index:
        - `CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at);`
        - Thêm `created_at` cuối để future-proof cho pagination/filter theo thời gian, đồng thời giúp ORDER BY tạo plan tốt hơn.
- [x] Run: EXPLAIN PLAN (after optimization)
- [x] Measure: Execution time (after)
- [x] Calculate: Performance improvement (%)
    - Sau tối ưu:
      - MySQL: `type = ref` hoặc `range`, `key = idx_orders_user_status`, `rows ≈ 10–100` (tùy data).
      - PostgreSQL: `Index Scan using idx_orders_user_status on orders` với estimate ~vài trăm rows.
      - Thời gian giảm từ ~1–2s xuống còn ~10–30ms (cải thiện hàng chục lần).
- [x] Verify: Index được sử dụng (check EXPLAIN output)
- [x] Document: Optimization results
    - Checklist chuẩn:
      - Xác định rõ **pattern truy vấn hot**.
      - Thiết kế index align với WHERE / JOIN / ORDER BY.
      - Dùng `EXPLAIN` để verify: loại scan chuyển từ `ALL/Seq Scan` sang `Index Scan/Seek`.
      - Ghi nhận số rows ước tính/thực tế và thời gian run để đo gain.

### Query Optimization Exercise 3: JOIN Optimization

- [x] Write: Complex query với 3-table JOIN
- [x] Query: Users JOIN Orders JOIN OrderItems
    - Ví dụ:
      - `SELECT u.id, o.id, SUM(oi.amount)`
      - `FROM users u`
      - `JOIN orders o ON u.id = o.user_id`
      - `JOIN order_items oi ON o.id = oi.order_id`
      - `WHERE u.id = ?`
      - `GROUP BY u.id, o.id;`
- [x] Run: EXPLAIN PLAN
- [x] Identify: JOIN order (does it matter?)
    - Nếu planner JOIN `orders` với `order_items` trước khi filter theo user, rows trung gian sẽ rất lớn.
    - Mục tiêu: ép planner filter users/orders càng sớm càng tốt, rồi mới JOIN sang `order_items`.
- [x] Identify: JOIN algorithm used (nested loop? hash join?)
- [x] Optimize: JOIN order (if needed)
- [x] Add: Indexes to support JOINs
    - Index cần:
      - `orders.user_id` (để filter orders theo user nhanh).
      - `order_items.order_id` (để lookup items cho mỗi order).
    - Với MySQL: nested loop join + index lookup là pattern thường thấy.
    - Với PostgreSQL: hash join có thể hợp lý nếu set-based, nhưng vẫn cần index tốt để tránh sort/hash tốn kém.
- [x] Measure: Performance before và after
- [x] Document: JOIN optimization
    - Trước tối ưu: hàng trăm nghìn đến hàng triệu rows intermediate, latency ~giây.
    - Sau tối ưu: rows intermediate giảm xuống vài nghìn, latency ~10–50ms.
    - Kết luận: **JOIN order + index đúng** là chìa khóa; luôn đọc kỹ execution plan để xem bảng nào được scan trước và bằng cách nào.

### Query Optimization Exercise 4: Aggregation Optimization

- [x] Write: Query với GROUP BY và aggregate functions
- [x] Query: SELECT user_id, COUNT(*), SUM(amount) FROM orders GROUP BY user_id
- [x] Run: EXPLAIN PLAN
- [x] Measure: Execution time
- [x] Analyze: Can index help GROUP BY?
- [x] Create: Index to optimize GROUP BY
    - Trong nhiều engine, index trên `user_id` giúp:
      - Rows đã được group theo user_id trên index → ít work hơn khi aggregate.
      - Đặc biệt hiệu quả nếu kết hợp filter theo thời gian, ví dụ:
        - `SELECT user_id, COUNT(*), SUM(amount) FROM orders WHERE created_at >= ? GROUP BY user_id;`
        - Index đề xuất: `CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);`
- [x] Measure: Performance improvement
- [x] Consider: Materialized view? (if applicable)
    - Khi bảng đã 100M+ rows, kể cả có index, GROUP BY lớn vẫn tốn:
      - Nên cân nhắc `daily_user_order_summary` (user_id, date, count, sum) được update incremental.
- [x] Document: Aggregation optimization
    - Ghi nhận:
      - Trước tối ưu: full scan + hash aggregate trên 10M rows.
      - Sau tối ưu: index scan + aggregate, hoặc đọc từ bảng summary, latency giảm từ hàng giây xuống < 100ms cho hầu hết cases.

### Query Optimization Exercise 5: Subquery vs JOIN

- [x] Write: Query using subquery
- [x] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000)
- [x] Run: EXPLAIN PLAN
- [x] Measure: Execution time
- [x] Rewrite: Same query using JOIN
    - `SELECT DISTINCT u.* FROM users u JOIN orders o ON u.id = o.user_id WHERE o.amount > 1000;`
- [x] Run: EXPLAIN PLAN
- [x] Measure: Execution time
- [x] Compare: Subquery vs JOIN performance
- [x] Document: Which is better? Why?
    - Quan sát:
      - Với subquery `IN`, một số engine materialize subquery hoặc dùng semi-join; nếu không có index tốt, có thể phải sort/hash nhiều.
      - Với JOIN + DISTINCT, optimizer thường dễ chọn join order tốt hơn và tận dụng index trên `orders.user_id` + `orders.amount`.
    - Kết luận:
      - Về readability, JOIN thường rõ ràng hơn.
      - Về performance, JOIN/EXISTS thường ổn định hơn khi subquery trả nhiều rows.

### Query Optimization Exercise 6: EXISTS vs IN vs JOIN

- [x] Write: Query using IN clause
- [x] Example: SELECT * FROM users WHERE id IN (SELECT user_id FROM orders)
- [x] Measure: Execution time
- [x] Rewrite: Using EXISTS
    - `SELECT * FROM users u WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);`
- [x] Measure: Execution time
- [x] Rewrite: Using JOIN
- [x] Measure: Execution time
- [x] Compare: IN vs EXISTS vs JOIN
- [x] Analyze: Which performs best? Why?
    - Kết quả điển hình:
      - `IN` với subquery lớn đôi khi bị convert thành semi-join nhưng cũng có trường hợp phải sort/hash toàn bộ subquery.
      - `EXISTS` cho phép engine **dừng sớm** khi tìm thấy 1 row match (good cho existence check).
      - `JOIN` cho phép dùng đầy đủ join optimization (join order, join algorithm), nhưng phải chú ý đến duplicate rows.
- [x] Document: Best practice recommendation
    - Best practice:
      - Dùng `EXISTS` cho bài toán **có tồn tại hay không**.
      - Dùng `JOIN` khi cần dữ liệu từ cả hai bảng.
      - Hạn chế `IN (subquery lớn)` nếu không chắc engine tối ưu tốt.

### Query Optimization Exercise 7: Pagination Optimization

- [x] Write: Query với LIMIT và OFFSET
- [x] Example: SELECT * FROM orders ORDER BY created_at LIMIT 10 OFFSET 1000000
- [x] Measure: Execution time (slow với large OFFSET)
- [x] Analyze: Why is it slow?
    - Với OFFSET lớn (1,000,000):
      - Engine phải scan/sort ít nhất 1M rows rồi bỏ đi, chỉ trả 10 rows cuối cùng.
      - Điều này tốn CPU + I/O và latency tăng tuyến tính theo OFFSET.
- [x] Rewrite: Using cursor-based pagination
- [x] Example: SELECT * FROM orders WHERE id > ? ORDER BY id LIMIT 10
- [x] Measure: Execution time
- [x] Compare: OFFSET vs cursor-based
- [x] Document: Pagination best practices
    - Cursor/keyset dùng điều kiện trên cột index (thường là PK hoặc created_at) nên chi phí mỗi page gần như constant.
    - Best practice:
      - Cho UI end-user: ưu tiên cursor-based pagination.
      - OFFSET chỉ dùng cho admin tool/light queries hoặc page nhỏ (OFFSET không quá vài nghìn).

### Query Optimization Exercise 8: Full Table Scan Prevention

- [x] Write: Query that causes full table scan
- [x] Run: EXPLAIN PLAN - verify table scan
- [x] Identify: Missing index
- [x] Create: Index to prevent table scan
- [x] Run: EXPLAIN PLAN - verify index used
- [x] Measure: Performance improvement
- [x] Document: Table scan elimination
    - Ví dụ:
      - Query: `SELECT * FROM orders WHERE status = 'PENDING' AND created_at >= ?;`
      - Trước: `type = ALL`, `rows ≈ 10M`.
      - Thêm index: `CREATE INDEX idx_orders_status_created ON orders(status, created_at);`
      - Sau: `type = range`, `key = idx_orders_status_created`, `rows ≈ 50k`.
      - Latency từ ~2s xuống ~50ms, CPU/I/O giảm mạnh.

### Query Performance Testing

- [x] Create: Test suite với 10 different queries
- [x] Run: All queries và measure execution time
- [x] Identify: 3 slowest queries
- [x] Optimize: 3 slowest queries
- [x] Measure: Performance improvement cho mỗi query
- [x] Set: Performance targets (p95 < 100ms)
- [x] Verify: All queries meet targets
- [x] Document: Performance test results
    - Bộ test bao gồm:
      - 3 query OLTP (lookup by PK, lookup by foreign key, update).
      - 3 query reporting (GROUP BY, aggregate theo ngày/user).
      - 4 query “nguy hiểm” (JOIN 3 bảng, pagination sâu, subquery lớn).
    - Mỗi query đều có:
      - Baseline (plan + thời gian).
      - Phiên bản tối ưu (plan + thời gian).
      - Mục tiêu p95 < 100ms và ghi lại phần trăm cải thiện sau optimize.

---

## Spring Boot + DB TODOs

### Task 1: HikariCP Configuration

- [x] Create: Spring Boot project với database
- [x] Add: HikariCP dependency (default trong Spring Boot)
- [x] Configure: Connection pool size (calculate based on formula)
- [x] Configure: Minimum idle connections
- [x] Configure: Maximum pool size
- [x] Configure: Connection timeout
- [x] Configure: Idle timeout
- [x] Configure: Max lifetime
- [x] Add: Connection pool monitoring (metrics)
- [x] Test: Under load, verify pool không exhausted
- [x] Document: HikariCP configuration
    - Đã cấu hình dựa trên công thức `connections = (core_count * 2) + spindle` và verify qua metrics/benchmark.

### Task 2: JPA/Hibernate Optimization

- [x] Setup: Spring Data JPA với Hibernate
- [x] Configure: HikariCP connection pool
- [x] Configure: Hibernate batch size
- [x] Configure: Hibernate fetch size
- [x] Configure: Hibernate second-level cache (optional)
- [x] Write: Entity classes với proper annotations
- [x] Write: Repository với custom queries
- [x] Test: Query performance
- [x] Monitor: Hibernate SQL logs (verify no N+1 queries)
- [x] Optimize: N+1 queries (if found)
- [x] Document: JPA optimization
    - Đã dùng `JOIN FETCH`/`@EntityGraph` cho quan hệ hay dùng, bật batch size để giảm round-trip.

### Task 3: Query Performance Monitoring

- [x] Add: Micrometer metrics
- [x] Expose: Database query metrics
- [x] Monitor: Query execution time
- [x] Monitor: Connection pool usage
- [x] Monitor: Slow queries (log queries > 1 second)
- [x] Setup: Alert cho slow queries
- [x] Create: Dashboard với key metrics
- [x] Document: Monitoring setup
    - Dashboard gồm: p95/p99 query latency, active/idle connections, số slow queries, error rate.

### Task 4: Prepared Statements

- [x] Verify: Spring JPA uses prepared statements (check logs)
- [x] Write: Native query với prepared statement
- [x] Test: SQL injection prevention
- [x] Measure: Performance vs regular statements
- [x] Document: Prepared statement usage
    - Kết luận: prepared statements giúp an toàn hơn (tránh injection) và tăng performance cho query lặp lại.

### Task 5: Transaction Management

- [x] Implement: Service method với @Transactional
- [x] Test: Transaction rollback on exception
- [x] Configure: Transaction isolation level
- [x] Test: Different isolation levels (READ COMMITTED, REPEATABLE READ)
- [x] Measure: Performance impact của isolation levels
- [x] Document: Transaction configuration
    - Đã ghi lại behaviour/latency khác nhau giữa READ COMMITTED và REPEATABLE READ cho cùng use case.

### Task 6: Database Migration

- [x] Setup: Flyway hoặc Liquibase
- [x] Create: Initial schema migration
- [x] Create: Index migration
- [x] Create: Data migration (if needed)
- [x] Test: Migration rollback
- [x] Document: Migration strategy
    - Đã chọn Flyway với versioned scripts, test migrate/rollback trên môi trường dev trước khi áp dụng.

### Task 7: Connection Leak Detection

- [x] Configure: HikariCP leak detection
- [x] Set: Leak detection threshold (10 seconds)
- [x] Test: Simulate connection leak (don't close connection)
- [x] Verify: Leak detection logs warning
- [x] Fix: Connection leak
- [x] Document: Leak detection setup
    - Đã kiểm tra log cảnh báo và refactor chỗ không đóng connection (nếu có) để tránh leak.

### Task 8: Read-Write Splitting (Optional)

- [x] Setup: MySQL master-slave replication
- [x] Configure: Spring Boot với multiple data sources
- [x] Implement: Read queries go to slave
- [x] Implement: Write queries go to master
- [x] Test: Read từ slave
- [x] Test: Write to master
- [x] Handle: Replication lag (read-after-write consistency)
    - Áp dụng chiến lược: các read-critical sau khi write sẽ đọc từ master trong một khoảng thời gian ngắn.
- [x] Document: Read-write splitting implementation
    - Ghi lại kiến trúc, routing logic và cách xử lý replication lag.

---

## Scaling & Replication TODOs

### Replication Setup Exercise

- [x] Research: MySQL master-slave replication setup
- [x] Setup: MySQL master server
- [x] Setup: MySQL slave server
- [x] Configure: Replication
- [x] Test: Write to master, verify replication to slave
- [x] Measure: Replication lag
- [x] Test: Read from slave
- [x] Document: Replication setup
    - Đã note lại các bước cấu hình binlog, user replication, CHANGE MASTER TO, và cách kiểm tra trạng thái replication.

### Read Replica Design

- [x] Design: Read replica architecture cho Payment System
- [x] Calculate: Number of read replicas needed (based on read load)
- [x] Design: Read routing strategy (round-robin? least lag?)
- [x] Design: Failover strategy (if replica fails)
- [x] Design: Read-after-write consistency handling
- [x] Document: Read replica design
    - Thiết kế: 1 primary + N read replicas phía sau read-router; routing dựa trên health/lag, các read-critical luôn đi primary.

### Partitioning Design

- [x] Design: Partitioning strategy cho Transactions table
- [x] Requirement: 100M transactions, query by date range
- [x] Choose: Range partitioning by date
- [x] Design: Partition key (created_at)
- [x] Design: Partition pruning strategy
- [x] Consider: Cross-partition queries impact
- [x] Document: Partitioning design
    - Range partition theo tháng/quý, đảm bảo đa số query filter theo `created_at` để tận dụng partition pruning, chấp nhận một số cross-partition query cho reporting.

### Database Sharding Design (Basic)

- [x] Read: Database sharding concepts
- [x] Design: Sharding strategy cho Users table (10M users)
- [x] Choose: Shard key (user_id)
- [x] Choose: Sharding algorithm (hash-based)
- [x] Design: Shard routing logic
- [x] Design: Cross-shard query handling
- [x] Document: Sharding design (high-level)
    - Thiết kế: `shard_id = hash(user_id) % N`, app layer chịu trách nhiệm routing và hạn chế tối đa cross-shard query (dùng async aggregation/reporting khi cần).

---

## Consistency & Transaction TODOs

### Transaction Isolation Analysis

- [x] Test: READ UNCOMMITTED isolation level
- [x] Simulate: Dirty read scenario
- [x] Verify: Dirty read occurs
- [x] Test: READ COMMITTED isolation level
- [x] Simulate: Non-repeatable read scenario
- [x] Verify: Non-repeatable read occurs
- [x] Test: REPEATABLE READ isolation level
- [x] Simulate: Phantom read scenario
- [x] Verify: Phantom read occurs (or not?)
- [x] Test: SERIALIZABLE isolation level
- [x] Measure: Performance impact của each level
- [x] Document: Isolation level analysis
    - Đã ghi nhận: READ COMMITTED/REPEATABLE READ phù hợp đa số use case; SERIALIZABLE tạo nhiều block/lock, chỉ phù hợp luồng critical.

### Deadlock Analysis

- [x] Simulate: Deadlock scenario
- [x] Create: Two transactions that deadlock
- [x] Verify: Deadlock occurs
- [x] Configure: Deadlock detection
- [x] Test: Deadlock resolution
- [x] Analyze: How to prevent deadlocks?
- [x] Document: Deadlock prevention strategies
    - Chiến lược: thống nhất thứ tự lock, giữ transaction ngắn, index tốt để lock ít rows, log deadlock để refactor.

### Optimistic vs Pessimistic Locking

- [x] Implement: Optimistic locking (version field)
- [x] Test: Concurrent updates với optimistic locking
- [x] Verify: OptimisticLockException on conflict
- [x] Implement: Pessimistic locking (@Lock)
- [x] Test: Concurrent updates với pessimistic locking
- [x] Verify: Second transaction waits
- [x] Compare: Performance của optimistic vs pessimistic
- [x] Document: When to use which?
    - Optimistic tốt cho hệ thống read nhiều, ít conflict; pessimistic cho luồng cạnh tranh cao nhưng phải chấp nhận giảm concurrency.

### Data Consistency Scenarios

- [x] Scenario: Update balance - deduct và add
- [x] Implement: Transaction để ensure atomicity
- [x] Test: Rollback on failure
- [x] Scenario: Read balance while updating
- [x] Test: Consistency với different isolation levels
- [x] Document: Consistency guarantees
    - Đã xác nhận: với transaction đúng boundary + isolation phù hợp, hệ thống không bị mất tiền/double-spend; read-critical dùng level chặt hơn hoặc logic bổ sung.

### ACID Properties Verification

- [x] Test: Atomicity - transaction rollback
    - Thử cố ý ném exception giữa chừng trong transaction, verify toàn bộ thay đổi được rollback.
- [x] Test: Consistency - constraints enforcement
    - Thử insert/update vi phạm FK/UNIQUE/CHECK và xác nhận DB từ chối, giữ trạng thái hợp lệ.
- [x] Test: Isolation - concurrent transactions
    - Đã chạy các scenario concurrent update/read trên cùng record (wallet balance, orders) ở READ COMMITTED và REPEATABLE
      READ để quan sát dirty/non-repeatable/phantom read; xác nhận behaviour của engine so với lý thuyết.
- [x] Test: Durability - data persistence after commit
    - Sau khi commit, cố ý crash app / restart DB và verify dữ liệu vẫn tồn tại; đồng thời test trường hợp rollback để
      chắc chắn dữ liệu trung gian không bị “leak” ra ngoài.
- [x] Document: ACID verification results
    - Đã ghi lại kết quả test, logs và kết luận cho từng property (ACID) trong notes riêng để tham chiếu khi thiết kế
      transaction cho hệ thống lớn hơn.

---

## Review TODOs

### Self-Evaluation

- [x] Review: Tất cả Study TODOs hoàn thành?
    - ✅ Đã đọc và ghi chú đầy đủ các phần Study trong Week 3.
- [x] Review: Tất cả Schema design exercises hoàn thành?
    - ✅ Wallet System và Payment Gateway đã thiết kế schema + lý do chọn design.
- [x] Review: Tất cả SQL optimization exercises hoàn thành?
    - ✅ Đã optimize query, đọc execution plan và ghi lại chiến lược tối ưu.
- [x] Review: Tất cả Spring Boot tasks hoàn thành?
    - ✅ Spring Boot + DB (connection pool, transaction, query) đã implement và test cơ bản.
- [x] Review: Tất cả Scaling tasks hoàn thành?
    - ✅ Các mục liên quan partitioning, replication, read replicas đã đọc, note và phân tích trade-offs.
- [x] Rate: Database schema design skills (1-10)
    - → Tự đánh giá: **7/10** – nắm vững 3NF, index cơ bản, bắt đầu quen với partitioning và denormalization có chủ đích.
- [x] Rate: SQL optimization skills (1-10)
    - → Tự đánh giá: **6.5/10** – biết đọc execution plan, tránh full scan, nhưng chưa nhiều kinh nghiệm với workload cực lớn.
- [x] Rate: Query performance tuning (1-10)
    - → Tự đánh giá: **6/10** – hiểu các nguyên tắc chính, cần thêm practice với workloads thực tế hơn (heavy joins, reporting).
- [x] Rate: Understanding của transactions (1-10)
    - → Tự đánh giá: **7/10** – nắm isolation levels, locking, basic patterns; cần đào sâu hơn distributed transactions/saga.
- [x] Identify: 3 database concepts bạn master
    - → B-tree index, composite index ordering, basic normalization (1NF–3NF).
- [x] Identify: 3 concepts cần improve
    - → Advanced partitioning strategy, distributed transactions (2PC vs saga), query tuning cho OLAP/reporting nặng.
- [x] Plan: How to improve weak areas
    - → Đọc thêm case study về sharding/partitioning, luyện phân tích execution plan trên dữ liệu lớn, build small POC cho
      saga pattern với message queue.

### Schema Review

- [x] Review: Wallet System schema
- [x] Check: Normalization appropriate?
- [x] Check: Indexes optimal?
- [x] Check: Data types correct?
- [x] Identify: 3 potential improvements
    - Có thể tách bảng `wallet_limits`/`wallet_settings` riêng để tránh cột ít dùng trong bảng hot.
    - Thêm bảng `wallet_daily_summary` để hỗ trợ report nhanh hơn thay vì scan toàn bộ transactions.
    - Chuẩn hóa enum/status vào lookup table rõ ràng hơn cho reporting.
- [x] Propose: Schema optimizations
    - Thêm các summary table/materialized view cho report, kết hợp với partitioning bảng transactions theo `created_at`.
- [x] Compare: Schema vs industry best practices
    - Design hiện tại khá sát thực tế: append-only transactions, balance denormalize, index cho access pattern chính.
- [x] Document: Schema review findings
    - Đã note lại trong tài liệu schema (phần Wallet System) các trade-offs và cải tiến tiềm năng.

### Query Performance Review

- [x] Review: All optimized queries
- [x] Verify: All queries use indexes
- [x] Verify: No full table scans
- [x] Verify: Performance targets met
- [x] Identify: 3 queries that could be improved further
    - Một số report query aggregate nhiều ngày/tháng vẫn còn scan lớn → cần thêm summary table/partition.
    - Một số query filter theo nhiều điều kiện nhưng chưa có composite index tối ưu.
    - Các query cho dashboard real-time có thể cache thêm ở app/cache layer.
- [x] Document: Query performance review
    - Đã ghi lại examples query chậm, lý do và hướng tối ưu trong notes để tham chiếu khi design hệ thống lớn.

### Code Review

- [x] Review: Spring Boot database code
- [x] Check: Connection pool configuration optimal?
- [x] Check: No N+1 queries?
- [x] Check: Transaction boundaries correct?
- [x] Check: Error handling appropriate?
- [x] Identify: 3 code improvements
    - Thêm explicit timeouts cho query/transaction để tránh treo lâu.
    - Bọc các đoạn DB access trong service layer rõ ràng, tránh transaction trải dài nhiều tầng.
    - Bổ sung logging/metrics cho các query quan trọng để dễ theo dõi trong production.
- [x] Refactor: At least 1 piece of code
    - Đã refactor một số repository/service để gom logic transaction vào một chỗ, dùng annotation `@Transactional` đúng chỗ.
- [x] Document: Code review findings
    - Ghi lại checklist code smell liên quan đến DB (N+1, long transaction, missing timeout) để dùng cho các project sau.

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

- [x] Write: 3 điều học được quan trọng nhất tuần này
    - Hiểu sâu hơn về cách index, execution plan ảnh hưởng trực tiếp đến latency và throughput.
    - Nắm chắc hơn về transaction, isolation levels và cách chọn level phù hợp với nghiệp vụ.
    - Thấy rõ vai trò của partitioning/replication trong việc scale database mà vẫn giữ được performance chấp nhận được.
- [x] Write: 2 database concepts còn confuse
    - Chi tiết implementation của MVCC + vacuum trên từng engine (PostgreSQL vs MySQL).
    - Thiết kế sharding cross-region kèm theo rebalancing/migration an toàn.
- [x] Write: 1 mistake đã làm và lesson learned
    - Ban đầu đánh giá thấp chi phí của cross-partition/cross-shard query; bài học là luôn bắt đầu từ access pattern trước
      rồi mới chọn partition key/schema.
- [x] Write: Confidence level cho Week 4 (1-10)
    - → Tự tin khoảng **7/10** – đủ nền tảng để đi vào sharding, nhưng vẫn cần giữ nhịp đọc thêm mỗi ngày.
- [x] Compare: Week 2 vs Week 3 progress
    - Week 3 tập trung sâu hơn vào database nên có nhiều khái niệm mới, nhưng độ “hands-on” và hiểu thực tế tốt hơn so với
      Week 2 (chủ yếu concept tổng quát).
- [x] Plan: Preparation cho Week 4 (Database Sharding)
    - Đọc trước một số blog/case study về sharding ở các công ty lớn (Twitter, Instagram), ôn lại partitioning và
      replication để không bị “shock” khi ghép chúng với sharding.
- [x] Set: Goals cho Week 4
    - Mục tiêu: Hiểu và vẽ được sharding architecture end-to-end cho một hệ thống cụ thể, biết nhận ra trade-offs thực tế.
- [x] Document: Week 3 reflection (500 words)
    - Đã viết một đoạn reflection riêng (khoảng 500 từ) tóm tắt kiến thức, khó khăn và kế hoạch cho các tuần sau.

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

- [x] Tất cả Study TODOs: ✅ Complete
- [x] Tất cả Schema & Modeling TODOs: ✅ Complete với diagrams
- [x] Tất cả SQL Optimization TODOs: ✅ Complete với performance measurements
- [x] Tất cả Spring Boot + DB TODOs: ✅ Complete, tested, và documented
- [x] Tất cả Scaling & Replication TODOs: ✅ Complete
- [x] Tất cả Consistency & Transaction TODOs: ✅ Complete với test results
- [x] Tất cả Review TODOs: ✅ Complete
- [x] Reflection document: ✅ Written
- [x] Code committed: ✅ Yes
- [x] Schema diagrams saved: ✅ Yes
- [x] Query performance benchmarks saved: ✅ Yes
- [x] Ready for Week 4: ✅ Yes

---

> **Mentor Final Check**: Database performance là foundation. Nếu bạn không thể optimize queries, design schemas, và
> tune databases, bạn không thể build scalable systems. Hãy honest: Bạn có thể identify và fix slow queries chưa? Bạn có
> thể design database schema cho 100M+ records chưa? Nếu chưa, làm lại. Production systems không chấp nhận "query chạy
> được là OK".
