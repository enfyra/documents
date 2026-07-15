---
slug: tham-chieu-api/tep-va-luu-tru
---

# File và Storage

Dùng các endpoint này để tải file lên, sắp xếp chúng theo folder và phân phối asset từ ứng dụng. Base URL: `{appUrl}/api`

## File

### Liệt kê file

```
GET {appUrl}/api/enfyra_file?fields=id,filename,mimetype,size,folder&filter={"folder":{"_eq":123}}
```

### Tải file lên

Dùng `multipart/form-data` với field `file`. Khi cần, thêm `folder`, `title`, `description` và `isPublic` vào form.

```
POST {appUrl}/api/enfyra_file
```

### Lấy metadata / cập nhật / xóa file

```
GET    {appUrl}/api/enfyra_file?filter={"id":{"_eq":123}}&limit=1
PATCH  {appUrl}/api/enfyra_file/{id}
DELETE {appUrl}/api/enfyra_file/{id}
```

`enfyra_file.isPublic` kiểm soát việc truy cập asset không cần xác thực qua `/api/assets/{id}`. Chỉ đặt `isPublic: true` cho file có thể được phân phối công khai. Quy tắc hiển thị column và relation vẫn dùng `isPublished`; quyền truy cập file công khai không dùng cờ này.

---

## Folder

**Liệt kê folder:**

```
GET {appUrl}/api/enfyra_folder?filter={"parent":{"_is_null":true}}
```

**Tạo folder:**

```
POST {appUrl}/api/enfyra_folder
Body: { "name": "My Folder", "parent": null }
```

**Cây folder:**

```
GET {appUrl}/api/enfyra_folder/tree
```

Trả về cấu trúc folder lồng nhau.

---

## Tải file xuống

Phân phối file theo ID (ảnh, PDF, v.v.):

```
GET {appUrl}/api/assets/{id}
```

Endpoint trả về dữ liệu nhị phân của file với `Content-Type` phù hợp. File công khai có thể được phân phối không cần xác thực khi `enfyra_file.isPublic=true`; file riêng tư phải đi qua luồng request đã xác thực thông thường.
