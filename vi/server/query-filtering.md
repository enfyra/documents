---
slug: loc-truy-van
---

# Lọc truy vấn

Enfyra hỗ trợ các toán tử lọc theo phong cách MongoDB để truy vấn dữ liệu.

- **HTTP / REST:** truyền điều kiện qua tham số truy vấn **`filter`** (JSON trong query string).
- **Handler, hook, flow (`$ctx.$repos.*.find`, `#table.find`, v.v.):** truyền cùng điều kiện vào **`filter`** khi gọi repository.

## Điều hướng nhanh

- [Toán tử so sánh](#toan-tu-so-sanh) - `_eq`, `_gt`, `_lt`, v.v.
- [Toán tử mảng](#toan-tu-mang) - `_in`, `_not_in`
- [Tìm kiếm văn bản](#tim-kiem-van-ban) - `_contains`, `_starts_with`, `_ends_with`
- [Kiểm tra null](#kiem-tra-null) - `_is_null`, `_is_not_null`
- [Toán tử logic](#toan-tu-logic) - `_and`, `_or`, `_not`
- [Toán tử khoảng](#toan-tu-khoang) - `_between`
- [Ví dụ nâng cao](#vi-du-nang-cao) - Các tình huống thực tế

## Toán tử so sánh

### _eq (Bằng)

Khớp giá trị chính xác.

```javascript
// Tìm sản phẩm có giá = 100
const result = await $ctx.$repos.products.find({
  filter: { price: { _eq: 100 } }
});

// URL: /products?filter={"price":{"_eq":100}}
```

### _neq (Không bằng)

Loại trừ giá trị khớp.

```javascript
// Tìm sản phẩm có giá != 0
const result = await $ctx.$repos.products.find({
  filter: { price: { _neq: 0 } }
});
```

### _gt (Lớn hơn)

Khớp các giá trị lớn hơn.

```javascript
// Tìm sản phẩm có giá > 100
const result = await $ctx.$repos.products.find({
  filter: { price: { _gt: 100 } }
});
```

### _gte (Lớn hơn hoặc bằng)

Khớp các giá trị lớn hơn hoặc bằng.

```javascript
// Tìm sản phẩm có giá >= 100
const result = await $ctx.$repos.products.find({
  filter: { price: { _gte: 100 } }
});
```

### _lt (Nhỏ hơn)

Khớp các giá trị nhỏ hơn.

```javascript
// Tìm sản phẩm có giá < 500
const result = await $ctx.$repos.products.find({
  filter: { price: { _lt: 500 } }
});
```

### _lte (Nhỏ hơn hoặc bằng)

Khớp các giá trị nhỏ hơn hoặc bằng.

```javascript
// Tìm sản phẩm có giá <= 500
const result = await $ctx.$repos.products.find({
  filter: { price: { _lte: 500 } }
});
```

## Toán tử mảng

### _in (Nằm trong mảng)

Khớp bất kỳ giá trị nào trong mảng.

```javascript
// Tìm sản phẩm thuộc các danh mục cụ thể
const result = await $ctx.$repos.products.find({
  filter: {
    category: { _in: ['electronics', 'gadgets', 'phones'] }
  }
});

// URL: /products?filter={"category":{"_in":["electronics","gadgets"]}}
```

### _not_in (Không nằm trong mảng)

Loại trừ các giá trị thuộc mảng.

```javascript
// Tìm sản phẩm không thuộc các danh mục này
const result = await $ctx.$repos.products.find({
  filter: {
    category: { _not_in: ['discontinued', 'old'] }
  }
});
```

## Tìm kiếm văn bản

Các toán tử tìm kiếm văn bản không phân biệt chữ hoa và chữ thường.

### _contains (Chứa văn bản)

Khớp các trường có chứa văn bản.

```javascript
// Tìm sản phẩm có tên chứa "phone"
const result = await $ctx.$repos.products.find({
  filter: {
    name: { _contains: 'phone' }
  }
});

// Khớp: "iPhone", "Phone Case", "Smartphone"
```

### _starts_with (Bắt đầu bằng văn bản)

Khớp các trường bắt đầu bằng văn bản.

```javascript
// Tìm sản phẩm bắt đầu bằng "Apple"
const result = await $ctx.$repos.products.find({
  filter: {
    name: { _starts_with: 'Apple' }
  }
});

// Khớp: "Apple iPhone", "Apple Watch"
// Không khớp: "iPhone", "My Apple Product"
```

### _ends_with (Kết thúc bằng văn bản)

Khớp các trường kết thúc bằng văn bản.

```javascript
// Tìm sản phẩm kết thúc bằng "Pro"
const result = await $ctx.$repos.products.find({
  filter: {
    name: { _ends_with: 'Pro' }
  }
});

// Khớp: "iPhone Pro", "MacBook Pro"
// Không khớp: "Pro iPhone", "Professional"
```

## Kiểm tra null

### _is_null (Trường là null)

Khớp các trường có giá trị null.

```javascript
// Tìm sản phẩm không có mô tả
const result = await $ctx.$repos.products.find({
  filter: {
    description: { _is_null: true }
  }
});
```

### _is_not_null (Trường không phải null)

Khớp các trường không có giá trị null.

```javascript
// Tìm sản phẩm có mô tả
const result = await $ctx.$repos.products.find({
  filter: {
    description: { _is_not_null: true }
  }
});
```

## Toán tử khoảng

### _between (Nằm giữa hai giá trị)

Khớp các giá trị nằm giữa hai số, bao gồm cả hai đầu mút.

```javascript
// Tìm sản phẩm có giá từ 100 đến 500
const result = await $ctx.$repos.products.find({
  filter: {
    price: { _between: [100, 500] }
  }
});

// Tương đương: price >= 100 AND price <= 500
```

## Toán tử logic

### _and (Mọi điều kiện phải khớp)

Kết hợp nhiều điều kiện bằng logic AND.

```javascript
// Tìm sản phẩm đang hoạt động thuộc danh mục electronics, giá >= 100
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { category: { _eq: 'electronics' } },
      { price: { _gte: 100 } },
      { isActive: { _eq: true } }
    ]
  }
});
```

### _or (Ít nhất một điều kiện phải khớp)

Kết hợp nhiều điều kiện bằng logic OR.

```javascript
// Tìm sản phẩm đang hoạt động HOẶC đang giảm giá
const result = await $ctx.$repos.products.find({
  filter: {
    _or: [
      { isActive: { _eq: true } },
      { isOnSale: { _eq: true } }
    ]
  }
});
```

### _not (Phủ định điều kiện)

Phủ định một điều kiện.

```javascript
// Tìm sản phẩm chưa ngừng kinh doanh
const result = await $ctx.$repos.products.find({
  filter: {
    _not: {
      status: { _eq: 'discontinued' }
    }
  }
});
```

### Kết hợp logic phức tạp

Kết hợp các toán tử logic để tạo truy vấn phức tạp.

```javascript
// Tìm sản phẩm đang hoạt động thuộc electronics HOẶC gadgets, giá từ 100-500
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      {
        _or: [
          { category: { _eq: 'electronics' } },
          { category: { _eq: 'gadgets' } }
        ]
      },
      { price: { _between: [100, 500] } },
      { isActive: { _eq: true } }
    ]
  }
});
```

## Ví dụ nâng cao

### Ví dụ 1: Nhiều điều kiện

```javascript
// Tìm sản phẩm đang hoạt động thuộc danh mục electronics hoặc gadgets,
// có giá từ 100-500 và tên chứa "phone"
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      {
        _or: [
          { category: { _eq: 'electronics' } },
          { category: { _eq: 'gadgets' } }
        ]
      },
      { price: { _between: [100, 500] } },
      { name: { _contains: 'phone' } },
      { isActive: { _eq: true } }
    ]
  }
});
```

### Ví dụ 2: Loại trừ giá trị cụ thể

```javascript
// Tìm sản phẩm không thuộc danh mục discontinued hoặc old,
// có mô tả và giá >= 50
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      {
        category: { _not_in: ['discontinued', 'old'] }
      },
      { description: { _is_not_null: true } },
      { price: { _gte: 50 } }
    ]
  }
});
```

### Ví dụ 3: Tìm kiếm văn bản kèm điều kiện khác

```javascript
// Tìm sản phẩm có tên bắt đầu bằng "Apple",
// giá >= 200 và chưa ngừng kinh doanh
const result = await $ctx.$repos.products.find({
  filter: {
    _and: [
      { name: { _starts_with: 'Apple' } },
      { price: { _gte: 200 } },
      {
        _not: {
          status: { _eq: 'discontinued' }
        }
      }
    ]
  }
});
```

### Ví dụ 4: Truy vấn theo khoảng ngày

```javascript
// Tìm đơn hàng được tạo giữa hai ngày
const startDate = new Date('2024-01-01');
const endDate = new Date('2024-12-31');

const result = await $ctx.$repos.orders.find({
  filter: {
    createdAt: { _between: [startDate, endDate] }
  }
});
```

## Lọc theo quan hệ

Lọc bản ghi dựa trên dữ liệu ở bảng liên quan. Bạn có thể lọc quan hệ theo hai cách:

### Cách 1: Lọc trực tiếp theo ID quan hệ

Lọc trực tiếp trên quan hệ bằng toán tử so sánh ID. Đây là cách ngắn gọn nhất để lọc theo ID quan hệ.

```javascript
// Tìm menu item không có parent (parent là null)
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _is_null: true }
  }
});

// Tìm menu item có parent (parent không phải null)
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _is_not_null: true }
  }
});

// Tìm menu item có parent ID bằng 3
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _eq: 3 }
  }
});

// Tìm menu item có parent ID thuộc [3, 4, 5]
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _in: [3, 4, 5] }
  }
});

// Tìm menu item có parent ID khác 3
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _neq: 3 }
  }
});

// Tìm menu item có parent ID không thuộc [1, 2]
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: { _not_in: [1, 2] }
  }
});
```

### Cách 2: Lọc theo ID quan hệ qua trường id/_id

Bạn cũng có thể chỉ định rõ trường `id` hoặc `_id` của quan hệ để lọc.

```javascript
// Tìm menu item có parent ID bằng 3
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: {
      id: { _eq: 3 }
    }
  }
});

// Tìm menu item không có parent
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: {
      id: { _is_null: true }
    }
  }
});

// Tìm menu item có parent ID thuộc [3, 4, 5]
const result = await $ctx.$repos.enfyra_menu.find({
  filter: {
    parent: {
      id: { _in: [3, 4, 5] }
    }
  }
});
```

### Cách 3: Lọc theo trường của quan hệ

Lọc theo các trường trong bảng liên quan.

```javascript
// Tìm sản phẩm có tên danh mục là "Electronics"
const result = await $ctx.$repos.products.find({
  filter: {
    category: {
      name: { _eq: 'Electronics' }
    }
  }
});

// Tìm đơn hàng có email khách hàng chứa "@example.com"
const result = await $ctx.$repos.orders.find({
  filter: {
    customer: {
      email: { _contains: '@example.com' }
    }
  }
});
```

**Lưu ý:** Cách 1 và Cách 2 cho cùng kết quả khi lọc theo ID quan hệ. Dùng Cách 1 nếu muốn cú pháp ngắn gọn, hoặc Cách 2 nếu muốn chỉ định trường tường minh.

## Dùng bộ lọc trong URL

Bộ lọc có thể dùng trong query string của URL khi gọi REST API.

```http
# Simple filter
GET /products?filter={"category":{"_eq":"electronics"}}

# Multiple conditions
GET /products?filter={"price":{"_gte":100},"isActive":{"_eq":true}}

# Complex filter with logical operators
GET /products?filter={"_and":[{"category":{"_eq":"electronics"}},{"price":{"_between":[100,500]}}]}

# Text search
GET /products?filter={"name":{"_contains":"phone"}}
```

## Kết hợp sắp xếp và phân trang

Kết hợp bộ lọc với sắp xếp và phân trang.

```javascript
const result = await $ctx.$repos.products.find({
  filter: {
    category: { _eq: 'electronics' },
    price: { _between: [100, 500] },
    isActive: { _eq: true }
  },
  fields: 'id,name,price',
  sort: '-price',
  limit: 20,
  meta: 'totalCount'
});
```

**Quy tắc sắp xếp:**
- Nếu không chỉ định `sort`, kết quả được sắp theo `id` tăng dần.
- Sắp xếp chỉ áp dụng cho bảng cha.
- Các mảng lồng nhau luôn được sắp xếp nội bộ theo `id`.
- Các cột `createdAt`, `updatedAt` và cột scalar kiểu `date`, `datetime`, `timestamp` được tự tạo index đơn trường. SQL dùng `id` để phân định thứ tự ổn định; Mongo dùng `_id`.
- Với mẫu truy vấn thường xuyên kết hợp thời gian và điều kiện khác, hãy tạo compound index tường minh, chẳng hạn `status + lastMessageAt` hoặc `project + createdAt`. Không tạo index đơn trường trùng lặp cho cột thời gian.
- Có thể sắp xếp các hàng cha theo aggregate của quan hệ danh sách trực tiếp bằng `_count(relationName)`, `_max(relationName.fieldName)` hoặc `_min(relationName.fieldName)`.
- Không thể sắp xếp hàng cha bằng dotted sort đi qua quan hệ to-many, chẳng hạn `messages.createdAt`.

## Chọn trường

Tuỳ chọn `fields` hỗ trợ chế độ bao gồm và loại trừ trường.

Chế độ bao gồm là mặc định:

```javascript
const result = await $ctx.$repos.users.find({
  fields: 'id,email,role.name'
});
```

Chế độ loại trừ được dùng khi bất kỳ tên trường nào có tiền tố `-`:

```javascript
const result = await $ctx.$repos.enfyra_route_handler.find({
  fields: '-compiledCode'
});
```

Trong chế độ loại trừ, các tên trường dương ở cùng phạm vi bị bỏ qua. `fields: 'id,-compiledCode'` trả về mọi trường có thể đọc trừ `compiledCode`, không chỉ `id`.

Loại trừ bằng dotted path cũng dùng được với bản ghi liên quan:

```javascript
const result = await $ctx.$repos.users.find({
  fields: '-role.avatar'
});
```

Tên trường và quan hệ bị loại trừ phải tồn tại trong metadata. Tên không xác định sẽ gây lỗi request thay vì bị bỏ qua.

## Hạn chế với trường mã hoá

Trường có `isEncrypted=true` được mã hoá khi lưu trữ và chỉ giải mã sau khi bản ghi được chọn. Không dùng trường mã hoá trong `filter` hoặc `sort`, kể cả truy vấn gốc lẫn truy vấn `deep` lồng nhau.

```javascript
// Không hỗ trợ: api_token đã được mã hoá
await $ctx.$repos.integrations.find({
  filter: {
    api_token: { _eq: 'plaintext-token' }
  }
});

// Not supported: api_token is encrypted
await $ctx.$repos.integrations.find({
  sort: 'api_token'
});
```

Bản mã không thể được so sánh có ý nghĩa cho phép bằng, tìm kiếm văn bản, truy vấn khoảng hay sắp xếp; cho phép các thao tác này cũng làm lộ chi tiết triển khai. Khi cần tìm theo giá trị suy ra từ bí mật, hãy lưu một lookup key hoặc hash riêng không chứa bí mật.

## Thực hành tốt

1. **Chọn toán tử phù hợp** - Dùng toán tử đúng với tình huống của bạn.
2. **Kết hợp điều kiện hợp lý** - Dùng `_and` và `_or` để tạo truy vấn phức tạp.
3. **Dùng index** - Tạo database index cho các trường thường xuyên được lọc.
4. **Giới hạn kết quả** - Luôn dùng `limit` cho truy vấn có thể trả về nhiều bản ghi.
5. **Dùng tìm kiếm văn bản cẩn thận** - Tìm kiếm văn bản có thể chậm hơn, nên kết hợp với bộ lọc khác.

## Truy vấn Deep (quan hệ lồng nhau)

Dùng tham số `deep` để truy vấn nhiều cấp dữ liệu liên quan trong một request. Cách này đặc biệt hữu ích khi cần lấy object graph phức tạp.

### Cú pháp Deep cơ bản

```javascript
// Lấy người dùng kèm các bài viết của họ
const result = await $ctx.$repos.users.find({
  fields: 'id,name,email',
  deep: {
    posts: {
      fields: 'id,title,content'
    }
  }
});
```

`fields` bên trong mỗi quan hệ deep cũng có thể dùng chế độ loại trừ:

```javascript
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: '-compiledCode,-author.avatar',
      limit: 5,
      deep: {
        author: {
          fields: '-avatar'
        }
      }
    }
  }
});
```

### Truy vấn lồng nhiều cấp

Lấy quan hệ lồng sâu qua nhiều bảng:

```javascript
// Lấy người dùng kèm bài viết, bình luận và tác giả
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      deep: {
        comments: {
          fields: 'id,content',
          deep: {
            author: {
              fields: 'id,name,email'
            }
          }
        }
      }
    }
  }
});
```

### Deep query kèm filter

Lọc quan hệ lồng nhau ở bất kỳ cấp nào:

```javascript
// Chỉ lấy người dùng có bài viết đang hoạt động
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      filter: {
        isActive: { _eq: true }
      }
    }
  }
});

// Filter lồng nhau ở nhiều cấp
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      filter: {
        status: { _eq: 'published' }
      },
      deep: {
        comments: {
          fields: 'id,content',
          filter: {
            isApproved: { _eq: true }
          }
        }
      }
    }
  }
});
```

### Deep query kèm sort

Sắp xếp quan hệ lồng nhau ở bất kỳ cấp nào bằng tham số `deep`:

```javascript
// Lấy người dùng với bài viết được sắp theo ngày
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      sort: '-createdAt'
    }
  }
});

// Sắp xếp nhiều cấp
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      sort: '-createdAt',
      deep: {
        comments: {
          fields: 'id,content',
          sort: 'createdAt'
        }
      }
    }
  }
});
```

Deep sort sắp xếp các hàng con đã tải bên trong từng hàng cha. Nó không thay đổi thứ tự tập kết quả cha. Muốn sắp xếp các hàng cha theo dữ liệu con, hãy dùng aggregate sort ở cấp gốc:

```javascript
const result = await $ctx.$repos.cloud_support_tickets.find({
  fields: 'id,subject,status,project.id,project.name',
  sort: '-_max(messages.createdAt),-createdAt',
  limit: 25,
  deep: {
    messages: {
      fields: 'id,authorKind,body,createdAt',
      sort: '-createdAt',
      limit: 3
    }
  }
});
```

Dùng `_max(messages.createdAt)` cho giá trị con mới nhất, `_min(messages.createdAt)` cho giá trị con sớm nhất và `_count(messages)` cho số lượng con. Các aggregate sort helper chỉ hỗ trợ quan hệ trực tiếp `one-to-many` và `many-to-many`; trường aggregate phải là trường scalar không mã hoá của bảng liên quan.

### Deep query kèm limit và offset

Giới hạn số bản ghi lồng nhau được trả về:

```javascript
// Lấy người dùng kèm 5 bài viết gần nhất
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      sort: '-createdAt',
      limit: 5
    }
  }
});

// Phân trang quan hệ lồng nhau
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title',
      page: 2,      // Page 2
      limit: 10     // 10 items per page
    }
  }
});
```

### Kết hợp mọi tuỳ chọn Deep

Kết hợp filter, sort và phân trang trong deep query:

```javascript
// Ví dụ deep query phức tạp
const result = await $ctx.$repos.users.find({
  fields: 'id,name',
  deep: {
    posts: {
      fields: 'id,title,createdAt',
      filter: {
        status: { _eq: 'published' }
      },
      sort: '-createdAt',
      limit: 10,
      deep: {
        comments: {
          fields: 'id,content,createdAt',
          filter: {
            isApproved: { _eq: true }
          },
          sort: 'createdAt',
          limit: 5
        }
      }
    }
  }
});
```

### Ví dụ URL

Deep query hoạt động trong URL REST API:

```http
# Single level deep
GET /users?fields=id,name&deep={"posts":{"fields":"id,title"}}

# Multi-level deep
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","deep":{"comments":{"fields":"id,content"}}}}

# Deep with filter
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","filter":{"status":{"_eq":"published"}}}}

# Deep with sort and limit
GET /users?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt","limit":5}}

# Complete deep query
GET /users?fields=id,name&deep={"posts":{"fields":"id,title,createdAt","filter":{"isActive":{"_eq":true}},"sort":"-createdAt","limit":10}}
```

### Lưu ý về hiệu năng

1. **Giới hạn kết quả lồng nhau** - Luôn dùng `limit` cho quan hệ one-to-many và many-to-many.
2. **Lọc đúng cấp** - Áp dụng filter càng sớm càng tốt để giảm dữ liệu truyền đi.
3. **Chỉ chọn trường cần thiết** - Chỉ định chính xác trường trong `fields` thay vì dùng `*`.
4. **Tránh lồng quá sâu** - Deep query nhiều cấp có thể ảnh hưởng hiệu năng.
5. **Dùng phân trang** - Với tập dữ liệu lồng nhau lớn, dùng `page` và `limit` thay vì lấy toàn bộ.

### SQL và MongoDB

**Cơ sở dữ liệu SQL (PostgreSQL, MySQL):**
- Dùng CTE (Common Table Expressions) để tối ưu.
- Sinh subquery hiệu quả cho từng cấp lồng nhau.
- Tự động thêm mệnh đề WHERE để join bảng liên quan chính xác.

**MongoDB:**
- Dùng aggregation pipeline với thao tác $lookup.
- Tự xác định localField và foreignField từ metadata.
- Giữ nguyên cách xử lý kiểu dữ liệu (chuyển đổi ObjectId).

Cả hai loại cơ sở dữ liệu đều xử lý cùng cú pháp deep query một cách nhất quán.

## Bước tiếp theo

- Xem [Repository Methods](./repository-methods/) để biết đầy đủ về `find()`.
- Xem [Context Reference](./context-reference/) để truy cập tham số truy vấn.
- Xem [API Lifecycle](./api-lifecycle.md) để hiểu quá trình xử lý truy vấn.
