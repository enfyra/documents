---
slug: ung-dung/he-thong-bieu-mau
---

# Hệ thống biểu mẫu

Hệ thống biểu mẫu của Enfyra tự động tạo biểu mẫu từ lược đồ cơ sở dữ liệu của bạn. Các biểu mẫu rất năng động, được xác thực và được tích hợp đầy đủ với các quyền và quan hệ.

## Biểu mẫu hoạt động như thế nào

Khi bạn tạo bảng trong Enfyra, hệ thống sẽ tự động tạo biểu mẫu để tạo và chỉnh sửa bản ghi. Những hình thức này:

- Hiển thị các loại đầu vào phù hợp dựa trên các loại trường
- Xác thực dữ liệu theo quy tắc lược đồ
- Xử lý quan hệ giữa các bảng
- Tôn trọng quyền của người dùng

## Loại trường và thành phần đầu vào

### Các loại trường cơ bản

| Loại trường | Thành phần đầu vào | Mô tả |
| ------------------ | --------------- | ----------------------------------------- |
| **varchar/chuỗi** | Nhập văn bản | Trường văn bản một dòng |
| **văn bản** | Vùng văn bản | Vùng văn bản nhiều dòng |
| **int/số** | Nhập số | Đầu vào số có xác nhận |
| **boolean** | Công tắc chuyển đổi | Bật/tắt chuyển đổi |
| **ngày/dấu thời gian** | Công cụ chọn ngày | Công cụ chọn ngày theo lịch |
| **uuid** | trường UUID | ID duy nhất được tạo tự động bằng nút sao chép |

### Loại trường nâng cao

| Loại trường | Thành phần đầu vào | Mô tả |
| ---------------- | ---------------- | ------------------------------------ |
| **văn bản phong phú** | Trình soạn thảo văn bản phong phú | Trình soạn thảo WYSIWYG với các công cụ định dạng (cấu hình qua `column.metadata.richText`) |
| **mã** | Trình chỉnh sửa mã | Nhập mã được tô sáng theo cú pháp |
| **đơn giản-json** | Trình soạn thảo JSON | Đầu vào JSON có xác thực |
| **enum** | Thả xuống | Lựa chọn duy nhất từ ​​​​các tùy chọn |
| **chọn mảng** | Chọn nhiều | Nhiều lựa chọn từ các tùy chọn |

### Trình soạn thảo văn bản đa dạng thức (`column.metadata.richText`)

Với cột `richtext`, cấu hình editor nằm tại `column.metadata.richText` trong schema bảng. Bạn có thể dùng cùng một cấu trúc trong ô JSON metadata của column hoặc trong các giá trị mặc định ở `enfyra.config.ts`. Metadata của column phải là JSON-safe; callback và hàm resolver theo theme chỉ dùng được trong cấu hình cấp source của app.

**Ví dụ column metadata:**
```json
{
  "richText": {
    "plugins": ["link", "lists", "code", "table"],
    "toolbar": "clear | h1 h2 h3 | bold italic underline | bullist numlist | link image table blockquote hr codeblock",
    "customButtons": [
      { "name": "callout", "text": "Callout", "tooltip": "Insert a callout", "format": "callout" }
    ],
    "formats": {
      "callout": {
        "wrapper": true,
        "tag": "aside",
        "classes": "callout",
        "css": {
          "backgroundColor": "var(--surface-muted)",
          "borderLeft": "4px solid var(--primary-500)",
          "padding": "12px"
        }
      }
    }
  }
}
```
#### Các loại định dạng

| Thuộc tính | Mô tả |
|----------|-------------|
| `inline: true` | Phần tử inline (ví dụ: `<code>`, `<span>`). `tag` mặc định là `span`. |
| `block: true` hoặc bỏ qua | Phần tử block (ví dụ: `<pre>`, `<div>`). Đây là mặc định khi không đặt `inline` hoặc `wrapper`. |
| `wrapper: true` | Node bao quanh chứa các block khác (ví dụ: `<blockquote>`). |
| `tag` | Tên thẻ HTML (ví dụ: `"code"`, `"pre"`). Không bắt buộc; mặc định dùng key của format. |

#### Thuộc tính editor và format

