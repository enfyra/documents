# Query Parameters

These parameters apply to **GET** requests for list endpoints (e.g. `GET {appUrl}/api/enfyra_user`).

## fields

Select which fields to return. Comma-separated list.

**Example:**
```
GET {appUrl}/api/enfyra_user?fields=id,email,name,role.name
```

- Use dot notation for related fields: `role.name`
- Use `*` for a relation to get all its fields: `role.*`
- Omit to return all fields (default)
- Prefix a field with `-` to switch that `fields` scope to exclude mode: `fields=-compiledCode` returns all readable fields except `compiledCode`
- If any field is excluded, positive field names in the same scope are ignored: `fields=id,-compiledCode` still returns all readable fields except `compiledCode`
- Nested exclusions use the same rule: `fields=-role.avatar` excludes `avatar` from the related `role` object

---

## filter

MongoDB-like filter object. Pass as JSON in the query string.

**Example:**
```
GET {appUrl}/api/enfyra_user?filter={"email":{"_contains":"@example.com"}}
```

**Common operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| _eq | Equal | `{"status":{"_eq":"active"}}` |
| _neq | Not equal | `{"status":{"_neq":"deleted"}}` |
| _gt | Greater than | `{"price":{"_gt":100}}` |
| _gte | Greater than or equal | `{"price":{"_gte":100}}` |
| _lt | Less than | `{"price":{"_lt":500}}` |
| _lte | Less than or equal | `{"price":{"_lte":500}}` |
| _in | In array | `{"category":{"_in":["a","b"]}}` |
| _not_in | Not in array | `{"status":{"_not_in":["archived"]}}` |
| _contains | Contains text (case-insensitive) | `{"name":{"_contains":"john"}}` |
| _starts_with | Starts with | `{"email":{"_starts_with":"admin"}}` |
| _ends_with | Ends with | `{"email":{"_ends_with":"@example.com"}}` |
| _is_null | Is null | `{"deletedAt":{"_is_null":true}}` |
| _is_not_null | Is not null | `{"description":{"_is_not_null":true}}` |
| _between | Between (inclusive) | `{"price":{"_between":[100,500]}}` |
| _and | All conditions | `{"_and":[{"a":{"_eq":1}},{"b":{"_eq":2}}]}` |
| _or | Any condition | `{"_or":[{"status":{"_eq":"active"}},{"status":{"_eq":"pending"}}]}` |
| _not | Negate | `{"_not":{"status":{"_eq":"archived"}}}` |

**Filter by relation:**
```
GET {appUrl}/api/order?filter={"customer":{"name":{"_contains":"Smith"}}}
```

See [Query Filtering](../server/query-filtering.md) for full reference.

---

## sort

Sort order for parent/root records. Comma-separated fields. Prefix with `-` for descending.

**Examples:**
```
?sort=name            name ascending
?sort=-createdAt      createdAt descending
?sort=category,-price  category asc, then price desc
?sort=-_max(messages.createdAt)  parent rows by latest child message
?sort=-_count(messages)          parent rows by child count
```

**Notes:**
- If no `sort` is specified, results are sorted by `id` ascending
- Sort applies only to the parent table, not nested relations
- Nested arrays (one-to-many, many-to-many) are always sorted by `id` internally
- `createdAt`, `updatedAt`, and scalar `date`, `datetime`, or `timestamp` columns receive auto-generated single-field indexes. SQL adds `id` as the stable tie-breaker and Mongo adds `_id`.
- Add explicit indexes for compound hot paths, such as `status + createdAt`, but do not duplicate single-field time indexes.
- Use `_count(relationName)`, `_max(relationName.fieldName)`, or `_min(relationName.fieldName)` to sort parent records by a direct `one-to-many` or `many-to-many` relation aggregate
- Do not use raw dotted to-many sort such as `messages.createdAt`; it is ambiguous and rejected
- To sort nested relations, use the `deep` parameter with a `sort` option:

```
GET /api/enfyra_user?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt"}}
```

The `deep` sort above orders `posts` inside each user. It does not sort the user list.

---

## limit

Maximum number of records to return. Default is 10.

**Examples:**
```
?limit=20
?limit=0    No limit (return all matching records)
```

---

## page

Page number for pagination (1-based). Use with `limit`.

**Example:**
```
?page=2&limit=20   Records 21–40
```

---

## meta

Request metadata (e.g. total count). Comma-separated.

**Values:**
- `totalCount` – Total records in table (ignores filter)
- `filterCount` – Records matching the filter

**Example:**
```
GET {appUrl}/api/enfyra_user?limit=10&meta=totalCount,filterCount
```

**Response:**
```json
{
  "data": [ ... ],
  "meta": {
    "totalCount": 150,
    "filterCount": 45
  }
}
```

---

## deep

Nested relations with filters, sort, and limit per level. Pass as JSON.

**Example:**
```
GET {appUrl}/api/enfyra_user?fields=id,name&deep={"posts":{"fields":"id,title","sort":"-createdAt","limit":5}}
```

Each `deep.<relation>.fields` value also supports exclude mode:

```
GET {appUrl}/api/enfyra_user?fields=id,name&deep={"posts":{"fields":"-compiledCode,-author.avatar","limit":5}}
```

See [Deep Queries](../server/query-filtering.md#deep-queries-nested-relations) for details.

---

## Combined Example

```
GET {appUrl}/api/product?filter={"category":{"_eq":"electronics"},"price":{"_gte":100}}&fields=id,name,price,category.name&sort=-price&limit=20&page=1&meta=filterCount
```
