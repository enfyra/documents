---
slug: ung-dung/xem-nhat-ky
---

# Xem nhật ký máy chủ

Trang Nhật ký Máy chủ (`/settings/admin/logs`) cung cấp giao diện hiện đại để theo dõi và kiểm tra các tệp nhật ký phụ trợ.

## Yêu cầu truy cập

- **Quản trị viên gốc**: Toàn quyền truy cập
- **Quyền**: hành động `read` trên tuyến `/logs`

## Tổng quan

Giao diện nhật ký hiển thị:

- **Thẻ thống kê**: Tổng số tệp, tổng kích thước và trạng thái theo dõi trực tiếp
- **Danh sách tệp**: Lưới các tệp nhật ký có nút tải xuống
- **Trình xem nhật ký**: Trình xem toàn màn hình với tìm kiếm và hành động

## Loại tệp nhật ký

| Biểu tượng | Loại | Mô tả |
|------|------|-------------|
|  Sọ | `crash-*.log` | Tai nạn nghiêm trọng, ngoại lệ chưa được phát hiện |
|  Cảnh báo | `lỗi-*.log` | Nhật ký cấp độ lỗi (HTTP 500+) |
|  Quả cầu | `access-*.log` | Nhật ký truy cập |
|  Lỗi | `gỡ lỗi-*.log` | Nhật ký gỡ lỗi |
|  Tập tin | `app-*.log` | Nhật ký ứng dụng chung |

## Xem nội dung nhật ký

1. Nhấp vào bất kỳ thẻ tệp nhật ký nào để mở trình xem
2. Trình xem hiển thị các mục nhật ký ở định dạng JSON
3. Sử dụng nút quay lại trình duyệt hoặc nút đóng để quay lại danh sách tập tin

### Hành động của người xem nhật ký

| Hành động | Mô tả |
|--------|-------------|
|  Tìm kiếm | Tìm kiếm theo ID nhật ký hoặc ID tương quan |
|  Sao chép | Sao chép nội dung hiển thị vào clipboard |
| ⬇ Tải xuống | Tải xuống 10.000 dòng cuối cùng |
|  Tải lại | Làm mới nội dung nhật ký |
|  Đóng | Quay lại danh sách tập tin |

## Nhật ký tìm kiếm

### Phát hiện ID thông minh

Đầu vào tìm kiếm tự động phát hiện loại ID:

- **ID nhật ký** (bắt đầu bằng `log_`): Tìm mục nhật ký cụ thể
- **ID tương quan** (bắt đầu bằng `req_`): Tìm tất cả nhật ký cho một yêu cầu

### Ví dụ
```
log_mmtajhqm_003e_0n5p     Find specific log entry
req_1773672046211_abc123   Find all logs for request
```
### Hành vi tìm kiếm

- Tìm kiếm bị trả lại (độ trễ 500ms)
- Hiển thị thông báo "Không có kết quả" nếu không tìm thấy gì
- Xóa tìm kiếm để trở về nội dung đầy đủ

## Phân trang

Nội dung nhật ký được phân trang:

- Mặc định: 20 dòng/trang
- Nhấp vào nút "Tải thêm" ở phía dưới để tải trang tiếp theo
- Tiếp tục tải cho đến khi không còn nội dung nữa (`hasMore: false`)
- Tải xuống toàn bộ nội dung (tối đa 10.000 dòng)

## Đang tải xuống nhật ký

Nhấp vào nút tải xuống trên bất kỳ thẻ tệp nào hoặc trong trình xem:

- Tải xuống 10.000 dòng cuối cùng
- Định dạng: Văn bản thuần túy với các mục JSON
- Tên file trùng với tên file log

## Bảng điều khiển thống kê

Các thẻ thống kê ở trên cùng hiển thị:

| Thống kê | Mô tả |
|------|-------------|
| Tổng số tệp | Số lượng tệp nhật ký |
| Tổng kích thước | Kích thước kết hợp của tất cả các tệp |
| Trực tiếp | Chỉ báo trạng thái giám sát |

## Mẹo

1. **Yêu cầu theo dõi**: Sử dụng ID tương quan từ phản hồi lỗi đến vòng đời yêu cầu theo dõi
2. **Kiểm tra nhật ký lỗi trước**: Đối với 500 lỗi, hãy kiểm tra `error.log` trước `app.log`
3. **Điều tra sự cố**: Trống `crash.log` = máy chủ hoạt động tốt
4. **Tải xuống để phân tích**: Tải xuống nhật ký lớn để phân tích ngoại tuyến

## Thiết kế đáp ứng

Giao diện thích ứng với kích thước màn hình:

- **Máy tính để bàn**: Lưới 3-4 cột cho tệp, thanh công cụ ngang trong trình xem
- **Máy tính bảng**: Lưới 2 cột, thanh công cụ nhỏ gọn
- **Di động**: Cột đơn, các nút thanh công cụ xếp chồng lên nhau
