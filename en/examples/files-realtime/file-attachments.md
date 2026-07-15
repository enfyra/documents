# File Attachments

This example attaches uploaded files to tasks, tickets, or comments.

## Tables

Create `task_attachment`.

| Field or relation | Type | Notes |
| --- | --- | --- |
| `task` | many-to-one relation to `todo_task` | Required |
| `file` | many-to-one relation to `enfyra_file` | Required |
| `uploadedBy` | many-to-one relation to `enfyra_user` | Required |

## Upload A File

Use the built-in file route for the binary upload.

```bash
curl "$ENFYRA_API_URL/enfyra_file" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -F "file=@./contract.pdf" \
  -F "folder=attachments"
```

The response contains the `enfyra_file` record id.

## Attach The File

Add a `POST /task_attachment` pre-hook.

```javascript
if (!@USER?.id) {
  @THROW401();
}

@BODY.uploadedBy = { id: @USER.id };
```

```bash
curl "$ENFYRA_API_URL/task_attachment" \
  -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"task":{"id":42},"file":{"id":318}}'
```

## List Attachments

```bash
curl "$ENFYRA_API_URL/task_attachment?filter={\"task\":{\"id\":{\"_eq\":42}}}&fields=id,file.id,file.filename,file.url,file.size,uploadedBy.email,createdAt" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Keep ownership checks on the parent task route or on `task_attachment` hooks so a user cannot attach files to a task they cannot access.
