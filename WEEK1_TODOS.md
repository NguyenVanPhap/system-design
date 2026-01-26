# Week 1 – Fundamentals: Scalability & Availability

> **Mentor Note**: Đây là TODO list nghiêm khắc. Mỗi item phải được hoàn thành 100%. Không có "gần như xong" hay "hiểu đại khái". Bạn phải CODE, MEASURE, và DOCUMENT.

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

| Loại Serial (System Design)   | Ví dụ                                | Vì sao làm chậm             | Định luật áp dụng | Giải thích |
| ----------------------------- | ------------------------------------ | --------------------------- | ----------------- | ---------- |
| 🗄️ **Database bottleneck**   | Transaction dài, `SELECT FOR UPDATE` | DB xử lý tuần tự            | **Amdahl** | DB bottleneck > 10% query → phải fix (replica, shard) |
| 🔌 **Chung tài nguyên**       | 1 file, 1 connection, 1 queue        | Phải đợi nhau               | **Amdahl** | Shared resource > 10% contention → phải fix (shard, separate) |
| 📐 **Logic bắt buộc tuần tự** | step1 → step2 → step3                | Không tách được             | **Amdahl** | Serial logic > 10% time → phải optimize |
| 🚀 **Init / Startup**         | Load config, warmup                  | Chạy 1 luồng                | **Gustafson** | Init < 1% total time → không đáng kể, có nhiều task khác |

Scalability = Throughput (RPS, job/s) tăng theo server.

Amdahl nói:

❌ Thêm server ≠ tăng vô hạn
✅ Phải giảm cổ chai trước

5️⃣ Serial trong hệ thống thường là 

| Bottleneck            | Nghĩa là gì                   | Serial ở đâu        | Dấu hiệu thường gặp                      | Scale app có giúp không? | Định luật áp dụng | Cách xử lý chính                 |
| --------------------- | ----------------------------- | ------------------- | ---------------------------------------- | ------------------------ | ----------------- | -------------------------------- |
| **Database**          | DB xử lý quá nhiều read/write | 1 DB node / primary | Query chậm, CPU DB 100%, connection full | ❌ Không                  | **Amdahl** (> 10% query) | Read replica, sharding, cache    |
| **Hot Key Redis**     | Nhiều request đập vào 1 key   | Redis single thread | Redis latency cao, key bị spam           | ❌ Không                  | **Amdahl** (> 10% traffic) | Shard key, cache local, cluster  |
| **Single Leader**     | Chỉ 1 node được write/xử lý   | Leader node         | Write TPS thấp, leader overload          | ❌ Không                  | **Amdahl** (> 10% write) | Shard, multi-leader, partition   |
| **Lock Global**       | 1 lock khóa toàn hệ thống     | Critical section    | Thread waiting nhiều, TPS thấp           | ❌ Không                  | **Amdahl** (> 10% request) | Fine-grained lock, optimistic    |
| **Queue 1 Partition** | Queue chỉ có 1 partition      | 1 consumer          | Lag cao, consumer idle                   | ❌ Không                  | **Amdahl** (> 10% message) | Tăng partition, parallel consume |
| **File dùng chung**   | Nhiều node dùng chung file    | File lock / IO      | IO wait cao, ghi file chậm               | ❌ Không                  | **Amdahl** (> 10% I/O) | File riêng, object storage       |

**Lưu ý**: Tất cả các bottleneck trên đều áp dụng **Amdahl's Law** khi chiếm > 10% workload. Nếu < 1% workload và có nhiều partition/key/task khác → có thể áp dụng **Gustafson's Law** (không cần fix).


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
      
| Tiêu chí | **Amdahl's Law** | **Gustafson's Law** |
|----------|------------------|---------------------|
| **Giả định** | Kích thước vấn đề **cố định** | Kích thước vấn đề **có thể tăng** |
| **Công thức** | `Speedup = 1 / (S + (1-S)/N)` | `Speedup = S + (1-S) × N` |
| **Speedup tối đa** | Bị giới hạn bởi phần serial (hữu hạn) | Gần tuyến tính với N (có thể rất lớn) |
| **Ví dụ (S=0.1, N=100)** | **9.17x** | **90.1x** |
| **Quan điểm** | Phần serial là bottleneck nghiêm trọng | Phần serial không cản trở nhiều nếu tăng quy mô |
| **Ứng dụng thực tế** | Fixed workload, hệ thống hiện tại | Scalable workload, big data, distributed computing |
| **Bài học** | Phải giảm phần serial trước khi scale | Có thể scale tốt nếu workload tăng theo tài nguyên |
      
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
      
