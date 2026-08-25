---
slug: tao-cap-nhat-xoa-ban-ghi
---

# Tạo, cập nhật và xóa bản ghi

## Tạo

Tạo bản ghi mới trong bảng. Phương thức tự trả về bản ghi đã tạo đầy đủ, gồm ID và timestamp.

### Cách dùng cơ bản

```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'New Product',
    price: 99.99,
    category: 'electronics'
  }
});

const newProduct = result.data[0];  // Full product record with ID, timestamps, etc.
```

### Tham số

```javascript
await $ctx.$repos.tableName.create({
  data: { ... },       // Data to insert (required)
  fields: '...'        // Fields to return (optional)
})
```

### Tạo bản ghi

#### Tạo đơn giản
```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'iPhone 15',
    price: 999,
    category: 'electronics'
  }
});
```

#### Tạo với mọi trường
```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'iPhone 15',
    price: 999,
    category: 'electronics',
    description: 'Latest iPhone model',
    isActive: true,
    stock: 50
  }
});
```

#### Trả về các trường xác định sau khi tạo
```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'iPhone 15',
    price: 999
  },
  fields: 'id,name,price'  // Only return these fields
});
```

### Lưu ý quan trọng

1. **ID được tạo tự động**: Không thêm `id` vào đối tượng dữ liệu; cơ sở dữ liệu tự tạo giá trị này.
2. **Timestamp được tạo tự động**: Hệ thống tự thêm `createdAt` và `updatedAt`.
3. **Trả về bản ghi đầy đủ**: Sau khi chèn, phương thức tự gọi `find()` để trả về bản ghi hoàn chỉnh.
4. **Tự động validation**: Hệ thống kiểm tra schema của bảng và các quy tắc bảo vệ hệ thống.

### Tạo kèm quan hệ

```javascript
// Create order with related items
const orderResult = await $ctx.$repos.orders.create({
  data: {
    customerId: 123,
    total: 299.99,
    status: 'pending'
  }
});

const order = orderResult.data[0];

// Create order items
const itemResult = await $ctx.$repos.order_items.create({
  data: {
    orderId: order.id,
    productId: 456,
    quantity: 2,
    price: 149.99
  }
});
```

### Xử lý lỗi

```javascript
try {
  const result = await $ctx.$repos.products.create({
    data: {
      name: 'Product Name',
      price: 99.99
    }
  });
} catch (error) {
  // Handle validation errors, constraint violations, etc.
  $ctx.$logs(`Failed to create product: ${error.message}`);
}
```

---

## Cập nhật

Cập nhật bản ghi có sẵn theo ID. Phương thức tự trả về bản ghi đã cập nhật đầy đủ.

### Cách dùng cơ bản

```javascript
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    price: 89.99,
    isOnSale: true
  }
});

const updatedProduct = result.data[0];  // Full updated product record
```

### Parameters

```javascript
await $ctx.$repos.tableName.update({
  id: 123,              // Record ID to update (required)
  data: { ... },        // Fields to update (required)
  fields: '...'         // Fields to return (optional)
})
```

### Cập nhật bản ghi

#### Cập nhật một trường
```javascript
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    price: 89.99
  }
});
```

#### Cập nhật nhiều trường
```javascript
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    price: 89.99,
    isOnSale: true,
    description: 'Updated description'
  }
});
```

#### Trả về các trường xác định sau khi cập nhật
```javascript
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    price: 89.99
  },
  fields: 'id,name,price'  // Only return these fields
});
```

### Lưu ý quan trọng

1. **ID phải tồn tại**: Bản ghi có ID đã cho phải tồn tại, nếu không hệ thống sẽ báo lỗi.
2. **Cập nhật một phần**: Chỉ cần truyền các field muốn cập nhật.
3. **Trả về bản ghi đầy đủ**: Sau khi cập nhật, phương thức tự gọi `find()` để trả về bản ghi hoàn chỉnh.
4. **Timestamp được cập nhật tự động**: Hệ thống tự cập nhật `updatedAt`.
5. **Tự động validation**: Hệ thống kiểm tra schema của bảng và các quy tắc bảo vệ hệ thống.

### Kiểm tra bản ghi có tồn tại trước

