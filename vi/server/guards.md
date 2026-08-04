---
slug: bao-ve-tuyen
---

# Guard

Guard bảo vệ các route bằng những quy tắc khai báo sẵn, không cần viết code. Bạn có thể cấu hình trong giao diện quản trị hoặc qua API để chặn IP, giới hạn tần suất request và kiểm soát quyền truy cập.

## Guard hoạt động như thế nào

Guard tự động chạy tại hai thời điểm trong vòng đời của request:

```
1. Nhận diện route
2. Guard trước xác thực     <-- Chặn IP, giới hạn tần suất (trước khi đăng nhập)
3. Xác thực JWT
4. Kiểm tra vai trò/quyền hạn
5. Guard sau xác thực       <-- Giới hạn theo từng người dùng (sau khi đăng nhập)
6. Pre-Hook  Handler  Post-Hook
```

- Guard **pre-auth** chạy trước khi đăng nhập, phù hợp với quy tắc dựa trên IP và giới hạn tần suất toàn cục.
- Guard **post-auth** chạy sau khi đăng nhập, phù hợp với quy tắc theo từng người dùng.

## Các loại quy tắc

| Loại quy tắc | Mô tả | Cấu hình |
|---|---|---|
| `ip_whitelist` | Chỉ cho phép các IP có trong danh sách; chặn toàn bộ IP còn lại | `{"ips": ["1.2.3.4", "10.0.0.0/8"]}` |
| `ip_blacklist` | Chặn các IP có trong danh sách; cho phép toàn bộ IP còn lại | `{"ips": ["5.6.7.8"]}` |
| `rate_limit_by_ip` | Giới hạn tần suất cho từng IP client trên mỗi route | `{"maxRequests": 100, "perSeconds": 60}` |
| `rate_limit_by_user` | Giới hạn tần suất cho từng người dùng trên mỗi route, chỉ dùng sau xác thực | `{"maxRequests": 50, "perSeconds": 60}` |
| `rate_limit_by_route` | Giới hạn tần suất cho mỗi route; mọi người dùng dùng chung hạn mức | `{"maxRequests": 200, "perSeconds": 60}` |

Quy tắc IP hỗ trợ **ký hiệu CIDR**: `10.0.0.0/8`, `192.168.1.0/24`, `172.16.0.0/12`.

Khi vượt hạn mức, response trả về **429** cùng các header có cấu trúc (xem [Response khi bị từ chối](#response-khi-bi-tu-choi) bên dưới).

## Cây guard

Bạn có thể kết hợp guard bằng logic **AND** / **OR** để xây dựng quy tắc phức tạp.

### AND (mọi quy tắc đều phải đạt)

```
Guard AND
├── ip_whitelist: chỉ IP văn phòng
└── rate_limit_by_ip: tối đa 100/phút
```

Cả hai điều kiện đều phải đạt. Nếu một trong hai không đạt, request bị từ chối.

### OR (ít nhất một quy tắc phải đạt)

```
Guard OR
├── ip_whitelist: cho phép IP nội bộ (đạt ngay)
└── rate_limit_by_ip: tối đa 100/phút (chỉ kiểm tra khi IP không thuộc whitelist)
```

IP nội bộ bỏ qua giới hạn tần suất. IP bên ngoài sẽ bị áp dụng giới hạn này.

### Lồng guard

Bạn có thể lồng guard để biểu diễn logic phức tạp hơn:

```
Guard OR (gốc)
├── ip_whitelist: 10.0.0.0/8
└── Guard AND (con)
    ├── rate_limit_by_ip: tối đa 100/phút
    └── rate_limit_by_user: tối đa 50/phút
```

Kết quả: IP nội bộ đạt ngay. IP bên ngoài phải vượt qua cả hai giới hạn tần suất.

## Cấu hình

### Tạo guard

```
POST /api/enfyra_guard
{
  "name": "Login Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<route_id>" },
  "methods": [
    { "id": "<POST_method_id>" }
  ]
}
```

| Trường | Giá trị | Mô tả |
|---|---|---|
| `position` | `pre_auth`, `post_auth` | Thời điểm guard chạy |
| `combinator` | `and`, `or` | Cách kết hợp các quy tắc (mặc định: `and`) |
| `isGlobal` | `true` / `false` | Áp dụng cho mọi route |
| `route` | relation | Áp dụng cho một route cụ thể; để `null` nếu là global |
| `methods` | relation | Áp dụng cho các HTTP method cụ thể; để trống để áp dụng cho mọi method |
| `priority` | number | Thứ tự thực thi; giá trị nhỏ chạy trước |

### Thêm quy tắc vào guard

```
POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 5, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

Để chỉ áp dụng một quy tắc cho những người dùng cụ thể:

```
POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 30, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" },
  "users": [
    { "id": "<user_id_1>" },
    { "id": "<user_id_2>" }
  ]
}
```

Khi `users` rỗng, quy tắc áp dụng cho tất cả người dùng.

### Lồng guard

Đặt `parent` để tạo guard con:

```
POST /api/enfyra_guard
{
  "name": "External Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "parent": { "id": "<parent_guard_id>" },
  "isEnabled": true
}
```

## Mẫu cấu hình phổ biến

### Chặn IP không tin cậy trên mọi route

```
POST /api/enfyra_guard
{
  "name": "Global IP Blacklist",
  "position": "pre_auth",
  "combinator": "and",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/enfyra_guard_rule
{
  "type": "ip_blacklist",
  "config": { "ips": ["1.2.3.4", "5.6.7.8"] },
  "guard": { "id": "<guard_id>" }
}
```

### Giới hạn số lần đăng nhập thử

Giới hạn 5 request mỗi phút cho mỗi IP, trước khi xác thực:

```
POST /api/enfyra_guard
{
  "name": "Login Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<login_route_id>" },
  "methods": [{ "id": "<POST_method_id>" }]
}

POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 5, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### Chỉ cho phép IP văn phòng và giới hạn tần suất cho route quản trị

```
POST /api/enfyra_guard
{
  "name": "Admin Access Control",
  "position": "post_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<admin_route_id>" }
}

POST /api/enfyra_guard_rule
{
  "type": "ip_whitelist",
  "config": { "ips": ["10.0.0.0/8", "203.0.113.0/24"] },
  "guard": { "id": "<guard_id>" }
}

POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 100, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### Danh sách cho phép HOẶC giới hạn tần suất (IP nội bộ bỏ qua giới hạn)

```
POST /api/enfyra_guard
{
  "name": "Internal or Rate Limited",
  "position": "pre_auth",
  "combinator": "or",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/enfyra_guard_rule
{
  "type": "ip_whitelist",
  "config": { "ips": ["10.0.0.0/8"] },
  "guard": { "id": "<guard_id>" }
}

// Tạo guard con cho lưu lượng từ bên ngoài
POST /api/enfyra_guard
{
  "name": "External Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "parent": { "id": "<root_guard_id>" },
  "isEnabled": true
}

POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 100, "perSeconds": 60 },
  "guard": { "id": "<child_guard_id>" }
}
```

### Giới hạn tần suất API toàn cục

```
POST /api/enfyra_guard
{
  "name": "Global API Rate Limit",
  "position": "pre_auth",
  "combinator": "and",
  "isGlobal": true,
  "isEnabled": true
}

POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_ip",
  "config": { "maxRequests": 200, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" }
}
```

### Hạn mức chặt hơn cho từng người dùng

Áp dụng hạn mức chặt hơn cho những người dùng cụ thể:

```
POST /api/enfyra_guard
{
  "name": "Heavy User Limit",
  "position": "post_auth",
  "combinator": "and",
  "isEnabled": true,
  "route": { "id": "<api_route_id>" }
}

POST /api/enfyra_guard_rule
{
  "type": "rate_limit_by_user",
  "config": { "maxRequests": 30, "perSeconds": 60 },
  "guard": { "id": "<guard_id>" },
  "users": [{ "id": "<user_id_1>" }, { "id": "<user_id_2>" }]
}
```

## Response khi bị từ chối

Khi guard chặn một request, response có **cấu trúc rõ ràng** để client có thể xác định lý do bị chặn và thời điểm có thể thử lại.

### Mã lỗi

| Lý do | `errorCode` | HTTP Status |
|---|---|---|
| Vượt giới hạn tần suất | `RATE_LIMIT_EXCEEDED` | 429 |
| IP không thuộc whitelist | `IP_NOT_ALLOWED` | 403 |
| IP nằm trong blacklist | `IP_BLOCKED` | 403 |

### Body response REST

```json
{
  "statusCode": 429,
  "message": "Too Many Requests",
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "details": {
    "reason": "rate_limit",
    "scope": "ip",
    "limit": 100,
    "remaining": 0,
    "windowSeconds": 60,
    "retryAfterSeconds": 42,
    "resetAt": 1722650400000
  }
}
```

Với từ chối IP, `details` tối giản:

```json
{
  "statusCode": 403,
  "message": "Forbidden",
  "errorCode": "IP_BLOCKED",
  "details": { "reason": "ip_blocked" }
}
```

### Header response

**Khi bị từ chối (429):**

| Header | Mô tả |
|---|---|
| `Retry-After` | Số giây chờ trước khi thử lại |
| `X-RateLimit-Limit` | Số request tối đa trong cửa sổ |
| `X-RateLimit-Remaining` | Số request còn lại (0 khi bị chặn) |
| `X-RateLimit-Reset` | Epoch ms khi cửa sổ đặt lại |
| `X-RateLimit-Window` | Thời lượng cửa sổ (giây) |
| `X-RateLimit-Scope` | `ip`, `user`, hoặc `route` |
| `X-RateLimit-Used` | Số request đã dùng trong cửa sổ |
| `X-Enfyra-Guard-Reason` | `rate_limit`, `ip_not_allowed`, hoặc `ip_blocked` |
| `X-Enfyra-Guard-Error-Code` | Giống `errorCode` trong body |
| `X-Enfyra-Guard-Scope` | Phạm vi giới hạn (chỉ rate limit) |

**Khi request thành công qua guard:**

Ngay cả khi request được chấp nhận, Enfyra trả về header của bucket giới hạn nghiêm ngặt nhất (tỷ lệ `remaining / limit` thấp nhất) để client có thể chủ động điều tiết:

| Header | Mô tả |
|---|---|
| `X-RateLimit-Limit` | Số request tối đa trong cửa sổ |
| `X-RateLimit-Remaining` | Số request còn lại |
| `X-RateLimit-Reset` | Epoch ms khi cửa sổ đặt lại |
| `X-RateLimit-Window` | Thời lượng cửa sổ (giây) |
| `X-RateLimit-Scope` | `ip`, `user`, hoặc `route` |
| `X-RateLimit-Used` | Số request đã dùng |

### Từ chối trong GraphQL

Guard từ chối trong GraphQL resolver trả về extensions có cấu trúc:

```json
{
  "errors": [{
    "message": "Too Many Requests",
    "extensions": {
      "code": "RATE_LIMIT_EXCEEDED",
      "statusCode": 429,
      "details": {
        "reason": "rate_limit",
        "scope": "ip",
        "limit": 100,
        "remaining": 0,
        "windowSeconds": 60,
        "retryAfterSeconds": 42,
        "resetAt": 1722650400000
      }
    }
  }]
}
```

### Những gì không bao giờ lộ ra

Thông tin nội bộ của guard không gửi tới client: tên guard, ID quy tắc, khóa Redis, danh sách IP đã cấu hình, và user ID gốc không xuất hiện trong response.

## Cảnh báo guard

Mỗi lần guard từ chối được tự động ghi vào bảng hệ thống `enfyra_guard_alert` để giám sát.

| Cột | Mô tả |
|---|---|
| `scope` | `ip`, `user`, hoặc `route` |
| `scopeKey` | Đối tượng bị chặn (địa chỉ IP, user ID, hoặc route path) |
| `routePath` | Route đã từ chối request |
| `method` | HTTP method của request bị từ chối |
| `errorCode` | `RATE_LIMIT_EXCEEDED`, `IP_NOT_ALLOWED`, hoặc `IP_BLOCKED` |
| `guardName` | Tên guard đã từ chối (tham khảo cho quản trị) |
| `createdAt` | Thời điểm từ chối |

Cảnh báo hiển thị tại **Cài đặt > Quản trị > Giám sát runtime > tab Guard**, gồm phần tổng hợp đối tượng vi phạm lặp lại và nhật ký từ chối theo thời gian.

Route cảnh báo (`/enfyra_guard_alert`) chỉ hỗ trợ `GET` và `DELETE` — bản ghi được tạo tự động bởi middleware guard, không thể chèn qua API.

## Guard và giới hạn tần suất trong preHook

| Khía cạnh | Guard | preHook (`$helpers.$rateLimit`) |
|---|---|---|
| Thiết lập | Giao diện quản trị / API (không cần code) | Script tùy chỉnh |
| Chạy trước xác thực? | Có (`pre_auth`) | Không |
| Logic cây (AND/OR) | Tích hợp sẵn | Tự viết script |
| Phù hợp nhất | Bảo vệ IP/tần suất theo chuẩn | Logic động, tùy chỉnh |

Dùng **guard** cho cơ chế bảo vệ tiêu chuẩn, cấu hình bằng metadata. Dùng **giới hạn tần suất trong preHook** khi cần logic tùy chỉnh như hạn mức có điều kiện hoặc khóa động.

## Lưu ý

- `rate_limit_by_user` chỉ dùng được ở vị trí `post_auth` vì cần context người dùng.
- Các quy tắc trong một guard được đánh giá theo chi phí, từ rẻ đến đắt; kiểm tra IP chạy trước bộ đếm giới hạn tần suất.
- Guard áp dụng theo từng instance; thay đổi được đồng bộ giữa các instance qua Redis Pub/Sub.
- `priority` quyết định thứ tự thực thi; giá trị nhỏ chạy trước.

## Đọc tiếp

- Xem [Vòng đời API](./api-lifecycle.md) để nắm toàn bộ luồng request.
- Xem [Hook và Handler](./hooks-handlers/prehooks.md) để triển khai cơ chế bảo vệ bằng code.
- Xem [Tham chiếu Context - Giới hạn tần suất](./context-reference/helpers-cache.md#rate-limiting) để dùng giới hạn tần suất trong preHook.
