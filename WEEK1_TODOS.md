# Week 1 – Fundamentals: Scalability & Availability

> **Mentor Note**: Đây là TODO list nghiêm khắc. Mỗi item phải được hoàn thành 100%. Không có "gần như xong" hay "hiểu
> đại khái". Bạn phải CODE, MEASURE, và DOCUMENT.

---

## Study TODOs

### Scalability Concepts

- [ ] Đọc "Designing Data-Intensive Applications" Chapter 1 - Scalability (pages 1-30)
- [ ] Viết notes: Định nghĩa chính xác của "scalability" trong 2 câu
  As the system grows (in data volume, traffic volume, or complexity), there should
  be reasonable ways of dealing with that growth.
- [ ] Liệt kê 5 điểm khác biệt giữa vertical và horizontal scaling
  + Cách mở rộng: một máy mạnh hơn - thêm nhiều máy
  + Giới hạn: có - không giới hạn
  + Khả năng chịu lỗi: kém - tốt
  + Độ phức tạp khi vận hành: dễ - khó
  + Quy mô: Nhỏ, Trung - Trung, Lớn

- [ ] Tìm 3 real-world examples của vertical scaling (và tại sao họ chọn)
  + Blog cá nhân: Rẻ, đơn giản
  + ERP nội bộ: ít user
  + Single DB
- [ ] Tìm 3 real-world examples của horizontal scaling (và tại sao họ chọn)
  + Nexflix API: Scale lớn
  + Shopee: Flash Sale Traffic
  + Redis, Cache Cluster: Cache phải cực nhanh, RAM có giới hạn, Data lớn
- [ ] Đọc về "Amdahl's Law" và viết công thức + giải thích ý nghĩa
  **Công thức:**
  ```
  Speedup = 1 / ((1-P) + P/N)
  ```
  Hoặc viết dưới dạng:
  ```
  Speedup = 1 / (S + (1-S)/N)
  ```
  Trong đó:
  - **S** = phần không thể song song (serial portion), tỷ lệ từ 0 đến 1
  - **P** = phần có thể song song (parallel portion), P = 1 - S
  - **N** = số lõi/tài nguyên xử lý (số processors/cores)
  - **Speedup** = mức tăng tốc so với chạy tuần tự

      **Ý nghĩa:**
      - Amdahl's Law cho thấy giới hạn tối đa của việc tăng tốc khi chỉ một phần công việc có thể song song hóa
      - Phần serial (không thể song song) luôn là bottleneck và giới hạn hiệu suất của toàn hệ thống
      - Ví dụ: Nếu 5% thời gian không thể song song (S=0.05), dù có vô hạn CPU thì tốc độ tối đa chỉ tăng 20 lần (1/0.05 = 20x)
      - Thực tế: Speedup_real = 1 / (S + (1-S)/N + Overhead)
        - Overhead bao gồm: Context switch, Lock contention, Garbage Collection, Network latency
      - **Bài học**: Muốn scale hiệu quả, phải giảm phần serial (bottleneck) trước khi thêm tài nguyên

| Loại Serial (System Design)   | Ví dụ                                | Vì sao làm chậm  | Định luật áp dụng | Giải thích                                                    |
|-------------------------------|--------------------------------------|------------------|-------------------|---------------------------------------------------------------|
| 🗄️ **Database bottleneck**   | Transaction dài, `SELECT FOR UPDATE` | DB xử lý tuần tự | **Amdahl**        | DB bottleneck > 10% query → phải fix (replica, shard)         |
| 🔌 **Chung tài nguyên**       | 1 file, 1 connection, 1 queue        | Phải đợi nhau    | **Amdahl**        | Shared resource > 10% contention → phải fix (shard, separate) |
| 📐 **Logic bắt buộc tuần tự** | step1 → step2 → step3                | Không tách được  | **Amdahl**        | Serial logic > 10% time → phải optimize                       |
| 🚀 **Init / Startup**         | Load config, warmup                  | Chạy 1 luồng     | **Gustafson**     | Init < 1% total time → không đáng kể, có nhiều task khác      |

Scalability = Throughput (RPS, job/s) tăng theo server.

Amdahl nói:

❌ Thêm server ≠ tăng vô hạn
✅ Phải giảm cổ chai trước

5️⃣ Serial trong hệ thống thường là

| Bottleneck            | Nghĩa là gì                   | Serial ở đâu        | Dấu hiệu thường gặp                      | Scale app có giúp không? | Định luật áp dụng          | Cách xử lý chính                 |
|-----------------------|-------------------------------|---------------------|------------------------------------------|--------------------------|----------------------------|----------------------------------|
| **Database**          | DB xử lý quá nhiều read/write | 1 DB node / primary | Query chậm, CPU DB 100%, connection full | ❌ Không                  | **Amdahl** (> 10% query)   | Read replica, sharding, cache    |
| **Hot Key Redis**     | Nhiều request đập vào 1 key   | Redis single thread | Redis latency cao, key bị spam           | ❌ Không                  | **Amdahl** (> 10% traffic) | Shard key, cache local, cluster  |
| **Single Leader**     | Chỉ 1 node được write/xử lý   | Leader node         | Write TPS thấp, leader overload          | ❌ Không                  | **Amdahl** (> 10% write)   | Shard, multi-leader, partition   |
| **Lock Global**       | 1 lock khóa toàn hệ thống     | Critical section    | Thread waiting nhiều, TPS thấp           | ❌ Không                  | **Amdahl** (> 10% request) | Fine-grained lock, optimistic    |
| **Queue 1 Partition** | Queue chỉ có 1 partition      | 1 consumer          | Lag cao, consumer idle                   | ❌ Không                  | **Amdahl** (> 10% message) | Tăng partition, parallel consume |
| **File dùng chung**   | Nhiều node dùng chung file    | File lock / IO      | IO wait cao, ghi file chậm               | ❌ Không                  | **Amdahl** (> 10% I/O)     | File riêng, object storage       |

**Lưu ý**: Tất cả các bottleneck trên đều áp dụng **Amdahl's Law** khi chiếm > 10% workload. Nếu < 1% workload và có
nhiều partition/key/task khác → có thể áp dụng **Gustafson's Law** (không cần fix).

- [ ] Đọc về "Gustafson's Law" và so sánh với Amdahl's Law
  **Gustafson's Law - Công thức:**
  ```
  Speedup = (1 - P) + P × N
  ```
  Hoặc viết dưới dạng:
  ```
  Speedup = S + (1 - S) × N
  ```
  Trong đó:
  - **S** = phần không thể song song (serial portion), tỷ lệ từ 0 đến 1
  - **P** = phần có thể song song (parallel portion), P = 1 - S
  - **N** = số lõi/tài nguyên xử lý (số processors/cores)
  - **Speedup** = mức tăng tốc so với chạy tuần tự

      **Ý nghĩa:**
      - Gustafson's Law có cách nhìn khác với Amdahl's Law: khi có nhiều lõi hơn, ta có thể **tăng kích thước vấn đề** (scaled speedup)
      - Giả định: Kích thước vấn đề có thể tăng theo số lõi (ví dụ: xử lý nhiều data hơn khi có nhiều CPU hơn)
      - Speedup tăng **gần tuyến tính** với số lõi N, không bị giới hạn nghiêm ngặt như Amdahl's Law
      - Ví dụ: Nếu 10% công việc tuần tự (S=0.1), với 100 cores: Speedup = 0.1 + 0.9×100 = 90.1x
      - **Ứng dụng**: Big data processing, distributed systems, khi workload có thể scale theo tài nguyên
      
      **So sánh Amdahl's Law vs Gustafson's Law:**

| Tiêu chí                 | **Amdahl's Law**                       | **Gustafson's Law**                                |
|--------------------------|----------------------------------------|----------------------------------------------------|
| **Giả định**             | Kích thước vấn đề **cố định**          | Kích thước vấn đề **có thể tăng**                  |
| **Công thức**            | `Speedup = 1 / (S + (1-S)/N)`          | `Speedup = S + (1-S) × N`                          |
| **Speedup tối đa**       | Bị giới hạn bởi phần serial (hữu hạn)  | Gần tuyến tính với N (có thể rất lớn)              |
| **Ví dụ (S=0.1, N=100)** | **9.17x**                              | **90.1x**                                          |
| **Quan điểm**            | Phần serial là bottleneck nghiêm trọng | Phần serial không cản trở nhiều nếu tăng quy mô    |
| **Ứng dụng thực tế**     | Fixed workload, hệ thống hiện tại      | Scalable workload, big data, distributed computing |
| **Bài học**              | Phải giảm phần serial trước khi scale  | Có thể scale tốt nếu workload tăng theo tài nguyên |

      **Cái nào đúng? Cái nào sai?**
      
      ✅ **CẢ HAI ĐỀU ĐÚNG** - Không có cái nào sai!
      
      - Cả hai định luật đều **toán học chính xác**, nhưng áp dụng cho **giả định khác nhau**
      - **Amdahl's Law** đúng khi: Workload **cố định**, không thay đổi khi thêm tài nguyên
      - **Gustafson's Law** đúng khi: Workload **có thể tăng** cùng với tài nguyên
      
      **Ví dụ cụ thể:**
      
      **Amdahl's Law đúng khi:**
      
      **1. Multithreading:**
      - Xử lý 1 file 10GB với 1 CPU → muốn xử lý nhanh hơn với 10 CPU
      - Workload cố định: vẫn là 10GB, không tăng
      - Kết quả: Speedup bị giới hạn bởi phần serial (đọc file, ghi kết quả)
      
      **2. System Design - API Server với Database Bottleneck:**
      - API server có 10 instances, nhưng tất cả query vào 1 database primary
      - Workload cố định: 1K QPS, không tăng
      - Bottleneck: Database primary xử lý 100% query (phần serial)
      - Kết quả: Thêm API server không giúp tăng throughput, bị giới hạn bởi DB
      - **Giải pháp**: Phải fix DB bottleneck trước (read replica, sharding, cache)
      
      **3. System Design - Hot Key Redis:**
      - Có 10 Redis nodes, nhưng 1 key `user:123` nhận 50% traffic
      - Workload cố định: 10K requests/giây, không tăng
      - Bottleneck: 1 key xử lý 50% traffic (phần serial)
      - Kết quả: Thêm Redis node không giúp, vì 1 key vẫn là bottleneck
      - **Giải pháp**: Phải shard key hoặc cache local trước
      
      **4. System Design - Single Leader:**
      - Có 10 application servers, nhưng chỉ 1 database leader xử lý tất cả write
      - Workload cố định: 5K write/giây, không tăng
      - Bottleneck: 1 leader xử lý 100% write (phần serial)
      - Kết quả: Thêm app server không giúp tăng write throughput
      - **Giải pháp**: Phải shard database hoặc multi-leader trước
      
      **5. Scalability - Vertical Scaling:**
      - API server 1K QPS, muốn tăng lên 2K QPS bằng cách upgrade server
      - Workload cố định: 2K QPS, không tăng
      - Bottleneck: I/O, network latency (phần serial)
      - Kết quả: Upgrade CPU/RAM không đạt 2x speedup, bị giới hạn bởi I/O
      - **Giải pháp**: Phải optimize I/O hoặc dùng horizontal scaling
      
      **Gustafson's Law đúng khi:**
      
      **1. Multithreading:**
      - Có 10 CPU → xử lý 10 file, mỗi file 10GB (tổng 100GB)
      - Workload tăng theo tài nguyên: nhiều CPU hơn → xử lý nhiều data hơn
      - Kết quả: Speedup gần tuyến tính vì mỗi CPU xử lý 1 file riêng
      
      **2. System Design - Horizontal Scaling:**
      - API server 1K QPS, muốn tăng lên 10K QPS
      - Workload tăng: Thêm 10 server, mỗi server xử lý 1K QPS độc lập
      - Kết quả: Throughput tăng gần tuyến tính (10x) với số server
      
      **3. System Design - Read Replica:**
      - Database primary xử lý 100% read + write
      - Workload tăng: Thêm 5 read replica, mỗi replica xử lý read request độc lập
      - Kết quả: Read throughput tăng gần tuyến tính với số replica
      
      **4. System Design - Sharding:**
      - Database 1TB data, muốn scale lên 10TB
      - Workload tăng: Chia thành 10 shard, mỗi shard xử lý data độc lập
      - Kết quả: Write throughput tăng gần tuyến tính với số shard
      
      **Khi nào dùng cái nào?**

