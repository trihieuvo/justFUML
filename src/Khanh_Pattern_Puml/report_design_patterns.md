# Báo Cáo Phân Tích Chuyên Sâu: Thiết Kế UML Class Diagram Cho Design Patterns
**Dự án:** AudioGear E-Commerce
**Mục tiêu:** Báo cáo giải trình sự lựa chọn các mối quan hệ (Relationships) giữa các Class và Object, tuân thủ nghiêm ngặt tiêu chuẩn trong giáo trình *"Dive Into Design Patterns"*.

---

## 1. Mở Đầu: Lý Thuyết Nền Tảng (Quy Chuẩn Sách)
Theo chuẩn sơ đồ UML trong giáo trình, chúng ta sử dụng các mũi tên tương ứng với cấp độ phụ thuộc từ lỏng lẻo nhất đến chặt chẽ nhất:
1. **Dependency (`---->`)**: Lớp A bị ảnh hưởng bởi thay đổi của lớp B (Sài tạm thời).
2. **Association (`--->`)**: Object A biết về Object B (Có field nhưng không rõ vòng đời).
3. **Aggregation (`<>--->`)**: Object A chứa Object B, nhưng B có thể tồn tại bên ngoài A.
4. **Composition (`<đặc>--->`)**: Object A chứa Object B và quản trị vòng đời (sống/chết) của B.
5. **Implementation (`----|>`)**: Lớp A thực thi giao diện B.
6. **Inheritance (`--->|>`)**: Lớp A kế thừa lớp B.

---

## 2. Phân Tích Các Sơ Đồ Thiết Kế Trong Hệ Thống

### 2.1. Nhóm Creational Patterns (Mẫu Khởi Tạo)

#### A. Singleton Pattern (Eager & Thread-Safe Double-Checked)
*   **Tham chiếu file:** `singleton_eager_database.puml` và `singleton_threadsafe_redis.puml`
*   **Đặc điểm mũi tên:** **Không có mũi tên liên kết rời rạc bên ngoài.**
*   **Lý do chọn lựa:** Bản chất của Singleton là **"Đứng một mình độc tôn"**. Object không được tồng hợp hay tạo ra bởi Class khác. Việc sơ đồ chỉ thể hiện 1 class duy nhất với biến `private static instance` thể hiện đúng tính chất "Tự quản trị vòng đời của chính mình" mà không lệ thuộc vào interface bên ngoài.

#### B. Builder Pattern (Quản lý Sản Phẩm)
*   **Tham chiếu file:** `builder_product.puml`
*   **Mối quan hệ:** **Implementation (`--|>`)**
    *   *Nơi dùng:* `DefaultProductBuilder` trỏ tới `ProductBuilder`.
    *   *Lý do theo sách:* Class `DefaultProductBuilder` định nghĩa (defines) các phương thức khai báo từ interface `ProductBuilder`.
*   **Mối quan hệ:** **Composition (`<đặc>--->`)**
    *   *Nơi dùng:* Từ `DefaultProductBuilder` trỏ tới `Product`.
    *   *Lý do theo sách:* Object A knows B, consists of B, and manages B's life cycle. `DefaultProductBuilder` làm nhiệm vụ lắp rắp và quản trị hoàn toàn sự sống còn của bản nháp `Product` cho đến khi `build()` được xuất ra.
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* `ProductServiceImpl` trỏ tới `ProductBuilder`.
    *   *Lý do theo sách:* Class A can be affected by changes in B. Class Service không chứa biến builder, nó chỉ mượn (uses) builder tạm thời trong thân hàm.

---

### 2.2. Nhóm Structural Patterns (Mẫu Cấu Trúc)

#### C. Adapter Pattern (Xác thực Google)
*   **Tham chiếu file:** `adapter_auth.puml`
*   **Mối quan hệ:** **Implementation (`--|>`)**
    *   *Nơi dùng:* `GoogleToUserAdapter` trỏ về `OAuthUserAdapter`.
    *   *Lý do theo sách:* Adapter class đang thể hiện (implements) những khai báo của Target Interface.
*   **Mối quan hệ:** **Composition (`<đặc>--->`)**
    *   *Nơi dùng:* `GoogleToUserAdapter` trỏ về `GoogleProfile`.
    *   *Lý do theo sách:* Object A consists of B and manages life cycle. Cái Adapter là một cái vỏ rỗng tuếch nếu mất đi Adaptee là cái GoogleProfile nằm trong lõi nó. Do đó, quan hệ này là khăng khít sống còn (Composition).
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* `GoogleLoginStrategy` (Client) trỏ tới `OAuthUserAdapter` (Target).
    *   *Lý do theo sách:* Class A bị ảnh hưởng bởi B vì Strategy lệ thuộc vào cấu hình Interface mà không thực sự "sở hữu" nó dưới dạng property.