| Tình huống | Dùng định luật nào | Lý do |
|------------|---------------------|-------|
| API server xử lý request cố định | **Amdahl** | Số request/giây cố định, không tăng theo số server |
| Database query optimization | **Amdahl** | Query cố định, chỉ muốn chạy nhanh hơn |
| Big data processing (Hadoop, Spark) | **Gustafson** | Nhiều node hơn → xử lý nhiều data hơn |
| Video rendering, image processing | **Gustafson** | Nhiều CPU hơn → render nhiều frame hơn |
| Web scraping với rate limit | **Amdahl** | Rate limit cố định, không tăng theo số worker |
| Distributed training ML | **Gustafson** | Nhiều GPU hơn → train với dataset lớn hơn |
      
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
      
| Loại Serial (Multithreading) | Ví dụ                                | Vì sao làm chậm             | Định luật áp dụng | Giải thích |
| ----------------------------- | ------------------------------------ | --------------------------- | ----------------- | ---------- |
| 🔒 **Lock / synchronized**    | `synchronized`, `ReentrantLock`      | Chỉ 1 thread vào → xếp hàng | **Amdahl** | Lock block > 10% thread → phải fix (fine-grained) |
| 💾 **IO blocking**            | Đọc file, gọi API, upload            | Thread đứng chờ             | **Amdahl** | IO blocking > 10% time → phải fix (async, non-blocking) |
| 🧹 **GC Pause (Java)**        | Stop-the-world GC                    | Tất cả đứng im              | **Amdahl** | GC pause > 10% time → phải fix (tune GC, reduce allocation) |
| 📝 **Logging đồng bộ**        | File log sync                        | Block thread                | **Amdahl** | Sync logging > 10% time → phải fix (async logging) |
| 🔁 **Single-thread executor** | `newSingleThreadExecutor()`          | Ép về 1 luồng               | **Gustafson** | Nếu có nhiều executor khác → không đáng kể |
      
      **Decision Matrix - Multithreading**
      
| Tình huống | Câu hỏi | Amdahl (Phải fix) | Gustafson (Có thể bỏ qua) |
|------------|---------|-------------------|---------------------------|
| **Thread Pool Size** | Serial task chiếm bao nhiêu % thời gian? | > 10% → Giảm serial (lock, sync) | < 1% → Không cần fix, có nhiều task khác |
| **Lock Contention** | Lock này block bao nhiêu % thread? | > 10% thread → Fine-grained lock | < 1% thread → Không cần fix |
| **Critical Section** | Critical section chiếm bao nhiêu % thời gian? | > 10% → Optimize, giảm thời gian | < 1% → Không cần fix |
| **Task Distribution** | 1 task lớn vs nhiều task nhỏ? | 1 task lớn → Chia nhỏ (Amdahl) | Nhiều task nhỏ → Thread pool (Gustafson) |
| **Context Switch** | Context switch overhead? | > 10% → Giảm số thread | < 1% → Không cần fix |
      
      **Ví dụ Multithreading:**
      
| Tình huống | Dùng định luật nào | Giải thích |
|------------|---------------------|------------|
| Sort 1 mảng lớn với nhiều threads | **Amdahl** | Task cố định (1 mảng), phần merge là serial bottleneck |
| Xử lý 1000 request với thread pool | **Gustafson** | Nhiều request độc lập, mỗi thread xử lý 1 request |
| Parallel for loop xử lý array | **Amdahl** | Array cố định, chia nhỏ và xử lý, nhưng có overhead |
| Producer-Consumer với nhiều workers | **Gustafson** | Nhiều item trong queue, mỗi worker xử lý item riêng |
| Tính toán matrix với shared memory | **Amdahl** | Matrix cố định, chia nhỏ nhưng có memory contention |
| Web server xử lý nhiều HTTP request | **Gustafson** | Nhiều request độc lập, mỗi thread xử lý 1 request |
| Image processing: xử lý nhiều ảnh | **Gustafson** | Nhiều ảnh độc lập, mỗi thread xử lý 1 ảnh |
| Image processing: xử lý 1 ảnh lớn | **Amdahl** | 1 ảnh cố định, chia nhỏ nhưng có overhead merge |
      
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
      
