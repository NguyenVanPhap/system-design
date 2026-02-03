# Week 2 – Scaling Architecture & High Availability

> **Mentor Note**: Tuần 1 bạn đã học fundamentals. Tuần 2 này bạn phải THỰC HÀNH scaling và high availability. Không có
> lý thuyết suông. Mỗi concept phải được implement và test. Nếu bạn không code, bạn không học được gì.

---

## Study TODOs

### Horizontal vs Vertical Scaling Deep Dive

- [ ] Đọc "Designing Data-Intensive Applications" Chapter 1 - Scaling approaches (pages 30-50)
- [ ] Viết comparison table: Horizontal vs Vertical (10 điểm so sánh)
    - Compare: Concept, Scalability Limit, Cost, Availability, Fault Tolerance, Performance, Complexity, Maintenance,
      Flexibility, Use Cases
        - Concept: Vertical scaling là tăng tài nguyên của một server (CPU, RAM), horizontal scaling là thêm nhiều
          server hơn.
        - Scalability Limit: Vertical scaling bị giới hạn bởi kích thước tối đa của server, horizontal scaling gần như
          không giới hạn.
        - Cost: Vertical scaling có thể rẻ hơn ban đầu nhưng đắt hơn ở quy mô lớn, horizontal scaling có chi phí linh
          hoạt hơn.
        - Availability: Horizontal scaling cung cấp độ sẵn sàng cao hơn do không có SPOF.
        - Fault Tolerance: Horizontal scaling tốt hơn trong việc chịu lỗi vì có nhiều bản sao.
        - Performance: Vertical scaling có thể cung cấp hiệu suất cao hơn cho các tác vụ đơn luồng.
        - Complexity: Horizontal scaling phức tạp hơn trong quản lý và điều phối.
        - Maintenance: Vertical scaling dễ bảo trì hơn vì ít thành phần hơn.
        - Flexibility: Horizontal scaling linh hoạt hơn trong việc mở rộng và thu nhỏ.
        - Use Cases: Vertical scaling phù hợp cho các ứng dụng nhỏ, horizontal scaling cho các ứng dụng lớn, phân tán.
- [ ] Liệt kê 5 limitations của vertical scaling
    1. Giới hạn phần cứng: Có giới hạn về kích thước và khả năng của một máy chủ đơn.
    2. Chi phí tăng cao: Máy chủ lớn hơn thường đắt đỏ hơn và chi phí bảo trì cũng cao hơn.
    3. Single Point of Failure (SPOF): Nếu máy chủ gặp sự cố, toàn bộ hệ thống có thể bị ảnh hưởng.
    4. Downtime during upgrades: Nâng cấp phần cứng có thể yêu cầu downtime.
    5. Khả năng mở rộng hạn chế: Không thể mở rộng vô hạn chỉ bằng cách nâng cấp một máy chủ.
- [ ] Liệt kê 5 challenges của horizontal scaling
    1. Phức tạp trong quản lý: Cần có hệ thống để quản lý nhiều máy chủ.
    2. Đồng bộ dữ liệu: Cần giải pháp để đồng bộ dữ liệu giữa các máy chủ.
    3. Chi phí mạng: Giao tiếp giữa các máy chủ có thể tạo ra chi phí mạng đáng kể.
    4. Load balancing: Cần có cơ chế phân phối tải hiệu quả.
    5. Giám sát và bảo trì: Cần hệ thống giám sát để theo dõi trạng thái của nhiều máy chủ.

- [ ] Đọc về "scaling laws" - khi nào vertical scaling fails?
    - Vertical scaling thường thất bại khi:
        1. Đạt đến giới hạn phần cứng tối đa của máy chủ.
        2. Chi phí nâng cấp vượt quá lợi ích thu được.
        3. Yêu cầu về độ sẵn sàng và chịu lỗi cao không thể đáp ứng.
        4. Tăng tải đột ngột vượt quá khả năng xử lý của máy chủ đơn.
        5. Cần mở rộng linh hoạt và nhanh chóng mà vertical scaling không thể đáp ứng kịp thời.
- [ ] Research: Maximum server size available (CPU cores, RAM) - current limits
    - Hiện tại, các máy chủ thương mại có thể có tới 128 CPU cores và 4TB RAM, nhưng các máy chủ chuyên dụng có thể vượt
      qua con số này.
- [ ] Research: Cost comparison - 1 large server vs 10 small servers (same capacity)
    - Chi phí của một máy chủ lớn có thể cao hơn 20-30% so với việc sử dụng 10 máy chủ nhỏ hơn với tổng công suất tương
      đương, nhưng chi phí bảo trì và quản lý có thể thấp hơn.
    - Tuy nhiên, chi phí mạng và phức tạp trong quản lý có thể làm tăng tổng chi phí sở hữu (TCO) của giải pháp
      horizontal scaling.
    - Tổng chi phí phụ thuộc vào nhiều yếu tố như nhà cung cấp, cấu hình cụ thể và yêu cầu về hiệu suất.
    - Nên thực hiện phân tích chi phí cụ thể dựa trên nhu cầu và môi trường triển khai.
    - Tóm lại, chi phí ban đầu của vertical scaling có thể thấp hơn, nhưng horizontal scaling cung cấp sự linh hoạt và
      độ sẵn sàng cao hơn với chi phí dài hạn có thể cạnh tranh.
- [ ] Đọc về "scaling out" vs "scaling up" terminology
    - Scaling out (horizontal scaling) là quá trình thêm nhiều máy chủ hoặc nút vào hệ thống để tăng khả năng xử lý.
    - Scaling up (vertical scaling) là quá trình nâng cấp tài nguyên của một máy chủ hiện có, như tăng CPU, RAM hoặc
      dung lượng lưu trữ.
- [ ] Tìm 3 systems that MUST use horizontal scaling (và tại sao)
    1. Google Search: Cần xử lý hàng tỷ truy vấn mỗi ngày, yêu cầu khả năng mở rộng linh hoạt và độ sẵn sàng cao.
    2. Facebook: Với hàng tỷ người dùng và lượng dữ liệu khổng lồ, horizontal scaling giúp duy trì hiệu suất và độ sẵn
       sàng.
    3. Amazon Web Services (AWS): Cung cấp dịch vụ đám mây cho hàng triệu khách hàng, yêu cầu khả năng mở rộng nhanh
       chóng và linh hoạt để đáp ứng nhu cầu thay đổi.
- [ ] Tìm 2 systems that CAN use vertical scaling (và tại sao họ chọn)
    1. Small Business Websites: Các trang web nhỏ thường có lưu lượng truy cập thấp và không yêu cầu khả năng mở rộng
       lớn, do đó vertical scaling là đủ và tiết kiệm chi phí.
    2. Legacy Enterprise Applications: Nhiều ứng dụng doanh nghiệp cũ được thiết kế để chạy trên một máy chủ duy nhất và
       có thể không cần mở rộng lớn, do đó vertical scaling là lựa chọn hợp lý để duy trì hiệu suất mà không cần thay
       đổi kiến trúc.

### Load Balancer Strategies

- [ ] Đọc về 5 load balancing algorithms: Round-robin, Weighted Round-robin, Least Connections, IP Hash, Least Response
  Time
    1. Round-robin: Phân phối các yêu cầu đến các máy chủ theo thứ tự tuần tự.
    2. Weighted Round-robin: Tương tự như round-robin nhưng phân phối dựa trên trọng số được gán cho mỗi máy chủ.
    3. Least Connections: Gửi yêu cầu đến máy chủ có ít kết nối hiện tại nhất.
    4. IP Hash: Sử dụng địa chỉ IP của khách hàng để xác định máy chủ nhận yêu cầu, giúp duy trì phiên làm việc (sticky
       sessions).
    5. Least Response Time: Gửi yêu cầu đến máy chủ có thời gian phản hồi nhanh nhất.

- [ ] Viết algorithm pseudocode cho mỗi algorithm
    - Round-robin:
        ```
        currentIndex = 0
        function getNextServer(servers):
            server = servers[currentIndex]
            currentIndex = (currentIndex + 1) % length(servers)
            return server
        ```
    - Weighted Round-robin:
        ```
        weights = [weight1, weight2, ..., weightN]
        currentIndex = -1
        currentWeight = 0
        function getNextServer(servers):
            while true:
                currentIndex = (currentIndex + 1) % length(servers)
                if currentIndex == 0:
                    currentWeight = currentWeight - gcd(weights)
                    if currentWeight <= 0:
                        currentWeight = max(weights)
                        if currentWeight == 0:
                            return null
                if weights[currentIndex] >= currentWeight:
                    return servers[currentIndex]
        ```
    - Least Connections:
        ```
        function getNextServer(servers):
            minConnections = infinity
            selectedServer = null
            for server in servers:
                if server.activeConnections < minConnections:
                    minConnections = server.activeConnections
                    selectedServer = server
            return selectedServer
        ```
    - IP Hash:
        ```
        function getNextServer(servers, clientIP):
            hashValue = hash(clientIP)
            index = hashValue % length(servers)
            return servers[index]
        ```
    - Least Response Time:
        ```
        function getNextServer(servers):
            minResponseTime = infinity
            selectedServer = null
            for server in servers:
                if server.responseTime < minResponseTime:
                    minResponseTime = server.responseTime
                    selectedServer = server
            return selectedServer
        ```
