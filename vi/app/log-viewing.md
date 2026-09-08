---
slug: ung-dung/xem-nhat-ky
---

# Trace lỗi và log của script

Mở **Settings → Server Logs** (`/settings/admin/logs`) để tìm nguyên nhân thao tác thất bại hoặc xem nội dung từ `@LOGS(...)`. Trang này đọc các record trong database, bao gồm log từ nhiều instance của server.

## Quyền truy cập

Root administrator có thể đọc cả hai tab và toàn bộ chi tiết. Administrator khác cần quyền nhìn thấy menu và quyền `GET` trên `/enfyra_system_error` hoặc `/enfyra_user_log`. Các field private `stack`, `details` và `entries` còn cần field permission cho thao tác đọc. Chỉ có route permission thì chưa đọc được các field private.

## Tìm request bị lỗi

1. Sao chép `correlationId` từ response lỗi và ghi lại thời điểm request thất bại.
2. Chọn tab **System errors**.
3. Chọn khoảng thời gian, dán ID vào ô **Correlation ID**, rồi nhấn **Search / Refresh**.
4. Mở một record để xem mã lỗi, component, instance, source, status và các chi tiết được phép đọc.
5. Nhấn **Find related user logs** để xem output của script có cùng correlation ID.

Nếu chưa có ID, hãy bắt đầu từ khoảng thời gian. Bạn có thể lọc theo component hoặc mã lỗi chính xác. Kết quả mới nhất nằm trước, mỗi trang có 25 record. Nhấn Refresh khi cần cập nhật.

## Ý nghĩa hai tab

| Tab | Nội dung |
|---|---|
| System errors | Lỗi server, lỗi thực thi script, worker crash, lỗi database hoặc bootstrap đã đi tới bộ ghi lỗi |
| User logs | Nội dung được ghi bằng `@LOGS(...)` / `$ctx.$logs(...)`, bao gồm console output của script do executor thu thập |

Record worker crash có thể chứa exit code, exit signal, thông tin bộ nhớ ở lần lấy mẫu gần nhất và ID các script đang chạy. Hãy dùng các chi tiết này để điều tra; riêng dòng “Worker crashed” chưa chứng minh được lỗi hết RAM.

Validation hoặc từ chối quyền hợp lệ không tự động được coi là lỗi hệ thống. Flow execution history và Runtime Monitor vẫn cung cấp trạng thái thực thi và chỉ số đang hoạt động của từng phần.

## Thêm log hữu ích trong script

```javascript
@LOGS('order validation started', { orderId: @BODY.orderId });
// Perform the operation.
@LOGS('order validation finished');
```

Nên ghi các mốc xử lý và identifier cần thiết để hiểu thao tác. Không ghi password, API key, authorization header, toàn bộ request body hoặc dữ liệu mật. Output lưu vào database được sanitize, nhưng cơ chế tự động không thể nhận diện mọi secret nằm trong chuỗi văn bản tùy ý.

Output được nhóm theo task của executor. Một route batch có thể chứa log của pre-hook, handler và post-hook; một flow có thể tạo nhiều record cùng correlation ID. Output quá lớn bị giới hạn và màn hình chi tiết sẽ báo truncation. Field `entries` có thể không xuất hiện nếu bạn chưa có quyền đọc.

## Thời gian lưu và record bị thiếu

Record được giữ 30 ngày, sau đó được dọn theo từng batch giới hạn. Việc ghi log diễn ra ngoài transaction nghiệp vụ của request, vì vậy rollback nghiệp vụ không rollback record chẩn đoán.

Khi database tạm thời không truy cập được, server giữ log trong buffer RAM có giới hạn và thử ghi lại. Buffer đầy hoặc process bị tắt có thể làm mất các record chưa ghi xong. Lỗi xảy ra trước khi schema chẩn đoán tồn tại cũng có thể không có record trong database. Vì vậy, kết quả tìm kiếm rỗng không chứng minh hệ thống không có lỗi. Hãy kiểm tra khoảng thời gian, quyền truy cập và stdout/stderr của container khi điều tra sự cố database hoặc lỗi khởi động nghiêm trọng.

Ứng dụng không ghi hoặc đọc file log cục bộ. Khi nâng cấp, chạy đầy đủ bootstrap upgrade để tạo hai bảng log trước khi sử dụng trang này.

## Tiếp theo

- [Runtime Monitor](./runtime-monitor.md) để xem tình trạng worker, queue và database hiện tại.
- [Ghi log và xử lý lỗi](../server/context-reference/logging-errors.md) để xem ví dụ script.
