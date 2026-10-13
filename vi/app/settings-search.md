---
slug: ung-dung/tim-nhanh-settings
---

# Tìm nhanh bản ghi trong Settings

Dùng ô Search của bảng để tìm bản ghi mà không cần mở bộ lọc nâng cao. Mỗi tìm kiếm nhanh trong Settings chỉ nhắm đến một field.

| Trang Settings | Field tìm kiếm |
|---|---|
| Users | `email` |
| Roles | `name` |
| Routes | `path` |
| Guards | `name` |
| Flows | `name` |
| Extensions | `name` |
| WebSockets | `path` |
| Helpers | `name` |
| API Tester | `path` |
| OAuth Accounts | `user.email` |
| Server Logs | Tra cứu chính xác `correlationId` hiện có |

Path của menu liên kết không phải path riêng của extension, nên Extensions tìm theo tên. Methods, Auth Headers, OAuth Config và Menus không có ô tìm nhanh.

## Gửi hoặc xóa tìm kiếm

1. Nhập giá trị của field. Gõ chỉ thay đổi draft, chưa gửi request.
2. Nhấn Enter hoặc bấm **Search** để gửi. Khoảng trắng đầu/cuối được bỏ, danh sách trở về trang một và text đã gửi xuất hiện ở `?search=`.
3. Xóa text rồi gửi để bỏ điều kiện tìm kiếm. Gửi lại cùng nội dung khi đang ở trang một sẽ refresh danh sách.

Tìm nhanh dùng toán tử substring `_contains` được hỗ trợ, không dùng `_icontains`. OAuth Accounts dùng filter lồng `user.email`. Text trông giống JSON vẫn là giá trị literal của field, không được diễn giải thành filter object. Việc khớp tuân theo ngữ nghĩa `_contains` của server/database, không bảo đảm chế độ không phân biệt hoa thường.

Filter nâng cao hiện có được giữ thành một nhóm hoàn chỉnh và kết hợp với tìm kiếm bằng AND. Scope System/Custom/All và trạng thái Collection Routes của Routes, loại Guard, trạng thái Helper và scope API Tester vẫn có hiệu lực. Query URL không liên quan được giữ nguyên; Back/Forward khôi phục tìm kiếm đã gửi và draft trong ô. Tổng số khi lọc dùng `meta.filterCount`, kể cả giá trị zero.

## Bố cục mobile

Dưới 768px, Search chiếm toàn hàng đầu trong header của bảng và đẩy các control còn lại xuống dưới. Desktop giữ tiêu đề dataset, Search và quick filter bên trái, custom action và Columns bên phải. Xem [DataTable](./data-table.md) để biết cách bố trí toolbar/footer trong extension.