| Tình huống                          | Dùng định luật nào | Lý do                                              |
|-------------------------------------|--------------------|----------------------------------------------------|
| API server xử lý request cố định    | **Amdahl**         | Số request/giây cố định, không tăng theo số server |
| Database query optimization         | **Amdahl**         | Query cố định, chỉ muốn chạy nhanh hơn             |
| Big data processing (Hadoop, Spark) | **Gustafson**      | Nhiều node hơn → xử lý nhiều data hơn              |
| Video rendering, image processing   | **Gustafson**      | Nhiều CPU hơn → render nhiều frame hơn             |
| Web scraping với rate limit         | **Amdahl**         | Rate limit cố định, không tăng theo số worker      |
| Distributed training ML             | **Gustafson**      | Nhiều GPU hơn → train với dataset lớn hơn          |

      **Kết luận:**
      - **Amdahl's Law**: Cảnh báo rằng lợi ích của song song hóa có giới hạn nghiêm ngặt, phải giảm bottleneck serial
      - **Gustafson's Law**: Cho thấy nếu ta **tăng quy mô vấn đề** cùng với tăng tài nguyên, lợi ích có thể tăng gần tuyến tính
      - **Trong thực tế**: Cả hai đều đúng tùy vào context. Chọn định luật nào phụ thuộc vào **workload có thể scale hay không**
      
      **🔧 Liên hệ tới Multithreading:**
      
      **Amdahl's Law áp dụng khi:**
      - Xử lý **1 task cố định** với nhiều thread
      - Ví dụ: Sort 1 mảng 1 triệu phần tử với 4 threads
        - Task cố định: vẫn là 1 mảng 1 triệu phần tử
        - Phần serial: chia mảng, merge kết quả
        - Speedup bị giới hạn bởi phần merge (serial)
      - **Khi nào dùng**: Khi muốn xử lý **nhanh hơn** 1 task cụ thể
      - **Bottleneck**: Lock contention, critical section, synchronization overhead
      
      **Gustafson's Law áp dụng khi:**
      - Xử lý **nhiều task độc lập** với nhiều thread
      - Ví dụ: Xử lý 1000 request với 4 threads
        - Workload tăng: 4 threads → xử lý 4 request đồng thời
        - Mỗi thread xử lý 1 request riêng (không cần sync)
        - Speedup gần tuyến tính: 4x với 4 threads
      - **Khi nào dùng**: Khi có **nhiều task độc lập** có thể xử lý song song
      - **Lợi ích**: Không có bottleneck serial giữa các task
      
      **Bảng: Loại Serial trong Multithreading**

| Loại Serial (Multithreading)  | Ví dụ                           | Vì sao làm chậm             | Định luật áp dụng | Giải thích                                                  |
|-------------------------------|---------------------------------|-----------------------------|-------------------|-------------------------------------------------------------|
| 🔒 **Lock / synchronized**    | `synchronized`, `ReentrantLock` | Chỉ 1 thread vào → xếp hàng | **Amdahl**        | Lock block > 10% thread → phải fix (fine-grained)           |
| 💾 **IO blocking**            | Đọc file, gọi API, upload       | Thread đứng chờ             | **Amdahl**        | IO blocking > 10% time → phải fix (async, non-blocking)     |
| 🧹 **GC Pause (Java)**        | Stop-the-world GC               | Tất cả đứng im              | **Amdahl**        | GC pause > 10% time → phải fix (tune GC, reduce allocation) |
| 📝 **Logging đồng bộ**        | File log sync                   | Block thread                | **Amdahl**        | Sync logging > 10% time → phải fix (async logging)          |
| 🔁 **Single-thread executor** | `newSingleThreadExecutor()`     | Ép về 1 luồng               | **Gustafson**     | Nếu có nhiều executor khác → không đáng kể                  |

      **Decision Matrix - Multithreading**

| Tình huống            | Câu hỏi                                       | Amdahl (Phải fix)                | Gustafson (Có thể bỏ qua)                |
|-----------------------|-----------------------------------------------|----------------------------------|------------------------------------------|
| **Thread Pool Size**  | Serial task chiếm bao nhiêu % thời gian?      | > 10% → Giảm serial (lock, sync) | < 1% → Không cần fix, có nhiều task khác |
| **Lock Contention**   | Lock này block bao nhiêu % thread?            | > 10% thread → Fine-grained lock | < 1% thread → Không cần fix              |
| **Critical Section**  | Critical section chiếm bao nhiêu % thời gian? | > 10% → Optimize, giảm thời gian | < 1% → Không cần fix                     |
| **Task Distribution** | 1 task lớn vs nhiều task nhỏ?                 | 1 task lớn → Chia nhỏ (Amdahl)   | Nhiều task nhỏ → Thread pool (Gustafson) |
| **Context Switch**    | Context switch overhead?                      | > 10% → Giảm số thread           | < 1% → Không cần fix                     |

      **Ví dụ Multithreading:**

| Tình huống                          | Dùng định luật nào | Giải thích                                             |
|-------------------------------------|--------------------|--------------------------------------------------------|
| Sort 1 mảng lớn với nhiều threads   | **Amdahl**         | Task cố định (1 mảng), phần merge là serial bottleneck |
| Xử lý 1000 request với thread pool  | **Gustafson**      | Nhiều request độc lập, mỗi thread xử lý 1 request      |
| Parallel for loop xử lý array       | **Amdahl**         | Array cố định, chia nhỏ và xử lý, nhưng có overhead    |
| Producer-Consumer với nhiều workers | **Gustafson**      | Nhiều item trong queue, mỗi worker xử lý item riêng    |
| Tính toán matrix với shared memory  | **Amdahl**         | Matrix cố định, chia nhỏ nhưng có memory contention    |
| Web server xử lý nhiều HTTP request | **Gustafson**      | Nhiều request độc lập, mỗi thread xử lý 1 request      |
| Image processing: xử lý nhiều ảnh   | **Gustafson**      | Nhiều ảnh độc lập, mỗi thread xử lý 1 ảnh              |
| Image processing: xử lý 1 ảnh lớn   | **Amdahl**         | 1 ảnh cố định, chia nhỏ nhưng có overhead merge        |

      **Bài học cho Multithreading:**
      - **Dùng Amdahl khi**: Cần xử lý nhanh 1 task lớn → phải giảm phần serial (lock, sync)
      - **Dùng Gustafson khi**: Có nhiều task độc lập → tăng số thread để xử lý nhiều task hơn
      - **Thực tế**: Hầu hết multithreading apps dùng **Gustafson** vì thường có nhiều task độc lập (request, job, item)
      - **Thread Pool**: Trong thực tế, hầu hết ứng dụng đều dùng Thread Pool thay vì tạo thread trực tiếp
      
      **🎯 FRAMEWORK THỰC TẾ: Khi Nào Dùng Amdahl vs Gustafson (System Design)**
      
      **Bước 1: Phân tích Bottleneck**
      
      Với mỗi bottleneck, hỏi 2 câu:
      1. **Bottleneck chiếm bao nhiêu % workload?**
         - > 10% → **Nghe Amdahl** → Phải giảm bottleneck
         - < 1% → **Nghe Gustafson** → Không đáng kể, có thể bỏ qua
         - 1-10% → **Cân nhắc** → Tùy vào cost/benefit
      2. **Workload có thể tăng không?**
         - Cố định → **Nghe Amdahl** → Phải giảm bottleneck
         - Có thể tăng → **Nghe Gustafson** → Tăng workload để bottleneck không đáng kể
      
      **Bước 2: Decision Matrix - System Design**

| Bottleneck            | Câu hỏi                                  | Amdahl (Phải fix)                        | Gustafson (Có thể bỏ qua)                             |
|-----------------------|------------------------------------------|------------------------------------------|-------------------------------------------------------|
| **Hot Key Redis**     | Key này chiếm bao nhiêu % traffic?       | > 10% traffic → Shard key, cache local   | < 1% traffic → Không cần fix, có nhiều key khác       |
| **Single Leader**     | Leader này xử lý bao nhiêu % write?      | > 10% write → Shard, multi-leader        | < 1% write → Không cần fix, có nhiều partition khác   |
| **Lock Global**       | Lock này block bao nhiêu % request?      | > 10% request → Fine-grained lock        | < 1% request → Không cần fix, có nhiều task khác      |
| **Queue 1 Partition** | Partition này xử lý bao nhiêu % message? | > 10% message → Tăng partition           | < 1% message → Không cần fix, có nhiều partition khác |
| **File dùng chung**   | File này xử lý bao nhiêu % I/O?          | > 10% I/O → File riêng, shard            | < 1% I/O → Không cần fix, có nhiều file khác          |
| **Database**          | DB này xử lý bao nhiêu % query?          | > 10% query → Read replica, shard, cache | < 1% query → Không cần fix, có nhiều DB khác          |

      **Bước 3: Scalability trong System Design - Liên hệ với Amdahl/Gustafson**
      
      **Định nghĩa Scalability:**
      - Scalability = Throughput (RPS, QPS, TPS) tăng theo số server/tài nguyên
      - Khả năng hệ thống xử lý nhiều workload hơn khi thêm tài nguyên
      
      **Vertical Scaling (Scale Up) - Amdahl's Law:**
      - Tăng sức mạnh 1 server (CPU, RAM, I/O)
      - **Workload cố định** → Muốn xử lý nhanh hơn
      - **Bottleneck**: Giới hạn phần cứng (max CPU, max RAM)
      - **Áp dụng Amdahl**: Phần serial (I/O, network) giới hạn speedup
      - **Ví dụ**: 1 server mạnh hơn xử lý cùng 1 lượng request → bị giới hạn bởi I/O, network
      
      **Horizontal Scaling (Scale Out) - Gustafson's Law:**
      - Thêm nhiều server để xử lý nhiều workload hơn
      - **Workload tăng** → Mỗi server xử lý phần workload riêng
      - **Bottleneck**: Load balancing, data consistency
      - **Áp dụng Gustafson**: Nhiều server → xử lý nhiều request độc lập
      - **Ví dụ**: 10 server xử lý 10x request → speedup gần tuyến tính
      
      **Decision Matrix - Scalability Patterns:**

