# Week 6 – Load Balancing & API Design

> **Mentor Note**: API design và load balancing là foundation của mọi distributed system. Một API design sai sẽ cost bạn years of technical debt. Một load balancing config sai sẽ cause cascading failures. Bạn phải think như principal engineer - challenge mọi assumption, question mọi decision. Không có "it works" - phải có "it works correctly under all conditions". Production APIs need to be robust, scalable, và maintainable.

---

## Study TODOs

### Load Balancing Fundamentals
- [x] Đọc "Designing Data-Intensive Applications" Chapter 5 - Replication (load balancing sections)
    - **Key takeaways (DDIA Ch5 – Load Balancing):**
        - Load balancing là kỹ thuật phân phối traffic across multiple servers để improve performance, availability, và reliability.
        - **Stateless load balancing**: mỗi request độc lập, không cần track state → dễ scale, nhưng không hỗ trợ sticky sessions.
        - **Stateful load balancing**: track connection state → hỗ trợ sticky sessions, nhưng phức tạp hơn và có overhead.
        - **Health checks**: critical để detect và remove unhealthy backends, tránh traffic đến dead servers.
        - **Load balancing algorithms**: Round-Robin, Least Connections, IP Hash, Weighted - mỗi cái có trade-offs.
        - **Insight**: Load balancing không chỉ là distribute traffic, mà còn là resilience mechanism - khi 1 server fail, traffic tự động chuyển sang servers khác.
- [x] Định nghĩa chính xác: Load balancing
    - **Load balancing** là kỹ thuật phân phối incoming network traffic (requests) across multiple backend servers để:
        - ✅ Tăng **throughput** (xử lý nhiều requests hơn)
        - ✅ Giảm **latency** (phân tải giúp mỗi server xử lý ít hơn)
        - ✅ Tăng **availability** (nếu 1 server fail, traffic chuyển sang servers khác)
        - ✅ Tăng **scalability** (dễ dàng thêm/bớt servers)
    - Load balancer hoạt động như một **traffic director**, quyết định request nào đi đến server nào dựa trên algorithm (Round-Robin, Least Connections, etc.).
- [x] Định nghĩa chính xác: Reverse proxy
    - **Reverse proxy** là server đứng trước backend servers, nhận requests từ clients và forward đến backend servers.
    - **Khác với forward proxy** (đứng trước clients, forward requests ra internet):
        - Forward proxy: Client → Proxy → Internet
        - Reverse proxy: Internet → Proxy → Backend servers
    - **Chức năng của reverse proxy:**
        - ✅ **Load balancing**: phân phối traffic
        - ✅ **SSL termination**: xử lý HTTPS, backend chỉ cần HTTP
        - ✅ **Caching**: cache responses để giảm load backend
        - ✅ **Security**: ẩn backend servers, filter requests
        - ✅ **Compression**: compress responses
    - **Ví dụ**: Nginx, HAProxy có thể hoạt động như reverse proxy.
- [x] Định nghĩa chính xác: API Gateway
    - **API Gateway** là single entry point cho tất cả API requests, đứng trước multiple backend services (microservices).
    - **Chức năng chính:**
        - ✅ **Routing**: route requests đến đúng service dựa trên URL/path
        - ✅ **Authentication & Authorization**: validate tokens, check permissions
        - ✅ **Rate limiting**: giới hạn số requests per user/IP
        - ✅ **Request/Response transformation**: transform data format
        - ✅ **Protocol translation**: HTTP → gRPC, REST → GraphQL
        - ✅ **Monitoring & Logging**: track metrics, log requests
        - ✅ **Circuit breaker**: fail fast khi service down
    - **Khác với Load Balancer:**
        - Load balancer: chỉ phân phối traffic (L4 hoặc L7)
        - API Gateway: có business logic, auth, rate limiting, transformation (L7 only)
    - **Ví dụ**: Kong, AWS API Gateway, Spring Cloud Gateway, Zuul.
- [x] So sánh: Load balancer vs Reverse proxy vs API Gateway (5 differences)
    1. **Scope of functionality:**
        - **Load Balancer**: chỉ phân phối traffic, không có business logic
        - **Reverse Proxy**: có caching, SSL termination, nhưng không có auth/rate limiting
        - **API Gateway**: có đầy đủ: routing, auth, rate limiting, transformation, monitoring
    2. **Layer hoạt động:**
        - **Load Balancer**: có thể L4 (TCP/IP) hoặc L7 (HTTP)
        - **Reverse Proxy**: thường L7 (HTTP/HTTPS)
        - **API Gateway**: chỉ L7 (HTTP/HTTPS, có thể gRPC, WebSocket)
    3. **Use case:**
        - **Load Balancer**: distribute traffic cho cùng 1 service (multiple instances)
        - **Reverse Proxy**: đứng trước web servers, xử lý SSL/caching
        - **API Gateway**: đứng trước multiple microservices, xử lý cross-cutting concerns
    4. **Configuration complexity:**
        - **Load Balancer**: đơn giản (chỉ cần backend list + algorithm)
        - **Reverse Proxy**: trung bình (cần config SSL, caching rules)
        - **API Gateway**: phức tạp (cần config routes, auth policies, rate limits, transformations)
    5. **Business logic:**
        - **Load Balancer**: không có business logic
        - **Reverse Proxy**: không có business logic (chỉ infrastructure concerns)
        - **API Gateway**: có business logic (auth rules, rate limit policies, transformation logic)
- [x] Đọc về "Layer 4 (L4) load balancing" - how it works
    - **L4 Load Balancing** hoạt động ở **Transport Layer** (TCP/UDP), chỉ xem IP và port, không xem nội dung HTTP.
    - **How it works:**
        1. Client gửi TCP connection đến load balancer
        2. Load balancer chọn backend server dựa trên **source IP + port** (hoặc algorithm)
        3. Load balancer tạo connection đến backend server
        4. **Forward packets** giữa client và backend (NAT hoặc Direct Server Return)
    - **Characteristics:**
        - ✅ **Fast**: không cần parse HTTP, chỉ forward packets
        - ✅ **Low latency**: overhead thấp
        - ✅ **Protocol agnostic**: hoạt động với TCP/UDP (HTTP, HTTPS, gRPC, database connections)
        - ❌ **Không biết HTTP content**: không thể route dựa trên URL, headers
        - ❌ **Không thể SSL termination**: phải forward encrypted traffic
    - **Use cases:**
        - Database load balancing
        - High-throughput TCP services
        - Khi cần performance tối đa, không cần HTTP-aware routing
- [x] Đọc về "Layer 7 (L7) load balancing" - how it works
    - **L7 Load Balancing** hoạt động ở **Application Layer** (HTTP/HTTPS), xem nội dung HTTP requests.
    - **How it works:**
        1. Client gửi HTTP request đến load balancer
        2. Load balancer **parse HTTP request** (method, URL, headers, body)
        3. Load balancer chọn backend server dựa trên **HTTP content** (URL path, headers, cookies)
        4. Load balancer forward HTTP request đến backend
        5. Backend trả response, load balancer forward về client
    - **Characteristics:**
        - ✅ **HTTP-aware**: có thể route dựa trên URL, headers, cookies
        - ✅ **SSL termination**: decrypt HTTPS, backend chỉ cần HTTP
        - ✅ **Content-based routing**: route `/api/users` → user service, `/api/orders` → order service
        - ✅ **Sticky sessions**: dựa trên cookies
        - ❌ **Higher latency**: phải parse HTTP, overhead cao hơn L4
        - ❌ **More CPU intensive**: cần parse và process HTTP
    - **Use cases:**
        - API Gateway
        - Web applications
        - Microservices routing
        - Khi cần content-based routing
- [x] So sánh: L4 vs L7 load balancing (10 criteria)
    1. **OSI Layer:**
        - L4: Transport Layer (TCP/UDP)
        - L7: Application Layer (HTTP/HTTPS)
    2. **Information used:**
        - L4: IP address, port number
        - L7: HTTP method, URL, headers, cookies, body
    3. **Routing capability:**
        - L4: chỉ dựa trên IP/port, không thể route theo URL
        - L7: có thể route theo URL path, headers, cookies
    4. **Performance:**
        - L4: nhanh hơn (không parse HTTP)
        - L7: chậm hơn (phải parse HTTP)
    5. **SSL/TLS handling:**
        - L4: không thể SSL termination, phải forward encrypted traffic
        - L7: có thể SSL termination, decrypt ở load balancer
    6. **Protocol support:**
        - L4: bất kỳ TCP/UDP protocol (HTTP, database, custom)
        - L7: chỉ HTTP/HTTPS (và protocols trên HTTP như gRPC-Web)
    7. **Sticky sessions:**
        - L4: có thể dựa trên IP hash
        - L7: có thể dựa trên cookies, headers
    8. **Health checks:**
        - L4: chỉ check TCP connection
        - L7: có thể check HTTP endpoint (`/health`), validate response
    9. **Use cases:**
        - L4: database load balancing, high-throughput TCP services
        - L7: web applications, API Gateway, microservices
    10. **Cost:**
        - L4: rẻ hơn (ít CPU hơn)
        - L7: đắt hơn (nhiều CPU hơn, cần hardware mạnh hơn)
- [x] Analyze: When to use L4? When to use L7? (5 use cases each)
    - **Use L4 khi:**
        1. **Database load balancing**: cần forward TCP connections, không cần HTTP routing
        2. **High-throughput TCP services**: cần performance tối đa, không cần HTTP-aware
        3. **Protocol-agnostic services**: services không dùng HTTP (custom TCP protocols)
        4. **Simple load distribution**: chỉ cần distribute traffic đều, không cần content-based routing
        5. **Cost-sensitive**: muốn giảm CPU overhead, tiết kiệm chi phí
    - **Use L7 khi:**
        1. **API Gateway**: cần route dựa trên URL path (`/api/users` → user service)
        2. **Web applications**: cần SSL termination, content-based routing
        3. **Microservices**: cần route requests đến đúng service dựa trên path/headers
        4. **Sticky sessions với cookies**: cần session affinity dựa trên HTTP cookies
        5. **Advanced routing**: cần route dựa trên headers, cookies, query parameters
- [x] Research: Popular load balancers (Nginx, HAProxy, AWS ALB, F5)
    - **Nginx:**
        - ✅ Open-source, free
        - ✅ Hỗ trợ cả L4 và L7
        - ✅ High performance, low memory footprint
        - ✅ Có thể làm reverse proxy, load balancer, web server
        - ❌ Configuration phức tạp hơn HAProxy
        - **Use case**: web applications, API Gateway, reverse proxy
    - **HAProxy:**
        - ✅ Open-source, free
        - ✅ Chuyên về load balancing, rất mạnh
        - ✅ Hỗ trợ cả L4 và L7
        - ✅ Health checks tốt, nhiều algorithms
        - ✅ Configuration linh hoạt
        - **Use case**: high-performance load balancing, TCP/HTTP services
    - **AWS ALB (Application Load Balancer):**
        - ✅ Managed service, không cần maintain
        - ✅ L7 load balancing, content-based routing
        - ✅ Tích hợp với AWS services (ECS, EC2, Lambda)
        - ✅ Auto-scaling, health checks tự động
        - ❌ Vendor lock-in, chi phí theo usage
        - **Use case**: AWS-based applications, microservices trên AWS
    - **F5 BIG-IP:**
        - ✅ Enterprise-grade, rất mạnh
        - ✅ Hỗ trợ L4, L7, SSL offloading
        - ✅ Advanced features (WAF, DDoS protection)
        - ✅ High availability, redundancy
        - ❌ Đắt, phức tạp, cần expertise
        - **Use case**: enterprise applications, high-security requirements

### Load Balancing Algorithms
- [x] Đọc về "Round-Robin" algorithm
    - **Round-Robin** là algorithm đơn giản nhất: phân phối requests tuần tự qua các backend servers theo vòng tròn.
    - **How it works:**
        1. Load balancer giữ danh sách backend servers: [Server1, Server2, Server3]
        2. Request 1 → Server1
        3. Request 2 → Server2
        4. Request 3 → Server3
        5. Request 4 → Server1 (quay lại đầu)
        6. Lặp lại...
    - **Characteristics:**
        - ✅ Đơn giản, dễ implement
        - ✅ Phân phối đều (nếu tất cả servers có cùng capacity)
        - ✅ Stateless (không cần track state)
        - ❌ Không xem server load, có thể gửi request đến server đang busy
        - ❌ Không phù hợp khi servers có capacity khác nhau
