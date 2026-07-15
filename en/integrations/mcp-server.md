# Use Enfyra with an AI coding assistant

Enfyra MCP connects your Enfyra project to MCP-compatible coding assistants such as Codex, Claude Code, Cursor, VS Code with GitHub Copilot, and Google Antigravity. Once connected, you can ask the assistant to inspect and change your project using normal language instead of copying schemas, API responses, or configuration between tools.

## What you can do

Use Enfyra MCP to:

- Explore tables, relations, routes, permissions, flows, extensions, and runtime configuration.
- Create or update data models while preserving existing project structure.
- Build and test custom API endpoints, handlers, hooks, and automations.
- Create admin pages and widgets that follow the current Enfyra theme.
- Investigate permission errors, failed requests, logs, and runtime behavior.
- Verify a change against the running Enfyra instance before considering it complete.

The assistant reads the live project, so its work is based on the instance you connected rather than a stale description of your schema.

## Before you start

You need:

1. An Enfyra instance you can open in the browser.
2. A programmatic API token from **Account → API tokens** in Enfyra Admin.
3. An MCP-compatible coding tool.
4. A local project folder where the assistant will work.

Use a token with only the permissions required for that project. Keep separate tokens for development and production.

## Connect your project

Open a terminal in the project folder and run:

```bash
npx @enfyra/mcp-server config
```

The setup asks for your Enfyra URL and API token, then writes the correct project-local configuration for the detected coding tool.

For automated setup:

```bash
npx @enfyra/mcp-server config --yes \
  --app-url https://demo.enfyra.io \
  --api-token efy_pat_your-token
```

Use the URL you normally open for Enfyra Admin. After configuration, restart or reload your coding tool so it can discover the new MCP server.

## Confirm the connection

Start with a read-only request in your coding assistant:

> Check which Enfyra instance is connected, then list the main tables in this project. Do not change anything.

Confirm that the reported URL and tables belong to the intended environment before asking for changes.

## Common workflows

### Understand an existing project

Ask for an overview before changing an unfamiliar project:

> Review this Enfyra project and explain its main tables, relations, public API routes, permissions, and automations. Highlight anything that needs attention, but do not modify it yet.

### Create a data model

Describe the business model and important constraints instead of specifying low-level metadata:

> Add `projects` and `project_members` so a project can have many members. Prevent duplicate membership, keep existing data unchanged, and verify the relations after applying the change.

The assistant can inspect the current schema, choose compatible field types, apply the change, and verify the result.

### Build an API endpoint

Describe who may call the endpoint and which records they may access:

> Create an endpoint that returns the signed-in user's active projects. A user must only see projects where they are a member. Test both an allowed request and an unauthorized request.

Authorization requirements are part of the feature. Do not rely on route availability alone when records are owned by users, teams, or tenants.

### Create an automation

Describe the trigger, action, and failure behavior:

> When an order becomes paid, create an invoice record and notify the operations team. Make the flow safe to retry and show me the final flow structure before enabling it.

### Add an admin page or widget

Explain the job the interface should help users complete:

> Add an admin page for reviewing pending orders. Include filters for date and status, use the current Enfyra theme, and only show actions the signed-in user is allowed to perform.

### Diagnose a problem

Provide the failing URL, visible error, and expected behavior:

> Requests to `/api/projects` return 403 for the editor role. Inspect the route, role permissions, and record-level authorization, then explain the cause. Do not change permissions until the cause is confirmed.

## Writing effective requests

A useful request states:

- The outcome you want.
- The users or roles involved.
- Which existing data or behavior must remain unchanged.
- Important authorization or tenant boundaries.
- How the result should be tested.
- Whether the assistant may apply changes or should only prepare a plan.

For larger work, ask the assistant to inspect the current project first and apply the change only after it can describe the affected tables, routes, and permissions.

## Work safely

- Confirm the connected environment before every production change.
- Use separate project-local configurations for different repositories or environments.
- Ask for a preview before deleting tables, fields, routes, or data.
- Keep backups before destructive schema changes or large data migrations.
- Give production tokens the minimum permissions required.
- Revoke a token immediately if it is exposed in logs, source control, or chat.
- Test authorization with both an allowed user and a user who must be denied.

## Switch environments

MCP configuration is stored in the current project. To point the project at another Enfyra instance, run the configuration command again with the new URL and token, then restart the coding tool.

Do not reuse one production token across unrelated local projects.

## Troubleshooting

### Enfyra tools do not appear

Restart the coding tool after running the configuration command. Confirm that the generated MCP configuration is inside the current project and that Node.js can run `npx`.

### Authentication fails

Create a new programmatic API token in Enfyra Admin and run the configuration command again. Check that the Enfyra URL belongs to the same instance that issued the token.

### A read or change is denied

The connection is working, but the token's user or role does not have the required route or table permission. Grant only the missing permission, then retry the original request.

### The assistant sees an old schema

Ask it to refresh the relevant table or route metadata. If you changed the MCP configuration, restart the coding tool before retrying.

### A large task is difficult to review

Split the request by outcome: schema, API behavior, automation, interface, then final end-to-end verification. Keep each step independently testable.

## Advanced configuration

The setup command manages these values for you:

| Setting | Purpose |
|---|---|
| `ENFYRA_APP_URL` | The Enfyra URL you open in the browser. |
| `ENFYRA_API_URL` | The API base derived from the app URL. |
| `ENFYRA_API_TOKEN` | The programmatic token used to authenticate the connection. |
| `ENFYRA_MCP_TOOLSET` | Optional tool visibility mode; the default guided mode is recommended. |

Use the full toolset only for advanced debugging or compatibility work. It exposes lower-level operations that are usually unnecessary for normal project development.