| Scaling Type           | Workload         | Định luật     | Khi nào dùng                       | Bottleneck                  |
|------------------------|------------------|---------------|------------------------------------|-----------------------------|
| **Vertical Scaling**   | Cố định          | **Amdahl**    | Workload nhỏ, muốn xử lý nhanh hơn | I/O, network, phần cứng max |
| **Horizontal Scaling** | Tăng theo server | **Gustafson** | Workload lớn, muốn xử lý nhiều hơn | Load balancing, consistency |
| **Read Replica**       | Read tăng        | **Gustafson** | Nhiều read request độc lập         | Write consistency           |
| **Sharding**           | Data tăng        | **Gustafson** | Data lớn, chia thành nhiều shard   | Cross-shard query           |
| **Caching**            | Read tăng        | **Gustafson** | Nhiều read request giống nhau      | Cache invalidation          |

      **Scalability Metrics:**
      - **Throughput**: RPS/QPS tăng theo số server
      - **Latency**: Thời gian xử lý 1 request
      - **Amdahl**: Latency giảm nhưng bị giới hạn bởi phần serial
      - **Gustafson**: Throughput tăng gần tuyến tính với số server
      
      **Bước 4: Rule of Thumb Thực Tế**
      
      **Nghe Amdahl khi:**
      - ✅ Bottleneck > 10% workload
      - ✅ Workload cố định, không thể tăng
      - ✅ Có thể fix với cost hợp lý
      
      **Nghe Gustafson khi:**
      - ✅ Bottleneck < 1% workload
      - ✅ Workload có thể tăng theo tài nguyên
      - ✅ Có nhiều task/workload độc lập khác
      - ✅ Fix bottleneck tốn kém hơn lợi ích
      
      **Bước 5: Checklist Khi Design System**
      
      1. **Đo lường**: Bottleneck chiếm bao nhiêu % workload?
      2. **Quyết định**:
         - > 10% → Fix ngay (Amdahl)
         - 1-10% → Cân nhắc cost/benefit
         - < 1% → Có thể bỏ qua (Gustafson)
      3. **Giải pháp**: Shard, cache, optimize, fine-grained
      4. **Verify**: Sau khi fix, bottleneck còn bao nhiêu %?
      
      **Bước 6: Ví Dụ Thực Tế - System Design**
      
      **Ví dụ 1: Hot Key Redis (System)**
      - **Tình huống**: Key `user:123` nhận 50% traffic
      - **Phân tích**: 50% > 10% → **Nghe Amdahl**
      - **Giải pháp**: Shard key → `user:123:shard0`, `user:123:shard1`
      - **Kết quả**: Mỗi shard chỉ còn 25% → OK
      
      **Ví dụ 2: Single Leader (System)**
      - **Tình huống**: 1 leader xử lý 0.5% write, có 200 partition khác
      - **Phân tích**: 0.5% < 1% → **Nghe Gustafson**
      - **Giải pháp**: Không cần fix, có nhiều partition khác
      - **Kết quả**: Bottleneck không đáng kể → OK
      
      **🔧 Ví Dụ Thực Tế - Multithreading (Xem phần "Liên hệ tới Multithreading" ở trên)**
      
      **Ví dụ 1: Thread Pool với Lock Contention**
      - **Tình huống**: Global lock block 30% thread time
      - **Phân tích**: 30% > 10% → **Nghe Amdahl**
      - **Giải pháp**: Fine-grained lock → `lock(userId)` thay vì `lock(global)`
      - **Kết quả**: Lock contention giảm xuống < 1% → OK
      
      **Ví dụ 2: Thread Pool với Nhiều Task**
      - **Tình huống**: Thread pool 10 threads, xử lý 1000 request độc lập
      - **Phân tích**: Nhiều task độc lập → **Nghe Gustafson**
      - **Giải pháp**: Không cần fix, mỗi thread xử lý 1 request riêng
      - **Kết quả**: Speedup gần tuyến tính → OK
      
      **Ví dụ 3: Vertical Scaling (Scalability - Amdahl)**
      - **Tình huống**: API server 1K QPS, muốn tăng lên 2K QPS bằng cách upgrade server
      - **Phân tích**: Workload cố định (2K QPS), muốn xử lý nhanh hơn → **Nghe Amdahl**
      - **Giải pháp**: Upgrade CPU, RAM → nhưng bị giới hạn bởi I/O, network (phần serial)
      - **Kết quả**: Speedup bị giới hạn, không đạt 2x → Cần horizontal scaling
      
      **Ví dụ 4: Horizontal Scaling (Scalability - Gustafson)**
      - **Tình huống**: API server 1K QPS, muốn tăng lên 10K QPS
      - **Phân tích**: Workload tăng (10K QPS), mỗi server xử lý request độc lập → **Nghe Gustafson**
      - **Giải pháp**: Thêm 10 server, load balancer phân phối request
      - **Kết quả**: Throughput tăng gần tuyến tính (10x) → OK
      
      **Ví dụ 5: Read Replica (Scalability - Gustafson)**
      - **Tình huống**: Database primary xử lý 100% read + write, muốn scale read
      - **Phân tích**: Nhiều read request độc lập → **Nghe Gustafson**
      - **Giải pháp**: Thêm read replica, phân phối read request
      - **Kết quả**: Read throughput tăng gần tuyến tính với số replica → OK
      
      **Bước 7: Tóm Tắt Framework**
      
      ```
      Bottleneck > 10% workload? 
        → YES: Fix ngay (Amdahl)
          → System: Shard, cache, optimize
          → Scalability: Vertical scaling bị giới hạn, cần horizontal
        → NO: Bottleneck < 1%?
          → YES: Bỏ qua (Gustafson)
            → System: Có nhiều partition/key khác
            → Scalability: Horizontal scaling, read replica, sharding
          → NO: Cân nhắc cost/benefit
      
      Scalability Decision:
        Workload cố định? 
          → YES: Vertical scaling (Amdahl) - bị giới hạn
          → NO: Horizontal scaling (Gustafson) - gần tuyến tính
      
      Multithreading: Xem phần "Liên hệ tới Multithreading" ở trên
      ```
      
      **⚠️ GIỚI HẠN CỦA AMDahl/GUSTAFSON - Những Gì Cần Bổ Sung**
      
      **Câu hỏi: Chỉ dùng 2 định luật này đã đủ để giải quyết tất cả trường hợp scale khi design system chưa?**
      
      **Trả lời: ❌ CHƯA ĐỦ** - Amdahl/Gustafson chỉ giải quyết một phần của scaling
      
      **✅ Những gì Amdahl/Gustafson giải quyết:**
      
      1. **Performance/Speedup khi scale**
         - Giới hạn tăng tốc khi có bottleneck serial
         - Tối ưu hóa parallelization
         - Quyết định fix bottleneck hay không
      
      2. **Workload distribution**
         - Khi nào chia nhỏ workload
         - Khi nào tăng workload cùng với tài nguyên
      
      **❌ Những gì Amdahl/Gustafson KHÔNG giải quyết:**
      
      1. **Consistency & CAP Theorem**
         - Trade-off giữa Consistency, Availability, Partition tolerance
         - Strong consistency vs eventual consistency
         - Amdahl/Gustafson không nói về data consistency
         - **Cần học**: CAP theorem, ACID vs BASE, consistency patterns
      
      2. **Availability & Reliability**
         - Redundancy patterns (Active-Active, Active-Passive)
         - Failure handling, circuit breaker
         - Disaster recovery
         - MTBF, MTTR
         - **Cần học**: Availability patterns, failure modes, redundancy strategies
      
      3. **Network & Latency**
         - Network latency khi scale geographically
         - CDN, edge computing
         - Data locality
         - Amdahl/Gustafson giả định network không phải bottleneck
         - **Cần học**: Network topology, CDN, edge computing, data locality
      
      4. **Cost & Economics**
         - Cost per request khi scale
         - ROI của scaling
         - Amdahl/Gustafson không tính cost
         - **Cần học**: Cost optimization, TCO (Total Cost of Ownership)
      
      5. **Complexity & Operations**
         - Operational complexity khi scale
         - Monitoring, observability
         - Debugging distributed systems
         - Amdahl/Gustafson không tính complexity overhead
         - **Cần học**: Observability, monitoring, distributed tracing, logging
      
      6. **Data Partitioning Strategies**
         - Sharding strategies (range, hash, directory-based)
         - Rebalancing khi scale
         - Cross-shard queries
         - Amdahl/Gustafson không nói về cách partition
         - **Cần học**: Sharding strategies, partition keys, rebalancing
      
      7. **Load Balancing Strategies**
         - Round-robin, weighted, consistent hashing
         - Session affinity
         - Health checks
         - Amdahl/Gustafson giả định load balancing hoàn hảo
         - **Cần học**: Load balancing algorithms, session management
      
      8. **Caching Strategies**
         - Cache invalidation
         - Cache warming
         - Cache consistency
         - Amdahl/Gustafson không nói về caching
         - **Cần học**: Cache patterns, invalidation strategies, cache coherence
      
      9. **Message Queue Patterns**
         - Pub/sub, point-to-point
         - Message ordering
         - Exactly-once delivery
         - Amdahl/Gustafson không nói về messaging
         - **Cần học**: Message queue patterns, event-driven architecture
      
      10. **Database Replication Patterns**
          - Master-slave, master-master
          - Read replicas, write replicas
          - Replication lag
          - Amdahl/Gustafson không nói về replication
          - **Cần học**: Replication patterns, read/write splitting, lag handling
      
      **📚 Tóm Tắt:**

| Khía cạnh          | Amdahl/Gustafson | Cần học thêm                      |
|--------------------|------------------|-----------------------------------|
| **Performance**    | ✅ Giải quyết     | -                                 |
| **Consistency**    | ❌ Không          | CAP theorem, ACID vs BASE         |
| **Availability**   | ❌ Không          | Redundancy, failure handling      |
| **Network**        | ❌ Không          | CDN, edge computing, latency      |
| **Cost**           | ❌ Không          | Cost optimization, TCO            |
| **Operations**     | ❌ Không          | Monitoring, observability         |
| **Partitioning**   | ❌ Không          | Sharding strategies               |
| **Load Balancing** | ❌ Không          | LB algorithms, session management |
| **Caching**        | ❌ Không          | Cache patterns, invalidation      |
| **Messaging**      | ❌ Không          | Message queue patterns            |
| **Replication**    | ❌ Không          | Replication patterns              |

      **Kết luận:**
      - Amdahl/Gustafson là **nền tảng quan trọng** để hiểu performance khi scale
      - Nhưng **chưa đủ** để design system hoàn chỉnh
      - Cần học thêm: CAP theorem, Availability, Network, Cost, Operations, và các patterns khác
      - Amdahl/Gustafson giúp **quyết định khi nào scale**, nhưng không nói **cách scale như thế nào**

### Performance Metrics

- [ ] Định nghĩa chính xác: Latency, Throughput, QPS, TPS, RPS 
  - Latency = thời gian một request/operation bắt đầu cho đến khi hoàn thành 
  - Throughput = số lượng operation mà hệ thống xử lý thành công trên một đơn vị thời gian 
  - QPS = Queries per second = Số lượng query mà hệ thống xử lý trong một giây 
  - RPS = Requests per second = Số lượng request mà hệ thống nhận/xử lý trong một giây 
  - TPS = Transactions per second = Số lượng transaction hoàn chỉnh (atomic, commit thành công) mà hệ thống xử lý trong 1 giây
- [ ] Viết công thức tính: QPS = ?
  - QPS = number of queries / total time
- [ ] Viết công thức tính: Throughput = số lượng operation hoàn thành / thời gian
- [ ] Đọc về percentile metrics (p50, p95, p99, p999)
  - Percentile = “bao nhiêu % request nhanh hơn hoặc bằng giá trị này”
- [ ] Tính toán: Nếu p95 latency = 200ms, có nghĩa là gì? (viết câu trả lời)
  - 95% request có thời gian phản hồi ≤ 200ms, 
  - 5% request còn lại > 200ms.
- [ ] Tìm hiểu: Tại sao p99 quan trọng hơn average latency?
  - Average che giấu vấn đề.
  - p99 phản ánh user xui xẻo nhất 
    - p99 = latency của 1% user chậm nhất. 
  - Nó trả lời câu hỏi:
    - “User tệ nhất đang trải nghiệm thế nào?” 
  - Với hệ thống lớn:
    - 1% của 1 triệu user = 10.000 người 😱 
  - → Không thể bỏ qua.
- [ ] Đọc về "tail latency" và "latency SLOs"
  - Tail = những request rất chậm. 
  - Ví dụ:
    - 95% < 200ms 
    - 5% > 1s 
    - 1% > 5s 
    - → Đó là tail latency.
  - SLO = Service Level Objective 
    - Mục tiêu chất lượng dịch vụ. 
    - Ví dụ: 99% request phải < 300ms trong 30 ngày. 
    - Hoặc: p99 latency ≤ 500ms.
      | Khái niệm | Ai dùng                |
      | --------- | ---------------------- |
      | SLO       | Internal (team tự đặt) |
      | SLA       | Cam kết với khách      |

### Availability Concepts

- [ ] Tính toán downtime cho: 99%, 99.9%, 99.99%, 99.999% (theo năm, tháng, tuần, ngày)
  - Downtime = (1 − Availability) × Tổng thời gian
