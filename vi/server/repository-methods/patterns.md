---
slug: mau-repository-pho-bien
---

# Phương thức Repository - Các mẫu phổ biến

Các mẫu phổ biến và thực hành tốt khi làm việc với phương thức repository.

## Mẫu phổ biến

### Mẫu 1: Tạo và trả về bản ghi đầy đủ

```javascript
const result = await $ctx.$repos.products.create({
  data: {
    name: 'New Product',
    price: 99.99
  }
});

// result.data[0] contains the full record with ID, timestamps, etc.
const newProduct = result.data[0];
```

### Mẫu 2: Cập nhật theo dữ liệu hiện tại

```javascript
// Get current record
const current = await $ctx.$repos.products.find({
  filter: { id: { _eq: 123 } }
});

if (current.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

const product = current.data[0];

// Update based on current data
const result = await $ctx.$repos.products.update({
  id: 123,
  data: {
    stock: (product.stock || 0) + 10  // Increment stock
  }
});
```

### Mẫu 3: Cập nhật hoặc tạo có điều kiện

```javascript
const existing = await $ctx.$repos.products.find({
  filter: { name: { _eq: 'Product Name' } }
});

if (existing.data.length > 0) {
  // Update existing
  const result = await $ctx.$repos.products.update({
    id: existing.data[0].id,
    data: {
      price: 89.99
    }
  });
} else {
  // Create new
  const result = await $ctx.$repos.products.create({
    data: {
      name: 'Product Name',
      price: 89.99
    }
  });
}
```

### Mẫu 4: Xóa kèm xác thực

```javascript
// Check if record can be deleted
const product = await $ctx.$repos.products.find({
  filter: { id: { _eq: 123 } }
});

if (product.data.length === 0) {
  $ctx.$throw['404']('Product not found');
  return;
}

// Check business rules
if (product.data[0].status === 'active') {
  $ctx.$throw['400']('Cannot delete active product');
  return;
}

// Delete
const result = await $ctx.$repos.products.delete({
  id: 123
});
```

---

## Định dạng giá trị trả về

Mọi phương thức repository đều trả dữ liệu theo một định dạng nhất quán:

### Response thành công

```javascript
{
  data: [
    { id: 1, name: 'Product 1', price: 99.99 },
    { id: 2, name: 'Product 2', price: 149.99 }
  ],
  meta: {                    // Only when meta parameter is provided
    totalCount: 100,
    filterCount: 2
  }
}
```

### Response lỗi

Khi có lỗi, hệ thống ném exception; bạn nên bắt exception này:

```javascript
try {
  const result = await $ctx.$repos.products.find({ ... });
} catch (error) {
  // error.message contains the error description
  // Use $ctx.$throw to throw proper HTTP errors
}
```

---

## Thực hành tốt

1. **Luôn kiểm tra mảng dữ liệu**: Phương thức repository trả `{data: []}`; luôn kiểm tra `data.length` trước khi truy cập `data[0]`.
2. **Dùng tham số fields**: Khi chỉ cần các trường cụ thể, dùng `fields` để giảm dữ liệu truyền tải.
3. **Xác thực trước khi thao tác**: Kiểm tra bản ghi có tồn tại trước khi cập nhật hoặc xóa
4. **Xử lý lỗi**: Luôn bọc lời gọi repository trong `try/catch` để xử lý lỗi đúng cách.
5. **Dùng limit cho truy vấn**: Đừng lấy toàn bộ bản ghi khi không cần; hãy đặt giới hạn phù hợp.
6. **Tận dụng metadata**: Dùng tham số `meta` để lấy số lượng mà không cần tải toàn bộ dữ liệu.
7. **Thao tác theo lô**: Khi có thể, dùng filter để lấy nhiều bản ghi trong một truy vấn thay vì nhiều truy vấn riêng lẻ.

## Tiếp theo

- Tìm hiểu [Tìm bản ghi](./find.md) để truy vấn bản ghi
- Xem [Tạo, cập nhật và xóa bản ghi](./create-update-delete.md)
- Xem [Tham chiếu Context](../context-reference/) để biết mọi thuộc tính có sẵn
- Xem [Lọc truy vấn](../query-filtering.md) để biết các mẫu lọc nâng cao
- Xem [Vòng đời API](../api-lifecycle.md) để hiểu repository tham gia luồng request thế nào
