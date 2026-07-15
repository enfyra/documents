---
slug: quan-ly-du-lieu
---

# Quản lý dữ liệu

Data Management là nơi tạo, xem, sửa và xóa bản ghi trong bảng. Sau khi tạo bảng, vào **Data** ở sidebar rồi chọn bảng.

> **Điều kiện:** hoàn thành [Cài đặt](./installation.md) và [Bắt đầu](./getting-started.md), đồng thời đã tạo ít nhất một bảng theo [Hướng dẫn tạo bảng](./table-creation.md).

## Cách thao tác dữ liệu hoạt động

Mọi thao tác dữ liệu đi qua backend API:

- Khi tạo/sửa/xóa bản ghi, frontend gửi HTTP request đến backend.
- Backend xử lý request và cập nhật database.
- Frontend nhận response rồi cập nhật UI.

**Frontend không có API riêng**; nó chỉ là client sử dụng API do backend tạo ra.

## Đi tới dữ liệu bảng

1. Click **Data** ở sidebar.
2. Chọn bảng trong submenu.
3. Trang quản lý dữ liệu của bảng sẽ mở ra.

## Chế độ xem bảng dữ liệu

Bạn sẽ thấy table header với tên bảng, action button ở góc trên phải, bảng dữ liệu có phân trang và empty state nếu chưa có bản ghi.

### Các thao tác chính

**Filter**

- Hiện **Filter** khi chưa có filter; thành **Filters (N)** khi đang áp dụng filter.
- Click để mở filter drawer; xem [Filter System](../app/filter-system.md).
- Filter đang hoạt động hiện thành badge trên bảng kèm nút Clear.

**Create**

- Nút **Create** màu xanh có icon dấu cộng.
- Mở form tạo bản ghi; chỉ hiện khi bạn có create permission.

**Select Items**

- Click **Select Items** để vào selection mode; nút chuyển thành **Cancel Selection**.
- Tick checkbox để chọn nhiều bản ghi.
- **Delete Selected (N)** xuất hiện khi có bản ghi được chọn; chỉ hiện khi bạn có delete permission.

**Column Selector**

- Dropdown để hiện/ẩn cột; chọn field cần hiển thị.
- Lựa chọn được lưu cục bộ.

## Xem bản ghi

- Mỗi hàng là một bản ghi; click hàng bất kỳ để xem và sửa chi tiết.
- Date/timestamp hiển thị theo định dạng ngày giờ, boolean là badge Yes/No, long text bị cắt ở 50 ký tự và giá trị rỗng hiện `-`.
- Menu ba chấm bên phải hàng có **Delete** (nếu có quyền) và có thể có action khác theo permission.
- Phân trang xuất hiện ở cuối khi có nhiều trang; dùng Previous/Next hoặc số trang. URL cập nhật số trang để bookmark.

## Tạo bản ghi mới

1. Click **Create** để mở trang form.
2. Form hiện field theo table schema; field bắt buộc có dấu `*`.
3. Relation field có icon bút chì; xem [Relation Picker](../app/relation-picker.md).
4. Default value được điền sẵn, description hiện dưới label, validation chạy khi lưu.
5. Click **Save**. Nếu hợp lệ, bản ghi được tạo, bạn chuyển đến edit page và thấy thông báo thành công.

## Sửa và xóa bản ghi

Click một hàng để mở edit form với giá trị hiện tại. **Save** chỉ bật khi có thay đổi; validation giống form tạo và thay đổi được theo dõi tự động. Để xóa, click **Delete** màu đỏ, xác nhận trong dialog, rồi quay về table view.

## Thao tác hàng loạt

1. Click **Select Items**, tick các hàng cần chọn.
2. Click **Delete Selected (N)**.
3. Xác nhận xóa hàng loạt.
4. Các bản ghi đã chọn bị xóa và bảng tự refresh.

## Làm việc với dữ liệu liên quan

Relation field trong form có icon bút chì. Click để mở relation picker và chọn bản ghi từ bảng khác. Trong bảng, dữ liệu liên quan có thể hiện ID hoặc tên; click hàng để xem đầy đủ trong edit form và dùng filter để tìm theo dữ liệu liên quan.

## Mẹo quản lý dữ liệu

**Hiệu năng:** dùng filter để tìm nhanh, ẩn cột không cần thiết và để pagination tải từng phần.

**Tổ chức:** lưu filter hay dùng, tùy chỉnh cột thấy được cho từng bảng và dùng bulk operation khi thay đổi nhiều bản ghi.

**Điều hướng:** URL cập nhật theo filter và trang nên có thể bookmark view cụ thể; browser back/forward hoạt động bình thường và trạng thái bảng được giữ khi bạn quay lại.