- [ ] Viết bảng so sánh: Availability % → Downtime/year → Downtime/month
| Availability | Downtime/year | Downtime/month | Downtime/week | Downtime/day |
|--------------|----------------|----------------|----------------|---------------|
| 99%          | 3.65 days      | 7.2 hours      | 1.68 hours     | 14.4 minutes  |
| 99.9%        | 8.76 hours     | 43.2 minutes   | 10.08 minutes | 1.44 minutes  |
| 99.99%       | 52.56 minutes  | 4.32 minutes   | 1.008 minutes | 8.64 seconds  |
| 99.999%      | 5.256 minutes  | 25.92 seconds  | 6.048 seconds | 0.864 seconds |

- [ ] Đọc về "nines" trong availability (3 nines, 4 nines, 5 nines)
  - Nines = số chữ số 9 trong availability %
  - 3 nines = 99.9%
  - 4 nines = 99.99%
  - 5 nines = 99.999%
  - Mỗi nines tăng thêm → Giảm downtime đáng kể

- [ ] Định nghĩa: Single Point of Failure (SPOF)
  - SPOF = Thành phần hệ thống mà nếu nó hỏng → toàn bộ hệ thống ngừng hoạt động
  - Ví dụ: 1 database server không có replica → nếu nó hỏng → hệ thống không hoạt động
- [ ] Định nghĩa: Mean Time Between Failures (MTBF)
  - MTBF = Thời gian trung bình giữa các lần hỏng hóc
  - Ví dụ: Nếu MTBF = 1000 giờ → Trung bình mỗi 1000 giờ sẽ có 1 lần hỏng
- [ ] Định nghĩa: Mean Time To Recovery (MTTR)
  - MTTR = Thời gian trung bình để khôi phục sau khi hỏng
  - Ví dụ: Nếu MTTR = 2 giờ → Trung bình mất 2 giờ để sửa chữa và khôi phục
- [ ] Công thức: Availability = MTBF / (MTBF + MTTR) - verify và hiểu
    - Availability = MTBF / (MTBF + MTTR)
    - Ví dụ: MTBF = 1000 giờ, MTTR = 2 giờ
        - Availability = 1000 / (1000 + 2) ≈ 99.8%

### Redundancy Patterns

- [ ] Đọc về Active-Active redundancy pattern
  - Active-Active: Tất cả các nodes đều hoạt động và xử lý traffic cùng lúc. Nếu một node hỏng, các node còn lại tiếp tục phục vụ mà không gián đoạn.
- [ ] Đọc về Active-Passive (Hot Standby) redundancy pattern
  - Active-Passive: Một node chính (active) xử lý traffic, trong khi node phụ (passive) ở trạng thái chờ. Nếu node chính hỏng, node phụ sẽ được kích hoạt để tiếp quản.
- [ ] Đọc về Active-Passive (Cold Standby) redundancy pattern
  - Active-Passive (Cold Standby): Node phụ không hoạt động và không sẵn sàng ngay lập tức. Khi node chính hỏng, node phụ cần thời gian để khởi động và tiếp quản.
- [ ] So sánh: Active-Active vs Active-Passive (3 điểm khác biệt)
  - 1️⃣ Hiệu suất & Tài nguyên 
    - Active-Active: Tất cả node đều xử lý request → tận dụng tối đa tài nguyên. 
    - Active-Passive: Chỉ node chính hoạt động → node phụ gần như “để không”.
  - 2️⃣ Thời gian Failover (Chuyển đổi khi lỗi)
    - Active-Active: Gần như không gián đoạn (vì node khác đang chạy sẵn). 
    - Active-Passive: Phải chờ chuyển sang node phụ → có downtime ngắn. 
  - 3️⃣ Độ phức tạp & Chi phí
    - Active-Active:
    → Phức tạp hơn (sync data, conflict)
    → Chi phí cao hơn.

    - Active-Passive:
    → Dễ quản lý
    → Chi phí thấp hơn.

  - 📌 Bản rút gọn để học thuộc 
    - Nếu cần nhớ nhanh trong phỏng vấn/thi:
      - Active-Active: Nhanh – Tốn tiền – Phức tạp 
      - Active-Passive: Rẻ – Dễ – Chậm hơn khi fail
- [ ] Tìm 2 real-world examples của Active-Active
  - 1. Hệ thống DNS toàn cầu (ví dụ: Google Public DNS)
    - Nhiều server DNS hoạt động đồng thời trên toàn thế giới để xử lý các truy vấn DNS. Nếu một server gặp sự cố, các server khác vẫn tiếp tục phục vụ người dùng mà không gián đoạn.
  - 2. Hệ thống cân bằng tải web (Load Balancer)
    - Nhiều máy chủ web hoạt động song song để xử lý các yêu cầu từ người dùng. Nếu một máy chủ gặp sự cố, các máy chủ còn lại vẫn tiếp tục phục vụ mà không ảnh hưởng đến trải nghiệm người dùng.
- [ ] Tìm 2 real-world examples của Active-Passive
  - 1. Hệ thống cơ sở dữ liệu với replica
    - Một cơ sở dữ liệu chính (primary) xử lý tất cả các giao dịch, trong khi một bản sao (replica) ở trạng thái chờ. Nếu cơ sở dữ liệu chính gặp sự cố, bản sao sẽ được kích hoạt để tiếp quản.
    - Ví dụ: MySQL Master-Slave Replication. 
  - 2. Hệ thống máy chủ ứng dụng với máy chủ dự phòng
    - Một máy chủ ứng dụng chính xử lý tất cả các yêu cầu từ người dùng, trong khi một máy chủ dự phòng ở trạng thái chờ. Nếu máy chủ chính gặp sự cố, máy chủ dự phòng sẽ được kích hoạt để tiếp quản.
    - Ví dụ: Hệ thống web sử dụng Nginx với một máy chủ dự phòng.

### Bottleneck Identification

- [ ] Liệt kê 4 loại bottlenecks chính: CPU, Memory, I/O, Network
  - CPU Bottleneck: Khi CPU đạt 100% sử dụng, không thể xử lý thêm yêu cầu.
  - Memory Bottleneck: Khi hệ thống hết RAM, dẫn đến việc sử dụng swap hoặc crash.
  - I/O Bottleneck: Khi tốc độ đọc/ghi đĩa chậm, làm chậm toàn bộ hệ thống.
  - Network Bottleneck: Khi băng thông thấp hoặc latency cao → request timeout, service giao tiếp chậm.
- [ ] Với mỗi bottleneck, viết 2 cách identify
  - CPU Bottleneck:
    1. Sử dụng công cụ giám sát hệ thống (như top, htop) để kiểm tra mức sử dụng CPU.
    2. Phân tích logs để tìm các request có thời gian xử lý lâu, có thể do CPU quá tải.
  - Memory Bottleneck:
    1. Kiểm tra mức sử dụng RAM bằng công cụ giám sát hệ thống (như free, vmstat).
    2. Quan sát các lỗi liên quan đến OutOfMemory hoặc swap usage trong logs.
  - I/O Bottleneck:
    1. Sử dụng công cụ như iostat để kiểm tra tốc độ đọc/ghi đĩa và thời gian chờ I/O.
    2. Phân tích logs để tìm các request có thời gian xử lý lâu liên quan đến I/O.
  - Network Bottleneck:
    1. Sử dụng công cụ như iftop hoặc nload để giám sát băng thông mạng.
    2. Kiểm tra logs để tìm các lỗi timeout hoặc độ trễ cao trong giao tiếp giữa các dịch vụ.
  
- [ ] Với mỗi bottleneck, viết 2 cách resolve
  - CPU Bottleneck:
    1. Tối ưu hóa code để giảm tải CPU (ví dụ: giảm độ phức tạp thuật toán).
    2. Thêm nhiều instance của service để phân phối tải (horizontal scaling).
  - Memory Bottleneck:
    1. Tối ưu hóa việc sử dụng bộ nhớ trong ứng dụng (ví dụ: giảm memory leaks).
    2. Nâng cấp phần cứng hoặc thêm swap space.
  - I/O Bottleneck:
    1. Sử dụng caching để giảm số lần truy cập đĩa.
    2. Nâng cấp phần cứng lưu trữ (ví dụ: sử dụng SSD thay vì HDD).
  - Network Bottleneck:
    1. Tối ưu hóa giao tiếp mạng (ví dụ: giảm kích thước payload).
    2. Sử dụng CDN hoặc edge servers để giảm tải mạng.
- [ ] Đọc về "Amdahl's Law" trong context của bottlenecks
  - Amdahl's Law: Tốc độ tối đa của hệ thống khi cải thiện một phần phụ thuộc vào tỷ lệ phần đó trong tổng thời gian xử lý.
  - Công thức: Speedup = 1 / (S + P/N)
    - S = Phần serial (không thể parallel)
    - P = Phần parallel (có thể parallel)
    - N = Số lượng đơn vị xử lý (cores, servers)
    - → Nếu phần serial lớn (bottleneck) → speedup bị giới hạn.
    - → Cần giảm phần serial (bottleneck) để đạt speedup tốt hơn.
    - Ví dụ: Nếu 30% thời gian là serial → max speedup = 1 / 0.3 ≈ 3.33x dù có bao nhiêu cores.
    - → Giải pháp: Giảm phần serial (bottleneck) để cải thiện hiệu suất tổng thể.

### Capacity Planning

- [ ] Đọc về "back-of-envelope calculations"
  - Back-of-envelope calculations = Ước lượng nhanh, sơ bộ để có cái nhìn tổng quan về yêu cầu hệ thống.
  - Không cần chính xác tuyệt đối, chỉ cần đủ gần để đưa ra quyết định ban đầu.
  - Ví dụ: Ước lượng số lượng server cần thiết dựa trên QPS và khả năng xử lý của mỗi server.
  - Giúp nhanh chóng đánh giá tính khả thi của thiết kế hệ thống.
  - Thường dùng trong giai đoạn early design để tránh over-engineering hoặc under-provisioning.
  - Ví dụ: Nếu mỗi server xử lý được 100 QPS và cần phục vụ 10,000 QPS → Cần ít nhất 100 servers (10,000 / 100).
  - Giúp xác định các yêu cầu về tài nguyên như CPU, RAM, Storage, Bandwidth.
  - Cần kết hợp với các yếu tố khác như redundancy, peak load, growth rate để có kế hoạch chính xác hơn.
- [ ] Học cách estimate: Storage requirements
  - Estimate storage = Số lượng users × Data per user
  - Cân nhắc growth rate (dự kiến tăng bao nhiêu data theo thời gian)
  - Thêm buffer (dự phòng) cho unexpected growth
  - Xem xét loại data (structured, unstructured) để chọn storage phù hợp
  - Tính toán backup storage nếu cần
  - Xem xét retention policy (lưu trữ bao lâu)
  - 
  - Tính toán tổng storage cần thiết = Current data + Growth + Buffer + Backup
  - Ví dụ: 1M users × 10MB/user = 10TB + growth + buffer
  - Chọn storage type (HDD, SSD, Cloud storage) dựa trên performance và cost
  - Lập kế hoạch mở rộng storage khi cần thiết
  - Đánh giá cost liên quan đến storage (per GB/month)
  - Xem xét data compression để giảm storage usage
  - Tính toán IOPS requirements nếu cần cho performance
  - Xem xét data access patterns (read-heavy, write-heavy) để chọn storage phù hợp
  
- [ ] Học cách estimate: Bandwidth requirements
  - Estimate bandwidth = QPS × Average response size
  - Cân nhắc peak load (gấp bao nhiêu lần so với average)
  - Thêm buffer (dự phòng) cho unexpected spikes
  - Xem xét loại traffic (inbound, outbound)
  - Tính toán tổng bandwidth cần thiết = Average bandwidth + Peak load + Buffer
  - Ví dụ: 10K QPS × 2KB = 20MB/s + peak + buffer
  - Chọn network type (dedicated, shared, cloud) dựa trên performance và cost
  - Lập kế hoạch mở rộng bandwidth khi cần thiết
  - Đánh giá cost liên quan đến bandwidth (per GB/month)
  - Xem xét data transfer patterns (bursty, steady) để chọn network phù hợp
  - Tính toán latency requirements nếu cần cho performance
  
