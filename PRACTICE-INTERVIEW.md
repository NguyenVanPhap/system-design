# Practice Technical Interview Questions

File này chứa các câu hỏi practice cho technical interview, bao gồm:
1. **Code Output Questions**: Cho đoạn code, xác định output hoặc thay đổi code để đạt output mong muốn
2. **Code Review Questions**: Review code, trả lời câu hỏi, hoặc propose solution

---

## Phần 1: Code Output Questions

### Câu hỏi 1: Transaction và Concurrency

**Đoạn code:**
```java
@Service
@Transactional
public class WalletService {
    @Autowired
    private WalletRepository walletRepository;
    
    public void deposit(Long memberId, BigDecimal amount) {
        Wallet wallet = walletRepository.findByMemberId(memberId);
        BigDecimal currentBalance = wallet.getBalance();
        wallet.setBalance(currentBalance.add(amount));
        walletRepository.save(wallet);
        
        // Simulate external API call
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        log.info("Deposit completed. New balance: {}", wallet.getBalance());
    }
}
```

**Câu hỏi:**
1. Nếu có 2 threads cùng gọi `deposit(memberId=1, amount=100)` đồng thời, balance cuối cùng có thể là bao nhiêu? Giải thích.
2. Nếu muốn đảm bảo balance cuối cùng luôn đúng (200 nếu ban đầu = 0), cần sửa code như thế nào?

<details>
<summary>Đáp án</summary>

**1. Balance cuối cùng có thể là:**
- **100** (race condition - lost update)
- **200** (nếu may mắn không có race condition)

**Giải thích:**
- Thread 1 đọc balance = 0
- Thread 2 đọc balance = 0 (trước khi Thread 1 save)
- Thread 1 tính newBalance = 0 + 100 = 100
- Thread 2 tính newBalance = 0 + 100 = 100
- Thread 1 save balance = 100
- Thread 2 save balance = 100 (ghi đè)
- Kết quả: balance = 100 thay vì 200

**2. Solutions:**

**Option 1: Pessimistic Locking**
```java
@Transactional
public void deposit(Long memberId, BigDecimal amount) {
    Wallet wallet = walletRepository.findByMemberIdForUpdate(memberId);
    BigDecimal currentBalance = wallet.getBalance();
    wallet.setBalance(currentBalance.add(amount));
    walletRepository.save(wallet);
}
```

**Option 2: Optimistic Locking với @Version**
```java
@Entity
public class Wallet {
    @Version
    private Long version;
    // ...
}

@Transactional
public void deposit(Long memberId, BigDecimal amount) {
    Wallet wallet = walletRepository.findByMemberId(memberId);
    BigDecimal currentBalance = wallet.getBalance();
    wallet.setBalance(currentBalance.add(amount));
    walletRepository.save(wallet); // Nếu version changed, throw OptimisticLockException
}
```

**Option 3: Database-level atomic update**
```java
@Modifying
@Query("UPDATE Wallet w SET w.balance = w.balance + :amount WHERE w.memberId = :memberId")
void incrementBalance(@Param("memberId") Long memberId, @Param("amount") BigDecimal amount);
```

**Option 4: Ledger-based approach**
```java
@Transactional
public void deposit(Long memberId, BigDecimal amount) {
    // Insert transaction record
    Transaction tx = new Transaction();
    tx.setMemberId(memberId);
    tx.setAmount(amount);
    tx.setType(TransactionType.DEPOSIT);
    transactionRepository.save(tx);
    
    // Update balance atomically
    walletRepository.incrementBalance(memberId, amount);
}
```

**📚 Giải thích chi tiết về Ledger-based Approach:**

**1️⃣ Ledger-based approach là gì?**

Thay vì chỉ lưu balance, ta lưu thêm **Transaction Ledger (sổ cái)** = lịch sử tất cả giao dịch.

Ví dụ table transaction:
```
id | member_id | amount | type    | created_at
1  | 1001      | +50    | DEPOSIT | 10:01
2  | 1001      | -20    | BET     | 10:02
3  | 1001      | +30    | WIN     | 10:03
```

Balance = SUM(amount), nhưng để query nhanh → vẫn cache balance trong wallet.

**2️⃣ Vì sao Option 4 rất mạnh?**

Code kết hợp:
- ✅ **Audit log (ledger)**: Lưu lịch sử giao dịch
- ✅ **Atomic update**: DB đảm bảo thread-safe
- ✅ **Transaction boundary**: Đảm bảo ACID

→ Đây là chuẩn fintech/banking.

**3️⃣ incrementBalance hoạt động thế nào?**

Implementation:
```java
@Modifying
@Query("""
UPDATE Wallet w
SET w.balance = w.balance + :amount
WHERE w.memberId = :memberId
""")
int incrementBalance(@Param("memberId") Long memberId,
                     @Param("amount") BigDecimal amount);
```

SQL thực tế:
```sql
UPDATE wallet
SET balance = balance + 100
WHERE member_id = 1;
```

👉 DB đảm bảo:
- Atomic
- Thread-safe
- Không race condition
- Không lost update

**4️⃣ Concurrency xử lý thế nào?**

Giả sử 10 request cùng deposit:
```sql
UPDATE wallet SET balance = balance + 10
```

DB sẽ serialize nội bộ:
```
+10 → +10 → +10 → ...
```

➡️ Kết quả đúng 100%. Không cần `@Version` hay `FOR UPDATE`.

**5️⃣ Vì sao vẫn cần ledger?**

Nếu chỉ dùng:
```sql
UPDATE wallet SET balance = balance + ?
```

→ Bạn không biết:
- Tiền từ đâu ra?
- Ai cộng?
- Khi nào?
- Trace bug thế nào?