- **`plugins`**: Nhóm tính năng tùy chọn. `link`, `lists`, `code` và `table` bật/tắt extension và nút toolbar tương ứng; nếu bỏ qua thì tất cả nhóm built-in vẫn được bật.
- **`css`**: Các kiểu CSS được chèn dưới dạng biểu định kiểu (không phải nội tuyến). Hỗ trợ:
  - Đối tượng phẳng: áp dụng cho cả chủ đề sáng và tối
  - `{ light: {...}, dark: {...} }`: kiểu theo chủ đề
  - Chức năng: `(theme: 'light' | 'dark') => Record<string, string>`
- **`classes`**: Các lớp CSS cho thẻ (chuỗi, mảng hoặc hàm)
- **`classStyles`**: Style cho các class đã đặt tên, hỗ trợ cùng dạng static hoặc theo theme như `css`
- **`attributes`**: thuộc tính HTML cho phần tử

#### Thanh công cụ

Thanh công cụ mặc định: `clear | h1 h2 h3 h4 h5 h6 | bold italic underline strike | bullist numlist | alignleft aligncenter alignright alignjustify | link image table blockquote hr codeblock`

Đặt `toolbar` để thay thế toolbar mặc định. Thêm nút qua `customButtons`; nếu bỏ qua `toolbar`, tên các nút tùy chỉnh sẽ được nối tự động. Nếu đặt `toolbar` rõ ràng, hãy thêm tên từng nút tùy chỉnh vào chuỗi toolbar. Nút tùy chỉnh có thể dùng `format` để bật/tắt format đã cấu hình (ví dụ `format: "callout"`).

Ở cấu hình cấp app, `buttonActions` có thể ánh xạ tên `onAction` thành callback JavaScript. Cơ chế callback này không áp dụng cho JSON metadata của column.

### Các loại trường đặc biệt

| Loại trường | Thành phần | Mô tả |
| -------------- | --------------------------------------------- | ---------------------- |
| **quan hệ** | [Bộ chọn quan hệ](./relation-picker.md) | Chọn hồ sơ liên quan |
| **quyền** | [Trình tạo quyền](./permission-builder.md) | Cấu hình quy tắc truy cập |

## Bố cục biểu mẫu

### Bố cục lưới đáp ứng

- Biểu mẫu sử dụng lưới 2 cột trên máy tính để bàn, 1 cột trên thiết bị di động
- Các trường có chiều rộng đầy đủ tự động trải rộng trên cả hai cột:
  - Trình soạn thảo văn bản phong phú
  - Trình soạn thảo mã
  - Trình soạn thảo JSON
  - Vùng văn bản dài

### Thứ tự trường

1. Các cột trong bảng thông thường xuất hiện đầu tiên
2. Các trường quan hệ xuất hiện sau các trường cột
3. Đặt hàng tùy chỉnh qua cấu hình

## Xác thực biểu mẫu

### Trường bắt buộc

- Các trường được đánh dấu hoa thị màu đỏ (\*) là bắt buộc
- Quá trình xác thực diễn ra trước khi gửi biểu mẫu
- Xóa thông báo lỗi giải thích những gì còn thiếu

### Quy tắc xác thực

- **Bắt buộc**: Trường không được để trống (dựa trên cài đặt `isNullable`)
- **Loại**: Dữ liệu nhập phải khớp với loại trường (số, ngày, v.v.)
- **Định dạng**: Các định dạng đặc biệt như email, URL (khi được định cấu hình)
- **Tùy chỉnh**: Xác thực logic nghiệp vụ thông qua hook

### Hiển thị lỗi

- Thông báo lỗi cấp trường xuất hiện bên dưới đầu vào
- Tự động xóa lỗi khi sửa
- Thông báo nâng cốc cho các vấn đề ở cấp biểu mẫu

## Làm việc với các mối quan hệ

### Một-một / Nhiều-một

- Lựa chọn duy nhất từ ​​bảng liên quan
- Hiển thị bản ghi đã chọn dưới dạng huy hiệu
- Click để thay đổi lựa chọn

### Một-nhiều / Nhiều-nhiều

- Nhiều lựa chọn từ bảng liên quan
- Hiển thị số lượng hồ sơ đã chọn
- Quản lý lựa chọn theo phương thức

**Xem [Hệ thống chọn quan hệ](./relation-picker.md) để biết cách sử dụng chi tiết**

## Tích hợp quyền

