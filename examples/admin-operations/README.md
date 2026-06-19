# Admin Operations

Use these examples when operators need a purpose-built screen instead of raw data tables.

Dynamic admin extensions are Vue single-file components stored in `enfyra_extension`. Page extensions should register the app-level page header, use permissions for sensitive actions, and link raw record inspection to `/data/<table_name>`.

## Recipes

- [Moderation Console](./moderation-console.md) - Review pending comments from a focused operator page.
- [Operator Queue](./operator-queue.md) - Build a work queue with filters, pagination, and safe actions.
- [Settings Page](./settings-page.md) - Manage a single-record settings table with `FormEditor`.