Ledger cho bạn:
- ✅ Audit trail
- ✅ Reconcile
- ✅ Debug
- ✅ Compliance
- ✅ Rollback logic

**6️⃣ Transaction ở đây cực kỳ quan trọng**

`@Transactional` đảm bảo:
```
Insert TX + Update Wallet
   ↓
All-or-nothing
```

Nếu crash giữa chừng → rollback.

❗ Nếu thiếu `@Transactional`:
- TX insert OK
- Balance chưa update ❌
- → Data sai vĩnh viễn

**7️⃣ Chuẩn nâng cao: Idempotency (rất quan trọng với callback/payment)**

Trong betting/payment, callback có thể gửi lại nhiều lần.

👉 Phải chống double deposit.

Thêm unique key:
```java
@Entity
@Table(
  uniqueConstraints = {
    @UniqueConstraint(columnNames = {"external_tx_id"})
  }
)
class Transaction {
   private String externalTxId;
}
```

Khi insert trùng → fail → ignore.

**8️⃣ Xử lý rollback / cancel**

Ledger cho phép rollback đúng chuẩn:

Ví dụ cancel bet:
```java
// insert reversal tx
tx.amount = -100;
tx.type = CANCEL;
save(tx);

// update balance
incrementBalance(memberId, -100);
```

Không bao giờ sửa record cũ ❌  
Chỉ append record mới ✅

→ Financial correctness.

**9️⃣ Kiến trúc thực tế (Real system)**

Trong hệ thống betting lớn:
```
API
 ↓
Wallet Service
 ↓
--------------------------------
|  Transaction Table (Ledger)  |
|  Wallet Table (Snapshot)     |
--------------------------------
```

Luồng:
1. Validate
2. Check idempotent
3. Insert ledger
4. Update balance
5. Commit

**🔟 So sánh 4 options**

| Option | An toàn | Scale | Audit | Dùng cho tiền |
|--------|---------|-------|-------|---------------|
| Pessimistic | ⭐⭐⭐⭐ | ❌ | ❌ | ⚠️ |
| Optimistic | ⭐⭐ | ⭐⭐⭐ | ❌ | ❌ |
| Atomic SQL | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ | ⚠️ |
| **Ledger + Atomic** | **⭐⭐⭐⭐⭐** | **⭐⭐⭐⭐⭐** | **✅** | **✅✅✅** |

👉 **Option 4 = Best Practice** cho hệ thống tài chính.

**1️⃣1️⃣ Best Practice cho Wallet (Pro level)**

Với domain betting/payout/callback/wallet:

✅ Ledger table  
✅ Atomic update  
✅ Idempotency key  
✅ Unique index  
✅ Transaction boundary

Đây là level production lớn.

**1️⃣2️⃣ Bản nâng cấp (Advanced)**

Nếu traffic rất lớn:

➡️ Event-driven:
```
API → Kafka → Wallet Consumer → DB
```

Ledger là source of truth.  
Nhưng nền tảng vẫn là Option 4.

**✅ Kết luận**

Option 4:
- ✔️ Đúng hướng
- ✔️ Production-grade
- ✔️ Scalable
- ✔️ Safe cho tiền

Chỉ cần thêm:
- 🔹 Unique tx id
- 🔹 Retry DB deadlock
- 🔹 Monitoring

là thành "bank-grade" rồi 😄

**⚠️ Lưu ý:**

❌ **Không hợp với:**
- CMS content
- Wallet real-time
- Betting
- Stock trading
- Jackpot

(vì conflict nhiều → retry liên tục → lag)

✅ **Hợp với:**
- Wallet/Balance operations
- Payment processing
- Financial transactions
- Audit-critical systems
</details>

---

### Câu hỏi 2: Spring Transaction Propagation

**Đoạn code:**
```java
@Service
public class PaymentService {
    @Autowired
    private WalletService walletService;
    
    @Autowired
    private TransactionService transactionService;
    
    @Transactional
    public void processPayment(Long memberId, BigDecimal amount) {
        walletService.deductBalance(memberId, amount);
        transactionService.recordTransaction(memberId, amount);
        
        if (amount.compareTo(BigDecimal.valueOf(1000)) > 0) {
            throw new RuntimeException("Amount too large");
        }
    }
}

@Service
public class WalletService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void deductBalance(Long memberId, BigDecimal amount) {
        Wallet wallet = walletRepository.findByMemberId(memberId);
        wallet.setBalance(wallet.getBalance().subtract(amount));
        walletRepository.save(wallet);
    }
}

@Service
public class TransactionService {
    @Transactional(propagation = Propagation.REQUIRED)
    public void recordTransaction(Long memberId, BigDecimal amount) {
        Transaction tx = new Transaction();
        tx.setMemberId(memberId);
        tx.setAmount(amount);
        transactionRepository.save(tx);
    }
}
```

**Câu hỏi:**
1. Nếu `processPayment(1L, 2000)` được gọi, trạng thái database sau khi exception xảy ra là gì?
2. Nếu đổi `REQUIRES_NEW` thành `REQUIRED` trong `deductBalance()`, kết quả thay đổi như thế nào?

<details>
<summary>Đáp án</summary>

**1. Trạng thái database:**
- **Balance đã bị deduct** (vì `REQUIRES_NEW` tạo transaction riêng, commit trước khi exception)
- **Transaction record KHÔNG được tạo** (vì exception xảy ra trước khi save)

**Giải thích:**
- `processPayment()` bắt đầu transaction T1
- `deductBalance()` với `REQUIRES_NEW` tạo transaction T2 riêng
- T2 commit → balance bị deduct
- `recordTransaction()` chạy trong T1
- Exception xảy ra → T1 rollback → transaction record không được tạo

