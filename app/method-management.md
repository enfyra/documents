# Method Management

Method Management controls the HTTP method records used by route forms, handler/hook selectors, API testing, and method badges in the Enfyra App.

## Open Method Manager

1. Go to **Settings > Methods**.
2. Review the existing method records and their badge colors.
3. Click **New Method** to create a custom method.

Each method record stores:

- **Name**: the unique uppercase method key stored as `method_definition.name`, such as `GET`, `POST`, `PATCH`, `DELETE`, `PUT`, or a custom key such as `CUSTOM_METHOD`.
- **Background color**: `buttonColor`, used as the badge background.
- **Text color**: `textColor`, used as the badge text.
- **System**: marks built-in runtime methods.

## Create A Method

The create drawer starts with no method selected. Choose one of the available common methods, or choose **Custom** when the preset list does not fit.

Color selection is preset-first:

1. Pick one of the suggested color pairs.
2. Use **custom hex colors** only when no preset fits.
3. The preview shows the method badge with the selected background and text colors.
4. Save the method.

Color values must be full hex values such as `#dbeafe` and `#1d4ed8`.

## Edit A Method

Open **Edit** on a method card to update its color pair. System methods should keep their method key stable; update colors only unless you intentionally own the runtime consequence.

When a drawer has unsaved changes, closing it asks for discard confirmation. Choose **Keep editing** to return to the drawer, or **Discard** to close and lose local changes.

## Use Methods In Routes

Route forms use the method selector for fields such as **Available Methods**, **Public Methods**, and handler/hook method targeting. If the needed method does not exist, use the `+` action in the selector to open Method Manager.

Published method choices control whether an HTTP method is public or permission-protected. Route permissions also use method records to define which authenticated roles can call a route.

Route and permission forms store method relation ids, not raw strings. Method ids are instance data, so resolve methods by `method_definition.name` through the UI or MCP tools before wiring route fields such as **Available Methods**, **Public Methods**, hook methods, handler methods, or route permissions.

## MCP Tools

When managing method metadata through MCP, use the dedicated method tools instead of generic record CRUD:

- `list_methods`
- `create_method`
- `update_method`
- `delete_method`

`delete_method` is preview-first and should only be used for unused custom methods.

The MCP tools keep a user-facing `method` input name, but they read and write the backend `name` field. Do not use generic CRUD payloads with a `method` field on `method_definition`.