- [ ] So sánh: Round-robin vs Least Connections (3 use cases mỗi cái)
    - Round-robin:
        1. Use Case 1: Ứng dụng web với tải đồng đều, nơi mỗi máy chủ có khả năng xử lý tương tự.
        2. Use Case 2: Hệ thống không yêu cầu duy trì phiên làm việc (stateless applications).
        3. Use Case 3: Môi trường thử nghiệm hoặc phát triển với số lượng máy chủ nhỏ.
    - Least Connections:
        1. Use Case 1: Ứng dụng web với tải không đồng đều, nơi một số máy chủ có thể xử lý nhiều kết nối hơn.
        2. Use Case 2: Hệ thống có các phiên làm việc dài, nơi một số kết nối có thể chiếm nhiều tài nguyên hơn.
        3. Use Case 3: Ứng dụng thời gian thực như chat hoặc streaming, nơi kết nối có thể kéo dài.
- [ ] Đọc về "sticky sessions" (session affinity)
    - Sticky sessions là một kỹ thuật trong load balancing để đảm bảo rằng các yêu cầu từ cùng một người dùng (hoặc
      phiên làm việc) luôn được gửi đến cùng một máy chủ backend. Điều này quan trọng đối với các ứng dụng stateful, nơi
      trạng thái của người dùng được lưu trữ trên máy chủ cụ thể
- [ ] Analyze: Khi nào cần sticky sessions? Trade-offs?
    - Cần sticky sessions khi:
        1. Ứng dụng stateful: Khi trạng thái người dùng được lưu trữ trên máy chủ cụ thể.
        2. Phiên làm việc dài: Khi người dùng có các tương tác kéo dài với ứng dụng.
        3. Yêu cầu hiệu suất: Khi việc duy trì phiên làm việc trên cùng một máy chủ giúp giảm độ trễ.
- [ ] Đọc về Layer 4 (L4) vs Layer 7 (L7) load balancing
    - Layer 4 (L4) load balancing hoạt động ở tầng vận chuyển của mô hình OSI, sử dụng thông tin như địa chỉ IP và cổng
      để phân phối lưu lượng.
    - Layer 7 (L7) load balancing hoạt động ở tầng ứng dụng, sử dụng thông tin chi tiết hơn như URL, tiêu đề HTTP để đưa
      ra quyết định phân phối.
    - L4 load balancing thường nhanh hơn và ít phức tạp hơn
    - Trong khi L7 load balancing cung cấp khả năng tùy chỉnh cao hơn và hỗ trợ các tính năng như SSL termination,
      cookie-based routing.
    - L4 load balancing phù hợp cho các ứng dụng đơn giản và yêu cầu hiệu suất cao
    - Trong khi L7 load balancing thích hợp cho các ứng dụng phức tạp cần xử lý logic ứng dụng.
- [ ] So sánh: L4 vs L7 (performance, features, use cases)
    - Performance: L4 load balancing nhanh hơn do xử lý ít thông tin hơn, trong khi L7 load balancing chậm hơn do phân
      tích sâu hơn.
    - Features: L7 load balancing cung cấp nhiều tính năng hơn như SSL termination, URL-based routing, trong khi L4 tập
      trung vào phân phối cơ bản.
    - Use Cases: L4 phù hợp cho các ứng dụng đơn giản và yêu cầu hiệu suất cao, trong khi L7 thích hợp cho các ứng dụng
      phức tạp cần xử lý logic ứng dụng.
- [ ] Research: Popular load balancers (Nginx, HAProxy, AWS ELB, F5)
    - Nginx: Một web server và reverse proxy phổ biến, hỗ trợ L7 load balancing với nhiều tính năng như SSL termination,
      caching.
    - Ví dụ sử dụng: Netflix, Airbnb.
    - HAProxy: Một load balancer hiệu suất cao, hỗ trợ cả L4 và L7 load balancing, thường được sử dụng trong các môi
      trường yêu cầu hiệu suất cao.
    - AWS ELB (Elastic Load Balancing): Dịch vụ load balancing của Amazon Web Services, hỗ trợ cả L4 và L7, tích hợp tốt
      với các dịch vụ AWS khác.
    - F5: Một giải pháp load balancing doanh nghiệp với nhiều tính năng nâng cao như bảo mật ứng dụng, tối ưu hóa hiệu
      suất.
      Ok. Mình viết **ngắn – đúng thực tế – không lan man**.

---

# 📘 ỨNG DỤNG THỰC TẾ CỦA NGINX – HAPROXY – AWS ELB – F5 (VIẾT CHUẨN THỰC CHIẾN)

Mục tiêu: hiểu đúng **vai trò**, **điểm mạnh**, **vị trí trong kiến trúc**, và **khi nào chọn cái nào**.

---

## 0. Một câu để nhớ

- **NGINX**: reverse proxy/app edge “gần ứng dụng” (HTTP focus).
- **HAProxy**: load balancer “cứng” ở **L4/TCP**, cũng làm được L7.
- **AWS ELB**: load balancer **managed** trên AWS (ALB/NLB/GWLB).
- **F5**: **ADC enterprise** + policy + security modules (WAF/DDoS/TLS… tuỳ giải pháp).

---

## 1. NGINX – REVERSE PROXY / APP EDGE (L7)

### Dùng để làm gì ngoài đời (phổ biến nhất):

- Reverse proxy cho backend (hide origin, unify routing)
- TLS termination (SSL offload)
- Routing/rewrite/redirect
- Basic controls: allow/deny, rate limit, header controls
- Caching (static + một phần dynamic khi hợp lý)
- Serve static (assets)

### Vị trí điển hình:

```
Client → (CDN/WAF/LB tuỳ) → Nginx → App
```

### Khi nào dùng (rule-of-thumb):

- Bạn cần **gateway gần app** để chuẩn hoá: TLS, routing, rate limit, cache, header… và muốn tự vận hành cấu hình.
- Nếu dùng cloud-managed gateway/proxy (ALB/API Gateway/Envoy/Istio ingress), **có thể không cần Nginx riêng**.

---

## 2. HAPROXY – LOAD BALANCER MẠNH Ở L4 (TCP) (VÀ CŨNG LÀM ĐƯỢC L7)

### Dùng để làm gì ngoài đời:

- L4/TCP load balancing (hiệu năng cao, ổn định cho connection dài)
- Health check tốt, routing/failover theo tình trạng backend
- WebSocket/long-lived connections (tuỳ design; Nginx cũng làm được)
- Internal LB (on-prem/self-host) cho service/cluster
- Routing cho DB layer theo vai trò (primary/replica) **nếu bạn đã có cơ chế xác định vai trò**

### Điểm dễ hiểu sai (quan trọng):

> HAProxy **không tự bầu chọn/đề cử master DB**. Nó chỉ **route traffic** theo health-check.  
> Promote/failover DB là việc của DB/tooling (ví dụ Postgres: Patroni; MySQL: Orchestrator; Redis: Sentinel…).

### Vị trí điển hình:

```
Client/App → HAProxy → Backend nodes (TCP/HTTP)
```

### Khi nào dùng (rule-of-thumb):

- Khi workload thiên về **TCP/L4**, connection dài, cần LB “thực dụng”, observability/health-check rõ ràng.
- Khi bạn self-host và muốn **LB trung tâm** cho nhiều cluster (DB, MQ, realtime).

---

## 3. AWS ELB – LOAD BALANCER MANAGED TRÊN AWS

### Chọn đúng “loại”:

- **ALB**: L7 HTTP/HTTPS, host/path routing, phù hợp web/app API.
- **NLB**: L4 TCP/UDP/TLS, hiệu năng cao, IP-based, phù hợp TCP/realtime.
- **GWLB**: “chèn” security appliances (appliance insertion) vào traffic path.

### Dùng để làm gì ngoài đời:

- Multi-AZ high availability
- Target groups + health checks
- TLS termination (ALB/NLB TLS)
- Tích hợp ECS/EKS/EC2/Auto Scaling
- Hỗ trợ blue/green/canary thông qua target groups/weights (kết hợp deploy tooling tuỳ cách làm)

### Vị trí điển hình:

```
Internet → ALB/NLB/GWLB → Targets (EC2/ECS/EKS/Pods)
```

### Khi nào dùng (rule-of-thumb):

- Trên AWS thì **ưu tiên ELB trước**, vì nó managed, HA sẵn, tích hợp hệ sinh thái tốt.
- Chỉ thêm Nginx/Envoy “gần app” khi bạn cần: caching, rewrite phức tạp, custom behaviors, hoặc standardize giống on-prem.

---

## 4. F5 – ADC ENTERPRISE + POLICY + SECURITY (TUỲ MODULE)

### Dùng để làm gì ngoài đời (tuỳ công ty/mua license):

