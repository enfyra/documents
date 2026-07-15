# Automation & Integrations

Use these examples when Enfyra talks to another system, runs scheduled work, or protects a custom API.

## Recipes

- [Webhook Ingest](./webhook-ingest.md) - Verify and store signed external events idempotently.
- [Scheduled Cleanup](./scheduled-cleanup.md) - Run a daily flow that archives stale records.
- [Rate-Limited Public API](./rate-limited-public-api.md) - Protect anonymous or partner-facing endpoints.
- [Outbound Sync](./outbound-sync.md) - Send changed records to another API after writes.

## Design Notes

Keep external credentials in encrypted unpublished fields, usually on a single-record settings table. Do not read secrets from environment variables inside dynamic scripts.
