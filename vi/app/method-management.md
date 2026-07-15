---
slug: ung-dung/quan-ly-phuong-thuc
---

# Quản lý phương thức

Quản lý phương thức kiểm soát các bản ghi HTTP method dùng trong biểu mẫu tuyến, bộ chọn handler/hook, kiểm thử API và nhãn method của Enfyra App.

## Mở trình quản lý phương thức

1. Vào **Cài đặt > Phương thức**.
2. Xem các method hiện có và màu nhãn của chúng.
3. Nhấn **Phương thức mới** để tạo method tùy chỉnh.

Mỗi bản ghi method gồm:

- **Tên**: khóa method viết hoa, duy nhất, lưu tại `enfyra_method.name`; ví dụ `GET`, `POST`, `PATCH`, `DELETE`, `PUT` hoặc khóa tùy chỉnh như `CUSTOM_METHOD`.
- **Màu nền**: `buttonColor`, dùng cho nền nhãn.
- **Màu chữ**: `textColor`, dùng cho chữ trên nhãn.
- **Hệ thống**: đánh dấu method có sẵn của runtime.

## Tạo phương thức

Ngăn tạo mới không chọn sẵn method nào. Chọn một method phổ biến, hoặc chọn **Tùy chỉnh** khi danh sách có sẵn không phù hợp.

Ưu tiên chọn màu theo thứ tự:

1. Chọn một cặp màu được gợi ý.
2. Chỉ dùng **màu hex tùy chỉnh** khi không có cặp màu phù hợp.
3. Phần xem trước hiển thị nhãn method với màu nền và màu chữ đã chọn.
4. Lưu method.

Giá trị màu phải là mã hex đầy đủ, chẳng hạn `#dbeafe` và `#1d4ed8`.

## Sửa phương thức

Mở **Sửa** trên thẻ method để cập nhật cặp màu. Với method hệ thống, hãy giữ nguyên khóa method; chỉ đổi màu trừ khi bạn chủ động chịu trách nhiệm cho ảnh hưởng đến runtime.

Khi ngăn có thay đổi chưa lưu, thao tác đóng sẽ yêu cầu xác nhận. Chọn **Tiếp tục chỉnh sửa** để quay lại, hoặc **Bỏ thay đổi** để đóng và mất phần chỉnh sửa cục bộ.

## Dùng phương thức trong tuyến

Biểu mẫu tuyến dùng bộ chọn method cho các trường như **Phương thức khả dụng**, **Phương thức công khai** và mục tiêu method của handler/hook. Nếu method cần dùng chưa tồn tại, nhấn nút `+` trong bộ chọn để mở trình quản lý phương thức.

Các method được công bố quyết định một HTTP method là công khai hay cần quyền. Quyền tuyến cũng dùng bản ghi method để xác định vai trò đã xác thực nào có thể gọi một tuyến.

Biểu mẫu tuyến và quyền lưu id quan hệ của method, không lưu chuỗi thô. Id method là dữ liệu theo từng instance, vì vậy hãy tra method bằng `enfyra_method.name` qua giao diện hoặc MCP trước khi gán các trường như **Phương thức khả dụng**, **Phương thức công khai**, method hook, method handler hoặc quyền tuyến.

## Công cụ MCP

Khi quản lý metadata method qua MCP, hãy dùng công cụ method chuyên dụng thay vì CRUD bản ghi tổng quát:

- `list_methods`
- `create_method`
- `update_method`
- `delete_method`

`delete_method` luôn xem trước thay đổi và chỉ nên dùng cho method tùy chỉnh chưa được sử dụng.

Công cụ MCP giữ tên input thân thiện là `method`, nhưng đọc và ghi trường backend `name`. Không dùng payload CRUD tổng quát với trường `method` cho `enfyra_method`.
