# App Theme Guide

Use this guide when you build Enfyra App pages, widgets, or extensions and want them to match the admin console in both light and dark mode.

The short rule is: choose colors by meaning, not by palette name. Do not pick `green`, `violet`, `cyan`, hex colors, or raw CSS variables just because they look close. Use Enfyra classes and Nuxt UI semantic colors so the UI follows the current app theme.

## Quick Rules

- Use `color="primary"` for the main action or selected identity in the current screen.
- Use `color="success"`, `color="warning"`, `color="error"`, or `color="info"` only when the color communicates status.
- Keep large panels neutral. Put status colors on small badges, icons, or short status text inside the panel.
- Use `eapp-surface-*` classes for cards, rows, and panels.
- Use `eapp-text-*` classes for text hierarchy.
- Use `eapp-primary-*` classes for selected items, active progress, primary icons, and identity accents.
- Use `eapp-icon-tile` for square icon tiles instead of making tiles with padding.
- Use `CommonModal` and `CommonDrawer` footer props for standard Cancel, Save, and Delete actions.
- Use `UTabs` for tabbed sections instead of building custom tab bars.

## Common Choices

| You are building | Use |
| --- | --- |
| Main save/create/run button | `<UButton color="primary" variant="solid" />` |
| Back, refresh, filter, close, or secondary action | `<UButton color="neutral" variant="outline" />` or `variant="ghost"` |
| Destructive action | `<UButton color="error" />` |
| Status badge | `<UBadge color="success|warning|error|info|neutral" variant="soft" />` |
| Normal card or panel | `eapp-surface-card` |
| Recessed inner surface or meter track | `eapp-surface-muted` |
| Clickable row/card hover | `eapp-surface-hover` |
| Heading or primary value | `eapp-text-primary` |
| Body copy | `eapp-text-secondary` |
| Labels, helper text, metadata | `eapp-text-tertiary` |
| Selected/current entity block | `eapp-primary-surface` plus normal card chrome such as `border` and `eapp-radius-panel` |
| Small selected chip or icon tile | `eapp-primary-soft eapp-icon-tile` |
| Primary icon or inline primary text | `eapp-primary-text` |
| Progress fill or active meter | `eapp-primary-solid` |
| Row dividers | `eapp-divide-y` |

## Modal And Drawer Actions

For a normal modal or drawer footer, pass actions through props. This keeps Cancel, Save, Delete, loading, disabled, and spacing consistent across the app.

```vue
<CommonModal
  v-model:open="open"
  :cancel-action="{ label: 'Cancel', onClick: () => (open = false) }"
  :primary-action="{ label: 'Save', loading: saving, disabled: !canSave, onClick: save }"
>
  <template #header>
    <h3 class="text-lg font-semibold eapp-text-primary">Edit settings</h3>
  </template>

  <template #body>
    <FormEditor v-model="form" table-name="settings" />
  </template>
</CommonModal>
```

Use these action props:

| Prop | Use it for |
| --- | --- |
| `cancelAction` | Cancel, discard, or leave-editing actions |
| `primaryAction` | The main safe action, such as Save, Create, Update, or Run |
| `dangerAction` | Delete or irreversible destructive action |
| `leadingActions` | Secondary actions shown on the left side of the footer |
| `footerHint` | A short helper message beside the footer actions |

Cancel is styled as a neutral outline by default. Use `dangerAction` for irreversible destructive work such as delete, revoke, or permanently remove.

```vue
:cancel-action="{ label: 'Close', tone: 'neutral', onClick: () => (open = false) }"
```

For discard-confirmation dialogs, the safe action should be primary:

```vue
:cancel-action="{ label: 'Keep editing', tone: 'primary', onClick: () => (discardOpen = false) }"
```

Use a custom `#footer` only when the footer needs custom layout or non-button content.

For loading placeholders, use `USkeleton` or shared loading components. The app maps skeleton colors globally so placeholders stay visible in both light and dark mode and follow the current accent without hardcoded palette colors.

## Tabs

Use `UTabs` for page sections, settings panels, and multi-step forms. Enfyra styles active and inactive tab indicators globally, so extension tabs match system pages automatically.

```vue
<UTabs
  v-model="activeTab"
  :items="[
    { label: 'Overview', icon: '', value: 'overview' },
    { label: 'Activity', icon: '', value: 'activity' }
  ]"
/>
```

Avoid custom tab bars unless the layout is not really a tabbed interface.

## Cards And Lists

Use neutral surfaces for normal content:

```vue
<article class="eapp-surface-card p-4">
  <div class="flex items-start justify-between gap-3">
    <div>
      <p class="text-sm eapp-text-tertiary">Projects</p>
      <p class="mt-2 text-2xl font-semibold eapp-text-primary">{{ total }}</p>
    </div>

    <span class="eapp-primary-soft eapp-icon-tile">
      
    </span>
  </div>
</article>
```

For rows:

```vue
<section class="eapp-surface-card eapp-divide-y">
  <button
    v-for="item in items"
    :key="item.id"
    type="button"
    class="flex w-full items-center justify-between p-4 text-left eapp-surface-hover"
  >
    <span class="font-medium eapp-text-primary">{{ item.name }}</span>
    <UBadge :color="item.enabled ? 'success' : 'neutral'" variant="soft">
      {{ item.enabled ? 'Enabled' : 'Disabled' }}
    </UBadge>
  </button>
</section>
```

## Selected Or Active Items

Use a primary surface only when the whole block is selected, current, or active. `eapp-primary-surface` supplies the selected identity tint; keep normal card chrome such as `border` and `eapp-radius-panel` on the element too:

```vue
<article class="eapp-primary-surface eapp-radius-panel border p-4">
  <div class="flex items-center gap-3">
    <span class="eapp-primary-soft eapp-icon-tile">
      
    </span>
    <div>
      <p class="font-semibold eapp-text-primary">{{ name }}</p>
      <p class="text-sm eapp-text-tertiary">Currently selected</p>
    </div>
  </div>
</article>
```

Do not use `eapp-primary-surface` for every card in a grid. If everything is highlighted, nothing is highlighted.

## Status

Status colors should be small and specific:

```vue
<section class="eapp-surface-card p-4">
  <div class="flex items-center justify-between gap-3">
    <p class="font-semibold eapp-text-primary">Reconciliation</p>
    <UBadge color="warning" variant="soft">Needs review</UBadge>
  </div>
</section>
```

Avoid making the whole panel green, yellow, or red. Use a neutral panel with a status badge inside it.

## Progress

Use a neutral track and primary fill:

```vue
<div class="h-1.5 overflow-hidden eapp-radius-pill eapp-surface-muted">
  <div class="h-full eapp-primary-solid" :style="{ width: progressWidth }"></div>
</div>
```

## What To Avoid

Do not write:

```vue
<UButton color="violet">Save</UButton>
<UBadge color="green">Active</UBadge>
<div class="bg-slate-950 text-gray-200 border-gray-700"></div>
<div class="text-[var(--some-color)] bg-[var(--some-bg)]"></div>
<div style="color: #14b8a6"></div>
```

Write:

```vue
<UButton color="primary">Save</UButton>
<UBadge color="success">Active</UBadge>
<div class="eapp-surface-card eapp-text-primary"></div>
```

This lets the same extension look correct when the operator changes the app accent color or switches between light and dark mode.
