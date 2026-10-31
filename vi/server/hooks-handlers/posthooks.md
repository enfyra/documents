---
slug: post-hook
---

# Hooks và Handler - postHooks

postHook chạy sau handler (hoặc sau lỗi của preHook/handler). Dùng chúng để biến đổi response, ghi audit log và thực hiện tác vụ phụ.

## Cách thực thi

- postHook vẫn chạy khi preHook hoặc handler throw lỗi thông thường, dù các preHook sau và handler bị bỏ qua khi preHook từ chối request. Cancellation, execution deadline và lỗi stream terminal dùng nhánh audit opt-in riêng bên dưới.
- PreHook return sớm không từ chối request: các preHook sau và handler tiếp tục, rồi postHook chạy bình thường.
- Đặt `priority` khác nhau cho các postHook phụ thuộc nhau (số nhỏ chạy trước). Đọc decision của request qua `@SHARE`; kiểm tra `@ERROR` trước tác vụ chỉ dành cho thành công.
- Trên nhánh lỗi: `@ERROR` có giá trị, `@DATA` là `null`, `@STATUS` là mã trạng thái lỗi.
- Trên nhánh thành công: `@ERROR` là `undefined`, `@DATA` là kết quả của handler, `@STATUS` là `200`.
- postHook chạy như **tác vụ phụ**; trên nhánh lỗi, lỗi gốc luôn được ném lại sau khi mọi postHook hoàn tất.
- Nếu một postHook thất bại, các postHook khác vẫn chạy (mỗi postHook có `try/catch` riêng).

## Khi nào dùng postHook

- Biến đổi dữ liệu response (nhánh thành công)
- Thêm các trường tính toán vào response
- Ghi audit log (cả thành công lẫn lỗi)
- Kích hoạt tác vụ phụ (email, thông báo)
- Xử lý hoặc ghi log lỗi xảy ra trong preHook hay handler
- Thêm metadata vào response

## Ví dụ postHook cơ bản

```javascript
// Transform response (success path only)
if (@DATA && Array.isArray(@DATA.data)) {
  @DATA.data = @DATA.data.map(item => ({
    ...item,
    fullName: `${item.firstName} ${item.lastName}`
  }));
}
```

## Bổ sung response

```javascript
// Add metadata
if (@DATA) {
  @DATA.meta = {
    processedAt: new Date(),
    processedBy: @USER?.id,
    version: '1.0'
  };
}

// Add computed fields to each item
if (@DATA && Array.isArray(@DATA.data)) {
  @DATA.data = @DATA.data.map(item => ({
    ...item,
    isActive: item.status === 'active',
    displayName: item.nickname || item.name
  }));
}
```

## Audit log

```javascript
// Log all operations (success and error)
await #audit_logs.create({
  data: {
    action: `${@API.request.method} ${@API.request.url}`,
    userId: @USER?.id,
    resourceId: @PARAMS.id,
    statusCode: @STATUS,
    error: @ERROR ? @ERROR.message : null,
    timestamp: new Date()
  }
});
```

## Xử lý lỗi

postHook nhận context lỗi qua `@ERROR` (hoặc `$ctx.$error`). Trên nhánh lỗi, lỗi luôn được ném lại sau khi postHook hoàn tất; postHook không thể ghi đè lỗi.

```javascript
// Log errors for monitoring
if (@ERROR) {
  @LOGS(`Error: ${@ERROR.message}`);

  await #error_logs.create({
    data: {
      errorMessage: @ERROR.message,
      statusCode: @ERROR.statusCode,
      userId: @USER?.id,
      url: @API.request.url,
      timestamp: new Date()
    }
  });

  // Send notification on critical errors
  if (@ERROR.statusCode >= 500) {
    await @SOCKET.emitToRoom('/admin', 'admin', 'server-error', {
      message: @ERROR.message,
      url: @API.request.url
    });
  }
}
```

## Audit request HTTP bị kết thúc

Với Server release hỗ trợ terminal failure audit, post-hook được gắn vào route có thể opt-in bằng cách bắt đầu source bằng directive chính xác:

```javascript
'enfyra:client-disconnect-audit';
if (@ERROR?.origin) {
  @LOGS('Request terminated', {
    executionId: @API.request.executionId,
    origin: @ERROR.origin,
    code: @ERROR.code,
    statusCode: @STATUS,
  });
}
```

Server chỉ chạy các audit hook đã gắn này trong task mới, với giới hạn admission/execution riêng năm giây. Không chạy lại inference hay post-hook thông thường; task audit không có response đang hoạt động. Đây là finalization best-effort, không phải durable outbox; kiểm tra hỗ trợ trên deployment trước khi dựa vào nó.

Nguyên nhân đầu tiên do host ghi nhận là nguồn có thẩm quyền:

| Origin | Status / code |
|---|---|
| Client ngắt kết nối | `499 client_closed_request` |
| Response stream lỗi | `502 response_stream_error` |
| Deadline tổng/idle của stream | `504 response_stream_timeout` |
| Deadline execution | `504 execution_timeout` |
| Lỗi execution terminal khác | `500 execution_failed` |

Không diễn giải upstream HTTP 499 hoặc abort flag xuất hiện sau thành client disconnect. `@ERROR` có `origin`, `statusCode`, `code` và `message` an toàn; nhánh lỗi thông thường giữ error shape hiện có. Audit context giữ identity đã xác thực và snapshot cuối trên host, không bảo đảm có thay đổi `@SHARE` đang diễn ra trong isolate.

`@API.request.executionId` là UUID do Server sinh, độc lập với correlation/session ID của caller. Ứng dụng ghi failure row phải phối hợp với settlement thành công dưới cùng transaction/lock và kiểm tra trạng thái đã persist để execution đã settle không có thêm failure row trùng. Giữ fallback đã xác thực khi snapshot thiếu dữ liệu; không ghi prompt, credential hoặc diagnostic private của provider vào audit public.

Trước response header, lỗi do Server quản lý trả JSON an toàn với status khớp và cặp retry signal. Sau khi commit stream, wire status không đổi được; Server đóng stream. Không gửi response thứ hai sau client disconnect. Tác vụ bắt buộc phải bền vững vẫn cần outbox hoặc flow.

## Context có trong postHook

| Biến | Đường dẫn thành công | Đường dẫn lỗi |
|----------|-------------|------------|
| `@DATA` | Kết quả xử lý | `null` |
| `@STATUS` | `200` | Mã trạng thái lỗi (ví dụ `400`, `500`) |
| `@ERROR` | `undefined` | `{ message, name, statusCode, details, timestamp }` |
| `@API.error` | `undefined` | Tương tự như `@ERROR` |
| `@API.response` | `undefined` | `{ statusCode, responseTime, timestamp }` |
| `@USER` | Người dùng hiện tại | Người dùng hiện tại |
| `@SHARE` | Dữ liệu được chia sẻ từ preHooks | Dữ liệu được chia sẻ từ preHooks |
| `@LOGS(...)` | Khả dụng | Khả dụng |

## Tiếp theo

- Xem [preHooks](./prehooks.md) để biết thao tác trước handler
- Tìm hiểu [Handler tùy chỉnh](./custom-handlers.md) cho logic nghiệp vụ tùy chỉnh
- Xem [Mẫu phổ biến](./patterns.md) để biết thực hành tốt