Các biểu mẫu tự động tôn trọng quyền của người dùng:

### Quyền cấp trường

- Các trường chỉ hiển thị nếu người dùng có quyền đọc
- Khả năng chỉnh sửa dựa trên quyền cập nhật
- Tùy chọn xóa yêu cầu quyền xóa

### Quyền cấp biểu mẫu

- Tạo biểu mẫu yêu cầu quyền tạo trên bảng
- Chỉnh sửa biểu mẫu yêu cầu quyền cập nhật
- Nút lưu bị vô hiệu hóa khi không có quyền thích hợp

**Cách thức hoạt động của quyền:**

1. **Xác thực phần cuối** tất cả các quyền đối với lệnh gọi API2. **Kiểm tra giao diện người dùng** quyền trước khi hiển thị các thành phần giao diện người dùng
3. Tùy chọn **Ẩn menu** người dùng không thể truy cập
4. **Các trường **Tắt biểu mẫu** người dùng không thể chỉnh sửa

**Xem [Trình tạo quyền](./permission-builder.md) để biết chi tiết kỹ thuật**

## Tính năng biểu mẫu đặc biệt

### Sao chép giá trị trường

- Bấm vào biểu tượng sao chép bên cạnh trường bất kỳ
- Hữu ích cho việc gỡ lỗi hoặc chia sẻ dữ liệu
- Làm việc với tất cả các loại trường

### Phần giữ chỗ trường

- Gợi ý hữu ích xuất hiện trong các trường trống
- Được cấu hình theo từng trường trong cài đặt bảng
- Hướng dẫn người dùng về đầu vào dự kiến

### Mô tả trường

- Mô tả văn bản đa dạng thức bên dưới nhãn trường
- Cung cấp ngữ cảnh và hướng dẫn
- Đặt cấu hình cột trong bảng

### Giá trị mặc định

- Giá trị điền sẵn cho bản ghi mới
- Giá trị mặc định khác nhau cho mỗi loại trường
- Có thể năng động (như ngày hiện tại)

## Trạng thái biểu mẫu

### Trạng thái tải

- Bộ tải Skeleton trong khi tìm nạp dữ liệu
- Hoạt ảnh tải theo loại cụ thể
- Chuyển tiếp mượt mà khi sẵn sàng

### Chế độ chỉnh sửa

- Thay đổi phát hiện sửa đổi theo dõi
- Nút lưu chỉ bật khi có thay đổi
- Xác nhận về những thay đổi chưa được lưu

## Phát hiện thay đổi biểu mẫu

Enfyra tự động theo dõi các thay đổi của biểu mẫu để cung cấp trải nghiệm người dùng tốt hơn:

### Cách thức hoạt động

1. **Trạng thái ban đầu**: Tải biểu mẫu với dữ liệu gốc
2. **Phát hiện thay đổi**: Bất kỳ sửa đổi nào kích hoạt việc theo dõi thay đổi
3. **Nút Lưu**: Chỉ được bật khi phát hiện thấy thay đổi
4. **Đặt lại**: Đặt lại các thay đổi sau khi lưu thành công

### Mẫu triển khai
```typescript
const { register: registerHeaderActions } = useHeaderActionRegistry();
// In your page component
const hasFormChanges = ref(false);
const formEditorRef = ref();

// Template
<FormEditor
  ref="formEditorRef"
  v-model="form"
  v-model:errors="errors"
  @has-changed="(hasChanged) => hasFormChanges = hasChanged"
  table-name="enfyra_user"
  :loading="loading"
/>

// Save function
async function save() {
  // ... save logic
  await updateUser({ body: form.value });
  
  // Reset form changes after successful save
  formEditorRef.value?.confirmChanges();
}

// Header action
registerHeaderActions([
  {
    id: "save-user",
    label: "Save",
    disabled: computed(() => !hasFormChanges.value),
    submit: save,
  },
]);
```
### Thành phần chính

- **`hasFormChanges`**: Trạng thái biểu mẫu theo dõi boolean phản ứng
- **`formEditorRef`**: Tham chiếu tới thành phần FormEditor
- **`@has-changed`**: Sự kiện phát sinh khi trạng thái biểu mẫu thay đổi
- **`confirmChanges()`**: Phương thức reset phát hiện thay đổi

### Lợi ích

