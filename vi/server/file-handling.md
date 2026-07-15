---
slug: xu-ly-tep
---

# Xử lý tệp

Enfyra hỗ trợ tải tệp lên từ request, bản ghi tệp trên storage, cập nhật metadata và xóa tệp qua các helper trong script động.

## Tệp tải lên trong context

Request multipart cung cấp tệp được tải lên qua `$ctx.$uploadedFile` hoặc `@UPLOADED_FILE`.

```javascript
const file = @UPLOADED_FILE;

if (!file) {
  @THROW400("File is required");
}

return {
  originalname: file.originalname,
  mimetype: file.mimetype,
  size: file.size,
  fieldname: file.fieldname,
};
```

Các thuộc tính có sẵn:

| Thuộc tính | Kiểu | Mô tả |
| --- | --- | --- |
| `originalname` | `string` | Tên tệp do client gửi lên |
| `mimetype` | `string` | MIME type do client cung cấp |
| `encoding` | `string` | Kiểu mã hóa của biểu mẫu tải lên |
| `path` | `string` | Đường dẫn tệp tạm trên server, được các helper Enfyra dùng nội bộ |
| `size` | `number` | Kích thước tệp theo byte |
| `fieldname` | `string` | Tên trường multipart, thường là `file` |

Không đọc `@UPLOADED_FILE.path` thành `Buffer` trong script. Với tệp tải lên từ request thông thường, hãy truyền thẳng đối tượng tệp vào `@STORAGE.$upload` hoặc `@STORAGE.$update` để Enfyra stream từ đĩa sang storage backend đã chọn.

## Tải tệp lên

Dùng `$ctx.$storage.$upload` hoặc `@STORAGE.$upload` để tải tệp lên storage và tạo bản ghi `enfyra_file`.

### Tải tệp từ request lên

```javascript
if (!@UPLOADED_FILE) {
  @THROW400("File is required");
}

const saved = await @STORAGE.$upload({
  file: @UPLOADED_FILE,
  storageConfig: @BODY.storageConfig,
  folder: @BODY.folder,
  title: @BODY.title,
  description: @BODY.description,
});

return saved;
```

Luồng này an toàn với tệp lớn. Enfyra ghi phần tải lên multipart vào tệp tạm, rồi `$storage.$upload({ file })` mở `Readable` stream từ tệp đó. Toàn bộ tệp không bị nạp vào RAM.

### Tải tệp do script tạo hoặc xử lý lên

Chỉ dùng `buffer` khi chính script tạo hoặc biến đổi một tệp nhỏ, chẳng hạn ảnh thumbnail.

```javascript
const pngBuffer = await makeThumbnail();

const saved = await @STORAGE.$upload({
  filename: "thumbnail.png",
  mimetype: "image/png",
  buffer: pngBuffer,
  size: pngBuffer.length,
  description: "Generated thumbnail",
});

return saved;
```

Không dùng dạng `buffer` cho tệp lớn tải từ request hoặc bản sao lưu cơ sở dữ liệu. Với tệp lớn, dùng dạng `file: @UPLOADED_FILE`.

### Tùy chọn tải lên

| Tùy chọn | Kiểu | Bắt buộc | Mô tả |
| --- | --- | --- | --- |
| `file` | đối tượng `@UPLOADED_FILE` | Bắt buộc với tệp từ request | Stream tệp từ tệp tạm của Enfyra |
| `buffer` | `Buffer` | Bắt buộc với tệp do script tạo | Dữ liệu trong bộ nhớ cho tệp nhỏ được tạo hoặc biến đổi |
| `filename` / `originalname` | `string` | Bắt buộc với `buffer` | Tên tệp cần lưu |
| `mimetype` | `string` | Bắt buộc với `buffer` | MIME type cần lưu |
| `size` | `number` | Không | Mặc định là kích thước tệp request hoặc độ dài buffer |
| `folder` | `number \| { id: number }` | Không | Quan hệ thư mục |
| `storageConfig` | `number \| { id: number }` | Không | Quan hệ cấu hình storage |
| `title` | `string` | Không | Tiêu đề tệp |
| `description` | `string` | Không | Mô tả tệp |
| `isPublic` | `boolean` | Không | Cho phép truy cập ẩn danh qua `/assets/:id` khi là `true` |

Chỉ truyền `file` hoặc `buffer`, không truyền cả hai.

### Tiến trình tải lên qua HTTP

`$upload` và `$update` có thay blob không nhận `onProgress`. Tiến trình tải lên do tầng request multipart đã xác thực quản lý.

