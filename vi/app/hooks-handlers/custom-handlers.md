---
slug: ung-dung/hook-va-handler/custom-handler
---

# Trình xử lý tùy chỉnh

Trình xử lý tùy chỉnh cho phép bạn thay thế các thao tác CRUD mặc định bằng mã JavaScript của riêng bạn. Thay vì hành vi tạo/đọc/cập nhật/xóa cơ bản, bạn có thể viết các hàm tùy chỉnh xử lý logic nghiệp vụ phức tạp, tích hợp bên ngoài hoặc xử lý dữ liệu chuyên dụng.

**Để hiểu đầy đủ về vòng đời yêu cầu, hãy xem [Vòng đời API](../../server/api-lifecycle.md)**

> **Lưu ý cú pháp mẫu**: Tất cả các ví dụ đều sử dụng cú pháp `$ctx.$property` truyền thống, nhưng bạn cũng có thể sử dụng cú pháp mẫu ngắn hơn (`@BODY`, `@REPOS`, `#table_name`). Xem [Hướng dẫn cú pháp mẫu](../../server/template-syntax.md) để biết chi tiết. Cả hai đều hoạt động giống hệt nhau và có thể được trộn lẫn một cách tự do.

## Khi nào nên sử dụng Trình xử lý tùy chỉnh

- **Logic nghiệp vụ phức tạp**: Thực hiện các thao tác nhiều bước trên nhiều bảng
- **Xác thực dữ liệu**: Thêm quy tắc xác thực tùy chỉnh ngoài các ràng buộc lược đồ cơ bản
- **Tích hợp bên ngoài**: Gọi API của bên thứ ba, gửi email hoặc kích hoạt webhook
- **Chuyển đổi dữ liệu**: Xử lý hoặc chuyển đổi dữ liệu trước khi lưu vào cơ sở dữ liệu
- **Phản hồi tùy chỉnh**: Trả về các định dạng dữ liệu chuyên dụng hoặc giá trị được tính toán

## Tạo trình xử lý tùy chỉnh

### Bước 1: Quản lý bộ xử lý truy cập
1. Điều hướng đến **Cài đặt > Tuyến đường**
2. Mở tuyến đường cần logic tùy chỉnh
3. Trong **Luồng thực thi**, nhấp vào trình xử lý mặc định cho một phương thức hoặc nhấp vào **Thêm trình xử lý**

### Bước 2: Cấu hình Trình xử lý
Bạn sẽ thấy biểu mẫu tạo trình xử lý với các trường sau:

- **Logic**: Trình chỉnh sửa mã lớn nơi bạn viết JavaScript tùy chỉnh của mình
- **Mô tả**: Vùng văn bản để ghi lại những gì trình xử lý này thực hiện  
- **Tuyến đường**: Tuyến đường hiện tại được chọn tự động khi bạn tạo trình xử lý từ trang chi tiết tuyến đường
- **Phương thức**: Chọn bản ghi phương thức HTTP. Các phương thức tích hợp sẵn là `GET`, `POST`, `PATCH` và `DELETE`; tạo các phương thức tùy chỉnh trong **Cài đặt > Phương thức** trước khi gán chúng cho trình xử lý.

### Bước 3: Link tới Route và Phương thức
- **Lựa chọn phương thức**: Sử dụng bộ chọn quan hệ để chọn từ các phương thức HTTP có sẵn
- **Ràng buộc duy nhất**: Mỗi tổ hợp tuyến đường+phương thức chỉ có thể có một trình xử lý

### Bước 4: Thực thi trình xử lý
Khi một yêu cầu khớp với tuyến đường và phương thức, mã xử lý tùy chỉnh của bạn sẽ thực thi thay vì thao tác CRUD mặc định.

**Quan trọng: Giá trị trả về**  
Trình xử lý của bạn **PHẢI trả về một giá trị** - giá trị này sẽ trở thành phản hồi API. Nếu bạn không trả lại bất cứ thứ gì thì API sẽ không trả lại gì cho khách hàng.

## Bối cảnh xử lý ($ctx)

Mã xử lý của bạn nhận được một đối tượng ngữ cảnh phong phú `$ctx` với mọi thứ bạn cần.

**Để tham khảo ngữ cảnh đầy đủ, hãy xem [Tham khảo ngữ cảnh](../../server/context-reference/README.md)**

## Phương thức lưu trữ cơ sở dữ liệu

Mỗi kho lưu trữ trong `$ctx.$repos` cung cấp quyền truy cập cơ sở dữ liệu đầy đủ thông qua QueryEngine.

