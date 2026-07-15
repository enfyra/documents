# Operator Queue

This example builds a queue page for records that need human action.

## Table

Create `review_case`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `title` | string | Required |
| `status` | select | `open`, `in_review`, `resolved` |
| `priority` | select | `low`, `normal`, `high` |
| `assignee` | many-to-one relation to `enfyra_user` | Optional |
| `payload` | simple-json | Optional |

## Query Pattern

Use server-side filters and bounded pagination. Do not fetch an arbitrary large limit and filter in the browser.

```javascript
const page = ref(1);
const status = ref('open');
const search = ref('');

const query = computed(() => ({
  filter: JSON.stringify({
    _and: [
      { status: { _eq: status.value } },
      search.value
        ? { title: { _contains: search.value } }
        : {}
    ]
  }),
  fields: 'id,title,status,priority,assignee.email,createdAt',
  sort: 'priority,-createdAt',
  page: page.value,
  limit: 25,
  meta: 'filterCount'
}));
```

## Claim Action

Create `POST /review-cases/claim`.

```javascript
if (!@USER?.id) {
  @THROW401();
}

const id = @BODY.id;
if (!id) {
  @THROW400('id is required');
}

const updated = await #review_case.update({
  id,
  data: {
    status: 'in_review',
    assignee: { id: @USER.id }
  }
});

return updated.data?.[0] || null;
```

Use `PermissionGate` around the claim button and require `POST /review-cases/claim` permission.