- [x] Viết algorithm pseudocode cho Round-Robin
    ```python
    class RoundRobinLoadBalancer:
        def __init__(self, servers):
            self.servers = servers  # List of backend servers
            self.current_index = 0  # Current server index
            self.lock = Lock()  # Thread-safe lock
        
        def get_next_server(self):
            with self.lock:
                server = self.servers[self.current_index]
                self.current_index = (self.current_index + 1) % len(self.servers)
                return server
    ```
    - **Explanation:**
        - Giữ `current_index` để track server hiện tại
        - Mỗi request, chọn server tại `current_index`, rồi tăng index
        - Khi đến cuối list, quay lại đầu (dùng modulo `%`)
        - Dùng lock để thread-safe (nếu có concurrent requests)
- [x] Analyze: Pros và cons (5 points each)
    - **Pros:**
        1. ✅ **Đơn giản**: dễ implement, dễ debug
        2. ✅ **Fair distribution**: phân phối đều nếu servers có cùng capacity
        3. ✅ **Stateless**: không cần track state, dễ scale horizontally
        4. ✅ **Predictable**: dễ test, dễ verify behavior
        5. ✅ **Low overhead**: không cần tính toán phức tạp, nhanh
    - **Cons:**
        1. ❌ **Không xem server load**: có thể gửi request đến server đang busy
        2. ❌ **Không phù hợp servers khác capacity**: server mạnh và yếu nhận cùng số requests
        3. ❌ **Không xem request complexity**: request nặng và nhẹ được phân phối giống nhau
        4. ❌ **Không sticky**: không đảm bảo cùng user đến cùng server
        5. ❌ **Không adaptive**: không tự điều chỉnh khi server chậm/fail
- [x] Đọc về "Weighted Round-Robin" algorithm
    - **Weighted Round-Robin** là extension của Round-Robin, mỗi server có **weight** (trọng số).
    - **How it works:**
        1. Mỗi server có weight: Server1 (weight=3), Server2 (weight=2), Server3 (weight=1)
        2. Phân phối requests theo tỷ lệ weight:
           - Server1 nhận 3 requests
           - Server2 nhận 2 requests
           - Server3 nhận 1 request
        3. Lặp lại cycle
    - **Use case**: khi servers có capacity khác nhau (server mạnh có weight cao hơn).
- [x] Viết algorithm pseudocode cho Weighted Round-Robin
    ```python
    class WeightedRoundRobinLoadBalancer:
        def __init__(self, servers_with_weights):
            # servers_with_weights = [(server1, 3), (server2, 2), (server3, 1)]
            self.servers = servers_with_weights
            self.current_weights = [weight for _, weight in servers_with_weights]
            self.effective_weights = self.current_weights.copy()
            self.current_index = 0
        
        def get_next_server(self):
            # Find server with highest effective weight
            max_weight = max(self.effective_weights)
            index = self.effective_weights.index(max_weight)
            
            # Decrease effective weight
            self.effective_weights[index] -= sum(self.current_weights)
            
            # Increase all effective weights by their current weight
            for i in range(len(self.effective_weights)):
                self.effective_weights[i] += self.current_weights[i]
            
            return self.servers[index][0]  # Return server
    ```
    - **Simpler implementation (cycle-based):**
    ```python
    class WeightedRoundRobinSimple:
        def __init__(self, servers_with_weights):
            self.servers = []
            # Expand: server with weight=3 appears 3 times
            for server, weight in servers_with_weights:
                self.servers.extend([server] * weight)
            self.current_index = 0
        
        def get_next_server(self):
            server = self.servers[self.current_index]
            self.current_index = (self.current_index + 1) % len(self.servers)
            return server
    ```
- [x] Analyze: Pros và cons (5 points each)
    - **Pros:**
        1. ✅ **Handles different capacities**: server mạnh nhận nhiều requests hơn
        2. ✅ **Flexible**: có thể điều chỉnh weight theo server capacity
        3. ✅ **Still simple**: chỉ thêm weight logic vào Round-Robin
        4. ✅ **Fair for capacity**: phân phối theo tỷ lệ capacity
        5. ✅ **Easy to configure**: chỉ cần set weight cho mỗi server
    - **Cons:**
        1. ❌ **Static weights**: không tự động điều chỉnh khi server load thay đổi
        2. ❌ **Không xem current load**: vẫn có thể gửi đến server đang busy
        3. ❌ **Weight tuning**: cần manual tune weights, không tự động
        4. ❌ **Không xem request complexity**: request nặng/nhẹ được phân phối giống nhau
        5. ❌ **Không adaptive**: không tự điều chỉnh khi server chậm
- [x] Đọc về "Least Connections" algorithm
    - **Least Connections** chọn server có **ít active connections nhất**.
    - **How it works:**
        1. Load balancer track số active connections của mỗi server
        2. Khi có request mới, chọn server có ít connections nhất
        3. Khi connection close, giảm count
    - **Use case**: khi requests có duration khác nhau (long-running vs short requests).
    - **Advantage**: tự động balance dựa trên current load, không cần biết server capacity.
- [x] Viết algorithm pseudocode cho Least Connections
    ```python
    class LeastConnectionsLoadBalancer:
        def __init__(self, servers):
            self.servers = servers
            self.connection_counts = {server: 0 for server in servers}
            self.lock = Lock()
        
        def get_next_server(self):
            with self.lock:
                # Find server with minimum connections
                min_connections = min(self.connection_counts.values())
                candidates = [s for s in self.servers 
                             if self.connection_counts[s] == min_connections]
                # If tie, use round-robin among candidates
                server = candidates[0]  # Or use round-robin
                self.connection_counts[server] += 1
                return server
        
        def release_connection(self, server):
            with self.lock:
                if self.connection_counts[server] > 0:
                    self.connection_counts[server] -= 1
    ```
    - **Note**: Cần track connections (tăng khi request đến, giảm khi request hoàn thành).
- [x] Analyze: Pros và cons (5 points each)
    - **Pros:**
        1. ✅ **Adaptive**: tự động điều chỉnh theo current load
        2. ✅ **Handles varying request duration**: phù hợp khi requests có thời gian xử lý khác nhau
        3. ✅ **Fair distribution**: server ít load nhận nhiều requests hơn
        4. ✅ **Self-balancing**: không cần manual configuration
        5. ✅ **Good for long connections**: phù hợp WebSocket, long-polling
    - **Cons:**
        1. ❌ **Stateful**: cần track connection counts, phức tạp hơn
        2. ❌ **Overhead**: cần maintain state, update counts
        3. ❌ **Không xem request complexity**: connection count không phản ánh CPU/memory load
        4. ❌ **Race conditions**: cần thread-safe implementation
        5. ❌ **Không phù hợp short requests**: overhead không đáng nếu requests ngắn
- [x] Đọc về "IP Hash" algorithm
    - **IP Hash** (hoặc **Consistent Hashing**) hash source IP address để chọn server.
    - **How it works:**
        1. Hash source IP address: `hash(client_ip) % num_servers`
        2. Kết quả hash quyết định server nào nhận request
        3. Cùng IP → cùng server (sticky sessions)
    - **Use case**: khi cần sticky sessions (user luôn đến cùng server).
    - **Advantage**: đảm bảo cùng client IP luôn đến cùng server.
- [x] Viết algorithm pseudocode cho IP Hash
    ```python
    import hashlib
    
    class IPHashLoadBalancer:
        def __init__(self, servers):
            self.servers = servers
        
        def get_next_server(self, client_ip):
            # Hash IP address
            hash_value = int(hashlib.md5(client_ip.encode()).hexdigest(), 16)
            # Modulo to get server index
            server_index = hash_value % len(self.servers)
            return self.servers[server_index]
    ```
    - **Consistent Hashing version** (better for adding/removing servers):
    ```python
    class ConsistentHashLoadBalancer:
        def __init__(self, servers, replicas=100):
            self.servers = servers
            self.replicas = replicas
            self.hash_ring = {}
            
            # Create virtual nodes for each server
            for server in servers:
                for i in range(replicas):
                    hash_key = self._hash(f"{server}:{i}")
                    self.hash_ring[hash_key] = server
            self.sorted_keys = sorted(self.hash_ring.keys())
        
        def _hash(self, key):
            return int(hashlib.md5(key.encode()).hexdigest(), 16)
        
        def get_next_server(self, client_ip):
            hash_value = self._hash(client_ip)
            # Find first key >= hash_value
            for key in self.sorted_keys:
                if key >= hash_value:
                    return self.hash_ring[key]
            # Wrap around
            return self.hash_ring[self.sorted_keys[0]]
    ```
- [x] Analyze: Pros và cons (5 points each)
    - **Pros:**
        1. ✅ **Sticky sessions**: cùng IP → cùng server, tốt cho stateful applications
        2. ✅ **Deterministic**: cùng IP luôn đến cùng server, dễ debug
        3. ✅ **No state tracking**: không cần track connections, stateless
        4. ✅ **Consistent hashing**: khi thêm/bớt server, chỉ ảnh hưởng một phần traffic
        5. ✅ **Cache-friendly**: user data có thể cache trên server cụ thể
    - **Cons:**
        1. ❌ **Uneven distribution**: nếu IPs không phân bố đều, có thể lệch load
        2. ❌ **Not adaptive**: không xem server load, vẫn gửi đến server đang busy
        3. ❌ **NAT problems**: nhiều users sau NAT có cùng IP → cùng server (hotspot)
        4. ❌ **Server removal**: khi server fail, tất cả users của server đó phải migrate
        5. ❌ **Không phù hợp stateless**: nếu app stateless, sticky sessions không cần thiết
- [x] Đọc về "Least Response Time" algorithm
    - **Least Response Time** chọn server có **response time thấp nhất** (latency thấp nhất).
    - **How it works:**
        1. Load balancer đo response time của mỗi server (từ health checks hoặc actual requests)
        2. Khi có request mới, chọn server có response time thấp nhất
        3. Cập nhật response time liên tục
    - **Use case**: khi cần minimize latency, servers có performance khác nhau.
    - **Advantage**: tự động route đến server nhanh nhất.
- [x] Compare: All algorithms (10 criteria)
    1. **Complexity:**
        - Round-Robin: ⭐ Đơn giản
        - Weighted Round-Robin: ⭐⭐ Trung bình
        - Least Connections: ⭐⭐ Trung bình
        - IP Hash: ⭐⭐ Trung bình
        - Least Response Time: ⭐⭐⭐ Phức tạp
    2. **State tracking:**
        - Round-Robin: Stateless (chỉ cần index)
        - Weighted Round-Robin: Stateless (chỉ cần index)
        - Least Connections: Stateful (track connection counts)
        - IP Hash: Stateless (chỉ hash)
        - Least Response Time: Stateful (track response times)
    3. **Adaptive:**
        - Round-Robin: ❌ Không
        - Weighted Round-Robin: ❌ Không (static weights)
        - Least Connections: ✅ Có (theo connections)
        - IP Hash: ❌ Không
        - Least Response Time: ✅ Có (theo latency)
    4. **Sticky sessions:**
        - Round-Robin: ❌ Không
        - Weighted Round-Robin: ❌ Không
        - Least Connections: ❌ Không
        - IP Hash: ✅ Có (cùng IP → cùng server)
        - Least Response Time: ❌ Không
    5. **Handles different capacities:**
        - Round-Robin: ❌ Không (phân phối đều)
        - Weighted Round-Robin: ✅ Có (theo weights)
        - Least Connections: ✅ Có (tự động)
        - IP Hash: ❌ Không
        - Least Response Time: ✅ Có (tự động)
    6. **Overhead:**
        - Round-Robin: ⭐ Thấp
        - Weighted Round-Robin: ⭐ Thấp
        - Least Connections: ⭐⭐ Trung bình (update counts)
        - IP Hash: ⭐ Thấp (chỉ hash)
        - Least Response Time: ⭐⭐⭐ Cao (đo latency)
    7. **Best for:**
        - Round-Robin: Servers cùng capacity, stateless apps
        - Weighted Round-Robin: Servers khác capacity
        - Least Connections: Long connections, varying duration
        - IP Hash: Sticky sessions, stateful apps
        - Least Response Time: Latency-sensitive apps
    8. **Fairness:**
        - Round-Robin: ✅ Fair (nếu servers giống nhau)
        - Weighted Round-Robin: ✅ Fair theo capacity
        - Least Connections: ✅ Fair theo load
        - IP Hash: ⚠️ Có thể không fair (IP distribution)
        - Least Response Time: ✅ Fair theo performance
    9. **Scalability:**
        - Round-Robin: ✅ Tốt (stateless)
        - Weighted Round-Robin: ✅ Tốt (stateless)
        - Least Connections: ⚠️ Cần sync state nếu multiple LBs
        - IP Hash: ✅ Tốt (stateless)
        - Least Response Time: ⚠️ Cần sync state
    10. **Real-world usage:**
        - Round-Robin: ⭐⭐⭐ Rất phổ biến
        - Weighted Round-Robin: ⭐⭐ Phổ biến
        - Least Connections: ⭐⭐⭐ Rất phổ biến
        - IP Hash: ⭐⭐ Phổ biến (sticky sessions)
        - Least Response Time: ⭐ Ít phổ biến (overhead cao)
