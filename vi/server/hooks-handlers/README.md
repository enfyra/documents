---
slug: hook-va-handler
---

# Hooks và Handler

Hook và handler cho phép bạn tùy chỉnh hành vi API tại từng điểm trong vòng đời request. Hook chạy trước hoặc sau handler, còn handler chứa logic nghiệp vụ chính.

## Điều hướng nhanh

- [Tổng quan](#tong-quan) — Hook và handler là gì
- [preHooks](./prehooks.md) — Chạy trước handler
- [postHooks](./posthooks.md) — Chạy sau handler
- [Handler tùy chỉnh](./custom-handlers.md) — Thay thế CRUD mặc định
- [Mẫu phổ biến](./patterns.md) — Ví dụ thực tế và thực hành tốt

## Tổng quan

### Hooks

Hook là các đoạn mã chạy tại những thời điểm xác định trong vòng đời request:
- **preHooks**: Chạy trước handler
- **postHooks**: Chạy sau handler

### Handlers

Handler chứa logic nghiệp vụ chính:
- **Custom Handler**: Mã do bạn tùy chỉnh
- **CRUD mặc định**: Thao tác CRUD tự động dựa trên phương thức HTTP

### Luồng thực thi

```
Pre-Auth Guards  Auth  Post-Auth Guards  preHook #1  preHook #2  Handler  postHook #1  postHook #2
```

Guard chạy trước hook để chặn IP hoặc giới hạn tần suất. Hook chạy sau guard để xác thực dữ liệu và xử lý logic nghiệp vụ.

Mọi hook và handler đều truy cập cùng một đối tượng `$ctx`; thay đổi ở một giai đoạn sẽ hiển thị cho các giai đoạn sau.

## Nội dung tham khảo

- **[preHooks](./prehooks.md)** — Xác thực, biến đổi dữ liệu và kiểm tra quyền
- **[postHooks](./posthooks.md)** — Biến đổi response, audit log và tác vụ phụ
- **[Handler tùy chỉnh](./custom-handlers.md)** — Logic nghiệp vụ tùy chỉnh và các loại hook
- **[Mẫu phổ biến](./patterns.md)** — Thực hành tốt và ví dụ thực tế

## Tiếp theo

- Tìm hiểu [Phương thức Repository](../repository-methods/) để thao tác cơ sở dữ liệu
- Xem [Tham chiếu Context](../context-reference/) để biết mọi thuộc tính có sẵn
- Xem [Vòng đời API](../api-lifecycle.md) để hiểu thứ tự thực thi