**2. Nếu đổi thành `REQUIRED`:**
- **Balance KHÔNG bị deduct** (cùng transaction T1, rollback khi exception)
- **Transaction record KHÔNG được tạo** (cùng transaction T1, rollback)

**Giải thích:**
- `deductBalance()` và `recordTransaction()` cùng transaction T1
- Exception xảy ra → T1 rollback toàn bộ
</details>

---

### Câu hỏi 3: Async và Thread Pool

**Đoạn code:**
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(4);
        executor.setQueueCapacity(10);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class PaymentService {
    @Async
    public CompletableFuture<String> processPayment(Long memberId, BigDecimal amount) {
        log.info("Processing payment for member: {}", memberId);
        try {
            Thread.sleep(2000); // Simulate processing
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return CompletableFuture.completedFuture("Payment processed");
    }
}

@RestController
public class PaymentController {
    @Autowired
    private PaymentService paymentService;
    
    @PostMapping("/payment")
    public ResponseEntity<String> createPayment(@RequestBody PaymentRequest request) {
        for (int i = 0; i < 20; i++) {
            paymentService.processPayment(request.getMemberId(), request.getAmount());
        }
        return ResponseEntity.ok("Payment requests submitted");
    }
}
```

**Câu hỏi:**
1. Khi có 20 requests đồng thời đến `/payment`, có bao nhiêu tasks được execute ngay lập tức? Bao nhiêu tasks phải đợi?
2. Nếu muốn tất cả 20 tasks đều được execute (không reject), cần điều chỉnh config như thế nào?

<details>
<summary>Đáp án</summary>

**1. Với config hiện tại:**
- **Core pool size = 2**: 2 threads luôn sẵn sàng
- **Max pool size = 4**: Tối đa 4 threads
- **Queue capacity = 10**: Queue chứa tối đa 10 tasks

**Khi có 20 requests:**
- 2 tasks được execute ngay (core threads)
- 2 tasks nữa được execute (tăng lên max pool size = 4)
- 10 tasks vào queue
- **4 tasks bị reject** (vì queue đầy và không thể tạo thêm threads)

**2. Solutions:**

**Option 1: Tăng queue capacity**
```java
executor.setQueueCapacity(100); // Đủ cho 20 tasks
```

**Option 2: Tăng max pool size**
```java
executor.setCorePoolSize(5);
executor.setMaxPoolSize(20); // Đủ cho 20 concurrent tasks
```

**Option 3: Sử dụng CallerRunsPolicy (không reject)**
```java
executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
// Nếu queue đầy, task sẽ chạy trên caller thread (block caller)
```

**Option 4: Sử dụng Unbounded queue (không giới hạn)**
```java
executor.setQueueCapacity(Integer.MAX_VALUE); // Không giới hạn
// Lưu ý: Có thể gây memory issues nếu có quá nhiều tasks
```
</details>

---

### Câu hỏi 4: Spring Bean Scope và Singleton

**Đoạn code:**
```java
@Service
public class CounterService {
    private int count = 0;
    
    public void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}

@RestController
public class CounterController {
    @Autowired
    private CounterService counterService;
    
    @GetMapping("/increment")
    public ResponseEntity<Integer> increment() {
        counterService.increment();
        return ResponseEntity.ok(counterService.getCount());
    }
    
    @GetMapping("/count")
    public ResponseEntity<Integer> getCount() {
        return ResponseEntity.ok(counterService.getCount());
    }
}
```

**Câu hỏi:**
1. Nếu có 3 requests đồng thời đến `/increment`, giá trị `count` sau 3 requests là bao nhiêu? Giải thích.
2. Nếu muốn mỗi request có counter riêng, cần sửa code như thế nào?

<details>
<summary>Đáp án</summary>

**1. Giá trị `count` sau 3 requests:**
- **Có thể là 1, 2, hoặc 3** (race condition)
- **Hoặc có thể là 3** (nếu không có race condition)

**Giải thích:**
- `CounterService` là singleton (default Spring scope)
- Tất cả requests share cùng một instance
- `count++` không atomic → có thể xảy ra race condition:
  - Thread 1: read count = 0
  - Thread 2: read count = 0
  - Thread 3: read count = 0
  - Thread 1: write count = 1
  - Thread 2: write count = 1 (ghi đè)
  - Thread 3: write count = 1 (ghi đè)
  - Kết quả: count = 1 thay vì 3

**2. Solutions:**

**Option 1: Sử dụng AtomicInteger (thread-safe)**
```java
@Service
public class CounterService {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}
```

**Option 2: Synchronized method**
```java
@Service
public class CounterService {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

**Option 3: Request scope (mỗi request có instance riêng)**
```java
@Service
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class CounterService {
    private int count = 0;
    // ...
}
```

**Option 4: ThreadLocal (mỗi thread có giá trị riêng)**
```java
@Service
public class CounterService {
    private final ThreadLocal<Integer> count = ThreadLocal.withInitial(() -> 0);
    
    public void increment() {
        count.set(count.get() + 1);
    }
    
    public int getCount() {
        return count.get();
    }
}
```
</details>

---

### Câu hỏi 5: Java Stream và Lazy Evaluation

**Đoạn code:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> result = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n % 2 == 0;
    })
    .map(n -> {
        System.out.println("Mapping: " + n);
        return n * 2;
    })
    .limit(3)
    .collect(Collectors.toList());

