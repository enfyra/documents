---
slug: sdk/query-du-lieu-crud
---

# Dữ liệu và CRUD

Dùng Query Builder cho table được Enfyra expose qua route. Nó hỗ trợ fields, filter, sort, pagination, relation query, metadata và mutation theo ID.

## Khai báo record type

```ts
interface Article {
  id: number
  title: string
  status: 'draft' | 'published'
  author?: {
    id: number
    name: string
  }
  createdAt: string
}
```

## Đọc collection

```ts
const { data, meta } = await enfyra
  .from<Article>('articles')
  .select(['id', 'title', 'status', 'createdAt'])
  .filter({ status: { _eq: 'published' } })
  .sort('-createdAt')
  .limit(20)
  .page(1)
  .meta(['totalCount', 'filterCount'])
  .execute()
```

`sort('-createdAt')` sắp xếp giảm dần. Dùng `sort('createdAt')` để tăng dần.

## Lọc record

```ts
const result = await enfyra
  .from<Article>('articles')
  .filter({
    _and: [
      { status: { _eq: 'published' } },
      { createdAt: { _gte: '2026-01-01T00:00:00.000Z' } },
      {
        _or: [
          { title: { _icontains: 'enfyra' } },
          { category: { _in: ['guide', 'release'] } },
        ],
      },
    ],
  })
  .execute()
```

SDK hỗ trợ `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_in`, `_not_in`, `_contains`, `_icontains`, `_starts_with`, `_ends_with`, `_is_null`, `_is_not_null`, `_between`, `_and`, `_or` và `_not`.

Chỉ dùng field và operator mà target route cùng schema hỗ trợ.

## Đọc relation

Chọn relation field bằng dotted field và truyền deep query khi collection liên quan cần filter hoặc limit riêng:

```ts
const result = await enfyra
  .from<Article>('articles')
  .select(['id', 'title', 'author.id', 'author.name', 'comments.id', 'comments.body'])
  .deep({
    comments: {
      _limit: 5,
      _sort: '-createdAt',
      _filter: { isApproved: { _eq: true } },
    },
  })
  .execute()
```

Tên relation property đến từ metadata Enfyra. Không đoán physical foreign-key column.

## Đọc một record

```ts
const { data } = await enfyra
  .from<Article>('articles')
  .byId(articleId)
  .select(['id', 'title', 'status'])
  .execute()
```

`byId()` thêm ID filter và đặt request limit bằng một. `data` vẫn theo API response shape; hãy narrow có chủ đích nếu ứng dụng bắt buộc cần một object.

## Transform query data

Dùng `.transform()` cho transform cục bộ của một Query Builder:

```ts
const { data } = await enfyra
  .from<Article>('articles')
  .select(['id', 'title'])
  .transform((value) => {
    const rows = Array.isArray(value) ? value : [value]
    return rows.map((article) => ({
      ...article,
      title: article.title.trim(),
    }))
  })
  .execute()
```

Function nhận query `data`, không phải toàn bộ HTTP response.

## Tạo record

```ts
const created = await enfyra.from<Article>('articles').insert({
  title: 'SDK guide',
  status: 'draft',
})
```

## Cập nhật record

```ts
const updated = await enfyra
  .from<Article>('articles')
  .byId(articleId)
  .update({
    title: 'Updated SDK guide',
    status: 'published',
  })
```

Gọi `update()` mà không có `byId()` trả `VALIDATION_ERROR` trước khi request được gửi.

## Xóa record

```ts
await enfyra.from('articles').byId(articleId).delete()
```

Gọi `delete()` mà không có `byId()` bị chặn trước khi request được gửi.

## Cập nhật relation

Gửi relation value bằng đúng relation property name mà metadata Enfyra expose:

```ts
await enfyra.from<Article>('articles').byId(articleId).update({
  author: { id: authorId },
  tags: [{ id: firstTagId }, { id: secondTagId }],
})
```

Shape được chấp nhận phụ thuộc cardinality và server metadata. Kiểm tra table schema và generated API behavior.

## Checklist pagination

- Request `meta(['totalCount'])` khi UI cần tổng kích thước collection.
- Request `meta(['filterCount'])` khi UI cần số record sau filter.
- Giữ `limit` có giới hạn cho list hướng tới user.
- Reset `page` khi filter thay đổi.
- Chỉ select field màn hình hiện tại cần.

## Xử lý sự cố

- 403: route, role, user, owner hoặc field permission từ chối operation.
- Thiếu field: field có thể chưa được select hoặc bị field permission ẩn.
- Relation không tồn tại: dùng metadata relation property name, không đoán database column.
- Update/delete validation error: gọi `.byId(id)` trước.
- Nghiệp vụ riêng: gọi custom route phù hợp thay vì ép vào table CRUD.