- [ ] Học cách estimate: Compute requirements
  - Estimate compute = QPS / Requests per CPU core
  - Cân nhắc peak load (gấp bao nhiêu lần so với average)
  - Thêm buffer (dự phòng) cho unexpected spikes
  - Tính toán tổng compute cần thiết = Average compute + Peak load + Buffer
  - Ví dụ: 10K QPS / 100 RPS/core = 100 cores + peak + buffer
  
  - Chọn instance type (CPU, RAM) dựa trên performance và cost
  - Lập kế hoạch mở rộng compute khi cần thiết
  - Đánh giá cost liên quan đến compute (per hour)
  - Xem xét auto-scaling để tối ưu chi phí
  - Tính toán power và cooling requirements nếu cần cho data center
  - Xem xét workload type (CPU-bound, I/O-bound) để chọn instance phù hợp
  - Tính toán redundancy requirements (n+1, n+2) nếu cần cho availability
  - Xem xét virtualization/containerization để tối ưu resource usage
  - Tính toán licensing costs nếu dùng phần mềm có phí
- [ ] Practice: Estimate storage cho 1M users, mỗi user 10MB data
  - Step 1: 1M users × 10MB/user = 10TB
  - Step 2: Growth rate = 10%/month → 1TB/month
  - Step 3: Buffer = 20% of current = 2TB
  - Step 4: Backup storage = 50% of current = 5TB
  - Total storage = 10TB + 1TB/month + 2TB + 5TB = 18TB + growth
  - Chọn storage type: SSD cho performance tốt
  - Estimate cost: $0.10/GB/month → 18TB = $1,800/month
  - Plan for expansion: Monitor usage, add storage khi cần
  - Document calculations trong spreadsheet
  - Verify numbers có hợp lý không?
  
- [ ] Practice: Estimate bandwidth cho 10K QPS, mỗi request 2KB response
  - Step 1: 10K QPS × 2KB = 20MB/s
  - Step 2: Peak load = 5x average = 100MB/s
  - Step 3: Buffer = 20% of peak = 20MB/s
  - Total bandwidth = 20MB/s + 100MB/s + 20MB/s = 140MB/s
  - Chọn network type: Dedicated connection
  - Estimate cost: $0.05/GB/month → 140MB/s ≈ 360TB/month = $18,000/month
  - Plan for expansion: Monitor usage, upgrade bandwidth khi cần
  - Document calculations trong spreadsheet
  - Verify numbers có hợp lý không?

---

## Design TODOs

### Design Exercise 1: Payment Gateway

- [ ] Thiết kế architecture diagram cho Payment Gateway (10K TPS, 99.9% uptime)
- [ ] Vẽ diagram với các components: API Gateway, Payment Service, Database, Cache
- [ ] Label mỗi component với estimated QPS capacity
- [ ] Identify ít nhất 3 SPOF trong design đầu tiên
- [ ] Redesign để eliminate tất cả SPOF
- [ ] Tính toán: Cần bao nhiêu instances cho mỗi service? (show calculations)
- [ ] Design redundancy strategy cho database
- [ ] Design redundancy strategy cho application servers
- [ ] Estimate: Total cost (rough) cho infrastructure
- [ ] Viết document (500 words) giải thích design decisions

#### Deliverable: Payment Gateway Design (10K TPS, 99.9% uptime)

> Mục tiêu: xây hệ thống “payment orchestration” cho merchant/app nội bộ, kết nối nhiều PSP (VNPay/MoMo/Stripe/Adyen...). Tập trung: **đúng tiền**, **idempotent**, **chịu lỗi**, **scale 10K TPS**.

##### 1) Assumptions (để tính toán)

- **Peak**: 10K TPS (transactions/second) ở endpoint tạo payment (authorize/charge).
- **Headroom**: thiết kế **2× peak** để hấp thụ spike + failover → \(TPS_{design}=20K\).
- **Read/Write split**:
  - Write path (create/confirm/capture/refund): ~20K TPS thiết kế.
  - Read path (status/query): giả sử 3× write → 60K RPS đọc (đa phần cacheable).
- **SLO**:
  - Availability: 99.9% (downtime budget ~ 43.2 phút/tháng).
  - Latency mục tiêu: p95 < 200ms cho “create intent” (không bao gồm thời gian PSP xử lý async).
- **Consistency**: ledger phải strong/serializable ở mức giao dịch; còn “status view” có thể eventual.

##### 2) High-level architecture (ASCII diagram)

```text
Clients/Merchants
     |
 [Anycast DNS]  (multi-region optional)
     |
  [WAF/CDN]  (rate-limit, bot, DDoS)
     |
  [API Gateway / LB]  (stateless, multi-AZ)
     |
  [AuthN/AuthZ]----->(JWT/OAuth2, mTLS for internal)
     |
  [Payment API / Orchestrator]  (stateless)
     |         |        |
     |         |        +--> [Idempotency Store (Redis)]  (idempotency-key -> outcome)
     |         |
     |         +--> [Fraud/Risk Service] (async + rules)
     |
     +--> [Outbox Publisher] ---> [Queue/Stream (Kafka/SQS)] ---> [PSP Adapter Workers] ---> [PSP(s)]
     |                                  |                              |
     |                                  |                              +--> PSP webhooks/callbacks
     |                                  |
     |                                  +--> [Reconciliation Jobs] <--- PSP reports
     |
     +--> [Ledger DB] (multi-AZ, partition/shard)
     |
  [Read Model / Cache] (Redis) ---> [GET status APIs]
     |
 [Metrics/Logs/Tracing] (Prometheus/Grafana + ELK + OpenTelemetry)
```

##### 3) Core flows (tóm tắt)

- **Create payment (POST /payments)**:
  - Validate + auth.
  - Check **Idempotency-Key** (Redis): nếu đã có result → trả lại đúng response cũ.
  - Create `payment_intent` + `ledger_entry (PENDING)` trong DB (transaction).
  - Ghi event vào **outbox** (same DB transaction) → publisher đẩy sang queue.
  - Trả về `payment_id` + trạng thái `PENDING` (async).
- **Process payment (worker)**:
  - Consume event → gọi PSP adapter tương ứng.
  - Nhận result sync/async → cập nhật ledger (CAPTURED/FAILED) + ghi audit.
  - Emit event cập nhật read-model/cache.
- **Webhook/callback từ PSP**:
  - Verify signature + replay protection.
  - Map về `payment_id` và apply state machine (idempotent).

##### 4) SPOF trong “design đầu tiên” và cách eliminate

- **SPOF #1: 1 instance Payment Service**
  - Fix: stateless service + **N instances** sau LB, autoscaling, multi-AZ.
- **SPOF #2: 1 database single-node**
  - Fix: DB **multi-AZ** + automatic failover; thêm read replicas; partition/shard theo `merchant_id`/`payment_id`.
- **SPOF #3: 1 Redis node (idempotency/cache)**
  - Fix: Redis cluster (primary-replica + multi-AZ) hoặc managed (ElastiCache) + persistence phù hợp.
- **SPOF #4: gọi PSP trực tiếp synchronous trong request path**
  - Fix: async bằng queue/stream; request path chỉ “enqueue + persist”.
- **SPOF #5: 1 queue broker**
  - Fix: managed multi-AZ (Kafka cluster/MSK) hoặc SQS; producer/consumer retry + DLQ.

##### 5) Capacity & instance calculations (đơn giản, có công thức)

Thiết kế theo \(TPS_{design}=20K\).

- **API Gateway/LB tier**
  - Giả sử 1 node handle ~ **5K RPS** (L7 routing + auth offload cơ bản), dùng 50% utilization.
  - Required nodes \(= 20K / (5K * 0.5) = 8\) → **8–10 nodes** (multi-AZ).
- **Payment Orchestrator**
  - Mục tiêu: chủ yếu DB write + Redis read/write + enqueue; không chờ PSP.
  - Giả sử 1 instance handle **1K TPS** ở 50% util.
  - Required \(= 20K / (1K * 0.5)=40\) → **40–50 instances**.
- **PSP Adapter Workers**
  - Phụ thuộc latency PSP; giả sử 1 worker handle **100 TPS** (do I/O + retries), 50% util.
  - Required \(= 20K / (100 * 0.5)=400\) → **400–500 workers** (chia theo PSP + quota).
- **Redis (idempotency + cache)**
  - Ops/sec: create path ~2–3 ops/payment (GET+SET+TTL) → ~60K ops/s ở design peak.
  - Dùng cluster sharding; chọn số shard để giữ CPU < 60% và có replica.
- **Ledger DB**
  - Write: tối thiểu 1 record/payment + index updates → ~20K writes/s (design).
  - Giải pháp:
    - Option A: **NewSQL (CockroachDB/Spanner)** để scale write ngang.
    - Option B: Postgres/MySQL partition + **sharding** theo `merchant_id` (hoặc consistent-hash `payment_id`), mỗi shard multi-AZ.

> Ghi chú: các số “TPS per instance” là giả định để luyện capacity planning; trong thực tế phải benchmark (load test) rồi hiệu chỉnh.

##### 6) Redundancy strategy

- **Application servers**
  - Stateless + autoscaling; deploy **>= 2 AZ**, mỗi AZ giữ tối thiểu 40–50% capacity để chịu AZ failure.
  - Rolling deploy + canary; circuit breaker khi PSP lỗi.
- **Database**
  - Multi-AZ + automatic failover.
  - Read replicas cho query/report; write scaling bằng sharding/NewSQL.
  - PITR backups + immutable audit log (append-only) cho ledger.
- **Queue/Stream**
  - Multi-AZ; DLQ cho poison messages; at-least-once + idempotent consumer.

##### 7) Data model (tối thiểu)

- `payment_intents(payment_id, merchant_id, amount, currency, status, idempotency_key_hash, created_at, updated_at, psp, psp_ref)`
- `ledger_entries(entry_id, payment_id, type, amount, status, created_at)` (append-only)
- `outbox_events(event_id, aggregate_id, type, payload, created_at, published_at)`
- `webhook_events(event_id, psp, psp_event_id, signature_ok, received_at)` (dedupe)

##### 8) Key correctness & security points

- **Idempotency** bắt buộc ở mọi endpoint “money-moving” (create/capture/refund).
- **State machine** rõ ràng (PENDING → CAPTURED/FAILED/REVERSED), reject transitions sai.
- **Exactly-once effect** bằng: outbox + idempotent consumers + unique constraints.
- **Webhook verification**: signature, timestamp, nonce; store dedupe key `psp_event_id`.
- **PII/PCI**: không lưu PAN; tokenize; mã hóa at-rest + in-transit; least privilege IAM.

##### 9) Rough cost estimate (rất thô, để luyện tập)

Ví dụ AWS (chỉ minh họa):
- Compute:
  - Orchestrator: 50 pods/instances (mỗi cái ~2 vCPU/4GB) + workers 500 (nhỏ hơn, autoscale theo queue lag).
- Data:
  - Redis cluster (multi-AZ), DB cluster/shards, Kafka/MSK hoặc SQS.
- Infra:
  - ALB/NLB, NAT, logs/metrics.

**Order-of-magnitude**: vài chục nghìn USD/tháng đến >100K USD/tháng tùy benchmark, PSP latency, retention logs, và cách scale DB/stream.

##### 10) 500-word design decisions (tóm tắt)

