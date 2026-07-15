# GraphQL Read API

Use GraphQL when a client wants one strongly shaped read query and the table is explicitly enabled for GraphQL.

## Enable GraphQL

In the admin console:

1. Open the table.
2. Enable GraphQL.
3. Save the table.

The GraphQL endpoint uses the same API base as REST:

```text
POST {ENFYRA_API_URL}/graphql
GET  {ENFYRA_API_URL}/graphql-schema
```

GraphQL requires a Bearer token. REST `publicMethods` does not make GraphQL anonymous.

## Query Tasks

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

## Mutate A Record

Mutation names are generated from the table name.

```graphql
mutation CreateTask($input: todo_taskInput!) {
  create_todo_task(input: $input) {
    id
    title
    status
  }
}
```

Variables:

```json
{
  "input": {
    "title": "Write GraphQL example",
    "status": "open"
  }
}
```

Keep security in hooks, field permissions, and route permissions. GraphQL is another API surface over the same table model, not a replacement for server-side access control.
