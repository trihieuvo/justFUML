# Tài liệu Giải thích Design Patterns - AudioGear E-Commerce

Tài liệu này trình bày chi tiết về các Design Pattern được áp dụng trong hai module quan trọng nhất của hệ thống: **Thanh Toán (Checkout)** và **Đăng Ký & Xác Thực (Auth)**.

---

## I. Các Pattern trong module Thanh Toán (Checkout)

### 1. Strategy Pattern
*   **Lý thuyết:** Cho phép định nghĩa một tập hợp các thuật toán, đóng gói từng thuật toán lại và làm cho chúng có thể thay thế lẫn nhau. Strategy cho phép thuật toán biến đổi độc lập với các client sử dụng nó.
*   **Triển khai thực tế:**
    *   **Interface:** `PaymentStrategy` định nghĩa phương thức `pay(BigDecimal amount)`.
    *   **Concrete Strategies:** `CODStrategy`, `MomoStrategy`, `BankTransferStrategy`, `SePayStrategy`, `StorePickupStrategy`. Mỗi class xử lý logic thanh toán đặc thù cho phương thức đó.
*   **Luồng thực thi:**
    1.  `CheckoutServiceImpl` nhận yêu cầu đặt hàng với mã phương thức thanh toán (VD: "MOMO").
    2.  Hệ thống lấy đúng đối tượng `MomoStrategy` từ Factory.
    3.  Gọi `strategy.pay(amount)`. Nếu là Momo, nó sẽ sinh mã giao dịch giả lập; nếu là SePay, nó trả về trạng thái yêu cầu quét mã QR.

### 2. Factory Pattern (Simple Factory)
*   **Lý thuyết:** Cung cấp một giao diện chung để tạo ra các đối tượng trong một họ, nhưng cho phép các lớp con quyết định lớp nào sẽ được khởi tạo.
*   **Triển khai thực tế:**
    *   **Class:** `PaymentStrategyFactory`.
    *   Sử dụng một `Map<String, Supplier<PaymentStrategy>>` để ánh xạ các mã chuỗi (String) sang các hàm khởi tạo đối tượng tương ứng.
*   **Luồng thực thi:**
    1.  `CheckoutServiceImpl` gọi `PaymentStrategyFactory.fromCode("BANK")`.
    2.  Factory kiểm tra trong Map, tìm thấy `BankTransferStrategy::new`.
    3.  Khởi tạo và trả về một instance mới của `BankTransferStrategy`.

### 3. Facade Pattern
*   **Lý thuyết:** Cung cấp một giao diện hợp nhất (unified interface) cho một tập hợp các giao diện trong một hệ thống con (subsystem). Facade định nghĩa một giao diện cao cấp hơn giúp hệ thống con dễ sử dụng hơn.
*   **Triển khai thực tế:**
    *   **Class:** `CheckoutServiceImpl`.
    *   Nó giấu đi sự phức tạp của việc phối hợp giữa: `VoucherService` (giảm giá), `ProductDAO` (kho hàng), `PaymentStrategy` (thanh toán), và `OrderNotificationService` (thông báo).
*   **Luồng thực thi:**
    1.  `CheckoutController` chỉ gọi duy nhất hàm `checkout()`.
    2.  Bên trong `checkout()`, Facade thực hiện: Validate dữ liệu -> Kiểm tra tồn kho -> Tính giảm giá -> Gọi thanh toán -> Lưu DB -> Gửi mail.
    3.  `Controller` không cần biết các bước nhỏ nhặt bên trong.

### 4. Observer Pattern
*   **Lý thuyết:** Định nghĩa một sự phụ thuộc một-nhiều giữa các đối tượng sao cho khi một đối tượng thay đổi trạng thái, tất cả các đối tượng phụ thuộc của nó đều được thông báo và cập nhật tự động.
*   **Triển khai thực tế:**
    *   **Subject:** `CheckoutServiceImpl` hoặc `OrderContext`.
    *   **Observer:** `OrderNotificationService`.
*   **Luồng thực thi:**
    1.  Sau khi đơn hàng được lưu thành công vào Database.
    2.  `CheckoutServiceImpl` phát đi một "thông báo" bằng cách gọi `orderNotificationService.notifyProcessing(order)`.
    3.  `OrderNotificationServiceImpl` (người quan sát) nhận lệnh và thực hiện việc render email HTML và gửi tới khách hàng mà không làm ảnh hưởng đến luồng lưu DB chính.