- **Trải nghiệm tốt hơn**: Nút lưu chỉ xuất hiện khi cần
- **An toàn dữ liệu**: Ngăn chặn việc lưu ngẫu nhiên
- **Xóa phản hồi**: Người dùng biết khi nào có thay đổi
- **Tự động đặt lại**: Không cần quản lý trạng thái thủ công

## Mẫu nút đặt lại

Enfyra cung cấp mẫu nút đặt lại nhất quán để loại bỏ các thay đổi về biểu mẫu:

### Cách thức hoạt động

1. **Phát hiện thay đổi**: Theo dõi khi dữ liệu biểu mẫu được sửa đổi
2. **Nút đặt lại**: Chỉ xuất hiện khi có thay đổi
3. **Xác nhận**: Yêu cầu người dùng xác nhận trước khi loại bỏ
4. **Khôi phục**: Trả biểu mẫu về trạng thái ban đầu

### Mẫu triển khai
```typescript
const { register: registerHeaderActions } = useHeaderActionRegistry();
// In your page component
const { confirm } = useConfirm();
const hasFormChanges = ref(false);
const formEditorRef = ref();

// Reset function
async function handleReset() {
  const ok = await confirm({
    title: "Discard Changes",
    content: "Are you sure you want to discard all changes? This action cannot be undone.",
  });
  
  if (ok) {
    // Reset form to original state
    form.value = formChanges.discardChanges(form.value);
    formEditorRef.value?.confirmChanges();
  }
}

// Header actions
registerHeaderActions([
  {
    id: "save-user",
    label: "Save",
    disabled: computed(() => !hasFormChanges.value),
    submit: save,
  },
  {
    id: "reset-user",
    label: "Reset",
    variant: "outline",
    color: "gray",
    side: "left",
    show: computed(() => hasFormChanges.value),
    onClick: handleReset,
  },
]);
```
### Sử dụng mẫu useSchema

Đối với các biểu mẫu sử dụng `useSchema`, bạn có thể tận dụng chức năng `discardChanges` tích hợp sẵn:
```typescript
// Using useSchema composable
const { formChanges } = useSchema(tableName);

// Initialize form changes tracking
onMounted(async () => {
  const data = await fetchData();
  form.value = data;
  formChanges.update(data); // Track original state
});

// Reset function
async function handleReset() {
  const ok = await confirm({
    title: "Discard Changes",
    content: "Are you sure you want to discard all changes?",
  });
  
  if (ok) {
    // Use useSchema's discardChanges
    form.value = formChanges.discardChanges(form.value);
    formEditorRef.value?.confirmChanges();
  }
}
```
### Các tính năng chính

- **Hiển thị có điều kiện**: Nút đặt lại chỉ hiển thị khi có thay đổi
- **Hộp thoại xác nhận**: Ngăn ngừa mất dữ liệu do tai nạn
- **Thiết lập lại sâu**: Khôi phục tất cả các trường biểu mẫu về giá trị ban đầu
- **Đồng bộ hóa trạng thái**: Cập nhật trạng thái phát hiện thay đổi biểu mẫu
- **Trải nghiệm người dùng nhất quán**: Mẫu giống nhau trên tất cả các biểu mẫu

### Các phương pháp hay nhất

1. **Luôn xác nhận**: Sử dụng hộp thoại xác nhận cho các hành động phá hoại
2. **Xóa tin nhắn**: Giải thích những gì sẽ bị mất
3. **Định vị**: Đặt nút đặt lại ở phía bên trái
4. **Quản lý trạng thái**: Luôn gọi `confirmChanges()` sau khi đặt lại

### Trường bị vô hiệu hóa

- Không thể chỉnh sửa các trường hệ thống
- Các trường được tạo ở dạng chỉ đọc
- Vô hiệu hóa có điều kiện thông qua cấu hình

## Gửi biểu mẫu

### Tạo luồng

1. Điền vào các trường bắt buộc
2. Quá trình xác thực diễn ra tự động
3. Nhấp vào "Tạo" để lưu
4. Thông báo và chuyển hướng thành công

### Luồng cập nhật

1. Sửa đổi các giá trị hiện có
2. Những thay đổi được theo dõi tự động
3. Nhấp vào "Lưu" khi sẵn sàng
4. Phản hồi tức thì về thành công

### Xử lý lỗi

