---
slug: ung-dung/luong-tu-dong
---

# Luồng - Hướng dẫn giao diện người dùng

Các luồng được quản lý từ **Cài đặt > Luồng** trong thanh bên.

## Danh sách luồng

Trang danh sách hiển thị tất cả các luồng dưới dạng thẻ có:
- Tên luồng và mô tả
- Huy hiệu loại kích hoạt (Lịch trình, Thủ công)
- Chuyển đổi trạng thái (Hoạt động/Không hoạt động)
- Đếm bước và thời gian chờ
- Nhấp vào thẻ để mở trình chỉnh sửa quy trình

**Hành động:**
- Nút **Tạo luồng** (trên cùng bên phải) — mở biểu mẫu tạo
- **Công tắc bật tắt** trên mỗi thẻ — bật/tắt luồng mà không cần mở
- Nút **Xóa** — xóa luồng (có xác nhận)

## Tạo luồng

Biểu mẫu tạo thu thập:
- **Tên** — mã định danh luồng duy nhất
- **Mô tả** — luồng làm gì
- **Loại kích hoạt** — chọn từ Lịch trình, Thủ công
- **Cấu hình kích hoạt** — các trường được tạo tự động dựa trên loại trình kích hoạt
- **Hết thời gian** — thời gian thực hiện tối đa tính bằng mili giây

Sau khi lưu, bạn sẽ được chuyển hướng đến trình chỉnh sửa quy trình.

## Trình chỉnh sửa luồng

Trang soạn thảo có ba phần:

### Cài đặt luồng (trên cùng)

Chỉnh sửa tên luồng, mô tả, loại trình kích hoạt, cấu hình trình kích hoạt và thời gian chờ. Phần cấu hình trình kích hoạt thay đổi linh hoạt:
- **Lịch trình**: Nhập biểu thức cron + thả xuống múi giờ
- **Hướng dẫn sử dụng**: Không cần cấu hình. Kích hoạt thông qua "Chạy ngay", API hoặc `@TRIGGER()` từ trình xử lý/hook

Nhấp vào **Lưu** trong tiêu đề để tiếp tục thay đổi.

### Các bước trong quy trình (canvas)

Canvas luồng trực quan được cung cấp bởi Vue Flow hiển thị:
- **Nút kích hoạt** (màu hổ phách) ở trên cùng — hiển thị loại và cấu hình trình kích hoạt
- **Nút bước gốc** được kết nối theo chiều dọc ở giữa — thực hiện tuần tự
- **Nút điều kiện** chia thành hai nhánh:
  - **Nhánh thật** (bên phải, cạnh màu xanh lá cây) — các bước chạy khi điều kiện đúng
  - **Nhánh sai** (bên trái, cạnh màu đỏ) — các bước chạy khi điều kiện sai
- **Huy hiệu nhánh** trên các nút bước cho biết chúng thuộc nhánh nào (màu xanh lá cây "đúng" / màu đỏ "sai")
- **Nút "Thêm bước"** (nét đứt) ở dưới cùng — nhấp để thêm bước mới

**Nhấp vào nút bước bất kỳ** để mở ngăn soạn thảo bước.

### Lịch sử thực thi (dưới cùng)

Hiển thị luồng chạy gần đây với:
- Huy hiệu trạng thái (đang chờ xử lý, đang chạy, đã hoàn thành, không thành công, đã hủy)
- Thời gian bắt đầu
- Thời lượng
- Nút **Tải lại** để làm mới danh sách
- **Nhấp vào bất kỳ lệnh thực thi nào** để mở ngăn chi tiết hiển thị:
  - Trạng thái, thời lượng, thời gian bắt đầu/hoàn thành
  - Luồng dừng ở bước nào (có lý do nếu thất bại)
  - Danh sách các bước đã hoàn thành
  - Thông báo lỗi và dấu vết ngăn xếp nếu thất bại

Nút **Chạy ngay** trong tiêu đề phụ sẽ kích hoạt quy trình theo cách thủ công.

## Ngăn kéo trình chỉnh sửa bước

Mở khi nhấp vào nút bước hoặc "Thêm bước". Lĩnh vực:

