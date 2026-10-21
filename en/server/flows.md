# Flows (Automated Workflows)

Flows are automated workflows that execute a sequence of steps based on triggers. They support scheduled execution (cron) and manual triggers. For event-driven flows, use handlers/hooks with `@TRIGGER()`.

## Tables

| Table | Purpose |
|-------|---------|
| `enfyra_flow` | Flow configuration (name, trigger, timeout, maxExecutions) |
| `enfyra_flow_step` | Steps within a flow (ordered, typed script/condition steps) |
| `enfyra_flow_execution` | Execution history and state (query separately, not nested) |

## Trigger Types

| Type | Config Example | Description |
|------|---------------|-------------|
| `schedule` | `{"cron": "0 2 * * *", "timezone": "UTC"}` | Cron-based scheduling |
| `manual` | `{}` | User-initiated via UI, API, or `@TRIGGER()` from handlers/hooks |

## Step Types

| Type | Description |
|------|-------------|
| `script` | Execute custom JavaScript/TypeScript with full context. `sourceCode` is the only step input. |
| `condition` | Evaluate a script that must return a boolean. `true` runs the `"true"` branch, `false` runs the `"false"` branch; any other result fails the step. |

Steps are script-only: there is no step `config`. Do the work inside `sourceCode`.

| Need | Script |
|------|--------|
| Query | `return await #table.find({ filter, fields, limit, sort })` |
| Create | `return await #table.create({ data })` |
| Update | `return await #table.update({ id, data })` |
| Delete | `return await #table.delete({ id })` |
| External HTTP | `return await @FETCH(url, options)` |
| Delay | `await @HELPERS.$sleep(5000)` |
| Child flow | `return await @TRIGGER('flow-name', payload)` |
| Execution log | `@LOGS(message, details?)` |

A script step reaches the network through `@FETCH` (and packages), which keep their own private-IP and redirect guards; there is no separate fixed-HTTP step contract to configure.

## Template Syntax

Flow steps support the same template macros as handlers/hooks, plus flow-specific ones. Code is auto-transpiled when cached.

| Macro | Expands to | Description |
|-------|-----------|-------------|
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` | Input payload data |
| `@TRIGGER` | `$ctx.$trigger` | Trigger flow from handlers/hooks |
| `@FLOW_LAST` | `$ctx.$flow.$last` | Last step result |
| `@FLOW` | `$ctx.$flow` | Full flow data chain |
| `@FLOW_META` | `$ctx.$flow.$meta` | Flow metadata (flowId, flowName, executionId, depth) |
| `#table_name` | `$ctx.$repos.table_name` | Table repository |
| `@HELPERS` | `$ctx.$helpers` | Helpers (jwt, bcrypt, autoSlug) |
| `@USER` | `$ctx.$user` | Current user (null for cron) |
| `@THROW400(message)` … `@THROW503(message)`, `@THROW.http(statusCode, message?)` | `$ctx.$throw.http(fixedStatus, message)`, `$ctx.$throw.http(statusCode, message?)` | Quick generic HTTP errors; fixed-status helpers require one message |
| `%package` | `$ctx.$pkgs.package` | Installed packages |

## Data Chain

Each flow execution maintains a shared context (`$ctx.$flow` / `@FLOW`) that passes data between steps:

```
@FLOW
  ├── @FLOW_PAYLOAD      // Input data passed to the flow
  ├── @FLOW_LAST         // Result of the most recent step
  ├── @FLOW_META         // { flowId, flowName, executionId, startedAt }
  ├── @FLOW.step_1  // Result of step with key "step_1"
  └── @FLOW.step_2  // Result of step with key "step_2"
```

### Accessing data in script steps (template syntax recommended)

```javascript
// Template syntax (recommended)
const email = @FLOW_PAYLOAD.email;
const user = @FLOW.find_user?.data?.[0];
const prev = @FLOW_LAST;
const orders = await #order.find({
  filter: { userId: { _eq: user.id } }
});
return orders;
```

```javascript
// Equivalent verbose syntax (not recommended)
// $ctx.$flow.$payload.email, $ctx.$repos.order, etc.
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
POST /api/enfyra_flow_step
{
  "flow": { "id": 1 },
  "key": "process_users",
  "stepOrder": 1,
  "type": "script",
  "sourceCode": "return await #enfyra_user.find({ limit: 100 })",
  "parent": { "id": 5 },
  "branch": "true"
}
```

The condition step itself returns the boolean that picks the branch:

```
POST /api/enfyra_flow_step
{
  "flow": { "id": 1 },
  "key": "check_has_users",
  "stepOrder": 1,
  "type": "condition",
  "sourceCode": "const rows = await #enfyra_user.find({ limit: 1 }); return rows.data.length > 0"
}
```

Rules:
- Steps with `parent: null` are root level — execute sequentially
- Condition must return a real boolean: `return count > 0`, `return Boolean(payload.enabled)`. Returning an object, array, string, number, `null`, or `undefined` fails the step instead of branching.
- `true` children with `branch: "true"` execute
- `false` children with `branch: "false"` execute
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
| `POST /admin/test/run` | Test a flow step without saving by sending `kind: "flow_step"` |

## Triggering Flows from Handlers

```javascript
await @TRIGGER('send-welcome-email', { userId: @USER.id });
await @TRIGGER(5, { orderId: @PARAMS.id, total: 100 });
```

## Execution History

Query execution records separately (not nested under flow):

```
GET /api/enfyra_flow_execution?filter={"flow":{"_eq":1}}&sort=-id&limit=10
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
POST /api/enfyra_flow
{
  "name": "process-order",
  "triggerType": "manual",
  "timeout": 30000,
  "isEnabled": true
}
```

Then trigger from a post-hook on `/order`:
```javascript
await @TRIGGER('process-order', { data: @DATA });
```

### 2. Add steps

```
POST /api/enfyra_flow_step
{
  "flow": { "id": 1 },
  "key": "validate_stock",
  "stepOrder": 1,
  "type": "script",
  "sourceCode": "const order = @FLOW_PAYLOAD.data; const product = await #product.find({ filter: { id: { _eq: order.productId } }, limit: 1 }); return { inStock: product.data[0]?.stock > order.quantity }",
  "timeout": 5000,
  "onError": "stop"
}
```

### 3. Test a step before saving

```
POST /api/admin/test/run
{
  "kind": "flow_step",
  "type": "script",
  "sourceCode": "return await #enfyra_user.find({ filter: { status: { _eq: 'active' } }, limit: 5 })",
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
GET /api/enfyra_flow_execution?filter={"flow":{"_eq":1}}&sort=-id&limit=10
```

## Scheduled Flow Example

```
POST /api/enfyra_flow
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
| Max nesting depth | 10 (flow triggering flow via `@TRIGGER()` or `$trigger()`) |
| Circular detection | Tracks visited flow IDs in chain — ABA rejected immediately |
| HTTP timeout | `@FETCH` defaults to 8s per call; pass `timeoutMs` per request |
| Execution history | Auto-cleanup per flow via `maxExecutions` (default 100, oldest deleted) |
| Step timeout | Per-step or inherited from flow `timeout` (default 5000ms) |
| Retry backoff | Exponential: 1s, 2s, 4s, 8s... capped at 30s |

## Triggering Flows from Flow Steps

Flow steps have `$trigger()` available, same as handlers/hooks:

```javascript
// Inside a script step
const result = await @TRIGGER('send-notification', { userId: @FLOW_PAYLOAD.userId });
```

This respects the same nesting depth and circular detection limits.