- [x] Research: Real-world usage của mỗi algorithm
    - **Round-Robin:**
        - ✅ Default algorithm trong nhiều load balancers
        - ✅ Dùng khi servers có cùng capacity, stateless apps
        - ✅ Ví dụ: Nginx default, AWS ELB default
    - **Weighted Round-Robin:**
        - ✅ Dùng khi có servers khác capacity (old vs new hardware)
        - ✅ Ví dụ: HAProxy, F5 BIG-IP
    - **Least Connections:**
        - ✅ Rất phổ biến cho web applications
        - ✅ Dùng khi requests có duration khác nhau
        - ✅ Ví dụ: HAProxy default, AWS ALB option
    - **IP Hash:**
        - ✅ Dùng cho sticky sessions (e-commerce carts, user sessions)
        - ✅ Ví dụ: Nginx `ip_hash`, AWS ALB sticky sessions
    - **Least Response Time:**
        - ✅ Ít phổ biến vì overhead cao
        - ✅ Dùng cho latency-critical applications
        - ✅ Ví dụ: F5 BIG-IP, một số custom implementations

### Health Checks & Failover
- [x] Đọc về "health checks" trong load balancers
    - **Health checks** là mechanism để load balancer kiểm tra backend servers có healthy không.
    - **Mục đích:**
        - ✅ Detect unhealthy servers (down, slow, error)
        - ✅ Remove unhealthy servers khỏi pool (không gửi traffic đến)
        - ✅ Re-add servers khi healthy lại
        - ✅ Prevent cascading failures (không gửi traffic đến dead servers)
    - **Types:**
        - **Active health checks**: load balancer chủ động gửi requests để check
        - **Passive health checks**: monitor actual requests/responses
    - **Critical**: không có health checks, load balancer sẽ tiếp tục gửi traffic đến dead servers → users gặp errors.
- [x] Đọc về "active health checks" - how it works
    - **Active health checks** (proactive): load balancer chủ động gửi health check requests đến backend servers.
    - **How it works:**
        1. Load balancer gửi HTTP request đến `/health` endpoint mỗi N seconds (interval)
        2. Backend server trả response (200 OK = healthy, 5xx = unhealthy)
        3. Load balancer đếm success/failure counts
        4. Nếu failure count > threshold → mark server unhealthy, remove khỏi pool
        5. Tiếp tục check, nếu healthy lại → re-add vào pool
    - **Advantages:**
        - ✅ Detect failures sớm (trước khi users gặp errors)
        - ✅ Có thể check comprehensive health (database, cache, dependencies)
        - ✅ Predictable (check đều đặn)
    - **Disadvantages:**
        - ❌ Tạo thêm traffic (overhead)
        - ❌ Có thể false positive nếu health check endpoint fail nhưng app vẫn OK
- [x] Đọc về "passive health checks" - how it works
    - **Passive health checks** (reactive): load balancer monitor actual user requests/responses.
    - **How it works:**
        1. Load balancer forward user requests đến backend
        2. Monitor response: status code, latency, errors
        3. Nếu thấy nhiều errors/timeouts → mark server unhealthy
        4. Remove khỏi pool, tiếp tục monitor
        5. Nếu thấy responses OK lại → re-add vào pool
    - **Advantages:**
        - ✅ Không tạo thêm traffic (dùng actual requests)
        - ✅ Phản ánh real user experience
        - ✅ Tự động adapt theo actual load
    - **Disadvantages:**
        - ❌ Phát hiện chậm hơn (phải đợi user requests)
        - ❌ Users có thể gặp errors trước khi detect
        - ❌ Khó distinguish server issue vs request issue
- [x] So sánh: Active vs Passive health checks (5 criteria)
    1. **Detection speed:**
        - Active: ✅ Nhanh (check đều đặn, detect trong vài seconds)
        - Passive: ❌ Chậm (phải đợi user requests, có thể mất vài minutes)
    2. **Traffic overhead:**
        - Active: ❌ Có overhead (health check requests)
        - Passive: ✅ Không overhead (dùng actual requests)
    3. **False positives:**
        - Active: ⚠️ Có thể false positive (health endpoint fail nhưng app OK)
        - Passive: ✅ Ít false positive (dựa trên actual requests)
    4. **User impact:**
        - Active: ✅ Users ít gặp errors (detect sớm)
        - Passive: ❌ Users có thể gặp errors trước khi detect
    5. **Best practice:**
        - Active: ✅ Nên dùng cho production (detect sớm)
        - Passive: ✅ Nên dùng kết hợp (backup, validate)
- [x] Đọc về "health check intervals" - how often?
    - **Health check interval** là khoảng thời gian giữa các health check requests.
    - **Recommendations:**
        - **Fast detection**: 1-5 seconds (cho critical services)
        - **Normal**: 10-30 seconds (cho most services)
        - **Slow**: 60+ seconds (cho non-critical services)
    - **Trade-offs:**
        - ✅ Interval ngắn → detect nhanh, nhưng tạo nhiều traffic
        - ✅ Interval dài → ít traffic, nhưng detect chậm
    - **Best practice:**
        - Start với 10-15 seconds
        - Tune dựa trên: failover time requirement, traffic overhead tolerance
        - Critical services (payment, auth): 5 seconds
        - Normal services: 15-30 seconds
- [x] Đọc về "health check timeout" - how long?
    - **Health check timeout** là thời gian đợi response từ health check request.
    - **Recommendations:**
        - **Fast**: 1-2 seconds (cho local/private networks)
        - **Normal**: 3-5 seconds (cho most cases)
        - **Slow**: 10+ seconds (cho slow networks)
    - **Best practice:**
        - Timeout nên < interval (nếu timeout = interval, sẽ không kịp check)
        - Timeout = 30-50% của interval (ví dụ: interval=10s, timeout=3-5s)
        - Health check endpoint nên response nhanh (< 100ms)
- [x] Đọc về "health check thresholds" - success/failure counts
    - **Health check thresholds** là số lần success/failure cần để mark server healthy/unhealthy.
    - **Failure threshold:**
        - Sau N lần failure liên tiếp → mark unhealthy
        - Ví dụ: 3 failures → unhealthy (tránh false positive do network hiccup)
    - **Success threshold:**
        - Sau N lần success liên tiếp → mark healthy (re-add vào pool)
        - Ví dụ: 2 successes → healthy (nhanh chóng re-add)
    - **Best practice:**
        - Failure threshold: 2-3 (tránh false positive, nhưng vẫn detect nhanh)
        - Success threshold: 1-2 (nhanh chóng re-add khi healthy)
        - **Example config:**
            - Interval: 10s
            - Timeout: 3s
            - Failure threshold: 3 (3 failures = 30s để detect)
            - Success threshold: 2 (2 successes = 20s để re-add)
- [x] Đọc về "graceful degradation" - remove unhealthy backends
    - **Graceful degradation** là quá trình remove unhealthy backends khỏi pool một cách an toàn.
    - **Process:**
        1. Detect unhealthy (qua health checks)
        2. **Drain connections**: không nhận connections mới, nhưng vẫn xử lý connections hiện tại
        3. Đợi connections hiện tại hoàn thành (grace period)
        4. Remove khỏi pool
    - **Why important:**
        - ✅ Tránh drop active requests (users đang xử lý)
        - ✅ Smooth transition (không abrupt)
        - ✅ Better user experience
    - **Implementation:**
        - Load balancer stop forward new requests
        - Đợi active requests complete (timeout: 30-60s)
        - Sau đó remove khỏi pool
- [x] Đọc về "failover time" - how fast?
    - **Failover time** là thời gian từ khi server fail đến khi load balancer detect và remove khỏi pool.
    - **Calculation:**
        - Failover time = (interval × failure_threshold) + timeout
        - Ví dụ: interval=10s, failure_threshold=3, timeout=3s
        - Failover time = (10 × 3) + 3 = 33 seconds
    - **Targets:**
        - **Critical services**: < 10 seconds
        - **Normal services**: 10-30 seconds
        - **Non-critical**: 30-60 seconds
    - **How to improve:**
        - Giảm interval (5s thay vì 10s)
        - Giảm failure threshold (2 thay vì 3)
        - Dùng passive health checks (detect nhanh hơn)
        - Trade-off: nhiều traffic hơn, có thể false positive
- [x] Research: Health check best practices
    1. ✅ **Use active health checks** cho production (detect sớm)
    2. ✅ **Health check endpoint nên nhanh** (< 100ms), không check heavy dependencies
    3. ✅ **Separate liveness vs readiness:**
        - Liveness: app đang chạy? (process alive)
        - Readiness: app sẵn sàng nhận traffic? (dependencies OK)
    4. ✅ **Health check endpoint nên check dependencies** (database, cache) nhưng nhanh
    5. ✅ **Use failure threshold** để tránh false positive (2-3 failures)
    6. ✅ **Use success threshold** để nhanh chóng re-add (1-2 successes)
    7. ✅ **Graceful shutdown**: drain connections trước khi remove
    8. ✅ **Monitor health check metrics**: success rate, failover time
    9. ✅ **Alert on health check failures**: notify team khi server unhealthy
    10. ✅ **Test failover**: simulate failures, verify failover time

### Sticky Sessions
- [x] Đọc về "sticky sessions" (session affinity)
    - **Sticky sessions** (session affinity) đảm bảo requests từ cùng một client/user luôn được route đến cùng một backend server.
    - **Why needed:**
        - Stateful applications: server lưu session data trong memory
        - User-specific data: cache user data trên server cụ thể
        - File uploads: upload file đến server cụ thể
    - **How it works:**
        - Load balancer track mapping: client → server
        - Requests từ cùng client → cùng server
    - **Implementation:**
        - Cookie-based: dùng HTTP cookies
        - IP-based: hash IP address
- [x] Đọc về "cookie-based sticky sessions"
    - **Cookie-based sticky sessions** dùng HTTP cookies để track server assignment.
    - **How it works:**
        1. Client gửi request đầu tiên (không có cookie)
        2. Load balancer chọn server (theo algorithm), tạo cookie với server ID
        3. Response về client kèm cookie: `Set-Cookie: SERVERID=server1`
        4. Client gửi requests tiếp theo kèm cookie: `Cookie: SERVERID=server1`
        5. Load balancer đọc cookie, route đến server1
    - **Types:**
        - **Insert cookie**: load balancer tự tạo và insert cookie
        - **Rewrite cookie**: load balancer rewrite cookie từ backend
        - **Prefix cookie**: load balancer thêm prefix vào cookie
    - **Advantages:**
        - ✅ Works với multiple clients sau NAT (mỗi client có cookie riêng)
        - ✅ Flexible (có thể set TTL, domain, path)
        - ✅ Works với mobile apps (nếu support cookies)
    - **Disadvantages:**
        - ❌ Client phải support cookies
        - ❌ Có thể bị manipulate (client thay đổi cookie)
        - ❌ Overhead (cookie trong mỗi request)
- [x] Đọc về "IP-based sticky sessions"
    - **IP-based sticky sessions** hash source IP address để chọn server.
    - **How it works:**
        1. Load balancer hash client IP: `hash(client_ip) % num_servers`
        2. Kết quả hash quyết định server
        3. Cùng IP → cùng server
    - **Advantages:**
        - ✅ Đơn giản, không cần cookies
        - ✅ Works với clients không support cookies
        - ✅ Stateless (không cần track state)
    - **Disadvantages:**
        - ❌ NAT problem: nhiều users sau NAT có cùng IP → cùng server (hotspot)
        - ❌ Mobile users: IP thay đổi khi switch network → mất session
        - ❌ Uneven distribution: nếu IPs không phân bố đều
- [x] Analyze: When sticky sessions needed? (5 use cases)
    1. **Stateful applications:**
        - Server lưu session data trong memory (không dùng shared session store)
        - Ví dụ: legacy applications, in-memory sessions
    2. **File uploads:**
        - Upload file đến server cụ thể, cần requests tiếp theo đến cùng server để access file
        - Ví dụ: multi-part uploads, file processing
    3. **User-specific caching:**
        - Cache user data trên server cụ thể để tăng performance
        - Ví dụ: user profile cache, personalized content
    4. **WebSocket connections:**
        - WebSocket cần persistent connection đến cùng server
        - Ví dụ: real-time chat, gaming
    5. **Legacy systems:**
        - Systems không thể refactor để stateless
        - Ví dụ: old applications, third-party integrations
- [x] Analyze: Trade-offs của sticky sessions (5 points)
    1. **Load imbalance:**
        - ❌ Một số users có nhiều requests → server đó nhận nhiều load
        - ❌ Khó balance load evenly
    2. **Server failure:**
        - ❌ Khi server fail, tất cả users của server đó mất session
        - ❌ Phải re-authenticate, mất data
    3. **Scaling:**
        - ❌ Khó scale (thêm server không giúp users hiện tại)
        - ❌ Phải wait users disconnect mới balance được
    4. **Complexity:**
        - ❌ Phức tạp hơn (cần track sessions, handle failures)
        - ❌ Debug khó hơn (phải track server assignment)
    5. **Maintenance:**
        - ❌ Khó maintain (không thể easily remove server)
        - ❌ Rolling updates khó (phải drain sessions)