- ADC/traffic management enterprise (LB, routing policy, TLS policy)
- WAF (ví dụ BIG-IP ASM/Advanced WAF)
- DDoS/Firewall capabilities (tuỳ module/giải pháp)
- TLS offload quy mô lớn + kiểm soát cipher/policy
- Tích hợp SSO/identity/policy (tuỳ kiến trúc)

### Vị trí điển hình:

```
Internet → (F5/WAF/ADC) → (LB) → App/Core systems
```

### Khi nào dùng (rule-of-thumb):

- Enterprise/on-prem/hybrid, yêu cầu policy, audit, quy trình thay đổi chặt, và cần giải pháp “đóng gói” cho traffic + security.
- Không phải “ngân hàng mới dùng”; nhưng trong regulated/enterprise thì **tỷ lệ gặp cao**.

---

## 5. KIẾN TRÚC THỰC TẾ PHỔ BIẾN (MẪU NHANH)

### A) Startup / SME (đơn giản, tự host)

```
Nginx → App
```

### B) Web/SaaS trên AWS (managed-first)

```
CDN/WAF → ALB → App (ECS/EKS/EC2)
```

### C) AWS + cần “app edge” riêng (cache/rewrite/chuẩn hoá)

```
CDN/WAF → ALB → Nginx/Envoy → App
```

### D) DB/cluster/realtime TCP self-host

```
HAProxy → TCP/Cluster nodes
```

### E) Enterprise on-prem/hybrid (policy + security nặng)

```
WAF/ADC (F5) → LB/Proxy → App/Core
```

---

## 6. SO SÁNH NHANH (ĐÚNG TRỌNG TÂM)

| Tool    | Thế mạnh chính | Layer hay dùng | Khi chọn nhanh |
|---------|-----------------|----------------|----------------|
| Nginx   | Reverse proxy gần app, TLS, cache, routing | L7 | Web/API edge cần tự control |
| HAProxy | LB L4 “cứng”, health-check rõ | L4 (cũng L7) | TCP/cluster/realtime/self-host |
| ELB     | Managed HA + tích hợp AWS | ALB(L7)/NLB(L4)/GWLB | Chạy trên AWS |
| F5      | ADC enterprise + policy + security modules | L7/L4 tuỳ | Enterprise/regulated/on-prem/hybrid |

---

## 7. QUY TẮC CHỌN (RẤT THỰC TẾ)

- Nếu bạn **đang ở AWS**: chọn **ALB/NLB/GWLB** theo nhu cầu trước.
- Nếu bạn cần **app edge tự kiểm soát** (TLS/routing/cache/rate limit): thêm **Nginx/Envoy**.
- Nếu bạn cần **L4/TCP LB** hoặc self-host cluster: **HAProxy** là lựa chọn rất mạnh.
- Nếu bạn ở enterprise và “security/policy/compliance-driven”: **F5/ADC/WAF enterprise** thường xuất hiện.

Ghi chú: Không có “one size fits all”. Chọn theo **L4 vs L7**, **managed vs self-host**, **security/policy**, và **operational cost**.

- [ ] Compare: Nginx vs HAProxy (features, performance, use cases)
- [ ] Đọc về "health checks" trong load balancers
- [ ] Đọc về "graceful degradation" khi backend fails

### Stateless vs Stateful Services

- [ ] Định nghĩa chính xác: Stateless service
- **Stateless service**: service mà **mỗi request tự chứa đủ context** để xử lý (auth/context nằm trong request token/headers), và **không phụ thuộc state cục bộ trên instance** giữa các request.
  - State vẫn tồn tại, nhưng được **externalize** (DB/Redis/object storage/queue) thay vì nằm trong memory/disk cục bộ của instance.
  - Hệ quả: có thể scale horizontal dễ, thay instance không làm “mất session”.
- [ ] Định nghĩa chính xác: Stateful service
- **Stateful service**: service mà xử lý request **phụ thuộc state được giữ lại trên instance** (in-memory session, local cache mang tính quyết định, local files...), nên request sau **cần quay lại đúng instance** hoặc phải replicate state.
- [ ] Liệt kê 5 characteristics của stateless service
    1. Không yêu cầu sticky session để đúng chức năng (có thể dùng, nhưng không “bắt buộc”).
    2. Thay/kill instance không làm mất phiên (không mất “user state” tại app layer).
    3. Scale in/out nhanh, phù hợp autoscaling.
    4. Dễ rolling/canary vì instance interchangeable.
    5. State nằm ở storage chung: DB/Redis/Kafka/S3… (có chiến lược consistency rõ).
- [ ] Liệt kê 5 characteristics của stateful service
    1. Có session/state cục bộ (memory/disk) ảnh hưởng trực tiếp tới response.
    2. Thường cần sticky sessions hoặc state replication.
    3. Scale horizontal khó hơn (sharding theo user/room/key…).
    4. Deploy/failover phức tạp vì phải drain/transfer state.
    5. Risk mất dữ liệu phiên khi instance chết (nếu không replicate).
- [ ] Analyze: REST API - stateless hay stateful? Tại sao?
    - **REST API “đúng chuẩn” là stateless**: server không giữ session state giữa các request; mỗi request mang đủ info (token, params).
    - Thực tế: vẫn có thể “biến tướng” stateful (server-side session, in-memory cart…), nhưng sẽ giảm khả năng scale/HA.
- [ ] Analyze: WebSocket connection - stateless hay stateful? Tại sao?
    - **WebSocket bản chất là stateful ở tầng connection**: có connection dài, state như subscription, room membership, in-flight messages…
    - Có thể “giảm statefulness” ở app bằng cách:
        - Externalize presence/subscriptions vào Redis (pub/sub), Kafka, etc.
        - Dùng consistent hashing/sharding theo user/room để route ổn định.
        - Dùng reconnect + resume token (idempotency) để chịu lỗi.
- [ ] Đọc về "session state" - where to store?
    - **Options**:
        - Client-side: JWT (stateless auth) — không store session server, nhưng phải xử lý revoke/rotation.
        - Server-side external store: Redis (phổ biến), DB (nặng), distributed cache.
        - Tránh: in-memory session nếu cần scale/HA.
- [ ] Compare: In-memory session vs Redis session vs Database session (3 pros/cons mỗi cái)
    - **In-memory session**
        - Pros: nhanh, đơn giản, rẻ khi 1 instance.
        - Cons: mất session khi instance chết, cần sticky session, khó scale.
    - **Redis session**
        - Pros: nhanh, shared giữa nhiều instance, hỗ trợ TTL, scale tốt.
        - Cons: thêm dependency, cần HA cho Redis, có latency mạng.
    - **Database session**
        - Pros: durable, dễ audit, consistency rõ.
        - Cons: load cao, latency cao hơn, dễ thành bottleneck.
- [ ] Đọc về "stateless authentication" (JWT tokens)
    - JWT: client giữ token; server verify signature + claims.
    - Cần: key rotation, expiry ngắn + refresh token, chống replay (tuỳ threat model).
- [ ] Đọc về "stateful authentication" (server-side sessions)
    - Session id (cookie) trỏ tới session store server-side (memory/Redis/DB).
    - Dễ revoke ngay, nhưng cần store + scale strategy.
- [ ] Analyze: Khi nào cần stateful service? (3 use cases)
    1. Realtime apps cần connection state: chat, trading UI, multiplayer game (WebSocket).
    2. Streaming/long-running workflows có state machine tại edge (đôi khi).
    3. Các hệ thống cần local state để tối ưu cực mạnh và chấp nhận trade-off (hiếm; thường vẫn externalize).
- [ ] Analyze: Khi nào MUST use stateless? (3 use cases)
    1. Public REST APIs cần autoscaling + rolling deploy liên tục.
    2. Payment/critical APIs cần HA cao, failover nhanh, không phụ thuộc instance.
    3. Microservices chạy trên Kubernetes/ASG: instance ephemeral → stateless là mặc định tốt.
- [ ] Đọc về "shared nothing architecture"
    - “Shared nothing”: mỗi node không chia sẻ memory/disk; phối hợp qua network + storage phân tán.
    - Lợi: scale tốt, fault isolation; Hại: cần giải quyết consistency/partitioning.

### Redundancy & Failover Patterns

- [ ] Đọc về "Active-Active" redundancy pattern (deep dive)
    - **Active-Active**: nhiều instance cùng nhận traffic; LB phân phối; cần stateless hoặc shared state.
    - Ưu: tận dụng tài nguyên, failover nhanh; Nhược: phức tạp (consistency, split-brain ở data layer).
- [ ] Đọc về "Active-Passive" redundancy pattern (deep dive)
    - **Active-Passive**: 1 active phục vụ, 1+ standby không nhận traffic (hoặc minimal).
    - Ưu: đơn giản hơn; Nhược: lãng phí, failover có RTO, cần cơ chế promote.
- [ ] Đọc về "Active-Passive Hot Standby" vs "Active-Passive Cold Standby"
    - **Hot standby**: standby chạy sẵn, sync state/replication, failover nhanh (RTO thấp).
    - **Cold standby**: standby tắt/ít tài nguyên, khởi động khi sự cố (RTO cao, rẻ hơn).
