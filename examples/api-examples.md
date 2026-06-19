# API Examples

Small REST examples for generated Enfyra table routes.

Replace `{appUrl}` with your app URL, for example `http://localhost:3000`.

## Authentication

### Login

```bash
curl -X POST "{appUrl}/api/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"password"}'
```

### Current User

```bash
curl "{appUrl}/api/me" \
  -H "Authorization: Bearer <accessToken>"
```

### Logout

```bash
curl -X POST "{appUrl}/api/logout" \
  -H "Authorization: Bearer <accessToken>"
```

## Records

### List Rows

```http
GET {appUrl}/api/post?fields=id,title,createdAt&limit=20
```

### Find One Row By Id

```http
GET {appUrl}/api/post?filter={"id":{"_eq":123}}&limit=1
```

### Create Row

```bash
curl -X POST "{appUrl}/api/post" \
  -H "Authorization: Bearer <accessToken>" \
  -H "Content-Type: application/json" \
  -d '{"title":"Hello","status":"draft"}'
```

### Update Row

```bash
curl -X PATCH "{appUrl}/api/post/123" \
  -H "Authorization: Bearer <accessToken>" \
  -H "Content-Type: application/json" \
  -d '{"status":"published"}'
```

### Delete Row

```bash
curl -X DELETE "{appUrl}/api/post/123" \
  -H "Authorization: Bearer <accessToken>"
```

## Query

### Equality Filter

```http
GET {appUrl}/api/post?filter={"status":{"_eq":"published"}}
```

### Contains Filter

```http
GET {appUrl}/api/post?filter={"title":{"_contains":"release"}}
```

### Relation Filter

```http
GET {appUrl}/api/post?filter={"author":{"id":{"_eq":7}}}
```

### Sort

```http
GET {appUrl}/api/post?sort=-createdAt
```

### Count Rows

```http
GET {appUrl}/api/post?fields=id&limit=1&meta=totalCount
```

### Count Filtered Rows

```http
GET {appUrl}/api/post?fields=id&limit=1&meta=filterCount&filter={"status":{"_eq":"published"}}
```

## Fields

### Select Fields

```http
GET {appUrl}/api/post?fields=id,title,author.email
```

### Exclude A Large Field

```http
GET {appUrl}/api/enfyra_route_handler?fields=-compiledCode
```

### Exclude Nested Field

```http
GET {appUrl}/api/post?fields=-author.avatar
```

## Relations

### Load Related Fields

```http
GET {appUrl}/api/post?fields=id,title,author.id,author.email
```

### Deep Load Children

```http
GET {appUrl}/api/post?fields=id,title&deep={"comments":{"fields":"id,body","limit":5}}
```

### Deep Load With Nested Exclusion

```http
GET {appUrl}/api/post?fields=id,title&deep={"comments":{"fields":"-compiledCode,-author.avatar","limit":5}}
```

### Sort Parent Rows By Child Count

```http
GET {appUrl}/api/post?fields=id,title&sort=-_count(comments)
```

### Sort Parent Rows By Latest Child

```http
GET {appUrl}/api/post?fields=id,title&sort=-_max(comments.createdAt)
```
