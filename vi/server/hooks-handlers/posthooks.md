---
slug: post-hook
---

# Hooks và Handler - postHooks

postHook chạy sau handler (hoặc sau lỗi của preHook/handler). Dùng chúng để biến đổi response, ghi audit log và thực hiện tác vụ phụ.

## Cách thực thi

- postHook **luôn chạy**, kể cả khi preHook hoặc handler ném lỗi.
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