Khi trình duyệt hoặc API client tải tệp lên, hãy tạo id ở phía client và gửi qua `x-enfyra-upload-id`. Enfyra phát `$system:upload:progress` tới room Socket.IO Admin của người dùng hiện tại.

Dữ liệu sự kiện:

```javascript
{
  uploadId: "client-generated-id",
  phase: "receiving", // receiving, completed, or failed
  loaded: 1024,
  total: 2048,
  percent: 50,
  fileName: "avatar.png",
  route: "/profile/avatar",
  method: "POST"
}
```

Sự kiện có sẵn này chỉ bao phủ giai đoạn nhận multipart. Theo mặc định, Enfyra không cung cấp tiến trình truyền dữ liệu tới storage hoặc cloud provider.

## Cập nhật tệp

Dùng `$ctx.$storage.$update` hoặc `@STORAGE.$update`.

### Thay bằng tệp từ request

```javascript
if (!@UPLOADED_FILE) {
  @THROW400("File is required");
}

const updated = await @STORAGE.$update(@PARAMS.fileId, {
  file: @UPLOADED_FILE,
  title: @BODY.title,
  description: @BODY.description,
});

return updated;
```

### Chỉ cập nhật metadata

```javascript
const updated = await @STORAGE.$update(@PARAMS.fileId, {
  title: @BODY.title,
  description: @BODY.description,
  folder: @BODY.folder,
  isPublic: @BODY.isPublic,
});

return updated;
```

Đổi `storageConfig` bắt buộc phải thay blob trong cùng request. Enfyra từ chối chuyển storage chỉ bằng metadata vì bản ghi không thể trỏ tới backend không chứa đối tượng đó.

## Xóa tệp

```javascript
await @STORAGE.$delete(@PARAMS.fileId);

return { success: true };
```

Thao tác xóa sẽ loại bỏ bản ghi `enfyra_file` và đối tượng vật lý trên storage backend đã cấu hình.

## Truy cập asset công khai

`enfyra_file.isPublic` kiểm soát quyền truy cập ẩn danh vào asset đã lưu. Chỉ đặt `isPublic: true` khi asset có thể được phân phối không cần xác thực qua `/assets/:id` hoặc app proxy tương đương. Với tệp riêng tư cần xác thực, đặt `isPublic: false` hoặc không truyền thuộc tính này.

Cờ truy cập công khai ở cấp tệp này độc lập với khả năng hiển thị của cột và quan hệ. Cột và quan hệ vẫn dùng `isPublished` làm cơ sở phân quyền trường; bản ghi tệp dùng `isPublic`.

## Mẫu kiểm tra dữ liệu

Kiểm tra kiểu và kích thước trước khi lưu:

```javascript
const file = @UPLOADED_FILE;

if (!file) {
  @THROW400("File is required");
}

const allowed = ["image/jpeg", "image/png", "application/pdf"];
if (!allowed.includes(file.mimetype)) {
  @THROW400("Unsupported file type");
}

const maxBytes = 20 * 1024 * 1024;
if (file.size > maxBytes) {
  @THROW400("File is too large");
}

return await @STORAGE.$upload({
  file,
  folder: @BODY.folder,
  description: @BODY.description,
});
```

## Đăng ký đối tượng đã có trên storage

Dùng `@STORAGE.$registerFile` khi một tiến trình bên ngoài đáng tin cậy đã tải đối tượng lên storage backend đã chọn và script chỉ cần tạo bản ghi `enfyra_file`.

```javascript
return await @STORAGE.$registerFile({
  filename: "backup.sql.gz",
  mimetype: "application/gzip",
  location: @BODY.location,
  size: @BODY.size,
  storageConfig: @BODY.storageConfig,
  description: @BODY.description,
});
```

## Request từ client

Gửi dữ liệu multipart với tên trường `file`. Không tự đặt `Content-Type`; trình duyệt phải tự đặt multipart boundary.

```javascript
const form = new FormData();
form.append("file", file);
form.append("description", description);

const response = await fetch("/api/upload", {
  method: "POST",
  credentials: "include",
  body: form,
});
```

## Cách storage xử lý tệp

Nội bộ Enfyra dùng một stream contract thống nhất để tải tệp lên. Tệp từ request được lưu tạm trên đĩa rồi stream tới Local, S3, Cloudflare R2 hoặc Google Cloud Storage. Dạng helper `buffer` chỉ dành cho tệp nhỏ do script tạo và được chuyển thành stream ở ranh giới helper.

## Bước tiếp theo

- [Tham chiếu context](./context-reference/) để xem các trường trong request context.
- [Xử lý lỗi](./error-handling.md) để xem các mẫu `@THROW`.
- [Phương thức repository](./repository-methods/) để truy vấn bản ghi tệp.