- [x] Đọc về "session persistence" - how it works
    - **Session persistence** là mechanism để maintain sticky sessions qua multiple requests.
    - **How it works:**
        1. **Initial request**: client gửi request, load balancer chọn server, tạo mapping
        2. **Store mapping**: load balancer lưu mapping (cookie, IP hash, hoặc in-memory)
        3. **Subsequent requests**: load balancer lookup mapping, route đến cùng server
        4. **TTL/expiry**: mapping có TTL, expire sau một thời gian
    - **Storage options:**
        - **In-memory**: nhanh, nhưng mất khi LB restart
        - **Shared storage**: Redis, database (persist qua LB restarts)
    - **Configuration:**
        - TTL: 1-24 hours (tùy use case)
        - Domain/path: chỉ apply cho specific paths
- [x] Đọc về "sticky session problems" - what can go wrong?
    1. **Server failure:**
        - Server fail → tất cả users của server đó mất session
        - Users phải re-authenticate, mất data
    2. **Load imbalance:**
        - Một số users có nhiều requests → server đó overload
        - Khó balance load
    3. **NAT problems (IP-based):**
        - Nhiều users sau NAT có cùng IP → cùng server (hotspot)
        - Server đó nhận quá nhiều load
    4. **Mobile users:**
        - IP thay đổi khi switch network (WiFi → 4G)
        - Mất session, phải re-authenticate
    5. **Scaling issues:**
        - Thêm server mới không giúp users hiện tại
        - Phải wait sessions expire mới balance được
    6. **Cookie manipulation:**
        - Client có thể thay đổi cookie → route đến server khác
        - Có thể gây lỗi nếu server không có session data
- [x] Research: Alternatives to sticky sessions (stateless design)
    1. **Shared session store:**
        - Lưu session data trong shared storage (Redis, database)
        - Bất kỳ server nào cũng có thể access session
        - ✅ Stateless, dễ scale
        - ✅ Server fail không mất session
    2. **JWT tokens:**
        - Encode session data trong JWT token
        - Client gửi token trong mỗi request
        - Server validate và extract data từ token
        - ✅ Stateless, không cần shared storage
        - ✅ Works với mobile apps
    3. **Database-backed sessions:**
        - Lưu session data trong database
        - Bất kỳ server nào cũng query được
        - ✅ Persistent, không mất khi server fail
        - ❌ Overhead (database query mỗi request)
    4. **External session service:**
        - Dedicated session service (microservice)
        - Tất cả servers query service này
        - ✅ Centralized, dễ manage
        - ❌ Single point of failure (cần HA)
    5. **Client-side storage:**
        - Lưu data trên client (localStorage, cookies)
        - Server không cần track sessions
        - ✅ Stateless, không cần storage
        - ❌ Security concerns (data trên client)

### API Design Principles
- [x] Đọc về "RESTful API design" principles
    - **REST** (Representational State Transfer) là architectural style cho web services.
    - **Core principles:**
        - ✅ **Resource-based**: mọi thứ là resource (users, orders, products)
        - ✅ **Stateless**: mỗi request độc lập, không cần server state
        - ✅ **Uniform interface**: dùng HTTP methods chuẩn (GET, POST, PUT, DELETE)
        - ✅ **Client-server**: separation of concerns
        - ✅ **Cacheable**: responses có thể cache
        - ✅ **Layered system**: có thể có multiple layers (proxies, gateways)
    - **Benefits:**
        - Dễ hiểu, dễ sử dụng
        - Scalable (stateless)
        - Cacheable (improve performance)
        - Loosely coupled (client và server độc lập)
- [x] Đọc về "REST constraints" - 6 constraints
    1. **Client-Server:**
        - Client và server tách biệt, độc lập
        - Server không biết client UI, client không biết server storage
    2. **Stateless:**
        - Mỗi request chứa đầy đủ thông tin để server xử lý
        - Server không lưu client state giữa các requests
        - Session state nên ở client (cookies, tokens)
    3. **Cacheable:**
        - Responses phải indicate có thể cache không
        - Dùng HTTP cache headers (Cache-Control, ETag)
        - Improve performance, reduce server load
    4. **Uniform Interface:**
        - Dùng HTTP methods chuẩn (GET, POST, PUT, DELETE)
        - Resource identification qua URLs
        - Self-descriptive messages (headers, status codes)
    5. **Layered System:**
        - Có thể có multiple layers (load balancer, proxy, gateway)
        - Client không biết có bao nhiêu layers
        - Mỗi layer chỉ tương tác với layer kế tiếp
    6. **Code on Demand (optional):**
        - Server có thể gửi executable code cho client (JavaScript)
        - Ít dùng trong REST APIs
- [x] Đọc về "HTTP methods" - GET, POST, PUT, PATCH, DELETE semantics
    - **GET:**
        - Retrieve resource (read-only)
        - ✅ Idempotent, ✅ Safe
        - Không có request body (data trong query params)
        - Response: 200 OK (success), 404 Not Found
    - **POST:**
        - Create new resource hoặc trigger action
        - ❌ Không idempotent, ❌ Không safe
        - Có request body
        - Response: 201 Created (created), 200 OK (action completed)
    - **PUT:**
        - Update/replace entire resource (full update)
        - ✅ Idempotent, ❌ Không safe
        - Có request body (full resource data)
        - Response: 200 OK (updated), 201 Created (created nếu không tồn tại)
    - **PATCH:**
        - Partial update (chỉ update fields được gửi)
        - ⚠️ Nên idempotent (nhưng không bắt buộc), ❌ Không safe
        - Có request body (chỉ fields cần update)
        - Response: 200 OK (updated)
    - **DELETE:**
        - Delete resource
        - ✅ Idempotent, ❌ Không safe
        - Không có request body
        - Response: 204 No Content (deleted), 404 Not Found (không tồn tại)
- [x] Đọc về "idempotency" - what is it? Why important?
    - **Idempotency**: thực hiện operation nhiều lần với cùng input → kết quả giống như thực hiện 1 lần.
    - **Mathematical definition:** `f(f(x)) = f(x)`
    - **HTTP idempotent methods:**
        - ✅ GET, PUT, DELETE, HEAD, OPTIONS
        - ❌ POST (không idempotent)
        - ⚠️ PATCH (nên idempotent, nhưng không bắt buộc)
    - **Why important:**
        1. **Network retries**: client retry request → không tạo duplicate
        2. **Reliability**: đảm bảo operation chỉ thực hiện 1 lần
        3. **User experience**: user click nhiều lần → không tạo duplicate orders
        4. **Distributed systems**: đảm bảo consistency khi có network issues
    - **Example:**
        - PUT `/users/123` với cùng data nhiều lần → user vẫn giống nhau (idempotent)
        - POST `/orders` nhiều lần → tạo nhiều orders (không idempotent)
- [x] Đọc về "safety" - safe HTTP methods
    - **Safe methods**: không modify server state, chỉ đọc data.
    - **Safe methods:**
        - ✅ GET: chỉ đọc, không modify
        - ✅ HEAD: giống GET nhưng không có response body
        - ✅ OPTIONS: query server capabilities
    - **Unsafe methods:**
        - ❌ POST, PUT, PATCH, DELETE: modify server state
    - **Why important:**
        - Safe methods có thể cache, pre-fetch
        - Browsers có thể pre-fetch safe methods
        - Web crawlers chỉ crawl safe methods
        - Safe methods không có side effects
- [x] Đọc về "stateless" - REST stateless constraint
    - **Stateless**: mỗi request chứa đầy đủ thông tin để server xử lý, không cần server lưu client state.
    - **Requirements:**
        - Request phải chứa: authentication (token), session (nếu cần), all data cần thiết
        - Server không lưu client state giữa requests
        - Session state nên ở client (cookies, JWT tokens)
    - **Benefits:**
        - ✅ **Scalability**: dễ scale horizontally (bất kỳ server nào cũng xử lý được)
        - ✅ **Reliability**: server fail không mất state
        - ✅ **Simplicity**: không cần sync state giữa servers
    - **Trade-offs:**
        - ❌ Mỗi request phải gửi thêm data (token, session)
        - ❌ Không thể optimize bằng server-side caching của client state
- [x] Đọc về "resource-based URLs" - design principles
    - **Resource-based URLs**: URLs represent resources, không phải actions.
    - **Good examples:**
        - ✅ `GET /users/123` - get user
        - ✅ `POST /users` - create user
        - ✅ `PUT /users/123` - update user
        - ✅ `DELETE /users/123` - delete user
        - ✅ `GET /users/123/orders` - get user's orders
    - **Bad examples:**
        - ❌ `GET /getUser?id=123` - action trong URL
        - ❌ `POST /createUser` - action trong URL
        - ❌ `POST /users/123/delete` - action trong URL
    - **Principles:**
        1. **Use nouns, not verbs**: `/users` not `/getUsers`
        2. **Use plural nouns**: `/users` not `/user`
        3. **Hierarchical**: `/users/123/orders` not `/userOrders?userId=123`
        4. **Consistent**: `/users`, `/orders`, `/products` (all plural)
        5. **Versioning**: `/v1/users`, `/v2/users`
- [x] Đọc về "HTTP status codes" - proper usage
    - **2xx Success:**
        - `200 OK`: success (GET, PUT, PATCH)
        - `201 Created`: resource created (POST)
        - `204 No Content`: success, no response body (DELETE)
    - **3xx Redirection:**
        - `301 Moved Permanently`: resource moved
        - `302 Found`: temporary redirect
        - `304 Not Modified`: cache valid (conditional GET)
    - **4xx Client Error:**
        - `400 Bad Request`: invalid request (validation error)
        - `401 Unauthorized`: authentication required
        - `403 Forbidden`: authenticated but not authorized
        - `404 Not Found`: resource not found
        - `409 Conflict`: conflict (duplicate, concurrent update)
        - `429 Too Many Requests`: rate limit exceeded
    - **5xx Server Error:**
        - `500 Internal Server Error`: server error
        - `502 Bad Gateway`: gateway/proxy error
        - `503 Service Unavailable`: service down/maintenance
        - `504 Gateway Timeout`: gateway timeout
    - **Best practices:**
        - Dùng đúng status code (không dùng 200 cho mọi thứ)
        - 4xx = client error, 5xx = server error
        - Provide error details trong response body
- [x] Research: RESTful API best practices
    1. ✅ **Use resource-based URLs**: `/users/123` not `/getUser?id=123`
    2. ✅ **Use proper HTTP methods**: GET (read), POST (create), PUT (update), DELETE (delete)
    3. ✅ **Use proper status codes**: 200, 201, 400, 404, 500
    4. ✅ **Version your APIs**: `/v1/users`, `/v2/users`
    5. ✅ **Use consistent naming**: plural nouns, kebab-case hoặc camelCase
    6. ✅ **Support filtering, sorting, pagination**: `?filter=active&sort=name&page=1`
    7. ✅ **Return consistent error format**: `{error: {code, message, details}}`
    8. ✅ **Use HTTPS**: always encrypt traffic
    9. ✅ **Document your API**: OpenAPI/Swagger
    10. ✅ **Implement rate limiting**: prevent abuse
    11. ✅ **Use idempotency keys**: cho POST requests (payment, orders)
    12. ✅ **Support content negotiation**: `Accept: application/json`

### API Versioning Strategies
- [x] Đọc về "URL versioning" - /v1/, /v2/
    - **URL versioning** là đặt version trong URL path: `/v1/users`, `/v2/users`.
    - **How it works:**
        - Version là part của URL path
        - Mỗi version có thể có different implementation
        - Example: `GET /v1/users/123` vs `GET /v2/users/123`
    - **Implementation:**
        - Route `/v1/*` → version 1 handler
        - Route `/v2/*` → version 2 handler
        - Có thể share code hoặc completely separate
    - **Most common approach**: được dùng bởi GitHub, Twitter, Stripe.
- [x] Analyze: Pros và cons của URL versioning (5 points each)
    - **Pros:**
        1. ✅ **Simple và clear**: dễ hiểu, version rõ ràng trong URL
        2. ✅ **Easy to implement**: dễ route, dễ test
        3. ✅ **Cacheable**: có thể cache riêng cho mỗi version
        4. ✅ **Browser-friendly**: có thể test trực tiếp trong browser
        5. ✅ **Explicit**: client phải specify version, không ambiguous
    - **Cons:**
        1. ❌ **URL pollution**: version trong URL, không clean
        2. ❌ **Breaking URLs**: khi upgrade version, URL thay đổi
        3. ❌ **Harder to maintain**: phải maintain multiple code paths
        4. ❌ **Not RESTful**: version không phải resource
        5. ❌ **Client must update**: client phải update URL khi upgrade