### 5. State Pattern
*   **Lý thuyết:** Cho phép một đối tượng thay đổi hành vi của nó khi trạng thái nội bộ của nó thay đổi. Đối tượng sẽ có vẻ như thay đổi lớp của nó.
*   **Triển khai thực tế:**
    *   **Context:** `OrderContext` chứa đối tượng `Order` và một thực thể `OrderState`.
    *   **States:** `PendingState`, `ProcessingState`, `ShippingState`, `DeliveredState`, `CancelledState`.
*   **Luồng thực thi:**
    1.  Admin thực hiện hành động "Duyệt đơn hàng" trên một Order đang ở trạng thái `PENDING`.
    2.  `OrderContext` sẽ ủy quyền cho `PendingState.processOrder()`.
    3.  `PendingState` thực hiện: Cập nhật status đơn hàng -> Chuyển trạng thái của Context sang `ProcessingState` -> Gửi mail.
    4.  Nếu đơn hàng đang ở `DELIVERED` mà Admin bấm "Hủy", `DeliveredState` sẽ ném Exception vì đây là bước chuyển trạng thái không hợp lệ.

---

## II. Các Pattern trong module Đăng Ký & Xác Thực (Auth)

### 1. Facade Pattern
*   **Lý thuyết:** (Tương tự module Checkout) Đóng gói các logic phức tạp đằng sau một lớp trung gian đơn giản.
*   **Triển khai thực tế:** `AuthFacade`.
*   **Luồng thực thi:**
    1.  Khi người dùng đăng ký, `AuthController` gọi `authFacade.requestRegister()`.
    2.  `AuthFacade` điều phối: Kiểm tra spam qua `RedisService` -> Lưu user tạm qua `UserService` -> Sinh OTP qua `EmailService` -> Gửi mail.

### 2. Strategy Pattern
*   **Lý thuyết:** (Tương tự module Checkout) Thay đổi thuật toán đăng nhập linh hoạt.
*   **Triển khai thực tế:** `LoginStrategy` với 2 loại: `LocalLoginStrategy` (dùng DB + BCrypt) và `GoogleLoginStrategy` (dùng OAuth2).
*   **Luồng thực thi:** Tùy vào việc User bấm "Login thường" hay "Login Google", hệ thống sẽ chọn Strategy tương ứng để xác thực danh tính.

### 3. Adapter Pattern
*   **Lý thuyết:** Cho phép các lớp có giao diện không tương thích có thể làm việc cùng nhau bằng cách quấn (wrap) giao diện của đối tượng hiện có vào một giao diện mà client mong muốn.
*   **Triển khai thực tế:**
    *   **Target:** `OAuthUserAdapter`.
    *   **Adaptee:** `GoogleProfile` (dữ liệu trả về từ Google API).
    *   **Adapter:** `GoogleToUserAdapter`.
*   **Luồng thực thi:**
    1.  Hệ thống nhận thông tin từ Google (email, name, picture).
    2.  Dữ liệu này không khớp với cấu trúc class `User` của hệ thống.
    3.  `GoogleToUserAdapter` nhận `GoogleProfile`, trích xuất dữ liệu và "chuyển đổi" nó thành một đối tượng `User` chuẩn để lưu vào Postgres.

### 4. Singleton Pattern
*   **Lý thuyết:** Đảm bảo một lớp chỉ có duy nhất một instance và cung cấp một điểm truy cập toàn cục cho nó.
*   **Triển khai thực tế:**
    *   **Eager Singleton:** `DatabaseConfig` (khởi tạo ngay để quản lý EntityManager).
    *   **Lazy Sync Singleton:** `UserDAOImpl` (khởi tạo khi cần, dùng `synchronized`).
    *   **Double-Checked Locking:** `RedisConfig` (hiệu năng cao, an toàn đa luồng).
*   **Luồng thực thi:** Bất cứ khi nào Service cần truy cập DB hoặc Redis, nó gọi `getInstance()`. Hệ thống đảm bảo không tạo ra hàng nghìn kết nối thừa thãi, giúp tiết kiệm tài nguyên server.

### 5. DAO Pattern (Data Access Object)
*   **Lý thuyết:** Tách biệt logic nghiệp vụ (Business Logic) khỏi logic truy cập dữ liệu (Persistence Logic).
*   **Triển khai thực tế:** Interface `UserDAO` và lớp thực thi `UserDAOImpl`.
*   **Luồng thực thi:**
    1.  `UserService` muốn tìm user theo email.
    2.  Nó không viết câu lệnh SQL/JPQL trực tiếp mà gọi `userDAO.findByEmail()`.
    3.  `UserDAOImpl` chịu trách nhiệm mở `EntityManager`, thực hiện query và trả về kết quả. Nếu mai sau đổi từ Postgres sang MySQL, ta chỉ cần sửa ở lớp DAO.
