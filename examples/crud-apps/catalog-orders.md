# Catalog And Orders

This example covers a simple store flow: public products, authenticated checkout records, and server-side inventory checks.

## Tables

Create `shop_product`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `name` | string | Required |
| `slug` | string | Unique |
| `priceCents` | integer | Required |
| `stock` | integer | Required |
| `status` | select | `draft`, `active`, `archived` |

Create `shop_order`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `customer` | many-to-one relation to `enfyra_user` | Required |
| `status` | select | `pending`, `paid`, `fulfilled`, `canceled` |
| `totalCents` | integer | Server-derived |

Create `shop_order_item`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `order` | many-to-one relation to `shop_order` | Required |
| `product` | many-to-one relation to `shop_product` | Required |
| `quantity` | integer | Required |
| `unitPriceCents` | integer | Snapshot at checkout |

## Public Product Reads

Make `GET /shop_product` public and add a pre-hook.

```javascript
@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { status: { _eq: 'active' } }
  ]
};
```

## Create Checkout Route

Create a custom `POST /shop/checkout` route. The route handler creates the order and item rows atomically from trusted product records.

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

## Customer Order List

Add a `GET /shop_order` pre-hook so users only read their own orders.

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
