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
  @THROW404('Product not found');
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

## Đọc, tính và ghi an toàn với `updateLocked`

Dùng `updateLocked` khi giá trị mới phụ thuộc vào bản ghi đang lưu, chẳng hạn trừ credit hoặc đổi trạng thái sau khi kiểm tra giá trị hiện tại. Truyền một object có cùng các key như `update`: `id`, `fields` tùy chọn và `data`. Với method này, `data` là callback nhận bản ghi trong phạm vi database được bảo vệ và trả về các giá trị cần cập nhật.

```javascript
const amount = Number(@BODY.amount);
if (!Number.isFinite(amount) || amount <= 0) @THROW400("amount must be positive");

const result = await #secure.accounts.updateLocked({
  id: @PARAMS.id,
  fields: ['credit', 'status'],
  data: (current) => {
    if (current.credit < amount) @THROW400("Insufficient credit");
    return {
      credit: current.credit - amount,
      status: 'active'
    };
  }
});

return result.data[0];
```

Giá trị `credit` được tính và giá trị cố định `status` do `data` trả về được ghi cùng nhau. `fields` tùy chọn ở ngoài callback chỉ chọn dữ liệu trả về, giống `update`; nó không chọn field bị khóa hoặc được ghi. Có thể bỏ `fields` để dùng projection trả về thông thường của mutation. Bản ghi đưa vào callback không phụ thuộc pagination, projection hay deep query của request.

| Backend | Cơ chế bảo vệ |
|---|---|
| PostgreSQL và MySQL/InnoDB | Transaction trên primary khóa bản ghi gốc bằng `SELECT ... FOR UPDATE` trước khi gọi callback. |
| MongoDB hỗ trợ native transaction | Đọc, callback và ghi nằm trong cùng native transaction. Write conflict thuộc nhóm retryable làm transaction chạy lại, đọc lại bản ghi và tính lại patch. |
| MongoDB standalone hoặc phạm vi application saga | Không hỗ trợ; method báo lỗi trước khi gọi callback. |

Callback `data` chỉ nên tính toán, kiểm tra nghiệp vụ và trả các giá trị cần cập nhật. MongoDB có thể gọi lại callback, nên không gửi HTTP request, thay đổi cache/storage, emit socket, trigger flow hoặc gọi repository khác bên trong. Mỗi lần chạy phải tính từ `current` được truyền vào; không dùng giá trị thay thế đã tính từ một lần đọc trước đó. Đặt kiểm tra owner/tenant phụ thuộc trạng thái hiện tại trong callback.

`updateLocked` hỗ trợ các bảng generic thông thường. Method từ chối route schema/metadata và bảng có normalization hoặc lifecycle handler riêng cho update. Mutation policy, field permission, quy tắc field được cập nhật và cascade vẫn chạy như mutation thông thường. Secure repository chỉ đưa các field được phép đọc vào callback; `#accounts` vẫn là truy cập trusted có chủ đích.

Method mở transaction hoặc tham gia transaction đang có. Khi tham gia transaction ngoài, phạm vi ngoài quyết định commit, rollback và retry của MongoDB. Cache invalidation và mutation event chỉ được phát sau commit; attempt rollback hoặc bị retry không phát effects của nó. Thời gian chờ khóa SQL và native transaction MongoDB mới được giới hạn 30 giây; cancellation của script vẫn có hiệu lực.

Trên PostgreSQL/MySQL, row lock được giữ đến khi transaction ngoài commit hoặc rollback; callback `data` chạy xong không nhả khóa. Trạng thái đưa vào callback và kiểm tra update policy dùng locking read, nên snapshot repeatable-read cũ của MySQL không cung cấp giá trị gốc đã cũ. Các thao tác repository khác đặt trong transaction ngoài, bên ngoài callback `data` thuần tính toán. Dùng [`findLocked`](./find.md#findlocked) khi cần khóa owner chung trước các phép đọc phụ thuộc đó; kiểm tra runtime đã deploy có hỗ trợ method này.

Bảo vệ bản ghi gốc không tự đưa các bản ghi khác hay hệ thống bên ngoài vào cùng invariant. Client gửi giá trị tuyệt đối đã cũ vẫn cần điều kiện trạng thái/revision riêng. Bản ghi không tồn tại, callback trả sai dạng và transaction retry không thành công đều được báo lỗi thay vì trả update thành công.

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
  @THROW404('Product not found');
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