**Để biết các ví dụ và thao tác cơ sở dữ liệu đầy đủ, hãy xem [Tham chiếu repository](../../server/context-reference/repositories.md#truy-cap-repository)**

## Trình xử lý ví dụ

### Tạo sản phẩm tùy chỉnh
```javascript
// POST /products handler
// Target Tables: products, categories, audit_logs

// Validate category exists
const categoryResult = await $ctx.$repos.categories.find({
  filter: { id: { _eq: $ctx.$body.categoryId } },
  fields: 'id,name' // Only fetch required fields
});

if (!categoryResult.data.length) {
  throw new Error('Category not found');
}

// Create product with auto-generated slug
const slug = $ctx.$helpers.autoSlug($ctx.$body.name);
const productResult = await $ctx.$repos.products.create({
  data: {
    name: $ctx.$body.name,
    slug: slug,
    price: $ctx.$body.price,
    categoryId: $ctx.$body.categoryId,
    createdBy: $ctx.$user.id
  }
});

const newProduct = productResult.data[0];

// Log the creation
await $ctx.$repos.audit_logs.create({ data: {
  action: 'product_created',
  userId: $ctx.$user.id,
  entityId: newProduct.id,
  details: JSON.stringify({ productName: newProduct.name })
}});

$ctx.$logs(`Product created: ${newProduct.name}`);

return {
  success: true,
  product: newProduct,
  logs: $ctx.$share.$logs
};
```
### Xác thực người dùng
```javascript
// POST /auth/login handler  
// Target Tables: enfyra_user

const { email, password } = $ctx.$body;

// Find user by email
const userResult = await $ctx.$repos.enfyra_user.find({
  filter: { email: { _eq: email } },
  fields: 'id,email,password' // Only fetch authentication fields
});

if (!userResult.data.length) {
  throw new Error('User not found');
}

const user = userResult.data[0];

// Verify password
const validPassword = await $ctx.$helpers.$bcrypt.compare(
  password, 
  user.password
);

if (!validPassword) {
  throw new Error('Invalid password');
}

// Generate JWT token
const token = await $ctx.$helpers.$jwt(
  { userId: user.id, email: user.email },
  '7d'
);

// Update last login
await $ctx.$repos.enfyra_user.update({ id: user.id, data: {
  lastLoginAt: new Date()
} });

return {
  success: true,
  token: token,
  user: {
    id: user.id,
    email: user.email,
    name: user.name
  }
};
```
### Lọc nâng cao với hệ thống lọc API
```javascript
// GET /products/advanced-search handler  
// Target Tables: products, categories, users
// Demonstrates the full power of API Filtering in handlers

const result = await $ctx.$repos.products.find({
  filter: {
    // Logical operators
    _or: [
      {
        _and: [
          { price: { _between: [100, 500] } },    // Price range
          { category: { name: { _contains: 'electronics' } } }, // Relation filtering
          { stock: { _gt: 0 } }                   // In stock
        ]
      },
      {
        _and: [
          { featured: { _eq: true } },            // Featured products
          { rating: { _gte: 4.5 } },             // High rating
          { reviews: { _count: { _gte: 10 } } }   // Aggregation: min 10 reviews
        ]
      }
    ],
    // Text search across multiple fields
    _or: [
      { name: { _contains: $ctx.$query.search || '' } },
      { description: { _contains: $ctx.$query.search || '' } },
      { tags: { _contains: $ctx.$query.search || '' } }
    ],
    // Exclude discontinued products
    status: { _not_in: ['discontinued', 'out-of-stock'] },
    // Only published products
    publishedAt: { _is_null: false }
  }
});

// Use aggregation filtering to get categories with products count
const categoriesWithCount = await $ctx.$repos.categories.find({
  filter: {
    products: { _count: { _gt: 0 } }  // Categories that have products
  },
  fields: 'id,name,description' // Only fetch essential category info
});

return {
  products: result.data,
  categories: categoriesWithCount.data,
  meta: {
    ...result.meta,
    searchTerm: $ctx.$query.search,
    appliedFilters: 'advanced-filtering-demo'
  }
};
```
### Xử lý dữ liệu phức tạp
```javascript
// GET /reports/sales handler
// Target Tables: orders, products, users

const { startDate, endDate, category } = $ctx.$query;

// Advanced filtering using API Filtering operators
const ordersResult = await $ctx.$repos.orders.find({
  filter: {
    _and: [
      // Date range
      { createdAt: { _between: [startDate, endDate] } },
      // Only completed orders
      { status: { _in: ['completed', 'shipped', 'delivered'] } },
      // Filter by category if provided
      ...(category ? [{ 
        product: { 
          category: { name: { _eq: category } } 
        } 
      }] : []),
      // Minimum order value
      { total: { _gte: 10 } }
    ]
  }
});

// Get high-value customers using aggregation
const vipCustomers = await $ctx.$repos.users.find({
  filter: {
    orders: {
      _sum: { total: { _gte: 1000 } }  // Customers with $1000+ total orders
    }
  }
});

return {
  summary: {
    totalRevenue: ordersResult.data.reduce((sum, order) => sum + order.total, 0),
    totalOrders: ordersResult.data.length,
    vipCustomerCount: vipCustomers.data.length
  },
  orders: ordersResult.data,
  meta: ordersResult.meta
};
```
## Các phương pháp hay nhất

### Tạo trình xử lý
- **Truy cập qua Cài đặt > Tuyến**: Mở một tuyến và sử dụng phần Luồng thực thi của nó
- **Phạm vi tuyến đường và phương thức**: Tạo trình xử lý từ tuyến đường và phương thức mà nó sẽ thay thế
- **Kết hợp độc đáo**: Hãy nhớ rằng mỗi tuyến đường+phương thức chỉ có thể có một trình xử lý
- **Giá trị trả về**: Luôn trả về thứ gì đó - điều này trở thành phản hồi API

### Cấu hình tuyến đường  
- **Bảng mục tiêu**: Định cấu hình các bảng trong trường Bảng mục tiêu của tuyến để truy cập chúng thông qua `$ctx.$repos`
- **Tên mô tả**: Sử dụng mô tả trình xử lý rõ ràng để bảo trì
- **Tính đặc hiệu của phương thức**: Tạo các trình xử lý riêng cho các phương thức HTTP khác nhau

### Mẫu mã
- **Luôn chờ**: Tất cả các hàm `$ctx.$helpers` và `$ctx.$cache` đều yêu cầu `await` vì chúng là các hoạt động không đồng bộ
- **Trích xuất kết quả**: Lưu trữ kết quả trợ giúp trong các biến trước khi sử dụng chúng
- **Kiểm tra mảng dữ liệu**: Các phương thức lưu trữ trả về `{data: []}`, luôn kiểm tra `data.length`

### Xử lý lỗi
- **Sử dụng phương thức $throw**: Sử dụng `$ctx.$throw['400']()` thay vì `throw new Error()` để xử lý lỗi nhất quán
- **Mã trạng thái HTTP**: Sử dụng các phương thức số như `$ctx.$throw['404']()` cho các lỗi HTTP tiêu chuẩn
- **Lỗi ngữ nghĩa**: Sử dụng các phương thức mô tả như `$ctx.$throw.businessLogic()` cho các lỗi logic nghiệp vụ
- **Phục hồi lỗi**: Kiểm tra `$ctx.$api.error` trong postHook để xử lý lỗi một cách khéo léo (chỉ có trong postHook)
- **Thông báo mô tả**: Cung cấp thông báo lỗi rõ ràng và chi tiết để gỡ lỗi
- **Mã trạng thái**: Lỗi tự động trả về mã trạng thái HTTP thích hợp

### Hiệu suất
- **Truy vấn tối thiểu**: Chỉ tìm nạp dữ liệu bạn cần
- **Thao tác hàng loạt**: Sử dụng các thao tác hàng loạt cho nhiều bản ghi
- **Tránh N+1**: Chú ý đến các truy vấn trong vòng lặp

### Bảo mật
- **Ngữ cảnh của người dùng**: Luôn kiểm tra `$ctx.$user` để xác thực
- **Xác thực đầu vào**: Xác thực tất cả dữ liệu `$ctx.$body` và `$ctx.$query`
- **Vệ sinh**: Làm sạch đầu vào của người dùng trước khi vận hành cơ sở dữ liệu

### Ghi nhật ký
- **Thông tin gỡ lỗi**: Sử dụng `$ctx.$logs()` để khắc phục sự cố
- **Dấu vết kiểm tra**: Ghi lại các hoạt động kinh doanh quan trọng
- **Theo dõi hiệu suất**: Ghi nhật ký thời gian thực hiện để tối ưu hóa

**Để biết thêm về thao tác cache và lọc API, hãy xem [Helpers và cache](../../server/context-reference/helpers-cache.md#cache) và [Lọc truy vấn](../../server/query-filtering.md#thuc-hanh-tot)**

Trình xử lý tùy chỉnh mang đến sự linh hoạt với khả năng truy cập ngữ cảnh phong phú. Quá trình thực thi diễn ra thông qua nhóm công nhân vm biệt lập của Enfyra; Các cuộc gọi gói thời gian chạy nút được ủy quyền bên ngoài vùng cô lập. Hãy coi các tác giả xử lý như những cộng tác viên đáng tin cậy của dự án và sử dụng RBAC cùng với việc tăng cường độ cứng của máy chủ đối với mã không đáng tin cậy.

## Tài liệu liên quan

- **[Tham chiếu bối cảnh](../../server/context-reference/README.md)** - Hoàn thành tham chiếu đối tượng $ctx
- **[Xử lý tệp](../../server/file-handling.md)** - Hướng dẫn truyền tải tệp lên và phản hồi
- **[Hooks](./hooks.md)** - Móc yêu cầu/phản hồi nhẹ để tùy chỉnh đơn giản
- **[Hooks and Handlers](../../server/hooks-handlers/)** - Lập trình hook nâng cao kèm ví dụ
- **[Vòng đời API](../../server/api-lifecycle.md)** - Hoàn thiện quy trình xử lý yêu cầu
- **[Quản lý định tuyến](../routing-management.md)** - Hướng dẫn giao diện người dùng để tạo điểm cuối tùy chỉnh

## Ví dụ thực tế

- **[Ví dụ đăng ký người dùng](../../examples/user-registration-example.md)** - Hướng dẫn đầy đủ về cách tạo điểm cuối `/register` với xác thực, băm mật khẩu và email chào mừng
