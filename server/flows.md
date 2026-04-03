# Flows (Automated Workflows)

Flows are automated workflows that execute a sequence of steps based on triggers. They support scheduled execution (cron) and manual triggers. For event-driven flows, use handlers/hooks with `@DISPATCH.trigger()`.

## Tables

| Table | Purpose |
|-------|---------|
| `flow_definition` | Flow configuration (name, trigger, timeout, maxExecutions) |
| `flow_step_definition` | Steps within a flow (ordered, typed, configurable) |
| `flow_execution_definition` | Execution history and state (query separately, not nested) |

## Trigger Types

| Type | Config Example | Description |
|------|---------------|-------------|
| `schedule` | `{"cron": "0 2 * * *", "timezone": "UTC"}` | Cron-based scheduling |
| `manual` | `{}` | User-initiated via UI, API, or `@DISPATCH.trigger()` from handlers/hooks |

## Step Types

| Type | Config | Description |
|------|--------|-------------|
| `script` | `{"code": "..."}` | Execute custom JavaScript with full context |
| `condition` | `{"code": "return ..."}` | Evaluate condition using JS truthy/falsy. Truthy  `"true"` branch, falsy  `"false"` branch |
| `query` | `{"table": "...", "filter": {...}, "limit": 10}` | Query table data |
| `create` | `{"table": "...", "data": {...}}` | Create a record |
| `update` | `{"table": "...", "id": 1, "data": {...}}` | Update a record by ID |
| `delete` | `{"table": "...", "id": 1}` | Delete a record by ID |
| `http` | `{"url": "...", "method": "POST", "body": {...}, "headers": {...}, "timeout": 30000}` | Send HTTP request (auto `Content-Type: application/json` when body present). See [HTTP step URL rules](#http-step-url-rules-ssrf-hardening). |
| `trigger_flow` | `{"flowId": 2}` or `{"flowName": "..."}` | Trigger another flow |
| `sleep` | `{"ms": 5000}` | Pause execution for N ms |
| `log` | `{"message": "..."}` | Log a message to execution context |

### HTTP step URL rules (SSRF hardening)

The server only allows `http:` and `https:` URLs that are not obvious SSRF targets: `localhost` and common loopback names, raw private/reserved IP literals, and hostnames that resolve only to private IPs are rejected. Prefer **public internet hostnames** (e.g. `https://api.example.com`). Internal callbacks need a deliberate architecture change, not an arbitrary internal URL in this step.

## Template Syntax

Flow steps support the same template macros as handlers/hooks, plus flow-specific ones. Code is auto-transpiled when cached.

| Macro | Expands to | Description |
|-------|-----------|-------------|
| `@PAYLOAD` | `$ctx.$flow.$payload` | Input payload data |
| `@DISPATCH` | `$ctx.$dispatch` | Trigger flow service (handlers/hooks) |
| `@LAST` | `$ctx.$flow.$last` | Last step result |
| `@FLOW` | `$ctx.$flow` | Full flow data chain |
| `@META` | `$ctx.$flow.$meta` | Flow metadata (flowId, flowName, executionId, depth) |
| `#table_name` | `$ctx.$repos.table_name` | Table repository |
| `@HELPERS` | `$ctx.$helpers` | Helpers (jwt, bcrypt, autoSlug) |
| `@USER` | `$ctx.$user` | Current user (null for cron) |
| `@THROW4xx` | `$ctx.$throw['4xx']` | Error helpers |
| `%package` | `$ctx.$pkgs.package` | Installed packages |

## Data Chain

Each flow execution maintains a shared context (`$ctx.$flow` / `@FLOW`) that passes data between steps:

```
@FLOW
  ├── @PAYLOAD      // Input data passed to the flow
  ├── @LAST         // Result of the most recent step
  ├── @META         // { flowId, flowName, executionId, startedAt }
  ├── @FLOW.step_1  // Result of step with key "step_1"
  └── @FLOW.step_2  // Result of step with key "step_2"
```

### Accessing data in script steps (template syntax recommended)

```javascript
// Template syntax (recommended)
const email = @PAYLOAD.email;
const user = @FLOW.find_user?.data?.[0];
const prev = @LAST;
const orders = await #order_definition.find({
  filter: { userId: { _eq: user.id } }
});
return orders;
```

```javascript
// Equivalent verbose syntax (not recommended)
// $ctx.$flow.$payload.email, $ctx.$repos.order_definition, etc.
```

## Condition Branching

Condition steps support true/false branching. Child steps reference the condition via `parent` relation and `branch` field.

```
[query_users]              ← root step (parent: null)
    ↓
[check_has_users]          ← root step, type: condition
    ├── true:
    │   [process_users]    ← parent: check_has_users, branch: "true"
    │   [send_report]      ← parent: check_has_users, branch: "true"
    └── false:
        [log_empty]        ← parent: check_has_users, branch: "false"
    ↓
[cleanup]                  ← root step (continues after condition)
```

Creating branch steps via API:
```
POST /api/flow_step_definition
{
  "flow": { "id": 1 },
  "key": "process_users",
  "stepOrder": 1,
  "type": "script",
  "config": { "code": "return #user_definition.find({ limit: 100 })" },
  "parent": { "id": 5 },
  "branch": "true"
}
```

Rules:
- Steps with `parent: null` are root level — execute sequentially
- Condition uses JS truthy/falsy: `return user` (object = truthy), `return null` (falsy), `return count > 0` (boolean)
- Truthy  children with `branch: "true"` execute
- Falsy  children with `branch: "false"` execute
- After condition + branch children complete  next root step continues
- Branch steps can have `onError: skip/stop/retry` independently

## Error Handling

| Strategy | Behavior |
|----------|----------|
| `stop` | Halt flow, mark execution as failed (default) |
| `skip` | Record error, continue to next step |
| `retry` | Retry with exponential backoff up to `retryAttempts` times |

## Admin Endpoints

| Endpoint | Description |
|----------|-------------|
| `POST /admin/flow/trigger/:id` | Trigger a flow execution via BullMQ |
| `POST /admin/flow/test-step` | Test a single step without saving (body: `{type, config, timeout}`) |

## Triggering Flows from Handlers

```javascript
await @DISPATCH.trigger('send-welcome-email', { userId: @USER.id });
await @DISPATCH.trigger(5, { orderId: @PARAMS.id, total: 100 });
```

## Execution History

Query execution records separately (not nested under flow):

```
GET /api/flow_execution_definition?filter={"flow":{"_eq":1}}&sort=-id&limit=10
```

Each execution record contains:
- `status`: pending, running, completed, failed, cancelled
- `currentStep`: which step the flow stopped or is running at
- `completedSteps`: array of step keys that completed successfully
- `error`: error details if failed (message + stack)
- `duration`: total execution time in ms

## Example: Order Processing Flow

### 1. Create the flow

```
POST /api/flow_definition
{
  "name": "process-order",
  "triggerType": "manual",
  "timeout": 30000,
  "isEnabled": true
}
```

Then trigger from a post-hook on `/order_definition`:
```javascript
await @DISPATCH.trigger('process-order', { data: @DATA });
```

### 2. Add steps

```
POST /api/flow_step_definition
{
  "flow": { "id": 1 },
  "key": "validate_stock",
  "stepOrder": 1,
  "type": "script",
  "config": {
    "code": "const order = @PAYLOAD.data; const product = await #product_definition.find({ filter: { id: { _eq: order.productId } }, limit: 1 }); return { inStock: product.data[0]?.stock > order.quantity }"
  },
  "timeout": 5000,
  "onError": "stop"
}
```

### 3. Test a step before saving

```
POST /api/admin/flow/test-step
{
  "type": "query",
  "config": { "table": "user_definition", "filter": { "status": { "_eq": "active" } }, "limit": 5 },
  "timeout": 5000
}
```

Response: `{ "success": true, "result": { "data": [...] }, "duration": 42 }`

### 4. Trigger manually

```
POST /api/admin/flow/trigger/1
{ "payload": { "orderId": 123 } }
```

### 5. View execution history

```
GET /api/flow_execution_definition?filter={"flow":{"_eq":1}}&sort=-id&limit=10
```

## Scheduled Flow Example

```
POST /api/flow_definition
{
  "name": "daily-cleanup",
  "triggerType": "schedule",
  "triggerConfig": { "cron": "0 2 * * *", "timezone": "Asia/Ho_Chi_Minh" },
  "timeout": 60000,
  "isEnabled": true
}
```

Cron schedules are registered automatically via BullMQ Job Schedulers when the flow cache reloads.

## Safety & Limits

| Feature | Detail |
|---------|--------|
| Max nesting depth | 10 (flow triggering flow via `trigger_flow` step or `$dispatch.trigger()`) |
| Circular detection | Tracks visited flow IDs in chain — ABA rejected immediately |
| HTTP timeout | 30s default, configurable via `config.timeout` per step |
| Execution history | Auto-cleanup per flow via `maxExecutions` (default 100, oldest deleted) |
| Step timeout | Per-step or inherited from flow `timeout` (default 5000ms) |
| Retry backoff | Exponential: 1s, 2s, 4s, 8s... capped at 30s |

## Triggering Flows from Flow Steps

Flow steps have `$dispatch.trigger()` available, same as handlers/hooks:

```javascript
// Inside a script step
const result = await @DISPATCH.trigger('send-notification', { userId: @PAYLOAD.userId });
```

This respects the same nesting depth and circular detection limits.
