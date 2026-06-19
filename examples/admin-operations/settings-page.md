# Settings Page

This example manages a single-record settings table from the admin console.

## Table

Create `app_settings` with `isSingleRecord=true`.

| Field | Type | Notes |
| --- | --- | --- |
| `siteName` | string | Public name |
| `supportEmail` | string | Add email format rule |
| `webhookSecret` | string | `isEncrypted=true`, `isPublished=false` |

## Page Extension

Use `FormEditor` for table-backed settings instead of hand-building every input.

```vue
<template>
  <div class="space-y-4">
    <FormEditorLazy
      v-if="record"
      table-name="app_settings"
      :model-value="record"
      :includes="['siteName', 'supportEmail', 'webhookSecret']"
      @submit="save"
    />
    <LoadingState v-else-if="pending" />
    <ErrorState v-else-if="error" :error="error" />
  </div>
</template>

<script setup>
const { registerPageHeader } = usePageHeaderRegistry();
registerPageHeader({
  title: 'Application Settings',
  description: 'Manage runtime settings',
  leadingIcon: '',
  variant: 'minimal'
});

const record = ref(null);
const pending = ref(false);
const error = ref(null);

const load = async () => {
  pending.value = true;
  error.value = null;
  try {
    const response = await $fetch('/api/app_settings', { query: { limit: 1 } });
    record.value = response.data?.[0] || {};
  } catch (err) {
    error.value = err;
  } finally {
    pending.value = false;
  }
};

const save = async (value) => {
  const id = value.id || record.value?.id;
  if (id) {
    await $fetch(`/api/app_settings/${id}`, { method: 'PATCH', body: value });
  } else {
    await $fetch('/api/app_settings', { method: 'POST', body: value });
  }
  await load();
};

onMounted(load);
</script>
```

Only users with settings route permissions should see this menu item.
