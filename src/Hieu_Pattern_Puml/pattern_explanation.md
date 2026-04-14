# Tài liệu Phân tích & Luồng Thực thi Design Patterns - AudioGear E-Commerce

Tài liệu này cung cấp đầy đủ thông tin về lý thuyết, cách triển khai và luồng thực thi chi tiết của các Design Pattern được sử dụng trong hệ thống.

---

## I. Module Thanh Toán & Đặt Hàng (Checkout & Ordering)

### 1. Strategy Pattern (Chiến lược thanh toán)
*   **Lý thuyết & Lý do chọn:** Cho phép định nghĩa một tập hợp các thuật toán thanh toán khác nhau (COD, Momo, QR) và hoán đổi chúng linh hoạt. Chọn Strategy thay vì if-else để mã nguồn dễ mở rộng khi thêm phương thức thanh toán mới mà không làm hỏng logic cũ.
*   **Triển khai thực tế:** Interface `PaymentStrategy` định nghĩa khung; các lớp `CODStrategy`, `MomoStrategy`, `SePayStrategy`... thực hiện logic cụ thể.
*   **Luồng thực thi chi tiết (Dò trong `CheckoutServiceImpl.java`):**
    1.  **Dòng 132:** Gọi `PaymentStrategy strategy = PaymentStrategyFactory.fromCode(...)` để lấy đối tượng cụ thể.
    2.  **Dòng 133:** Gọi `strategy.pay(finalTotal)`.
    3.  **Tại Strategy (VD: `MomoStrategy.java`):** Thực thi hàm `pay()`, sinh mã giao dịch (Line 15), trả về `PaymentResult`.
    4.  **Quay lại Service (Line 135):** Kiểm tra `paymentResult.success()`. Nếu `false`, ném Exception để rollback transaction.

### 2. Factory Pattern (Khởi tạo Payment Strategy)
*   **Lý thuyết & Lý do chọn:** Cung cấp giải pháp tạo đối tượng mà không cần chỉ định chính xác lớp cụ thể sẽ được tạo. Giúp giấu logic khởi tạo phức tạp (như dùng Map hay Supplier) khỏi tầng nghiệp vụ Service.
*   **Triển khai thực tế:** Lớp `PaymentStrategyFactory` đóng vai trò là Simple Factory.
*   **Luồng thực thi chi tiết (Dò trong `PaymentStrategyFactory.java`):**
    1.  **Dòng 15:** Khởi tạo `Map<String, Supplier<PaymentStrategy>>` chứa danh sách các "công thức" tạo Strategy.
    2.  **Dòng 32 (Hàm `fromCode`):** Nhận tham số `code` (VD: "COD").
    3.  **Dòng 36-37:** Truy xuất Map lấy `Supplier` tương ứng -> Gọi `.get()` để tạo mới instance (VD: `new CODStrategy()`).
    4.  **Dòng 38:** Nếu không tìm thấy, ném `RuntimeException` báo lỗi phương thức thanh toán.

### 3. Facade Pattern (Đơn giản hóa Checkout)
*   **Lý thuyết & Lý do chọn:** Cung cấp một giao diện hợp nhất cho một tập hợp các giao diện trong một subsystem. Chọn Facade để Controller chỉ cần gọi một hàm `checkout()` thay vì phải tự tay điều phối hàng loạt Service (Voucher, Kho, Thanh toán...).
*   **Triển khai thực tế:** `CheckoutServiceImpl` là lớp Facade kết nối nhiều subsystem.
*   **Luồng thực thi chi tiết (Dò trong `CheckoutServiceImpl.java`):**
    1.  **Dòng 49:** Gọi `validateCheckoutRequest()` check dữ liệu thô.
    2.  **Dòng 73-108:** Duyệt danh sách SP, gọi `em.find(Product.class)` check tồn kho.
    3.  **Dòng 115:** Gọi `voucherService.validateVoucher()` để tính toán giảm giá.
    4.  **Dòng 132-133:** Gọi cụm `PaymentFactory` & `Strategy` để xử lý tiền.
    5.  **Dòng 140-168:** Tạo thực thể `Order`, `OrderItem`, lưu vào Database qua JPA.
    6.  **Dòng 188:** Gọi `OrderNotificationService` gửi mail thông báo.

### 4. Observer Pattern (Thông báo Thay đổi Đơn hàng)
*   **Lý thuyết & Lý do chọn:** Định nghĩa sự phụ thuộc 1-nhiều. Khi Subject (Order) thay đổi, các Observer (MailService) sẽ nhận được thông báo. Giúp tách rời logic nghiệp vụ chính khỏi logic thông báo phụ trợ.
*   **Triển khai thực tế:** `OrderNotificationService` là Observer thực hiện việc gửi email.
*   **Luồng thực thi chi tiết (Dò trong `OrderNotificationServiceImpl.java`):**
    1.  **Dòng 22 (Hàm `notifyProcessing`):** Nhận vào đối tượng `Order` vừa lưu.
    2.  **Dòng 24-33:** Chuẩn bị `Thymeleaf Context`, nạp dữ liệu đơn hàng.
    3.  **Dòng 34:** Gọi `ThymeleafConfig` render file template `.html` thành chuỗi String nội dung Email.
    4.  **Dòng 37:** Gọi `emailService.sendHtmlEmail()` để gửi đi thực tế.