| Bottleneck | Câu hỏi | Amdahl (Phải fix) | Gustafson (Có thể bỏ qua) |
|------------|---------|-------------------|---------------------------|
| **Hot Key Redis** | Key này chiếm bao nhiêu % traffic? | > 10% traffic → Shard key, cache local | < 1% traffic → Không cần fix, có nhiều key khác |
| **Single Leader** | Leader này xử lý bao nhiêu % write? | > 10% write → Shard, multi-leader | < 1% write → Không cần fix, có nhiều partition khác |
| **Lock Global** | Lock này block bao nhiêu % request? | > 10% request → Fine-grained lock | < 1% request → Không cần fix, có nhiều task khác |
| **Queue 1 Partition** | Partition này xử lý bao nhiêu % message? | > 10% message → Tăng partition | < 1% message → Không cần fix, có nhiều partition khác |
| **File dùng chung** | File này xử lý bao nhiêu % I/O? | > 10% I/O → File riêng, shard | < 1% I/O → Không cần fix, có nhiều file khác |
| **Database** | DB này xử lý bao nhiêu % query? | > 10% query → Read replica, shard, cache | < 1% query → Không cần fix, có nhiều DB khác |
      
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
      
| Scaling Type | Workload | Định luật | Khi nào dùng | Bottleneck |
|--------------|----------|-----------|--------------|------------|
| **Vertical Scaling** | Cố định | **Amdahl** | Workload nhỏ, muốn xử lý nhanh hơn | I/O, network, phần cứng max |
| **Horizontal Scaling** | Tăng theo server | **Gustafson** | Workload lớn, muốn xử lý nhiều hơn | Load balancing, consistency |
| **Read Replica** | Read tăng | **Gustafson** | Nhiều read request độc lập | Write consistency |
| **Sharding** | Data tăng | **Gustafson** | Data lớn, chia thành nhiều shard | Cross-shard query |
| **Caching** | Read tăng | **Gustafson** | Nhiều read request giống nhau | Cache invalidation |
      
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
      
| Khía cạnh | Amdahl/Gustafson | Cần học thêm |
|-----------|------------------|--------------|
| **Performance** | ✅ Giải quyết | - |
| **Consistency** | ❌ Không | CAP theorem, ACID vs BASE |
| **Availability** | ❌ Không | Redundancy, failure handling |
| **Network** | ❌ Không | CDN, edge computing, latency |
| **Cost** | ❌ Không | Cost optimization, TCO |
| **Operations** | ❌ Không | Monitoring, observability |
| **Partitioning** | ❌ Không | Sharding strategies |
| **Load Balancing** | ❌ Không | LB algorithms, session management |
| **Caching** | ❌ Không | Cache patterns, invalidation |
| **Messaging** | ❌ Không | Message queue patterns |
| **Replication** | ❌ Không | Replication patterns |
      
      **Kết luận:**
      - Amdahl/Gustafson là **nền tảng quan trọng** để hiểu performance khi scale
      - Nhưng **chưa đủ** để design system hoàn chỉnh
      - Cần học thêm: CAP theorem, Availability, Network, Cost, Operations, và các patterns khác
      - Amdahl/Gustafson giúp **quyết định khi nào scale**, nhưng không nói **cách scale như thế nào**

### Performance Metrics
- [ ] Định nghĩa chính xác: Latency, Throughput, QPS, TPS, RPS
- [ ] Viết công thức tính: QPS = ?
- [ ] Viết công thức tính: Throughput = ?
- [ ] Đọc về percentile metrics (p50, p95, p99, p999)
- [ ] Tính toán: Nếu p95 latency = 200ms, có nghĩa là gì? (viết câu trả lời)
- [ ] Tìm hiểu: Tại sao p99 quan trọng hơn average latency?
- [ ] Đọc về "tail latency" và "latency SLOs"

### Availability Concepts
- [ ] Tính toán downtime cho: 99%, 99.9%, 99.99%, 99.999% (theo năm, tháng, tuần, ngày)
- [ ] Viết bảng so sánh: Availability % → Downtime/year → Downtime/month
- [ ] Đọc về "nines" trong availability (3 nines, 4 nines, 5 nines)
- [ ] Định nghĩa: Single Point of Failure (SPOF)
- [ ] Định nghĩa: Mean Time Between Failures (MTBF)
- [ ] Định nghĩa: Mean Time To Recovery (MTTR)
- [ ] Công thức: Availability = MTBF / (MTBF + MTTR) - verify và hiểu

