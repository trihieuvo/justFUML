# Báo Cáo Phân Tích Chuyên Sâu: Thiết Kế Pattern Tái Sử Dụng Của Controller & Facade
**Nhóm phân tích**: Luồng Tra cứu đơn hàng & Luồng Cập nhật địa chỉ
**Chung cơ chế kiến trúc**: Layered Architecture, Command, Factory Method, Facade.

---

## 1. Mở Đầu
Cả cấu trúc Tra cứu đơn hàng và Cập nhật địa chỉ (thuộc hồ sơ Profile), dù khác nhau về nghiệp vụ Business, nhưng lại dùng chung mô hình chia cắt giao tiếp Controller và Service tương đồng một cách chặt chẽ. Hệ thống đã loại trừ thành công luồng if-else cồng kềnh (Fat Controller) bằng 3 trụ cột Design Pattern chủ đạo.

---

## 2. Phân Tích Các Sơ Đồ Thiết Kế

### 2.1. Nhóm Tra cứu Đơn Hàng (Order Tracking)
*Tham chiếu sơ đồ:* `command_facade_order_tracking.puml`

#### A. Command Pattern
*   **Mối quan hệ:** **Inheritance (`--->|>`)** / UML nét liền tam giác rỗng (Biểu diễn `<|--` trong PUML)
    *   *Nơi dùng:* Lớp `TrackOrderCommand` kế thừa từ Abstract class `OrderCommand`.
    *   *Sách định nghĩa:* Lớp con kế thừa lớp cha để tái sử dụng lại các biến thành viên có sẵn (như `facade`, `templateEngine`) và chỉ phải ghi đè phương thức lõi `execute()`. Do dùng Abstract Class thay vì Interface thuần túy, ký hiệu phải là mũi tên nét liền.
*   **Mối quan hệ:** **Dependency (`---->`)** / UML nét đứt (Biểu diễn `..>` trong PUML)
    *   *Nơi dùng:* `OrderTrackingController` đóng vai trò Invoker, mượn cục bộ và gọi phương thức `execute()` của đối tượng `OrderCommand` trả về từ Factory.
    *   *Phân tích:* Controller hoàn toàn không nắm giữ `OrderCommand` thành một biến môi trường dài hạn (không phải Association). Nó chỉ bóc vỏ sinh ra trong hàm `processRequest()` rồi hủy luồng. Điều này giúp Controller hoàn toàn sạch bóng sự lồng ghép dữ liệu tĩnh.

#### B. Factory Method Pattern
*   **Mối quan hệ:** **Dependency (`---->`)** / UML mũi tên chĩa đứt khúc tạo Object
    *   *Nơi dùng:* `OrderTrackingController` dùng `OrderCommandFactory` và trả về `OrderCommand` tương ứng.
    *   *Phân tích:* Centralize việc đọc route. Cung cấp HTTP Method và Path (`/track` hay `/track/submit`), Factory sẽ quăng trả object chứa logic phân luồng.

#### C. Facade Pattern
*   **Mối quan hệ:** **Aggregation (`<>--->`)** / UML khóa tâm
    *   *Nơi dùng:* Lớp Command giữ mốc object trỏ đến `OrderFacade`.
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* `OrderFacade` thao tác gọi xuyên vào `OrderServiceImpl` và `PaymentServiceImpl`.
    *   *Phân tích:* Command tuy nhàn nhưng không thể tự chọc API xuống DAO hay chắp vá từng field của Đơn hàng và Vận chuyển thành cục response trả view. Nó gọi hàm facade như `extractOrderFromRequest()`, lớp Facade sẽ tóm lấy sự bề bộn đó.

---

### 2.2. Nhóm Cập nhật Địa Chỉ Hồ Sơ (Address Update - Profile)
*Tham chiếu sơ đồ:* `command_facade_address_update.puml`

#### A. Command Pattern
*   **Mối quan hệ:** **Inheritance (`--->|>`)** / UML nét liền tam giác rỗng
    *   *Nơi dùng:* Các class như `EditAddressCommand`, `AddAddressCommand`, `DeleteAddressCommand` kế thừa `ProfileCommand`.
    *   *Phân tích:* Ở đây dùng Inherits vì `ProfileCommand` là một **Abstract Class** bảo lưu sẵn instance sinh ra Facade nội bộ (chứa biến `# facade`), code tuân thủ chặt cấp độ Reusable (Dùng chung), khỏi lặp lại khai báo biến ở các command con.

#### B. Factory Method Pattern
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* `ProfileController` gọi thẳng đến `ProfileCommandFactory`. Factory sinh trực tiếp `ProfileCommand` cụ thể.
    *   *Phân tích:* Rất gọn để check request bấm nút xoá, sửa, set mặc định và nạp ra class ứng biến.

#### C. Facade Pattern
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* `UserProfileFacade` gọi `AddressServiceImpl` và `UserServiceImpl`.
    *   *Phân tích:* Mặt tiền (Facade layer) cung cấp cho Command các method sạch sẽ như `facade.updateAddress(...)`. Lớp Facade chịu lỗi bắt dính việc user ảo giả mạo Address và truy xuất database.

---

## 3. Tổng Kết Kiến Trúc
2 Luồng tương quan sở hữu một chu trình (archetype) bất biến: **Controller -> Factory -> Command -> Facade -> Services**. Lớp mô phỏng UML được liên kết đúng với cấp độ Dependency - Association - Inheritance. Cơ cấu này minh chứng rõ nét mục tiêu chia nhỏ trách nhiệm Single Responsibility, giúp developer đằng sau thêm phương thức Tra cứu hay địa chỉ mà không phải sửa Controller khởi điểm ban đầu.