- [ ] Compare: Active-Active vs Active-Passive (cost, complexity, failover time, use cases)
    - **Cost**: Active-Active cao hơn (run full capacity nhiều nơi) vs Active-Passive thấp hơn nếu passive nhỏ.
    - **Complexity**: Active-Active cao (data consistency, routing) vs Active-Passive thấp hơn.
    - **Failover time**: Active-Active thường gần như tức thì; Active-Passive phụ thuộc detection + promote.
    - **Use cases**:
        - Active-Active: stateless app tier, CDN, multi-AZ web tier.
        - Active-Passive: primary-replica DB, một số hệ thống legacy.
- [ ] Đọc về "failover mechanisms": Automatic vs Manual
    - Automatic: nhanh, giảm downtime; cần test kỹ để tránh failover “false positive”.
    - Manual: an toàn hơn trong vài hệ, nhưng downtime lớn và phụ thuộc con người.
- [ ] Đọc về "failover time" - RTO (Recovery Time Objective)
    - **RTO**: thời gian tối đa chấp nhận để khôi phục dịch vụ sau sự cố (downtime budget).
- [ ] Đọc về "data loss" - RPO (Recovery Point Objective)
    - **RPO**: mức dữ liệu tối đa có thể mất (độ trễ replication/backup), ví dụ RPO=1 phút nghĩa là có thể mất tối đa 1 phút data.
- [ ] Calculate: Nếu RTO = 5 minutes, có nghĩa là gì?
    - Nghĩa là khi sự cố xảy ra, hệ thống phải **khôi phục phục vụ** trong ≤ 5 phút (từ góc nhìn người dùng/SLA).
- [ ] Calculate: Nếu RPO = 1 minute, có nghĩa là gì?
    - Nghĩa là có thể chấp nhận mất tối đa 1 phút dữ liệu gần nhất (do replication/backup lag).
- [ ] Đọc về "split-brain problem" trong Active-Active
    - Split-brain: network partition khiến 2 phía đều nghĩ mình “primary”, dẫn tới writes conflict/corruption nếu không có consensus/quorum.
- [ ] Đọc về "quorum" và "consensus" trong distributed systems
    - Quorum: quyết định dựa trên đa số (N/2+1) để tránh split-brain.
    - Consensus (Raft/Paxos): đảm bảo agreement về leader/log giữa các node.
- [ ] Research: How does database replication handle failover? (Master-Slave)
    - Mô hình phổ biến: primary-replica; failover cần:
        - Detect primary down (health checks/timeout).
        - Promote replica lên primary (tooling).
        - Re-point clients/LB/DNS tới primary mới.
        - Handle replication lag → liên quan RPO.
- [ ] Research: How does application-level failover work?
    - App tier: nhiều instances + LB health check.
    - Circuit breaker/retry + timeout để tránh “cascading failure”.
    - Readiness/liveness để LB/k8s loại instance xấu.

### Zero-Downtime Deployment

- [ ] Đọc về "blue-green deployment" strategy
    - Blue (old) & Green (new) chạy song song; switch traffic một lần (DNS/LB).
- [ ] Đọc về "canary deployment" strategy
    - Đẩy một phần nhỏ traffic vào version mới, tăng dần nếu ổn; rollback nhanh.
- [ ] Đọc về "rolling deployment" strategy
    - Thay dần từng batch instance; cần backward-compatible.
- [ ] Compare: Blue-Green vs Canary vs Rolling (cost, risk, complexity, downtime)
    - **Blue-Green**: downtime ~0; cost cao (2 môi trường), rollback nhanh; risk “big bang” khi switch.
    - **Canary**: cost vừa; risk thấp vì thử ít traffic; cần metrics/observability tốt.
    - **Rolling**: cost thấp; risk trung bình; rollback có thể chậm nếu đã rollout nhiều.
- [ ] Đọc về "feature flags" trong zero-downtime deployments
    - Deploy code tách khỏi release feature; bật/tắt theo cohort; rollback feature nhanh không cần deploy lại.
- [ ] Đọc về "database migrations" trong zero-downtime deployments
    - Nguyên tắc: **expand-and-contract**:
        - Expand: add columns/tables mới, code hỗ trợ cả cũ & mới.
        - Migrate: backfill.
        - Contract: remove cột cũ sau khi chắc chắn.
- [ ] Analyze: Backward compatibility requirements
    - API: versioning/optional fields.
    - DB: schema changes không breaking khi còn instance old/new cùng chạy.
- [ ] Đọc về "traffic shifting" strategies
    - Weighted target groups, header-based routing, region-based, time-based.
- [ ] Đọc về "rollback strategies" - how to rollback quickly?
    - Fast rollback = switch traffic về version cũ (blue-green) hoặc set weight=0 (canary).
    - Data rollback khó hơn code rollback → cần migration strategy.
- [ ] Research: Kubernetes deployment strategies
    - RollingUpdate, blue-green/canary qua Argo Rollouts/Flagger, service mesh (Istio/Linkerd).
- [ ] Research: Spring Boot zero-downtime deployment patterns
    - Graceful shutdown + readiness probes.
    - Connection draining ở LB.
    - Backward-compatible API/DB changes.

### Capacity Planning Basics

- [ ] Đọc về "capacity planning" process (5 steps)
    1. Đo baseline (QPS/latency/CPU/memory/IO).
    2. Xác định SLO/SLA + headroom.
    3. Dự báo growth + seasonality.
    4. Tính capacity theo bottleneck (CPU/DB/IO/network).
    5. Lập kế hoạch scale + monitoring/alerts + load test định kỳ.
- [ ] Đọc về "baseline measurement" - current capacity
    - Đo ở peak/average: QPS, P95/P99, saturation (CPU steal, GC), DB connections, cache hit rate.
- [ ] Đọc về "growth projection" - future capacity needs
    - Dựa trên MAU/DAU, conversion, feature changes; dùng mô hình worst-case + seasonal peaks.
- [ ] Đọc về "headroom" - buffer capacity
    - Headroom 20–50% tuỳ hệ; payment/realtime thường cần headroom cao hơn.
- [ ] Calculate: Nếu current load = 1K QPS, growth = 20%/month, cần capacity sau 6 tháng?
    - Công thức: \(QPS_{6} = 1000 \times 1.2^6 \approx 1000 \times 2.986 = 2986\) QPS.
    - Nếu headroom 20%: cần \(\approx 2986 \times 1.2 \approx 3583\) QPS capacity.
- [ ] Đọc về "capacity units" - how to measure?
    - QPS/RPS, concurrent users/connections, bytes/s, DB TPS, cache ops/s, queue lag, compute units.
- [ ] Đọc về "scaling triggers" - when to scale?
    - CPU > 60–70% sustained, P95 latency vượt SLO, queue lag tăng, DB connections/IO saturation.
- [ ] Đọc về "auto-scaling" vs "manual scaling"
    - Auto: phản ứng nhanh, cần guardrails; Manual: kiểm soát tốt nhưng chậm.
- [ ] Research: AWS Auto Scaling groups
    - ASG + scaling policies (target tracking, step scaling) + health checks + warm-up.
- [ ] Research: Kubernetes Horizontal Pod Autoscaler (HPA)
    - HPA theo CPU/memory/custom metrics; cần request/limit đúng; kết hợp VPA/Cluster Autoscaler khi cần.

---

## Design TODOs

### Design Exercise 1: Payment Gateway - Horizontal Scaling

- [ ] Thiết kế Payment Gateway với horizontal scaling requirement
- [ ] Requirement: Scale from 1K QPS to 50K QPS
- [ ] Design: Load balancer architecture (type, algorithm, placement)
- [ ] Design: Application server scaling strategy
- [ ] Design: Database scaling strategy (read replicas, sharding consideration)
- [ ] Design: Stateless service architecture (no session state)
- [ ] Identify: All stateful components (nếu có)
- [ ] Design: Solution để make stateful components stateless
- [ ] Calculate: Number of servers needed (assume 5K QPS per server)
- [ ] Calculate: Cost comparison - vertical vs horizontal scaling
- [ ] Design: Auto-scaling rules (when to scale up/down)
- [ ] Document: Complete architecture diagram với scaling paths
- [ ] Document: Design decisions và trade-offs (500 words)

### Design Exercise 2: Betting Platform - High Availability

- [ ] Thiết kế Betting Platform với 99.99% availability requirement
- [ ] Design: Multi-layer redundancy (application, database, load balancer)
- [ ] Design: Active-Active setup cho application servers
- [ ] Design: Active-Passive setup cho database (Master-Slave)
- [ ] Design: Failover mechanism (automatic, < 30 seconds)
- [ ] Design: Health check strategy (what to check, how often)
- [ ] Design: Monitoring và alerting (what to monitor, alert thresholds)
- [ ] Identify: All SPOFs trong design
- [ ] Eliminate: All SPOFs (redesign nếu cần)
- [ ] Design: Disaster recovery plan (RTO = 5 minutes, RPO = 1 minute)
- [ ] Design: Multi-region deployment (optional, nhưng nên consider)
- [ ] Calculate: Availability của toàn bộ system (series và parallel components)
- [ ] Document: Failure scenarios (ít nhất 5 scenarios) và mitigation
- [ ] Document: Complete HA architecture (500 words)