System.out.println("Result: " + result);
```

**Câu hỏi:**
1. Output của đoạn code trên là gì? Giải thích thứ tự execution.
2. Nếu đổi `limit(3)` thành `limit(5)`, output thay đổi như thế nào?

<details>
<summary>Đáp án</summary>

**1. Output:**
```
Filtering: 1
Filtering: 2
Mapping: 2
Filtering: 3
Filtering: 4
Mapping: 4
Filtering: 5
Filtering: 6
Mapping: 6
Result: [4, 8, 12]
```

**Giải thích (Lazy Evaluation):**
- Stream operations là **lazy** (chỉ execute khi có terminal operation)
- `limit(3)` là **short-circuit** operation
- Execution flow:
  1. Filter 1 → false, skip
  2. Filter 2 → true → Map 2 → 4 → collect (1st element)
  3. Filter 3 → false, skip
  4. Filter 4 → true → Map 4 → 8 → collect (2nd element)
  5. Filter 5 → false, skip
  6. Filter 6 → true → Map 6 → 12 → collect (3rd element)
  7. **Stop** vì đã có 3 elements (limit reached)
  8. Không process 7, 8, 9, 10

**2. Nếu `limit(5)`:**
```
Filtering: 1
Filtering: 2
Mapping: 2
Filtering: 3
Filtering: 4
Mapping: 4
Filtering: 5
Filtering: 6
Mapping: 6
Filtering: 7
Filtering: 8
Mapping: 8
Filtering: 9
Filtering: 10
Mapping: 10
Result: [4, 8, 12, 16, 20]
```

**Key Points:**
- Stream processes elements **one by one** (pipeline)
- `limit()` stops processing khi đủ số lượng
- Không process toàn bộ list trước, chỉ process đủ để có kết quả
</details>

---

## Phần 2: Code Review Questions

### Câu hỏi 6: Code Review - Payment Processing

**Đoạn code cần review:**
```java
@Service
public class PaymentService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Autowired
    private TransactionRepository transactionRepository;
    
    @Autowired
    private EmailService emailService;
    
    public void processDeposit(Long memberId, BigDecimal amount) {
        // 1. Check balance
        Wallet wallet = walletRepository.findByMemberId(memberId);
        if (wallet == null) {
            throw new RuntimeException("Wallet not found");
        }
        
        // 2. Update balance
        BigDecimal newBalance = wallet.getBalance().add(amount);
        wallet.setBalance(newBalance);
        walletRepository.save(wallet);
        
        // 3. Create transaction record
        Transaction tx = new Transaction();
        tx.setMemberId(memberId);
        tx.setAmount(amount);
        tx.setType(TransactionType.DEPOSIT);
        tx.setStatus(TransactionStatus.COMPLETED);
        transactionRepository.save(tx);
        
        // 4. Send email notification
        emailService.sendDepositConfirmation(memberId, amount);
        
        // 5. Log
        log.info("Deposit processed: memberId={}, amount={}", memberId, amount);
    }
}
```

**Câu hỏi:**
1. Có những vấn đề gì trong đoạn code này?
2. Propose solution để fix các vấn đề.

<details>
<summary>Đáp án</summary>

**1. Các vấn đề:**

**a) Race Condition:**
- Nếu có 2 requests đồng thời, có thể xảy ra lost update
- `getBalance()` và `setBalance()` không atomic

**b) Transaction Management:**
- Không có `@Transactional` → nếu email service fail, balance đã được update nhưng transaction record chưa được tạo
- Không đảm bảo ACID properties

**c) Exception Handling:**
- Sử dụng generic `RuntimeException` thay vì custom exception
- Không có error handling cho email service failure

**d) Performance:**
- Email service là blocking call → làm chậm response time
- Nên xử lý async

**e) Data Consistency:**
- Balance và transaction record có thể không đồng bộ nếu có lỗi

**f) Validation:**
- Không validate `amount > 0`
- Không validate member status

**2. Proposed Solution:**

```java
@Service
@Transactional
public class PaymentService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Autowired
    private TransactionRepository transactionRepository;
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void processDeposit(Long memberId, BigDecimal amount) {
        // 1. Validation
        validateDepositRequest(memberId, amount);
        
        // 2. Atomic balance update
        walletRepository.incrementBalance(memberId, amount);
        
        // 3. Create transaction record (in same transaction)
        Transaction tx = new Transaction();
        tx.setMemberId(memberId);
        tx.setAmount(amount);
        tx.setType(TransactionType.DEPOSIT);
        tx.setStatus(TransactionStatus.COMPLETED);
        transactionRepository.save(tx);
        
        // 4. Publish event for async processing (email, logging, etc.)
        eventPublisher.publishEvent(new DepositCompletedEvent(memberId, amount, tx.getId()));
    }
    
    private void validateDepositRequest(Long memberId, BigDecimal amount) {
        if (memberId == null) {
            throw new IllegalArgumentException("Member ID cannot be null");
        }
        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be greater than 0");
        }
        
        Wallet wallet = walletRepository.findByMemberId(memberId);
        if (wallet == null) {
            throw new WalletNotFoundException("Wallet not found for member: " + memberId);
        }
    }
}

// Event listener for async processing
@Component
public class DepositEventListener {
    @Autowired
    private EmailService emailService;
    