Thiết kế chọn mô hình “payment orchestration” tách request path khỏi PSP bằng cơ chế async (queue/stream) để đảm bảo hệ thống chịu được 10K TPS và không bị phụ thuộc latency/availability của PSP. Tính đúng tiền ưu tiên hàng đầu nên dữ liệu ledger được ghi bền vững trong DB theo mô hình append-only và có state machine rõ ràng; mọi thao tác gây side-effect đều bắt buộc idempotent dựa trên Idempotency-Key và/hoặc khóa unique trong DB. Để tránh mất event khi publish, dùng outbox pattern: ghi DB và outbox trong cùng transaction, sau đó publisher mới đẩy sang queue; consumer xử lý at-least-once nhưng bảo toàn “exactly-once effect” nhờ idempotent processing. Về availability 99.9%, loại bỏ SPOF bằng cách triển khai đa AZ cho mọi tier: API gateway/LB, service stateless autoscaling, Redis cluster, queue multi-AZ, DB multi-AZ và chiến lược scale-write bằng sharding hoặc NewSQL. Read-heavy traffic được phục vụ qua cache/read-model để giảm tải DB và giảm latency cho GET status. Security tập trung vào mTLS nội bộ, xác thực/ủy quyền ở gateway, kiểm tra chữ ký webhook, chống replay, và tuân thủ PCI (không lưu PAN, tokenize). Cuối cùng, khả năng vận hành được đảm bảo bởi quan sát (metrics/logs/tracing), circuit breaker và retry/backoff cho PSP, DLQ cho message lỗi, và reconciliation job để đối soát khi PSP trả kết quả trễ hoặc sai lệch.

##### 11) Mark TODOs as completed (khi bạn review xong)

- [x] Thiết kế architecture diagram cho Payment Gateway (10K TPS, 99.9% uptime)
- [x] Vẽ diagram với các components: API Gateway, Payment Service, Database, Cache
- [x] Label mỗi component với estimated QPS capacity
- [x] Identify ít nhất 3 SPOF trong design đầu tiên
- [x] Redesign để eliminate tất cả SPOF
- [x] Tính toán: Cần bao nhiêu instances cho mỗi service? (show calculations)
- [x] Design redundancy strategy cho database
- [x] Design redundancy strategy cho application servers
- [x] Estimate: Total cost (rough) cho infrastructure
- [x] Viết document (500 words) giải thích design decisions

### Design Exercise 2: Betting Platform

- [ ] Thiết kế architecture cho Betting Platform (100K concurrent users)
  **Components:** Load Balancer → API Gateway → Betting Service (stateless, N instances) → Cache (Redis cluster) → Database (Primary + Read Replicas). Message Queue cho settlement async.
  **100K concurrent users:** Giả sử 10% active (place bet/query) → 10K RPS. Mỗi instance 2K RPS → cần ≥5 app instances. DB: write ~2K TPS (place bet), read ~8K QPS (odds, history) → read replicas.
- [ ] Identify bottleneck trong design (CPU, Memory, I/O, Network - chọn 1)
  **Chọn: I/O (Database).** Lý do: Write path (place bet) phải ghi DB + validate odds + update balance → nhiều query/transaction. Read path (odds, live score) rất high QPS. DB dễ thành bottleneck trước CPU/Memory của app.
- [ ] Design solution để resolve bottleneck đó
  **Giải pháp:** (1) Read replicas cho read-heavy (odds, history). (2) Cache layer (Redis) cho odds, hot matches. (3) Write: connection pooling, batch nếu có thể. (4) Partition/shard theo match_id hoặc user_id nếu 1 DB không đủ.
- [ ] Tính toán: Peak traffic = 10x normal, design để handle
  **Normal:** 10K RPS. **Peak:** 100K RPS. **Design:** Auto-scaling app (min 5, max 50 instances). DB: đủ read replicas (10–20) + cache hit rate cao (80%+). Queue cho settlement để tách burst write khỏi real-time bet.
- [ ] Design horizontal scaling strategy
  **Strategy:** Stateless app servers; scale out theo CPU/RPS. Load balancer (L7). Database: thêm read replicas; khi write bottleneck → shard (e.g. by match_id). Cache: Redis cluster. Queue: Kafka/RabbitMQ partition theo match hoặc user.
- [ ] Design vertical scaling strategy (nếu cần)
  **Khi nào dùng:** DB primary có thể scale vertical trước khi shard (nhiều CPU/RAM cho connection và query). Cache node có thể tăng memory. **Giới hạn:** Vertical có ceiling (max instance size) → dài hạn vẫn cần horizontal (replica, shard).
- [ ] So sánh: Horizontal vs Vertical cho use case này (viết 3 điểm)
  1. **Burst traffic (World Cup, big match):** Horizontal phù hợp hơn (thêm app + replica), vertical không đủ nhanh và có limit.  
  2. **Cost:** Vertical đơn giản, rẻ lúc nhỏ; horizontal tốn thêm LB, ops, nhưng scale được xa hơn.  
  3. **Bottleneck DB:** Vertical cho primary có thể kéo dài thời gian trước khi phải shard; horizontal (replica, shard) là hướng tất yếu khi data và QPS lớn.
- [ ] Estimate: Latency cho mỗi component (p50, p95, p99)
  **API Gateway:** p50 2ms, p95 10ms, p99 25ms. **App (Betting Service):** p50 5ms, p95 30ms, p99 80ms. **Cache (Redis):** p50 1ms, p95 3ms, p99 8ms. **DB (read):** p50 5ms, p95 20ms, p99 50ms. **DB (write):** p50 10ms, p95 40ms, p99 100ms. **End-to-end place bet:** p50 ~25ms, p95 ~100ms, p99 ~250ms.
- [ ] Identify: Component nào sẽ là bottleneck? Tại sao?
  **Database (write path).** Lý do: Mỗi bet = transaction (insert bet, update balance, validate odds). Write TPS có giới hạn (disk, lock). Read có thể scale bằng replica + cache; write khó scale hơn → dễ bottleneck nhất.
- [ ] Viết document (500 words) về scaling strategy
  *(Template – điền số liệu thực tế của bạn.)*
  **1. Mục tiêu:** 100K concurrent users, peak 10x normal. **2. App tier:** Stateless, horizontal scaling (min/max instances), metric: CPU hoặc RPS. **3. Data tier:** Primary + N read replicas; cache (Redis) cho odds và hot data; queue cho settlement. **4. Bottleneck:** DB write → giảm write path (batch, async settlement), tăng read path (replica, cache). **5. Peak:** Auto-scale + cache warming + đủ replica; test load 100K RPS. **6. Trade-off:** Consistency (read-your-writes) vs scale (replica lag) – chọn strategy rõ ràng (e.g. sticky session cho critical read sau write).

### Design Exercise 3: Load Estimation

- [ ] Scenario: E-commerce site, 1M daily active users
- [ ] Estimate: Peak QPS (assume 20% traffic trong 1 hour)
  **1M DAU, 20% trong 1h → 200K users trong 1h.** Giả sử mỗi user 5 requests trong giờ peak → 200K × 5 = 1M requests / 3600s ≈ **278 QPS**. Nếu peak hơn (e.g. 30% trong 30 phút): 300K × 5 / 1800 ≈ **833 QPS**. *Ghi lại assumption của bạn.*
- [ ] Estimate: Average request size (assume 5KB)
  **5KB** (header + body). Có thể điều chỉnh theo API thực tế (e.g. upload lớn hơn).
- [ ] Estimate: Average response size (assume 10KB)
  **10KB** (JSON product list, HTML snippet). Có thể tách API nhẹ (metadata) vs nặng (full page).
- [ ] Calculate: Total bandwidth requirement (Mbps)
  **Ingress:** 278 × 5KB ≈ 1.39 MB/s ≈ **11.1 Mbps**. **Egress:** 278 × 10KB ≈ 2.78 MB/s ≈ **22.2 Mbps**. Peak 833 QPS → ~33 Mbps ingress, ~67 Mbps egress. *Làm tròn theo nhà cung cấp (e.g. 50 Mbps, 100 Mbps).*
- [ ] Estimate: Database size (assume 1GB per 10K users)
  **1M users → 1GB × (1M/10K) = 100GB** (chỉ user/product metadata). Thêm orders, logs → có thể 300–500GB. *Ghi lại schema và growth rate.*
- [ ] Calculate: Storage growth rate (per month)
  Ví dụ: 100GB base + 10GB/tháng (orders, logs) → tháng 1: 110GB, tháng 12: 210GB. *Điền số theo dự đoán business.*
- [ ] Estimate: Cache size needed (assume 20% of DB size)
  **20% × 100GB = 20GB** cache (hot products, sessions). Redis 20GB + overhead ~25GB. *Có thể tăng % nếu read-heavy.*
- [ ] Create spreadsheet với tất cả calculations
  Cột: Metric, Formula, Value, Unit. Dòng: Peak QPS, Request size, Response size, Ingress Mbps, Egress Mbps, DB size, Growth/month, Cache size.
- [ ] Verify: Tất cả numbers có hợp lý không? (write validation)
  **Validation:** (1) QPS so với 1M DAU – 278–833 QPS hợp lý cho e-commerce. (2) Bandwidth – vài chục Mbps hợp lý cho 1 server/ vài server. (3) DB 100GB cho 1M users – hợp lý nếu không lưu blob lớn. (4) Cache 20% – có thể đo hit rate sau và tinh chỉnh.

---

## Coding TODOs

### Task 1: Spring Boot Performance App

- [ ] Tạo Spring Boot project mới  
  `spring init --dependencies=web,actuator week1-perf-app`
- [ ] Tạo REST API endpoint: `GET /api/users/{id}` (return mock user data)  
  Return `User(id, name, email)` từ Map hoặc list in-memory.
- [ ] Tạo REST API endpoint: `POST /api/users` (create user, save to in-memory list)  
  Request body → validate → put vào `ConcurrentHashMap` hoặc list.
- [ ] Add logging: Log request time cho mỗi endpoint  
  Filter/Interceptor: `long start = now(); ... log("duration_ms", now()-start)`.
- [ ] Add metrics: Count requests, measure latency  
  Micrometer: `Counter.builder("requests.total").register(registry); Timer` cho duration.
- [ ] Run app và test với 100 requests (manual hoặc script)  
  curl loop hoặc script (bash/Python) gọi GET/POST, ghi thời gian.
- [ ] Measure: Average latency, p95 latency  
  Từ log hoặc từ `Timer` metrics (percentile).
- [ ] Document: Performance baseline  
  Ví dụ: GET p50=12ms, p95=45ms; POST p50=8ms, p95=30ms; QPS ~X với 1 thread.

### Task 2: Load Testing Setup

- [ ] Install JMeter hoặc Gatling  
  JMeter: download; Gatling: sbt hoặc Maven.
- [ ] Tạo test plan cho `/api/users/{id}` endpoint  
  Thread group: N users, R ramp-up, loop M lần.
- [ ] Configure: 100 concurrent users, 1000 total requests  
  100 threads, 10 iterations hoặc 1000/100.
- [ ] Run load test  
  Chạy và đợi kết thúc, không có error nghiêm trọng.
- [ ] Export results: QPS, latency (p50, p95, p99), error rate  
  JMeter: Summary Report, Aggregate Report; Gatling: report HTML.
- [ ] Identify: At what load does latency spike?  
  Tăng dần threads (50, 100, 200, 500) → ghi lại khi p95 tăng vọt (e.g. > 2x).
- [ ] Identify: At what load do errors start?  
  Ghi lại first N (threads hoặc RPS) khi có 4xx/5xx hoặc timeouts.
- [ ] Document: Performance limits của app hiện tại  
  Ví dụ: Safe < 200 concurrent; latency spike at 300; errors from 500.
- [ ] Tạo report với charts (response time, throughput)  
  JMeter: Graphs; Gatling: mặc định có charts.

### Task 3: Performance Profiling

- [ ] Install VisualVM hoặc JProfiler  
  Download, cài đặt, biết cách attach vào JVM (PID hoặc remote).
- [ ] Attach profiler to Spring Boot app  
  Start app với JMX; VisualVM: connect → Sampler hoặc Profiler.
- [ ] Run load test while profiling  
  Vừa chạy JMeter/Gatling vừa profile (CPU, Memory).
- [ ] Identify: Top 5 methods by CPU time  
  Tab CPU: sort by self time hoặc total time, ghi tên method.
- [ ] Identify: Memory allocation hotspots  
  Tab Allocations hoặc Heap; xem class nào allocate nhiều.
