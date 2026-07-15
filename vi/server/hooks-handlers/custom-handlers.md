---
slug: handler-tuy-chinh
---

# Hooks và Handler - Handler tùy chỉnh và loại Hook

Handler tùy chỉnh thay thế thao tác CRUD mặc định bằng logic nghiệp vụ của bạn.

## Handler tùy chỉnh

### Khi nào dùng handler tùy chỉnh

- Logic nghiệp vụ phức tạp không phù hợp với CRUD
- Hoạt động nhiều bước
- Tích hợp API bên ngoài
- Xác thực tùy chỉnh ngoài preHooks
- Các định dạng phản hồi đặc biệt

### Handler tùy chỉnh cơ bản

```javascript
// Create with additional logic
const result = await $ctx.$repos.products.create({
  data: {
    name: $ctx.$body.name,
    price: $ctx.$body.price,
    createdBy: $ctx.$user.id
  }
});

// Perform additional operations
const product = result.data[0];

// Create related records
await $ctx.$repos.product_images.create({
  data: {
    productId: product.id,
    imageUrl: $ctx.$body.imageUrl
  }
});

// Return custom response
return {
  success: true,
  product: product,
  message: 'Product created successfully'
};
```

### Thao tác nhiều bước

```javascript
// Create order with items
const orderResult = await $ctx.$repos.orders.create({
  data: {
    customerId: $ctx.$body.customerId,
    total: 0,
    status: 'pending'
  }
});

const order = orderResult.data[0];
let total = 0;

// Create order items
for (const item of $ctx.$body.items) {
  const productResult = await $ctx.$repos.products.find({
    filter: { id: { _eq: item.productId } }
  });
  
  if (productResult.data.length === 0) {
    $ctx.$throw['404'](`Product ${item.productId} not found`);
    return;
  }
  
  const product = productResult.data[0];
  const itemTotal = product.price * item.quantity;
  total += itemTotal;
  
  await $ctx.$repos.order_items.create({
    data: {
      orderId: order.id,
      productId: item.productId,
      quantity: item.quantity,
      price: product.price,
      total: itemTotal
    }
  });
}

// Update order total
await $ctx.$repos.orders.update({
  id: order.id,
  data: { total: total }
});

// Return complete order
const finalOrder = await $ctx.$repos.orders.find({
  filter: { id: { _eq: order.id } }
});

return finalOrder;
```

### Hành vi CRUD mặc định

Nếu bạn không cung cấp trình xử lý tùy chỉnh, hệ thống sẽ tự động thực hiện các hoạt động CRUD dựa trên phương thức HTTP:

- **GET**: Truy vấn bản ghi bằng repository `find()`
- **POST**: Tạo bản ghi bằng repository `create()`
- **PATCH**: Cập nhật bản ghi bằng repository `update()`
- **DELETE**: Xóa bản ghi bằng repository `delete()`

CRUD mặc định sử dụng dữ liệu từ:
- `$ctx.$query` cho request GET (`filter`, `fields`, `limit`, `sort`)
- `$ctx.$body` cho request POST/PATCH (dữ liệu cần tạo hoặc cập nhật)
- `$ctx.$params.id` cho request PATCH/DELETE (ID bản ghi)

---

## Các loại hook

### Hook toàn cục

Hook toàn cục chạy trên mọi route.

**Cấu hình**
- Route: `null` (không chỉ định route)
- Methods: `[]` (mọi phương thức) hoặc các phương thức cụ thể như `['POST', 'PATCH']`

**Ví dụ**
```javascript
// Global preHook - runs on all routes, all methods
// Configuration: route = null, methods = []

// Global preHook - runs on all routes, POST only
// Configuration: route = null, methods = ['POST']
```

### Hook theo route cụ thể

Hook theo route chỉ chạy trên route được chỉ định.

**Cấu hình**
- Route: Route cụ thể, ví dụ route có path `/users`
- Methods: `[]` (mọi phương thức) hoặc phương thức cụ thể

**Ví dụ**
```javascript
// Route preHook - runs on /users route, all methods
// Configuration: route = /users route, methods = []

// Route preHook - runs on /users route, POST only
// Configuration: route = /users route, methods = ['POST']
```

### Thứ tự thực thi

Hook thực thi theo thứ tự sau:

1. Hook toàn cục (mọi phương thức)
2. Hook toàn cục (phương thức cụ thể)
3. Hook theo route (mọi phương thức)
4. Hook theo route (phương thức cụ thể)

**Ví dụ cho `POST /users`:**
```
Global preHook (all)  Global preHook (POST)  Route preHook (all)  Route preHook (POST)  Handler  Global postHook (all)  Global postHook (POST)  Route postHook (all)  Route postHook (POST)
```

postHook cũng chạy theo nhóm **global rồi đến route** như preHook, không theo thứ tự đảo ngược. Xem [Vòng đời API – postHook](../api-lifecycle.md#phase-8-posthooks-execution).

## Tiếp theo

- Xem [preHooks](./prehooks.md) để biết thao tác trước handler
- Tìm hiểu [postHooks](./posthooks.md) để biết thao tác sau handler
- Xem [Mẫu phổ biến](./patterns.md) để biết thực hành tốt