```javascript
// Find the record first
const findResult = await $ctx.$repos.products.find({
  filter: { id: { _eq: 123 } }
});

if (findResult.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Update the record
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    price: 89.99
  }
});
```

### Xử lý lỗi

```javascript
try {
  const result = await $ctx.$repos.products.update({
    id: 123,
    data: {
      price: 89.99
    }
  });
} catch (error) {
  // Handle errors: record not found, validation errors, etc.
  $ctx.$logs(`Failed to update product: ${error.message}`);
}
```

---

## Xóa

Xóa bản ghi theo ID. Phương thức trả về thông báo thành công.

### Cách dùng cơ bản

```javascript
const result = await $ctx.$repos.products.delete({
  id: 123
});

// result: { message: 'Delete successfully!', statusCode: 200 }
```

### Parameters

```javascript
await $ctx.$repos.tableName.delete({
  id: 123  // Record ID to delete (required)
})
```

### Xóa bản ghi

#### Xóa đơn giản
```javascript
const result = await $ctx.$repos.products.delete({
  id: 123
});
```

#### Kiểm tra bản ghi có tồn tại trước khi xóa
```javascript
// Find the record first
const findResult = await $ctx.$repos.products.find({
  filter: { id: { _eq: 123 } }
});

if (findResult.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Delete the record
const result = await $ctx.$repos.products.delete({
  id: 123
});
```

### Lưu ý quan trọng

1. **ID phải tồn tại**: Bản ghi có ID đã cho phải tồn tại, nếu không hệ thống sẽ báo lỗi.
2. **Hành vi cascade**: Nếu bảng có relation với `onDelete: 'cascade'`, các bản ghi liên quan sẽ tự bị xóa.
3. **Trả về thông báo thành công**: Phương thức trả về `{ message: 'Delete successfully!', statusCode: 200 }`.
4. **Không thể hoàn tác**: Thao tác xóa là vĩnh viễn, hãy kiểm tra trước khi xóa.

### Xử lý lỗi

```javascript
try {
  const result = await $ctx.$repos.products.delete({
    id: 123
  });
} catch (error) {
  // Handle errors: record not found, constraint violations, etc.
  $ctx.$logs(`Failed to delete product: ${error.message}`);
}
```

---

## Batch create, update và delete

Chỉ dùng batch repository method trong dynamic handler, hook và flow. REST CRUD endpoint được tạo sẵn vẫn là endpoint thao tác từng bản ghi.

Batch method được giới hạn có chủ đích cho bảng generic thuần túy. Enfyra từ chối schema/metadata table và bảng có normalization hoặc lifecycle tùy chỉnh. Hệ thống kiểm tra validation và quyền trên mọi target trước một bulk write, sau đó chạy một runtime reload và phát một mutation event. Với code phục vụ người dùng, dùng secure repository và vẫn phải tự áp dụng quy tắc owner, tenant và membership trong handler.

### Create nhiều bản ghi

```javascript
const result = await $ctx.$repos.secure.products.createMany({
  data: [
    { name: 'Keyboard', price: 99.99 },
    { name: 'Mouse', price: 49.99 }
  ],
  fields: ['id', 'name', 'price']
});

// result.data chứa các bản ghi đã tạo; result.count là 2
```

### Update nhiều bản ghi

`updateMany` áp dụng cùng một object data cho mọi ID. Method này không nhận relation payload; hãy update từng bản ghi nếu cần thay đổi relation.

```javascript
const result = await $ctx.$repos.secure.products.updateMany({
  ids: [101, 102, 103],
  data: { isActive: false },
  fields: ['id', 'isActive']
});

// result.data chứa các bản ghi đã update; result.count là 3
```

### Delete nhiều bản ghi

```javascript
const result = await $ctx.$repos.secure.products.deleteMany({
  ids: [101, 102, 103]
});

// result: { message: 'Delete successfully!', statusCode: 200, count: 3 }
```

## Tiếp theo

- Xem [Tìm bản ghi](./find.md) để truy vấn bản ghi
- Tìm hiểu [Mẫu phổ biến](./patterns.md) để biết thực hành tốt
- Xem [Xử lý lỗi](../error-handling.md) để xử lý lỗi đúng cách
