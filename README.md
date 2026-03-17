# justFUML Project

## Tổng quan
Project **justFUML** là một dự án quản lý hệ thống thương mại điện tử chuyên về tai nghe và thiết bị âm thanh. Dự án tập trung vào việc mô hình hóa các thực thể (entities) và mối quan hệ giữa chúng thông qua các sơ đồ UML (Class Diagram và Use Case Diagram) sử dụng **PlantUML**.

Hệ thống bao gồm các tính năng cốt lõi như:
- Quản lý người dùng và địa chỉ.
- Quản lý sản phẩm, thương hiệu và danh mục.
- Hệ thống giỏ hàng và đặt hàng.
- Đánh giá sản phẩm (Reviews) và mã giảm giá (Vouchers).

## Cây thư mục (Project Structure)
Dưới đây là cấu trúc thư mục của dự án:

```text
justFUML/
├── .idea/                  # Cấu hình IDE (IntelliJ)
├── src/                    # Thư mục mã nguồn và tài liệu mô hình hóa
│   ├── reference/          # Các tệp tham chiếu chứa mã nguồn Java thực thể
│   │   └── package.txt     # Tổng hợp toàn bộ code Java entity (Address, Brand, Order, v.v.)
│   ├── class_diagram.puml  # Sơ đồ lớp (Class Diagram) chi tiết của hệ thống
│   └── use_case_diagam.puml # Sơ đồ Ca sử dụng (Use Case Diagram) của hệ thống
├── .gitignore              # Các tệp bị loại bỏ khỏi quản lý phiên bản
├── .gitattributes          # Cấu hình thuộc tính Git
└── justPUML.iml            # Tệp cấu hình dự án IntelliJ
```

## Thành phần chính
- **Entities**: Được định nghĩa trong `src/reference/package.txt`, bao gồm các quan hệ JPA như `@OneToMany`, `@ManyToOne`, và các ràng buộc dữ liệu.
- **UML Diagrams**: Các tệp `.puml` trong thư mục `src` giúp trực quan hóa kiến trúc hệ thống, đảm bảo tính nhất quán giữa thiết kế và mã nguồn thực tế.

---
*Dự án này được phát triển như một phần của môn học TKPMHDT (Thiết kế phần mềm hướng đối tượng).*
