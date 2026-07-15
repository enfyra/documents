# Build Your First App

This guide gives you one complete result before you learn every Enfyra feature: a `task` collection, a record in the admin app, and a generated REST API you can call.

**Time:** about 15 minutes.  
**You need:** a running Enfyra app and an admin account. If you do not have one yet, use [Installation](./installation.md).

## What You Will Build

```text
task collection
  ├── title: text
  ├── done: true/false
  └── generated REST API at /api/task
```

At the end, you will see a task in **Data > task** and receive it from `GET /api/task`.

## 1. Sign In

1. Open your app URL, usually `http://localhost:3000`.
2. Sign in with the admin credentials you set during installation.
3. Confirm that you can see **Collections** and **Data** in the sidebar.

If you are using the Docker quick-start defaults only for local evaluation, sign in and then replace the default credentials before deploying the instance anywhere public.

## 2. Create The Collection

1. Go to **Collections** and choose **Create**.
2. Set the collection name to `task`.
3. Add a column named `title`:
   - Type: `varchar`
   - Nullable: off
4. Add a column named `done`:
   - Type: `boolean`
   - Default value: `false`
5. Save the collection.

Enfyra adds the primary key automatically. After saving, it creates the physical schema and the REST route. REST is ready immediately; GraphQL is a separate per-table opt-in.

**Checkpoint:** `task` appears under **Data** in the sidebar. If it does not, refresh once and check that the collection saved without a validation error.

For every available column type, relation shape, index, or delete behavior, continue with [Table Creation](./table-creation.md) after this guide.

## 3. Add A Record In The Admin App

1. Open **Data > task**.
2. Select **Create**.
3. Enter `Plan the first release` as `title` and leave `done` unchecked.
4. Select **Save**.

**Checkpoint:** the new record appears in the task list. You now have a collection and data managed through Enfyra.

## 4. Verify The Generated API

For browser applications, use the app URL and its `/api` prefix. The app proxies the request to the Enfyra server, so you normally do not need to expose or call port `1105` directly.

Open this URL while still signed in:

```text
http://localhost:3000/api/task?fields=id,title,done&limit=10
```

You should receive a JSON response whose `data` contains `Plan the first release`.

To verify from a token-managed client, use the login and bearer-token flow in [API Reference](../api-reference/README.md). The generated CRUD contract is:

```text
GET    /api/task                 list tasks
POST   /api/task                 create a task
PATCH  /api/task/:id             update a task
DELETE /api/task/:id             delete a task
```

To retrieve one record, use `GET /api/task?filter={"id":{"_eq":1}}&limit=1`; dynamic table routes do not provide `GET /api/task/:id`.

## 5. Choose Your Next Step

- Need relations, validation, encryption, indexes, or GraphQL? Read [Table Creation](./table-creation.md).
- Need to manage records and filters in the admin app? Read [Data Management](./data-management.md).
- Need to restrict who can call the route? Read [Routing Management](../app/routing-management.md) and [Permission Builder](../app/permission-builder.md).
- Need to connect another frontend, mobile app, or server? Read [API Reference](../api-reference/README.md).
- Need business rules or automation? Read [Hooks and Handlers](../server/hooks-handlers/README.md) or [Flows](../server/flows.md).
