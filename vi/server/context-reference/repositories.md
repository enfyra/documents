---
slug: context-repository
---

# Tham chiếu Context - Repository

Truy cập các bảng cơ sở dữ liệu qua repository. Xem [Phương thức Repository](../repository-methods/) để biết đầy đủ.

## Truy cập repository

```javascript
// Access repository by table name
const productsRepo = $ctx.$repos.products;
const usersRepo = $ctx.$repos.enfyra_user;

// Access main table repository
const mainRepo = $ctx.$repos.main;

// Field permissions: main + explicit secure namespace
// - `$repos.main` — main route table, field permissions enforced
// - `$repos.secure.<tableName>` — same enforcement for any table name
// - `$repos.<tableName>` — same data access without field-permission enforcement (use `secure` when rules must apply)

// Check if repository exists
if ($ctx.$repos.products) {
  const result = await $ctx.$repos.products.find({});
}
```

## Phương thức repository

Mỗi repository cung cấp các phương thức sau:

```javascript
// Find records
const result = await $ctx.$repos.products.find({
  filter: { category: { _eq: 'electronics' } },
  fields: 'id,name,price',
  limit: 10,
  sort: '-createdAt'
});

// Create record
const createResult = await $ctx.$repos.products.create({
  data: {
    name: 'New Product',
    price: 99.99
  }
});

// Update record
const updateResult = await $ctx.$repos.products.update({
  id: 123,
  data: { price: 89.99 }
});

// Delete record
const deleteResult = await $ctx.$repos.products.delete({
  id: 123
});
```

**Lưu ý:** Mọi phương thức repository đều trả về theo dạng `{ data: [...], meta: {...} }`.

Xem [hướng dẫn về phương thức repository](../repository-methods/) để biết đầy đủ.

## Tiếp theo

- Xem [Dữ liệu request](./request-data.md) để truy cập thông tin request
- Xem [Hàm hỗ trợ và cache](./helpers-cache.md) để dùng các hàm tiện ích
- Tìm hiểu [Phương thức Repository](../repository-methods/) để xem tham chiếu đầy đủ