- **Khóa** — mã định danh duy nhất được sử dụng trong chuỗi dữ liệu (`@FLOW.<key>`)
- **Đơn hàng** — số thứ tự thực hiện
- **Hết giờ** — thời gian chờ cho mỗi bước tính bằng mili giây
- **Loại** — chọn loại bước (mỗi loại hiển thị các trường cấu hình khác nhau)
- **Config** — biểu mẫu dành riêng cho loại:
  - **Script/Condition**: Trình soạn thảo mã (JavaScript) — sử dụng cú pháp mẫu: `@FLOW_PAYLOAD`, `@FLOW_LAST`, `@FLOW.<key>`, `#table_name`, `@HELPERS`
  - **Truy vấn**: Bộ chọn bảng (tự động hoàn thành) + Trình tạo bộ lọc (trực quan) + Giới hạn + Trường
  - **Tạo**: Bộ chọn bảng + JSON dữ liệu
  - **Cập nhật**: Bộ chọn bảng + ID bản ghi + JSON dữ liệu
  - **Xóa**: Bộ chọn bảng + ID bản ghi
  - **HTTP**: URL + Phương thức thả xuống + JSON tiêu đề + JSON nội dung
  - **Luồng kích hoạt**: Tên hoặc ID luồng
  - **Ngủ**: Thời lượng tính bằng mili giây
  - **Nhật ký**: Nội dung tin nhắn
- **Bị lỗi** — Dừng luồng / Bỏ qua bước / Thử lại
- **Thử lại** — số lần thử lại (hiển thị khi Bật Lỗi = Thử lại)
- **Điều kiện gốc** — thả xuống để gán bước cho nhánh của điều kiện (hiển thị khi luồng có các bước điều kiện)- **Nhánh** — Đúng hoặc Sai (hiển thị khi Điều kiện gốc được chọn)

### Nút kiểm tra

Nhấp vào **Kiểm tra** để chạy ngay bước này mà không cần lưu. Kết quả xuất hiện nội tuyến:
- **Thành công**: dấu kiểm màu xanh lục + thời lượng + xem trước kết quả (chỉ đọc)
- **Thất bại**: thông báo lỗi X + màu đỏ

Điều này cho phép bạn xác minh logic bước trước khi thực hiện thay đổi.

### Hành động
- **Xóa** — xóa bước này (chỉ hiển thị khi chỉnh sửa bước hiện tại)
- **Hủy** — đóng ngăn mà không lưu
- **Tạo/Cập nhật** — lưu bước này

## Trạng thái URL

Việc mở ngăn bước hoặc chi tiết thực thi sẽ thêm các tham số truy vấn vào URL (`?editStep=...` hoặc `?exec=...`). Điều này có nghĩa là:
- Nút quay lại trình duyệt đóng ngăn kéo
- Liên kết trực tiếp tới công việc của trình chỉnh sửa bước cụ thể
- Làm mới trang duy trì trạng thái ngăn kéo

## Bộ chọn bảng

Được sử dụng trong Truy vấn, Tạo, Cập nhật, Xóa cấu hình bước. Tính năng:
- Tự động điền đầu vào (gõ để tìm kiếm)
- Tìm kiếm phía máy chủ với thời gian gỡ lỗi 500ms bằng bộ lọc `_contains`
- Tải 10 kết quả cùng một lúc
- Chỉ báo tải trong quá trình tìm kiếm

## Tham khảo nhanh cú pháp mẫu

Sử dụng các macro này trong mã bước Tập lệnh và Điều kiện. Chúng được tự động dịch sang cú pháp `$ctx` khi được lưu.

| Viết cái này | Được chuyển đổi thành |
|----------||-------------------|
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` |
| `@TRIGGER` | `$ctx.$trigger` (các luồng kích hoạt từ trình xử lý) |
| `@FLOW_LAST` | `$ctx.$flow.$last` |
| `@FLOW.step_key` | `$ctx.$flow.step_key` |
| `@FLOW_META` | `$ctx.$flow.$meta` |
| `#enfyra_user` | `$ctx.$repos.enfyra_user` |
| `@NGƯỜI GIÚP ĐỠ` | `$ctx.$helpers` |
| `@TRIGGER(...)` | Kích hoạt một luồng khác từ trình xử lý |
| `@USER` | `$ctx.$user` |
| `@THROW404(tin nhắn)` | `$ctx.$throw['404'](msg)` |
| `%dayjs` | `$ctx.$pkgs.dayjs` |

### Ví dụ
```javascript
// In a script step:
const user = await #enfyra_user.find({
  filter: { email: { _eq: @FLOW_PAYLOAD.email } },
  limit: 1
});
if (!user.data[0]) @THROW404('User not found');
return user.data[0];
```

