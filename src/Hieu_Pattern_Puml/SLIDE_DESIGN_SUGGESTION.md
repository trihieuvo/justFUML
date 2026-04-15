# 📊 GỢI Ý THIẾT KẾ SLIDE BÁO CÁO DESIGN PATTERNS - AUDIOGEAR E-COMMERCE

> **Lưu ý quan trọng**: DAO Pattern sẽ được đặt ở cuối cùng như yêu cầu đặc biệt của bạn.

---

## 📋 TỔNG QUAN CẤU TRÚC SLIDE

**Tổng số slide gợi ý**: 14-16 slides
**Thời gian báo cáo**: 15-20 phút
**Phần mềm**: Canva (gợi ý tỷ lệ 16:9)

---

## 📑 CHI TIẾT TỪNG SLIDE

### **SLIDE 1: SLIDE TIÊU ĐỀ** 🎯
**Mục đích**: Tạo ấn tượng đầu tiên, giới thiệu đề tài
**Nội dung**:
- **Tiêu đề lớn**: "ÁP DỤNG DESIGN PATTERNS TRONG HỆ THỐNG AUDIOGEAR E-COMMERCE"
- **Phụ đề**: "Phân tích các giải pháp tối ưu hóa code cho bộ Thanh Toán & Đăng Ký"
- **Tên người trình bày**: [Tên của bạn]
- **Logo**: AudioGear (nếu có)
- **Ngày tháng**: [Ngày báo cáo]

**Gợi ý hình ảnh**:
- Nền gradient xanh dương → tím (hiện đại)
- Biểu tượng pattern nhỏ ở góc (🔗)
- Kiểu chữ: Đậm (Inter/Montserrat Bold)

---

### **SLIDE 2: MỤC LỤC & ĐỊNH HƯỚNG** 📑
**Mục đích**: Giúp người nghe nắm bắt cấu trúc bài báo cáo
**Nội dung**:
1. **Giới thiệu dự án AudioGear** (Slide 3-4)
2. **Module Thanh Toán & Đặt Hàng** (Slide 5-9)
   - Facade Pattern
   - Strategy Pattern
   - Factory Pattern
   - Observer Pattern
3. **Module Đăng Ký & Xác Thực** (Slide 10-11)
   - Facade Pattern (Auth)
4. **DAO Pattern** (Slide 12) - **ĐẶC BIỆT**
5. **Tổng kết & Kết luận** (Slide 13-14)

**Gợi ý hình ảnh**:
- Sử dụng biểu tượng cho từng mục
- Thanh tiến trình ở dưới (thanh tiến trình)
- Màu sắc từng mục khác nhau

---

### **SLIDE 3: GIỚI THIỆU DỰ ÁN AUDIOGEAR** 🎧
**Mục đích**: Giới thiệu bối cảnh và tầm quan trọng của dự án
**Nội dung**:
- **Tên dự án**: AudioGear E-Commerce
- **Mô tả ngắn**: Hệ thống bán hàng thiết bị âm thanh online
- **Công nghệ**: Java EE, Kiến trúc MVC, JPA
- **Module phức tạp**:
  - 🛒 Thanh Toán & Đặt Hàng
  - 🔐 Đăng Ký & Xác Thực
- **Thách thức**: Codebase khó maintain, khó mở rộng

**Gợi ý hình ảnh**:
- Ảnh mockup website AudioGear
- Tập hợp biểu tượng công nghệ (Java, Database, v.v.)
- Infographic về số lượng users/orders (nếu có)

---

### **SLIDE 4: PHÂN TÍCH VẤN ĐỀ GẶP PHẢI** ⚠️
**Mục đích**: Đặt vấn đề, giải thích tại sao cần Design Patterns
**Nội dung** (2 cột hoặc 2 card):

**Cột 1 - Module Thanh Toán**:
- ❌ Controller gọi 7-8 service riêng lẻ
- ❌ Logic if-else lồng nhau cho từng phương thức thanh toán
- ❌ Khó thêm phương thức mới (VNPay, ZaloPay)
- ❌ Gửi email làm chậm response

**Cột 2 - Module Auth**:
- ❌ Nhiều bước: check spam → tạo user → sinh OTP → lưu cache → gửi mail
- ❌ Code Controller ~200 dòng
- ❌ Khó unit test
- ❌ SQL/JPQL nằm lẫn trong service

