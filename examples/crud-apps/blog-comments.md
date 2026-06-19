# Blog With Comments

This example builds public post reads with authenticated comments and an admin moderation path.

## Tables

Create `blog_post`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `title` | string | Required |
| `slug` | string | Unique |
| `body` | rich text or text | Required |
| `status` | select | `draft`, `published`, `archived` |
| `publishedAt` | datetime | Optional |
| `author` | many-to-one relation to `enfyra_user` | No inverse relation required |

Create `blog_comment`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `post` | many-to-one relation to `blog_post` | Required |
| `author` | many-to-one relation to `enfyra_user` | Required |
| `body` | text | Required |
| `status` | select | `pending`, `approved`, `rejected` |

Add indexes on `blog_post.status,publishedAt` and `blog_comment.post,status,createdAt`.

## Public Post List

Make `GET /blog_post` public, then add a pre-hook that limits anonymous reads to published posts.

```javascript
if (@USER?.isRootAdmin) {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { status: { _eq: 'published' } }
  ]
};
```

## Read A Post With Approved Comments

```bash
curl "$ENFYRA_API_URL/blog_post?filter={\"slug\":{\"_eq\":\"hello-enfyra\"}}&limit=1&fields=id,title,body,publishedAt,author.email&deep={\"comments\":{\"fields\":\"id,body,createdAt,author.email\",\"filter\":{\"status\":{\"_eq\":\"approved\"}},\"sort\":\"createdAt\"}}"
```

The relation key inside `deep` must match the inverse relation name configured on `blog_comment.post`. If you did not create an inverse relation, load comments with a separate `GET /blog_comment` call filtered by `post`.

## Create A Comment

Add a `POST /blog_comment` pre-hook.

```javascript
if (!@USER?.id) {
  @THROW401();
}

if (!@BODY.post) {
  @THROW400('post is required');
}

@BODY.author = { id: @USER.id };
@BODY.status = 'pending';
```

Then call the table route.

```bash
curl "$ENFYRA_API_URL/blog_comment" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"post":{"id":12},"body":"This helped me understand hooks."}'
```

## Moderate Comments

Give moderators `PATCH /blog_comment` permission and keep normal users scoped to their own comments with a PATCH pre-hook.

```javascript
if (@USER?.isRootAdmin || @USER?.role?.name === 'moderator') {
  return;
}

@QUERY.filter = {
  _and: [
    @QUERY.filter || {},
    { author: { id: { _eq: @USER.id } } }
  ]
};
```
