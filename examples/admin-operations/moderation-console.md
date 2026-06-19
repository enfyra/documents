# Moderation Console

This example creates an admin page for pending comments.

## Menu

Create a menu item:

| Field | Value |
| --- | --- |
| `label` | `Moderation` |
| `path` | `/moderation` |
| `permission` | `{"or":[{"route":"/blog_comment","methods":["GET"]}]}` |

## Page Extension

Create a page extension attached to that menu.

```vue
<template>
  <div class="space-y-4">
    <ErrorState v-if="error" :error="error" />
    <LoadingState v-else-if="pending" />
    <div v-else-if="rows.length" class="space-y-3">
      <div v-for="comment in rows" :key="comment.id" class="rounded-md border border-gray-200 p-4 dark:border-gray-800">
        <div class="flex items-start justify-between gap-3">
          <div>
            <p class="text-sm text-gray-500">{{ comment.author?.email }}</p>
            <p class="mt-2">{{ comment.body }}</p>
          </div>
          <div class="flex gap-2">
            <PermissionGate :condition="{ and: [{ route: '/blog_comment', methods: ['PATCH'] }] }">
              <UButton size="sm" color="primary" @click="updateStatus(comment.id, 'approved')">Approve</UButton>
              <UButton size="sm" color="neutral" variant="outline" @click="updateStatus(comment.id, 'rejected')">Reject</UButton>
            </PermissionGate>
          </div>
        </div>
      </div>
    </div>
    <EmptyState v-else title="No pending comments" />
  </div>
</template>

<script setup>
const { registerPageHeader } = usePageHeaderRegistry();
registerPageHeader({
  title: 'Moderation',
  description: 'Review pending comments',
  leadingIcon: '',
  variant: 'minimal'
});

const rows = ref([]);
const pending = ref(false);
const error = ref(null);

const load = async () => {
  pending.value = true;
  error.value = null;
  try {
    const response = await $fetch('/api/blog_comment', {
      query: {
        filter: JSON.stringify({ status: { _eq: 'pending' } }),
        fields: 'id,body,status,author.email,createdAt',
        sort: '-createdAt',
        limit: 25
      }
    });
    rows.value = response.data || [];
  } catch (err) {
    error.value = err;
  } finally {
    pending.value = false;
  }
};

const updateStatus = async (id, status) => {
  await $fetch(`/api/blog_comment/${id}`, {
    method: 'PATCH',
    body: { status }
  });
  await load();
};

onMounted(load);
</script>
```

Use the raw data route `/data/blog_comment` as a secondary inspection link when operators need full metadata.