- [ ] Check: Memory leaks (heap growth over time)  
  Heap dump 2–3 lần (cách nhau vài phút load) → so sánh; xem object nào tăng.
- [ ] Check: Thread contention  
  Tab Threads / Monitors; xem thread nào blocked nhiều.
- [ ] Document: 3 performance issues found  
  Ví dụ: (1) Method X chiếm 40% CPU, (2) Class Y allocate nhiều, (3) Lock Z contention.
- [ ] Propose: 3 optimizations (không cần implement, chỉ propose)  
  Ví dụ: cache kết quả X, giảm tạo object Y, thu hẹp lock scope Z.

### Task 4: Simple Load Balancer

- [ ] Tạo Java class: `SimpleLoadBalancer`  
  Field: `List<Backend> servers`, `AtomicInteger index` (round-robin).
- [ ] Implement: Round-robin algorithm  
  `getNextServer(): servers.get(index.getAndIncrement() % servers.size())`.
- [ ] Add: List of backend servers (hardcoded URLs)  
  Constructor nhận `List<String>` URLs.
- [ ] Add: `getNextServer()` method  
  Return URL (hoặc Backend object) theo round-robin.
- [ ] Add: Health check mechanism (ping endpoint)  
  Scheduled task: HTTP GET /health mỗi N giây; đánh dấu unhealthy nếu fail.
- [ ] Add: Skip unhealthy servers  
  Trong `getNextServer()` chỉ chọn server có `healthy == true`.
- [ ] Test: With 3 mock backend servers  
  Ứng dụng hoặc mock server (e.g. WireMock) trên 3 port.
- [ ] Test: Mark one server unhealthy, verify it's skipped  
  Tắt 1 server hoặc trả 5xx → gọi getNextServer nhiều lần, không trả server đó.
- [ ] Measure: Overhead của load balancer (latency added)  
  So sánh latency direct call vs qua LB (nên < 1–2ms).
- [ ] Document: Code và test results  
  Mô tả class, thuật toán, kết quả test và overhead.

### Task 5: Health Check Endpoints

- [ ] Add endpoint: `GET /health` (basic health check)  
  Spring Boot Actuator: `management.endpoint.health.probes.enabled=true` hoặc custom controller trả 200.
- [ ] Add endpoint: `GET /health/readiness` (readiness probe)  
  Actuator: `readiness`; hoặc custom: check DB connection, nếu fail trả 503.
- [ ] Add endpoint: `GET /health/liveness` (liveness probe)  
  Actuator: `liveness`; hoặc custom: process còn sống (có thể luôn 200).
- [ ] Implement: Database connection check trong readiness  
  `DataSource.getConnection()` hoặc query đơn giản (e.g. SELECT 1); fail → 503.
- [ ] Implement: Memory check trong liveness (fail if memory > 90%)  
  `Runtime.getRuntime().maxMemory()` và `totalMemory() - freeMemory()`; nếu used ratio > 0.9 → 503.
- [ ] Test: All health endpoints return correct status  
  Gọi từng endpoint khi app OK → 200; khi DB down → readiness 503.
- [ ] Test: Simulate DB down, verify readiness fails  
  Tắt DB hoặc sai connection string → readiness 503, liveness vẫn 200.
- [ ] Test: Simulate high memory, verify liveness fails  
  (Khó trên local: có thể mock hoặc giảm ngưỡng 90% xuống để test.)
- [ ] Document: Khi nào dùng readiness vs liveness  
  Readiness: có nhận traffic không (DB OK, cache OK). Liveness: process còn chạy không (restart nếu die).

### Task 6: Metrics Collection

- [ ] Add Micrometer dependency  
  `spring-boot-starter-actuator` + `micrometer-registry-prometheus` (nếu export Prometheus).
- [ ] Expose metrics endpoint: `GET /actuator/metrics`  
  Bật `management.endpoints.web.exposure.include=health,metrics,prometheus`.
- [ ] Add custom metric: `requests.total` (counter)  
  `Counter.builder("requests.total").tag("uri", uri).register(registry).increment()`.
- [ ] Add custom metric: `request.duration` (timer)  
  `Timer.builder("request.duration").register(registry).record(duration)`.
- [ ] Instrument: All API endpoints với metrics  
  Filter/Interceptor: tăng counter, record timer cho mỗi request.
- [ ] Verify: Metrics được update correctly  
  Gọi API vài lần → mở `/actuator/metrics/requests.total` và `request.duration` xem số tăng.
- [ ] Export: Metrics to Prometheus format (optional)  
  `GET /actuator/prometheus` trả text format.
- [ ] Document: Metrics available và ý nghĩa  
  Liệt kê: requests_total, request_duration_seconds, jvm_memory_used, etc. và cách đọc.

---

## Analysis TODOs

### Analysis Task 1: Bottleneck Analysis

- [ ] Chọn một Spring Boot app hiện tại (hoặc tạo simple one)  
  Dùng app từ Task 1 (users API) hoặc app có sẵn.
- [ ] Run load test với increasing load: 10, 50, 100, 500, 1000 concurrent users  
  JMeter/Gatling: 5 test runs với N = 10, 50, 100, 500, 1000.
- [ ] Measure: Latency, throughput, error rate cho mỗi load level  
  Ghi bảng: N | p50 | p95 | p99 | QPS | error %.
- [ ] Plot graph: Latency vs Load  
  Trục X: concurrent users, Y: p95 latency (ms).
- [ ] Plot graph: Throughput vs Load  
  Trục X: concurrent users, Y: QPS.
- [ ] Identify: Breaking point (khi latency spikes)  
  Ví dụ: p95 tăng gấp đôi khi N > 200 → breaking point ~200.
- [ ] Identify: Type of bottleneck (CPU-bound, I/O-bound, Memory-bound)  
  CPU: CPU % cao; I/O: DB/disk wait; Memory: heap/GC. Dùng profiler hoặc metrics OS.
- [ ] Analyze: Tại sao bottleneck xảy ra ở điểm đó?  
  Ví dụ: 200 threads → connection pool DB 20 → queue → latency tăng.
- [ ] Propose: 3 solutions để resolve bottleneck  
  Ví dụ: (1) Tăng connection pool, (2) Thêm cache, (3) Scale out app.
- [ ] Estimate: Improvement expected từ mỗi solution  
  Ví dụ: cache → giảm 60% DB load; scale out → tăng 3x throughput.

### Analysis Task 2: Capacity Planning

- [ ] Scenario: Design system cho 10M users  
- [ ] Estimate: Peak concurrent users (assume 10% of total)  
  **10% × 10M = 1M concurrent.** (Có thể giả sử 1–5% tùy loại app.)
- [ ] Estimate: Peak QPS (assume 5 requests/user/minute)  
  **1M × 5 / 60 ≈ 83,333 QPS.** (Điều chỉnh theo use case.)
- [ ] Calculate: Database size (assume 1KB per user)  
  **10M × 1KB = 10GB** (chỉ user data). Thêm bảng khác → nhân thêm.
- [ ] Calculate: Cache size (assume 10% of DB)  
  **10% × 10GB = 1GB** cache. Redis 1GB + overhead.
- [ ] Calculate: Bandwidth (assume 5KB per request)  
  **83,333 × 5KB × 2 (req+resp) ≈ 833 MB/s ≈ 6.7 Gbps** (peak). Có thể giảm nếu cache/CDN.
- [ ] Estimate: Server count (assume 1 server = 10K QPS)  
  **83,333 / 10,000 ≈ 9** app servers (tối thiểu). Thêm headroom → 15–20.
- [ ] Estimate: Cost (rough, assume $100/server/month)  
  **20 × $100 = $2,000/month** (chỉ app). Thêm DB, cache, LB → ước lượng tổng.
- [ ] Create: Capacity planning spreadsheet  
  Cột: Metric, Formula, Value. Dòng: Users, Concurrent, QPS, DB size, Cache, Bandwidth, Servers, Cost.
- [ ] Validate: Tất cả assumptions có realistic không?  
  So sánh với benchmark thực tế hoặc case study (e.g. 10M user app dùng ~X server).

### Analysis Task 3: Availability Calculation

- [ ] Calculate: Downtime budget cho 99.9% availability (per year)  
  **99.9% → 0.1% downtime → 365×24×60×0.001 = 525.6 phút/năm ≈ 8.76 giờ/năm.**
- [ ] Calculate: Downtime budget cho 99.99% availability (per year)  
  **99.99% → 0.01% → 52.56 phút/năm ≈ 52 phút/năm.**
- [ ] Scenario: System có 5 components, mỗi component có 99.9% availability  
  **Series: A = 0.999^5 ≈ 0.995 (99.5%).** Downtime ≈ 43.8 giờ/năm.
- [ ] Calculate: Overall system availability (series)  
  **A_total = A1 × A2 × A3 × A4 × A5.**
- [ ] Scenario: System có 2 redundant components (parallel), mỗi 99.9%  
  **Parallel: A = 1 - (1-0.999)^2 = 1 - 0.000001 = 99.9999%.**
- [ ] Calculate: Overall system availability (parallel)  
  **A = 1 - (1-A1)(1-A2).**
- [ ] Analyze: Cần bao nhiêu nines để có < 1 hour downtime/year?  
  **1 hour = 60 min → 60/(365×24×60) ≈ 0.0114% downtime → 99.9886% → cần ~4 nines (99.99%).**
- [ ] Analyze: Nếu MTTR = 1 hour, cần MTBF = ? để đạt 99.99%?  
  **A = MTBF/(MTBF+MTTR) = 0.9999 → MTBF = 0.9999×MTTR/0.0001 = 9999×1h ≈ 416 ngày.**
- [ ] Create: Availability calculator spreadsheet  
  Cột: Component, Availability, (Series/Parallel), Overall. Dòng: từng component.
- [ ] Document: Findings và insights  
  Ví dụ: 5 component series → availability giảm mạnh; redundancy cải thiện rõ.

### Analysis Task 4: Scaling Strategy Comparison

- [ ] Scenario: Payment API, current load = 1K QPS, expected = 10K QPS  
- [ ] Option 1: Vertical scaling (bigger server)  
  Tăng từ 4 CPU → 16 CPU, 8GB → 32GB.
- [ ] Calculate: Cost của vertical scaling  
  Ví dụ: server lớn gấp 2–3 lần giá → $300/tháng thay vì $100.
- [ ] Calculate: Limitations (max server size)  
  Max instance (e.g. 64 vCPU) → không scale mãi; single point of failure.
- [ ] Option 2: Horizontal scaling (more servers)  
  10 server nhỏ (mỗi 1K QPS).
- [ ] Calculate: Cost của horizontal scaling  
  10 × $100 = $1,000/tháng + LB, ops phức tạp hơn.
- [ ] Calculate: Complexity added  
  Load balancer, stateless app, monitoring, deployment.
- [ ] Compare: Vertical vs Horizontal (cost, complexity, limits)  
  Bảng: Cost | Complexity | Limit | Fault tolerance. Vertical: rẻ hơn lúc nhỏ, đơn giản, có limit, SPOF. Horizontal: scale xa, phức tạp, tốn hơn.
- [ ] Recommend: Which strategy? Tại sao?  
  Ví dụ: 10K QPS → horizontal vì cần HA và scale tiếp sau này; vertical nếu budget rất hạn chế và 10K là ceiling.
- [ ] Document: Decision matrix  
  Ghi lại bảng so sánh và recommendation.

### Analysis Task 5: Performance Baseline

- [ ] Measure: Current app performance (baseline)  
  Chạy load test 100–500 users, ghi QPS, p50/p95/p99, error %.
- [ ] Metrics: QPS, latency (p50, p95, p99), error rate  
  Ví dụ: QPS 800, p50 40ms, p95 120ms, p99 250ms, error 0%.
- [ ] Document: Baseline performance  
  Ở mục nào (hardware, JVM, code version), số liệu trên.
- [ ] Set: Performance goals (improve by 2x)  
  Mục tiêu: QPS 1600, p95 60ms (giảm 2x).