### Redundancy Patterns
- [ ] Đọc về Active-Active redundancy pattern
- [ ] Đọc về Active-Passive (Hot Standby) redundancy pattern
- [ ] Đọc về Active-Passive (Cold Standby) redundancy pattern
- [ ] So sánh: Active-Active vs Active-Passive (3 điểm khác biệt)
- [ ] Tìm 2 real-world examples của Active-Active
- [ ] Tìm 2 real-world examples của Active-Passive

### Bottleneck Identification
- [ ] Liệt kê 4 loại bottlenecks chính: CPU, Memory, I/O, Network
- [ ] Với mỗi bottleneck, viết 2 cách identify
- [ ] Với mỗi bottleneck, viết 2 cách resolve
- [ ] Đọc về "Amdahl's Law" trong context của bottlenecks

### Capacity Planning
- [ ] Đọc về "back-of-envelope calculations"
- [ ] Học cách estimate: Storage requirements
- [ ] Học cách estimate: Bandwidth requirements
- [ ] Học cách estimate: Compute requirements
- [ ] Practice: Estimate storage cho 1M users, mỗi user 10MB data
- [ ] Practice: Estimate bandwidth cho 10K QPS, mỗi request 2KB response

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

### Design Exercise 2: Betting Platform
- [ ] Thiết kế architecture cho Betting Platform (100K concurrent users)
- [ ] Identify bottleneck trong design (CPU, Memory, I/O, Network - chọn 1)
- [ ] Design solution để resolve bottleneck đó
- [ ] Tính toán: Peak traffic = 10x normal, design để handle
- [ ] Design horizontal scaling strategy
- [ ] Design vertical scaling strategy (nếu cần)
- [ ] So sánh: Horizontal vs Vertical cho use case này (viết 3 điểm)
- [ ] Estimate: Latency cho mỗi component (p50, p95, p99)
- [ ] Identify: Component nào sẽ là bottleneck? Tại sao?
- [ ] Viết document (500 words) về scaling strategy

### Design Exercise 3: Load Estimation
- [ ] Scenario: E-commerce site, 1M daily active users
- [ ] Estimate: Peak QPS (assume 20% traffic trong 1 hour)
- [ ] Estimate: Average request size (assume 5KB)
- [ ] Estimate: Average response size (assume 10KB)
- [ ] Calculate: Total bandwidth requirement (Mbps)
- [ ] Estimate: Database size (assume 1GB per 10K users)
- [ ] Calculate: Storage growth rate (per month)
- [ ] Estimate: Cache size needed (assume 20% of DB size)
- [ ] Create spreadsheet với tất cả calculations
- [ ] Verify: Tất cả numbers có hợp lý không? (write validation)

---

## Coding TODOs

### Task 1: Spring Boot Performance App
- [ ] Tạo Spring Boot project mới
- [ ] Tạo REST API endpoint: `GET /api/users/{id}` (return mock user data)
- [ ] Tạo REST API endpoint: `POST /api/users` (create user, save to in-memory list)
- [ ] Add logging: Log request time cho mỗi endpoint
- [ ] Add metrics: Count requests, measure latency
- [ ] Run app và test với 100 requests (manual hoặc script)
- [ ] Measure: Average latency, p95 latency
- [ ] Document: Performance baseline

### Task 2: Load Testing Setup
- [ ] Install JMeter hoặc Gatling
- [ ] Tạo test plan cho `/api/users/{id}` endpoint
- [ ] Configure: 100 concurrent users, 1000 total requests
- [ ] Run load test
- [ ] Export results: QPS, latency (p50, p95, p99), error rate
- [ ] Identify: At what load does latency spike?
- [ ] Identify: At what load do errors start?
- [ ] Document: Performance limits của app hiện tại
- [ ] Tạo report với charts (response time, throughput)

### Task 3: Performance Profiling
- [ ] Install VisualVM hoặc JProfiler
- [ ] Attach profiler to Spring Boot app
- [ ] Run load test while profiling
- [ ] Identify: Top 5 methods by CPU time
- [ ] Identify: Memory allocation hotspots
- [ ] Check: Memory leaks (heap growth over time)
- [ ] Check: Thread contention
- [ ] Document: 3 performance issues found
- [ ] Propose: 3 optimizations (không cần implement, chỉ propose)

