---
slug: vi-du/dang-ky-nguoi-dung
---

# Ví dụ đăng ký người dùng

Xây dựng endpoint công khai `POST /register` để tạo người dùng một cách an toàn.

Ví dụ này dùng một custom route, một pre-hook và một custom handler. Nội dung được giữ gọn để bạn có thể áp dụng mẫu này cho ứng dụng của mình.

## Nội dung sẽ xây dựng

```text
POST /api/register
  -> pre-hook validates input
  -> handler hashes password
  -> handler creates enfyra_user row
  -> response returns safe user fields only
```

Dùng cách này khi route `POST /enfyra_user` mặc định quá khái quát cho việc đăng ký công khai.

## 1. Tạo route

Trong admin app, tạo một route:

| Trường | Giá trị |
|-------|-------|
| Path | `/register` |
| Method | `POST` |
| Handler | Custom |
| Target table | `enfyra_user` |

Chỉ giữ route ở chế độ public nếu endpoint này được dành cho đăng ký. Nếu không, hãy gắn các route permission thông thường.

## 2. Thêm pre-hook

Gắn pre-hook này vào `POST /register`.

Nó kiểm tra body trước khi handler chạy.

```js
const { email, password } = @BODY

if (!email) @THROW400("Email is required")
if (!password) @THROW400("Password is required")
if (password.length < 8) @THROW400("Password must be at least 8 characters")

const existing = await #enfyra_user.find({
  filter: { email: { _eq: email } },
  fields: "id",
  limit: 1
})

if (existing.data[0]) @THROW409("Email already exists")
```

Vì sao phần này thuộc pre-hook:

- Nó từ chối request không hợp lệ trước khi handler xử lý.
- Nó giúp handler chỉ tập trung vào việc tạo dữ liệu.
- Mẫu này cũng phù hợp cho kiểm tra tenant, quota hoặc xác thực lời mời.

## 3. Thêm custom handler

Gắn handler này vào `POST /register`.

```js
const { email, password, name } = @BODY

const hashedPassword = await @HELPERS.$bcrypt.hash(password)

const result = await #enfyra_user.create({
  data: {
    email,
    password: hashedPassword,
    name: name || null,
    isActive: true
  }
})

const user = result.data[0]

return {
  id: user.id,
  email: user.email,
  name: user.name,
  isActive: user.isActive
}
```

Handler không trả về `password`.

## 4. Kiểm thử endpoint

```bash
curl -X POST "http://localhost:3000/api/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "mai@example.com",
    "password": "password123",
    "name": "Mai Tran"
  }'
```

Response mong đợi:

```json
{
  "id": 12,
  "email": "mai@example.com",
  "name": "Mai Tran",
  "isActive": true
}
```

## 5. Tùy chọn: kích hoạt flow chào mừng

Nếu cần gửi email hoặc thực hiện onboarding, hãy kích hoạt flow trong post-hook.

```js
if (!@ERROR && @DATA?.id) {
  await @TRIGGER("welcome-user", {
    userId: @DATA.id,
    email: @DATA.email
  })
}
```

Dùng flow khi công việc có thể chạy sau response hoặc cần retry/history.

## Lỗi thường gặp

### Tạo người dùng từ trình duyệt qua `/enfyra_user`

Dùng route `/register` chuyên biệt cho đăng ký công khai. Route này cho bạn một nơi để kiểm tra, hash và chỉ trả về các trường an toàn.

### Trả về toàn bộ dòng người dùng

Không bao giờ trả về `password`, reset token, OAuth secret hoặc trường nội bộ từ endpoint đăng ký công khai.

### Đặt toàn bộ logic vào một handler

Dùng pre-hook để kiểm tra, handler để tạo dữ liệu và post-hook/flow cho side effect.

## Tài liệu liên quan

- [Routing Management](../app/routing-management.md)
- [Custom Handlers](../app/hooks-handlers/custom-handlers.md)
- [Hooks](../app/hooks-handlers/hooks.md)
- [Flows](../server/flows.md)
