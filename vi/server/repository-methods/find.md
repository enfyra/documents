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
