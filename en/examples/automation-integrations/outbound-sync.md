# Outbound Sync

This example sends approved records to another API after they are saved.

## Settings Table

Create `sync_settings` as a single-record table.

| Field | Type | Notes |
| --- | --- | --- |
| `endpointUrl` | string | Destination URL |
| `apiKey` | string | `isEncrypted=true`, `isPublished=false` |

## Post-Hook

Attach this post-hook to `PATCH /blog_comment`.

```javascript
if (@ERROR || !@DATA || @DATA.status !== 'approved') {
  return;
}

const settingsResult = await #sync_settings.find({
  fields: 'endpointUrl,apiKey',
  limit: 1
});

const settings = settingsResult.data?.[0];
if (!settings?.endpointUrl || !settings?.apiKey) {
  return;
}

const response = await fetch(settings.endpointUrl, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${settings.apiKey}`
  },
  body: JSON.stringify({
    id: @DATA.id,
    body: @DATA.body,
    approvedAt: new Date().toISOString()
  })
});

if (!response.ok) {
  @THROW502('external sync failed');
}
```

For slow or unreliable destinations, trigger a flow instead of doing the HTTP request directly inside the post-hook.
