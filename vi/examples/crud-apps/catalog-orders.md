---
slug: vi-du/ung-dung-crud/catalog-va-don-hang
---

# Catalog và đơn hàng

Ví dụ này bao quát một luồng cửa hàng đơn giản: sản phẩm công khai, bản ghi checkout của người dùng đã xác thực và kiểm tra tồn kho ở server.

## Table

Tạo `shop_product`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `name` | string | Bắt buộc |
| `slug` | string | Duy nhất |
| `priceCents` | integer | Bắt buộc |
| `stock` | integer | Bắt buộc |
| `status` | select | `draft`, `active`, `archived` |

Tạo `shop_order`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `customer` | relation many-to-one tới `enfyra_user` | Bắt buộc |
| `status` | select | `pending`, `paid`, `fulfilled`, `canceled` |
| `totalCents` | integer | Do server tính |

Tạo `shop_order_item`.

| Field hoặc relation | Kiểu | Ghi chú |
| --- | --- | --- |
| `order` | relation many-to-one tới `shop_order` | Bắt buộc |
| `product` | relation many-to-one tới `shop_product` | Bắt buộc |
| `quantity` | integer | Bắt buộc |
| `unitPriceCents` | integer | Snapshot tại thời điểm checkout |

## Đọc sản phẩm công khai

Cho phép công khai `GET /shop_product` và thêm pre-hook.

```javascript
@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { status: { _eq: 'active' } }
  ]
};
```

## Tạo checkout route

Tạo custom route `POST /shop/checkout`. Route handler tạo bản ghi đơn hàng và các item một cách nguyên tử từ dữ liệu sản phẩm đáng tin cậy.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const items = Array.isArray(@BODY.items) ? @BODY.items : [];
if (!items.length) {
  @THROW400('items are required');
}

const productIds = items.map((item) => item.product);
const productsResult = await #shop_product.find({
  filter: {
    id: { _in: productIds },
    status: { _eq: 'active' }
  },
  fields: 'id,name,priceCents,stock',
  limit: productIds.length
});

const products = productsResult.data || [];
if (products.length !== productIds.length) {
  @THROW400('one or more products are unavailable');
}

let totalCents = 0;
for (const item of items) {
  const product = products.find((row) => row.id === item.product);
  if (!product || item.quantity < 1 || item.quantity > product.stock) {
    @THROW400('invalid quantity');
  }
  totalCents += product.priceCents * item.quantity;
}

const orderResult = await #shop_order.create({
  data: {
    customer: { id: @USER.id },
    status: 'pending',
    totalCents
  }
});

const order = orderResult.data?.[0];
if (!order) {
  @THROW500('order was not created');
}

for (const item of items) {
  const product = products.find((row) => row.id === item.product);
  await #shop_order_item.create({
    data: {
      order: { id: order.id },
      product: { id: product.id },
      quantity: item.quantity,
      unitPriceCents: product.priceCents
    }
  });
}

return order;
```

## Danh sách đơn hàng của khách

Thêm `GET /shop_order` pre-hook để người dùng chỉ đọc được đơn hàng của họ.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { customer: { id: { _eq: @USER.id } } }
  ]
};
```

```bash
curl "$ENFYRA_API_URL/shop_order?fields=id,status,totalCents,createdAt&sort=-createdAt" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```
