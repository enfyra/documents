---
slug: vi-du/xac-thuc-va-phan-quyen/pham-vi-chu-so-huu-ho-so
---

# Phạm vi chủ sở hữu hồ sơ

Ví dụ này cho phép người dùng đã đăng nhập quản lý bản ghi hồ sơ của chính họ.

## Table

Tạo `user_profile`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `user` | relation one-to-one với `enfyra_user` | Bắt buộc |
| `displayName` | string | Tên hiển thị công khai |
| `bio` | text | Không bắt buộc |
| `avatar` | relation many-to-one với `enfyra_file` | Không bắt buộc |

Thêm unique constraint cho `user`.

## Đọc hồ sơ hiện tại

Thêm pre-hook cho `GET /user_profile`.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

if (!@USER?.id) {
  @THROW401();
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { user: { id: { _eq: @USER.id } } }
  ]
};
```

```bash
curl "$ENFYRA_API_URL/user_profile?limit=1&fields=id,displayName,bio,avatar.id,avatar.url" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

## Tạo hoặc cập nhật hồ sơ hiện tại

Tạo custom handler `POST /profile/me`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const existing = await #user_profile.find({
  filter: { user: { id: { _eq: @USER.id } } },
  fields: 'id',
  limit: 1
});

const data = {
  displayName: @BODY.displayName,
  bio: @BODY.bio,
  user: { id: @USER.id }
};

if (@BODY.avatar) {
  data.avatar = @BODY.avatar;
}

if (existing.data?.[0]) {
  const updated = await #user_profile.update({
    id: existing.data[0].id,
    data
  });
  return updated.data?.[0] || null;
}

const created = await #user_profile.create({ data });
return created.data?.[0] || null;
```

## Bảo vệ thao tác ghi trực tiếp

Với `POST /user_profile` và `PATCH /user_profile/:id` trực tiếp, thêm hook để ép relation về người dùng hiện tại và giới hạn cập nhật theo chủ sở hữu. Root admin vẫn có thể quản lý bản ghi từ admin console.
