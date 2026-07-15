---
slug: ung-dung/he-thong-bo-loc
---

# Hệ thống bộ lọc

Hệ thống lọc giúp bạn tìm kiếm và lọc dữ liệu trong bảng của mình. Thay vì cuộn qua nhiều bản ghi, hãy tạo điều kiện tìm kiếm để tìm chính xác thứ bạn cần. Tìm nút **Bộ lọc** - nó hiển thị "Bộ lọc" bình thường và "Bộ lọc (N)" khi hoạt động.

## Cách lọc dữ liệu

**Bắt đầu lọc:**
1. Nhấp vào nút **Bộ lọc** trên thanh công cụ bảng của bạn
2. Nhấp vào **"+ Thêm bộ lọc"** để tạo điều kiện đầu tiên của bạn

**Xây dựng điều kiện:**
1. **Chọn một trường** từ danh sách thả xuống đầu tiên (các cột trong bảng của bạn hoặc các trường trong bảng có liên quan)
2. **Chọn cách so sánh** từ danh sách thả xuống thứ hai (bằng, chứa, lớn hơn, v.v.)  
3. **Nhập giá trị tìm kiếm** vào hộp nhập (nếu cần)

**Áp dụng bộ lọc của bạn:**
- Nhấp vào **"Áp dụng bộ lọc"** để tìm kiếm dữ liệu của bạn
- Bảng của bạn cập nhật để chỉ hiển thị các bản ghi phù hợp
- Nút Bộ lọc hiện hiển thị "Bộ lọc (X)" để cho biết nó đang hoạt động

## Nhiều điều kiện và logic

**Thêm điều kiện khác:**
- Nhấp vào **"+ Thêm bộ lọc"** lần nữa để biết thêm tiêu chí tìm kiếm
- Chọn **"VÀ"** nếu tất cả các điều kiện đều đúng
- Chọn **"HOẶC"** nếu bất kỳ điều kiện nào có thể đúng

**Logic phức tạp nhóm:**
- Nhấp vào **"+ Thêm nhóm"** để tạo điều kiện lồng nhau
- Mỗi nhóm có logic AND/OR riêng
- Sử dụng công cụ này cho các tìm kiếm phức tạp như: *(tên chứa "John" HOẶC email chứa "john") AND status = "active"*

**Xóa điều kiện:**
- Nhấp vào nút **X** ở bất kỳ điều kiện nào để loại bỏ nó
- Nhấp vào **"Xóa tất cả"** để xóa mọi thứ và bắt đầu lại

## Sử dụng bộ lọc đã lưu

**Tự động lưu:**
- Khi bạn áp dụng các bộ lọc, chúng sẽ tự động được lưu để sử dụng sau
- Mỗi bộ lọc có một tên thông minh như "status = active" hoặc "name contains John"

**Bộ lọc tái sử dụng:**
- Nhấp vào bất kỳ bộ lọc đã lưu nào để áp dụng ngay lập tức
- Các bộ lọc phổ biến (được sử dụng nhiều nhất) xuất hiện ở trên cùng
- Đổi tên bộ lọc bằng cách nhấp vào nút chỉnh sửa
- Xóa các bộ lọc không mong muốn bằng nút thùng rác

**Tìm kiếm Bộ lọc đã lưu:**
- Nếu bạn có nhiều bộ lọc đã lưu, hãy sử dụng hộp tìm kiếm để tìm những bộ lọc cụ thể

## Ví dụ về bộ lọc phổ biến

**Tìm kiếm văn bản:**
- Tìm khách hàng: `tên chứa "Smith"`
- Tìm email: `email kết thúc bằng "@company.com"`
- Tìm mô tả trống: `mô tả trống`

**Phạm vi số:**
- Tìm sản phẩm đắt tiền: `price > 100`
- Tìm độ tuổi: `tuổi từ 25 đến 65`
- Tìm đơn hàng gần đây: `total >= 50`

**Lọc ngày:**
- Bản ghi gần đây: `created_date > "2024-01-01"`
- Phạm vi ngày: `order_date trong khoảng từ "2024-01-01" đến "2024-12-31"`

**Nhiều điều kiện:**
- Khách hàng đang hoạt động ở New York: `status = "active" AND city = "New York"`
- Khách hàng Premium hoặc VIP: `plan = "premium" HOẶC plan = "VIP"`

**Lọc quan hệ:**
- Đơn hàng từ khách hàng cụ thể: `tên khách hàng chứa "John"`
- Sản phẩm thuộc một số danh mục nhất định: `tên danh mục = "Điện tử"`
- **Lưu ý**: Các trường quan hệ này đến từ các mối quan hệ bảng mà bạn tạo - xem [Bắt đầu](../getting-started/getting-started.md) để thiết lập các mối quan hệ và [Hệ thống bộ chọn quan hệ](./relation-picker.md) để chọn các bản ghi liên quan

## Mẹo để lọc tốt hơn

**Giữ nó đơn giản:**
- Bắt đầu với một điều kiện, thêm nhiều điều kiện nếu cần
- Sử dụng tên mô tả rõ ràng khi đổi tên các bộ lọc đã lưu
- Xóa các bộ lọc cũ bạn không còn sử dụng nữa

**Hiệu suất:**
- Bộ lọc đơn giản (như trạng thái = "hoạt động") là nhanh nhất
- Các mối quan hệ lồng nhau phức tạp có thể mất nhiều thời gian hơn để tải
- Sử dụng các điều kiện cụ thể thay vì tìm kiếm rất rộng

**Tổ chức:**
- Lưu các bộ lọc thường dùng để truy cập nhanh
- Sử dụng các nhóm cho logic "và/hoặc" phức tạp- Đặt tên rõ ràng cho các bộ lọc tùy chỉnh của bạn (ví dụ: "Khách hàng đang hoạt động ở NY" thay vì "Bộ lọc 1")
