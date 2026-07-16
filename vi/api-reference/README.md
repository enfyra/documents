---
slug: tham-chieu-api
---

# API

Hướng dẫn **xây dựng ứng dụng** với Enfyra API. Dùng các endpoint này để xác thực người dùng, đọc và thay đổi dữ liệu, cũng như làm việc với file từ frontend riêng, ứng dụng di động hoặc dịch vụ bên ngoài.

Mọi yêu cầu API đều dùng **URL của ứng dụng** với prefix `/api`.

## Base URL

```
{appUrl}/api/{endpoint}
```

**Ví dụ:**

- Môi trường development: `http://localhost:3000/api/me`
- Production: `https://your-app.enfyra.com/api/products`

Enfyra App chuyển tiếp yêu cầu đến backend, vì vậy ứng dụng của bạn chỉ cần gọi `{appUrl}/api/...` mà không phải giao tiếp trực tiếp với backend.

## Điều hướng nhanh

| Chủ đề | Tài liệu |
| --- | --- |
| **Tổng quan** | Base URL, header, xác thực, định dạng response | [overview.md](./overview.md) |
| **Xác thực** | Đăng nhập, đăng xuất, refresh token, OAuth, `/me` | [authentication.md](./authentication.md) |
| **CRUD** | Liệt kê, tạo, cập nhật, xóa bản ghi | [crud-operations.md](./crud-operations.md) |
| **Tham số query** | `filter`, `fields`, `sort`, `limit`, `page` | [query-parameters.md](./query-parameters.md) |
| **Tệp và lưu trữ** | Tải file lên, liệt kê folder, phân phối asset | [file-storage.md](./file-storage.md) |

## Dùng API

### Ví dụ cURL

```bash
# Login
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"your_password"}'

# Get current user
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"

# List your data
curl "http://localhost:3000/api/products?limit=10" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### JavaScript / fetch (dùng cookie, nên dùng trên trình duyệt)

```javascript
const appUrl = 'http://localhost:3000';

// Login
const response = await fetch(`${appUrl}/api/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' }),
}).then(r => r.json());

// Fetch your data. The browser sends httpOnly auth cookies automatically.
const products = await fetch(`${appUrl}/api/products?limit=20`).then(r => r.json());
```

### Client dùng token

Chỉ dùng token client khi bạn chủ động quản lý credential ngoài phiên cookie của trình duyệt. Đăng nhập qua `/api/auth/login`, lưu token trong credential store an toàn của bạn, sau đó gửi access token tường minh.

```javascript
const appUrl = 'http://localhost:3000';

const login = await fetch(`${appUrl}/api/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' })
}).then(r => r.json());

const products = await fetch(`${appUrl}/api/products?limit=20`, {
  headers: { Authorization: `Bearer ${login.accessToken}` },
}).then(r => r.json());
```

Với Nuxt, Next hoặc app SSR khác, hãy proxy mọi lệnh gọi Enfyra qua app origin của bạn. Enfyra App thường dùng `/api`; app bên thứ ba có thể dùng prefix như `/enfyra` rồi forward về base `/api` của Enfyra App. Dùng `{prefix}/login` cho đăng nhập mật khẩu. Với OAuth, bắt đầu tại `{prefix}/auth/{provider}?redirect=<absoluteReturnUrl>&cookieBridgePrefix=<prefix>` và bật chế độ Enfyra OAuth set-cookie. Enfyra chuyển hướng qua `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies`, trả `Set-Cookie` cho app origin đó rồi chuyển đến `redirect`. Xem [SSR Frameworks](../integrations/ssr-frameworks.md) để cấu hình theo framework.

**Lợi ích của phiên dùng cookie:**

- **An toàn hơn**: JavaScript không truy cập được HTTP-only cookie.
- **Bảo vệ CSRF**: Có sẵn bảo vệ bằng thuộc tính SameSite.
- **Tự refresh**: Server tự xử lý việc refresh token.
- **Đơn giản**: Không cần tự xử lý cookie; trình duyệt làm việc đó.

## Những gì bạn có

- **Auth** — Đăng nhập, đăng xuất, refresh token, OAuth (Google, Facebook, GitHub).
- **Table của bạn** — Mỗi table tạo ra có các endpoint CRUD (ví dụ `/products`, `/orders`).
- **Query** — Lọc, sắp xếp và phân trang bằng operator theo kiểu MongoDB.
- **File** — Tải lên, sắp xếp trong folder và phân phối asset.

Để quản trị và cấu hình Enfyra (route, hook, handler, v.v.), xem [Tài liệu Server](../server/README.md).
