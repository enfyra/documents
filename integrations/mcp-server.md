# MCP Server

Use `@enfyra/mcp-server` to connect MCP-compatible coding tools to an Enfyra instance. The server exposes focused tools for metadata discovery, schema changes, routes, handlers, hooks, permissions, flows, websocket events, files, packages, menus, extensions, logs, and runtime checks.

## Install And Configure

From the project you want the coding agent to work in:

```bash
npx @enfyra/mcp-server config
```

The config helper writes project-local MCP config for Codex, Claude Code, Cursor, VS Code / GitHub Copilot, and Google Antigravity. It asks for the Enfyra app/admin URL and a programmatic API token from the Enfyra admin UI `/me`.

For non-interactive setup:

```bash
npx @enfyra/mcp-server config --yes \
  --app-url https://demo.enfyra.io \
  --api-token efy_pat_your-token
```

The helper derives the runtime API base from the app URL. Normal apps and demos should point at the app/admin URL, not a hidden backend host.

## Authentication

`ENFYRA_API_TOKEN` is not used directly as a Bearer JWT. The MCP server exchanges it through:

```text
POST {ENFYRA_API_URL}/auth/token/exchange
```

The exchanged short-lived access token is used for authenticated MCP tool calls. If the access token expires or is rejected, the MCP server exchanges the API token again.

Non-root API tokens can use admin helper tools when the backing admin route has ordinary route permission. Use `get_permission_profile`, `audit_route_access`, and `ensure_route_access` to inspect or grant access.

## Required Knowledge Handshake

Before an agent saves dynamic server code or Enfyra extension code, it must call:

```text
get_enfyra_required_knowledge
```

That tool returns required contracts and acknowledgement keys. Code-writing tools verify those keys before saving:

- Pass `dynamicCodeAckKey` as `knowledgeAckKey` when saving handlers, hooks, websocket event scripts, script or condition flow steps, and script-backed records.
- Pass `extensionAckKey` as `extensionKnowledgeAckKey` when saving page, widget, or global extension code.

Discovery, validation, and preview tools do not require the acknowledgement. This lets the agent inspect the instance, read required contracts, validate source, and preview patches before it saves anything.

## Dynamic Repository Paths

Dynamic scripts have two repository trust paths:

- `@REPOS.main` is the secure repository for the current route main table.
- `@REPOS.secure.<table>` is the secure explicit-table repository for public or user-facing custom handlers, hooks, websocket scripts, flows that return data, and third-party app integrations.
- `@REPOS.<table>` is the trusted internal repository for server-owned maintenance or admin logic that intentionally needs hidden fields.

Do not return raw trusted-repository records to users. If trusted access is necessary, project or sanitize the response before returning it.

Repository choice does not replace authorization. Handlers and hooks still need route access, owner or tenant filters, membership checks, and explicit mutation checks.

## Hidden Field Query Surfaces

Unpublished fields and private relations are sensitive even when the value is not selected. User-facing APIs should not expose:

- Filters that act as predicate oracles over hidden fields.
- Aggregate values such as `sum`, `avg`, `max`, or `min` over hidden fields.
- Sort helpers such as `_max(relation.field)`, `_min(relation.field)`, or `_count(relation)` over private relations unless the endpoint intentionally exposes that fact.

If a normal REST read returns an `isPublished=false` field through `fields`, dotted relation fields, or `deep`, treat it as an Enfyra core bug and confirm the minimal REST repro.

## Extension Rules

Before writing or reviewing page, widget, or global extension UI, call:

```text
get_extension_theme_contract
```

Call `get_theme_class_reference` when the exact `eapp-*` class or Nuxt UI color mapping matters.

Extension code should follow the Enfyra app shell:

- Use `eapp-surface-*`, `eapp-text-*`, `eapp-divide-y`, and `eapp-divider` for neutral surfaces.
- Use `eapp-primary-*` or Nuxt UI `primary` only for runtime-primary identity or accent intent.
- Use semantic state colors only for real status, warning, success, info, or error indicators.
- Use `useMenuNotificationRegistry` for sidebar menu counts/dots and `useAccountPanelRegistry` `count`/`badgeColor` fields for account panel notifications.
- Do not hard-code concrete palettes such as `color="violet"` for themeable UI.
- Do not use raw CSS variable utilities when an app token class exists.
- Do not add root-level page padding; page extensions already render inside the Enfyra admin shell.
- Save extensions as Vue SFC records in `enfyra_extension.code`; do not use static import statements.

## Recommended Tool Flow

For custom API work:

1. Call `inspect_route`, `inspect_table`, or `discover_script_contexts`.
2. Call `get_enfyra_required_knowledge`.
3. Validate source with `validate_dynamic_script`.
4. Use `api_endpoint_workflow` or the focused route/handler/hook tool.
5. Test behavior with `test_rest_endpoint`, `run_admin_test`, or `test_flow_step`.

For extension work:

1. Call `get_extension_theme_contract`.
2. Call `get_theme_class_reference` when exact classes are needed.
3. Call `get_enfyra_required_knowledge`.
4. Validate source with `validate_extension_code`.
5. Save with `ensure_page_extension`, `ensure_widget_extension`, or `ensure_global_extension`.

Use the most specific operation tool available before falling back to generic record CRUD.
