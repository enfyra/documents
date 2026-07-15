---
slug: luong-tu-dong
---

# Luồng tự động

Flow là workflow tự động thực thi một chuỗi step theo trigger. Flow hỗ trợ chạy theo lịch (cron) và trigger thủ công. Với luồng dựa trên sự kiện, hãy dùng handler/hook cùng `@TRIGGER()`.

## Bảng dữ liệu

| Bảng | Mục đích |
|-------|---------|
| `enfyra_flow` | Cấu hình flow (name, trigger, timeout, maxExecutions) |
| `enfyra_flow_step` | Các step của flow, có thứ tự, kiểu và cấu hình |
| `enfyra_flow_execution` | Lịch sử và trạng thái thực thi, truy vấn riêng chứ không lồng trong flow |

## Kiểu trigger

| Kiểu | Ví dụ cấu hình | Mô tả |
|------|---------------|-------------|
| `schedule` | `{"cron": "0 2 * * *", "timezone": "UTC"}` | Chạy theo lịch cron |
| `manual` | `{}` | Do người dùng kích hoạt từ UI, API hoặc `@TRIGGER()` trong handler/hook |

## Kiểu step

| Kiểu | Cấu hình | Mô tả |
|------|--------|-------------|
| `script` | `{"code": "..."}` | Chạy JavaScript tùy chỉnh với đầy đủ context |
| `condition` | `{"code": "return ..."}` | Đánh giá điều kiện theo JS truthy/falsy; truthy đi nhánh `"true"`, falsy đi nhánh `"false"` |
| `query` | `{"table": "...", "filter": {...}, "limit": 10}` | Truy vấn dữ liệu bảng |
| `create` | `{"table": "...", "data": {...}}` | Tạo bản ghi |
| `update` | `{"table": "...", "id": 1, "data": {...}}` | Cập nhật bản ghi theo ID |
| `delete` | `{"table": "...", "id": 1}` | Xóa bản ghi theo ID |
| `http` | `{"url": "...", "method": "POST", "body": {...}, "headers": {...}, "timeout": 30000}` | Gửi HTTP request (tự thêm `Content-Type: application/json` khi có body). Xem [quy tắc URL của HTTP step](#quy-tac-url-cho-http-step-chong-ssrf). |
| `trigger_flow` | `{"flowId": 2}` hoặc `{"flowName": "..."}` | Kích hoạt flow khác |
| `sleep` | `{"ms": 5000}` | Tạm dừng thực thi N ms |
| `log` | `{"message": "..."}` | Ghi message vào execution context |

### Quy tắc URL cho HTTP step (chống SSRF)

Server chỉ cho phép URL `http:` và `https:` không phải mục tiêu SSRF rõ ràng: `localhost`, tên loopback phổ biến, IP private/reserved dạng literal và hostname chỉ phân giải về IP private đều bị từ chối. Hãy ưu tiên **hostname công khai trên Internet**, ví dụ `https://api.example.com`. Callback nội bộ cần thay đổi kiến trúc có chủ đích, không dùng tùy tiện URL nội bộ trong step này.

## Cú pháp mẫu

Flow step hỗ trợ cùng các macro mẫu như handler/hook, đồng thời có macro riêng cho flow. Code được tự transpile khi đưa vào cache.

| Macro | Mở rộng thành | Mô tả |
|-------|-----------|-------------|
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` | Dữ liệu payload đầu vào |
| `@TRIGGER` | `$ctx.$trigger` | Kích hoạt flow từ handler/hook |
| `@FLOW_LAST` | `$ctx.$flow.$last` | Kết quả step gần nhất |
| `@FLOW` | `$ctx.$flow` | Toàn bộ chuỗi dữ liệu flow |
| `@FLOW_META` | `$ctx.$flow.$meta` | Metadata flow (flowId, flowName, executionId, depth) |
| `#table_name` | `$ctx.$repos.table_name` | Repository của bảng |
| `@HELPERS` | `$ctx.$helpers` | Helper (jwt, bcrypt, autoSlug) |
| `@USER` | `$ctx.$user` | Người dùng hiện tại (null với cron) |
| `@THROW400` … `@THROW503`, `@THROW` | `$ctx.$throw['400']`, …, `$ctx.$throw` | Helper lỗi HTTP (dùng key dạng số, không dùng `'4xx'`) |
| `%package` | `$ctx.$pkgs.package` | Package đã cài |

## Chuỗi dữ liệu

Mỗi lần chạy flow giữ một context dùng chung (`$ctx.$flow` / `@FLOW`) để truyền dữ liệu giữa các step:

```
@FLOW
  ├── @FLOW_PAYLOAD      // Input data passed to the flow
  ├── @FLOW_LAST         // Result of the most recent step
  ├── @FLOW_META         // { flowId, flowName, executionId, startedAt }
  ├── @FLOW.step_1  // Result of step with key "step_1"
  └── @FLOW.step_2  // Result of step with key "step_2"
```

### Truy cập dữ liệu trong script step (nên dùng cú pháp mẫu)

```javascript
// Cú pháp mẫu (khuyến nghị)
const email = @FLOW_PAYLOAD.email;
const user = @FLOW.find_user?.data?.[0];
const prev = @FLOW_LAST;
const orders = await #order.find({
  filter: { userId: { _eq: user.id } }
});
return orders;
```

```javascript
// Cú pháp đầy đủ tương đương (không khuyến nghị)
// $ctx.$flow.$payload.email, $ctx.$repos.order, etc.
```

## Rẽ nhánh điều kiện

Condition step hỗ trợ rẽ nhánh true/false. Child step tham chiếu condition qua relation `parent` và trường `branch`.

```
[query_users]              ← root step (parent: null)
    ↓
[check_has_users]          ← root step, type: condition
    ├── true:
    │   [process_users]    ← parent: check_has_users, branch: "true"
    │   [send_report]      ← parent: check_has_users, branch: "true"
    └── false:
        [log_empty]        ← parent: check_has_users, branch: "false"
    ↓
[cleanup]                  ← root step (continues after condition)
```

Tạo branch step qua API:
```
POST /api/enfyra_flow_step
{
  "flow": { "id": 1 },
  "key": "process_users",
  "stepOrder": 1,
  "type": "script",
  "config": { "code": "return #enfyra_user.find({ limit: 100 })" },
  "parent": { "id": 5 },
  "branch": "true"
}
```

Quy tắc:
- Step có `parent: null` nằm ở root level và chạy tuần tự
- Condition dùng JS truthy/falsy: `return user` (object = truthy), `return null` (falsy), `return count > 0` (boolean)
- Child truthy có `branch: "true"` được chạy
- Child falsy có `branch: "false"` được chạy
- Sau khi condition và child branch hoàn tất, root step kế tiếp tiếp tục
- Mỗi branch step có thể đặt `onError: skip/stop/retry` độc lập

## Xử lý lỗi

| Chiến lược | Hành vi |
|----------|----------|
| `stop` | Dừng flow, đánh dấu lần chạy thất bại (mặc định) |
| `skip` | Ghi nhận lỗi và tiếp tục step kế tiếp |
| `retry` | Thử lại với exponential backoff tối đa `retryAttempts` lần |

## Endpoint quản trị

| Endpoint | Mô tả |
|----------|-------------|
| `POST /admin/flow/trigger/:id` | Kích hoạt một lần chạy flow qua BullMQ |
| `POST /admin/test/run` | Kiểm thử flow step không cần lưu bằng `kind: "flow_step"` |

## Kích hoạt flow từ handler

```javascript
await @TRIGGER('send-welcome-email', { userId: @USER.id });
await @TRIGGER(5, { orderId: @PARAMS.id, total: 100 });
```

## Lịch sử thực thi

Truy vấn riêng bản ghi thực thi, không lồng bên dưới flow:

```
GET /api/enfyra_flow_execution?filter={"flow":{"_eq":1}}&sort=-id&limit=10
```

Mỗi bản ghi thực thi gồm:
- `status`: pending, running, completed, failed, cancelled
- `currentStep`: step nơi flow dừng hoặc đang chạy
- `completedSteps`: mảng key của các step đã hoàn tất thành công
- `error`: chi tiết lỗi khi thất bại (message + stack)
- `duration`: tổng thời gian chạy tính bằng ms

## Ví dụ: flow xử lý đơn hàng

### 1. Tạo flow

```
POST /api/enfyra_flow
{
  "name": "process-order",
  "triggerType": "manual",
  "timeout": 30000,
  "isEnabled": true
}
```

Sau đó kích hoạt từ post-hook của `/order`:
```javascript
await @TRIGGER('process-order', { data: @DATA });
```

### 2. Thêm step

```
POST /api/enfyra_flow_step
{
  "flow": { "id": 1 },
  "key": "validate_stock",
  "stepOrder": 1,
  "type": "script",
  "config": {
    "code": "const order = @FLOW_PAYLOAD.data; const product = await #product.find({ filter: { id: { _eq: order.productId } }, limit: 1 }); return { inStock: product.data[0]?.stock > order.quantity }"
  },
  "timeout": 5000,
  "onError": "stop"
}
```

### 3. Kiểm thử step trước khi lưu

```
POST /api/admin/test/run
{
  "kind": "flow_step",
  "type": "query",
  "config": { "table": "enfyra_user", "filter": { "status": { "_eq": "active" } }, "limit": 5 },
  "timeout": 5000
}
```

Response: `{ "success": true, "result": { "data": [...] }, "duration": 42 }`

### 4. Kích hoạt thủ công

```
POST /api/admin/flow/trigger/1
{ "payload": { "orderId": 123 } }
```

### 5. Xem lịch sử thực thi

```
GET /api/enfyra_flow_execution?filter={"flow":{"_eq":1}}&sort=-id&limit=10
```

## Ví dụ flow chạy theo lịch

```
POST /api/enfyra_flow
{
  "name": "daily-cleanup",
  "triggerType": "schedule",
  "triggerConfig": { "cron": "0 2 * * *", "timezone": "Asia/Ho_Chi_Minh" },
  "timeout": 60000,
  "isEnabled": true
}
```

Lịch cron được đăng ký tự động bằng BullMQ Job Scheduler khi cache của flow reload.

## An toàn và giới hạn

| Hạng mục | Chi tiết |
|---------|--------|
| Độ sâu lồng tối đa | 10 (flow kích hoạt flow qua `trigger_flow` hoặc `$trigger()`) |
| Phát hiện vòng lặp | Theo dõi flow ID đã đi qua trong chuỗi; ABA bị từ chối ngay |
| HTTP timeout | Mặc định 30 giây, cấu hình từng step bằng `config.timeout` |
| Lịch sử thực thi | Tự dọn theo `maxExecutions` của flow, mặc định 100 và xóa bản ghi cũ nhất |
| Step timeout | Theo từng step hoặc kế thừa `timeout` của flow, mặc định 5000ms |
| Retry backoff | Exponential: 1s, 2s, 4s, 8s... tối đa 30 giây |

## Kích hoạt flow từ flow step

Flow step có `$trigger()` giống handler/hook:

```javascript
// Bên trong một script step
const result = await @TRIGGER('send-notification', { userId: @FLOW_PAYLOAD.userId });
```

Lời gọi này tuân theo cùng giới hạn độ sâu lồng và phát hiện vòng lặp.