- [ ] Identify: Bottleneck preventing goal achievement  
  Từ profiling: DB query chậm hoặc CPU 100%.
- [ ] Propose: Optimization plan  
  (1) Thêm index / optimize query, (2) Cache hot data, (3) Tăng connection pool.
- [ ] Estimate: Expected improvement từ mỗi optimization  
  Ví dụ: cache → -40% DB load; index → -50% query time.
- [ ] Prioritize: Optimizations by impact/effort  
  Impact cao, effort thấp trước (e.g. index → cache → refactor).
- [ ] Create: Performance improvement roadmap  
  Tuần 1: index; Tuần 2: cache; Tuần 3: tuning pool.
- [ ] Document: Analysis và recommendations  
  Baseline, goal, bottleneck, plan, ước lượng kết quả.

---

## Review TODOs

### Self-Evaluation

- [ ] Review: Tất cả study TODOs đã hoàn thành chưa?  
  Đi từng mục Study TODOs, đánh dấu đã đọc/đã làm/đã ghi chú.
- [ ] Review: Tất cả design exercises đã làm chưa?  
  Payment Gateway, Betting Platform, Load Estimation – đã có diagram và document chưa.
- [ ] Review: Tất cả coding tasks đã code và test chưa?  
  Task 1–6: project tồn tại, chạy được, có test/load test/profiling.
- [ ] Review: Tất cả analysis tasks đã complete chưa?  
  Bottleneck, Capacity, Availability, Scaling comparison, Performance baseline – có số liệu và document.
- [ ] Rate yourself: Understanding của scalability (1-10)  
  Ghi số và 1–2 câu giải thích (ví dụ: 7 – hiểu vertical/horizontal, chưa sâu Amdahl/Gustafson).
- [ ] Rate yourself: Understanding của availability (1-10)  
  Ghi số và giải thích ngắn.
- [ ] Rate yourself: Practical skills (load testing, profiling) (1-10)  
  Ghi số và giải thích (đã dùng JMeter/Gatling, VisualVM chưa).
- [ ] Identify: 3 concepts bạn hiểu rõ nhất  
  Ví dụ: Horizontal scaling, Availability %, Load testing flow.
- [ ] Identify: 3 concepts bạn còn confuse  
  Ví dụ: Amdahl vs Gustafson khi nào dùng, p99 trong thực tế đo thế nào.
- [ ] Plan: Làm sao để clarify 3 concepts còn confuse?  
  Ví dụ: Đọc lại chương X, làm thêm bài tập Y, hỏi mentor.

### Design Review

- [ ] Review Payment Gateway design  
  Xem lại diagram và doc Payment Gateway (10K TPS, 99.9%).
- [ ] Check: Có SPOF không?  
  Single DB? Single LB? Single region? Nếu có → ghi và đề xuất redundancy.
- [ ] Check: Scaling strategy có realistic không?  
  Số instance, QPS/instance, DB capacity có khớp với 10K TPS không.
- [ ] Check: Capacity estimates có hợp lý không?  
  So với benchmark hoặc case study tương tự.
- [ ] Identify: 3 weaknesses trong design  
  Ví dụ: chưa multi-region, cache chưa rõ invalidation, chưa có queue cho peak.
- [ ] Propose: Improvements cho 3 weaknesses  
  Mỗi weakness → 1–2 câu cải thiện cụ thể.
- [ ] Compare: Design của bạn vs best practices  
  So với tài liệu (e.g. AWS Well-Architected, SRE book) – thiếu gì, thừa gì.
- [ ] Document: Lessons learned  
  Đoạn ngắn: 3–5 điều rút ra từ design và review.

### Code Review

- [ ] Review: Code quality (clean code principles)  
  Đặt tên, hàm ngắn, ít dependency, dễ test.
- [ ] Review: Error handling  
  Exception, retry, logging lỗi, không nuốt lỗi.
- [ ] Review: Logging và monitoring  
  Có log request/error, có metrics (counter, timer).
- [ ] Review: Performance optimizations  
  Connection pool, cache (nếu có), N+1 query (nếu có DB).
- [ ] Identify: 3 code improvements needed  
  Ví dụ: tách service, thêm validation, chuẩn hóa error response.
- [ ] Refactor: At least 1 piece of code  
  Chọn 1 improvement và refactor (commit + message rõ ràng).
- [ ] Document: Code review findings  
  File README hoặc doc: 3 findings + 1 refactor đã làm.

### Performance Review

- [ ] Review: Load test results  
  Bảng/graph QPS, latency, error theo load.
- [ ] Review: Profiling results  
  Top CPU, memory, contention (nếu có).
- [ ] Identify: Top 3 performance issues  
  Ví dụ: DB query chậm, thiếu cache, connection pool nhỏ.
- [ ] Verify: Performance goals đã đạt chưa?  
  So với baseline và target (e.g. 2x QPS, p95 giảm 50%).
- [ ] Document: Performance analysis  
  Mô tả hiện trạng, vấn đề, đã làm gì, kết quả.
- [ ] Create: Performance improvement plan  
  Danh sách việc tiếp theo (optimize query, thêm cache, …) với ưu tiên.

### Knowledge Check

- [ ] Explain: Vertical vs Horizontal scaling (viết 1 paragraph, không xem notes)  
  **Template:** Vertical = tăng tài nguyên 1 máy (CPU, RAM); horizontal = thêm máy. Vertical đơn giản, có giới hạn; horizontal scale xa hơn, cần LB, stateless, data tier scale. Chọn theo cost, limit, HA.
- [ ] Explain: Availability calculation (viết công thức và example)  
  **Công thức:** A = MTBF/(MTBF+MTTR) hoặc Downtime = (1-A)×8760 giờ/năm. **Ví dụ:** 99.9% → 8.76h downtime/năm.
- [ ] Explain: Bottleneck identification process (viết 5 steps)  
  **(1)** Đo latency/throughput theo load. **(2)** Tìm điểm latency tăng vọt hoặc throughput plateau. **(3)** Thu thập metrics (CPU, memory, I/O, DB). **(4)** Profiler để xem method/query nào tốn thời gian. **(5)** Kết luận bottleneck (CPU/I/O/Memory/Network) và đề xuất fix.
- [ ] Explain: Capacity planning approach (viết 5 steps)  
  **(1)** Ước lượng users/DAU và peak %. **(2)** Tính QPS (requests/user/time). **(3)** Tính storage (data/user × users), bandwidth (QPS × size). **(4)** Tính số server (QPS/server), DB size, cache size. **(5)** Validate với benchmark hoặc case study, ghi assumptions.
- [ ] Solve: System có 3 components (99%, 99.9%, 99.99%), tính overall availability  
  **Series:** A = 0.99 × 0.999 × 0.9999 ≈ **0.9899 (98.99%).**
- [ ] Solve: Estimate QPS cho 1M users, 10% online, 2 requests/user/minute  
  **1M × 10% = 100K online.** 100K × 2 / 60 ≈ **3,333 QPS.** (Peak có thể ×2–3.)
- [ ] Verify: Answers của bạn có đúng không?  
  So lại công thức và số với tài liệu hoặc calculator.
- [ ] Document: Knowledge gaps found  
  Ghi lại câu nào chưa trả lời chắc, cần ôn thêm.

### Reflection

- [ ] Write: 3 điều học được quan trọng nhất tuần này  
  Ví dụ: (1) Amdahl/Gustafson và bottleneck, (2) Availability % và downtime, (3) Load test + profiling flow.
- [ ] Write: 2 điều còn confuse hoặc cần học thêm  
  Ví dụ: p99 trong production đo thế nào, khi nào chọn vertical vs horizontal cho từng layer.
- [ ] Write: 1 mistake bạn đã làm và lesson learned  
  Ví dụ: ước lượng QPS quá thấp → điều chỉnh lại formula.
- [ ] Write: Confidence level cho Week 2 (1-10)  
  Ghi số và 1 câu (ví dụ: 7 – sẵn sàng HA, circuit breaker).
- [ ] Plan: Preparation cho Week 2 (Availability & Reliability)  
  Đọc trước tài liệu Week 2, xem lại circuit breaker, health check.
- [ ] Set: Goals cho Week 2  
  Ví dụ: Implement circuit breaker, thiết kế HA cho 1 service.
- [ ] Document: Week 1 reflection (500 words)  
  Tổng hợp 3 điều học được, 2 confuse, 1 mistake, confidence, plan Week 2.

### Mentor Questions (Answer these)

- [ ] Q1: Nếu bạn phải scale từ 1K QPS lên 100K QPS, bạn sẽ làm gì? (viết 5 steps)  
  **Template:** (1) Đo bottleneck hiện tại (DB, CPU, network). (2) Scale read: cache + read replica. (3) Scale app: horizontal, stateless, LB. (4) Scale write: shard DB hoặc queue. (5) Load test 100K, monitor, tune.
- [ ] Q2: System có 99.9% availability nhưng vẫn bị complain về downtime. Tại sao? (viết analysis)  
  **Gợi ý:** Downtime tập trung (1 lần 30 phút) vs rải rác; user nhạy cảm giờ peak; SLA đo sai (region/endpoint); perceived downtime (latency cao = “chậm”); dependency bên ngoài không nằm trong 99.9%.
- [ ] Q3: Làm sao bạn identify bottleneck trong production system? (viết process)  
  **Template:** (1) Metrics (CPU, memory, disk, network, DB connections). (2) APM/tracing (slow request, slow query). (3) Log (error, timeout). (4) So sánh với load (traffic tăng đúng lúc latency tăng?). (5) Reproduce trong staging + profiler.
- [ ] Q4: Vertical scaling có giới hạn không? Giới hạn là gì? (viết answer)  
  **Có.** Giới hạn: max CPU/RAM của 1 instance (e.g. 64 vCPU, 256GB); giá tăng phi tuyến; single point of failure; OS/scheduler overhead khi quá lớn.
- [ ] Q5: Tại sao p99 latency quan trọng hơn average latency? (viết explanation)  
  Average bị kéo bởi nhiều request nhanh; p99 phản ánh trải nghiệm user tệ nhất (1% request chậm). SLA và user satisfaction thường gắn với tail latency (p95, p99).
- [ ] Review: Answers của bạn có đủ depth chưa?  
  Đọc lại 5 câu trả lời: có số, có ví dụ, có process rõ ràng chưa.
- [ ] Improve: Answers nếu cần  
  Bổ sung 1–2 câu hoặc 1 ví dụ cho câu còn ngắn.

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete  
  Đánh dấu khi đã đọc/ghi chú đủ các mục Study.
- [ ] Tất cả Design TODOs: ✅ Complete  
  Payment Gateway, Betting Platform, Load Estimation đã có design + document.
- [ ] Tất cả Coding TODOs: ✅ Complete và tested  
  Task 1–6 đã code, chạy, test/load test/profiling, có doc.
- [ ] Tất cả Analysis TODOs: ✅ Complete với documentation  
  Bottleneck, Capacity, Availability, Scaling, Performance baseline đã có số và doc.
- [ ] Tất cả Review TODOs: ✅ Complete  
  Self-eval, Design review, Code review, Performance review, Knowledge check, Reflection, Mentor questions đã làm và ghi lại.
- [ ] Reflection document: ✅ Written  
  Có file hoặc section Week 1 reflection ~500 từ.
- [ ] Code committed to repo: ✅ Yes  
  Code Week 1 đã push (GitHub/GitLab/…).
- [ ] Design diagrams saved: ✅ Yes  
  Diagram Payment Gateway, Betting (và Load Estimation nếu có) đã lưu (draw.io, image, …).
- [ ] Ready for Week 2: ✅ Yes  
  Chỉ đánh dấu khi thật sự hoàn thành các mục trên và cảm thấy sẵn sàng.

---

> **Mentor Final Check**: Nếu bạn check tất cả items trên, bạn đã sẵn sàng cho Week 2. Nếu không, bạn chưa sẵn sàng. Hãy honest với bản thân.
