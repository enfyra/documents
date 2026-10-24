---
slug: ung-dung/lich-su-code
---

# Audit thay đổi code

Dùng Audit để kiểm tra source của phiên bản trước và phục hồi mà không đặt lùi version của record.

## Lưu record code

Khi cập nhật handler, hook, shared helper, WebSocket gateway hoặc event, cấu hình OAuth, flow step hay extension:

1. Sửa và kiểm tra record, rồi chọn Save.
2. Enfyra Admin đọc version hiện tại và mở **Save a new code version**.
3. Kiểm tra patch được đề xuất, ví dụ `3.0.1` lên `3.0.2`. Có thể đổi thành version hợp lệ lớn hơn như `4.0.0`.
4. Chọn **Save version** để gửi update, hoặc **Close** để giữ nguyên draft và tiếp tục sửa.

Modal cũng xuất hiện khi chỉ đổi metadata. Create không bắt buộc modal này: nếu bỏ version thì dùng default `1.0.0`. Cần quyền GET record đích để đọc version hiện tại. Nếu không đọc được hoặc bị từ chối quyền, thao tác lưu dừng thay vì đoán version.

Các update dùng `useApi` chung áp dụng xác nhận này cho cả chín bảng code. Caller gọi HTTP trực tiếp vẫn phải tự gửi version mới hợp lệ.

## Tìm phiên bản cũ

1. Mở **Settings → Audit**, hoặc truy cập `/settings/audit`.
2. Danh sách mặc định ở **All**, gồm mọi bảng có version. Mở menu chọn bảng trên toolbar để thu hẹp về một bảng.
3. Điền ID vào **Exact record ID**, rồi nhấn Enter hoặc chọn **Search** để lọc đúng record đó. Xóa nội dung tìm kiếm và gửi lại để xem toàn bộ record trong phạm vi đang chọn.
4. Xem action, version trước/sau, email người thao tác, loại credential và ngày ghi.
5. Mở một dòng để xem ai đổi, đổi lúc nào, source trước thay đổi và các giá trị khác tại revision đó.

Bảng tải từng trang bằng cursor qua **Previous** và **Next**. Số record cạnh tiêu đề là kích thước của phạm vi đang xem, đọc một lần từ trang đầu nên giữ nguyên khi bạn chuyển trang. Chỉ lấy source lịch sử khi mở dòng được chọn. Bạn cần quyền GET `/enfyra_audit`; mục Audit được ẩn nếu thiếu quyền này.

Chọn các dòng để xóa revision. Nút thùng rác trên toolbar xóa toàn bộ dòng đã chọn sau một lần xác nhận, và số lượng hiển thị ở footer cho biết đang chọn bao nhiêu. Bulk delete cần quyền DELETE trên `/enfyra_audit`; thiếu quyền thì cột chọn và nút này không hiện. Revision đã xóa không xem lại hay phục hồi được nữa, và việc xóa history không làm thay đổi bản thân record.

## Phục hồi thành version mới

Mở một dòng update hoặc revert và chọn **Restore as new version**. Kiểm tra version kế tiếp trong modal xác nhận rồi lưu. Phục hồi tạo thêm một update và một dòng history; không hạ version hay xóa các thay đổi ở giữa.

Ngoài quyền đọc history, bạn cần quyền POST `/admin/code-audit/revert` và PATCH trên bảng đích. Quyền ghi từng field vẫn được áp dụng. Các field riêng tư như OAuth credential và cách gán relation không được phục hồi.

Dòng create không có phiên bản trước đó. Dòng delete chỉ để xem lại; record đã xóa không thể phục hồi. Nếu record đích không còn tồn tại, revert trả lỗi và không tạo lại record.

## Khi lưu hoặc phục hồi thất bại

- **Xung đột version:** tải lại record, kiểm tra thay đổi của người khác rồi mới thử tiếp.
- **Bị từ chối quyền:** kiểm tra riêng quyền history, revert, GET/PATCH bảng đích và quyền field. Hiển thị menu không đồng nghĩa với quyền API.
- **Ghi audit thất bại:** đọc lại record hiện tại trước khi thử lại vì record có thể đã được ghi thành công.

Xem [contract version và độ tin cậy phía server](../server/code-history.md). History không thay thế backup và chưa phải audit đầy đủ cho mutation lồng nhau hoặc xóa cascade.
