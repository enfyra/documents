# Scheduled Cleanup

This example runs a daily flow that archives stale records.

## Flow

Create a flow named `archive-stale-tasks`.

| Field | Value |
| --- | --- |
| `triggerType` | `schedule` |
| `triggerConfig` | `{"cron":"0 2 * * *","timezone":"UTC"}` |
| `isEnabled` | `true` |

## Step 1: Find Stale Tasks

Type: `script`

```javascript
const cutoff = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString();

const result = await #todo_task.find({
  filter: {
    status: { _eq: 'done' },
    updatedAt: { _lt: cutoff }
  },
  fields: 'id',
  limit: 100
});

return {
  cutoff,
  ids: (result.data || []).map((row) => row.id)
};
```

## Step 2: Archive Tasks

Type: `script`

```javascript
const ids = @FLOW.find_stale_tasks?.ids || [];
let archived = 0;

for (const id of ids) {
  await #todo_task.update({
    id,
    data: { status: 'archived' }
  });
  archived += 1;
}

return { archived };
```

Keep each flow step small. For high-volume cleanup, run bounded batches and let the next scheduled execution continue the work.
