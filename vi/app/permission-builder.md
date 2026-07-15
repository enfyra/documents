---
slug: ung-dung/trinh-tao-quyen
---

# Trình tạo quyền

Trình tạo quyền là giao diện trực quan để thiết lập các quy tắc kiểm soát truy cập phức tạp. Thay vì viết mã phân quyền, bạn có thể ghép điều kiện bằng các nhóm và toán tử logic.

## Mở Trình tạo quyền

Trường quyền xuất hiện trong nhiều biểu mẫu của Enfyra:

- **Quản lý menu**: thiết lập quyền xem menu.
- **Quản lý tuyến**: cấu hình quyền truy cập tuyến.
- **Quản lý người dùng**: đặt quyền riêng cho người dùng.
- **Biểu mẫu tùy chỉnh**: mọi trường có kiểu quyền.

**Cách nhận biết trường quyền:**

- Có biểu tượng khiên.
- Nhấn vào trường để mở ngăn Trình tạo quyền.

## Thiết lập quyền

### Bước 1: Chọn kiểu quyền

Khi mở Trình tạo quyền, bạn có hai lựa chọn.

**Cho phép toàn bộ truy cập:**

- Bật **"Cho phép tất cả"** để cấp quyền không giới hạn.
- Tùy chọn này bỏ qua mọi kiểm tra quyền.
- Chỉ nên dùng cho trường hợp quản trị cần toàn quyền.

**Tạo quyền tùy chỉnh:**

- Giữ **"Cho phép tất cả"** ở trạng thái tắt để tạo các quy tắc cụ thể.
- Ghép điều kiện chi tiết bằng nhóm và quy tắc.

### Bước 2: Tạo nhóm quyền

**Thêm nhóm:**

1. Nhấn **"+ Thêm nhóm"** để tạo một nhóm điều kiện.
2. Chọn logic của nhóm:
   - **AND**: mọi điều kiện trong nhóm đều phải đúng.
   - **OR**: chỉ cần một điều kiện trong nhóm đúng.

**Ví dụ về nhóm:**

- **Nhóm AND**: người dùng phải có cả quyền "đọc bài viết" và "sửa bài viết".
- **Nhóm OR**: người dùng có vai trò "quản trị viên" hoặc "biên tập viên".

### Bước 3: Thêm quy tắc quyền

**Trong mỗi nhóm:**

1. Nhấn **"+ Thêm quyền"** để tạo quy tắc.
2. **Chọn tuyến**: dùng bộ chọn tuyến để chọn endpoint API cần áp dụng.
3. **Chọn thao tác**: bật những thao tác mà quy tắc này cho phép:
   - **Tạo** (POST): tạo bản ghi mới.
   - **Đọc** (GET): xem hoặc liệt kê bản ghi.
   - **Cập nhật** (PATCH): sửa bản ghi hiện có.
   - **Xóa** (DELETE): xóa bản ghi.

**Bộ chọn tuyến:**

- Tìm tuyến theo tên hoặc đường dẫn.
- Tuyến hiển thị dưới dạng `/ten-bang` hoặc đường dẫn tùy chỉnh.
- Chọn đúng endpoint bạn muốn kiểm soát.

**Nút thao tác:**

- Nhấn vào nhãn thao tác để bật hoặc tắt quyền.
- Thao tác đã bật có màu; thao tác tắt hiển thị mờ.
- Mỗi quy tắc phải có ít nhất một thao tác thì mới hợp lệ.

### Bước 4: Tạo logic phức tạp

**Nhóm lồng nhau:**

- Có thể thêm nhóm bên trong nhóm để mô tả điều kiện phức tạp.
- Ví dụ: `(Quản trị viên OR Biên tập viên) AND (Đọc bài viết OR Viết bài viết)`.

**Nhiều quy tắc:**

- Có thể thêm nhiều quy tắc vào cùng một nhóm.
- Mỗi quy tắc có thể nhắm đến một tuyến khác nhau.
- Kết hợp các tuyến bằng logic AND hoặc OR.

## Ví dụ dùng Trình tạo quyền

### Quyền đơn giản

**Mục tiêu**: cho phép đọc hồ sơ người dùng.

1. **Thêm nhóm** (logic AND mặc định).
2. **Thêm quyền** vào nhóm:
   - **Tuyến**: `/users`.
   - **Thao tác**: Đọc.

### Quyền phức tạp

**Mục tiêu**: quản trị viên hoặc biên tập viên được quản lý bài viết, hoặc mọi người dùng đều được đọc bài viết.

1. **Nhóm chính** (logic OR).
2. **Nhóm con thứ nhất** (logic AND):
   - **Quyền 1**: tuyến `/roles`, thao tác Đọc.
   - **Quyền 2**: tuyến `/posts`, thao tác Tạo, Cập nhật, Xóa.
3. **Quyền thứ hai** (trong nhóm OR chính):
   - **Tuyến**: `/posts`.
   - **Thao tác**: Đọc.

### Ví dụ menu thực tế

**Mục tiêu**: chỉ hiển thị menu "Quản lý người dùng" với người có thể quản lý người dùng hoặc xem báo cáo người dùng.

1. **Nhóm chính** (logic OR).
2. **Quyền 1**: tuyến `/users`, thao tác Tạo, Cập nhật, Xóa.
3. **Quyền 2**: tuyến `/reports/users`, thao tác Đọc.

