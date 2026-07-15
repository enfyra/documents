---
slug: xem-truoc-chuyen-doi-luoc-do
---

# Xem trước thay đổi schema

Khi thay đổi schema của một bảng bằng **`PATCH /api/enfyra_table/:id`** (trên giao diện quản trị hoặc qua API), Enfyra có thể yêu cầu bạn xác nhận bản xem trước trước khi áp dụng thay đổi có nguy cơ mất dữ liệu. Cơ chế này giữ hành vi nhất quán giữa các database adapter và tránh xóa dữ liệu ngoài ý muốn.

## Khi nào cần xem trước

- Việc xóa **column**, **relation** hoặc một thay đổi phá hủy khác sẽ đặt `isDestructive` trong policy preview.
- Client phải gửi **`schemaConfirmHash`** (hoặc **`schema_confirm_hash`**) khớp với hash được tạo từ payload preview của server. Nhờ vậy server biết bạn đã xem xét thay đổi.

Nếu hash bị thiếu hoặc không khớp, API trả về **`422`** với mã **`SCHEMA_CONFIRM_HASH_MISMATCH`** (hoặc chỉ trả preview khi chưa gửi hash).

## Những gì cần đọc trong preview

Các trường thường gặp trong `details`:

| Trường | Ý nghĩa |
|-------|---------|
| `removedColumns` / `removedRelations` | Thành phần sẽ biến mất khỏi schema |
| `addedColumns` / `addedRelations` | Thành phần mới được thêm |
| `renamedColumns` / `changedColumns` | Đổi tên hoặc thay đổi thuộc tính |
| `requiredConfirmHash` | Giá trị gửi lại dưới dạng `schemaConfirmHash` trong PATCH thực tế sau khi xem xét |
| `owningSideInverseCascadeWarnings` | Cảnh báo được giải thích bên dưới |

## `owningSideInverseCascadeWarnings`

Relation cung cấp **`mappedBy`** (tên thuộc tính inverse ở bảng còn lại) và **`mappedByRelationId`** ổn định (id của relation owning) trong metadata cache.

Khi bạn **xóa relation ở phía owning** và relation đó **không** có `mappedBy` (nghĩa là nó là phía owning), các bảng khác có thể vẫn chứa bản ghi `enfyra_relation` **inverse** trỏ đến relation này. Xóa bản ghi owning sẽ **xóa cascade** các relation inverse đó.

Preview liệt kê các trường hợp này để bạn có thể điều chỉnh các bảng khác trước, hoặc chủ động chấp nhận cascade. **Chỉ xóa một relation inverse** sẽ không tạo cảnh báo này.

## Quy trình qua API

1. Gửi **PATCH** với payload `columns` / `relations` mong muốn nhưng **không** gửi `schemaConfirmHash` (hoặc gửi hash cũ) để nhận **`preview: true`** và toàn bộ `details`.
2. Xem `details`, đặc biệt là `owningSideInverseCascadeWarnings` và các trường bị xóa.
3. Gửi **PATCH** lần nữa với cùng ý định thay đổi schema và thêm hash vào **query parameter**: `?schemaConfirmHash=<details.requiredConfirmHash>` (bí danh: `schema_confirm_hash`). Server đọc giá trị từ `$query`, không phải JSON body.

Bạn cũng có thể dùng Enfyra Admin; giao diện này thực hiện đúng quy trình xác nhận trên.

## Migration metadata khi khởi động

Server snapshot có thể khai báo các thay đổi column dựa trên metadata trong `data/snapshot-migration.json` qua `columnsToModify`. Trong lần khởi động đầu tiên, Enfyra áp dụng các thay đổi đó trước khi đồng bộ metadata để column vật lý và bản ghi metadata luôn khớp nhau.

Migration đổi tên column phải mang tính tổng quát và dựa trên bảng. Migration runner đọc tên bảng cùng `oldName` / `newName` trong snapshot migration, đổi tên field SQL hoặc Mongo khi cần, cập nhật metadata của column, và tự xử lý trường hợp trùng column cũ/mới: giữ field đích rồi bỏ field cũ. Đừng viết logic sửa chữa riêng cho một bảng metadata nếu cùng hợp đồng `columnsToModify` có thể mô tả thay đổi đó.

Với metadata HTTP method, nhãn method duy nhất là `enfyra_method.name`. Hãy dùng `name` trong metadata, dữ liệu mặc định, migration, tool, filter và API payload.

## Liên quan

- [Hướng dẫn tạo bảng](../getting-started/table-creation.md) — thứ tự relation và bảng
- [Quyền theo trường](./field-permissions.md) — `enfyra_field_permission` xác định column/relation theo id
