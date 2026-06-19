# Rate-Limited Public API

This example protects a public quote endpoint without requiring login.

## Route

Create a custom `GET /public/quote` route and make `GET` public.

## Pre-Hook

```javascript
const result = await @HELPERS.$rateLimit.byIp({
  maxRequests: 60,
  perSeconds: 60
});

if (!result.allowed) {
  @THROW429(`Rate limit exceeded. Try again in ${result.retryAfter}s`);
}
```

## Handler

```javascript
const quotes = await #quote.find({
  filter: { status: { _eq: 'published' } },
  fields: 'id,text,author',
  sort: '-createdAt',
  limit: 10
});

return {
  data: quotes.data?.[Math.floor(Math.random() * quotes.data.length)] || null
};
```

Keep the pre-hook as the protection boundary. For standard route-wide limits, use declarative Guards from the admin console.