- Lỗi API hiển thị dưới dạng thông báo
- Lỗi trường làm nổi bật đầu vào cụ thể
- Khả năng thử lại cho các sự cố mạng

## Cấu hình nâng cao

### Hệ thống FieldMap

Prop `fieldMap` cho phép bạn tùy chỉnh hành vi biểu mẫu cho các trường cụ thể mà không cần sửa đổi lược đồ cơ sở dữ liệu. Chuyển nó tới **`FormEditor`** hoặc **`FormEditorLazy`** (cùng một đạo cụ; lười biếng trì hoãn tải gói trình soạn thảo nặng - sử dụng trong ngăn kéo và chế độ xem phụ).
```typescript
<FormEditorLazy
  v-model="form"
  :table-name="tableName"
  :field-map="fieldMap"
/>
```
#### Phần (các trường được nhóm)

Truyền **`sections`** để hiển thị các trường trong các khối được đặt tên. Trường **thứ tự trong một phần** theo sau mảng `fields`. Bất kỳ trường lược đồ nào không được liệt kê trong một phần sẽ được hiển thị sau các phần đó (vẫn được sắp xếp theo quy tắc lược đồ mặc định).
```typescript
const sections = [
  {
    id: 'main',
    title: 'General',
    fields: ['name', 'slug', 'description'],
  },
  {
    id: 'meta',
    title: 'Metadata',
    hideHeading: true,
    class: 'surface-card rounded-xl p-4 ring-1 ring-[var(--surface-panel-ring)]',
    fields: ['createdAt'],
  },
];
```
| Trường phần | Mô tả |
|---------------|-------------|
| `id` | Chìa khóa ổn định cho khối |
| `tiêu đề` | Tiêu đề tùy chọn phía trên khối |
| `ẩnTiêu đề` | Nếu đúng, `title` không được hiển thị |
| `headingClass` | Ghi đè kiểu chữ tiêu đề |
| `đẳng cấp` | Bao quanh lưới trường của phần (ví dụ: bảng thẻ) |
| `rootClass` | Lớp bổ sung ở khối phần bên ngoài |
| `trường` | Danh sách tên trường theo thứ tự |

#### Cách sử dụng FieldMap cơ bản
```typescript
// Example: Change field type
const fieldMap = computed(() => ({
  description: {
    type: 'richtext'  // Make description a rich text field
  }
}));

// Example: Span full width on desktop grid (layout="grid" uses md:grid-cols-2)
const fieldMap = computed(() => ({
  content: {
    fieldProps: {
      class: 'md:col-span-2'
    }
  }
}));

// Example: Custom placeholder
const fieldMap = computed(() => ({
  email: {
    placeholder: 'user@example.com'
  }
}));

// Example: Disable a field
const fieldMap = computed(() => ({
  type: {
    disabled: true  // Field cannot be edited
  }
}));
```
#### Lọc các tùy chọn Enum

Đối với các trường enum và chọn mảng, bạn có thể lọc các tùy chọn xuất hiện trong danh sách thả xuống:

**Loại trừ tùy chọn:**
```typescript
// Hide specific options from enum dropdown
const fieldMap = computed(() => ({
  storageType: {
    excludedOptions: ['Local Storage', 'Legacy System']
  }
}));
```
**Chỉ bao gồm các tùy chọn cụ thể:**
```typescript
// Show only specific options in enum dropdown
const fieldMap = computed(() => ({
  status: {
    includedOptions: ['Active', 'Pending', 'Review']
  }
}));
```
**Ví dụ đầy đủ - Cấu hình lưu trữ:**
```typescript
// In storage config create page
const fieldMap = computed(() => ({
  type: {
    excludedOptions: ['Local Storage']  // Hide Local Storage from create form
  }
}));

// In storage config detail page
const fieldMap = computed(() => ({
  type: {
    disabled: true,  // Cannot change type after creation
    excludedOptions: ['Local Storage']  // Still hide it
  }
}));
```
#### Quyền cấp trường

