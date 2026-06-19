# Public Contact Form

This example accepts anonymous website submissions without exposing the inbox publicly.

## Table

Create `contact_request`.

| Field | Type | Notes |
| --- | --- | --- |
| `email` | string | Add an email format rule |
| `name` | string | Optional |
| `message` | text | Required |
| `status` | select | `new`, `reviewing`, `closed` |
| `source` | string | Optional |

Make `POST /contact_request` public. Keep `GET`, `PATCH`, and `DELETE` protected for operators only.

## Validate And Normalize

Add a `POST /contact_request` pre-hook.

```javascript
const email = typeof @BODY.email === 'string' ? @BODY.email.trim().toLowerCase() : '';
const message = typeof @BODY.message === 'string' ? @BODY.message.trim() : '';

if (!email || !message) {
  @THROW400('email and message are required');
}

@BODY.email = email;
@BODY.message = message;
@BODY.status = 'new';
@BODY.source = @BODY.source || 'website';
```

For standard email validation, add a column rule on `email` with format `email`.

## Rate Limit Submissions

Add this to the same pre-hook or a dedicated pre-hook.

```javascript
const result = await @HELPERS.$rateLimit.byIp({
  maxRequests: 5,
  perSeconds: 300
});

if (!result.allowed) {
  @THROW429(`Too many submissions. Try again in ${result.retryAfter}s`);
}
```

## Submit From A Website

```bash
curl "$ENFYRA_API_URL/contact_request" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"ada@example.com","name":"Ada","message":"Can we talk about Enfyra Cloud?","source":"landing"}'
```

## Operator Inbox

Use the admin data table or a small admin extension to read:

```text
GET /contact_request?filter={"status":{"_eq":"new"}}&sort=-createdAt&fields=id,email,name,message,status,createdAt
```
