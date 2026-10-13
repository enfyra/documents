---
slug: tim-ban-ghi
---

# Tìm bản ghi

Truy vấn bản ghi từ bảng với lọc, sắp xếp, phân trang và chọn trường.

## Cách dùng cơ bản

```javascript
// Find all records (up to default limit of 10)
const result = await $ctx.$repos.products.find({});

// Access the data
const products = result.data; // Array of product records
```

## Tham số

```javascript
await $ctx.$repos.tableName.find({
  filter: { ... },       // Filter conditions (optional)
  fields: '...',         // Fields to return (optional)
  limit: 10,             // Max records to return (optional, default: 10)
  sort: '...',           // Sort order (optional, default: primary key field)
  meta: 'totalCount'     // Request metadata (optional)
})
```

## findLocked

Dùng `findLocked` khi cần khóa một row đang tồn tại trước các bước repository phụ thuộc. Method nhận `{ id, fields?, deep? }` và trả collection `{ data: [...] }` như các phép đọc thông thường. Phải gọi bên trong `$ctx.$transaction.run()` trên PostgreSQL hoặc MySQL/InnoDB; MongoDB và SQLite không hỗ trợ phép đọc chỉ lấy khóa này. Kiểm tra Server và Kernel đã deploy có method này trước khi sử dụng.

```javascript
return await @TRANSACTION.run(async () => {
  const result = await #secure.products.findLocked({
    id: @PARAMS.id,
    fields: ['id', 'stock']
  });
  const product = result.data[0];
  if (!product) @THROW404('Product not found');
  if (Number(product.stock) < 1) @THROW400('Out of stock');

  const reservation = await #secure.stock_reservations.create({
    data: { product: { id: product.id }, quantity: 1 },
    fields: ['id']
  });
  await #secure.products.update({
    id: product.id,
    data: { stock: Number(product.stock) - 1 },
    fields: ['id']
  });
  return { reservationId: reservation.data[0].id };
});
```

Phép đọc có khóa không cập nhật row, đổi `updatedAt`, invalidate cache hay phát mutation event. Nó đọc trạng thái hiện tại của row gốc kể cả khi transaction MySQL đã có snapshot cũ. Khóa row gốc được giữ đến khi transaction ngoài commit hoặc rollback, kể cả sau khi `findLocked` trả về. Mọi phép đọc, tính toán và ghi phụ thuộc phải nằm trong cùng callback transaction đó. Method báo lỗi khi row gốc không tồn tại hoặc khi gọi ngoài explicit transaction.

Read authorization và field projection vẫn áp dụng. Secure access không thay thế kiểm tra owner/tenant: phải kiểm tra quyền route và phạm vi bản ghi của nghiệp vụ. `fields` và `deep` chọn dữ liệu trả về, không khóa các row liên quan. Method không nhận filter danh sách, pagination hay count metadata.

Với bản ghi chưa tồn tại, khóa một owner chung đang tồn tại trước. Ví dụ, row của user đã xác thực có thể phối hợp việc tìm và tạo balance. Mọi creator hoặc writer cạnh tranh phải khóa cùng owner trước khi đọc hoặc tính toán; vẫn cần database uniqueness ở nơi có writer có thể đi ngoài cơ chế phối hợp đó. Khi khóa nhiều owner, mọi luồng phải lấy khóa theo cùng một thứ tự xác định.

Cache, socket, HTTP và flow trigger nằm ngoài database rollback. Row lock kết thúc khi commit và không sắp thứ tự các lần phát dữ liệu sau đó. Giữ cơ chế sắp thứ tự publication cần thiết hoặc dùng revision/outbox thuộc commit trước khi thay cơ chế phối hợp rộng hơn bằng khóa DB đơn thuần.

Dùng [`updateLocked`](./create-update-delete.md#doc-tinh-va-ghi-an-toan-voi-updatelocked) để tính rồi ghi patch trên một generic row. Dùng `findLocked` khi cần lấy khóa sớm trước nhiều bước repository; không lấy khóa bằng cách ghi lại giá trị cũ hoặc cập nhật timestamp giả.

## Tìm bản ghi

### Get all records (no limit)
```javascript
const result = await $ctx.$repos.products.find({
  limit: 0  // 0 = no limit, fetch all records
});
const allProducts = result.data;
```

### Get limited records
```javascript
const result = await $ctx.$repos.products.find({
  limit: 20  // Return max 20 records
});
```

### Filter records
```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    category: { _eq: 'electronics' },
    price: { _gte: 100 }
  }
});
```

### Lọc với nhiều điều kiện
```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { category: { _eq: 'electronics' } },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } }
    ]
  }
});
```

## Chọn trường

### Trả về các trường xác định
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,price'  // Comma-separated field names
});
```

### Trả về mọi trường (mặc định)
```javascript
const result = await $ctx.$repos.products.find({
  // No fields parameter = return all fields
});
```

### Gồm trường của bảng liên quan
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,category.name,category.description'
});
// Returns: [{id: 1, name: "Phone", category: {name: "Electronics", description: "..."}}]
```