- [x] Đọc về "Header versioning" - Accept: application/vnd.api+json;version=1
    - **Header versioning** là đặt version trong HTTP headers (Accept header hoặc custom header).
    - **How it works:**
        - Version trong header: `Accept: application/vnd.api+json;version=1`
        - Hoặc custom header: `API-Version: 1`
        - URL giữ nguyên: `/users/123`
    - **Implementation:**
        - Server đọc header, route đến đúng version handler
        - URL không thay đổi giữa các versions
    - **Less common**: được dùng bởi một số APIs (Microsoft Graph API).
- [x] Analyze: Pros và cons của Header versioning (5 points each)
    - **Pros:**
        1. ✅ **Clean URLs**: URL không có version, clean hơn
        2. ✅ **RESTful**: version không phải resource, nên không nên trong URL
        3. ✅ **Flexible**: có thể support multiple versions cùng lúc
        4. ✅ **Backward compatible**: URL không thay đổi
        5. ✅ **Content negotiation**: phù hợp HTTP content negotiation
    - **Cons:**
        1. ❌ **Less discoverable**: version không rõ ràng, phải đọc docs
        2. ❌ **Harder to test**: không thể test trực tiếp trong browser
        3. ❌ **Client must set header**: client phải nhớ set header
        4. ❌ **Caching complexity**: cache phải consider headers
        5. ❌ **Less common**: ít được dùng, developers ít quen
- [x] Đọc về "Query parameter versioning" - ?version=1
    - **Query parameter versioning** là đặt version trong query parameter: `/users?version=1`.
    - **How it works:**
        - Version trong query string: `?version=1` hoặc `?v=1`
        - URL path giữ nguyên: `/users`
    - **Implementation:**
        - Server đọc query parameter, route đến đúng version
        - Có thể có default version nếu không specify
    - **Less common**: ít được dùng, thường không recommended.
- [x] Analyze: Pros và cons của Query versioning (5 points each)
    - **Pros:**
        1. ✅ **Simple**: dễ implement, chỉ cần đọc query param
        2. ✅ **Optional**: có thể có default version
        3. ✅ **Easy to test**: có thể test trong browser
        4. ✅ **Flexible**: có thể override version per request
    - **Cons:**
        1. ❌ **Not RESTful**: query params nên dùng cho filtering, không phải versioning
        2. ❌ **Confusing**: dễ nhầm với filtering params
        3. ❌ **Cache complexity**: query params affect caching
        4. ❌ **Less explicit**: version không rõ ràng như URL
        5. ❌ **Not recommended**: không phải best practice
- [x] Đọc về "content negotiation" - versioning via content type
    - **Content negotiation versioning** dùng HTTP content negotiation để specify version.
    - **How it works:**
        - Client gửi `Accept: application/vnd.company.api+json;version=2`
        - Server trả response với `Content-Type: application/vnd.company.api+json;version=2`
        - Version trong content type
    - **Implementation:**
        - Server parse Accept header, route đến đúng version
        - Response có Content-Type matching
    - **RESTful approach**: phù hợp HTTP standards.
- [x] Compare: All versioning strategies (10 criteria)
    1. **Simplicity:**
        - URL: ⭐⭐⭐ Rất đơn giản
        - Header: ⭐⭐ Trung bình
        - Query: ⭐⭐⭐ Rất đơn giản
        - Content Negotiation: ⭐ Phức tạp
    2. **Discoverability:**
        - URL: ✅ Rất rõ ràng
        - Header: ❌ Không rõ ràng
        - Query: ⚠️ Trung bình
        - Content Negotiation: ❌ Không rõ ràng
    3. **RESTful:**
        - URL: ❌ Không (version không phải resource)
        - Header: ✅ Có (HTTP headers)
        - Query: ❌ Không (query cho filtering)
        - Content Negotiation: ✅ Có (HTTP standard)
    4. **Browser-friendly:**
        - URL: ✅ Có thể test trong browser
        - Header: ❌ Khó test (cần tools)
        - Query: ✅ Có thể test trong browser
        - Content Negotiation: ❌ Khó test
    5. **Caching:**
        - URL: ✅ Dễ cache (URL khác nhau)
        - Header: ⚠️ Phức tạp (phải consider headers)
        - Query: ⚠️ Phức tạp (query params affect cache)
        - Content Negotiation: ⚠️ Phức tạp
    6. **Backward compatibility:**
        - URL: ❌ URL thay đổi
        - Header: ✅ URL không thay đổi
        - Query: ✅ URL không thay đổi
        - Content Negotiation: ✅ URL không thay đổi
    7. **Industry adoption:**
        - URL: ⭐⭐⭐ Rất phổ biến (GitHub, Twitter, Stripe)
        - Header: ⭐ Ít phổ biến
        - Query: ⭐ Ít phổ biến
        - Content Negotiation: ⭐ Ít phổ biến
    8. **Client complexity:**
        - URL: ✅ Đơn giản (chỉ cần update URL)
        - Header: ⚠️ Phức tạp hơn (phải set header)
        - Query: ✅ Đơn giản
        - Content Negotiation: ⚠️ Phức tạp
    9. **Maintenance:**
        - URL: ⚠️ Phải maintain multiple routes
        - Header: ⚠️ Phải maintain multiple handlers
        - Query: ⚠️ Phải maintain multiple handlers
        - Content Negotiation: ⚠️ Phức tạp
    10. **Best practice:**
        - URL: ✅ Recommended (most common)
        - Header: ✅ Alternative (RESTful)
        - Query: ❌ Not recommended
        - Content Negotiation: ✅ Alternative (RESTful)
- [x] Research: Real-world examples của mỗi strategy
    - **URL versioning:**
        - ✅ GitHub: `/api/v3/users`
        - ✅ Twitter: `/1.1/statuses/update.json`
        - ✅ Stripe: `/v1/charges`
        - ✅ Facebook: `/v2.12/me`
    - **Header versioning:**
        - ✅ Microsoft Graph API: `Accept: application/json;odata.metadata=minimal;odata.version=4.0`
        - ✅ Some internal APIs
    - **Query parameter versioning:**
        - ⚠️ Ít được dùng trong production
        - Một số legacy APIs
    - **Content negotiation:**
        - ✅ Một số APIs dùng custom content types
        - ✅ RESTful APIs theo HTTP standards
- [x] Analyze: When to use which strategy? (5 use cases each)
    - **Use URL versioning khi:**
        1. **Public APIs**: cần rõ ràng, dễ discover (GitHub, Stripe)
        2. **Simple requirements**: không cần content negotiation phức tạp
        3. **Browser testing**: cần test dễ dàng trong browser
        4. **Clear versioning**: muốn version rõ ràng trong URL
        5. **Industry standard**: follow industry best practices
    - **Use Header versioning khi:**
        1. **RESTful APIs**: muốn follow REST principles strictly
        2. **Clean URLs**: muốn URLs clean, không có version
        3. **Content negotiation**: cần support multiple content types
        4. **Internal APIs**: internal services, không cần discoverability
        5. **Flexible versioning**: cần flexibility trong versioning

### Idempotency
- [x] Đọc về "idempotency" - mathematical definition
    - **Idempotency**: thực hiện operation nhiều lần với cùng input → kết quả giống như thực hiện 1 lần.
    - **Mathematical definition:** `f(f(x)) = f(x)`
    - **Example:**
        - `PUT /users/123` với cùng data nhiều lần → user vẫn giống nhau
        - `DELETE /users/123` nhiều lần → user vẫn deleted (không error)
    - **Why important:**
        - Network retries không tạo duplicates
        - User click nhiều lần không tạo duplicate orders
        - Distributed systems consistency
- [x] Đọc về "HTTP idempotency" - which methods are idempotent?
    - **Idempotent methods:**
        - ✅ **GET**: đọc data, không modify → idempotent
        - ✅ **PUT**: replace resource → idempotent (cùng data → cùng result)
        - ✅ **DELETE**: delete resource → idempotent (delete nhiều lần vẫn deleted)
        - ✅ **HEAD**: giống GET → idempotent
        - ✅ **OPTIONS**: query capabilities → idempotent
    - **Non-idempotent methods:**
        - ❌ **POST**: create resource → không idempotent (mỗi lần tạo mới)
        - ⚠️ **PATCH**: partial update → nên idempotent nhưng không bắt buộc
    - **Note**: POST có thể làm idempotent bằng idempotency keys.
- [x] Đọc về "idempotency keys" - how to implement
    - **Idempotency keys** là unique key client gửi để đảm bảo request chỉ được xử lý 1 lần.
    - **How it works:**
        1. Client tạo unique key (UUID): `Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000`
        2. Client gửi request với header `Idempotency-Key`
        3. Server check key trong storage (Redis, database)
        4. Nếu key chưa có → process request, lưu key + response
        5. Nếu key đã có → return cached response (không process lại)
    - **Key format:**
        - UUID v4: `550e8400-e29b-41d4-a716-446655440000`
        - Hoặc client-generated unique string
    - **Implementation:**
        ```python
        def handle_request(request):
            idempotency_key = request.headers.get('Idempotency-Key')
            if idempotency_key:
                cached_response = redis.get(f"idempotency:{idempotency_key}")
                if cached_response:
                    return cached_response  # Return cached response
            
            # Process request
            response = process_request(request)
            
            # Store response
            if idempotency_key:
                redis.setex(f"idempotency:{idempotency_key}", 
                           ttl=24*3600, value=response)
            
            return response
        ```
- [x] Đọc về "idempotency storage" - where to store keys?
    - **Storage options:**
        1. **Redis** (recommended):
            - ✅ Fast (in-memory)
            - ✅ TTL support (auto expire)
            - ✅ Distributed (nếu dùng Redis cluster)
            - ✅ Low latency
            - ❌ Volatile (mất khi restart, nhưng OK vì có TTL)
        2. **Database:**
            - ✅ Persistent
            - ✅ ACID guarantees
            - ❌ Slower (disk I/O)
            - ❌ Cần cleanup job (delete old keys)
        3. **In-memory (local):**
            - ✅ Fastest
            - ❌ Không distributed (mỗi server có cache riêng)
            - ❌ Mất khi server restart
    - **Best practice:**
        - Dùng **Redis** với TTL
        - Key format: `idempotency:{key}`
        - Store cả request hash và response
- [x] Đọc về "idempotency TTL" - how long to store?
    - **TTL (Time To Live)** là thời gian lưu idempotency key.
    - **Recommendations:**
        - **Short-lived operations** (payments, orders): 24 hours
        - **Long-lived operations** (file uploads): 7 days
        - **Default**: 24 hours
    - **Considerations:**
        - Phải > max retry window của client
        - Phải > max processing time của request
        - Không cần quá lâu (sau 24h, duplicate request không còn relevant)
    - **Best practice:**
        - Start với 24 hours
        - Tune dựa trên use case
        - Monitor storage usage
- [x] Đọc về "idempotency scope" - per user? per request?
    - **Idempotency scope** quyết định key unique trong scope nào.
    - **Options:**
        1. **Global scope** (per request):
            - Key unique globally
            - Cùng key → cùng response (bất kỳ user nào)
            - ✅ Simple
            - ❌ User A có thể reuse key của User B
        2. **Per-user scope:**
            - Key unique per user: `idempotency:{user_id}:{key}`
            - Cùng key, khác user → different responses
            - ✅ More secure
            - ✅ User không thể conflict với nhau
            - ⚠️ Phức tạp hơn
    - **Best practice:**
        - **Per-user scope** cho security (recommended)
        - Format: `idempotency:{user_id}:{key}`
        - Hoặc include user_id trong key hash
- [x] Research: Idempotency implementation patterns
    1. **Stripe pattern:**
        - Client gửi `Idempotency-Key` header
        - Server check Redis
        - Return cached response nếu key exists
        - TTL: 24 hours
    2. **Two-phase pattern:**
        - Phase 1: Check key, lock nếu chưa có
        - Phase 2: Process request, store response
        - Prevent concurrent processing của cùng key
    3. **Request fingerprinting:**
        - Hash request (method + URL + body + headers)
        - Dùng hash làm idempotency key
        - Automatic idempotency (không cần client send key)
    4. **Database transaction:**
        - Store key trong database với unique constraint
        - Transaction đảm bảo atomicity
        - Slower nhưng more reliable
