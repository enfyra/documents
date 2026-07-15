---
slug: vi-du/tu-dong-hoa-va-tich-hop/don-dep-theo-lich
---

# Dọn dẹp theo lịch

Ví dụ này chạy flow hằng ngày để lưu trữ các bản ghi đã cũ.

## Flow

Tạo flow có tên `archive-stale-tasks`.

| Field | Giá trị |
| --- | --- |
| `triggerType` | `schedule` |
| `triggerConfig` | `{"cron":"0 2 * * *","timezone":"UTC"}` |
| `isEnabled` | `true` |

## Bước 1: Tìm task cũ

Kiểu: `script`

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

## Bước 2: Lưu trữ task

Kiểu: `script`

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

Giữ từng bước flow gọn và có phạm vi nhỏ. Khi cần dọn dẹp khối lượng lớn, hãy chạy batch có giới hạn để lần thực thi theo lịch tiếp theo tiếp tục công việc.