### Gồm mọi trường từ một quan hệ
```javascript
const result = await $ctx.$repos.products.find({
  fields: 'id,name,category.*'  // category.* = all category fields
});
```

### Loại trừ trường
```javascript
const result = await $ctx.$repos.enfyra_route_handler.find({
  fields: '-compiledCode'
});
```

Trường có tiền tố `-` sẽ chuyển phạm vi `fields` sang chế độ loại trừ. Ở chế độ này, tên trường dương bị bỏ qua; vì vậy `fields: 'id,-compiledCode'` trả về mọi trường có thể đọc trừ `compiledCode`.

Loại trừ lồng nhau dùng ký pháp dấu chấm:

```javascript
const result = await $ctx.$repos.posts.find({
  fields: '-author.avatar'
});
```

### Truy vấn deep (quan hệ lồng nhau)

Với truy vấn lồng nhau phức tạp qua nhiều cấp, hãy dùng tham số `deep`:

```javascript
// Fetch products with category, and category's parent category
const result = await $ctx.$repos.products.find({
  fields: 'id,name',
  deep: {
    category: {
      fields: 'id,name',
      deep: {
        parent: {
          fields: 'id,name'
        }
      }
    }
  }
});

// With filter, sort, and pagination
const result = await $ctx.$repos.products.find({
  fields: 'id,name',
  deep: {
    category: {
      fields: 'id,name',
      filter: { isActive: { _eq: true } },
      sort: 'name',
      limit: 10
    }
  }
});
```

Trường deep cũng hỗ trợ chế độ loại trừ ở từng phạm vi lồng nhau:

```javascript
const result = await $ctx.$repos.posts.find({
  fields: 'id,title',
  deep: {
    comments: {
      fields: '-compiledCode,-author.avatar',
      limit: 10,
      deep: {
        author: {
          fields: '-avatar'
        }
      }
    }
  }
});
```

