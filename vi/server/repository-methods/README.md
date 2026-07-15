---
slug: phuong-thuc-repository
---

# Phương thức Repository

Repository là cách chính để làm việc với bảng cơ sở dữ liệu trong Enfyra. Mỗi bảng bạn tạo tự có một repository, truy cập qua `$ctx.$repos.tableName`.

## Tham chiếu nhanh

**Mọi phương thức repository trả dữ liệu theo dạng:**
```javascript
{
  data: [...],        // Array of records
  meta: {            // Metadata (when requested)
    totalCount: 100,
    filterCount: 25
  }
}
```

**Các phương thức có sẵn:**
- [Tìm](./find.md) — Truy vấn bản ghi, lọc, sắp xếp và phân trang
- [Tạo](./create-update-delete.md#create) — Tạo bản ghi mới
- [Cập nhật](./create-update-delete.md#update) — Cập nhật bản ghi theo ID
- [Xóa](./create-update-delete.md#delete) — Xóa bản ghi theo ID

## Truy cập repository

Repository có trong đối tượng context (`$ctx`) của hook và handler:

```javascript
// Access a repository by table name
const productsRepo = $ctx.$repos.products;
const usersRepo = $ctx.$repos.enfyra_user;

// Access the main table repository (if configured in route)
const mainRepo = $ctx.$repos.main;
```

**Lưu ý:**
- Repository được phân giải từ **metadata** theo **`name` hoặc `alias` của bảng** (xem `RepoRegistryService`). Truy cập cơ bản không cần cấu hình danh sách riêng cho từng route.
- **`$ctx.$repos.main`** là bảng chính của route hiện tại (áp dụng quyền trường).
- **`$ctx.$repos.secure.<name>`** áp dụng quyền tương tự cho bảng khác; **`$ctx.$repos.<name>`** không áp dụng quyền trường, trừ khi dùng `main` / `secure`.
- Mọi phương thức repository đều bất đồng bộ và cần `await`.

## Nội dung tham khảo

- **[Tìm bản ghi](./find.md)** — Hướng dẫn truy vấn bản ghi đầy đủ
- **[Tạo, cập nhật và xóa bản ghi](./create-update-delete.md)** — Các thao tác tạo, cập nhật và xóa
- **[Mẫu phổ biến](./patterns.md)** — Thực hành tốt và mẫu phổ biến

## Tiếp theo

- Tìm hiểu [đối tượng Context ($ctx)](../context-reference/) để biết mọi thuộc tính có sẵn
- Xem [Lọc truy vấn](../query-filtering.md) để biết các mẫu lọc nâng cao
- Xem [Vòng đời API](../api-lifecycle.md) để hiểu vị trí của repository trong luồng request