Bạn có thể kiểm soát mức độ hiển thị của trường bằng tùy chọn `permission` trong fieldMap. Các trường được tự động hiển thị nếu không có quyền nào được chỉ định hoặc chỉ hiển thị khi người dùng có quyền được yêu cầu:
```typescript
// Example: Show field only to users with specific permission
const fieldMap = computed(() => ({
  adminNotes: {
    permission: {
      and: [
        { route: '/enfyra_user', methods: ['PATCH'] }
      ]
    }
  }
}));

// Example: Show field with OR condition
const fieldMap = computed(() => ({
  sensitiveData: {
    permission: {
      or: [
        { route: '/admin_panel', methods: ['GET'] },
        { route: '/reports', methods: ['GET'] }
      ]
    }
  }
}));
```
**Lưu ý:** Nếu `quyền` không được chỉ định, trường này sẽ tự động được hiển thị cho tất cả người dùng. Điều này cho phép bạn hạn chế có chọn lọc các trường mà không ảnh hưởng đến những trường khác.

#### Tham khảo tùy chọn FieldMap

| Tùy chọn | Loại | Mô tả |
|--------|------|-------------|
| `loại` | `chuỗi` | Ghi đè loại đầu vào của trường (richtext, mã, v.v.) |
| `bị vô hiệu hóa` | `boolean` | Tạo trường chỉ đọc |
| `giữ chỗ` | `chuỗi` | Văn bản giữ chỗ tùy chỉnh |
| `quyền` | `Điều kiện cấp phép` | Kiểm soát mức độ hiển thị của trường dựa trên quyền của người dùng (xem [Trình tạo quyền](./permission-builder.md)) |
| `tùy chọn loại trừ` | `chuỗi[]` | Ẩn các tùy chọn này khỏi danh sách thả xuống enum/array-select |
| `tùy chọn được bao gồm` | `chuỗi[]` | Chỉ hiển thị các tùy chọn này trong danh sách thả xuống enum/array-select |
| `fieldProps` | `đối tượng` | Đã chuyển đến hàng trường; sử dụng `class` cho các nhịp lưới (ví dụ: `md:col-span-2` khi `layout="grid"`) |
| `booleanWrapperClass` | `chuỗi` | Thay thế các lớp trình bao bọc flex mặc định cho các trường **boolean** (nhãn + hàng chuyển đổi) |
| `fieldWrapperClass` | `chuỗi` | Các lớp bổ sung ở trình bao bọc bên ngoài cho các trường **không phải boolean** |
| `thành phầnProps` | `đối tượng` | sáp nhập vào kiểm soát; `class` được sáp nhập với các lớp kiểm soát nội bộ |
| `nhãn` / `mô tả` | `chuỗi` | Ghi đè nhãn cột / trợ giúp HTML |
| `hideLabel` / `hideDescription` | `boolean` | Ẩn nhãn hoặc mô tả |
| `thành phần` | `Thành phần` | Thành phần đầu vào tùy chỉnh (`modelValue` / `update:modelValue`) |
| `tùy chọn` | `bất kỳ[]` | Ghi đè tất cả các tùy chọn (sử dụng với các tùy chọn loại trừ/bao gồmTùy chọn để lọc) |

**Lưu ý:** `excludedOptions` và `includedOptions` hoạt động với các tùy chọn enum ban đầu từ lược đồ của bạn. Họ lọc các tùy chọn, không thay thế chúng.

### Hiển thị trường

- Bao gồm/loại trừ các trường cụ thể
- Khả năng hiển thị có điều kiện dựa trên các giá trị
- Hiển thị trường dựa trên vai trò

## Tích hợp với các hành động tiêu đề

### Nút Lưu

- Xuất hiện ở tiêu đề khi chỉnh sửa
- Vô hiệu hóa khi không có thay đổi
- Hiển thị trạng thái tải trong khi lưu

### Nút Xóa

- Có sẵn khi xem bản ghi hiện có
- Yêu cầu quyền xóa
- Hộp thoại xác nhận ngăn ngừa tai nạn

### Hành động tùy chỉnh

- Thêm các nút tùy chỉnh thông qua cấu hình
- Tích hợp với dữ liệu biểu mẫu
- Khả năng hiển thị được kiểm soát bằng quyền

## Các phương pháp hay nhất

### Thiết kế biểu mẫu

- Giữ các hình thức tập trung và ngắn gọn
- Nhóm các lĩnh vực liên quan lại với nhau
- Sử dụng mô tả cho các trường phức tạp
- Cung cấp phần giữ chỗ hữu ích

### Xác thực