---

### 2.3. Nhóm Behavioral Patterns (Mẫu Hành Vi)

#### D. Strategy Pattern (Xử lý Đăng Nhập)
*   **Tham chiếu file:** `strategy_auth.puml`
*   **Mối quan hệ:** **Implementation (`--|>`)**
    *   *Nơi dùng:* `LocalLoginStrategy` trỏ lên `LoginStrategy`.
    *   *Lý do theo sách:* Thuật toán tuân thủ khung mãnh lệnh (Interface rules).
*   **Mối quan hệ:** **Aggregation (`<>--->`)**
    *   *Nơi dùng:* `AuthServiceImpl` (Context) trỏ về `LoginStrategy`.
    *   *Lý do theo sách:* Object A knows B, consists of B. Lớp Context có 1 Map gồm nhiều Strategy. Tuy nhiên, các thuật toán không bị ép sinh tử (life cycle managed) cùng Context; chúng có thể được dùng độc lập và chỉ là danh sách tập hợp.

#### E. State Pattern (Luồng Đơn Hàng)
*   **Tham chiếu file:** `state_order.puml`
*   **Mối quan hệ:** **Implementation (`--|>`)**
    *   *Nơi dùng:* Các `ConcreteState` trỏ về `OrderState`.
    *   *Lý do theo sách:* Chúng đều là con của interface.
*   **Mối quan hệ:** **Aggregation (`<>--->`)**
    *   *Nơi dùng:* `OrderContext` trỏ về `OrderState`.
    *   *Lý do theo sách:* Mỗi đối tượng Đơn Hàng luôn đính kèm Trạng thái thay đổi liên tục. Sự chuyển trạng thái chính là việc tráo Object - biểu hiện thuần khiết của Aggregation lỏng lẻo thay vì Composition ràng buộc.
*   **Mối quan hệ:** **Dependency (`---->`)**
    *   *Nơi dùng:* Giữa `PendingState` với `ProcessingState`.
    *   *Lý do theo sách:* Biến đổi từ A sang B làm A tác động và sinh ra B.

#### F. Observer Pattern (Sự Kiện Thông Báo)
*   **Tham chiếu file:** `observer_order.puml`
*   **Mối quan hệ:** **Implementation (`--|>`)**
    *   *Nơi dùng:* Tại `OrderSubject` và `OrderObserver`.
    *   *Lý do theo sách:* Phân định trách nhiệm Nhà xuất bản và Người đăng ký.
*   **Mối quan hệ:** **Aggregation (`<>--->`)**
    *   *Nơi dùng:* Lớp `OrderServiceImpl` quản trị danh sách `List<OrderObserver>`.
    *   *Lý do theo sách:* Danh sách người nhận (Observer) nằm trong Subject nhưng vòng đời thì độc lập. Hủy Observer không ảnh hưởng Subject và ngược lại.

#### G. Chain of Responsibility (Bảo Mật Bộ Lọc)
*   **Tham chiếu file:** `chain_of_responsibility_auth.puml`
*   **Mối quan hệ:** **Inheritance (`--->|>`)**
    *   *Nơi dùng:* `TokenValidationHandler` trỏ cha là `AuthHandler`.
    *   *Lý do theo sách:* Đây là **Inherits** (chữ nét liền tam giác rỗng) vì cha là Abstract Class chứa mã nguồn thực thi `handleNext()`, chứ không phải là Interface thuần túy.
*   **Mối quan hệ:** **Aggregation (`<>--->`)**
    *   *Nơi dùng:* Lớp `AuthHandler` tự tạo mối đính kèm tới chính lớp của nó (`next: AuthHandler`).
    *   *Lý do theo sách:* Tạo thành phần móc nối đệ quy. Object A biết Object B (chính là bản thể sao của nó) để liên kết tạo thành cái xích. Cuộn mắt xích này không cố định mà có thể được hoán đổi.

---

## 3. Tổng Kết
Sự sử dụng các mối quan hệ (Dependency, Aggregation, Composition, Implementation, Inheritance) trong mã nguồn dự án mang tính chuyên môn cao, đối chiếu sát với nguyên tắc trừu tượng hoá hướng đối tượng trong ngành Công Nghệ Phần Mềm.
