---
slug: tao-bang
---

# Hướng dẫn tạo bảng

Hướng dẫn này đi từ việc tạo bảng đầu tiên đến những gì diễn ra sau khi lưu.

> **Điều kiện:** đã hoàn thành [Cài đặt](./installation.md) và quen với [Bắt đầu](./getting-started.md).

## Mở màn hình tạo bảng

1. Click **Collections** ở sidebar.
2. Click **+ Create New Table** trong panel bên trái.

## Quan trọng: thứ tự tạo

Để relation hoạt động, tạo theo thứ tự đúng:

1. **Tạo target table trước.** Muốn quan hệ `Post → Category` thì tạo `Category` trước `Post`.
2. **Thêm cột trước constraint.** Unique constraint và index cần cột có sẵn để tham chiếu.
3. **Thêm cột/relation trước constraint.** Cả unique constraint lẫn index đều cần field có sẵn.

## Form tạo bảng

### Thông tin cơ bản

- **Table Name** – tên bảng, ví dụ `products`, `posts`.
- **Description** – mô tả tùy chọn về dữ liệu bảng lưu.

### Cấu hình cột

**Columns** là các field của bảng. Enfyra tự thêm primary key phù hợp database.

- Click **+ Add Column** để thêm field.
- Cấu hình property của cột trong drawer.

#### Primary key mặc định

Với SQL, mọi bảng tự có field `id`:

- **Type:** `int`
- **Primary Key:** Yes
- **Auto Increment:** Yes
- **Nullable:** No
- **Updatable:** No

Với MongoDB, mọi collection tự có primary field `_id` kiểu `ObjectId`. Cả hai đều do hệ thống quản lý.

#### Property của cột

**Thiết lập cơ bản:**

- `name` – tên field, bắt buộc, không được bắt đầu bằng số hoặc `_`.
- `type` – kiểu dữ liệu; xem bảng Field Types.
- `isNullable` – cho phép giá trị rỗng/null.
- `isUpdatable` – cho phép cập nhật field sau khi tạo bản ghi.
- `isGenerated` – tự bật cho `uuid`, do hệ thống quản lý.
- `isPublished` – baseline hiển thị, mặc định `true`. Đặt `false` để chặn mặc định; dùng field permission để cấp cho role/user cụ thể.
- `isEncrypted` – mã hóa giá trị field kiểu chuỗi khi lưu tại data access layer. Script, REST và repository đọc/ghi plaintext; Enfyra mã hóa khi insert/update và giải mã sau select. Field đã mã hóa không filter hoặc sort được. `isEncrypted` không làm field immutable; nếu giá trị không được đổi sau khi tạo, đặt thêm `isUpdatable=false`. Giá trị phụ thuộc `SECRET_KEY` của server, nên self-hosted phải giữ key ổn định, backup và giống nhau giữa mọi server instance.

**Hiển thị và UX:**

- `description` – tài liệu field bằng rich text, hiện dưới label trong form.
- `placeholder` – placeholder input của form.
- `options` – lựa chọn cho `enum`/`array-select`.

**Default value:** `boolean` dùng true/false, `int`/`number` dùng numeric input, `date` dùng date picker, `enum` dùng dropdown, `array-select` dùng multi-select, `uuid` tự tạo, còn kiểu text dùng text input.

#### Kiểu field

| Type | Mô tả | Trường hợp dùng |
|------|-------|-----------------|
| `uuid` | UUID tự tạo, tự đặt `isGenerated=true` | Định danh duy nhất |
| `varchar` | Text độ dài biến thiên | Tên, tiêu đề, text ngắn |
| `text` | Nội dung text dài | Mô tả, bài viết |
| `int`, `bigint`, `number` | Giá trị số; SQL `id` là auto-increment primary key | Giá, số lượng, ID |
| `boolean` | Giá trị true/false | Cờ, chỉ báo trạng thái |
| `date`, `timestamp` | Ngày/giờ | Ngày tạo, deadline |
| `enum` | Chọn một từ option định sẵn | Trạng thái, danh mục |
| `array-select` | Chọn nhiều option định sẵn | Tag, nhiều danh mục |
| `simple-json` | Dữ liệu JSON | Cấu trúc phức tạp |
| `richtext` | Text có định dạng | Bài viết, mô tả có format |
| `code` | Code có syntax highlighting | Ví dụ code, script |

### Cấu hình nâng cao

#### Unique constraint

- Chỉ hiện sau khi đã thêm cột.
- Click **+ Add** để tạo nhóm constraint.
- Chọn một hay nhiều field phải unique cùng nhau.
- Tạo unique constraint tự tạo index, không cần index riêng cho cùng các field.

#### Index

- Chỉ hiện sau khi đã thêm cột và relation; có thể index cột lẫn relation field.
- Click **+ Add** để tạo nhóm index.
- Chọn một hay nhiều field để index chung.
- Không tạo index cho field đã có unique constraint.

#### Relation

- Chỉ hiện table đã tồn tại; luôn tạo target table trước.
- Danh sách relation hiện type, target table và nullable status.
- Click relation để sửa hoặc **+ Add Relation** để thêm.
- Trong drawer cấu hình: `type` (`one-to-one`, `one-to-many`, `many-to-one`, `many-to-many`), `propertyName`, `inversePropertyName` (tùy chọn, bắt buộc cho O2M), `targetTable`, `isNullable`, `description`.
- Relation đã tạo xuất hiện như field có icon bút chì trong form; xem [Relation Picker System](../app/relation-picker.md).

