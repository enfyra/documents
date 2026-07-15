# Files & Realtime

Use these examples for uploads, attachments, notifications, and live updates.

Files are managed through `enfyra_file` and storage helpers. Realtime uses Socket.IO gateways and event scripts. Browser apps should connect through the app-origin Socket.IO bridge rather than a hidden backend host.

## Recipes

- [File Attachments](./file-attachments.md) - Attach uploaded files to domain records.
- [Avatar Upload](./avatar-upload.md) - Store a user's avatar and expose a safe profile read.
- [Realtime Notifications](./realtime-notifications.md) - Persist notifications and emit them to the current user.
- [Activity Feed](./activity-feed.md) - Broadcast project activity to a room after writes.
