---
slug: app/log-viewing
---

# Trace Errors and Script Logs

Open **Settings → Server Logs** (`/settings/admin/logs`) to investigate a failed operation or inspect output from `@LOGS(...)`. The page reads database records across server instances.

## Access

Root administrators can read both tabs and their details. Other administrators need menu visibility and `GET` permission on `/enfyra_system_error` or `/enfyra_user_log`. Private `stack`, `details`, and `entries` fields also require the corresponding field read permissions. A route permission alone does not reveal private fields.

## Find a Failed Request

1. Copy the `correlationId` from the error response, and note when the request failed.
2. Select **System errors**.
3. Choose a time window, paste the ID into **Correlation ID**, then press **Search / Refresh**.
4. Open an entry to inspect its error code, component, instance, source, status, and available diagnostic details.
5. Press **Find related user logs** to view script output with the same correlation ID.

If you do not have an ID, start with the time window. You can filter by the exact component or error code. Results show the newest records first, with 25 records per page. Refresh is manual.

## Understand the Two Tabs

| Tab | Content |
|---|---|
| System errors | Server and script execution failures, worker crashes, and database or bootstrap errors that reached the error recorder |
| User logs | Explicit `@LOGS(...)` / `$ctx.$logs(...)` output, including script console output captured by the executor |

A worker crash entry can include the exit code, exit signal, last sampled memory usage, and active script IDs. Use those details when investigating; the text “Worker crashed” alone does not establish an out-of-memory failure.

Expected validation or permission rejections are not automatically system failures. Flow execution history and Runtime Monitor remain useful for their own execution state and live metrics.

## Add Useful Script Logs

```javascript
@LOGS('order validation started', { orderId: @BODY.orderId });
// Perform the operation.
@LOGS('order validation finished');
```

Log checkpoints and identifiers that help explain the operation. Do not log passwords, API keys, authorization headers, complete request bodies, or other confidential data. Stored output is sanitized, but automatic redaction cannot recognize every secret embedded in arbitrary text.

Script output is grouped by executor task. A route batch can include pre-hook, handler, and post-hook output together; a flow can produce several records under one correlation ID. Large output is bounded and the detail view reports truncation. Private entries may be absent when your field permissions do not allow reading them.

## Retention and Missing Records

Records are retained for 30 days and expired records are removed in bounded batches. Database writes run outside the request's business transaction, so a business rollback does not roll back its diagnostic record.

If the database is temporarily unavailable, the server retains a bounded in-memory buffer and retries. A full buffer or a process termination can lose unpersisted records. A failure before the diagnostic schema exists may also have no database record. An empty search therefore does not prove the system was healthy. Check the time window, permissions, and container stdout/stderr when investigating a database outage or fatal startup failure.

The application does not write or read local log files. During an upgrade, run the complete bootstrap upgrade so both log tables exist before using the page.

## Next Steps

- [Runtime Monitor](./runtime-monitor.md) for current worker, queue, and database health.
- [Logging and Error Handling](../server/context-reference/logging-errors.md) for script examples.
