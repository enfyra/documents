# Data & CRUD

Use the fluent Query Builder for route-backed Enfyra tables. It handles field selection, filters, sorting, pagination, relation queries, metadata, and ID-scoped mutations.

## Define a Record Type

```ts
interface Article {
  id: number
  title: string
  status: 'draft' | 'published'
  author?: {
    id: number
    name: string
  }
  createdAt: string
}
```

## Read a Collection

```ts
const { data, meta } = await enfyra
  .from<Article>('articles')
  .select(['id', 'title', 'status', 'createdAt'])
  .filter({ status: { _eq: 'published' } })
  .sort('-createdAt')
  .limit(20)
  .page(1)
  .meta(['totalCount', 'filterCount'])
  .execute()
```

`sort('-createdAt')` sorts descending. Use `sort('createdAt')` for ascending order.

## Filter Records

Common operators include:

```ts
const result = await enfyra
  .from<Article>('articles')
  .filter({
    _and: [
      { status: { _eq: 'published' } },
      { createdAt: { _gte: '2026-01-01T00:00:00.000Z' } },
      {
        _or: [
          { title: { _icontains: 'enfyra' } },
          { category: { _in: ['guide', 'release'] } },
        ],
      },
    ],
  })
  .execute()
```

Supported SDK filter operators are `_eq`, `_neq`, `_gt`, `_gte`, `_lt`, `_lte`, `_in`, `_not_in`, `_contains`, `_icontains`, `_starts_with`, `_ends_with`, `_is_null`, `_is_not_null`, `_between`, `_and`, `_or`, and `_not`.

Use only fields and operators supported by the target route and schema.

## Read Relations

Select relation fields using dotted field names and pass deep query configuration when the related collection needs its own filtering or limit:

```ts
const result = await enfyra
  .from<Article>('articles')
  .select(['id', 'title', 'author.id', 'author.name', 'comments.id', 'comments.body'])
  .deep({
    comments: {
      _limit: 5,
      _sort: '-createdAt',
      _filter: { isApproved: { _eq: true } },
    },
  })
  .execute()
```

Relation property names come from Enfyra metadata. Do not guess physical foreign-key column names.

## Read One Record

```ts
const { data } = await enfyra
  .from<Article>('articles')
  .byId(articleId)
  .select(['id', 'title', 'status'])
  .execute()
```

`byId()` adds an ID filter and sets the request limit to one. The returned `data` follows the API response shape, so narrow it deliberately in your application if you require one object.

## Transform Query Data

Use `.transform()` for a transformation local to one Query Builder execution:

```ts
const { data } = await enfyra
  .from<Article>('articles')
  .select(['id', 'title'])
  .transform((value) => {
    const rows = Array.isArray(value) ? value : [value]
    return rows.map((article) => ({
      ...article,
      title: article.title.trim(),
    }))
  })
  .execute()
```

The function receives the query `data`, not the full HTTP response.

## Create a Record

```ts
const created = await enfyra.from<Article>('articles').insert({
  title: 'SDK guide',
  status: 'draft',
})

console.log(created.data)
```

## Update a Record

```ts
const updated = await enfyra
  .from<Article>('articles')
  .byId(articleId)
  .update({
    title: 'Updated SDK guide',
    status: 'published',
  })
```

Calling `update()` without `byId()` throws a `VALIDATION_ERROR` before any request is sent.

## Delete a Record

```ts
await enfyra.from('articles').byId(articleId).delete()
```

Calling `delete()` without `byId()` is rejected before any request is sent.

## Mutate Relations

Send relation values using the relation property name exposed by Enfyra metadata:

```ts
await enfyra.from<Article>('articles').byId(articleId).update({
  author: { id: authorId },
  tags: [{ id: firstTagId }, { id: secondTagId }],
})
```

The accepted relation shape depends on its cardinality and server metadata. Verify it against the table schema and generated API behavior.

## Pagination Checklist

- Request `meta(['totalCount'])` when the UI must show the total collection size.
- Request `meta(['filterCount'])` when the UI must show the size after filters.
- Keep `limit` bounded for user-facing lists.
- Reset `page` when a filter changes.
- Select only the fields needed by the current screen.

## Troubleshooting

- 403: the route, role, user, owner, or field permission rejected the operation.
- Empty fields: the field may not be selected or may be hidden by field permissions.
- Unknown relation: use the metadata relation property name, not a guessed database column.
- Update/delete validation error: call `.byId(id)` first.
- Custom business action: use a custom POST route instead of forcing it into table CRUD.
