---
slug: ung-dung/quy-tac-cot
---

# Quy tắc cột

Quy tắc cột cho phép bạn đính kèm các ràng buộc xác thực bổ sung (tối thiểu/tối đa, độ dài, mẫu, định dạng, v.v.) vào một cột mà không cần viết bất kỳ mã nào. Các quy tắc được thực thi phía máy chủ mỗi khi API nhận được `POST` hoặc `PATCH` đối với bảng — các tải trọng không hợp lệ sẽ bị từ chối bằng HTTP 400 và danh sách các thông báo lỗi mà con người có thể đọc được.

Các quy tắc được định cấu hình hoàn toàn từ giao diện người dùng quản trị viên; không cần tạo bản ghi thủ công.

## Khi quy tắc chạy

Các quy tắc áp dụng cho việc xác thực nội dung trên điểm cuối REST động của bảng:

- `POST /<table_name>` (tạo)
- `PATCH /<table_name>/<id>` (cập nhật)

Chúng **chỉ** chạy khi bảng có `validateBody = true` (mặc định cho các bảng mới). Nếu `validateBody` bị tắt, các quy tắc sẽ bị bỏ qua — máy chủ chỉ thực thi hình dạng cơ bản (loại, tính chất vô hiệu) ở lớp lưu giữ lâu dài.

Các quy tắc **không** áp dụng cho:
- `NHẬN` yêu cầu
- Các tuyến tùy chỉnh bỏ qua đường ống CRUD tiêu chuẩn
- Các bản ghi được chèn trực tiếp thông qua lệnh gọi kho lưu trữ bên trong trình xử lý/hook (những bản ghi này được máy chủ tin cậy)

## Chuyển đổi `validateBody` trên một bảng

1. Mở **Bộ sưu tập > [bảng của bạn]** (hoặc bất kỳ bảng hệ thống nào bạn sở hữu).
2. Mở biểu mẫu bảng (bánh răng/chỉnh sửa).
3. Bật hoặc tắt **Xác thực nội dung**.
4. Lưu.

Mặc định cho các bảng mới tạo là **bật**.

## Thêm quy tắc vào cột

Quy tắc hiển thị bên cạnh các cột trong trình chỉnh sửa bảng.

1. Mở bảng trong **Bộ sưu tập** (hoặc **Cài đặt > Bảng** cho các bảng hệ thống).
2. Xác định vị trí hàng cột trong bảng cột.
3. Nhấp vào **biểu tượng thước kẻ** trong hàng cột (bên cạnh biểu tượng lá chắn cấp phép trường).
4. Phương thức **Quản lý quy tắc** mở ra với danh sách quy tắc hiện có cho cột đó.
5. Nhấp vào **Thêm quy tắc**.
6. Chọn **Loại quy tắc** từ danh sách thả xuống (chỉ những loại có ý nghĩa đối với loại dữ liệu của cột mới được hiển thị; xem ma trận bên dưới).
7. Điền vào trường **Value** — trình chỉnh sửa sẽ điều chỉnh theo loại quy tắc (nhập số, biểu thức chính quy + cờ, thả xuống định dạng).
8. Tùy ý điền **Message** để ghi đè nội dung lỗi mặc định.
9. Chuyển đổi **Đã bật** (mặc định là bật).
10. Nhấp vào **Lưu**.

## Tham chiếu loại quy tắc

| Loại quy tắc | Áp dụng cho | Hình dạng Giá trị | Hiệu ứng |
|---|---|---|---|
| `min` | int, bigint, float, thập phân | `{ v: số }` | Số phải ≥ v |
| `tối đa` | int, bigint, float, thập phân | `{ v: số }` | Số phải là ≤ v |
| `độ dài phút` | varchar, văn bản, richtext, mã, uuid | `{ v: số }` | Độ dài chuỗi phải ≥ v |
| `Độ dài tối đa` | varchar, văn bản, richtext, mã, uuid | `{ v: số }` | Độ dài chuỗi phải ≤ v |
| `mẫu` | varchar, văn bản, richtext, mã | `{ v: chuỗi, cờ?: chuỗi }` | Phải khớp `RegExp(v, flags)` |
| `định dạng` | varchar, văn bản, mã | `{ v: 'email' \| 'url' \| 'uuid' \| 'ngày giờ' }` | Phải khớp với định dạng đã đặt tên |
| `minItems` | chọn mảng | `{ v: số }` | Độ dài mảng phải ≥ v |
| `maxItems` | chọn mảng | `{ v: số }` | Độ dài mảng phải ≤ v |
| `tùy chỉnh` | bất kỳ | (dạng tự do) | Dành riêng - hiện đang đi qua; sử dụng pre-hook để có logic tùy chỉnh hoàn toàn |

