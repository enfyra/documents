---
slug: vi-du/tu-dong-hoa-va-tich-hop/dong-bo-ra-ngoai
---

# Đồng bộ ra ngoài

Ví dụ này gửi các bản ghi đã được duyệt tới một API khác sau khi chúng được lưu.

## Table cài đặt

Tạo `sync_settings` dưới dạng table một bản ghi.

| Field | Kiểu | Ghi chú |
| --- | --- | --- |
| `endpointUrl` | string | URL đích |
| `apiKey` | string | `isEncrypted=true`, `isPublished=false` |

## Post-hook

Gắn post-hook này vào `PATCH /blog_comment`.

```javascript
if (@ERROR || !@DATA || @DATA.status !== 'approved') {
  return;
}

const settingsResult = await #sync_settings.find({
  fields: 'endpointUrl,apiKey',
  limit: 1
});

const settings = settingsResult.data?.[0];
if (!settings?.endpointUrl || !settings?.apiKey) {
  return;
}

const response = await fetch(settings.endpointUrl, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${settings.apiKey}`
  },
  body: JSON.stringify({
    id: @DATA.id,
    body: @DATA.body,
    approvedAt: new Date().toISOString()
  })
});

if (!response.ok) {
  @THROW502('external sync failed');
}
```

Với hệ thống đích chậm hoặc thiếu ổn định, hãy trigger flow thay vì thực hiện HTTP request trực tiếp trong post-hook.
