---
slug: vi-du/ung-dung-crud/api-doc-graphql
---

# GraphQL Read API

Dùng GraphQL khi client cần một query đọc có shape rõ ràng và table đã được bật GraphQL một cách chủ động.

## Bật GraphQL

Trong admin console:

1. Mở table.
2. Bật GraphQL.
3. Lưu table.

GraphQL endpoint dùng cùng API base với REST:

```text
POST {ENFYRA_API_URL}/graphql
GET  {ENFYRA_API_URL}/graphql-schema
```

GraphQL yêu cầu Bearer token. `publicMethods` của REST không khiến GraphQL trở thành endpoint ẩn danh.

## Query công việc

```graphql
query MyTasks {
  todo_task(filter: { status: { _neq: "archived" } }, sort: "-createdAt") {
    id
    title
    status
    dueAt
    owner {
      id
      email
    }
  }
}
```

```bash
curl "$ENFYRA_API_URL/graphql" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"query MyTasks { todo_task(filter: { status: { _neq: \"archived\" } }, sort: \"-createdAt\") { id title status dueAt owner { id email } } }"}'
```

## Thay đổi một bản ghi

Tên mutation được tạo từ tên table.

```graphql
mutation CreateTask($input: todo_taskInput!) {
  create_todo_task(input: $input) {
    id
    title
    status
  }
}
```

Biến truyền vào:

```json
{
  "input": {
    "title": "Write GraphQL example",
    "status": "open"
  }
}
```

Giữ bảo mật trong hook, field permission và route permission. GraphQL là một API surface khác trên cùng table model, không thay thế kiểm soát truy cập phía server.