    @Async
    @EventListener
    public void handleDepositCompleted(DepositCompletedEvent event) {
        try {
            emailService.sendDepositConfirmation(event.getMemberId(), event.getAmount());
            log.info("Deposit processed: memberId={}, amount={}, txId={}", 
                event.getMemberId(), event.getAmount(), event.getTransactionId());
        } catch (Exception e) {
            log.error("Failed to send deposit confirmation email", e);
            // Don't throw - email failure shouldn't affect deposit
        }
    }
}
```

**Improvements:**
- ✅ `@Transactional` đảm bảo ACID
- ✅ Atomic balance update (database-level)
- ✅ Async email processing (không block)
- ✅ Event-driven architecture (loose coupling)
- ✅ Proper validation và exception handling
- ✅ Error handling cho non-critical operations (email)
</details>

---

### Câu hỏi 7: Code Review - Thread Safety

**Đoạn code cần review:**
```java
@Service
public class CacheService {
    private Map<String, Object> cache = new HashMap<>();
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.get(key);
    }
    
    public void clear() {
        cache.clear();
    }
    
    public int size() {
        return cache.size();
    }
}
```

**Câu hỏi:**
1. Có vấn đề gì với đoạn code này trong môi trường multi-threaded?
2. Propose solution để fix.

<details>
<summary>Đáp án</summary>

**1. Vấn đề:**

**a) Thread Safety:**
- `HashMap` không thread-safe
- Concurrent modifications có thể gây:
  - `ConcurrentModificationException`
  - Data corruption
  - Infinite loops (trong một số trường hợp)

**b) Visibility:**
- Nếu không có synchronization, một thread có thể không thấy updates từ thread khác (memory visibility issue)

**c) Race Conditions:**
- `put()` và `get()` có thể xảy ra đồng thời → inconsistent state
- `size()` có thể không chính xác nếu có concurrent modifications

**2. Solutions:**

**Option 1: ConcurrentHashMap (Recommended)**
```java
@Service
public class CacheService {
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.get(key);
    }
    
    public void clear() {
        cache.clear();
    }
    
    public int size() {
        return cache.size();
    }
}
```

**Option 2: Synchronized methods**
```java
@Service
public class CacheService {
    private final Map<String, Object> cache = new HashMap<>();
    
    public synchronized void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public synchronized Object get(String key) {
        return cache.get(key);
    }
    
    public synchronized void clear() {
        cache.clear();
    }
    
    public synchronized int size() {
        return cache.size();
    }
}
```

**Option 3: ReadWriteLock (for better read performance)**
```java
@Service
public class CacheService {
    private final Map<String, Object> cache = new HashMap<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    public void put(String key, Object value) {
        lock.writeLock().lock();
        try {
            cache.put(key, value);
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    public Object get(String key) {
        lock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            lock.readLock().unlock();
        }
    }
}
```

**Option 4: Caffeine Cache (Production-ready)**
```java
@Service
public class CacheService {
    private final Cache<String, Object> cache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .build();
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.getIfPresent(key);
    }
}
```

**Recommendation:**
- Sử dụng **ConcurrentHashMap** cho simple use case
- Sử dụng **Caffeine** hoặc **Guava Cache** cho production (có TTL, size limit, statistics)
</details>

---

### Câu hỏi 8: Code Review - Resource Management

**Đoạn code cần review:**
```java
@Service
public class FileService {
    public String readFile(String filePath) {
        FileInputStream fis = null;
        BufferedReader reader = null;
        try {
            fis = new FileInputStream(filePath);
            reader = new BufferedReader(new InputStreamReader(fis));
            StringBuilder content = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                content.append(line).append("\n");
            }
            return content.toString();
        } catch (IOException e) {
            log.error("Error reading file: " + filePath, e);
            return null;
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
                if (fis != null) {
                    fis.close();
                }
            } catch (IOException e) {
                log.error("Error closing file", e);
            }
        }
    }
}
```

**Câu hỏi:**
1. Có những vấn đề gì trong đoạn code này?
2. Propose solution để fix.

<details>
<summary>Đáp án</summary>

**1. Vấn đề:**

**a) Resource Leak:**
- Nếu `reader.readLine()` throw exception, `fis` có thể không được close
- Nếu `reader.close()` throw exception, `fis` không được close

**b) Exception Swallowing:**
- Return `null` khi có exception → caller không biết lỗi gì
- Exception trong finally block bị log nhưng không propagate

**c) Code Complexity:**
- Nested try-catch blocks khó đọc và maintain

**d) Missing Validation:**
- Không check `filePath` null hoặc empty

**2. Solutions:**

**Option 1: Try-with-resources (Java 7+) - Recommended**
```java
@Service
public class FileService {
    public String readFile(String filePath) throws IOException {
        if (filePath == null || filePath.isEmpty()) {
            throw new IllegalArgumentException("File path cannot be null or empty");
        }
        
        try (FileInputStream fis = new FileInputStream(filePath);
             BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
            
            StringBuilder content = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                content.append(line).append("\n");
            }
            return content.toString();
        }
        // Resources automatically closed, even if exception occurs
    }
}
```

**Option 2: Using Files utility (Java 7+)**
```java
@Service
public class FileService {
    public String readFile(String filePath) throws IOException {
        if (filePath == null || filePath.isEmpty()) {
            throw new IllegalArgumentException("File path cannot be null or empty");
        }
        
        Path path = Paths.get(filePath);
        return Files.readString(path, StandardCharsets.UTF_8);
    }
}
```

**Option 3: Custom Exception**
```java
@Service
public class FileService {
    public String readFile(String filePath) throws FileReadException {
        if (filePath == null || filePath.isEmpty()) {
            throw new IllegalArgumentException("File path cannot be null or empty");
        }
        
        try (FileInputStream fis = new FileInputStream(filePath);
             BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
            
            StringBuilder content = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                content.append(line).append("\n");
            }
            return content.toString();
        } catch (IOException e) {
            throw new FileReadException("Failed to read file: " + filePath, e);
        }
    }
}
```

**Key Points:**
- ✅ Try-with-resources đảm bảo resources được close tự động
- ✅ Proper exception handling (không swallow exceptions)
- ✅ Input validation
- ✅ Cleaner code
</details>

---

### Câu hỏi 9: Code Review - N+1 Query Problem

**Đoạn code cần review:**
```java
@RestController
public class MemberController {
    @Autowired
    private MemberRepository memberRepository;
    
