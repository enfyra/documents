---
slug: endpoint-xac-thuc
---

# Endpoint xác thực

Mọi URL dùng base `{appUrl}/api`.

## POST /login

Xác thực bằng email và mật khẩu qua Enfyra app proxy. Ở chế độ SSR/cookie, app lưu Enfyra token trong httpOnly cookie cho bạn.

**URL:** `{appUrl}/api/login`

```json
{
  "email": "admin@example.com",
  "password": "your_password",
  "remember": false
}
```

| Field | Kiểu | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| email | string | Có | Email người dùng |
| password | string | Có | Mật khẩu |
| remember | boolean | Không | `true` thì refresh token có thời hạn dài hơn, mặc định `false` |

Response `200`:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expTime": 1735689600000,
  "loginProvider": null
}
```

`POST /login` cũng đặt httpOnly cookie trên app origin. Với client tự quản lý token, gọi `{appUrl}/api/auth/login`; response JSON giống nhau nhưng không có tiện ích cookie session của app.

```bash
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"1234"}'
```

Chỉ dùng `{appUrl}/api/auth/login` khi bạn chủ ý nhận raw backend token và tự quản lý chúng. Với Nuxt, Next hay SSR app khác, ưu tiên `{appUrl}/api/login`.

## POST /auth/logout

Hủy session hiện tại (refresh token).

**URL:** `{appUrl}/api/auth/logout`

Với manual-token client, gửi access token qua `Authorization: Bearer …` và refresh token trong JSON body. Browser app nên dùng `{appUrl}/api/logout`; endpoint này đọc rồi xóa httpOnly cookie sau khi server hủy session.

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

| Field | Kiểu | Bắt buộc | Mô tả |
|-------|------|----------|-------|
| refreshToken | string | Có | Refresh token cần hủy |

Response `200`: thông báo thành công.

## POST /auth/refresh-token

Lấy access token mới từ refresh token.

**URL:** `{appUrl}/api/auth/refresh-token`

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expTime": 1735689600000,
  "loginProvider": null
}
```

Response `200` trả `accessToken`, `refreshToken`, `expTime`, `loginProvider`. Hãy thay cả hai token đã lưu bằng giá trị response; refresh-token rotation làm token bạn gửi hết hiệu lực.

## GET /auth/:provider

Khởi động đăng nhập OAuth và redirect người dùng đến trang ủy quyền của provider.

**URL:** `{appUrl}/api/auth/{provider}?redirect={redirectUrl}`

| Provider | Path |
|----------|------|
| Google | `{appUrl}/api/auth/google?redirect=...` |
| Facebook | `{appUrl}/api/auth/facebook?redirect=...` |
| GitHub | `{appUrl}/api/auth/github?redirect=...` |

| Query parameter | Bắt buộc | Mô tả |
|-----------------|----------|-------|
| redirect | Có | URL `http(s)` tuyệt đối để redirect sau OAuth thành công; phải URL-encode |
| cookieBridgePrefix | Không | Tiền tố app proxy dùng để đặt cookie trên caller origin. Mặc định `/api`; app thứ ba có thể dùng `/enfyra`. |

```
http://localhost:3000/api/auth/google?redirect=http%3A%2F%2Flocalhost%3A3000%2Fdashboard
```

Ví dụ app thứ ba:

```text
https://demo.enfyra.io/api/auth/google?redirect=https%3A%2F%2Fchat.example.com%2Fchat&cookieBridgePrefix=%2Fenfyra
```

Response: HTTP 302 redirect đến OAuth provider.

### Hợp đồng OAuth redirect

- `redirect` là bắt buộc và phải là URL `http(s)` tuyệt đối.
- Client truyền trang muốn quay lại sau đăng nhập.
- Với Nuxt, Next và SSR app khác, bắt đầu OAuth ở URL Enfyra app như `{enfyraAppUrl}/api/auth/:provider`, truyền `redirect` và `cookieBridgePrefix`.
- Nếu `enfyra_oauth_config.autoSetCookies = true`, backend redirect qua `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies`, đặt auth cookie trên origin đó qua proxy response, rồi redirect đến `redirect`. Bỏ `cookieBridgePrefix` thì mặc định là `/api`.
- Nếu `autoSetCookies = false`, backend redirect đến `enfyra_oauth_config.appCallbackUrl` với `code` ngắn hạn, dùng một lần cùng `redirect` trên query string. Token không bao giờ nằm trên URL.

