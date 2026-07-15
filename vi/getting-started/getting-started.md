---
slug: bat-dau/dang-nhap-lan-dau
---

# Bắt đầu

Sau khi cài xong Enfyra backend và app, hãy làm theo các bước sau.

> **Mới dùng Enfyra?** Bắt đầu từ [Hướng dẫn cài đặt](./installation.md) để thiết lập workspace app + server.

## Đăng nhập lần đầu

1. Mở Enfyra app (mặc định: `http://localhost:3000`).
2. Bạn được chuyển đến trang login.
3. Dùng tài khoản admin đã tạo khi thiết lập backend:
   - Thông tin bạn đã nhập lúc setup backend.
   - Nếu dùng setup mặc định, xem console backend để biết thông tin admin.

## Sau khi đăng nhập

Bạn sẽ thấy giao diện chính của Enfyra.

### Bố cục giao diện

**Bên trái:**

- **Sidebar** – menu điều hướng đầy đủ, có menu và dropdown (mặc định hiện trên desktop; có thể bật/tắt trên mobile).
- **Collapse Toggle** – nút hiện/ẩn sidebar; khi thu gọn chỉ hiện icon.

**Khu vực chính:**

- **Header** – phần trên có tiêu đề trang và action button.
- **Sub-Header** – điều hướng phụ và breadcrumb (`Home > settings > routings`).
- **Content Area** – nội dung chính có nền gradient và pattern nhẹ.

**Header Actions:**

- Nằm ở góc trên bên phải mỗi trang.
- Là các nút tùy theo ngữ cảnh: Filter, Create, Save, Delete, ...
- Nút **Create** màu xanh dùng để thêm item mới.

### Điều hướng sidebar

- **Dashboard** (Grid icon) – tổng quan và số liệu nhanh của hệ thống.
- **Data** (List icon) – xem, tạo, sửa và xóa bản ghi trong bảng.
- **Collections** (Database icon) – tạo và quản lý bảng/schema database.
- **Settings** (Gear icon) – cấu hình hệ thống, user, role và permission.
- **Storage** (Folder icon) – upload và quản lý media, folder, storage configuration.

**Cách sidebar hoạt động:**

- Click menu item để điều hướng hoặc mở/đóng dropdown.
- Desktop: sidebar hiện mặc định, có thể thu về icon.
- Mobile/tablet: sidebar thu gọn mặc định, mở ra thành overlay.
- Dùng toggle để hiện/ẩn menu đầy đủ.

## Bước tiếp theo: xây dựng ứng dụng đầu tiên

Đừng cố học mọi tùy chọn collection trước khi có kết quả. Hãy theo lộ trình ngắn để tạo collection, thêm bản ghi và gọi API tự sinh.

**[Xây dựng ứng dụng đầu tiên](./first-app.md)** – bước tiếp theo được khuyến nghị cho người mới Enfyra.

Hướng dẫn này gồm:

- **Quy trình tạo bảng** – từng bước cụ thể.
- **Các kiểu field** – từ text cơ bản tới rich content và relation.
- **Tính năng nâng cao** – constraint, index và relation.
- **Những gì xảy ra sau khi tạo** – tự động sinh API và tích hợp.

Sau khi hoàn thành, bạn có:

- **4 CRUD endpoint tự động** trên backend server.
- **Tích hợp frontend** trong phần Data.
- **Toàn quyền API** cho ứng dụng bên ngoài.

**Sau đó tiếp tục với:**

- **[Tạo bảng](./table-creation.md)** – kiểu field, relation, index và constraint.
- **[Quản lý dữ liệu](./data-management.md)** – thêm, sửa và quản lý bản ghi trong bảng.