### Design Exercise 3: Wallet System - Stateless Architecture

- [ ] Thiết kế Wallet System với stateless requirement
- [ ] Requirement: Must be horizontally scalable, no session state
- [ ] Design: Authentication strategy (stateless - JWT)
- [ ] Design: Session management (nếu cần, dùng external store)
- [ ] Design: State storage (Redis cho shared state)
- [ ] Identify: All stateful operations
- [ ] Design: Solution để make operations stateless
- [ ] Design: Load balancer configuration (algorithm, sticky sessions?)
- [ ] Design: Cache strategy (shared cache, not in-memory)
- [ ] Design: Database connection strategy (connection pooling, no local state)
- [ ] Verify: System có thể scale horizontally không? (checklist)
- [ ] Document: Stateless architecture design (500 words)

### Design Exercise 4: Zero-Downtime Deployment Strategy

- [ ] Design: Deployment strategy cho Payment Gateway
- [ ] Requirement: Zero downtime, support rollback
- [ ] Choose: Blue-Green, Canary, or Rolling deployment
- [ ] Justify: Tại sao chọn strategy đó?
- [ ] Design: Traffic routing during deployment
- [ ] Design: Database migration strategy (backward compatible)
- [ ] Design: Feature flag strategy (nếu cần)
- [ ] Design: Rollback procedure (step-by-step)
- [ ] Design: Health check during deployment
- [ ] Design: Monitoring during deployment (what to watch)
- [ ] Estimate: Deployment time và risk
- [ ] Document: Complete deployment strategy (500 words)

### Design Exercise 5: Capacity Planning

- [ ] Scenario: E-commerce platform, current 10K users, expected 1M users in 1 year
- [ ] Measure: Current capacity (QPS, storage, bandwidth) - estimate nếu không có data
- [ ] Project: Future capacity needs (1M users)
- [ ] Calculate: Growth rate (monthly)
- [ ] Calculate: Peak capacity needed (assume 10x average)
- [ ] Design: Scaling plan (when to add servers)
- [ ] Calculate: Server count needed (assume 5K QPS per server)
- [ ] Calculate: Storage needed (assume 1GB per 1K users)
- [ ] Calculate: Bandwidth needed (assume 5KB per request)
- [ ] Design: Auto-scaling rules (triggers, min/max instances)
- [ ] Estimate: Cost projection (monthly, yearly)
- [ ] Design: Monitoring để track capacity usage
- [ ] Document: Complete capacity planning document

---

## Coding TODOs

### Task 1: Stateless Spring Boot Application

- [ ] Tạo Spring Boot project: Stateless Payment API
- [ ] Implement: REST API endpoints (no session state)
- [ ] Implement: JWT-based authentication (stateless)
- [ ] Verify: No in-memory session storage
- [ ] Verify: No server-side session state
- [ ] Test: Deploy 2 instances, verify both can handle same user
- [ ] Test: Kill one instance, verify other instance continues working
- [ ] Document: Stateless design decisions

### Task 2: Load Balancer Implementation

- [ ] Implement: Simple load balancer trong Java
- [ ] Algorithm: Round-robin
- [ ] Algorithm: Weighted round-robin (add weights)
- [ ] Algorithm: Least connections (track connection count)
- [ ] Add: Health check mechanism (ping backend)
- [ ] Add: Remove unhealthy backends automatically
- [ ] Add: Add back healthy backends automatically
- [ ] Test: With 3 backend servers
- [ ] Test: Mark one server unhealthy, verify traffic stops
- [ ] Test: Mark server healthy again, verify traffic resumes
- [ ] Measure: Load balancer overhead (latency added)
- [ ] Document: Code và test results

### Task 3: Health Check Implementation

- [ ] Implement: `/health` endpoint (basic)
- [ ] Implement: `/health/readiness` endpoint
- [ ] Implement: `/health/liveness` endpoint
- [ ] Add: Database connection check trong readiness
- [ ] Add: External service check trong readiness
- [ ] Add: Memory check trong liveness (fail if > 90%)
- [ ] Add: Disk space check trong liveness (fail if > 90%)
- [ ] Add: Thread pool check (fail if all threads busy)
- [ ] Test: All health endpoints
- [ ] Test: Simulate DB down, verify readiness fails
- [ ] Test: Simulate high memory, verify liveness fails
- [ ] Integrate: Health checks với load balancer
- [ ] Document: Health check strategy

### Task 4: Circuit Breaker với Resilience4j

- [ ] Add: Resilience4j dependency
- [ ] Implement: Circuit breaker cho external API call
- [ ] Configure: Failure threshold (50% failures)
- [ ] Configure: Wait duration (60 seconds)
- [ ] Configure: Half-open state (allow 3 requests)
- [ ] Add: Fallback mechanism
- [ ] Test: Normal operation (circuit closed)
- [ ] Test: Simulate failures, verify circuit opens
- [ ] Test: Verify fallback được called khi circuit open
- [ ] Test: Wait duration, verify circuit goes half-open
- [ ] Test: Successful requests in half-open, verify circuit closes
- [ ] Monitor: Circuit breaker metrics
- [ ] Document: Circuit breaker configuration và behavior

### Task 5: Retry với Exponential Backoff

- [ ] Implement: Retry mechanism cho failed requests
- [ ] Algorithm: Exponential backoff (1s, 2s, 4s, 8s)
- [ ] Add: Max retry attempts (3)
- [ ] Add: Jitter (random delay to prevent thundering herd)
- [ ] Test: Transient failure, verify retry works
- [ ] Test: Permanent failure, verify stops after max retries
- [ ] Test: Success after retry, verify no more retries
- [ ] Measure: Total time với retries
- [ ] Integrate: Retry với circuit breaker
- [ ] Document: Retry strategy

### Task 6: Multi-Instance Deployment

- [ ] Build: Spring Boot application JAR
- [ ] Deploy: 2 instances trên local (different ports)
- [ ] Setup: Nginx như load balancer (hoặc use Spring Cloud Gateway)
- [ ] Configure: Round-robin load balancing
- [ ] Configure: Health checks
- [ ] Test: Send 100 requests, verify distributed across instances
- [ ] Test: Kill one instance, verify traffic goes to other
- [ ] Test: Restart killed instance, verify traffic resumes
- [ ] Monitor: Request distribution (logs)
- [ ] Document: Deployment setup

### Task 7: State Externalization (Redis)

- [ ] Setup: Redis server
- [ ] Implement: Session storage trong Redis (thay vì in-memory)
- [ ] Implement: Cache trong Redis (thay vì local cache)
- [ ] Test: Deploy 2 instances, verify shared state (Redis)
- [ ] Test: User login on instance 1, verify session available on instance 2
- [ ] Test: Kill instance 1, verify user still logged in on instance 2
- [ ] Measure: Redis latency impact
- [ ] Document: State externalization strategy

### Task 8: Graceful Shutdown

- [ ] Implement: Graceful shutdown trong Spring Boot
- [ ] Add: Shutdown hook để stop accepting new requests
- [ ] Add: Wait for in-flight requests to complete
- [ ] Add: Timeout (30 seconds max wait)
- [ ] Test: Send requests, then shutdown app
- [ ] Verify: In-flight requests complete before shutdown
- [ ] Verify: New requests rejected during shutdown
- [ ] Test: With load balancer (remove from pool during shutdown)
- [ ] Document: Graceful shutdown implementation

---

## Failure & Resilience TODOs

### Failure Simulation 1: Server Failure

- [ ] Setup: 3 application instances behind load balancer
- Gợi ý setup: 3 instance (A/B/C) + 1 LB (Nginx/HAProxy/ALB). Mỗi instance expose `/health` + log `instance_id`.
- [ ] Test: Normal operation (all healthy)
- Kỳ vọng: phân phối traffic gần đều (RR) hoặc theo thuật toán đã chọn; error rate ~0; P95 ổn định.
- [ ] Simulate: Kill one instance abruptly
- Cách làm: kill process/container của instance B ngay lập tức (không graceful).
- [ ] Measure: Time for load balancer to detect failure
- Đo: từ lúc kill đến lúc health-check chuyển `fail`. Phụ thuộc interval/timeout/rise/fall (ví dụ 5s interval, 2 fails ⇒ ~10–15s).
- [ ] Measure: Time for traffic to stop going to dead instance
- Đo: từ lúc kill đến lúc LB không còn route vào B (quan sát logs/LB stats).
- [ ] Measure: Impact on users (errors, latency)
- Kỳ vọng: có thể có một spike lỗi ngắn (trong cửa sổ detect), sau đó về bình thường; latency có thể tăng nhẹ do retry.
- [ ] Verify: Other instances handle increased load
- Kỳ vọng: A/C CPU tăng, nhưng vẫn nằm dưới threshold SLO.
- [ ] Restore: Dead instance
- Khởi động lại B; đảm bảo warm-up/readiness pass trước khi nhận traffic.
- [ ] Measure: Time for instance to rejoin pool
- Đo: từ lúc start đến khi LB đưa B vào lại (rise count + readiness).
- [ ] Document: Failure scenario và recovery
- Ghi: cấu hình health-check, timeline, số request lỗi, lesson learned (tuning interval, retries, graceful shutdown).

