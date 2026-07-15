---
slug: vi-du/xac-thuc-va-phan-quyen/form-lien-he-cong-khai
---

# Form liên hệ công khai

Ví dụ này nhận nội dung gửi từ website mà không công khai hộp thư tiếp nhận.

## Table

Tạo `contact_request`.

| Field | Kiểu | Ghi chú |
| --- | --- | --- |
| `email` | string | Thêm format rule cho email |
| `name` | string | Không bắt buộc |
| `message` | text | Bắt buộc |
| `status` | select | `new`, `reviewing`, `closed` |
| `source` | string | Không bắt buộc |

Cho phép công khai `POST /contact_request`. Giữ `GET`, `PATCH` và `DELETE` chỉ dành cho operator.

## Kiểm tra và chuẩn hóa dữ liệu

Thêm pre-hook cho `POST /contact_request`.

```javascript
const email = typeof @BODY.email === 'string' ? @BODY.email.trim().toLowerCase() : '';
const message = typeof @BODY.message === 'string' ? @BODY.message.trim() : '';

if (!email || !message) {
  @THROW400('email and message are required');
}

@BODY.email = email;
@BODY.message = message;
@BODY.status = 'new';
@BODY.source = @BODY.source || 'website';
```

Để kiểm tra email theo chuẩn, thêm column rule cho `email` với format `email`.

## Giới hạn số lượt gửi

Thêm đoạn này vào cùng pre-hook hoặc một pre-hook riêng.

```javascript
const result = await @HELPERS.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 300
});

if (!result.allowed) {
  @THROW429(`Too many submissions. Try again in ${result.retryAfter}s`);
}
```

## Gửi từ website

```bash
curl "$ENFYRA_API_URL/contact_request" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"ada@example.com","name":"Ada","message":"Can we talk about Enfyra Cloud?","source":"landing"}'
```

## Hộp thư của operator

Dùng data table trong admin hoặc một admin extension nhỏ để đọc:

```text
GET /contact_request?filter={"status":{"_eq":"new"}}&sort=-createdAt&fields=id,email,name,message,status,createdAt
```