## Hướng dẫn đọc giao diện

### Trạng thái trường quyền

- **Khiên có dấu chọn**: đã cấu hình quyền.
- **Khiên thường**: đang bật Cho phép tất cả.
- **Khiên có dấu X**: chưa cấu hình quyền.

### Tiêu đề nhóm

- **Nút AND/OR**: chuyển giữa các toán tử logic.
- **Thao tác của nhóm**: thêm quyền, thêm nhóm con hoặc xóa nhóm.

### Quy tắc quyền

- **Nhãn tuyến**: hiển thị đường dẫn tuyến đã chọn.
- **Nhãn thao tác**: hiển thị màu cho các thao tác đang bật.
- **Nút sửa**: điều chỉnh quy tắc.
- **Nút xóa**: xóa quy tắc.

### Dấu hiệu kiểm tra hợp lệ

- **Viền đỏ**: thiếu trường bắt buộc.
- **Thông báo lỗi**: mô tả cụ thể phần chưa hợp lệ.
- **Trạng thái nút lưu**: chỉ bật khi biểu mẫu hợp lệ.

## Tích hợp với các hệ thống khác

### Hệ thống menu

Trình tạo quyền kiểm soát việc hiển thị menu:

- Menu chỉ xuất hiện với người dùng có quyền cần thiết.
- Xem [Quản lý menu](./menu-management.md) để biết cách dùng riêng cho menu.

### Hệ thống tuyến

Kiểm soát quyền truy cập endpoint API:

- Hoạt động cùng Public Methods và Role Permissions.
- Xem [Các thành phần quyền](./permission-components.md) để biết chi tiết tích hợp giao diện.

### Hệ thống biểu mẫu

Được tích hợp vào cơ chế hiển thị biểu mẫu của Enfyra:

- Tự động nhận diện trường quyền.
- Giao diện nhất quán cho mọi trường quyền.
- Xem [Tích hợp với hệ thống biểu mẫu](#tich-hop-voi-he-thong-bieu-mau) bên dưới.

## Tích hợp với hệ thống biểu mẫu

### Tự động nhận diện trường

Enfyra tự nhận diện trường quyền trong biểu mẫu và hiển thị Trình tạo quyền.

### Các kiểu trường

- **Trường quyền**: mở Trình tạo quyền khi nhấn.
- **Trình sửa nội tuyến**: hiển thị tóm tắt quyền và cho phép chỉnh sửa.
- **Trình sửa JSON**: người dùng nâng cao có thể sửa trực tiếp JSON quyền.

### Kiểm tra biểu mẫu

- Trường quyền được kiểm tra tự động.
- Tuyến và thao tác bắt buộc được kiểm tra.
- Không thể gửi biểu mẫu khi quyền chưa hợp lệ.

### Liên kết dữ liệu

- Thay đổi quyền được lưu tự động khi đóng trình tạo.
- Không cần nhấn thêm nút lưu.
- Kiểm tra hợp lệ diễn ra ngay khi chỉnh sửa.

## Thực hành nên áp dụng

### Thiết kế quyền

- **Bắt đầu đơn giản**: dùng quyền cơ bản trước, chỉ thêm độ phức tạp khi cần.
- **Nhóm logic rõ ràng**: nhóm các quyền liên quan với logic AND/OR phù hợp.
- **Mục đích rõ ràng**: mỗi quy tắc cần có một mục đích nghiệp vụ cụ thể.

### Trải nghiệm người dùng

- **Kiểm tra kỹ**: thử quyền với tài khoản kiểm thử.
- **Ghi chú quy tắc phức tạp**: thêm mô tả cho cấu trúc quyền khó đọc.
- **Rà soát định kỳ**: kiểm tra lại quyền để loại bỏ những quyền không còn cần thiết.

### Lưu ý hiệu năng

- **Tránh cấu trúc quá phức tạp**: quá nhiều nhóm lồng nhau có thể ảnh hưởng hiệu năng.
- **Dùng tuyến cụ thể**: ưu tiên tuyến cần thiết thay vì cấp quyền quá rộng.
- **Thân thiện với cache**: cấu trúc quyền đơn giản dễ cache hơn.

## Khắc phục sự cố

### Quyền không hoạt động

- **Kiểm tra đường dẫn tuyến**: đường dẫn phải khớp chính xác endpoint API.
- **Kiểm tra thao tác**: xác nhận đã chọn đúng CRUD cần thiết.
- **Kiểm tra logic nhóm**: bảo đảm AND/OR tạo đúng điều kiện mong muốn.

### Lỗi kiểm tra biểu mẫu

- **Thiếu tuyến**: mỗi quy tắc phải chọn một tuyến.
- **Thiếu thao tác**: mỗi quy tắc phải bật ít nhất một thao tác.
- **Nhóm rỗng**: xóa những nhóm không chứa quy tắc quyền nào.

### Vấn đề hiệu năng

- **Đơn giản hóa cấu trúc**: giảm số nhóm lồng nhau nếu giao diện chậm.
- **Dùng tuyến cụ thể**: ưu tiên tuyến được nhắm chính xác thay vì quyền rộng.
- **Làm mới cache**: thay đổi quyền phức tạp có thể cần làm mới cache.