**Incoming relation:** khi table khác đã trỏ vào bảng này nhưng chưa có reverse property ở bảng hiện tại, link đó hiện thành hàng nét đứt **incoming**, với nguồn `SourceTable.propertyName`. Dùng **Create Inverse**, chọn property name cho relation mới rồi lưu schema. Relation là inverse side có badge **inverse**.

#### onDelete (cascade behavior)

`onDelete` kiểm soát điều xảy ra khi xóa record liên quan:

- **CASCADE**: 1-N/N-1 xóa parent thì xóa child; 1-1 xóa một bên sẽ xóa cả cặp; N-N chỉ cascade các junction row, không xóa row bảng đối diện.
- **SET NULL**: giữ child record nhưng đặt foreign key thành `NULL`; phù hợp khi child có thể tồn tại độc lập.
- **RESTRICT / NO ACTION**: database chặn xóa parent khi còn child; dùng khi cần cleanup hoặc gán lại tường minh trước khi xóa.

Lưu ý runtime: ở N-N, cập nhật mảng relation thay junction row để khớp ID/object mới; target entity không bị xóa. Ở 1-N, cập nhật parent với list child làm child bị bỏ hiện tại **detached** (FK thành `NULL`) thay vì bị xóa; `onDelete` chủ yếu tác động khi xóa chính parent row.

Ví dụ: `user.profile` 1-1 `CASCADE` xóa user sẽ xóa profile và ngược lại; `post.comments` 1-N dùng CASCADE, SET NULL hoặc RESTRICT tùy nghiệp vụ; `post.tags` N-N chỉ xóa junction row khi bỏ tag, không xóa record `tag`.

**Lưu ý quan trọng cho MongoDB:** nếu **update hoặc delete** relation sau khi tạo, toàn bộ relation data trên record bị bỏ, gồm field relation và inverse field. Bạn phải nạp lại dữ liệu sau khi đổi relation metadata. Xem [Query Filtering](../server/query-filtering.md) để biết cách query relation MongoDB.

## Lưu bảng

Sau khi cấu hình, click **+ Create New Table** màu xanh ở góc trên bên phải.

## Điều gì xảy ra sau khi tạo bảng

### 1. API endpoint được sinh ở backend

- REST API có ngay. GraphQL chỉ có sau khi bạn bật GraphQL cho bảng.
- Bốn CRUD endpoint được sinh tự động:
  - `GET /[your-table-name]` – liệt kê record
  - `POST /[your-table-name]` – tạo record
  - `PATCH /[your-table-name]/:id` – cập nhật record
  - `DELETE /[your-table-name]/:id` – xóa record
- API ở backend; frontend dùng HTTP request để gọi chúng.

### 2. Route tự tạo

Backend tự tạo `/[your-table-name]`, xử lý bốn CRUD operation. Vào **Settings > Routings** ở sidebar để xem các table route.

### 3. Tích hợp frontend

Bảng mới xuất hiện trong **Data** ở sidebar. Frontend gọi backend API để tương tác dữ liệu; click **Data** để thấy bảng và thêm record.

## Bước tiếp theo và thực hành tốt

- Vào **Data > [Your Table Name]** để thêm record; xem [Data Management](./data-management.md).
- Mọi thao tác dữ liệu đi qua backend API; frontend không chạm database trực tiếp.
- Dùng [Relation Picker System](../app/relation-picker.md) cho relation field và [Filter System](../app/filter-system.md) để tìm/filter dữ liệu.

**Thiết kế:** lên kế hoạch relation trước, dùng tên bảng/cột rõ ràng, đặt constraint hợp lý. Thêm description, default value hợp lý và chọn type khớp dữ liệu. Dùng `isEncrypted=true` cho API key, token, password, private key và secret khác; không tự viết pre-hook encryption cho app data thông thường. Secret chỉ được đặt một lần thì thêm `isUpdatable=false`; luôn backup `SECRET_KEY`.

**Hiệu năng:** index field query thường xuyên. Enfyra tự tạo single-field index cho `createdAt`, `updatedAt` và scalar `date`, `datetime`, `timestamp`; SQL index thêm `id`, Mongo thêm `_id`. Chỉ thêm compound index cho hot query như `status + createdAt`, `owner + updatedAt`, `project + lastMessageAt`; quá nhiều index làm chậm ghi.

## Một số mẫu bảng

```text
Table: enfyra_user (built-in, but as example)
- id (uuid, generated)
- email (varchar, unique)
- name (varchar)
- isActive (boolean, default: true)
- createdAt (timestamp, default: now)
```

```text
Table: products
- id (uuid, generated)
- name (varchar)
- description (text)
- price (number)
- categoryId (relation to categories)
- isActive (boolean, default: true)
```

```text
Table: categories (create first)
- id (uuid, generated)
- name (varchar, unique)
- slug (varchar, unique)

Table: posts (create after categories)
- id (uuid, generated)
- title (varchar)
- content (richtext)
- categoryId (relation to categories)
- publishedAt (timestamp, nullable)
```

## Xử lý sự cố

- **Không tạo được relation:** kiểm tra target table đã tồn tại và đã chọn đúng bảng.
- **Lỗi unique constraint:** thêm cột trước, bảo đảm đã chọn đúng constraint field.
- **Tạo index thất bại:** thêm cột/relation trước và tránh index field có unique constraint.
- **Tạo bảng thất bại:** kiểm tra tên bảng không trùng, field bắt buộc đủ và relation hợp lệ.

## Tài liệu liên quan

- [Data Management](./data-management.md)
- [Schema migration preview](../server/schema-migration-preview.md)
- [Relation Picker System](../app/relation-picker.md)
- [Filter System](../app/filter-system.md)
- [Form System](../app/form-system.md)
- [Query Filtering](../server/query-filtering.md)
