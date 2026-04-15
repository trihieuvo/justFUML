# KỊCH BẢN BÁO CÁO: DESIGN PATTERNS TRONG AUDIAGEAR E-COMMERCE

## 📋 MỤC LỤC
1. [MỞ ĐẦU - GIỚI THIỆU ĐỀ TÀI](#1)
2. [MODULE THANH TOÁN & ĐẶT HÀNG](#2)
3. [MODULE ĐĂNG KÝ & XÁC THỰC](#3)
4. [TỔNG KẾT LỢI ÍCH](#4)

---

## 1. MỞ ĐẦU - GIỚI THIỆU ĐỀ TÀI <a id="1"></a>

### 🎯 Mục Tiêu Báo Cáo
*Giới thiệu với hội đồng về việc áp dụng Design Patterns vào dự án AudioGear E-Commerce*

**Câu mở đầu:**
"Em xin phép trình bày về việc áp dụng Design Patterns vào module Thanh toán và Đăng ký của hệ thống AudioGear E-Commerce. Đây là một hệ thống bán hàng âm thanh online, được xây dựng bằng Java EE với kiến trúc MVC."

**Show:** README.md (Project Overview)

**Chuyển:** "Trong quá trình phát triển, em nhận thấy có 2 usecase phức tạp nhất cần tối ưu: **Thanh Toán** và **Đăng Ký**. Hôm nay em xin phân tích chi tiết các pattern được sử dụng."

---

## 2. MODULE THANH TOÁN & ĐẶT HÀNG <a id="2"></a>

### 📱 Use Case: Thanh Toán Check

**Bắt đầu bằng diagram tổng quan:**

#### 2.1 FACADE PATTERN - Đơn Giản Hóa Workflow

**Câu dẫn:**
"Đầu tiên, chúng ta cùng nhìn vào tổng thể module Checkout. Đây là một workflow cực kỳ phức tạp, liên quan đến nhiều bước và nhiều service khác nhau."
rder order);
    void notifyCancelled(Order order, String reason);
}

// OrderNotificationServiceImpl.java
public class OrderNotificationServiceImpl implements OrderNotificationService {
    private final EmailService emailService;
    
    @Override
    public void notifyProcessing(Order order) {
        try {
            Context context = new Context();
            context.setVariable("order", order);
            // Tính discount từ voucher
            BigDecimal discount = order.getVoucher() != null ? 
                order.getVoucher().getDiscountValue() : BigDecimal.ZERO;
            context.setVariable("discountAmount", discount);
            
            // Render template HTML
            String html = ThymeleafConfig.getTemplateEngine()
                .process("mail/order-confirmation", context);
            
            // Gửi email
**Show:** `Hieu_Pattern_Puml/facade_checkout.puml`

**Giải thích dễ hiểu:**
*"Theo lý thuyết, Facade Pattern cung cấp một giao diện thống nhất cho một tập hợp các interface trong subsystem, giúp client không phải tự tay điều phối nhiều service."

**Vấn đề AudioGear gặp:**
*"Nếu không dùng Facade, Controller Checkout sẽ phải:*
- Gọi VoucherService để validate voucher
- Gọi Product DAO để check tồn kho
- Gọi Factory để tạo PaymentStrategy
- Gọi Notification Service để gửi mail
- Tự xử lý transaction
- Tự rollback khi có lỗi"

*=> Quá phức tạp, dễ sai sót, khó bảo trì"

**Giải pháp:**
*"Em đóng gói toàn bộ workflow vào một class duy nhất: `CheckoutServiceImpl`*"

**Luồng thực thi chi tiết** (Show code bên cạnh diagram):
```java
// Trong CheckoutServiceImpl.java - Dòng 48
public CheckoutResponse checkout(Long userId, CheckoutRequest request) {
    validateCheckoutRequest(request);  // Bước 1: Validate input
    DatabaseConfig.getInstance().beginTransaction();  // Bước 2: Bắt đầu transaction
    
    // Bước 3: Kiểm tra tồn kho (Product)
    // Bước 4: Validate voucher
    // Bước 5: Xử lý thanh toán qua Strategy
    PaymentStrategy strategy = PaymentStrategyFactory.fromCode(request.getPaymentMethod());
    PaymentResult paymentResult = strategy.pay(finalTotal);
    
    // Bước 6: Tạo Order và trừ kho
    // Bước 7: Commit/Rollback transaction
    // Bước 8: Gửi mail thông báo
}
```

**Highlight:**
- **Show** dòng 32: `PaymentStrategyFactory.fromCode()`
- **Show** dòng 33: `strategy.pay(finalTotal)`
- **Giải thích:** "Client chỉ cần biết là gọi `checkout()`, không cần biết bên trong có bao nhiêu service"

**Lợi ích:**
✅ Giảm coupling giữa Controller và các Service
✅ Tập trung business logic workflow ở một chỗ
✅ Controller gọn gàng, chỉ cần ~10 dòng code
✅ Dễ unit test (có thể mock CheckoutService)

---

#### 2.2 STRATEGY PATTERN - Đa dạng phương thức thanh toán

**Câu dẫn:**
"Tiếp theo, chúng ta sẽ đi sâu vào một subsystem quan trọng: Xử lý thanh toán. AudioGear cần hỗ trợ nhiều phương thức thanh toán khác nhau."

**Show:** `Hieu_Pattern_Puml/strategy_payment.puml`

**Giải thích dễ hiểu:**
*"Strategy Pattern định nghĩa một tập hợp các thuật toán, đóng gói chúng lại và chúng có thể hoán đổi lẫn nhau."

**Vấn đề AudioGear gặp:**
*"Nếu dùng if-else truyền thống:*

```java
if (paymentMethod.equals("COD")) {
    // process COD
} else if (paymentMethod.equals("MOMO")) {
    // process Momo
} else if (paymentMethod.equals("BANK")) {
    // process Bank
}
// => Khi thêm method mới (VNPay) phải sửa code cũ
```

*=> Vi phạm Open/Closed Principle, dễ bug, khó mở rộng"

**Giải pháp:**
*"Em tạo một interface `PaymentStrategy` và các implementation riêng biệt:*"

**Show interface code** (bên cạnh diagram):
```java
// PaymentStrategy.java
public interface PaymentStrategy {
    PaymentResult pay(BigDecimal amount);
    String getStrategyCode();
}
```

**Show 2-3 implementations:**
```java
// CODStrategy.java
public class CODStrategy implements PaymentStrategy {
    @Override
    public PaymentResult pay(BigDecimal amount) {
        return PaymentResult.success("Thanh toán khi nhận hàng");
    }
}

// MomoStrategy.java
public class MomoStrategy implements PaymentStrategy {
    @Override
    public PaymentResult pay(BigDecimal amount) {
        String transactionId = "MOMO-" + UUID.randomUUID();
        return PaymentResult.success("Thanh toán qua Momo", transactionId);
    }
}

// SePayStrategy.java
public class SePayStrategy implements PaymentStrategy {
    @Override
    public PaymentResult pay(BigDecimal amount) {
        return new PaymentResult(true, "Vui lòng quét mã QR", null);
    }
}
```

**Highlight:**
- **Giải thích:** "Mỗi strategy có logic riêng: COD luôn success, Momo sinh transactionId, SePay yêu cầu QR"
- **Nói rõ:** "Khi cần thêm VNPay, chỉ cần tạo thêm một class mới implements PaymentStrategy, không sửa code cũ"

**Lợi ích:**
✅ Đóng gói các thuật toán thanh toán thành các class độc lập
✅ Có thể thay đổi algorithm tại runtime (dựa trên lựa chọn của user)
✅ Tuân thủ Open/Closed Principle
✅ Dễ unit test từng strategy một cách tách biệt

---

#### 2.3 FACTORY PATTERN - Tạo PaymentStrategy object

**Câu dẫn:**
"Vậy Strategy Pattern giải quyết việc lưu trữ thuật toán, nhưng còn việc tạo đối tượng Strategy? Đây là công việc của Factory Pattern"

**Show:** `Hieu_Pattern_Puml/factory_payment.puml`

**Giải thích dễ hiểu:**
*"Factory Pattern cung cấp giao diện tạo object mà không cần chỉ rõ class cụ thể sẽ được tạo."

**Vấn đề:**
*"Nếu CheckoutServiceImpl tự tạo strategy bằng `new CODStrategy()`, nó sẽ bị phụ thuộc vào concrete class. Khi thay đổi cách tạo (VD: thêm caching), phải sửa code nhiều nơi."

**Giải pháp:**
*"Em tạo một `PaymentStrategyFactory` tập trung logic tạo:*"

**Show code Factory** (bên cạnh diagram):
```java
// PaymentStrategyFactory.java
public final class PaymentStrategyFactory {
    private static final Map<String, Supplier<PaymentStrategy>> STRATEGIES = Map.of(
        "COD", CODStrategy::new,
        "MOMO", MomoStrategy::new,
        "BANK", BankTransferStrategy::new,
        "BANK_TRANSFER", BankTransferStrategy::new, // Alias
        "STORE_PICKUP", StorePickupStrategy::new,
        "SEPAY_QR", SePayStrategy::new
    );
    
    public static PaymentStrategy fromCode(String code) {
        return Optional.ofNullable(code)
            .map(String::toUpperCase)
            .map(STRATEGIES::get)
            .map(Supplier::get)
            .orElseThrow(() -> new RuntimeException("Phương thức thanh toán không hợp lệ: " + code));
    }
}
```

**Highlight:**
- **Giải thích:** "Dùng `Map<String, Supplier<PaymentStrategy>>` để ánh xạ mã thanh toán thành supplier"
- **Nhấn mạnh:** "Constructor private → Utility class, không cho phép tạo instance"
- **Note:** "BANK và BANK_TRANSFER đều trỏ tới BankTransferStrategy (alias)"

**Cách sử dụng trong CheckoutServiceImpl:**
```java
// Dòng 132-133
PaymentStrategy strategy = PaymentStrategyFactory.fromCode(request.getPaymentMethod());
PaymentResult result = strategy.pay(finalTotal);
```

**Lợi ích:**
✅ Ẩn logic tạo object, client chỉ cần gọi static method
✅ Không bị phụ thuộc vào concrete class
✅ Dễ maintain logic tạo object tại một chỗ
✅ Có thể lazy load, cache, hoặc thêm factory methods mà không ảnh hưởng client

---

#### 2.4 OBSERVER PATTERN - Thông báo trạng thái đơn hàng

**Câu dẫn:**
"Sau khi checkout thành công và đã commit DB, cần gửi email xác nhận cho khách hàng. Nhưng không muốn làm chậm response. Đây là công việc của Observer Pattern"

**Show:** `Hieu_Pattern_Puml/observer_order.puml`

**Giải thích dễ hiểu:**
*"Observer Pattern định nghĩa relationship 1-nhiều. Khi Subject (đơn hàng) thay đổi trạng thái, tất cả Observer (các service thông báo) sẽ được thông báo"

**Vấn đề:**
*"Nếu gọi `sendEmail()` ngay trong CheckoutServiceImpl:*
```java
// Bad practice
if (orderCreated) {
    emailService.send(...); // Có thể mất 2-3 giây
    return response; // User phải chờ lâu
}
```
*=> Tốc độ response bị ảnh hưởng"

**Giải pháp:**
*"Tách riêng phần gửi email thành một service riêng, chỉ được gọi sau khi thanh toán thành công:*"

**Show interface và implementation:**
```java
// OrderNotificationService.java (Interface)
public interface OrderNotificationService {
    void notifyProcessing(Order order);
    void notifyShipping(Order order);
    void notifyDelivered(Order order);
    void notifyCancelled(Order order, String reason);
}

// OrderNotificationServiceImpl.java
public class OrderNotificationServiceImpl implements OrderNotificationService {
    private final EmailService emailService;
    
    @Override
    public void notifyProcessing(Order order) {
        try {
            Context context = new Context();
            context.setVariable("order", order);
            // Tính discount từ voucher
            BigDecimal discount = order.getVoucher() != null ? 
                order.getVoucher().getDiscountValue() : BigDecimal.ZERO;
            context.setVariable("discountAmount", discount);
            
            // Render template HTML
            String html = ThymeleafConfig.getTemplateEngine()
                .process("mail/order-confirmation", context);
            
            // Gửi email
            emailService.sendHtmlEmail(order.getEmail(), 
                "Xác nhận đơn hàng #" + order.getOrderCode(), html);
        } catch (Exception e) {
            e.printStackTrace(); // Try-catch để không ảnh hưởng flow chính
        }
    }
}
```

**Highlight:**
- **Giải thích:** "Service này giống như một observer đăng ký để lắng nghe sự kiện 'đơn hàng được tạo'"
- **Nhấn mạnh:** "Dùng `try-catch` để nếu gửi mail lỗi, vẫn không rollback order đã tạo"
- **Ghi chú:** "Chỉ gửi mail ngay nếu không phải SEPAY_QR (trừ phương thức QR sẽ gửi khi nhận callback thanh toán)"

**Cách sử dụng trong CheckoutServiceImpl:**
```java
// Dòng 188 (sau khi commit transaction thành công)
if (!"SEPAY_QR".equalsIgnoreCase(request.getPaymentMethod())) {
    orderNotificationService.notifyProcessing(order);
}
```

**Lợi ích:**
✅ Decouple: Checkout service không cần biết cách gửi mail
✅ Dễ mở rộng: Có thể thêm SMS notification, Slack notification không sửa checkout code
✅ Event-driven: Có thể chuyển sang message queue sau này
✅ Không ảnh hưởng flow chính nếu thông báo lỗi

**⚠️ QUAN TRỌNG:**
*"Chú ý rằng OrderNotificationService thuộc Observer Pattern, không liên quan đến OrderContext (thuộc State Pattern)"

---


## 3. MODULE ĐĂNG KÝ & XÁC THỰC <a id="3"></a>

### 📱 Use Case: Đăng ký tài khoản

**Câu dẫn:**
"Bây giờ chúng ta chuyển sang module thứ 2: Đăng ký và xác thực. Đây cũng là một workflow phức tạp với nhiều bước."

#### 3.1 FACADE PATTERN - AuthFacade

**Show:** `Hieu_Pattern_Puml/facade_auth.puml`

**Giải thích dễ hiểu:**
*"Tương tự như Checkout, module Đăng ký cũng có nhiều bước: kiểm tra spam, tạo user pending, sinh OTP, lưu cache, gửi email."

**Vấn đề:**
*"Nếu Controller tự xử lý, nó sẽ phải:*
- Gọi Redis check spam
- Gọi UserService tạo user
- Gọi EmailService sinh OTP
- Gọi Redis lưu OTP
- Gọi EmailService gửi mail
- Xử lý lỗi từng bước

=> Controller sẽ rất lớn và khó maintain"

**Giải pháp:**
*"Tạo AuthFacade gói gọn toàn bộ:*"

**Show AuthFacade code:**
```java
// AuthFacade.java
public class AuthFacade {
    private final UserService userService;
    private final RedisService redisService;
    private final EmailService emailService;
    
    public void requestRegister(RegisterRequest request) {
        // 1. Anti-spam check
        if (!redisService.canSendOtp(request.getEmail())) {
            throw new RuntimeException("Vui lòng đợi 1 phút");
        }
        
        // 2. Tạo user PENDING
        userService.registerPendingUser(request);
        
        // 3. Sinh OTP
        String otp = emailService.generateOTP();
        
        // 4. Lưu Redis
        redisService.saveRegisterOtp(request.getEmail(), otp);
        
        // 5. Gửi email
        emailService.sendOtp(request.getEmail(), otp);
    }
}
```

**Luồng thực thi:**
- **Giải thích từng dòng từ 37-60 trong AuthFacade.java**
- **Show kết nối với diagram:** "Như diagram đã thể hiện, AuthFacade điều phối 3 subsystem"

**Lợi ích:**
✅ Controller chỉ cần gọi 1 method: `authFacade.requestRegister(body)`
✅ Tách rời logic phức tạp khỏi UI layer
✅ Dễ unit test, có thể mock cả facade
✅ Consistent error handling

---

#### 3.2 DAO PATTERN - Truy cập Database

**Câu dẫn:**
"Để AuthFacade gọi UserService, UserService cần truy cập database. Đây là công việc của DAO Pattern"

**Show:** `Hieu_Pattern_Puml/dao_user.puml`

**Giải thích dễ hiểu:**
*"DAO Pattern tách biệt logic truy cập database khỏi business logic. Nghĩa là Service không cần biết dùng JPA, JDBC hay gọi API."

**Vấn đề:**
*"Nếu viết JPQL trực tiếp trong Service:*
```java
// Bad practice
public User findByEmail(String email) {
    EntityManager em = ...;
    return em.createQuery("SELECT u FROM User u WHERE u.email = :email").getSingleResult();
}
// => Service bị phụ thuộc vào JPA, khó test unit, khó đổi database"
```

**Giải pháp:**
*"Tạo interface UserDAO và implementation với JPA:*"

**Show UserDAO interface + UserDAOImpl:**
```java
// UserDAO.java
public interface UserDAO {
    Optional<User> findByEmail(String email);
    Optional<User> findByUsername(String username);
    Optional<User> findByPhoneNumber(String phone);
    User save(User user);
    List<User> search(String keyword, UserRole role, UserStatus status, int offset, int limit);
    long countSearch(...);
}

// UserDAOImpl.java - Dòng 22
public static synchronized UserDAOImpl getInstance() {
    if (instance == null) {
        instance = new UserDAOImpl();
    }
    return instance;
}

// UserDAOImpl.java - Dòng 28-40
public Optional<User> findByEmail(String email) {
    EntityManager em = DatabaseConfig.getInstance().getEntityManager();
    try {
        User user = em.createQuery("SELECT u FROM User u WHERE u.email = :email", User.class)
                     .setParameter("email", email)
                     .getSingleResult();
        return Optional.of(user);
    } catch (NoResultException e) {
        return Optional.empty();
    }
}
```

**Highlight:**
- **Giải thích:** "Singleton Pattern đảm bảo chỉ có 1 instance duy nhất, tiết kiệm resource"
- **Nhấn mạnh:** "Interface UserDAO giúp service không phụ thuộc vào implementation"
- **Ví dụ:** "Khi muốn chuyển sang MongoDB, chỉ cần tạo UserDAOMongoImpl, không sửa Service"

**Lợi ích:**
✅ Separation of Concerns: Service chỉ lo business logic, DAO lo database
✅ Dễ mock trong unit test (có thể tạo UserDAOMock)
✅ Dễ thay đổi công nghệ persistence
✅ Quản lý transaction tập trung

---



**Nhấn mạnh:** "Có thể thêm Facebook, GitHub login một cách dễ dàng"

---

## 4. TỔNG KẾT LỢI ÍCH <a id="4"></a>

### 📊 Bảng So Sánh: Before vs After

| Vấn Đề | Before (Không dùng Pattern) | After (Có Pattern) |
|--------|------------------------------|-------------------|
| **Thanh toán** | if-else lồng nhau, khó thêm method mới | Strategy + Factory: Dễ mở rộng |
| **Checkout flow** | Controller phải gọi 7-8 service, lớn ~200 dòng | Facade: Controller chỉ 10 dòng |
| **Thông báo** | Gọi trực tiếp trong checkout, làm chậm response | Observer: Gửi async, decoupled |
| **Database access** | JPQL trong service, khó test | DAO: Dễ mock, dễ đổi DB |
| **Auth flow** | Nhiều bước trong Servlet, khó maintain | AuthFacade: Workflow gọn gàng |

### ✅ Lợi Ích Tổng Quát

1. **Maintainability**: Mỗi class có trách nhiệm đơn nhất (SRP), dễ sửa, dỽ debug
2. **Extensibility**: Thêm tính năng mới không sửa code cũ (OCP)
3. **Testability**: Có thể unit test từng component riêng biệt với mocks
4. **Reusability**: Các strategy, DAO có thể tái sử dụng ở nhiều nơi
5. **Loose Coupling**: Các module ít phụ thuộc vào nhau

### 🎯 Kết Luận

**Câu kết:**
"Như vậy, em đã phân tích chi tiết các Design Patterns được sử dụng trong 2 usecase phức tạp nhất của AudioGear. Các pattern không chỉ là lý thuyết, mà đã giải quyết các vấn đề thực tế, giúp codebase dễ bảo trì, mở rộng và test hơn rất nhiều."

**Mời câu hỏi:**
"Em xin phép kết thúc phần trình bày, và sẵn sàng trả lời các câu hỏi từ hội đồng."

---

## 🎯 GỢI Ý KHI BÁO CÁO

### 💡 Tips Để Báo Cáo Mượt Mà

1. **Luôn show diagram trước, rồi mới show code**: Để người nghe có cái nhìn tổng quan trước khi đi sâu

2. **Highlight dòng code quan trọng**: Dùng con trỏ chuột hoặc tô màu dòng code đang giải thích

3. **Dùng ví dụ trực quan**: 
   - Khi nói Strategy: Dùng ví dụ tính tiền ở siêu thị (tiền mặt, credit, QR)
   - Khi nói Facade: Dùng ví dụ gọi đồ ăn (khách chỉ gọi "giao đồ ăn", không cần biết bếp, shipper làm gì)

4. **Nhấn mạnh vào "Tại sao”**: Đừng chỉ nói “Có pattern”, mà phải giải thích “Vì sao phải dùng pattern này thay vì giải pháp khác”

5. **Chuẩn bị cho câu hỏi phản biện**:
   - *"Tại sao không dùng dependency injection thay vì new() trong Facade?"* → "Đây là decision đơn giản cho project, DI đòi hỏi framework nặng hơn"
   - *"Factory có cần thiết không? Tại sao không dùng Enum?"* → "Enum không tạo new instance mỗi lần, Supplier mới đảm bảo state riêng"
   - *"SePayStrategy return success nhưng không có transaction?"* → "Đúng, vì thanh toán QR là pending, sẽ có webhook callback sau"

6. **Chuẩn bị demo (nếu có thời gian)**: Có thể chạy ứng dụng live, show quy trình checkout hoặc đăng ký

---

## 📁 THAM KHẢO TÀI LIỆU

- **PlantUML diagrams**: `src_3/justFUML/src/Hieu_Pattern_Puml/`
- **Source code**: `src/main/java/vn/edu/ute/`
- **Key classes**:
  - `order/payment/strategy/`
  - `service/impl/CheckoutServiceImpl.java`
  - `auth/facade/AuthFacade.java`
  - `dao/impl/UserDAOImpl.java`

---

**Chúc bạn báo cáo thành công! 🎉**