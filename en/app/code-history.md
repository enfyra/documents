# Audit Code Changes

Use Audit to inspect earlier dynamic-code revisions and restore one without resetting the record's version.

## Save a code record

When updating a handler, hook, shared helper, WebSocket gateway or event, OAuth configuration, flow step, or extension:

1. Edit and validate the record, then choose Save.
2. Enfyra Admin reads its current version and opens **Save a new code version**.
3. Review the proposed patch increment, such as `3.0.1` to `3.0.2`. You can replace it with a greater valid version such as `4.0.0`.
4. Choose **Save version** to send the update, or **Close** to leave your draft unchanged.

The dialog also appears for metadata-only updates. A create does not require this dialog: an omitted version defaults to `1.0.0`. Reading the current version requires GET access to the target record. A denied or unavailable read stops the save rather than guessing a version.

For updates sent with the shared `useApi`, the same confirmation applies to all nine code tables. Direct HTTP callers still need to provide their own advancing version.

## Find an earlier revision

1. Open **Settings → Audit**, or visit `/settings/audit`.
2. The list starts on **All**, covering every versioned table. Open the table menu in the table toolbar to narrow it to one table.
3. Enter a record ID in **Exact record ID** and press Enter or choose **Search** to restrict the list to that exact record. Clear the search and submit to show all records in the current scope.
4. Review the action, before/after versions, caller email, credential type, and date.
5. Open a row to see who changed it and when, the source before that change, and the other stored values at that revision.

The table loads one cursor page at a time using **Previous** and **Next**, and hides the record count because the total is not queried. Historical source is fetched only for the selected row. You need GET permission for `/enfyra_audit`; the Audit entry is hidden without it.

## Restore as a new version

Open an update or revert entry and choose **Restore as new version**. Review the proposed next version in the confirmation dialog, then save. Restoring creates another update and another history entry; it does not move the version backward or erase intervening changes.

You need POST access to `/admin/code-audit/revert` and PATCH access to the target table, as well as history read access. Field-write permissions still apply. Private fields such as OAuth credentials and relation assignments are not restored.

Create entries have no earlier revision. Delete entries are history-only; deleted records cannot be restored. If the target no longer exists, restore fails without recreating it.

## If a save or restore fails

- **Version conflict:** refresh the record, inspect changes from the other writer, and retry only after reviewing them.
- **Access denied:** check the history, revert, target GET/PATCH, and field permissions separately. Menu visibility does not grant API access.
- **Audit insertion failed:** inspect the current record before retrying, because the record write may already have completed.

See the [server version and reliability contract](../server/code-history.md). History does not replace backups or provide a complete audit of nested and cascade writes.
