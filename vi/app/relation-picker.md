---
slug: ung-dung/chon-ban-ghi-lien-quan
---

# Bộ chọn bản ghi liên quan

Bộ chọn bản ghi liên quan giúp bạn chọn bản ghi có quan hệ khi làm việc với biểu mẫu. Khi thấy **biểu tượng bút chì** cạnh một trường, đó là trường quan hệ kết nối đến bản ghi ở bảng khác. Thay vì nhập id, bạn có thể duyệt và chọn dữ liệu liên quan trực quan.

**Schema và biểu mẫu:** bộ chọn chỉ dùng để **chọn hàng liên quan trong biểu mẫu tạo/sửa**. Việc định nghĩa quan hệ, liên kết **đến** từ bảng khác và **Tạo quan hệ nghịch đảo** (tên thuộc tính nghịch đảo trên bảng hiện tại) được thực hiện trong **Schema bảng > Quan hệ** — xem [Hướng dẫn tạo bảng](../getting-started/table-creation.md#relations-setup).

## Cách dùng trường quan hệ

**Chọn bản ghi liên quan:**

1. **Nhấn vào trường** có biểu tượng bút chì.
2. **Chọn bản ghi** từ danh sách hiện ra.
3. Nhấn **"Áp dụng"** để xác nhận.

**Chọn một bản ghi** (ví dụ chọn một khách hàng):

- Nhấn vào bản ghi để chọn.
- Chọn bản ghi khác sẽ thay thế lựa chọn hiện tại.

**Chọn nhiều bản ghi** (ví dụ chọn nhiều danh mục):

- Nhấn bản ghi để thêm vào hoặc bỏ khỏi lựa chọn.
- Bản ghi đã chọn có dấu chọn.

## Thao tác bổ sung

Bộ chọn chỉ chọn bản ghi đã có. Để tạo bản ghi mới, dùng trang của bảng liên quan rồi tải lại bộ chọn.

**Lọc bản ghi:**

- Dùng **Lọc** để thu hẹp danh sách; bộ chọn không có ô tìm kiếm nhanh riêng.
- Dùng cùng cơ chế lọc như trong hướng dẫn [Hệ thống bộ lọc](./filter-system.md).

**Duyệt trang:**

- Danh sách là bảng có phân trang con trỏ; dùng **Trước** / **Sau** để chuyển trang.
- Chuyển trang không làm mất lựa chọn hiện tại.
- Header của bảng hiển thị tổng số bản ghi khớp bộ lọc hiện tại, không chỉ số dòng trên trang; chuyển trang không làm thay đổi tổng này.

**Xem chi tiết bản ghi:**

- Nhấn **biểu tượng liên kết ngoài** cạnh một bản ghi để mở trang chi tiết.
- Thao tác này không làm thay đổi lựa chọn hiện tại.

## Ví dụ thường gặp

**Bài viết blog — Danh mục:**

- Trường **"Danh mục"** có biểu tượng bút chì.
- **Nhấn vào trường** để xem toàn bộ danh mục.
- **Chọn nhiều danh mục** để đánh dấu lựa chọn.
- **Nhấn "Áp dụng"** và trường hiển thị "Đã chọn 3 danh mục".

**Đơn hàng — Khách hàng:**

- Trường **"Khách hàng"** có biểu tượng bút chì.
- **Nhấn vào trường** để xem toàn bộ khách hàng.
- **Chọn một khách hàng** để thay thế lựa chọn trước đó.
- **Nhấn "Áp dụng"** và trường hiển thị tên khách hàng đã chọn.

**Sản phẩm — Nhà cung cấp:**

- Trường **"Nhà cung cấp"** có biểu tượng bút chì.
- **Nhấn vào trường** để xem các nhà cung cấp.
- **Nhấn Lọc** để tìm theo địa điểm hoặc tên.
- **Chọn nhà cung cấp** và trường hiển thị lựa chọn đó.

## Mẹo

**Tìm bản ghi:**

- Dùng nút Lọc khi có nhiều bản ghi để chọn.
- Tìm tên có ý nghĩa; hệ thống hiển thị tên, tiêu đề hoặc trường nhận diện khác.
- Dùng biểu tượng liên kết ngoài để mở trang chi tiết bản ghi trước khi chọn.

**Làm việc hiệu quả:**

- Với lựa chọn nhiều, có thể chọn nhiều bản ghi trước khi nhấn Áp dụng.
- Nhấn Áp dụng để giữ lựa chọn. Đóng mà chưa áp dụng sẽ bỏ thay đổi nháp; mở lại sẽ khôi phục lựa chọn đã xác nhận gần nhất.