### Failure Simulation 2: Database Failure

- [ ] Setup: Application với database
- Setup tối thiểu: app có endpoint truy DB thật (SELECT 1 / query đơn giản) + readiness check kiểm DB connection.
- [ ] Test: Normal operation
- Kỳ vọng: readiness OK; error rate thấp; DB connections ổn định.
- [ ] Simulate: Database connection failure
- Cách làm: tắt DB, drop network (iptables/security group), hoặc đổi credential để fail fast.
- [ ] Verify: Readiness check fails
- Kỳ vọng: `/health/readiness` fail nhanh (timeout ngắn 1–2s).
- [ ] Verify: Load balancer removes instance from pool
- Kỳ vọng: instance bị loại khỏi upstream/target group; traffic không vào instance “đã mất DB”.
- [ ] Verify: Circuit breaker opens (nếu có)
- Kỳ vọng: sau một số lỗi, CB open để bảo vệ DB và giảm latency cascade.
- [ ] Restore: Database connection
- Bật DB lại/khôi phục network.
- [ ] Verify: Readiness check passes
- Kỳ vọng: readiness OK sau khi DB ổn định.
- [ ] Verify: Instance rejoins pool
- Kỳ vọng: LB add lại instance theo rise count.
- [ ] Measure: Total downtime
- Đo: thời gian user gặp lỗi (nếu có) và thời gian hệ tự hồi phục.
- [ ] Document: Database failure handling
- Ghi: timeout, retry policy, CB config, cách tránh “thundering herd” khi DB vừa lên.

### Failure Simulation 3: Partial Failure

- [ ] Setup: Multi-instance application
- Ít nhất 2–3 instances + LB + metrics.
- [ ] Simulate: One instance slow (add artificial delay)
- Cách làm: thêm delay 500ms–2s vào 1 instance (B).
- [ ] Verify: Load balancer behavior (least connections?)
- Quan sát: RR vẫn gửi đều; LeastConn/LeastRT (nếu có) sẽ tránh B theo thời gian.
- [ ] Measure: Impact on overall latency
- Kỳ vọng: nếu RR, P95/P99 tăng; nếu least-response-time, impact thấp hơn.
- [ ] Simulate: One instance high error rate (50%)
- Cách làm: inject lỗi 5xx ngẫu nhiên trên B.
- [ ] Verify: Circuit breaker behavior
- Kỳ vọng: CB ở client/app tier giảm số call vào B (nếu B là downstream) hoặc giảm call downstream nếu downstream lỗi.
- [ ] Verify: Load balancer removes unhealthy instance
- Nếu health-check dựa trên error rate/active check: B bị loại; nếu chỉ TCP health-check thì có thể vẫn giữ ⇒ cần cấu hình health-check đúng.
- [ ] Document: Partial failure handling
- Ghi: chọn thuật toán LB, health-check kiểu gì (active HTTP check vs passive), ngưỡng loại instance.

### Failure Simulation 4: Network Partition