**Để xem hướng dẫn đầy đủ về deep query**, hãy đọc [Hướng dẫn Deep Queries](../query-filtering.md#deep-queries-nested-relations), bao gồm:
- Ví dụ lồng relation nhiều cấp
- Lọc relation lồng nhau
- Sắp xếp và phân trang theo từng cấp
- Thực hành tốt về hiệu năng
- URL query examples

## Aggregate kết quả tính toán

Dùng `aggregate()` khi cần các giá trị đã tính toán thay vì raw record. Đây là phương thức riêng, không phải option của `find()`. Kết quả nằm trong `data`, không trả raw record và không dùng `meta.aggregate`.

### Summary đơn giản

Bỏ qua `dimensions` để nhận một summary row cho tập dữ liệu sau filter:

```javascript
const result = await $ctx.$repos.orders.aggregate({
  filter: {
    status: { _eq: 'paid' }
  },
  measures: {
    orders: { count: 'id' },
    revenue: { sum: 'amount' },
    averageOrderValue: { avg: 'amount' }
  }
});

// result.data[0] = { orders: 42, revenue: 18400, averageOrderValue: 438.1 }
```

Các operation được hỗ trợ là `count`, `countDistinct`, `sum`, `avg`, `min` và `max`. Các operation tính trên số yêu cầu field dạng số. Dùng `countDistinct` để đếm giá trị duy nhất, chẳng hạn số khách hàng.

### Grouped analytics

Thêm `dimensions` khi cần một row cho từng giá trị hoặc time bucket:

```javascript
const result = await $ctx.$repos.ai_usage.aggregate({
  filter: {
    createdAt: {
      _gte: '2026-08-01T00:00:00+07:00',
      _lte: '2026-08-31T23:59:59.999+07:00'
    }
  },
  dimensions: [
    {
      field: 'createdAt',
      bucket: 'day',
      timezone: 'Asia/Ho_Chi_Minh'
    }
  ],
  measures: {
    requests: { count: 'id' },
    totalTokens: { sum: 'totalTokens' },
    creditChargedMilli: { sum: 'creditChargedMilli' }
  },
  sort: [{ field: 'createdAt', direction: 'asc' }],
  limit: 100
});
```

Dimension có thể là scalar field hoặc date bucket với `hour`, `day`, `week`, `month` hay `year`. Field trong `sort` phải trỏ tới output key của dimension hoặc tên measure. Dùng `page` và `limit` để phân trang các grouped row.

Các field dùng trong aggregate vẫn chịu kiểm tra authorization và giới hạn với field được mã hóa như các repository query khác. Khi query đi qua ranh giới bảo mật của user, hãy dùng repository có enforcement quyền trường như `$ctx.$repos.main` hoặc `$ctx.$repos.secure.<tableName>`.

Không đặt `aggregate` bên trong `find()`; dạng cũ này không còn được hỗ trợ.

## Sắp xếp

### Sort ascending
```javascript
const result = await $ctx.$repos.products.find({
  sort: 'name'  // Sort by name ascending
});
```

### Sort descending
```javascript
const result = await $ctx.$repos.products.find({
  sort: '-price'  // Prefix with "-" for descending
});
```

### Multi-field sorting
```javascript
const result = await $ctx.$repos.products.find({
  sort: 'category,-price'  // Sort by category ASC, then price DESC
});
```

## Toán tử lọc

Tham số `filter` hỗ trợ các toán tử tương tự MongoDB:

### Toán tử so sánh
```javascript
filter: {
  price: { _eq: 100 },        // Equal to
  price: { _neq: 100 },       // Not equal to
  price: { _gt: 100 },        // Greater than
  price: { _gte: 100 },       // Greater than or equal
  price: { _lt: 500 },        // Less than
  price: { _lte: 500 },       // Less than or equal
  price: { _between: [100, 500] }  // Between (inclusive)
}
```

### Array Operators
```javascript
filter: {
  category: { _in: ['electronics', 'gadgets'] },      // In array
  category: { _not_in: ['discontinued', 'old'] }     // Not in array
}
```

### Text Search Operators
```javascript
filter: {
  name: { _contains: 'phone' },        // Contains text (case-insensitive)
  name: { _starts_with: 'Apple' },     // Starts with
  name: { _ends_with: 'Pro' }          // Ends with
}
```

### Null Checks
```javascript
filter: {
  description: { _is_null: true },        // Field is null
  description: { _is_not_null: true }     // Field is not null
}
```

### Logical Operators
```javascript
filter: {
  _and: [                                    // All conditions must match
    { category: { _eq: 'electronics' } },
    { price: { _gte: 100 } }
  ],
  _or: [                                     // At least one condition must match
    { status: { _eq: 'active' } },
    { status: { _eq: 'pending' } }
  ],
  _not: {                                    // Negate condition
    category: { _eq: 'discontinued' }
  }
}
```

### Complex Combinations
```javascript
filter: {
  _and: [
    { category: { _in: ['electronics', 'gadgets'] } },
    {
      _or: [
        { price: { _lt: 100 } },
        { isOnSale: { _eq: true } }
      ]
    },
    { description: { _is_not_null: true } }
  ]
}
```

## Lọc theo relation

Lọc bản ghi theo dữ liệu của bảng liên quan:

```javascript
// Find products where category name is 'Electronics'
const result = await $ctx.$repos.products.find({
  filter: {
    category: {
      name: { _eq: 'Electronics' }
    }
  }
});
```

## Metadata của truy vấn

Yêu cầu metadata cho truy vấn:

```javascript
const result = await $ctx.$repos.products.find({
  filter: { category: { _eq: 'electronics' } },
  meta: 'totalCount'  // Get total count of all records (before filter)
});

console.log(result.meta.totalCount);  // Total records in table
console.log(result.data.length);      // Records matching filter
```

Các tùy chọn metadata có sẵn:
- `'totalCount'` - Total number of records in table (ignores filter)
- `'filterCount'` - Number of records matching the filter
- `['totalCount', 'filterCount']` - Both counts

## Ví dụ hoàn chỉnh

```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { category: { _in: ['electronics', 'gadgets'] } },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } },
      { description: { _is_not_null: true } }
    ]
  },
  fields: 'id,name,price,category.name',
  sort: '-price',
  limit: 20,
  meta: ['totalCount', 'filterCount']
});

const products = result.data;
const totalCount = result.meta.totalCount;
const filteredCount = result.meta.filterCount;
```

## Tiếp theo

- Xem [Tạo, cập nhật và xóa bản ghi](./create-update-delete.md)
- Tìm hiểu [Mẫu phổ biến](./patterns.md)
- Xem [Lọc truy vấn](../query-filtering.md) để biết thêm ví dụ lọc