Phương thức Quản lý quy tắc ẩn các loại quy tắc không tương thích trên mỗi cột và ngăn việc thêm cùng một loại quy tắc hai lần (ngoại trừ `tùy chỉnh`).

## Quan trọng: Quy tắc là bổ sung, không thay thế

Quy tắc **không bao giờ** thay thế các kiểm tra tích hợp của cột. Máy chủ luôn thực thi (theo thứ tự này):

1. **Loại** (từ `column.type` — int, varchar, boolean, v.v.)
2. **Tính chất rỗng** (từ `column.isNullable` — null bị từ chối nếu `false`)
3. **Giới hạn độ dài** cho `varchar` (từ `options.length`)
4. **Sau đó là các quy tắc của bạn**, bên cạnh các quy tắc trênVì vậy, bạn không thể sử dụng quy tắc để *đặt* một trường thành null, *thay đổi* loại của trường đó hoặc xóa giới hạn độ dài — đó là các thuộc tính của chính cột đó. Chỉnh sửa cột để thay đổi chúng.

Không có loại quy tắc `bắt buộc`. Tính chất bắt buộc xuất phát từ `isNullable = false` trên cột, cộng với việc cột đó có giá trị mặc định hay không.

## Vô hiệu hóa và xóa quy tắc

- **Tắt** (tắt) — giữ bản ghi quy tắc nhưng ngừng thực thi. Hữu ích cho việc gỡ lỗi tạm thời.
- **Xóa** (biểu tượng thùng rác) — xóa hoàn toàn quy tắc.

Việc vô hiệu hóa bộ đệm xảy ra tự động; lược đồ trình xác thực mới được xây dựng lại trong vòng ~50 mili giây trên tất cả các phiên bản.

## Thông báo lỗi tùy chỉnh

Khi một quy tắc không thành công, nội dung phản hồi sẽ liệt kê từng vi phạm dưới dạng một chuỗi. Theo mặc định, thông báo được tạo từ loại và giá trị quy tắc (ví dụ: `"tên: Chuỗi phải chứa ít nhất 3 ký tự"`).

Đặt trường **Thông báo** trên quy tắc để ghi đè văn bản. Chuỗi đơn giản - không có biến mẫu.

## Hình dạng phản hồi lỗi

Xác thực không thành công trả về:
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "statusCode": 400,
  "message": [
    "name: String must contain at least 3 character(s)",
    "email: Invalid email"
  ],
  "error": "Bad Request"
}
```
`message` luôn là một **mảng các chuỗi**. Các biểu mẫu giao diện người dùng có thể lặp lại và liên kết từng mục nhập với trường tương ứng của nó (tiền tố trước `:` là tên trường).

## Xác thực tầng (Tạo lồng nhau)

Khi bảng chấp nhận các bản ghi lồng nhau (ví dụ: `POST /post` có `comments: [{ body: '...' }]` nội tuyến), các quy tắc con từ bảng liên quan cũng được xác thực — miễn là bảng liên quan cũng có `validateBody = true`. Tải trọng kết nối theo id (`{ tác giả: 5 }` hoặc `{ tác giả: { id: 5 } }`) bỏ qua xác thực lồng nhau vì không có bản ghi mới nào được tạo.

## Bảng hệ thống

Một số bảng hệ thống (`enfyra_table`, `enfyra_field_permission`, v.v.) chấp nhận một tập hợp nhỏ các trường ảo không phải là cột thực (`graphqlEnabled`, `config`). Trình xác thực cho phép những cái đó theo tên; mọi thứ khác bên ngoài lược đồ đều bị từ chối vì "không được phép". Bạn không cần phải làm gì đặc biệt đối với các bảng hệ thống — các quy tắc cũng hoạt động theo cách tương tự.

## Liên quan

- Phía máy chủ: việc xác thực được triển khai trong `buildZodFromMetadata` (`src/shared/utils/zod-from-metadata.ts`). Các quy tắc được lưu trữ trong `ColumnRuleCacheService`.
- Truy cập đọc/ghi ở cấp độ trường: xem [Quyền của trường](../server/field-permissions.md).
- Xác nhận/xem trước lược đồ khi chỉnh sửa bảng: xem [xem trước di chuyển lược đồ](../server/schema-migration-preview.md).
