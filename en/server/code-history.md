# Code Audit and Versioning

Code audit identifies a changed record by its exact table and ID, records the caller, and keeps the earlier source so it can be reviewed or restored. The current source stays on the record itself.

## Versioned records

These nine tables require an advancing `version` on every update, including metadata-only changes such as enabling or disabling a record:

| Table | Source field |
|---|---|
| `enfyra_route_handler` | `sourceCode` |
| `enfyra_pre_hook` | `sourceCode` |
| `enfyra_post_hook` | `sourceCode` |
| `enfyra_helper` | `sourceCode` |
| `enfyra_websocket` | `sourceCode` |
| `enfyra_websocket_event` | `sourceCode` |
| `enfyra_oauth_config` | `sourceCode` |
| `enfyra_flow_step` | `sourceCode` |
| `enfyra_extension` | `code` |

Menus and other tables are outside this code-history feature. Flow-step source belongs in `sourceCode`, not inside `config`.

## Choose a version

The server validates versions; it never generates an update version.

- **Create:** omit `version` to use the column default `1.0.0`, or supply a valid starting version.
- **Update:** supply a valid version strictly greater than the stored value. This also applies to a source-independent change.
- **Bulk update:** when the table supports the bulk operation, the shared version must be greater than every target's current version.

Use three numeric segments, for example `1.0.0` or `3.0.2`. Numeric segments compare numerically: `1.0.10` is newer than `1.0.9`. The Enfyra comparator also supports extra numeric segments and hyphenated suffixes. A suffixed value follows its base value, so `1.0.0-patch-1` is newer than `1.0.0`; this is not SemVer prerelease ordering. A leading `v` is normalized away. Build metadata using `+` is not supported.

Enfyra Admin proposes the next patch in an editable confirmation dialog. MCP reads the current version and increments its patch when an update omits `version`. A custom MCP update version must be valid and greater; an invalid supplied value is rejected rather than silently replaced. Direct REST or repository callers supply their own version.

```http
PATCH /api/enfyra_route_handler/12
Content-Type: application/json

{
  "version": "1.0.1",
  "sourceCode": "return $ctx.$body.id;"
}
```

Missing, malformed, equal, or lower update versions return `400`. If another writer has advanced the record, inspect its current source and version before retrying. Do not blindly retry a code replacement with a higher version.

## Read history

`GET /api/enfyra_audit` is the canonical read endpoint. Application mutations cannot create, edit, or delete audit rows through repositories; the runtime appends them internally. Database administrators remain outside this application-level protection.

| Field | Meaning |
|---|---|
| `tableName`, `recordId` | Exact target table and record ID; the ID is stored as text |
| `action` | `create`, `update`, `delete`, or `revert` |
| `fromVersion`, `toVersion` | Versions before and after the mutation |
| `restoredFrom` | Earlier version selected by a revert |
| `previous` | Published declared column values before the mutation |
| `changedBy` | Caller relation to `enfyra_user`, when available |
| `changedByEmail` | Caller email captured at write time |
| `changedBySource` | `pat` or `jwt` for authenticated HTTP callers |
| `createdAt` | Audit timestamp |

`previous` excludes record identity, timestamps, generated `compiledCode`, unpublished columns, and expanded relations. OAuth `clientId` and `clientSecret` are therefore not copied into history. A revert cannot recover these excluded values or restore relation assignments. Create entries have no previous state; delete entries have no next version.

Read a bounded page with a JSON `filter` query parameter:

```json
{
  "tableName": { "_eq": "enfyra_route_handler" },
  "recordId": { "_eq": "12" }
}
```

For a list, request fields such as `id,tableName,recordId,action,fromVersion,toVersion,changedByEmail,createdAt`, `sort=-createdAt,-id`, `limit=20`, and `page=1`. Request `previous` only when inspecting a specific entry, because source can be large. Historical source is privileged data: grant history read permission only to trusted code authors.

## Restore an earlier revision

```http
POST /api/admin/code-audit/revert
Content-Type: application/json

{
  "auditId": 41,
  "version": "1.0.4"
}
```

The caller needs access to the revert endpoint, read access to the audit entry, and PATCH access to the target table's route. The restore uses a secure repository, preserving field-write permissions and normal validation, source compilation, reload, and mutation behavior.

The server restores the entry's previous column values into the existing record, using the supplied **new** version. It appends a `revert` entry whose `restoredFrom` identifies the earlier revision. Existing history is not reset or deleted.

A create entry has no previous revision to restore. A delete entry cannot be restored, and a missing target is not recreated. Deletion is irreversible through this feature.

See [Audit in Enfyra Admin](../app/code-history.md) for the UI workflow, or [AI assistant workflows](../integrations/mcp-server.md) for MCP usage.

## Reliability boundary

Audit insertion follows the record write. If insertion fails, the request reports an error; outside a surrounding database transaction, the record may already have changed. Inspect current state before retrying. This is not an atomic code-and-history guarantee across all supported databases, and process failure between writes can leave a gap.

Version validation is not a compare-and-swap lock. Concurrent writers, nested relation mutations, cascade deletion, and administrative reorder paths need separate transactional verification; do not treat this history feature as a complete database audit or a substitute for backups. Version jumps are allowed, so a missing numeric version is not proof of lost history.