    @GetMapping("/members")
    public List<MemberDTO> getAllMembers() {
        List<Member> members = memberRepository.findAll();
        
        List<MemberDTO> dtos = new ArrayList<>();
        for (Member member : members) {
            MemberDTO dto = new MemberDTO();
            dto.setId(member.getId());
            dto.setName(member.getName());
            dto.setEmail(member.getEmail());
            
            // N+1 Problem: Query for each member
            Wallet wallet = member.getWallet();
            if (wallet != null) {
                dto.setBalance(wallet.getBalance());
            }
            
            // Another N+1 Problem
            List<Transaction> transactions = member.getTransactions();
            dto.setTransactionCount(transactions.size());
            
            dtos.add(dto);
        }
        return dtos;
    }
}
```

**Câu hỏi:**
1. Vấn đề performance trong đoạn code này là gì?
2. Propose solution để optimize.

<details>
<summary>Đáp án</summary>

**1. Vấn đề: N+1 Query Problem**

**Giải thích:**
- 1 query để lấy danh sách members: `SELECT * FROM member`
- N queries để lấy wallet cho mỗi member: `SELECT * FROM wallet WHERE member_id = ?` (N lần)
- N queries để lấy transactions cho mỗi member: `SELECT * FROM transaction WHERE member_id = ?` (N lần)
- **Tổng cộng: 1 + N + N = 2N + 1 queries**

**Ví dụ:** Nếu có 100 members → 201 queries!

**2. Solutions:**

**Option 1: Eager Fetch với JOIN FETCH**
```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT DISTINCT m FROM Member m " +
           "LEFT JOIN FETCH m.wallet w " +
           "LEFT JOIN FETCH m.transactions t")
    List<Member> findAllWithWalletAndTransactions();
}

@RestController
public class MemberController {
    @Autowired
    private MemberRepository memberRepository;
    
    @GetMapping("/members")
    public List<MemberDTO> getAllMembers() {
        List<Member> members = memberRepository.findAllWithWalletAndTransactions();
        // Now all data is loaded in 1 query
        return members.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
}
```

**Option 2: Entity Graph**
```java
@Entity
@NamedEntityGraph(
    name = "Member.withWalletAndTransactions",
    attributeNodes = {
        @NamedAttributeNode("wallet"),
        @NamedAttributeNode("transactions")
    }
)
public class Member {
    // ...
}

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @EntityGraph("Member.withWalletAndTransactions")
    List<Member> findAll();
}
```

**Option 3: Batch Fetch (Hibernate)**
```java
@Entity
public class Member {
    @OneToOne(fetch = FetchType.LAZY)
    @BatchSize(size = 50)
    private Wallet wallet;
    
    @OneToMany(fetch = FetchType.LAZY)
    @BatchSize(size = 50)
    private List<Transaction> transactions;
}
```

**Option 4: Custom Query với DTO Projection**
```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("SELECT new com.cm.service.dto.MemberDTO(" +
           "m.id, m.name, m.email, w.balance, COUNT(t.id)) " +
           "FROM Member m " +
           "LEFT JOIN m.wallet w " +
           "LEFT JOIN m.transactions t " +
           "GROUP BY m.id, m.name, m.email, w.balance")
    List<MemberDTO> findAllWithWalletAndTransactionCount();
}
```

**Option 5: Two-Step Query (if needed)**
```java
@RestController
public class MemberController {
    @Autowired
    private MemberRepository memberRepository;
    
    @Autowired
    private WalletRepository walletRepository;
    