### Task 4: Simple Load Balancer
- [ ] Tạo Java class: `SimpleLoadBalancer`
- [ ] Implement: Round-robin algorithm
- [ ] Add: List of backend servers (hardcoded URLs)
- [ ] Add: `getNextServer()` method
- [ ] Add: Health check mechanism (ping endpoint)
- [ ] Add: Skip unhealthy servers
- [ ] Test: With 3 mock backend servers
- [ ] Test: Mark one server unhealthy, verify it's skipped
- [ ] Measure: Overhead của load balancer (latency added)
- [ ] Document: Code và test results

### Task 5: Health Check Endpoints
- [ ] Add endpoint: `GET /health` (basic health check)
- [ ] Add endpoint: `GET /health/readiness` (readiness probe)
- [ ] Add endpoint: `GET /health/liveness` (liveness probe)
- [ ] Implement: Database connection check trong readiness
- [ ] Implement: Memory check trong liveness (fail if memory > 90%)
- [ ] Test: All health endpoints return correct status
- [ ] Test: Simulate DB down, verify readiness fails
- [ ] Test: Simulate high memory, verify liveness fails
- [ ] Document: Khi nào dùng readiness vs liveness

### Task 6: Metrics Collection
- [ ] Add Micrometer dependency
- [ ] Expose metrics endpoint: `GET /actuator/metrics`
- [ ] Add custom metric: `requests.total` (counter)
- [ ] Add custom metric: `request.duration` (timer)
- [ ] Instrument: All API endpoints với metrics
- [ ] Verify: Metrics được update correctly
- [ ] Export: Metrics to Prometheus format (optional)
- [ ] Document: Metrics available và ý nghĩa

---

## Analysis TODOs

### Analysis Task 1: Bottleneck Analysis
- [ ] Chọn một Spring Boot app hiện tại (hoặc tạo simple one)
- [ ] Run load test với increasing load: 10, 50, 100, 500, 1000 concurrent users
- [ ] Measure: Latency, throughput, error rate cho mỗi load level
- [ ] Plot graph: Latency vs Load
- [ ] Plot graph: Throughput vs Load
- [ ] Identify: Breaking point (khi latency spikes)
- [ ] Identify: Type of bottleneck (CPU-bound, I/O-bound, Memory-bound)
- [ ] Analyze: Tại sao bottleneck xảy ra ở điểm đó?
- [ ] Propose: 3 solutions để resolve bottleneck
- [ ] Estimate: Improvement expected từ mỗi solution

### Analysis Task 2: Capacity Planning
- [ ] Scenario: Design system cho 10M users
- [ ] Estimate: Peak concurrent users (assume 10% of total)
- [ ] Estimate: Peak QPS (assume 5 requests/user/minute)
- [ ] Calculate: Database size (assume 1KB per user)
- [ ] Calculate: Cache size (assume 10% of DB)
- [ ] Calculate: Bandwidth (assume 5KB per request)
- [ ] Estimate: Server count (assume 1 server = 10K QPS)
- [ ] Estimate: Cost (rough, assume $100/server/month)
- [ ] Create: Capacity planning spreadsheet
- [ ] Validate: Tất cả assumptions có realistic không?

### Analysis Task 3: Availability Calculation
- [ ] Calculate: Downtime budget cho 99.9% availability (per year)
- [ ] Calculate: Downtime budget cho 99.99% availability (per year)
- [ ] Scenario: System có 5 components, mỗi component có 99.9% availability
- [ ] Calculate: Overall system availability (series)
- [ ] Scenario: System có 2 redundant components (parallel), mỗi 99.9%
- [ ] Calculate: Overall system availability (parallel)
- [ ] Analyze: Cần bao nhiêu nines để có < 1 hour downtime/year?
- [ ] Analyze: Nếu MTTR = 1 hour, cần MTBF = ? để đạt 99.99%?
- [ ] Create: Availability calculator spreadsheet
- [ ] Document: Findings và insights

### Analysis Task 4: Scaling Strategy Comparison
- [ ] Scenario: Payment API, current load = 1K QPS, expected = 10K QPS
- [ ] Option 1: Vertical scaling (bigger server)
- [ ] Calculate: Cost của vertical scaling
- [ ] Calculate: Limitations (max server size)
- [ ] Option 2: Horizontal scaling (more servers)
- [ ] Calculate: Cost của horizontal scaling
- [ ] Calculate: Complexity added
- [ ] Compare: Vertical vs Horizontal (cost, complexity, limits)
- [ ] Recommend: Which strategy? Tại sao?
- [ ] Document: Decision matrix