- [ ] Setup: 2 instances, shared Redis
- Redis dùng cho session/cache/presence; cả 2 instance đọc/ghi Redis.
- [ ] Simulate: Network partition (instance 1 can't reach Redis)
- Chặn network từ instance1 → Redis.
- [ ] Verify: Instance 1 behavior (fail fast? retry?)
- Best practice: fail fast + limited retry + degrade (nếu có) để tránh treo thread.
- [ ] Verify: Instance 2 continues working
- Kỳ vọng: instance2 vẫn OK vì còn access Redis.
- [ ] Verify: Load balancer removes instance 1
- Nếu readiness check include Redis dependency: instance1 readiness fail ⇒ LB loại.
- [ ] Restore: Network connectivity
- Mở lại network.
- [ ] Verify: Instance 1 recovers
- Kỳ vọng: readiness pass; LB add lại sau rise.
- [ ] Document: Network partition handling
- Ghi: dependency checks nào đưa vào readiness, timeout/retry/jitter, bài học về cascading failure.

### Resilience Testing

- [ ] Implement: Chaos testing scenario
- Kịch bản đơn giản: mỗi 30s kill ngẫu nhiên 1 instance; hoặc inject latency/error; giữ load ổn định.
- [ ] Test: Random instance failures (kill random instance every 30s)
- Chạy 10–30 phút để thấy xu hướng, không chỉ 1–2 lần.
- [ ] Measure: System behavior under chaos
- Track: error rate, P95/P99 latency, recovery time, autoscaling events.
- [ ] Verify: System continues serving requests
- Target: không vượt SLO/SLA quá nhiều; không “global outage”.
- [ ] Measure: Error rate during chaos
- Ghi theo thời gian; correlate với failover detection windows.
- [ ] Measure: Latency during chaos
- Quan sát tail latency; latency spike thường do retry/connection churn.
- [ ] Document: Resilience test results
- Tóm tắt: timeline, charts, bottlenecks, config cần tune.
- [ ] Identify: Weak points trong system
- Thường gặp: DB/Redis SPOF, timeout quá dài, retry không jitter, health-check sai.
- [ ] Propose: Improvements để handle chaos better
- Timeout budgets, retry w/ jitter, CB, bulkheads, better readiness, autoscaling thresholds, multi-AZ.

---

## Analysis TODOs

### Analysis Task 1: Scaling Strategy Decision

- [ ] Scenario: Current system handles 1K QPS, need to handle 10K QPS
- Goal: tăng 10x; ưu tiên horizontal + stateless.
- [ ] Option 1: Vertical scaling (bigger server)
- Ví dụ: tăng CPU/RAM 8–16x; có lợi nếu bottleneck compute đơn giản.
- [ ] Calculate: Cost của vertical scaling
- Quy tắc: máy càng lớn chi phí tăng không tuyến tính; thêm risk downtime khi resize (tuỳ platform).
- [ ] Identify: Limitations (max server size)
- Giới hạn: max instance type, NUMA/GC, single disk IO ceiling, single network interface.
- [ ] Identify: Single point of failure risk
- 1 máy chết = outage (trừ khi có HA/failover) ⇒ SPOF.
- [ ] Option 2: Horizontal scaling (more servers)
- Ví dụ: từ 2 instances lên 20 instances (tuỳ QPS/instance).
- [ ] Calculate: Cost của horizontal scaling
- Cost = N * instance + LB + ops; thường tuyến tính hơn; có thể tối ưu spot/reserved.
- [ ] Identify: Complexity added
- LB, statelessness, shared state, distributed tracing, deploy orchestration.
- [ ] Identify: Load balancer requirement
- Cần L4/L7 LB + health checks + connection draining.
- [ ] Compare: Vertical vs Horizontal (cost, complexity, risk, scalability)
- Kết luận chung: 10x thường nên **horizontal**; vertical chỉ là bước đầu/đệm.
- [ ] Recommend: Which strategy? Justify với numbers
- Mẫu: nếu 1 instance xử lý 500 QPS an toàn, 10K QPS cần 20 instances + 20% headroom ⇒ 24 instances.
- [ ] Document: Decision analysis (500 words)
- Nêu bottleneck, giả định, công thức, trade-offs.

### Analysis Task 2: Load Balancer Algorithm Selection

- [ ] Scenario: Payment API, different servers have different capacities
- Server có CPU khác nhau hoặc autoscaling mixed types.
- [ ] Analyze: Round-robin suitability
- RR tốt khi backend đồng nhất + request cost gần giống nhau; không tốt khi heterogenous.
- [ ] Analyze: Weighted round-robin suitability
- WRR phù hợp heterogenous: weight theo capacity (CPU, QPS benchmark).
- [ ] Analyze: Least connections suitability
- Hợp với request thời gian xử lý khác nhau/connection dài; lưu ý keep-alive có thể làm lệch.
- [ ] Analyze: IP hash suitability (sticky sessions needed?)
- IP hash dùng cho session affinity; payment API nên stateless ⇒ thường **không cần** sticky.
- [ ] Test: Each algorithm với realistic load
- Dùng k6/JMeter; mix endpoint nặng/nhẹ.
- [ ] Measure: Load distribution (even? uneven?)
- So sánh RPS/instance, CPU, queue time.
- [ ] Measure: Latency impact
- Track P95/P99 theo thuật toán.
- [ ] Recommend: Best algorithm cho scenario
- Gợi ý: **Weighted RR** (capacity-aware) + health checks; nếu workload biến thiên mạnh, cân nhắc least-time/latency-aware (tuỳ LB).
- [ ] Justify: Recommendation với data
- Chốt bằng số liệu từ test.
- [ ] Document: Algorithm selection analysis
- Ghi assumptions, graphs, config.

### Analysis Task 3: Stateless vs Stateful Trade-off

- [ ] Scenario: E-commerce cart system
- Cart là state theo user, cần tồn tại qua nhiều request.
- [ ] Option 1: Stateless (cart in Redis)
- Pattern: app stateless; cart stored Redis (key=userId), TTL + persistence (AOF/RDB) tuỳ yêu cầu.
- [ ] Analyze: Pros (3 points)
    1. Scale app dễ, không cần sticky session.
    2. HA tốt hơn (instance chết không mất cart).
    3. Deploy/failover đơn giản hơn.
- [ ] Analyze: Cons (3 points)
    1. Redis trở thành dependency quan trọng (cần HA).
    2. Thêm latency mạng + chi phí.
    3. Consistency/locking cần thiết nếu multi-write.
- [ ] Analyze: Complexity
- Cần schema key, TTL, idempotency, cache stampede control.
- [ ] Option 2: Stateful (cart in server memory)
- Pattern: sticky sessions hoặc consistent routing theo user.
- [ ] Analyze: Pros (3 points)
    1. Latency cực thấp (local memory).
    2. Đơn giản khi chỉ 1 instance.
    3. Giảm phụ thuộc external store.
- [ ] Analyze: Cons (3 points)
    1. Mất cart khi instance chết/restart.
    2. Scale khó, cần sticky, mất cân bằng tải.
    3. Rolling deploy/auto-scale phức tạp.
- [ ] Analyze: Scalability limitations
- Trần scale theo memory/instance; khó redistribute cart state khi scale.
- [ ] Compare: Stateless vs Stateful
- Production phổ biến: **stateless app + Redis/DB** (state external).
- [ ] Recommend: Which approach? Justify
- Khuyến nghị: stateless + Redis; nếu cần durability cao, ghi DB (event sourcing hoặc periodic snapshot).
- [ ] Document: Trade-off analysis (500 words)
- Nêu RPO/RTO, cost, operability.

### Analysis Task 4: Availability Calculation

- [ ] System có 5 components: Load Balancer (99.9%), App Server (99.9%), Database (99.95%), Cache (99.9%), External
  API (99%)
- [ ] Calculate: Overall availability (all in series)
    - Series availability: nhân các availability:
      \[
      A = 0.999 \times 0.999 \times 0.9995 \times 0.999 \times 0.99 \approx 0.9865
      \]
      ⇒ **~98.65%** (downtime ~ 4.9 ngày/năm).
- [ ] Scenario: App Servers có 3 instances (parallel, each 99.9%)
- [ ] Calculate: App Server cluster availability
    - Parallel (ít nhất 1 sống): \(A_{cluster} = 1-(1-0.999)^3 = 1-10^{-9} = 0.999999999\) (~ “9 số 9”).
- [ ] Scenario: Database có Master-Slave (active-passive)
- [ ] Calculate: Database availability
    - Nếu giả định độc lập và failover “hoàn hảo”: \(A \approx 1-(1-0.9995)^2 = 0.99999975\).
    - Thực tế thấp hơn vì failover không hoàn hảo + replication lag (RPO) + operator errors.
- [ ] Recalculate: Overall system availability với redundancy
    - Thay app cluster và DB HA vào:
      \(A \approx 0.999(LB)\times 0.999999999(app)\times 0.99999975(DB)\times 0.999(cache)\times 0.99(ext)\)
      \(\approx 0.9880\) ⇒ vẫn bị kéo xuống bởi **External API 99%**.
- [ ] Analyze: Which component is weakest link?
    - Weakest link: **External API 99%** (và các dependency “series” khác).
- [ ] Propose: How to improve weakest component?
    - Thêm cache/fallback, degrade mode, async queue, multi-provider, bulkheads + timeouts.
- [ ] Calculate: New overall availability after improvement
    - Ví dụ: thay External API bằng 99.9% hiệu dụng nhờ cache/fallback:
      tổng availability tăng xấp xỉ theo tỉ lệ (0.999/0.99) ≈ 1.009 ⇒ cải thiện đáng kể.
- [ ] Document: Availability analysis và improvements
    - Nêu rõ giả định (independence, failover time), và cách đo thật (SLO-based).

### Analysis Task 5: Capacity Planning

- [ ] Current: 1K QPS, 2 servers (500 QPS each)
- [ ] Growth: 50% per month
- [ ] Calculate: QPS after 3 months
    - \(QPS_3 = 1000 \times 1.5^3 = 3375\) QPS.
- [ ] Calculate: QPS after 6 months
    - \(QPS_6 = 1000 \times 1.5^6 \approx 11390.6\) QPS.
- [ ] Calculate: QPS after 12 months
    - \(QPS_{12} = 1000 \times 1.5^{12} \approx 129746\) QPS.
- [ ] Plan: When to add servers? (trigger points)
    - Trigger theo SLO: CPU > 65% 10 phút, P95 > SLO, queue lag tăng; scale trước peak.
- [ ] Calculate: Server count needed each month
    - Nếu 1 server an toàn 500 QPS: cần \(N = \lceil QPS/500 \rceil\). Với 20% headroom: \(N = \lceil 1.2\times QPS/500 \rceil\).
- [ ] Calculate: Cost projection (monthly)
    - Cost ≈ N * instance_cost + LB + DB/Redis + logs/metrics; tách fixed vs variable.
- [ ] Design: Auto-scaling rules (min, max, scale-up threshold, scale-down threshold)
    - Mẫu: min=2, max=40; scale up khi CPU>60% 5 phút hoặc P95> SLO; scale down khi CPU<30% 15 phút.
- [ ] Estimate: Headroom needed (20% buffer)
    - Headroom 20% cho web bình thường; payment peak/flash sale nên 30–50% + pre-scale.
- [ ] Document: Capacity planning và scaling plan
    - Include: assumptions, peak factor, bottlenecks, test plan.

### Analysis Task 6: Deployment Strategy Comparison

- [ ] Compare: Blue-Green vs Canary vs Rolling deployment
- [ ] Criteria: Downtime, Risk, Cost, Complexity, Rollback speed
- [ ] Score: Each strategy on each criteria (1-10)
    - Tham khảo (tuỳ hệ):
        - Blue-Green: Downtime 9, Risk 6, Cost 4, Complexity 6, Rollback 9
        - Canary: Downtime 9, Risk 8, Cost 6, Complexity 7, Rollback 8
        - Rolling: Downtime 7, Risk 6, Cost 9, Complexity 5, Rollback 6
- [ ] Analyze: Best use case cho Blue-Green
    - Khi cần rollback tức thì, release ít nhưng “big”, có thể chạy 2 môi trường.
- [ ] Analyze: Best use case cho Canary
    - Khi hệ critical, cần giảm blast radius, có metrics tốt, muốn progressive delivery.
- [ ] Analyze: Best use case cho Rolling
    - Khi cost-sensitive, release thường xuyên, backward-compatible tốt.
- [ ] Scenario: Critical payment system
- [ ] Recommend: Deployment strategy
    - Khuyến nghị: **Canary** + feature flags + strict SLO monitoring; một số thay đổi lớn dùng blue-green.
- [ ] Justify: Recommendation
    - Payment cần giảm rủi ro, rollback nhanh, kiểm soát theo cohort.
- [ ] Document: Deployment strategy analysis
    - Nêu pipeline, gating metrics, rollback procedure.

---

## Review TODOs

### Self-Evaluation

- [ ] Review: Tất cả Study TODOs hoàn thành chưa?
- Checklist: đọc xong + có ghi chép + có ví dụ/mini-lab cho từng phần.
- [ ] Review: Tất cả Design exercises hoàn thành chưa?
- (Không điền design ở đây theo yêu cầu của bạn) Chỉ nhắc: phải có diagram + trade-offs + assumptions.
- [ ] Review: Tất cả Coding tasks đã code và test chưa?
- (Không điền coding ở đây theo yêu cầu của bạn) Nhắc: có logs chứng minh distribution/failover.
- [ ] Review: Tất cả Failure simulations đã run chưa?
- Đảm bảo có timeline + số liệu error/latency + config health-check.
- [ ] Review: Tất cả Analysis tasks đã complete chưa?
- Các phép tính/công thức phải rõ giả định (independence, headroom, per-server capacity).
- [ ] Rate: Understanding của horizontal scaling (1-10)
- Gợi ý tự chấm: 7+ nếu bạn tự tính được server count + chỉ ra bottleneck + nói được trade-off.
- [ ] Rate: Understanding của load balancing (1-10)
- 7+ nếu bạn giải thích được L4 vs L7, sticky sessions, health-check, thuật toán.
- [ ] Rate: Understanding của stateless design (1-10)
- 7+ nếu bạn biết externalize state, thiết kế JWT/session đúng, và xử lý WebSocket state.
- [ ] Rate: Understanding của high availability (1-10)
- 7+ nếu bạn tính availability, biết RTO/RPO, và loại SPOF.
- [ ] Rate: Practical skills (implementation) (1-10)
- 7+ nếu bạn tự dựng 2–3 instances + LB + health-check + chaos test cơ bản.
- [ ] Identify: 3 concepts bạn master
- Mẫu: (1) L4 vs L7 LB, (2) stateless & externalize state, (3) health-check/readiness/liveness.
- [ ] Identify: 3 concepts cần improve
- Mẫu: (1) DB failover tooling, (2) SLO/SLI & observability, (3) canary + feature flags + safe migrations.
- [ ] Plan: How to improve 3 weak areas
- Mẫu plan 1 tuần: đọc 2 bài + làm 1 lab + viết 1 trang recap cho mỗi weak area.

### Architecture Review

- [ ] Review: Payment Gateway design
- Focus: SLA/SLO, idempotency, retries/timeouts, rate limiting, DB bottlenecks.
- [ ] Check: Horizontal scaling strategy có realistic không?
- Có dựa trên per-instance benchmark? có headroom? có autoscaling triggers?
- [ ] Check: Load balancer configuration đúng chưa?
- L4/L7 đúng chưa, health-check đúng endpoint chưa, connection draining, timeouts.
- [ ] Check: Stateless design đúng chưa? (no session state?)
- Session/cached state có nằm local không? có Redis/DB shared không?
- [ ] Check: SPOFs eliminated chưa?
- LB/DB/cache/queue có HA chưa, secrets/config có replicated chưa?
- [ ] Identify: 3 weaknesses trong design
- Mẫu: External API dependency, DB single-writer bottleneck, thiếu backpressure.
- [ ] Propose: Improvements cho weaknesses
- Mẫu: cache+fallback, CQRS/read replicas, queue + rate limit + circuit breaker.
- [ ] Compare: Design vs industry best practices
- So với: stateless app tier, managed LB, safe deploy, observability, DR.
- [ ] Document: Architecture review findings
- 1–2 trang: vấn đề → impact → fix → trade-off.

### Code Review

- [ ] Review: Stateless application code
- Check: không dùng HttpSession/in-memory state quyết định response.
- [ ] Check: No session state trong code?
- Nếu có login/session: phải externalize (JWT/Redis), không pin vào instance.
- [ ] Check: JWT implementation correct?
- Verify: expiry, issuer/audience, signature alg, key rotation, refresh token, revoke strategy (nếu cần).
- [ ] Review: Load balancer implementation
- Check: thread-safety, health-check scheduling, remove/add backend đúng.
- [ ] Check: Algorithm implementation correct?
- Test cases: RR wrap-around, WRR weights, least-conn accuracy.
- [ ] Check: Health check logic correct?
- Active check + thresholds; tránh flapping (rise/fall); timeout nhỏ.
- [ ] Review: Circuit breaker implementation
- Check: failure threshold, half-open probes, metrics.
- [ ] Check: Configuration correct?
- Timeouts < CB window; retry không “đánh” vào CB sai cách.
- [ ] Check: Fallback mechanism works?
- Fallback không làm sai business (payment thường không được “fake success”).
- [ ] Identify: 3 code improvements
- Mẫu: better timeouts, structured logs + traceId, idempotency key handling.
- [ ] Refactor: At least 1 piece of code
- Mẫu: tách LB policy khỏi health-check scheduler; thêm interfaces/testability.
- [ ] Document: Code review findings
- Link commit + notes + test evidence.

### Performance Review

- [ ] Review: Load test results
- Nhìn P95/P99, saturation, error rate, throughput; đừng chỉ nhìn average.
- [ ] Review: Failure simulation results
- So sánh trước/sau tuning health-check/timeout/retry.
- [ ] Measure: System performance under normal load
- Baseline: QPS at P95<SLO, CPU<70%, DB<80% capacity.
- [ ] Measure: System performance under failure scenarios
- Track: error spike window, recovery time, tail latency.
- [ ] Identify: Performance bottlenecks
- Thường gặp: DB locks, connection pool, GC, synchronous downstream calls.
- [ ] Verify: Scaling works as expected?
- Khi tăng instances 2x, throughput có tăng gần 2x không? nếu không, bottleneck ở đâu.
- [ ] Document: Performance analysis
- Ghi: graphs + root cause + actions.
- [ ] Create: Performance improvement plan
- 3 items: quick wins (timeouts/cache), mid (DB index/read replicas), long (sharding/async).

### Knowledge Check

- [ ] Explain: Horizontal vs Vertical scaling (viết 1 paragraph, không xem notes)
- Sample (ngắn): Vertical = scale up 1 máy (CPU/RAM), nhanh nhưng giới hạn & SPOF; Horizontal = scale out nhiều máy, cần LB + statelessness, phức tạp hơn nhưng HA/scalability tốt.
- [ ] Explain: Load balancer algorithms (viết comparison, không xem notes)
- Sample: RR (đơn giản, backend đồng nhất), WRR (backend khác capacity), LeastConn (workload biến thiên/connection dài), IP hash (sticky), LeastRT (tối ưu latency nhưng cần measurement).
- [ ] Explain: Stateless vs Stateful (viết 1 paragraph, không xem notes)
- Sample: Stateless không giữ session local; state externalize → dễ scale/HA. Stateful giữ state local → cần sticky/replication, deploy/failover khó.
- [ ] Explain: Active-Active vs Active-Passive (viết comparison, không xem notes)
- Sample: Active-Active phục vụ song song → failover nhanh nhưng phức tạp consistency; Active-Passive đơn giản hơn nhưng failover có RTO và lãng phí.
- [ ] Explain: Zero-downtime deployment strategies (viết 1 paragraph, không xem notes)
- Sample: Blue-green switch môi trường, rollback nhanh; Canary shift dần traffic giảm risk; Rolling thay dần instances yêu cầu backward-compatible.
- [ ] Solve: System có 3 components (99%, 99.9%, 99.99%), 2 app servers parallel (99.9% each), tính overall availability
    - App parallel: \(A_{app}=1-(1-0.999)^2 = 0.999999\).
    - Overall series: \(A = 0.99 \times 0.9999 \times 0.9999 \times 0.999999 \approx 0.9897\) (**~98.97%**).
- [ ] Solve: Current 1K QPS, growth 30%/month, cần capacity sau 6 tháng? (với 20% headroom)
    - \(QPS_6 = 1000 \times 1.3^6 \approx 4827\) QPS.
    - Headroom 20%: \(4827 \times 1.2 \approx 5792\) QPS capacity.
- [ ] Verify: Answers có đúng không?
- Check lại công thức series/parallel + làm tròn; ghi giả định independence.
- [ ] Document: Knowledge gaps
- Ghi 5 bullet: mình chưa chắc chỗ nào + plan học bù.

### Reflection

- [ ] Write: 3 điều học được quan trọng nhất tuần này
- Mẫu: (1) LB là “cốt lõi” của HA, (2) stateless + external state là chìa khoá scale, (3) timeout/retry/CB quyết định stability.
- [ ] Write: 2 concepts còn confuse
- Mẫu: (1) DB failover tooling & split-brain, (2) safe DB migrations cho zero-downtime.
- [ ] Write: 1 mistake đã làm và lesson learned
- Mẫu: retry không jitter gây spike; lesson: retry budget + jitter + CB.
- [ ] Write: Confidence level cho Week 3 (1-10)
- Gợi ý: 7 nếu bạn tự thiết kế DB schema + index + replication basics.
- [ ] Compare: Week 1 vs Week 2 progress
- Week1 fundamentals; Week2 thực chiến HA/scale (LB + stateless + deploy).
- [ ] Plan: Preparation cho Week 3 (Database Design)
- Plan 5 ngày: modeling + indexing + transactions + replication + caching patterns.
- [ ] Set: Goals cho Week 3
- Goals: thiết kế schema + query patterns + scaling DB (read replicas/sharding intro).
- [ ] Document: Week 2 reflection (500 words)
- Bố cục: what learned → what failed → what next → metrics (labs done).

### Mentor Questions (Answer these - be honest)

- [ ] Q1: Bạn có thể scale một stateful service horizontally không? Tại sao? (viết analysis)
- [ ] Q2: Load balancer với sticky sessions - có phải stateless không? Giải thích. (viết answer)
- [ ] Q3: Active-Active setup có eliminate SPOF không? Tại sao? (viết analysis)
- [ ] Q4: Zero-downtime deployment có nghĩa là zero risk không? Giải thích. (viết answer)
- [ ] Q5: Nếu bạn có 99.9% availability nhưng vẫn bị complain, có thể do gì? (viết analysis)
- [ ] Q6: Horizontal scaling có giới hạn không? Giới hạn là gì? (viết answer)
- [ ] Review: Answers có đủ depth và critical thinking chưa?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Design TODOs: ✅ Complete với diagrams
- [ ] Tất cả Coding TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Failure & Resilience TODOs: ✅ Complete với results
- [ ] Tất cả Analysis TODOs: ✅ Complete với documentation
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] Design diagrams saved: ✅ Yes
- [ ] Ready for Week 3: ✅ Yes

---

> **Mentor Final Check**: Tuần 2 này bạn phải MASTER scaling và high availability. Nếu bạn chỉ hiểu lý thuyết mà không
> implement được, bạn chưa sẵn sàng. Hãy honest: Bạn có thể design và implement một hệ thống scalable, highly available
> chưa? Nếu chưa, làm lại.