    @GetMapping("/members")
    public List<MemberDTO> getAllMembers() {
        // Step 1: Get all members (1 query)
        List<Member> members = memberRepository.findAll();
        List<Long> memberIds = members.stream()
            .map(Member::getId)
            .collect(Collectors.toList());
        
        // Step 2: Get all wallets in batch (1 query)
        Map<Long, Wallet> walletMap = walletRepository
            .findByMemberIdIn(memberIds)
            .stream()
            .collect(Collectors.toMap(Wallet::getMemberId, w -> w));
        
        // Step 3: Get transaction counts in batch (1 query)
        Map<Long, Long> transactionCountMap = transactionRepository
            .countByMemberIdIn(memberIds)
            .stream()
            .collect(Collectors.toMap(
                result -> result.getMemberId(),
                result -> result.getCount()
            ));
        
        // Map to DTOs
        return members.stream()
            .map(member -> {
                MemberDTO dto = new MemberDTO();
                dto.setId(member.getId());
                dto.setName(member.getName());
                dto.setEmail(member.getEmail());
                
                Wallet wallet = walletMap.get(member.getId());
                if (wallet != null) {
                    dto.setBalance(wallet.getBalance());
                }
                
                dto.setTransactionCount(transactionCountMap.getOrDefault(member.getId(), 0L));
                return dto;
            })
            .collect(Collectors.toList());
    }
}
```

**Recommendation:**
- Sử dụng **JOIN FETCH** hoặc **Entity Graph** cho simple cases
- Sử dụng **DTO Projection** nếu chỉ cần một số fields
- Sử dụng **Two-Step Query** nếu có complex aggregations
</details>

---

### Câu hỏi 10: Code Review - Deadlock Prevention

**Đoạn code cần review:**
```java
@Service
public class TransferService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Transactional
    public void transfer(Long fromMemberId, Long toMemberId, BigDecimal amount) {
        // Lock from wallet
        Wallet fromWallet = walletRepository.findByMemberIdForUpdate(fromMemberId);
        
        // Check balance
        if (fromWallet.getBalance().compareTo(amount) < 0) {
            throw new InsufficientBalanceException();
        }
        
        // Lock to wallet
        Wallet toWallet = walletRepository.findByMemberIdForUpdate(toMemberId);
        
        // Transfer
        fromWallet.setBalance(fromWallet.getBalance().subtract(amount));
        toWallet.setBalance(toWallet.getBalance().add(amount));
        
        walletRepository.save(fromWallet);
        walletRepository.save(toWallet);
    }
}
```

**Câu hỏi:**
1. Có vấn đề gì với đoạn code này?
2. Propose solution để fix.

<details>
<summary>Đáp án</summary>

**1. Vấn đề: Deadlock Risk**

**Scenario gây deadlock:**
- Thread 1: `transfer(1, 2, 100)` → Lock wallet 1, đợi lock wallet 2
- Thread 2: `transfer(2, 1, 50)` → Lock wallet 2, đợi lock wallet 1
- **Deadlock!** Cả 2 threads đợi nhau vô hạn

**2. Solutions:**

**Option 1: Consistent Lock Ordering (Recommended)**
```java
@Service
public class TransferService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Transactional
    public void transfer(Long fromMemberId, Long toMemberId, BigDecimal amount) {
        // Always lock in consistent order (smaller ID first)
        Long firstId = fromMemberId < toMemberId ? fromMemberId : toMemberId;
        Long secondId = fromMemberId < toMemberId ? toMemberId : fromMemberId;
        
        Wallet firstWallet = walletRepository.findByMemberIdForUpdate(firstId);
        Wallet secondWallet = walletRepository.findByMemberIdForUpdate(secondId);
        
        // Determine which is from and which is to
        Wallet fromWallet = firstId.equals(fromMemberId) ? firstWallet : secondWallet;
        Wallet toWallet = firstId.equals(fromMemberId) ? secondWallet : firstWallet;
        
        // Check balance
        if (fromWallet.getBalance().compareTo(amount) < 0) {
            throw new InsufficientBalanceException();
        }
        
        // Transfer
        fromWallet.setBalance(fromWallet.getBalance().subtract(amount));
        toWallet.setBalance(toWallet.getBalance().add(amount));
        
        walletRepository.save(fromWallet);
        walletRepository.save(toWallet);
    }
}
```

**Option 2: Single Atomic Operation**
```java
@Service
public class TransferService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Transactional
    public void transfer(Long fromMemberId, Long toMemberId, BigDecimal amount) {
        // Check balance first (without lock)
        Wallet fromWallet = walletRepository.findByMemberId(fromMemberId);
        if (fromWallet.getBalance().compareTo(amount) < 0) {
            throw new InsufficientBalanceException();
        }
        
        // Atomic transfer using database-level operations
        walletRepository.decrementBalance(fromMemberId, amount);
        walletRepository.incrementBalance(toMemberId, amount);
    }
}

@Repository
public interface WalletRepository extends JpaRepository<Wallet, Long> {
    @Modifying
    @Query("UPDATE Wallet w SET w.balance = w.balance - :amount WHERE w.memberId = :memberId")
    void decrementBalance(@Param("memberId") Long memberId, @Param("amount") BigDecimal amount);
    
    @Modifying
    @Query("UPDATE Wallet w SET w.balance = w.balance + :amount WHERE w.memberId = :memberId")
    void incrementBalance(@Param("memberId") Long memberId, @Param("amount") BigDecimal amount);
}
```

**Option 3: Ledger-Based Approach (Best for Financial Systems)**
```java
@Service
public class TransferService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Autowired
    private TransactionRepository transactionRepository;
    
    @Transactional
    public void transfer(Long fromMemberId, Long toMemberId, BigDecimal amount) {
        // Create transaction records (atomic)
        Transaction debitTx = new Transaction();
        debitTx.setMemberId(fromMemberId);
        debitTx.setAmount(amount.negate());
        debitTx.setType(TransactionType.TRANSFER_OUT);
        debitTx.setStatus(TransactionStatus.COMPLETED);
        
        Transaction creditTx = new Transaction();
        creditTx.setMemberId(toMemberId);
        creditTx.setAmount(amount);
        creditTx.setType(TransactionType.TRANSFER_IN);
        creditTx.setStatus(TransactionStatus.COMPLETED);
        
        transactionRepository.save(debitTx);
        transactionRepository.save(creditTx);
        
        // Update balances atomically
        walletRepository.decrementBalance(fromMemberId, amount);
        walletRepository.incrementBalance(toMemberId, amount);
    }
}
```

**Option 4: Timeout và Retry**
```java
@Service
public class TransferService {
    @Autowired
    private WalletRepository walletRepository;
    
    @Transactional(timeout = 5) // 5 seconds timeout
    public void transfer(Long fromMemberId, Long toMemberId, BigDecimal amount) {
        // Use consistent lock ordering
        // ...
    }
}
```

**Recommendation:**
- Sử dụng **Consistent Lock Ordering** cho pessimistic locking
- Sử dụng **Atomic Operations** hoặc **Ledger-Based** cho production financial systems
- Luôn có **timeout** để tránh deadlock vô hạn
</details>

---

## Phần 3: Advanced Questions

### Câu hỏi 11: Spring Bean Lifecycle và Circular Dependency

**Đoạn code:**
```java
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
    
    public void methodA() {
        serviceB.methodB();
    }
}

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
    
    public void methodB() {
        serviceA.methodA();
    }
}
```

**Câu hỏi:**
1. Có vấn đề gì với đoạn code này?
2. Spring sẽ xử lý như thế nào? Có bị lỗi không?
3. Nếu có lỗi, làm sao để fix?

<details>
<summary>Đáp án</summary>

**1. Vấn đề: Circular Dependency**

**2. Spring xử lý:**
- Spring **có thể** xử lý circular dependency bằng **constructor injection** (nếu dùng setter/field injection)
- Spring sử dụng **proxy objects** và **lazy initialization** để break cycle
- Tuy nhiên, có thể gây **StackOverflowError** nếu có infinite loop trong logic

**3. Solutions:**

**Option 1: Refactor - Remove Circular Dependency (Best)**
```java
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
    
    public void methodA() {
        // Do something
        serviceB.methodB();
    }
}