- [x] Analyze: Idempotency trade-offs (performance, storage)
    - **Performance:**
        - ✅ **Redis lookup**: ~1ms overhead (acceptable)
        - ✅ **Prevent duplicate processing**: tiết kiệm CPU nếu duplicate
        - ❌ **Storage overhead**: cần Redis memory
        - ❌ **Network overhead**: thêm 1 Redis call
    - **Storage:**
        - **Memory usage**: ~1KB per key (key + response)
        - **1M requests/day, 1% duplicates**: ~10K keys = ~10MB
        - **With TTL 24h**: peak ~10MB, average ~5MB
        - ✅ **Acceptable** cho most cases
    - **Trade-offs:**
        - ✅ **Reliability**: prevent duplicates, better UX
        - ✅ **Cost**: tiết kiệm processing cost nếu duplicate
        - ❌ **Complexity**: thêm code, thêm infrastructure
        - ❌ **Latency**: thêm ~1ms cho Redis lookup

### Rate Limiting Deep Dive
- [x] Đọc về "rate limiting" - why needed?
    - **Rate limiting** là mechanism giới hạn số requests client có thể gửi trong một khoảng thời gian.
    - **Why needed:**
        1. ✅ **Prevent abuse**: tránh DDoS, brute force attacks
        2. ✅ **Fair usage**: đảm bảo fair distribution của resources
        3. ✅ **Cost control**: giới hạn API usage để control costs
        4. ✅ **Stability**: prevent server overload, ensure service availability
        5. ✅ **Business model**: different tiers (free vs paid) có limits khác nhau
    - **Common limits:**
        - Per user: 100 requests/minute
        - Per IP: 1000 requests/hour
        - Per API key: based on tier
- [x] Đọc về "Token Bucket" algorithm (review from Week 5)
    - **Token Bucket**: bucket chứa tokens, mỗi request consume 1 token.
    - **How it works:**
        1. Bucket có capacity (max tokens)
        2. Tokens được refill với rate (ví dụ: 10 tokens/second)
        3. Request đến → check tokens
        4. Nếu có token → allow, consume 1 token
        5. Nếu không có token → reject (429)
    - **Characteristics:**
        - ✅ Allow bursts (nếu bucket đầy)
        - ✅ Smooth rate limiting
        - ✅ Memory efficient
    - **Example:** capacity=100, refill=10/sec → allow 100 requests burst, sau đó 10/sec
- [x] Đọc về "Sliding Window" algorithm (review from Week 5)
    - **Sliding Window**: track requests trong time window, slide window theo thời gian.
    - **How it works:**
        1. Track timestamps của requests trong window (ví dụ: last 60 seconds)
        2. Request đến → count requests trong window
        3. Nếu count < limit → allow
        4. Nếu count >= limit → reject
        5. Window slide theo thời gian
    - **Characteristics:**
        - ✅ Accurate (không có burst như Fixed Window)
        - ✅ Smooth distribution
        - ❌ Memory overhead (track timestamps)
    - **Example:** limit=100/min → track requests trong last 60 seconds
- [x] Đọc về "Fixed Window" algorithm
    - **Fixed Window**: chia time thành fixed windows, limit requests per window.
    - **How it works:**
        1. Chia time thành windows (ví dụ: mỗi phút)
        2. Track request count trong current window
        3. Request đến → check count trong current window
        4. Nếu count < limit → allow, increment count
        5. Nếu count >= limit → reject
        6. Khi window expires → reset count
    - **Characteristics:**
        - ✅ Simple implementation
        - ✅ Memory efficient (chỉ cần counter)
        - ❌ Burst problem (reset count → burst at window boundary)
    - **Example:** limit=100/min → window 0:00-0:59, window 1:00-1:59
- [x] Đọc về "Leaky Bucket" algorithm
    - **Leaky Bucket**: bucket chứa requests, leak (process) với constant rate.
    - **How it works:**
        1. Bucket có capacity (max requests)
        2. Requests đến → add vào bucket
        3. Bucket leak với constant rate (ví dụ: 10 requests/second)
        4. Nếu bucket full → reject
        5. Requests được process với constant rate
    - **Characteristics:**
        - ✅ Smooth output rate (constant rate)
        - ✅ No bursts
        - ❌ Requests có thể bị delay (queue trong bucket)
    - **Use case**: khi cần smooth, constant processing rate.
- [x] Compare: All rate limiting algorithms (5 criteria)
    1. **Accuracy:**
        - Token Bucket: ⭐⭐ Good (allow bursts)
        - Sliding Window: ⭐⭐⭐ Best (most accurate)
        - Fixed Window: ⭐ Fair (burst problem)
        - Leaky Bucket: ⭐⭐⭐ Best (smooth rate)
    2. **Memory usage:**
        - Token Bucket: ⭐⭐⭐ Low (chỉ cần counter)
        - Sliding Window: ⭐⭐ Medium (track timestamps)
        - Fixed Window: ⭐⭐⭐ Low (chỉ cần counter)
        - Leaky Bucket: ⭐⭐ Medium (queue requests)
    3. **Burst handling:**
        - Token Bucket: ✅ Allow bursts (nếu có tokens)
        - Sliding Window: ❌ No bursts (smooth)
        - Fixed Window: ⚠️ Burst at window boundary
        - Leaky Bucket: ❌ No bursts (smooth)
    4. **Implementation complexity:**
        - Token Bucket: ⭐⭐ Medium
        - Sliding Window: ⭐⭐⭐ Complex (track timestamps)
        - Fixed Window: ⭐ Simple
        - Leaky Bucket: ⭐⭐ Medium
    5. **Best for:**
        - Token Bucket: General purpose, allow bursts
        - Sliding Window: Accurate rate limiting
        - Fixed Window: Simple cases, low memory
        - Leaky Bucket: Smooth processing rate
