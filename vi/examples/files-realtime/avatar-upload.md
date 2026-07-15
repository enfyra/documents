---
slug: vi-du/file-va-realtime/tai-avatar
---

# Tải avatar

Ví dụ này tải avatar profile lên và liên kết với `user_profile`.

## Route tải lên

Tạo custom route `POST /profile/avatar` nhận upload multipart. Handler lưu file rồi cập nhật profile của người dùng hiện tại.

```javascript
if (!@USER?.id) {
  @THROW401();
}

if (!@UPLOADED_FILE) {
  @THROW400('file is required');
}

const uploaded = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  folder: 'avatars',
  isPublic: true
});

const profileResult = await #user_profile.find({
  filter: { user: { id: { _eq: @USER.id } } },
  fields: 'id',
  limit: 1
});

const data = {
  user: { id: @USER.id },
  avatar: { id: uploaded.id }
};

if (profileResult.data?.[0]) {
  const updated = await #user_profile.update({
    id: profileResult.data[0].id,
    data
  });
  return updated.data?.[0] || null;
}

const created = await #user_profile.create({ data });
return created.data?.[0] || null;
```

## Request từ trình duyệt

```javascript
const form = new FormData();
form.append('file', fileInput.files[0]);

const uploadId = crypto.randomUUID();

await fetch('/enfyra/profile/avatar', {
  method: 'POST',
  headers: { 'x-enfyra-upload-id': uploadId },
  body: form,
  credentials: 'include'
});
```

Lắng nghe `$system:upload:progress` trên admin socket đã xác thực và ghép event theo `uploadId`.

## Đọc profile công khai

Chỉ mở các field an toàn.

```text
GET /user_profile?fields=id,displayName,avatar.url
```