@Service
public class ServiceB {
    // Remove dependency on ServiceA
    public void methodB() {
        // Do something
    }
}
```

**Option 2: Use @Lazy**
```java
@Service
public class ServiceA {
    @Autowired
    @Lazy
    private ServiceB serviceB;
    // ...
}

@Service
public class ServiceB {
    @Autowired
    @Lazy
    private ServiceA serviceA;
    // ...
}
```

**Option 3: Use Setter Injection**
```java
@Service
public class ServiceA {
    private ServiceB serviceB;
    
    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

**Option 4: Use ApplicationContext**
```java
@Service
public class ServiceA {
    @Autowired
    private ApplicationContext applicationContext;
    
    public void methodA() {
        ServiceB serviceB = applicationContext.getBean(ServiceB.class);
        serviceB.methodB();
    }
}
```

**Option 5: Extract Common Logic to Third Service**
```java
@Service
public class CommonService {
    public void commonMethod() {
        // Common logic
    }
}

@Service
public class ServiceA {
    @Autowired
    private CommonService commonService;
    
    @Autowired
    private ServiceB serviceB;
}

@Service
public class ServiceB {
    @Autowired
    private CommonService commonService;
}
```
</details>

---

### Câu hỏi 12: Memory Leak trong ThreadLocal

**Đoạn code:**
```java
@Service
public class UserContextService {
    private static final ThreadLocal<User> userContext = new ThreadLocal<>();
    
    public void setUser(User user) {
        userContext.set(user);
    }
    
    public User getUser() {
        return userContext.get();
    }
    
    public void clear() {
        userContext.remove();
    }
}

@RestController
public class UserController {
    @Autowired
    private UserContextService userContextService;
    
    @GetMapping("/user")
    public UserDTO getCurrentUser() {
        User user = userContextService.getUser();
        return toDTO(user);
    }
}
```

**Câu hỏi:**
1. Có vấn đề gì với đoạn code này?
2. Propose solution.

<details>
<summary>Đáp án</summary>

**1. Vấn đề: Memory Leak**

**Giải thích:**
- ThreadLocal lưu data trong thread's local storage
- Nếu thread được **reuse** (như trong thread pool), ThreadLocal data không được clear
- Data cũ vẫn còn trong memory → **memory leak**
- Đặc biệt nguy hiểm với **application server thread pools** (threads sống lâu)

**2. Solutions:**

**Option 1: Always Clear in Finally Block**
```java
@RestController
public class UserController {
    @Autowired
    private UserContextService userContextService;
    
    @GetMapping("/user")
    public UserDTO getCurrentUser() {
        try {
            User user = userContextService.getUser();
            return toDTO(user);
        } finally {
            userContextService.clear(); // Always clear
        }
    }
}
```

**Option 2: Use Interceptor/Filter**
```java
@Component
public class UserContextInterceptor implements HandlerInterceptor {
    @Autowired
    private UserContextService userContextService;
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Set user from request
        User user = extractUserFromRequest(request);
        userContextService.setUser(user);
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // Always clear
        userContextService.clear();
    }
}
```

**Option 3: Use InheritableThreadLocal (if needed)**
```java
@Service
public class UserContextService {
    // InheritableThreadLocal allows child threads to inherit parent's value
    private static final InheritableThreadLocal<User> userContext = new InheritableThreadLocal<>();
    // ...
}
```

**Option 4: Use Request Scope Bean**
```java
@Service
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserContextService {
    private User user;
    
    public void setUser(User user) {
        this.user = user;
    }
    
    public User getUser() {
        return user;
    }
    // No need to clear - Spring handles it
}
```

**Best Practice:**
- Luôn clear ThreadLocal trong **finally block** hoặc **interceptor**
- Sử dụng **Request Scope** nếu có thể (Spring tự động cleanup)
- Monitor memory usage trong production
</details>

---

## Tips cho Technical Interview

### 1. Khi được cho code để review:
- Đọc kỹ code, tìm các vấn đề tiềm ẩn
- Phân loại vấn đề: Performance, Security, Thread Safety, Resource Management, etc.
- Propose solutions với trade-offs
- Đề cập đến best practices

### 2. Khi được cho code để xác định output:
- Hiểu rõ execution flow
- Chú ý đến: concurrency, transaction boundaries, lazy evaluation, etc.
- Trace từng bước execution
- Xem xét edge cases

### 3. Khi propose solution:
- Đưa ra nhiều options với trade-offs
- Giải thích tại sao chọn solution này
- Đề cập đến production considerations (monitoring, error handling, etc.)

### 4. Common Topics cần nắm vững:
- **Concurrency**: Thread safety, deadlock, race conditions
- **Transactions**: Propagation, isolation levels, ACID
- **Spring**: Bean lifecycle, dependency injection, AOP
- **Performance**: N+1 queries, caching, lazy loading
- **Resource Management**: Try-with-resources, connection pooling
- **Design Patterns**: Singleton, Factory, Strategy, etc.

---

## Practice Exercises

### Exercise 1: Implement Thread-Safe Counter
Viết một thread-safe counter service có thể handle concurrent increments.

### Exercise 2: Fix N+1 Query
Tối ưu đoạn code có N+1 query problem trong project của bạn.

### Exercise 3: Implement Retry Mechanism
Implement retry mechanism với exponential backoff cho external API calls.

### Exercise 4: Design Payment Processing
Design một payment processing system với các yêu cầu:
- Thread-safe
- Transactional
- Idempotent
- Async notification

---

**Chúc bạn practice tốt và thành công trong interview! 🚀**