## GET /auth/:provider/callback

OAuth provider redirect về đây sau khi người dùng chấp thuận. Backend đổi code thành token, rồi:

- redirect qua app proxy `{redirect.origin}{cookieBridgePrefix}/auth/set-cookies` khi `autoSetCookies = true`, hoặc
- redirect đến `appCallbackUrl` với exchange `code` dùng một lần khi `autoSetCookies = false`.

**URL:** `{appUrl}/api/auth/{provider}/callback?code=...&state=...`

Endpoint này thường do OAuth provider gọi, không gọi trực tiếp từ app.

### Ví dụ: Nuxt tự đặt cookie

```text
https://localhost:3000/api/auth/google?redirect=https%3A%2F%2Flocalhost%3A3000%2Fdashboard
```

Cho SSR flow này, bật `autoSetCookies` trong OAuth config. Tiền tố cookie bridge mặc định là `/api`, nên Enfyra redirect qua `/api/auth/set-cookies`; browser code không đọc OAuth token query parameter.

### Ví dụ: OAuth app thứ ba

App thứ ba proxy `/enfyra/**` tới base `/api/**` của Enfyra app. Nút login redirect browser đến:

```text
https://demo.enfyra.io/api/auth/google?redirect=https%3A%2F%2Fchat.example.com%2Fchat&cookieBridgePrefix=%2Fenfyra
```

Sau provider callback, Enfyra redirect tới:

```text
https://chat.example.com/enfyra/auth/set-cookies?code=...&redirect=https%3A%2F%2Fchat.example.com%2Fchat
```

URL này vẫn được proxy tới Enfyra. Enfyra trả `Set-Cookie` trên origin app thứ ba rồi redirect đến `redirect` gốc.

### Ví dụ: callback cho external client

Cấu hình:

- `redirectUri = https://api.example.com/auth/google/callback`
- `appCallbackUrl = https://client.example.com/oauth/callback`
- `autoSetCookies = false`

Sau đó gọi:

```text
https://admin.enfyra.com/api/auth/google?redirect=https%3A%2F%2Fclient.example.com%2Fafter-login
```

Callback nhận `?code=...&redirect=...`. Server của bạn phải đổi code ngay; không đưa token nhận được vào URL trình duyệt, log hay local storage.

## POST /auth/oauth/exchange

Đổi `code` ngắn hạn, dùng một lần từ OAuth callback thành token. Endpoint này dành cho trusted server của external app khi `autoSetCookies = false`.

**URL:** `{appUrl}/api/auth/oauth/exchange`

```bash
curl -X POST https://admin.enfyra.com/api/auth/oauth/exchange \
  -H "Content-Type: application/json" \
  -d '{"code":"CODE_FROM_THE_CALLBACK"}'
```

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expTime": 1735689600000,
  "loginProvider": "google"
}
```

Code hết hạn sau 10 phút và bị dùng bởi lần exchange thành công đầu tiên. Xem token trả về là credential.

## GET /me

Lấy profile của người dùng đã xác thực.

**URL:** `{appUrl}/api/me`  
**Header:** `Authorization: Bearer {accessToken}`

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "id": 1,
    "email": "admin@example.com",
    "name": "Admin",
    "isActive": true,
    "role": { "id": 1, "name": "Admin" },
    "createdAt": "2024-01-15T10:00:00.000Z"
  }
}
```

```bash
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

Response `200` trả object người dùng gồm `id`, `email`, `name`, `isActive`, `role`, `createdAt`.

## PATCH /me

Cập nhật profile người dùng hiện tại, ví dụ tên hoặc mật khẩu.

**URL:** `{appUrl}/api/me`  
**Header:** `Authorization: Bearer {accessToken}`

```json
{
  "name": "New Name",
  "password": "new_password"
}
```

| Field | Kiểu | Mô tả |
|-------|------|-------|
| name | string | Tên hiển thị |
| password | string | Mật khẩu mới, sẽ được hash |

Response `200`: object người dùng đã cập nhật.
