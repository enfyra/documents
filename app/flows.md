# Flows - UI Guide

Flows are managed from **Settings > Flows** in the sidebar.

## Flow List

The list page shows all flows as cards with:
- Flow name and description
- Trigger type badge (Schedule, Manual)
- Status toggle (Active/Inactive)
- Step count and timeout
- Click a card to open the flow editor

**Actions:**
- **Create Flow** button (top right) — opens create form
- **Toggle switch** on each card — enable/disable flow without opening
- **Delete** button — remove flow (with confirmation)

## Create Flow

The create form collects:
- **Name** — unique flow identifier
- **Description** — what the flow does
- **Trigger Type** — select from Schedule, Manual
- **Trigger Config** — auto-generated fields based on trigger type
- **Timeout** — max execution time in ms

After saving, you are redirected to the flow editor.

## Flow Editor

The editor page has three sections:

### Flow Settings (top)

Edit flow name, description, trigger type, trigger config, and timeout. The trigger config section changes dynamically:
- **Schedule**: Cron expression input + timezone dropdown
- **Manual**: No config needed. Trigger via "Run Now", API, or `@TRIGGER()` from handlers/hooks

Click **Save** in the header to persist changes.

### Flow Steps (canvas)

Visual flow canvas powered by Vue Flow showing:
- **Trigger node** (amber) at the top — shows trigger type and config
- **Root step nodes** connected vertically in center — sequential execution
- **Condition nodes** split into two branches:
  - **True branch** (right side, green edges) — steps that run when condition is true
  - **False branch** (left side, red edges) — steps that run when condition is false
- **Branch badges** on step nodes show which branch they belong to (green "true" / red "false")
- **"Add Step" node** (dashed) at the bottom — click to add a new step

**Click any step node** to open the step editor drawer.

### Execution History (bottom)

Shows recent flow runs with:
- Status badge (pending, running, completed, failed, cancelled)
- Start time
- Duration
- **Reload** button to refresh the list
- **Click any execution** to open detail drawer showing:
  - Status, duration, start/complete times
  - Which step the flow stopped at (with reason if failed)
  - List of completed steps
  - Error message and stack trace if failed

**Run Now** button in the sub-header triggers the flow manually.

## Step Editor Drawer

Opens when clicking a step node or "Add Step". Fields:

- **Key** — unique identifier used in data chain (`@FLOW.<key>`)
- **Order** — execution sequence number
- **Timeout** — per-step timeout in ms
- **Type** — select step type (each shows different config fields)
- **Config** — type-specific form:
  - **Script/Condition**: Code editor (JavaScript) — use template syntax: `@FLOW_PAYLOAD`, `@FLOW_LAST`, `@FLOW.<key>`, `#table_name`, `@HELPERS`
  - **Query**: Table picker (autocomplete) + Filter Builder (visual) + Limit + Fields
  - **Create**: Table picker + Data JSON
  - **Update**: Table picker + Record ID + Data JSON
  - **Delete**: Table picker + Record ID
  - **HTTP**: URL + Method dropdown + Headers JSON + Body JSON
  - **Trigger Flow**: Flow name or ID
  - **Sleep**: Duration in ms
  - **Log**: Message text
- **On Error** — Stop flow / Skip step / Retry
- **Retries** — number of retry attempts (shown when On Error = Retry)
- **Parent Condition** — dropdown to assign step to a condition's branch (shown when flow has condition steps)
- **Branch** — True or False (shown when Parent Condition selected)

### Test Button

Click **Test** to run the step immediately without saving. The result appears inline:
- **Success**: green checkmark + duration + result preview (read-only)
- **Failure**: red X + error message

This lets you verify step logic before committing changes.

### Actions
- **Delete** — remove the step (shown only when editing existing step)
- **Cancel** — close drawer without saving
- **Create/Update** — save the step

## URL State

Opening a step drawer or execution detail adds query parameters to the URL (`?editStep=...` or `?exec=...`). This means:
- Browser back button closes the drawer
- Direct links to specific step editors work
- Page refresh preserves drawer state

## Table Picker

Used in Query, Create, Update, Delete step configs. Features:
- Autocomplete input (type to search)
- Server-side search with 500ms debounce using `_contains` filter
- Loads 10 results at a time
- Loading indicator during search

## Template Syntax Quick Reference

Use these macros in Script and Condition step code. They are auto-transpiled to `$ctx` syntax when saved.

| Write this | Gets converted to |
|-----------|-------------------|
| `@FLOW_PAYLOAD` | `$ctx.$flow.$payload` |
| `@TRIGGER` | `$ctx.$trigger` (trigger flows from handlers) |
| `@FLOW_LAST` | `$ctx.$flow.$last` |
| `@FLOW.step_key` | `$ctx.$flow.step_key` |
| `@FLOW_META` | `$ctx.$flow.$meta` |
| `#user_definition` | `$ctx.$repos.user_definition` |
| `@HELPERS` | `$ctx.$helpers` |
| `@TRIGGER(...)` | Trigger another flow from handler |
| `@USER` | `$ctx.$user` |
| `@THROW404(msg)` | `$ctx.$throw['404'](msg)` |
| `%dayjs` | `$ctx.$pkgs.dayjs` |

### Example

```javascript
// In a script step:
const user = await #user_definition.find({
  filter: { email: { _eq: @FLOW_PAYLOAD.email } },
  limit: 1
});
if (!user.data[0]) @THROW404('User not found');
return user.data[0];
```