- Xác nhận sớm và thường xuyên
- Cung cấp thông báo lỗi rõ ràng
- Sử dụng xác thực frontend và backend
- Xem xét trải nghiệm người dùng

### Hiệu suất

- Các thành phần nặng tải lười biếng
- Sử dụng phân trang cho bộ chọn quan hệ
- Tối ưu hóa logic xác nhận
- Lược đồ bộ đệm khi có thể

## Các tình huống phổ biến

### Biểu mẫu nhiều bước

Mặc dù Enfyra sử dụng các biểu mẫu một trang theo mặc định, bạn có thể:

- Sử dụng móc để tạo luồng kiểu thuật sĩ
- Có điều kiện hiển thị/ẩn các phần
- Lưu bản nháp giữa các bước

### Trường phụ thuộc

- Sử dụng hook để cập nhật các trường dựa trên các trường khác
- Tự động tính toán giá trị
- Xác nhận kết hợp trường

### Tải lên tệp

- Các trường tệp tích hợp với hệ thống tệp
- Hỗ trợ kéo và thả
- Chỉ số tiến trình tải lên

## Khắc phục sự cố

### Biểu mẫu không tải

- Kiểm tra lược đồ bảng tồn tại
- Xác minh người dùng có quyền đọc
- Đảm bảo điểm cuối API có thể truy cập được

### Lỗi xác thực

- Xem xét yêu cầu của trường
- Kiểm tra lược đồ phù hợp với kiểu dữ liệu
- Xác minh logic xác thực tùy chỉnh

### Lưu lỗi

- Xác nhận người dùng có quyền cập nhật
- Kiểm tra kết nối mạng- Xem lại thông báo lỗi API

## Bản đồ trường (Cấu hình trường tùy chỉnh)

Sử dụng prop `:field-map` trên `FormEditorLazy` để tùy chỉnh các trường riêng lẻ.
```vue
<FormEditorLazy
  v-model="form"
  :table-name="'my_table'"
  :field-map="{
    name: { label: 'Full Name', description: 'Enter your full name' },
    status: { hideDescription: true },
    config: { component: MyCustomEditor, componentProps: { lang: 'json' } },
    secret: { hideLabel: true },
  }"
/>
```
### Tùy chọn cấu hình được hỗ trợ

| Tùy chọn | Loại | Mô tả |
|--------|------|-------------|
| `nhãn` | chuỗi | Ghi đè nhãn trường |
| `mô tả` | chuỗi | Ghi đè văn bản mô tả trường (bên dưới đầu vào) |
| `ẩnNhãn` | boolean | Ẩn hoàn toàn phần nhãn + mô tả |
| `ẩnMô tả` | boolean | Chỉ ẩn mô tả, giữ nhãn |
| `thành phần` | chuỗi \| Thành phần | Thay thế trường bằng thành phần tùy chỉnh (nhận `v-model`) |
| `thành phầnProps` | đối tượng | Đạo cụ bổ sung được chuyển đến thành phần tùy chỉnh |
| `loại` | chuỗi | Ghi đè loại trường (ví dụ: `'code'`, `'method-selector'`) |
| `bị vô hiệu hóa` | boolean | Vô hiệu hóa trường |
| `giữ chỗ` | chuỗi | Ghi đè văn bản giữ chỗ |
| `quyền` | đối tượng | Điều kiện cho phép hiển thị/ẩn trường |
| `fieldProps` | đối tượng | Đạo cụ bổ sung trên phần tử bao bọc |

### Ví dụ về thành phần tùy chỉnh

Tạo một thành phần chấp nhận `modelValue` + phát ra `update:modelValue`:
```vue
<template>
  <div>
    <UInput :model-value="modelValue" @update:model-value="$emit('update:modelValue', $event)" />
    <p>Custom UI here</p>
  </div>
</template>

<script setup>
defineProps(['modelValue']);
defineEmits(['update:modelValue']);
</script>
```
Sau đó sử dụng trong bản đồ trường:
```javascript
const MyEditor = resolveComponent('MyCustomEditor');

const fieldMap = {
  myField: { component: MyEditor, componentProps: { extra: 'prop' } }
};
```
Hệ thống biểu mẫu cung cấp mọi thứ cần thiết để nhập dữ liệu trong khi vẫn duy trì tính bảo mật, xác thực và trải nghiệm người dùng tuyệt vời.