**Gợi ý hình ảnh**:
- Biểu tượng X đỏ cho mỗi vấn đề
- Số liệu thống kê: "200+ dòng code", "7 service calls"
- Mũi tên hỗn loạn biểu thị coupling cao

---

### **SLIDE 5: FACADE PATTERN - ĐƠN GIẢN HÓA WORKFLOW** 🎭
**Mục đích**: Pattern đầu tiên - tổng quan nhất
**Nội dung**:
- **Định nghĩa**: "Cung cấp giao diện thống nhất cho một tập hợp các interface trong hệ thống con"
- **Vấn đề giải quyết**: Ẩn sự phức tạp của hệ thống con khỏi người dùng
- **Ứng dụng**: `CheckoutServiceImpl` là Facade cho 7 hệ thống con

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `facade_checkout.puml`
- **Minh họa**: Ví dụ "Gọi đồ ăn" - Khách chỉ cần gọi "Giao đồ ăn", không cần biết bếp, shipper làm gì
- **Đoạn code mẫu** (nhỏ, làm nổi bật dòng chính):
```java
// Controller chỉ cần
checkoutService.checkout(userId, request);
```

**Bố cục Canva**:
- Biểu đồ ở trái (chiếm 2/3)
- Ghi chú và code ở phải (1/3)
- Màu nền: xanh nhạt (#E3F2FD)

---

### **SLIDE 6: LUỒNG THỰC THI FACADE - CHECKOUT** 🔍
**Mục đích**: Chi tiết hóa luồng thực thi
**Nội dung** (Timeline hoặc từng bước):
1. **Bước 1**: Kiểm tra dữ liệu đầu vào
2. **Bước 2**: Bắt đầu Giao dịch Cơ sở dữ liệu (Atomic)
3. **Bước 3**: Kiểm tra tồn kho
4. **Bước 4**: Kiểm tra & áp dụng voucher
5. **Bước 5**: Xử lý thanh toán (Strategy)
6. **Bước 6**: Tạo Order & OrderItem, trừ kho
7. **Bước 7**: Xác nhận/Hủy giao dịch
8. **Bước 8**: Gửi email thông báo (Observer)

**Gợi ý hình ảnh**:
- Sơ đồ luồng hoặc Timeline dọc
- Biểu tượng cho mỗi bước
- Màu xanh cho thành công, đỏ cho đường lỗi
- Tham chiếu code: `CheckoutServiceImpl.java:48-200`

---

### **SLIDE 7: STRATEGY PATTERN - ĐA DẠNG THANH TOÁN** 🎮
**Mục đích**: Pattern thứ hai - xử lý phương thức thanh toán
**Nội dung**:
- **Định nghĩa**: "Định nghĩa một tập hợp các thuật toán, đóng gói chúng lại và làm chúng có thể hoán đổi"
- **Vấn đề giải quyết**: Thay vì if-else, mỗi phương thức thanh toán là 1 class riêng
- **Ứng dụng**: Interface `PaymentStrategy` với 5 lớp thực thi

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `strategy_payment.puml`
- **Minh họa**: Ví dụ "Tính tiền ở siêu thị" - Tiền mặt, Thẻ, QR
- **Đoạn code mẫu** cho 2-3 strategies:
```java
// COD luôn success
public PaymentResult pay(BigDecimal amount) {
  return PaymentResult.success("Thanh toán khi nhận hàng");
}

// Momo sinh transactionId
public PaymentResult pay(BigDecimal amount) {
  String id = "MOMO-" + UUID.randomUUID();
  return PaymentResult.success("Thanh toán qua Momo", id);
}
```

**Bố cục Canva**:
- Biểu đồ ở trái
- Bảng so sánh: if-else (❌) vs Strategy (✅) ở phải
- Màu nền: tím nhạt (#F3E5F5)

---

### **SLIDE 8: FACTORY PATTERN - TẠO STRATEGY OBJECT** 🏭
**Mục đích**: Pattern hỗ trợ Strategy - tạo đối tượng
**Nội dung**:
- **Định nghĩa**: "Cung cấp giao diện tạo đối tượng mà không cần chỉ định chính xác lớp cụ thể sẽ được tạo"
- **Vấn đề giải quyết**: Ẩn logic khởi tạo, tránh `new` trực tiếp
- **Ứng dụng**: `PaymentStrategyFactory` dùng Map + Supplier

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `factory_payment.puml`
- **Minh họa**: Hình ảnh nhà máy sản xuất (Factory) tạo ra các sản phẩm (Strategy)
- **Đoạn code mẫu** Factory:
```java
private static final Map<String, Supplier<PaymentStrategy>> STRATEGIES = Map.of(
  "COD", CODStrategy::new,
  "MOMO", MomoStrategy::new,
  "BANK", BankTransferStrategy::new
);

public static PaymentStrategy fromCode(String code) {
  return STRATEGIES.get(code.toUpperCase()).get();
}
```

**Bố cục Canva**:
- Biểu đồ ở trên (tràn ngang)
- 3 thẻ nhỏ dưới: Lợi ích của Factory
- Màu nền: cam nhạt (#FFF3E0)

---

### **SLIDE 9: OBSERVER PATTERN - THÔNG BÁO BẤT ĐỒNG BỘ** 🔔
**Mục đích**: Pattern xử lý thông báo sau checkout
**Nội dung**:
- **Định nghĩa**: "Định nghĩa sự phụ thuộc một-nhiều. Khi đối tượng thay đổi trạng thái, các đối tượng phụ thuộc sẽ được thông báo"
- **Vấn đề giải quyết**: Gửi email không làm chậm response, giảm coupling
- **Ứng dụng**: `OrderNotificationService` là Observer

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `observer_order.puml`
- **Minh họa**: Ví dụ "YouTube Subscribe" - Khi video upload, người subscribe nhận thông báo
- **Đoạn code mẫu** với try-catch:
```java
public void notifyProcessing(Order order) {
  try {
    String html = renderTemplate(order);
    emailService.sendHtmlEmail(...);
  } catch (Exception e) {
    e.printStackTrace(); // Không rollback order
  }
}
```
- **Lưu ý**: Observer ≠ State Pattern (câu này để tránh nhầm lẫn)

**Bố cục Canva**:
- Biểu đồ ở giữa
- 2 mũi tên: Subscribe → Order Created → Notify
- Màu nền: xanh lá nhạt (#E8F5E9)

---

### **SLIDE 10: AUTH FACADE - WORKFLOW ĐĂNG KÝ** 🔐
**Mục đích**: Áp dụng Facade vào module Auth
**Nội dung**:
- **Vấn đề**: Đăng ký gồm nhiều bước phức tạp
- **Giải pháp**: `AuthFacade` đơn giản hóa thành 1 method
- **Luồng**:
  1. Kiểm tra chống spam (Redis)
  2. Tạo User PENDING
  3. Sinh OTP
  4. Lưu OTP vào Redis
  5. Gửi email

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `facade_auth.puml`
- **Đoạn code mẫu**:
```java
public void requestRegister(RegisterRequest request) {
  redisService.canSendOtp(request.getEmail());
  userService.registerPendingUser(request);
  String otp = emailService.generateOTP();
  redisService.saveRegisterOtp(request.getEmail(), otp);
  emailService.sendOtp(request.getEmail(), otp);
}
```

**Bố cục Canva**:
- Biểu đồ ở trái
- Sơ đồ luồng 5 bước ở phải
- Màu nền: xanh dương nhạt (#E3F2FD)

---

### **SLIDE 11: SO SÁNH BEFORE vs AFTER** 📊
**Mục đích**: Tổng hợp hiệu quả của các Pattern
**Nội dung** - Bảng 2 cột:

| Vấn Đề | Before ❌ | After ✅ |
|--------|-----------|----------|
| **Thanh toán** | if-else khó mở rộng | Strategy + Factory |
| **Checkout** | Controller 200 dòng | Facade: 10 dòng |
| **Thông báo** | Làm chậm response | Observer (async) |
| **Database** | SQL trong Service | DAO decoupled |
| **Auth** | Nhiều bước lộn xộn | AuthFacade gọn gàng |

**Gợi ý hình ảnh**:
- Bảng màu sắc rõ ràng (đỏ vs xanh lá)
- Số liệu cụ thể: "85% code reduction", "10x faster response"
- Biểu tượng emoji cho mỗi hàng

---

### **SLIDE 12: DAO PATTERN - TÁCH BIỆT DATA ACCESS** 💾
**Mục đích**: **PATTERN ĐẶC BIỆT** - đặt ở cuối cùng
**Nội dung**:
- **Định nghĩa**: "Cung cấp giao diện trừu tượng cho cơ sở dữ liệu hoặc cơ chế lưu trữ khác"
- **Vấn đề giải quyết**: Tách logic database khỏi business logic
- **Ứng dụng**: Interface `UserDAO` + `UserDAOImpl` với JPA
- **Đặc biệt**: Kết hợp với Singleton Pattern

**Gợi ý hình ảnh**:
- **Biểu đồ**: Sử dụng `dao_user.puml`
- **Minh họa**: Hình ảnh "Hộp đen" - Service không cần biết bên trong DB làm gì
- **Đoạn code mẫu**:
```java
// Interface - Không phụ thuộc JPA
public interface UserDAO {
  Optional<User> findByEmail(String email);
  User save(User user);
}

// Thực thi - Bao gồm JPQL
public class UserDAOImpl implements UserDAO {
  public Optional<User> findByEmail(String email) {
    EntityManager em = DatabaseConfig.getInstance().getEntityManager();
    return em.createQuery(...).getSingleResult();
  }
}
```

**Làm nổi bật**:
- Khi đổi DB (MySQL → MongoDB): Chỉ cần sửa `UserDAOImpl`, Service không đổi
- Dễ mock cho unit test
- Singleton: `UserDAOImpl.getInstance()` tiết kiệm resource

**Bố cục Canva**:
- Biểu đồ ở trái
- 3 thẻ lợi ích ở phải
- **Màu nền đặc biệt**: vàng/vàng gold gradient (#FFF9C4 → #FFD700)
- **Khung viền đậm** để nhấn mạnh đây là pattern đặc biệt

---

### **SLIDE 13: LỢI ÍCH TỔNG QUÁT** ✅
**Mục đích**: Tóm tắt tác động tích cực
**Nội dung** (5 thẻ hoặc bullet points):

1. **Maintainability** (SRP): Mỗi class chỉ 1 trách nhiệm
2. **Extensibility** (OCP): Thêm tính năng không sửa code cũ
3. **Testability**: Dễ mock, độ bao phủ unit test cao
4. **Reusability**: Strategy, DAO tái sử dụng ở nhiều nơi
5. **Loose Coupling**: Module ít phụ thuộc, dễ thay thế

**Gợi ý hình ảnh**:
- 5 hình lục giác hoặc hình tròn
- Mỗi biểu tượng có màu khác nhau
- Mô tả ngắn phía dưới mỗi biểu tượng
- Nền: trắng/xám nhạt

---

### **SLIDE 14: KẾT LUẬN & ĐỀ XUẤT** 🎯
**Mục đích**: Khép lại bài báo cáo, mở ra câu hỏi
**Nội dung**:

**Kết luận**:
"Design Patterns không chỉ là lý thuyết mà đã giải quyết thực tế vấn đề của AudioGear, giúp codebase dễ bảo trì, mở rộng và test hơn rất nhiều."

**Đề xuất hướng phát triển**:
- State Pattern cho Order Lifecycle
- Decorator Pattern cho Pricing Engine
- Proxy Pattern cho Caching Layer

**Mời câu hỏi**:
💬 "Cảm ơn sự lắng nghe. Em sẵn sàng trả lời các câu hỏi từ hội đồng."

**Gợi ý hình ảnh**:
- Phần cảm ơn lớn (70% slide)
- Thông tin liên hệ nhỏ ở góc dưới (Email, GitHub)
- Nền: gradient đẹp, có thể dùng ảnh blur nhẹ
- Nút "Questions?" nổi bật

---

### **SLIDE 15 (TÙY CHỌN): Q&A / THAM KHẢO** 📚
**Nội dung**: Dành riêng cho phần hỏi đáp
- **Tham khảo biểu đồ**: Link đến các file PlantUML
- **Mã nguồn**: `src/main/java/vn/edu/ute/`
- **Các file quan trọng**:
  - `CheckoutServiceImpl.java`
  - `AuthFacade.java`
  - `UserDAOImpl.java`

**Gợi ý hình ảnh**: Tối giản, tập trung vào văn bản và liên kết

---

## 🎨 MẸO THIẾT KẾ TRONG CANVA

### **Bảng màu** (gợi ý):
- **Màu chính**: #2563EB (xanh dương - hiện đại)
- **Màu phụ**: #7C3AED (tím - sáng tạo)
- **Thành công**: #10B981 (xanh lá - OK)
- **Cảnh báo**: #F59E0B (vàng cam - chú ý)
- **Lỗi**: #EF4444 (đỏ - lỗi)
- **Trung tính**: #6B7280 (xám - chữ)

### **Kết hợp kiểu chữ**:
- **Tiêu đề**: Montserrat Đậm / Inter Đậm (48-60pt)
- **Nội dung**: Inter Thường / Open Sans (24-32pt)
- **Code**: JetBrains Mono / Fira Code (18-20pt)

### **Thành phần hình ảnh**:
1. **Biểu đồ**: Chụp ảnh từ PlantUML → import vào Canva
2. **Biểu tượng**: Sử dụng Flaticon/Canva Elements (thống nhất style line)
3. **Minh họa**: Thêm hình ảnh mỗi pattern
4. **Khối code**: Dùng khối code trong Canva hoặc ảnh chụp
5. **Chú thích**: Dùng mũi tên, hình tròn để làm nổi bật code quan trọng

### **Mẫu bố cục**:
- Slide biểu đồ: 2/3 biểu đồ + 1/3 ghi chú
- Slide so sánh: Chia đôi màn hình (50/50)
- Slide code: Khối code to ở giữa, giải thích trên/dưới
- Slide lợi ích: Lưới 2x2 hoặc 3x2

---

## ⏱️ PHÂN BỔ THỜI GIAN BÁO CÁO

| Phần | Thời gian | Slide |
|------|-----------|-------|
| Mở đầu & Giới thiệu | 2 phút | 1-4 |
| Module Thanh Toán | 8 phút | 5-9 |
| Module Auth | 3 phút | 10-11 |
| DAO Pattern (Đặc biệt) | 3 phút | 12 |
| Tổng kết & Q&A | 2 phút | 13-14 |
| **Tổng** | **18 phút** | **14 slides** |

---

## 🎯 CÂU HỎI PHẢN BIỆN CÓ THỂ GẶP

Chuẩn bị sẵn các slide phụ (ẩn) để trả lời:
1. **"Tại sao không dùng DI thay vì new()?**" → Slide giải thích quyết định đơn giản
2. **"Factory có cần thiết không?**" → Slide so sánh Factory vs Enum
3. **"SePayStrategy return success mà chưa thanh toán?**" → Slide giải thích QR pending

---

## 📝 DANH SÁCH KIỂM TRA TRƯỚC KHI BÁO CÁO

- [ ] Xuất tất cả biểu đồ PlantUML sang PNG/SVG chất lượng cao
- [ ] Import vào Canva và đặt đúng vị trí
- [ ] Kiểm tra kiểu chữ thống nhất toàn bộ slide
- [ ] Thêm số trang
- [ ] Test hiển thị trên màn hình lớn (chữ không quá nhỏ)
- [ ] Chuẩn bị ghi chú cho người trình bày
- [ ] Xuất PDF dự phòng nếu Canva không hoạt động

---

## 📂 FILE OUTPUT CẦN CHUẨN BỊ

1. **PDF**: `AudioGear_DesignPatterns_Report.pdf` (Canva export)
2. **Hình ảnh**:
   - `facade_checkout.png`
   - `facade_auth.png`
   - `factory_payment.png`
   - `strategy_payment.png`
   - `observer_order.png`
   - **(Đặc biệt)** `dao_user.png` (lưu riêng folder)
3. **Dự phòng**: PowerPoint version (nếu cần)

---

**Chúc bạn báo cáo thành công! 🎉**

---

**Cập nhật lần cuối**: [Ngày hiện tại]
**Tạo bởi**: AI Assistant dựa trên Dự án AudioGear
