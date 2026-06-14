# Schema migration preview (`table_definition` PATCH)

When you change a table’s schema via **`PATCH /api/table_definition/:id`** (admin UI or API), Enfyra may require a **confirmed preview** before applying destructive changes. This keeps multi-DB behavior aligned and prevents silent data loss.

## When preview is required

- Removing **columns** or **relations**, or other destructive diffs, sets `isDestructive` in the policy preview.
- The client must send a matching **`schemaConfirmHash`** (or **`schema_confirm_hash`**) computed from the server’s preview payload so the server knows you reviewed the diff.

If the hash is missing or wrong, the API returns **`422`** with code **`SCHEMA_CONFIRM_HASH_MISMATCH`** (or a preview-only response when no hash was sent).

## What to read in the preview

Typical fields in `details`:

| Field | Meaning |
|-------|---------|
| `removedColumns` / `removedRelations` | What will disappear from the schema |
| `addedColumns` / `addedRelations` | New pieces |
| `renamedColumns` / `changedColumns` | Renames or property changes |
| `requiredConfirmHash` | Value to send back as `schemaConfirmHash` on the real PATCH after review |
| `owningSideInverseCascadeWarnings` | See below |

## `owningSideInverseCascadeWarnings`

Relations expose **`mappedBy`** (inverse property name on the other table) and a stable **`mappedByRelationId`** (owning relation id) in cached metadata.

When you **remove a relation on the owning side** and that relation had **no** `mappedBy` (it is the owning side), other tables may still hold **inverse** `relation_definition` rows that point at it. Removing the owning row **cascade-deletes** those inverse relation rows.

The preview lists those cases so you can adjust other tables first or accept the cascade. **Removing only an inverse relation** does not produce this warning.

## Workflow (API)

1. **PATCH** with your intended `columns` / `relations` payload **without** `schemaConfirmHash` (or with a stale hash) to receive **`preview: true`** and full `details`.
2. Review `details`, especially `owningSideInverseCascadeWarnings` and removed fields.
3. **PATCH** again with the same schema intent plus the hash as a **query parameter**: `?schemaConfirmHash=<details.requiredConfirmHash>` (alias: `schema_confirm_hash`). The server reads it from `$query`, not the JSON body.

Use the Enfyra admin UI if you prefer; it walks the same confirm flow.

## Boot-time metadata migrations

Server snapshots can declare metadata-driven column changes in `data/snapshot-migration.json` with `columnsToModify`. On first-run boot, Enfyra applies those changes before metadata sync so physical columns and metadata rows stay aligned.

Column rename migrations must be generic and table-driven. The migration runner reads the table name plus `oldName` / `newName` from the snapshot migration file, renames the SQL or Mongo field when needed, updates column metadata, and self-heals duplicate old/new columns by preserving the target field and removing the old field. Do not implement table-specific repair code for one metadata table when the same `columnsToModify` contract can describe the change.

For HTTP method metadata, the unique method label is `method_definition.name`. Use `name` in metadata, default data, migrations, tools, filters, and API payloads.

## Related

- [Table creation guide](../getting-started/table-creation.md) — ordering relations and tables
- [Field permissions](./field-permissions.md) — `field_permission_definition` targets columns/relations by id
