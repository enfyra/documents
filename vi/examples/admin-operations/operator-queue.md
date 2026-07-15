---
slug: hang-doi-van-hanh
---

# Hàng đợi vận hành

Ví dụ này xây dựng trang hàng đợi cho các bản ghi cần người xử lý.

## Bảng

Tạo `review_case`.

| Trường hoặc quan hệ | Kiểu | Ghi chú |
| --- | --- | --- |
| `title` | string | Bắt buộc |
| `status` | select | `open`, `in_review`, `resolved` |
| `priority` | select | `low`, `normal`, `high` |
| `assignee` | quan hệ many-to-one tới `enfyra_user` | Không bắt buộc |
| `payload` | simple-json | Không bắt buộc |

## Mẫu truy vấn

Dùng filter phía server và phân trang có giới hạn. Không lấy một giới hạn lớn tùy ý rồi lọc trong trình duyệt.

```javascript
const page = ref(1);
const status = ref('open');
const search = ref('');

const query = computed(() => ({
  filter: JSON.stringify({
    _and: [
      { status: { _eq: status.value } },
      search.value
        ? { title: { _contains: search.value } }
        : {}
    ]
  }),
  fields: 'id,title,status,priority,assignee.email,createdAt',
  sort: 'priority,-createdAt',
  page: page.value,
  limit: 25,
  meta: 'filterCount'
}));
```

## Thao tác nhận xử lý

Tạo `POST /review-cases/claim`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const id = @BODY.id;
if (!id) {
  @THROW400('id is required');
}

const updated = await #review_case.update({
  id,
  data: {
    status: 'in_review',
    assignee: { id: @USER.id }
  }
});

return updated.data?.[0] || null;
```

Dùng `PermissionGate` quanh nút nhận xử lý và yêu cầu quyền `POST /review-cases/claim`.
