# Webhook Ingest

This example receives signed provider events and stores each event once.

## Tables

Create `integration_settings` as a single-record table.

| Field | Type | Notes |
| --- | --- | --- |
| `providerWebhookSecret` | string | `isEncrypted=true`, `isPublished=false` |

Create `provider_event`.

| Field | Type | Notes |
| --- | --- | --- |
| `providerEventId` | string | Unique |
| `kind` | string | Event type |
| `payload` | simple-json | Raw event payload |
| `processedAt` | datetime | Nullable |

## Route

Create a custom `POST /webhooks/provider` route. Make it public only if the signature check is enforced by the handler.

```javascript
const settingsResult = await #integration_settings.find({
  fields: 'providerWebhookSecret',
  limit: 1
});

const settings = settingsResult.data?.[0];
if (!settings?.providerWebhookSecret) {
  @THROW500('webhook secret is not configured');
}

const signature = @REQ.headers['x-provider-signature'];
const expected = await @HELPERS.$crypto.hmacSha256(
  @REQ.rawBody,
  settings.providerWebhookSecret,
  'hex'
);

if (signature !== expected) {
  @THROW401('invalid signature');
}

const eventId = @BODY.id;
if (!eventId) {
  @THROW400('event id is required');
}

const existing = await #provider_event.find({
  filter: { providerEventId: { _eq: eventId } },
  fields: 'id',
  limit: 1
});

if (existing.data?.[0]) {
  return { received: true, duplicate: true };
}

await #provider_event.create({
  data: {
    providerEventId: eventId,
    kind: @BODY.type || 'unknown',
    payload: @BODY
  }
});

await @TRIGGER('process-provider-event', { eventId });

return { received: true };
```

Use the exact raw body when the provider signs raw request bytes.
