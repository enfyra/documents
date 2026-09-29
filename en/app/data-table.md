---
slug: app/data-table
---

# Display records with DataTable

Use `DataTable` for rows of structured records in an Enfyra Admin page or widget. It is the app's wrapper around Nuxt UI's `UTable`: column definitions, native sorting, column visibility, row selection, loading, and named cell/header slots follow the Nuxt UI table model. For a dashboard of distinct cards rather than records, use a responsive card grid instead.

## Render a table

Enfyra Admin extensions receive `DataTable` and `useDataTableColumns()` automatically. Do not import them inside an extension SFC. Provide an array of records and column definitions:

```vue
<template>
  <DataTable :data="records" :columns="columns" :loading="pending" />
</template>

<script setup>
const records = ref([])
const pending = ref(false)
const columns = [
  { accessorKey: 'name', header: 'Name' },
  { accessorKey: 'status', header: 'Status', enableSorting: false },
]
</script>
```

Each `accessorKey` must match a field in the returned record. Use `cell` or a named cell slot when the raw value is not appropriate for display. For example, show status as a `UBadge`, and render an empty optional value as `_`. Use a narrow column or an ellipsis/tooltip for long paths and descriptions rather than letting one value widen the whole table. `DataTable` owns its scroll surface; avoid nesting it in a second horizontal scroller or replacing its built-in table styling.

`DataTable` shows its own loading skeleton when loading begins with no rows, and an empty state when no data is available. Keep the last page of rows visible during refresh instead of replacing it with another skeleton. Row-click actions should open a detail page or editor; keep Enable/Disable and destructive actions in a row `…` menu, with the current state shown as a badge in a normal column. Check backend permissions before exposing actions and require confirmation for destructive operations.

## Fetch bounded pages

For a list that can grow, request a bounded server page and its matching count. Pass the returned page to `DataTable` unchanged: its wrapper does **not** slice server rows or perform client pagination. Use a page control below the table and refetch when the page or page size changes. If the query has a filter, use `meta.filterCount`; use `meta.totalCount` only for an unfiltered query. Reset to page 1 when the filter, scope, or page size changes. Avoid `limit: 0` for growing lists, including built-in/system records.

The built-in Settings lists use `DataTableSettingsTable` to add row actions, a bounded rows-per-page selector (10, 20, 50, or 100), and `CommonPaginationBar`. That adapter belongs to the app's built-in pages; extensions can compose `DataTable` with their own server-driven pagination controls. The selected size is stored per Settings list and restored on refresh.

## Customize columns and selection

`useDataTableColumns().buildActionsColumn({ actions })` builds the `…` dropdown column. Its `actions` can be a fixed array or a function of the current record. Keep the menu narrow by returning only actions that the record and permissions allow. For row selection, use the table's native `v-model:row-selection`, a checkbox column, and `getRowId` with a stable record identifier. Do not infer selection from row position or paginate an already paged response a second time.

On the built-in `/data/<table>` page, a column may declare `metadata.tableCell.formatter`. This eApp-only formatter receives the column metadata and cell value and returns a display value. It changes presentation, not stored data or server responses. For extension-specific formatting, use the table column's `cell` renderer instead.

## When a table appears wrong

- Empty table: confirm the response has a `data` array and the requested fields match the column accessors.
- Wrong count or missing pages: confirm the active filter and `meta.filterCount` describe the same server query.
- Rows disappear during refresh: retain the last successful page until the next response arrives; reserve skeletons for the first load.
- An action is missing: verify both backend route permissions and the record's eligibility before changing the UI.