### Analysis Task 5: Performance Baseline
- [ ] Measure: Current app performance (baseline)
- [ ] Metrics: QPS, latency (p50, p95, p99), error rate
- [ ] Document: Baseline performance
- [ ] Set: Performance goals (improve by 2x)
- [ ] Identify: Bottleneck preventing goal achievement
- [ ] Propose: Optimization plan
- [ ] Estimate: Expected improvement từ mỗi optimization
- [ ] Prioritize: Optimizations by impact/effort
- [ ] Create: Performance improvement roadmap
- [ ] Document: Analysis và recommendations

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả study TODOs đã hoàn thành chưa?
- [ ] Review: Tất cả design exercises đã làm chưa?
- [ ] Review: Tất cả coding tasks đã code và test chưa?
- [ ] Review: Tất cả analysis tasks đã complete chưa?
- [ ] Rate yourself: Understanding của scalability (1-10)
- [ ] Rate yourself: Understanding của availability (1-10)
- [ ] Rate yourself: Practical skills (load testing, profiling) (1-10)
- [ ] Identify: 3 concepts bạn hiểu rõ nhất
- [ ] Identify: 3 concepts bạn còn confuse
- [ ] Plan: Làm sao để clarify 3 concepts còn confuse?

### Design Review
- [ ] Review Payment Gateway design
- [ ] Check: Có SPOF không?
- [ ] Check: Scaling strategy có realistic không?
- [ ] Check: Capacity estimates có hợp lý không?
- [ ] Identify: 3 weaknesses trong design
- [ ] Propose: Improvements cho 3 weaknesses
- [ ] Compare: Design của bạn vs best practices
- [ ] Document: Lessons learned

### Code Review
- [ ] Review: Code quality (clean code principles)
- [ ] Review: Error handling
- [ ] Review: Logging và monitoring
- [ ] Review: Performance optimizations
- [ ] Identify: 3 code improvements needed
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Load test results
- [ ] Review: Profiling results
- [ ] Identify: Top 3 performance issues
- [ ] Verify: Performance goals đã đạt chưa?
- [ ] Document: Performance analysis
- [ ] Create: Performance improvement plan

### Knowledge Check
- [ ] Explain: Vertical vs Horizontal scaling (viết 1 paragraph, không xem notes)
- [ ] Explain: Availability calculation (viết công thức và example)
- [ ] Explain: Bottleneck identification process (viết 5 steps)
- [ ] Explain: Capacity planning approach (viết 5 steps)
- [ ] Solve: System có 3 components (99%, 99.9%, 99.99%), tính overall availability
- [ ] Solve: Estimate QPS cho 1M users, 10% online, 2 requests/user/minute
- [ ] Verify: Answers của bạn có đúng không?
- [ ] Document: Knowledge gaps found

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 điều còn confuse hoặc cần học thêm
- [ ] Write: 1 mistake bạn đã làm và lesson learned
- [ ] Write: Confidence level cho Week 2 (1-10)
- [ ] Plan: Preparation cho Week 2 (Availability & Reliability)
- [ ] Set: Goals cho Week 2
- [ ] Document: Week 1 reflection (500 words)

### Mentor Questions (Answer these)
- [ ] Q1: Nếu bạn phải scale từ 1K QPS lên 100K QPS, bạn sẽ làm gì? (viết 5 steps)
- [ ] Q2: System có 99.9% availability nhưng vẫn bị complain về downtime. Tại sao? (viết analysis)
- [ ] Q3: Làm sao bạn identify bottleneck trong production system? (viết process)
- [ ] Q4: Vertical scaling có giới hạn không? Giới hạn là gì? (viết answer)
- [ ] Q5: Tại sao p99 latency quan trọng hơn average latency? (viết explanation)
- [ ] Review: Answers của bạn có đủ depth chưa?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Design TODOs: ✅ Complete
- [ ] Tất cả Coding TODOs: ✅ Complete và tested
- [ ] Tất cả Analysis TODOs: ✅ Complete với documentation
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed to repo: ✅ Yes
- [ ] Design diagrams saved: ✅ Yes
- [ ] Ready for Week 2: ✅ Yes

---

> **Mentor Final Check**: Nếu bạn check tất cả items trên, bạn đã sẵn sàng cho Week 2. Nếu không, bạn chưa sẵn sàng. Hãy honest với bản thân.
