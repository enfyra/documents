---
slug: tham-so-truy-van
---

# Tham số truy vấn

Những tham số này áp dụng cho request **GET** tới list endpoint (ví dụ `GET {appUrl}/api/enfyra_user`).

## fields

Chọn field trả về, cách nhau bằng dấu phẩy.

```
GET {appUrl}/api/enfyra_user?fields=id,email,name,role.name
```

- Dùng dot notation cho field liên quan: `role.name`
- Dùng `*` cho relation để lấy toàn bộ field: `role.*`
- Bỏ qua để trả về tất cả field (mặc định)
- Thêm `-` trước field để chuyển scope `fields` sang exclude mode: `fields=-compiledCode` trả về các field có thể đọc trừ `compiledCode`
- Khi có field bị loại trừ, field dương cùng scope sẽ bị bỏ qua: `fields=id,-compiledCode` vẫn trả mọi field có thể đọc trừ `compiledCode`
- Exclude lồng nhau theo quy tắc tương tự: `fields=-role.avatar` loại `avatar` khỏi object `role`

## filter

Filter object kiểu MongoDB, truyền dưới dạng JSON trên query string.

```
GET {appUrl}/api/enfyra_user?filter={"email":{"_contains":"@example.com"}}
```

| Operator | Mô tả | Ví dụ |
|----------|-------|-------|
| _eq | Bằng | `{"status":{"_eq":"active"}}` |
| _neq | Khác | `{"status":{"_neq":"deleted"}}` |
| _gt / _gte | Lớn hơn / lớn hơn hoặc bằng | `{"price":{"_gte":100}}` |
| _lt / _lte | Nhỏ hơn / nhỏ hơn hoặc bằng | `{"price":{"_lte":500}}` |
| _in / _not_in | Có trong / không có trong mảng | `{"category":{"_in":["a","b"]}}` |
| _contains | Chứa text, không phân biệt hoa thường | `{"name":{"_contains":"john"}}` |
| _starts_with / _ends_with | Bắt đầu / kết thúc bằng | `{"email":{"_starts_with":"admin"}}` |
| _is_null / _is_not_null | Là / không là null | `{"deletedAt":{"_is_null":true}}` |
| _between | Nằm trong khoảng, gồm cả biên | `{"price":{"_between":[100,500]}}` |
| _and / _or / _not | Tất cả / bất kỳ / phủ định điều kiện | `{"_not":{"status":{"_eq":"archived"}}}` |

Filter theo relation:

```
GET {appUrl}/api/order?filter={"customer":{"name":{"_contains":"Smith"}}}
```

Xem [Query Filtering](../server/query-filtering.md) để có tham chiếu đầy đủ.

## sort

Thứ tự sort bản ghi root, các field cách nhau bằng dấu phẩy. Thêm `-` để giảm dần.

```
?sort=name            name tăng dần
?sort=-createdAt      createdAt giảm dần
?sort=category,-price  category tăng dần, sau đó price giảm dần
?sort=-_max(messages.createdAt)  bản ghi cha theo tin nhắn con mới nhất
?sort=-_count(messages)          bản ghi cha theo số lượng bản ghi con
```

- Không chỉ định `sort`: kết quả sort `id` tăng dần.
- Sort chỉ áp dụng bảng cha/root, không áp dụng nested relation.
- Mảng nested (`one-to-many`, `many-to-many`) luôn được sort nội bộ theo `id`.
- `createdAt`, `updatedAt` và scalar `date`, `datetime`, `timestamp` có index một field tự tạo. SQL thêm `id` làm tie-breaker ổn định; Mongo thêm `_id`.
- Thêm explicit index cho hot path phức hợp như `status + createdAt`, nhưng không lặp lại index thời gian một field.
- Dùng `_count(relationName)`, `_max(relationName.fieldName)`, `_min(relationName.fieldName)` để sort bản ghi cha theo aggregate của direct `one-to-many` hoặc `many-to-many`.
- Không dùng sort to-many dạng chấm thô như `messages.createdAt`; nó mơ hồ và bị từ chối.
- Sort nested relation qua `deep`:

```
GET /api/enfyra_user?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt"}}
```

`deep` ở trên sort `posts` bên trong mỗi user, không sort danh sách user.

## limit

Số bản ghi tối đa trả về; mặc định là 10.

```
?limit=20
?limit=0    Không giới hạn (trả về mọi bản ghi khớp)
```

## page

Số trang phân trang, tính từ 1; dùng cùng `limit`.

```
?page=2&limit=20   Bản ghi 21–40
```

## meta

Yêu cầu metadata, cách nhau bằng dấu phẩy.

- `totalCount` – Tổng số bản ghi trong bảng, không áp dụng filter
- `filterCount` – Số bản ghi khớp filter

```
GET {appUrl}/api/enfyra_user?limit=10&meta=totalCount,filterCount
```

```json
{
  "data": [ ... ],
  "meta": {
    "totalCount": 150,
    "filterCount": 45
  }
}
```

## deep

Lấy relation lồng nhau, có filter, sort và limit ở từng cấp; truyền JSON.

```
GET {appUrl}/api/enfyra_user?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt","limit":5}}
```

Mỗi `deep.<relation>.fields` cũng hỗ trợ exclude mode:

```
GET {appUrl}/api/enfyra_user?fields=id,name&deep={"posts":{"fields":"-compiledCode,-author.avatar","limit":5}}
```

Xem [Deep Queries](../server/query-filtering.md#deep-queries-nested-relations) để biết chi tiết.

## Ví dụ kết hợp

```
GET {appUrl}/api/product?filter={"category":{"_eq":"electronics"},"price":{"_gte":100}}&fields=id,name,price,category.name&sort=-price&limit=20&page=1&meta=filterCount
```