### 5. State Pattern (Vòng đời đơn hàng)
*   **Lý thuyết & Lý do chọn:** Cho phép đối tượng thay đổi hành vi khi trạng thái nội bộ thay đổi. Chọn State thay vì if-else status để kiểm soát các bước chuyển trạng thái (StateMachine) một cách an toàn và tường minh.
*   **Triển khai thực tế:** `OrderContext` chứa `OrderState`. Các lớp `PendingState`, `ProcessingState`, `ShippedState`... thực hiện logic.
*   **Luồng thực thi chi tiết (Dò trong `OrderContext.java` & `PendingState.java`):**
    1.  **Tại `OrderContext.java` (Line 46-64):** Constructor tự tạo State tương ứng với status hiện tại.
    2.  **Dòng 94:** Admin gọi `context.processOrder()`. Hàm này ủy quyền: `state.processOrder(this)`.
    3.  **Tại `PendingState.java`:**
        *   Cập nhật `order.setStatus(OrderStatus.PROCESSING)`.
        *   Gọi `context.setState(new ProcessingState())` để đổi "lớp" xử lý.
        *   Gửi email báo khách thông qua `context.getNotificationService()`.

---

## II. Module Đăng Ký & Xác Thực (Registration & Auth)

### 1. Facade Pattern (AuthFacade)
*   **Lý thuyết & Lý do chọn:** Tương tự module Checkout, đóng gói các bước phức tạp của quy trình Auth (Spam check, OTP, DB) để đơn giản hóa interface cho Controller.
*   **Triển khai thực tế:** Lớp `AuthFacade` điều phối `UserService`, `RedisService`, `EmailService`.
*   **Luồng thực thi chi tiết (Dò trong `AuthFacade.java`):**
    1.  **Dòng 41:** Gọi `redisService.canSendOtp()` chặn Spam.
    2.  **Dòng 46:** Gọi `userService.registerPendingUser()` tạo tài khoản PENDING.
    3.  **Dòng 49-52:** Sinh OTP và lưu vào Redis.
    4.  **Dòng 57:** Gọi `emailService.sendOtp()` gửi mã qua Email.

### 2. Strategy Pattern (Đa phương thức Login)
*   **Lý thuyết & Lý do chọn:** Cho phép thay đổi linh hoạt giữa đăng nhập truyền thống và OAuth2 (Google). Giúp hệ thống dễ dàng tích hợp thêm Facebook/Github login sau này.
*   **Triển khai thực tế:** Interface `LoginStrategy` với 2 bản mẫu: `LocalLoginStrategy` và `GoogleLoginStrategy`.
*   **Luồng thực thi chi tiết:**
    1.  User chọn phương thức Login -> `AuthServiceImpl` lấy đúng Strategy từ Map.
    2.  `LocalLoginStrategy`: Query SQL tìm user -> `BCrypt.checkpw()` so khớp.
    3.  `GoogleLoginStrategy`: Xác thực Token từ Google API -> Trả về User.

### 3. Adapter Pattern (Google OAuth)
*   **Lý thuyết & Lý do chọn:** Chuyển đổi giao diện của một lớp thành giao diện khác mà client mong muốn. Chọn Adapter để biến dữ liệu thô từ Google API thành đối tượng `User` mà hệ thống có thể hiểu được.
*   **Triển khai thực tế:** `GoogleToUserAdapter` bọc `GoogleProfile`.
*   **Luồng thực thi chi tiết (Dò trong `GoogleToUserAdapter.java`):**
    1.  **Dòng 24:** Nhận đối tượng `GoogleProfile` (Adaptee).
    2.  **Dòng 27 (Hàm `adaptToUser`):** Trích xuất `email`, `name` từ Profile gán vào thực thể `User` mới.
    3.  Trả về `User` chuẩn cho tầng Service xử lý.

### 4. Singleton Pattern (Quản lý tài nguyên)
*   **Lý thuyết & Lý do chọn:** Đảm bảo chỉ có một instance duy nhất. Chọn Singleton cho các kết nối Database/Redis để quản lý Connection Pool hiệu quả, tránh lãng phí tài nguyên.
*   **Triển khai thực tế:** `DatabaseConfig` (Eager), `RedisConfig` (Double-Checked Locking).
*   **Luồng thực thi chi tiết (Dò trong `UserDAOImpl.java`):**
    1.  **Dòng 20:** Gọi `UserDAOImpl.getInstance()`.
    2.  Kiểm tra nếu `instance` null thì mới khởi tạo trong khối `synchronized`.
    3.  Trả về instance duy nhất cho mọi yêu cầu truy xuất dữ liệu.

### 5. DAO Pattern (Data Access Object)
*   **Lý thuyết & Lý do chọn:** Tách biệt logic persistence khỏi business logic. Giúp lớp Service không bị phụ thuộc vào câu lệnh SQL/JPQL cụ thể, dễ dàng thay đổi công nghệ Database khi cần.
*   **Triển khai thực tế:** Interface `UserDAO` và lớp `UserDAOImpl` dùng JPA.
*   **Luồng thực thi chi tiết (Dò trong `UserDAOImpl.java`):**
    1.  **Dòng 59 (Hàm `findByUsernameOrEmail`):** Mở `EntityManager`.
    2.  **Dòng 61:** Thực thi JPQL tìm kiếm User theo định danh.
    3.  **Dòng 65:** Đóng gói kết quả vào `Optional` và trả về.
