---
slug: tong-quan-api
---

# Tổng quan API

Những điểm cốt lõi để gọi API Enfyra từ ứng dụng của bạn.

## Định dạng URL gốc

Mọi endpoint REST của Enfyra dùng dạng:

```
{appUrl}/api/{path}
```

| Môi trường | appUrl | Ví dụ |
|-------------|--------|-------|
| Phát triển cục bộ | `http://localhost:3000` | `http://localhost:3000/api/me` |
| Production | URL ứng dụng đã triển khai | `https://app.yourdomain.com/api/me` |

**Quan trọng:** luôn gọi qua proxy cùng origin của app, không gọi thẳng host backend. Enfyra app thường cung cấp `/api/**`; app bên thứ ba có thể proxy qua tiền tố riêng, ví dụ `/enfyra/**`, đến `/api/**` của Enfyra. Cách này tránh CORS và giữ cookie trên origin của app.

## Header của request

| Header | Bắt buộc | Mô tả |
|--------|----------|-------|
| `Content-Type` | Với POST/PATCH | `application/json` cho JSON body |
| `Authorization` | Phần lớn endpoint | `Bearer {accessToken}` cho request đã xác thực |
| `Cookie` | Phương án xác thực khác | Session cookie khi dùng cookie-based auth |

## Xác thực

Phần lớn endpoint cần JWT access token hợp lệ. Lấy token bằng:

1. **POST** `{appUrl}/api/login` – đăng nhập SSR/cookie qua app proxy
2. **GET** `{appUrl}/api/auth/{provider}?redirect=...` – đăng nhập OAuth (Google, Facebook, GitHub)

Gửi token cùng request:

```
Authorization: Bearer eyJhbGc...
```

Với Nuxt, Next hoặc một SSR app khác, nên dùng session cookie. Proxy Enfyra qua tiền tố cùng origin, gọi `{prefix}/login`, lấy người dùng tại `{prefix}/me`, rồi để trình duyệt tự gửi cookie khi request cùng origin.

Với app bên thứ ba, khởi động OAuth trên URL Enfyra app và truyền thêm query parameter:

```text
GET https://demo.enfyra.io/api/auth/google?redirect=https%3A%2F%2Fchat.example.com%2Fchat&cookieBridgePrefix=/enfyra
```

`redirect` phải là URL `http(s)` tuyệt đối. `cookieBridgePrefix` là tiền tố proxy của app thứ ba chuyển tiếp đến API Enfyra. Enfyra dùng nó để redirect qua `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies`, giúp cookie được ghi trên origin của app thứ ba trước khi quay lại `redirect`.

Ví dụ proxy cho app Nuxt:

```ts
export default defineNuxtConfig({
  routeRules: {
    '/enfyra/**': {
      proxy: {
        to: 'https://demo.enfyra.io/api/**',
        fetchOptions: { redirect: 'manual' },
      },
    },
    '/socket.io/**': {
      proxy: { to: 'https://demo.enfyra.io/ws/socket.io/**' },
    },
  },
})
```

Xem hướng dẫn Nuxt, Next.js, SvelteKit và Remix đầy đủ tại [SSR Frameworks](../integrations/ssr-frameworks.md).

**Các endpoint đầu vào chưa xác thực:**

- `POST /api/login`
- `POST /api/auth/login` cho client tự quản lý token
- `GET /api/auth/:provider` (OAuth redirect)
- `GET /api/auth/:provider/callback` (OAuth callback)

`POST /api/auth/refresh-token` cần refresh token hợp lệ trong body. `POST /api/auth/logout` cần cả refresh token lẫn ngữ cảnh người dùng đã xác thực. Đây là endpoint duy trì auth, không phải API ứng dụng ẩn danh.

## Định dạng response

### Thành công (2xx)

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": [ ... ],
  "meta": {
    "totalCount": 100,
    "filterCount": 25
  }
}
```

- `data`: Mảng bản ghi (list endpoint) hoặc object đơn (create/update/me)
- `meta`: Không bắt buộc; xuất hiện khi yêu cầu `meta=totalCount` hoặc `meta=filterCount`

### Lỗi (4xx, 5xx)

```json
{
  "statusCode": 400,
  "message": "Bad Request",
  "details": "Email is required"
}
```

## HTTP method

| Method | Mục đích thường dùng |
|--------|----------------------|
| `GET` | Liệt kê bản ghi, lấy một bản ghi bằng ID, /me |
| `POST` | Tạo bản ghi |
| `PATCH` | Cập nhật bản ghi theo ID |
| `DELETE` | Xóa bản ghi theo ID |

## Tiếp theo

- [Endpoint xác thực](./authentication.md) – Login, logout, refresh, OAuth, /me
- [Thao tác CRUD](./crud-operations.md) – Route bảng và mẫu request/response
