---
slug: phan-quyen-truong
---

# Quyền theo trường

Kiểm soát quyền truy cập từng column và relation theo role, người dùng hoặc điều kiện.

## Mặc định hiển thị: `isPublished`

Mọi column và relation đều có cờ `isPublished` (mặc định: `true`).

- `isPublished: true` — mặc định mọi người dùng đã xác thực đều có thể truy cập
- `isPublished: false` — mặc định bị chặn; chỉ truy cập được qua rule `allow` tường minh trong `enfyra_field_permission`

Trường chưa publish bị loại hẳn khỏi API response, không trả về dưới dạng `null`. Root admin bỏ qua toàn bộ kiểm tra quyền theo trường.

Các column nhạy cảm như `password`, OAuth `clientSecret` và thông tin xác thực storage dùng `isPublished: false`.

## enfyra_field_permission

Mỗi rule nhắm tới một column HOẶC một relation, không phải cả hai:

```
POST /api/enfyra_field_permission
{
  "column": { "id": "<enfyra_column_id>" },
  "role": { "id": "<role_id>" },
  "action": "read",
  "effect": "allow",
  "isEnabled": true
}
```

| Trường | Kiểu | Mô tả |
|---|---|---|
| `column` | FK `enfyra_column` | Column đích (null khi nhắm tới relation) |
| `relation` | FK `enfyra_relation` | Relation đích (null khi nhắm tới column) |
| `role` | FK `enfyra_role` | Role mà rule áp dụng |
| `allowedUsers` | M2M `enfyra_user` | Người dùng cụ thể (thay cho role — chỉ dùng một trong hai) |
| `action` | enum: `read`, `create`, `update` | Thao tác mà rule kiểm soát |
| `effect` | enum: `allow`, `deny` | Cho phép hay từ chối truy cập |
| `condition` | JSON | Điều kiện filter DSL tùy chọn, được đánh giá theo từng bản ghi |
| `isEnabled` | boolean | Bật hoặc tắt rule |

Table được suy ra từ FK column/relation, không cần trường `table` riêng.

### Relation inverse

`enfyra_column` và `enfyra_relation` đều có relation inverse `fieldPermissions` trỏ lại `enfyra_field_permission`. Nhờ đó bạn có thể lấy column/relation kèm các permission rule của nó:

```
GET /api/enfyra_column?filter[table][id][_eq]=<tableId>&fields=id,name,fieldPermissions.id,fieldPermissions.effect
```

## Phạm vi áp dụng

Mỗi rule dùng **một trong hai**: `role` hoặc `allowedUsers`, không dùng đồng thời:
- `role` — áp dụng cho mọi người dùng có role đó
- `allowedUsers` — áp dụng cho người dùng cụ thể, không phụ thuộc role
- Không đặt cả hai — global rule, áp dụng cho mọi người

## Điều kiện

JSON filter tùy chọn được đánh giá trên từng bản ghi. Nó dùng filter DSL chuẩn với macro `@USER.id`:

```json
{
  "ownerId": { "_eq": "@USER.id" }
}
```

Điều kiện này chỉ cho phép truy cập khi `ownerId` của bản ghi khớp người dùng hiện tại. Rule có điều kiện được đánh giá sau SQL vì cần dữ liệu bản ghi thực.

## Độ ưu tiên của rule

Khi có nhiều rule khớp, hệ thống đánh giá theo tầng, từ ưu tiên cao xuống thấp:

1. Theo người dùng + có điều kiện
2. Theo role + có điều kiện
3. Theo người dùng + không điều kiện
4. Theo role + không điều kiện

Trong cùng một tầng, `deny` luôn thắng `allow`.

## Luồng áp dụng quyền

```
find()
  1. assertQueryAllowed()     — throw 403 if denied field used in filter/sort/aggregate
  2. stripDeniedFields()      — remove denied columns/relations from SELECT/JOIN (pre-SQL)
  3. queryEngine.find()       — SQL only fetches allowed fields
  4. sanitizeFieldPermissions — safety net for conditional rules (post-SQL)
```

- **Read:** trường bị từ chối bị loại hẳn khỏi response
- **Filter/sort/aggregate:** dùng trường bị từ chối sẽ ném `403`
- **Create/update:** request body chứa trường bị từ chối sẽ ném `403`
- **Ghi cascade:** quyền theo trường cũng được áp dụng cho child record trong cascade create/update

### Tối ưu trước SQL

Việc từ chối không có điều kiện được xử lý trước khi chạy SQL:
- Column bị từ chối bị loại khỏi danh sách SELECT
- Relation bị từ chối bỏ qua toàn bộ JOIN
- Wildcard query (`fields=*`) được chuyển thành danh sách column tường minh, không gồm column bị từ chối

Rule có điều kiện (`condition`) không thể quyết định trước SQL vì phụ thuộc dữ liệu bản ghi. Chúng được xử lý bởi lớp bảo vệ sau SQL.

### Primary key

Column primary key (`id` / `_id`) không bao giờ bị loại, bất kể permission rule.

### Repository trong script

- **`$ctx.$repos.main`** và **`$ctx.$repos.secure.<table>`** áp dụng permission rule theo trường (loại/từ chối theo cấu hình).
- **`$ctx.$repos.<table>`** (trừ `main` / `secure`) mặc định **không** áp dụng quyền theo trường; dùng **`secure`** khi quyền truy cập cần khớp metadata rule.

## GraphQL

Column và relation chưa publish (`isPublished: false`) bị loại khỏi GraphQL schema được tạo. Chúng không xuất hiện trong type definition, input type hay update input type.

## Ví dụ

### Đặt column ở chế độ riêng tư (chỉ quản lý được đọc)

1. Đặt `isPublished: false` cho column trong trình sửa schema của bảng.
2. Tạo rule allow:

```
POST /api/enfyra_field_permission
{
  "column": { "id": "<salary_column_id>" },
  "role": { "id": "<manager_role_id>" },
  "action": "read",
  "effect": "allow"
}
```

### Chỉ chủ sở hữu được truy cập qua điều kiện

Chỉ cho người dùng xem lương của chính họ:

```
POST /api/enfyra_field_permission
{
  "column": { "id": "<salary_column_id>" },
  "action": "read",
  "effect": "allow",
  "condition": { "userId": { "_eq": "@USER.id" } }
}
```

### Chặn một role cập nhật trường

```
POST /api/enfyra_field_permission
{
  "column": { "id": "<status_column_id>" },
  "role": { "id": "<viewer_role_id>" },
  "action": "update",
  "effect": "deny"
}
```
