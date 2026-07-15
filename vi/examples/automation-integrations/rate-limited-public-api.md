---
slug: vi-du/tu-dong-hoa-va-tich-hop/api-cong-khai-co-rate-limit
---

# API công khai có rate limit

Ví dụ này bảo vệ endpoint báo giá công khai mà không yêu cầu đăng nhập.

## Route

Tạo custom route `GET /public/quote` và cho phép công khai `GET`.

## Pre-hook

```javascript
const result = await @HELPERS.$rateLimit.byIp({
  maxRequests: 60,
  perSeconds: 60
});

if (!result.allowed) {
  @THROW429(`Rate limit exceeded. Try again in ${result.retryAfter}s`);
}
```

## Handler

```javascript
const quotes = await #quote.find({
  filter: { status: { _eq: 'published' } },
  fields: 'id,text,author',
  sort: '-createdAt',
  limit: 10
});

return {
  data: quotes.data?.[Math.floor(Math.random() * quotes.data.length)] || null
};
```

Giữ pre-hook làm ranh giới bảo vệ. Với rate limit tiêu chuẩn cho toàn route, dùng Guard khai báo trong admin console.