- [x] Đọc về "rate limiting headers" - X-RateLimit-*
    - **Rate limiting headers** cung cấp thông tin về rate limits cho client.
    - **Standard headers:**
        - `X-RateLimit-Limit`: total limit (ví dụ: 100)
        - `X-RateLimit-Remaining`: remaining requests (ví dụ: 95)
        - `X-RateLimit-Reset`: timestamp khi limit reset (ví dụ: 1640995200)
        - `X-RateLimit-Used`: số requests đã dùng (ví dụ: 5)
    - **429 Too Many Requests response:**
        - Status: `429 Too Many Requests`
        - Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining: 0`, `X-RateLimit-Reset`
        - Optional: `Retry-After: 60` (seconds to wait)
    - **Example response:**
        ```
        HTTP/1.1 429 Too Many Requests
        X-RateLimit-Limit: 100
        X-RateLimit-Remaining: 0
        X-RateLimit-Reset: 1640995200
        Retry-After: 60
        ```
- [x] Đọc về "rate limiting strategies" - per user, per IP, per API key
    - **Per user:**
        - Limit dựa trên user ID
        - Key: `rate_limit:{user_id}`
        - ✅ Fair (mỗi user có limit riêng)
        - ❌ Cần authentication
    - **Per IP:**
        - Limit dựa trên IP address
        - Key: `rate_limit:{ip_address}`
        - ✅ Không cần authentication
        - ❌ NAT problem (nhiều users cùng IP)
    - **Per API key:**
        - Limit dựa trên API key
        - Key: `rate_limit:{api_key}`
        - ✅ Flexible (different keys có limits khác nhau)
        - ✅ Business model (free vs paid tiers)
    - **Best practice:**
        - Combine: per user + per IP (defense in depth)
        - Per API key cho business tiers
- [x] Đọc về "rate limiting tiers" - different limits for different users
    - **Rate limiting tiers** là different limits cho different user types.
    - **Common tiers:**
        - **Free tier**: 100 requests/minute
        - **Basic tier**: 1000 requests/minute
        - **Premium tier**: 10000 requests/minute
        - **Enterprise**: unlimited hoặc very high limit
    - **Implementation:**
        - Store tier trong user profile hoặc API key metadata
        - Lookup tier khi check rate limit
        - Apply limit based on tier
    - **Example:**
        ```python
        def get_rate_limit(user_id):
            tier = get_user_tier(user_id)
            limits = {
                'free': 100,
                'basic': 1000,
                'premium': 10000
            }
            return limits.get(tier, 100)
        ```
- [x] Research: Rate limiting best practices
    1. ✅ **Use appropriate algorithm**: Token Bucket cho general, Sliding Window cho accurate
    2. ✅ **Set reasonable limits**: không quá strict (block legitimate users)
    3. ✅ **Provide rate limit headers**: `X-RateLimit-*` để client biết limits
    4. ✅ **Return 429 with Retry-After**: cho client biết khi nào retry
    5. ✅ **Use multiple strategies**: per user + per IP (defense in depth)
    6. ✅ **Monitor rate limit hits**: track số lần hit limit, identify abuse
    7. ✅ **Whitelist/blacklist**: whitelist trusted IPs, blacklist abusive IPs
    8. ✅ **Gradual enforcement**: warning trước khi hard limit
    9. ✅ **Distributed rate limiting**: dùng Redis để sync across servers
    10. ✅ **Document limits**: clear documentation về limits và tiers

### Backward Compatibility
- [x] Đọc về "backward compatibility" - what is it?
    - **Backward compatibility** là khả năng API mới vẫn hoạt động với clients cũ (không break existing clients).
    - **Why important:**
        - ✅ Clients không cần update ngay lập tức
        - ✅ Smooth migration path
        - ✅ Reduce support burden
        - ✅ Better developer experience
    - **Types:**
        - **Full backward compatible**: clients cũ hoạt động hoàn toàn
        - **Partially backward compatible**: một số features không hoạt động
        - **Not backward compatible**: breaking changes
- [x] Đọc về "breaking changes" - what breaks compatibility?
    - **Breaking changes** là changes làm clients cũ không hoạt động được.
    - **Common breaking changes:**
        1. **Remove field**: client expect field nhưng không có → error
        2. **Change field type**: `string` → `number` → client parse error
        3. **Remove endpoint**: client call endpoint không tồn tại → 404
        4. **Change required field**: field optional → required → validation error
        5. **Change HTTP method**: GET → POST → client call sai method
        6. **Change response structure**: nested → flat → client parse error
        7. **Change authentication**: auth method thay đổi → client không authenticate được
    - **How to avoid:**
        - Version APIs (`/v1/`, `/v2/`)
        - Deprecate trước khi remove
        - Support multiple versions cùng lúc
- [x] Đọc về "additive changes" - safe changes
    - **Additive changes** là changes không break clients cũ (safe changes).
    - **Safe changes:**
        1. ✅ **Add new field**: client cũ ignore, client mới dùng
        2. ✅ **Add new endpoint**: không affect clients cũ
        3. ✅ **Make field optional**: required → optional (client cũ vẫn OK)
        4. ✅ **Add new enum value**: client cũ vẫn parse được
        5. ✅ **Add new query parameter**: optional params không break
        6. ✅ **Extend response**: thêm fields vào response (client cũ ignore)
    - **Principle**: "Be liberal in what you accept, conservative in what you send"
    - **Best practice**: chỉ làm additive changes trong cùng version
- [x] Đọc về "field deprecation" - how to deprecate safely
    - **Field deprecation** là process loại bỏ field một cách an toàn.
    - **Process:**
        1. **Mark as deprecated**: thêm `deprecated: true` trong response
        2. **Add deprecation header**: `Deprecation: true`, `Sunset: <date>`
        3. **Document**: update docs, notify developers
        4. **Wait period**: đợi clients migrate (3-6 months)
        5. **Remove**: remove field trong version mới
    - **Example:**
        ```json
        {
          "id": 123,
          "old_field": "value",  // deprecated
          "new_field": "value"
        }
        ```
        Headers: `Deprecation: true`, `Sunset: Sat, 01 Jan 2024 00:00:00 GMT`
    - **Best practice:**
        - Provide migration guide
        - Give enough time (3-6 months)
        - Monitor usage (track deprecated field usage)
        - Remove trong major version bump
- [x] Đọc về "API evolution" - how to evolve APIs
    - **API evolution** là process phát triển API mà không break clients.
    - **Strategies:**
        1. **Versioning**: `/v1/`, `/v2/` - support multiple versions
        2. **Additive only**: chỉ thêm, không remove trong cùng version
        3. **Deprecation**: mark deprecated, remove sau
        4. **Gradual migration**: migrate clients từ từ
        5. **Sunset policy**: clear timeline cho deprecation
    - **Process:**
        1. Design API với extensibility (có thể thêm fields)
        2. Version APIs khi cần breaking changes
        3. Deprecate old versions với timeline
        4. Support multiple versions trong transition period
        5. Sunset old versions sau khi clients migrate
    - **Best practice:**
        - Plan for evolution từ đầu
        - Document versioning strategy
        - Communicate changes clearly
        - Provide migration tools/guides
- [x] Đọc về "versioning strategy" for backward compatibility
    - **Versioning strategy** quyết định cách version APIs để maintain backward compatibility.
    - **Approaches:**
        1. **Semantic versioning**: `v1.0.0`, `v1.1.0`, `v2.0.0`
           - Major: breaking changes
           - Minor: additive changes
           - Patch: bug fixes
        2. **API versioning**: `/v1/`, `/v2/`
           - Major versions: breaking changes
           - Support multiple versions cùng lúc
        3. **Field versioning**: version fields trong response
           - `field_v1`, `field_v2` trong cùng response
    - **Best practice:**
        - Use semantic versioning cho APIs
        - Support at least 2 versions (current + previous)
        - Deprecate old versions với timeline
        - Document version differences
- [x] Research: Backward compatibility best practices
    1. ✅ **Design for extensibility**: có thể thêm fields mà không break
    2. ✅ **Version APIs**: dùng versioning cho breaking changes
    3. ✅ **Additive only**: chỉ thêm trong cùng version
    4. ✅ **Deprecate properly**: mark deprecated, provide timeline
    5. ✅ **Monitor usage**: track deprecated features usage
    6. ✅ **Communicate changes**: notify developers về changes
    7. ✅ **Provide migration guides**: help developers migrate
    8. ✅ **Support multiple versions**: support 2-3 versions trong transition
    9. ✅ **Sunset policy**: clear timeline cho deprecation
    10. ✅ **Test backward compatibility**: test với clients cũ
- [x] Analyze: Trade-offs của backward compatibility (maintenance cost)
    - **Benefits:**
        - ✅ Better developer experience (clients không cần update ngay)
        - ✅ Reduce support burden (ít breaking issues)
        - ✅ Smooth migration (gradual migration)
        - ✅ Customer retention (không force clients update)
    - **Costs:**
        - ❌ **Maintenance overhead**: phải maintain multiple versions
        - ❌ **Code complexity**: support multiple code paths
        - ❌ **Testing complexity**: test multiple versions
        - ❌ **Documentation**: document multiple versions
        - ❌ **Technical debt**: accumulate old code
    - **Trade-offs:**
        - Support 2-3 versions là reasonable
        - Deprecate old versions sau 6-12 months
        - Balance giữa compatibility và innovation
        - Clear deprecation policy

### API Gateway Patterns
- [x] Đọc về "API Gateway" pattern
    - **API Gateway** là single entry point cho tất cả API requests, đứng trước multiple backend services.
    - **Pattern:**
        - Client → API Gateway → Backend Services
        - Gateway handle cross-cutting concerns (auth, rate limiting, routing)
        - Backend services focus on business logic
    - **Benefits:**
        - ✅ Single entry point (simplify client)
        - ✅ Centralized cross-cutting concerns
        - ✅ Protocol translation (HTTP → gRPC)
        - ✅ Request/response transformation
        - ✅ Service aggregation
    - **Use case**: microservices architecture, multiple services cần unified API.
- [x] Đọc về "API Gateway responsibilities" - routing, auth, rate limiting, etc.
    - **Core responsibilities:**
        1. **Routing**: route requests đến đúng service dựa trên URL/path
        2. **Authentication**: validate tokens, authenticate users
        3. **Authorization**: check permissions, enforce access control
        4. **Rate limiting**: limit requests per user/IP/API key
        5. **Request/Response transformation**: transform data format
        6. **Protocol translation**: HTTP → gRPC, REST → GraphQL
        7. **Load balancing**: distribute traffic đến service instances
        8. **Circuit breaker**: fail fast khi service down
        9. **Monitoring & Logging**: track metrics, log requests
        10. **SSL termination**: handle HTTPS, backend chỉ cần HTTP
    - **Cross-cutting concerns**: concerns áp dụng cho tất cả requests.
- [x] Đọc về "BFF" (Backend for Frontend) pattern
    - **BFF (Backend for Frontend)** là dedicated backend cho mỗi frontend type (web, mobile, desktop).
    - **Pattern:**
        - Web BFF → Web Frontend
        - Mobile BFF → Mobile App
        - Desktop BFF → Desktop App
        - Mỗi BFF optimize cho frontend cụ thể
    - **Why needed:**
        - Different frontends có different needs
        - Web cần HTML, mobile cần JSON
        - Different data requirements
        - Different performance requirements
    - **Benefits:**
        - ✅ Optimized cho từng frontend
        - ✅ Reduce over-fetching (chỉ fetch data cần)
        - ✅ Better performance (aggregate data)
        - ✅ Frontend-specific logic
    - **Trade-offs:**
        - ❌ More services to maintain
        - ❌ Code duplication (nếu logic giống nhau)
- [x] So sánh: API Gateway vs BFF (5 differences)
    1. **Purpose:**
        - API Gateway: unified entry point cho tất cả clients
        - BFF: dedicated backend cho từng frontend type
    2. **Client focus:**
        - API Gateway: generic, serve multiple clients
        - BFF: specific, optimize cho 1 frontend type
    3. **Customization:**
        - API Gateway: generic, ít customization
        - BFF: highly customized cho frontend
    4. **Number of instances:**
        - API Gateway: 1 instance cho tất cả clients
        - BFF: multiple instances (1 per frontend type)
    5. **Use case:**
        - API Gateway: microservices, unified API
        - BFF: multiple frontends với different needs
- [x] Đọc về "service mesh" - alternative to API Gateway?
    - **Service Mesh** là infrastructure layer cho service-to-service communication.
    - **Components:**
        - **Sidecar proxy**: proxy chạy cùng mỗi service
        - **Control plane**: manage và configure proxies
        - **Data plane**: handle actual traffic
    - **Features:**
        - Service discovery
        - Load balancing
        - Circuit breaking
        - Retry/timeout
        - Security (mTLS)
        - Observability (metrics, tracing)
    - **API Gateway vs Service Mesh:**
        - **API Gateway**: north-south traffic (client → services)
        - **Service Mesh**: east-west traffic (service → service)
        - **Can use both**: Gateway cho external, Mesh cho internal
    - **Popular solutions**: Istio, Linkerd, Consul Connect.
- [x] Research: Popular API Gateway solutions (Kong, AWS API Gateway, Spring Cloud Gateway)
    - **Kong:**
        - ✅ Open-source, self-hosted
        - ✅ Plugin ecosystem (auth, rate limiting, logging)
        - ✅ High performance (Nginx-based)
        - ✅ On-premise hoặc cloud
        - ❌ Cần maintain infrastructure
        - **Use case**: self-hosted, need control
    - **AWS API Gateway:**
        - ✅ Managed service, không cần maintain
        - ✅ Tích hợp với AWS services (Lambda, EC2)
        - ✅ Auto-scaling, pay-per-use
        - ✅ Serverless support
        - ❌ Vendor lock-in, chi phí theo usage
        - **Use case**: AWS-based applications, serverless
    - **Spring Cloud Gateway:**
        - ✅ Java-based, integrate với Spring ecosystem
        - ✅ Reactive (WebFlux), high performance
        - ✅ Programmatic configuration
        - ✅ Filter-based architecture
        - ❌ Java-only, cần maintain
        - **Use case**: Spring Boot applications, Java ecosystem
    - **Other solutions:**
        - **Nginx**: có thể làm API Gateway với config
        - **HAProxy**: load balancer có thể extend thành Gateway
        - **Traefik**: modern reverse proxy với API Gateway features
        - **Ambassador**: Kubernetes-native API Gateway

---

## Traffic Routing TODOs

### Design Exercise 1: Load Balancer Architecture
- [ ] Design: Load balancer architecture cho Payment API
- [ ] Requirement: 10K QPS, 5 backend servers, 99.9% availability
- [ ] Choose: L4 or L7 load balancer? Justify
- [ ] Choose: Load balancing algorithm? Justify
- [ ] Design: Health check strategy (interval, timeout, thresholds)
- [ ] Design: Failover strategy (automatic removal, re-addition)
- [ ] Design: Sticky sessions? If yes, justify. If no, justify.
- [ ] Calculate: Load per backend server
- [ ] Design: Monitoring strategy
- [ ] Document: Complete load balancer design (500 words)

### Design Exercise 2: Multi-Tier Load Balancing
- [ ] Design: Multi-tier load balancing architecture
- [ ] Tier 1: External load balancer (ALB/Nginx)
- [ ] Tier 2: Internal load balancer (application layer)
- [ ] Design: Traffic flow through tiers
- [ ] Design: Health check at each tier
- [ ] Analyze: Single point of failure at each tier
- [ ] Design: Redundancy strategy
- [ ] Document: Multi-tier architecture (500 words)

### Design Exercise 3: Health Check Strategy
- [ ] Design: Health check strategy cho microservices
- [ ] Requirement: Fast failure detection (< 5 seconds)
- [ ] Design: Health check endpoint (/health)
- [ ] Design: Liveness probe vs Readiness probe
- [ ] Design: Health check interval và timeout
- [ ] Design: Success/failure thresholds
- [ ] Design: Graceful shutdown integration
- [ ] Test: Health check với various failure scenarios
- [ ] Document: Health check strategy

### Design Exercise 4: Sticky Session Strategy
- [ ] Scenario: E-commerce cart, need sticky sessions
- [ ] Design: Sticky session implementation
- [ ] Choose: Cookie-based or IP-based? Justify
- [ ] Design: Session persistence configuration
- [ ] Design: Fallback strategy (if sticky session fails)
- [ ] Analyze: Trade-offs của sticky sessions
- [ ] Design: Alternative - stateless cart (if possible)
- [ ] Compare: Sticky sessions vs stateless design
- [ ] Document: Session strategy với justification

### Design Exercise 5: Traffic Splitting
- [ ] Design: Traffic splitting strategy (A/B testing, canary)
- [ ] Scenario: Deploy new API version, route 10% traffic
- [ ] Design: Traffic routing rules
- [ ] Design: Monitoring cho each traffic split
- [ ] Design: Rollback strategy
- [ ] Design: Gradual rollout plan (10% → 50% → 100%)
- [ ] Document: Traffic splitting strategy

---

## API Design TODOs

### Design Exercise 1: RESTful Payment API
- [ ] Design: Complete RESTful API cho Payment System
- [ ] Design: Resource URLs (payments, merchants, transactions)
- [ ] Design: HTTP methods cho each operation
- [ ] Design: Request/response schemas
- [ ] Design: HTTP status codes cho each scenario
- [ ] Design: Error response format
- [ ] Design: Pagination strategy
- [ ] Design: Filtering và sorting
- [ ] Verify: RESTful principles compliance
- [ ] Document: Complete API specification

### Design Exercise 2: API Versioning Strategy
- [ ] Design: API versioning strategy cho Payment API
- [ ] Choose: Versioning approach (URL, Header, Query)? Justify
- [ ] Design: Version lifecycle (deprecation, sunset)
- [ ] Design: Backward compatibility policy
- [ ] Design: Breaking change policy
- [ ] Design: Migration path for clients
- [ ] Design: Version documentation strategy
- [ ] Document: Versioning strategy (500 words)

### Design Exercise 3: Idempotent API Design
- [ ] Design: Idempotent payment creation API
- [ ] Design: Idempotency key mechanism
- [ ] Design: Idempotency key format
- [ ] Design: Idempotency key storage (Redis? Database?)
- [ ] Design: Idempotency key TTL
- [ ] Design: Duplicate request handling
- [ ] Design: Response for duplicate requests
- [ ] Test: Idempotency với concurrent requests
- [ ] Document: Idempotency implementation

### Design Exercise 4: Rate Limiting Strategy
- [ ] Design: Rate limiting strategy cho Payment API
- [ ] Design: Rate limits per merchant tier (Basic: 100/min, Premium: 1000/min)
- [ ] Design: Rate limiting algorithm (Token Bucket? Sliding Window?)
- [ ] Design: Rate limit headers (X-RateLimit-*)
- [ ] Design: Rate limit error response (429 Too Many Requests)
- [ ] Design: Rate limit storage (Redis?)
- [ ] Design: Distributed rate limiting (across multiple gateways)
- [ ] Document: Rate limiting strategy

### Design Exercise 5: Pagination Design
- [ ] Design: Pagination strategy cho transaction list API
- [ ] Requirement: Support 1M+ transactions
- [ ] Compare: Offset-based vs Cursor-based pagination
- [ ] Choose: Pagination approach? Justify
- [ ] Design: Pagination parameters
- [ ] Design: Pagination response format
- [ ] Design: Performance optimization (indexes, query optimization)
- [ ] Document: Pagination design

### Design Exercise 6: Error Handling Design
- [ ] Design: Error response format
- [ ] Design: Error codes và messages
- [ ] Design: HTTP status code mapping
- [ ] Design: Validation error format
- [ ] Design: Business logic error format
- [ ] Design: System error format (500 errors)
- [ ] Design: Error logging strategy
- [ ] Document: Error handling design

### Design Exercise 7: Backward Compatibility Plan
- [ ] Scenario: Need to add new field to Payment response
- [ ] Design: Backward compatible change
- [ ] Scenario: Need to remove deprecated field
- [ ] Design: Deprecation strategy
- [ ] Scenario: Need to change field type
- [ ] Design: Breaking change migration plan
- [ ] Design: Client communication strategy
- [ ] Document: Backward compatibility strategy

---

## Gateway & Spring Boot TODOs

### Task 1: Spring Cloud Gateway Setup
- [ ] Create: Spring Boot project với Spring Cloud Gateway
- [ ] Configure: Gateway routes
- [ ] Configure: Route to multiple backend services
- [ ] Test: Basic routing functionality
- [ ] Configure: Gateway port và context path
- [ ] Document: Gateway setup

### Task 2: Load Balancing với Gateway
- [ ] Configure: Multiple instances của backend service
- [ ] Configure: Load balancing trong Gateway (Round-robin)
- [ ] Test: Requests distributed across instances
- [ ] Configure: Health checks cho backend services
- [ ] Test: Unhealthy instance removed from pool
- [ ] Test: Healthy instance re-added to pool
- [ ] Measure: Load distribution
- [ ] Document: Load balancing configuration

### Task 3: Rate Limiting Filter
- [ ] Implement: Rate limiting filter trong Gateway
- [ ] Algorithm: Token Bucket hoặc Sliding Window
- [ ] Configure: Rate limits per route
- [ ] Configure: Rate limits per user/IP
- [ ] Test: Rate limit enforcement
- [ ] Test: Rate limit headers (X-RateLimit-*)
- [ ] Test: 429 response khi limit exceeded
- [ ] Measure: Rate limiting overhead
- [ ] Document: Rate limiting implementation

### Task 4: Authentication & Authorization Filter
- [ ] Implement: JWT authentication filter
- [ ] Validate: JWT token trong Gateway
- [ ] Extract: User information từ token
- [ ] Add: User info to request headers
- [ ] Implement: Authorization checks (role-based)
- [ ] Test: Unauthenticated requests rejected
- [ ] Test: Unauthorized requests rejected
- [ ] Document: Auth filter implementation

### Task 5: Request/Response Transformation
- [ ] Implement: Request transformation filter
- [ ] Transform: Request headers
- [ ] Transform: Request body (if needed)
- [ ] Implement: Response transformation filter
- [ ] Transform: Response headers
- [ ] Transform: Response body (if needed)
- [ ] Test: Transformations work correctly
- [ ] Document: Transformation filters

### Task 6: Circuit Breaker Integration
- [ ] Add: Resilience4j circuit breaker dependency
- [ ] Configure: Circuit breaker cho backend routes
- [ ] Configure: Failure threshold và timeout
- [ ] Configure: Fallback responses
- [ ] Test: Circuit opens on failures
- [ ] Test: Fallback responses returned
- [ ] Test: Circuit closes after recovery
- [ ] Document: Circuit breaker configuration

### Task 7: API Versioning với Gateway
- [ ] Design: API versioning strategy
- [ ] Implement: Version routing (URL-based: /v1/, /v2/)
- [ ] Configure: Routes cho each version
- [ ] Test: Version routing works
- [ ] Implement: Header-based versioning (alternative)
- [ ] Compare: URL vs Header versioning implementation
- [ ] Document: Versioning implementation

### Task 8: Request Logging & Monitoring
- [ ] Implement: Request logging filter
- [ ] Log: Request method, URL, headers, body
- [ ] Log: Response status, latency
- [ ] Add: Correlation ID to requests
- [ ] Integrate: Với Micrometer metrics
- [ ] Expose: Gateway metrics (request count, latency, errors)
- [ ] Create: Dashboard với key metrics
- [ ] Document: Logging và monitoring setup

### Task 9: Idempotency Filter
- [ ] Implement: Idempotency filter trong Gateway
- [ ] Extract: Idempotency key từ header (Idempotency-Key)
- [ ] Check: Idempotency key trong Redis
- [ ] Store: Idempotency key với response
- [ ] Return: Cached response nếu duplicate request
- [ ] Configure: Idempotency key TTL
- [ ] Test: Duplicate requests return same response
- [ ] Document: Idempotency filter implementation

### Task 10: Health Check Endpoint
- [ ] Implement: Gateway health check endpoint
- [ ] Check: Gateway itself health
- [ ] Check: Backend services health (aggregate)
- [ ] Return: Overall health status
- [ ] Integrate: Với load balancer health checks
- [ ] Test: Health check endpoint
- [ ] Document: Health check implementation

---

## Reliability TODOs

### Failure Scenario 1: Backend Service Failure
- [ ] Simulate: Backend service failure
- [ ] Test: Load balancer removes unhealthy backend
- [ ] Measure: Failover time
- [ ] Test: Traffic redistributed to healthy backends
- [ ] Test: Circuit breaker behavior
- [ ] Test: Fallback responses
- [ ] Measure: Impact on users (errors, latency)
- [ ] Document: Failure handling

### Failure Scenario 2: Load Balancer Failure
- [ ] Simulate: Primary load balancer failure
- [ ] Design: Load balancer redundancy
- [ ] Test: Failover to secondary load balancer
- [ ] Measure: Failover time
- [ ] Test: DNS failover (if applicable)
- [ ] Test: Health check failover
- [ ] Document: Load balancer failover strategy

### Failure Scenario 3: Partial Backend Failure
- [ ] Simulate: One backend slow (high latency)
- [ ] Test: Load balancer behavior (Least Connections?)
- [ ] Measure: Impact on overall latency
- [ ] Test: Circuit breaker opens for slow backend
- [ ] Test: Traffic stops going to slow backend
- [ ] Document: Partial failure handling

### Failure Scenario 4: Rate Limit Exceeded
- [ ] Simulate: Rate limit exceeded scenario
- [ ] Test: 429 response returned
- [ ] Test: Rate limit headers included
- [ ] Test: Retry-After header (if applicable)
- [ ] Test: Client retry behavior
- [ ] Measure: Impact on legitimate users
- [ ] Document: Rate limit handling

### Failure Scenario 5: Sticky Session Failure
- [ ] Simulate: Backend với sticky session fails
- [ ] Test: Session lost? User impact?
- [ ] Design: Session recovery strategy
- [ ] Test: Alternative - stateless design
- [ ] Compare: Sticky sessions vs stateless
- [ ] Document: Session failure handling

### Failure Scenario 6: Health Check False Positive
- [ ] Simulate: Health check returns false positive (healthy but actually failing)
- [ ] Test: Impact on system
- [ ] Design: Better health check (more comprehensive)
- [ ] Test: Improved health check
- [ ] Document: Health check improvements

### Load Testing
- [ ] Setup: Load testing tool (JMeter, Gatling)
- [ ] Test: Load balancer với increasing load (1K, 5K, 10K QPS)
- [ ] Measure: Latency at each load level
- [ ] Measure: Error rate at each load level
- [ ] Identify: Breaking point
- [ ] Identify: Bottlenecks
- [ ] Optimize: Configuration
- [ ] Re-test: After optimization
- [ ] Document: Load test results

### Stress Testing
- [ ] Test: System behavior under extreme load (2x normal)
- [ ] Measure: Degradation patterns
- [ ] Test: Recovery after load spike
- [ ] Identify: Failure modes
- [ ] Document: Stress test results

---

## Review TODOs

### Self-Evaluation
- [ ] Review: Tất cả Study TODOs hoàn thành?
- [ ] Review: Tất cả Traffic Routing exercises hoàn thành?
- [ ] Review: Tất cả API Design exercises hoàn thành?
- [ ] Review: Tất cả Gateway tasks hoàn thành?
- [ ] Review: Tất cả Reliability scenarios tested?
- [ ] Rate: Load balancing understanding (1-10)
- [ ] Rate: API design skills (1-10)
- [ ] Rate: Gateway implementation skills (1-10)
- [ ] Rate: Reliability engineering skills (1-10)
- [ ] Identify: 3 concepts bạn master
- [ ] Identify: 3 concepts cần improve
- [ ] Plan: How to improve weak areas

### API Design Review
- [ ] Review: Payment API design
- [ ] Check: RESTful principles compliance?
- [ ] Check: Versioning strategy appropriate?
- [ ] Check: Idempotency implemented?
- [ ] Check: Error handling adequate?
- [ ] Check: Backward compatibility considered?
- [ ] Identify: 3 API design improvements
- [ ] Propose: API design optimizations
- [ ] Compare: Design vs industry best practices
- [ ] Document: API design review findings

### Architecture Review
- [ ] Review: Load balancer architecture
- [ ] Check: Single points of failure eliminated?
- [ ] Check: Health checks adequate?
- [ ] Check: Failover strategy robust?
- [ ] Check: Monitoring comprehensive?
- [ ] Identify: 3 architecture improvements
- [ ] Propose: Architecture optimizations
- [ ] Document: Architecture review

### Code Review
- [ ] Review: Gateway implementation code
- [ ] Check: Error handling adequate?
- [ ] Check: Logging và monitoring?
- [ ] Check: Configuration management?
- [ ] Review: Rate limiting implementation
- [ ] Review: Idempotency implementation
- [ ] Identify: 3 code improvements
- [ ] Refactor: At least 1 piece of code
- [ ] Document: Code review findings

### Performance Review
- [ ] Review: Load test results
- [ ] Review: Stress test results
- [ ] Verify: Performance targets met?
- [ ] Identify: Performance bottlenecks
- [ ] Analyze: Load balancer overhead
- [ ] Analyze: Gateway overhead
- [ ] Create: Performance improvement plan
- [ ] Document: Performance review

### Knowledge Check
- [ ] Explain: L4 vs L7 load balancing (viết comparison, không xem notes)
- [ ] Explain: Round-Robin vs Least Connections (viết comparison, không xem notes)
- [ ] Explain: URL vs Header versioning (viết comparison, không xem notes)
- [ ] Explain: Idempotency - why important? (viết explanation, không xem notes)
- [ ] Explain: Rate limiting algorithms (viết comparison, không xem notes)
- [ ] Solve: 10K QPS, 5 backends, Round-Robin. Calculate QPS per backend
- [ ] Solve: Rate limit = 100/min, 120 requests in 1 minute. How many rejected?
- [ ] Verify: Answers có đúng không?
- [ ] Document: Knowledge gaps

### Reflection
- [ ] Write: 3 điều học được quan trọng nhất tuần này
- [ ] Write: 2 API design concepts còn confuse
- [ ] Write: 1 architecture decision bạn made và rationale
- [ ] Write: Confidence level cho Week 7 (1-10)
- [ ] Compare: Week 5 vs Week 6 progress
- [ ] Plan: Preparation cho Week 7 (Message Queues)
- [ ] Set: Goals cho Week 7
- [ ] Document: Week 6 reflection (500 words)

### Mentor Questions (Answer these - challenge assumptions)
- [ ] Q1: Bạn design API với URL versioning (/v1/, /v2/). Client cũ không update, vẫn call /v1/. Bạn cần deprecate /v1/ sau 1 năm. Làm sao communicate và enforce? (viết deprecation strategy)
- [ ] Q2: Load balancer với Round-Robin, 5 backends. Backend 1 chậm (500ms), others fast (50ms). Average latency? Làm sao fix? (viết analysis và solution)
- [ ] Q3: API có rate limit 100/min. User có 10 parallel requests, mỗi request takes 1 second. Có violate rate limit không? Tại sao? (viết analysis)
- [ ] Q4: Sticky sessions - user A always goes to Backend 1. Backend 1 có 10x load của others. Làm sao fix? Justify solution. (viết solution với trade-off analysis)
- [ ] Q5: API Gateway adds 10ms latency. Worth it? Khi nào worth it? Khi nào không? (viết cost-benefit analysis)
- [ ] Q6: Idempotency key stored trong Redis, TTL = 24 hours. 1M requests/day, 1% duplicates. Redis memory needed? (calculate và analyze)
- [ ] Review: Answers có challenge assumptions? Có consider edge cases? Có justify decisions?
- [ ] Improve: Answers nếu cần

---

## Final Checklist

- [ ] Tất cả Study TODOs: ✅ Complete
- [ ] Tất cả Traffic Routing TODOs: ✅ Complete với designs
- [ ] Tất cả API Design TODOs: ✅ Complete với specifications
- [ ] Tất cả Gateway & Spring Boot TODOs: ✅ Complete, tested, và documented
- [ ] Tất cả Reliability TODOs: ✅ Complete với test results
- [ ] Tất cả Review TODOs: ✅ Complete
- [ ] Reflection document: ✅ Written
- [ ] Code committed: ✅ Yes
- [ ] API specifications saved: ✅ Yes
- [ ] Load test results saved: ✅ Yes
- [ ] Ready for Week 7: ✅ Yes

---

> **Mentor Final Check**: API design và load balancing là foundation. Nếu foundation weak, everything trên đó sẽ be weak. Hãy honest: Bạn có thể design production-ready API chưa? Bạn có thể configure load balancer cho high availability chưa? Bạn có thể handle all failure scenarios chưa? Nếu chưa, làm lại. Production systems need robust foundations, not "it works on my machine".
