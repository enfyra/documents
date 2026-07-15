---
slug: vi-du/tu-dong-hoa-va-tich-hop/nhan-webhook
---

# Nhận webhook

Ví dụ này nhận event đã được nhà cung cấp ký và chỉ lưu mỗi event một lần.

## Table

Tạo `integration_settings` dưới dạng table một bản ghi.

| Field | Kiểu | Ghi chú |
| --- | --- | --- |
| `providerWebhookSecret` | string | `isEncrypted=true`, `isPublished=false` |

Tạo `provider_event`.

| Field | Kiểu | Ghi chú |
| --- | --- | --- |
| `providerEventId` | string | Duy nhất |
| `kind` | string | Kiểu event |
| `payload` | simple-json | Payload event thô |
| `processedAt` | datetime | Có thể để trống |

## Route

Tạo custom route `POST /webhooks/provider`. Chỉ cho phép công khai nếu handler bắt buộc kiểm tra chữ ký.

```javascript
const settingsResult = await #integration_settings.find({
  fields: 'providerWebhookSecret',
  limit: 1
});

const settings = settingsResult.data?.[0];
if (!settings?.providerWebhookSecret) {
  @THROW500('webhook secret is not configured');
}

const signature = @REQ.headers['x-provider-signature'];
const expected = await @HELPERS.$crypto.hmacSha256(
  @REQ.rawBody,
  settings.providerWebhookSecret,
  'hex'
);

if (signature !== expected) {
  @THROW401('invalid signature');
}

const eventId = @BODY.id;
if (!eventId) {
  @THROW400('event id is required');
}

const existing = await #provider_event.find({
  filter: { providerEventId: { _eq: eventId } },
  fields: 'id',
  limit: 1
});

if (existing.data?.[0]) {
  return { received: true, duplicate: true };
}

await #provider_event.create({
  data: {
    providerEventId: eventId,
    kind: @BODY.type || 'unknown',
    payload: @BODY
  }
});

await @TRIGGER('process-provider-event', { eventId });

return { received: true };
```

Khi nhà cung cấp ký theo byte của request thô, hãy dùng chính raw body đó để kiểm tra chữ ký.
