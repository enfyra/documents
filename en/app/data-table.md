---
slug: app/data-table
---

# Display records with DataTable

Use `DataTable` for structured records in an Enfyra Admin page or widget. It wraps Nuxt UI's `UTable`: columns, sorting, visibility, selection, loading, and named cell/header slots follow the native table model. Use a card grid for distinct visual items, not as a replacement for an information list.

## Render a table

Extensions receive `DataTable` and `useDataTableColumns()` automatically. Do not import them inside an extension SFC.

```vue
<template>
  <DataTable :data="records" :columns="columns" :loading="pending" />
</template>

<script setup>
const records = ref([])
const pending = ref(false)
const columns = [
  { accessorKey: 'id', header: 'ID', size: 88, minSize: 88, maxSize: 88 },
  { accessorKey: 'name', header: 'Name' },
  { accessorKey: 'status', header: 'Status', enableSorting: false },
]
</script>
```

Match each `accessorKey` to a returned field. Use a `cell` renderer or named slot such as `#status-cell="{ row }"` with `row.original` for badges and custom text. Bound long values with ellipsis/tooltip rather than widening the table. A badge should remain inside its cell while its label truncates. `DataTable` renders one native table at every screen size, including phones, and owns horizontal scrolling. It does not switch to cards based on the device or user agent. Do not add another scroller or decorative card around it.

Bind `loading` to the request state for both initial load and refresh. The table uses Nuxt UI's animated header progress line, with no default row skeletons, and shows the empty message only after loading finishes. Retain the last successful rows during pagination/filter refreshes within one dataset; hide stale rows when switching to a different dataset. A custom `#loading` slot remains available when the workflow needs one.

The table owns its neutral border and app radius. Page headings and content share the shell's centered 75rem (1200px) container. The semantic page constraint classes follow that same width so tables stay aligned with the heading. Row clicks receive the original record and can open details; keep destructive actions in a row `…` menu, check backend permissions, and confirm before deleting.

Browse all data at `/data` uses the same table to list collections visible to your role, with their name, API path, description, Single/Multiple type, and pin action. Search by collection name, table name, or API path; use A–Z or Recent to sort, and click a row to open its records. Pins appear first and remain available in the sidebar. The directory defaults to 20 rows per page and remembers your chosen page size; it paginates the loaded collection catalog, while record lists inside a collection use server pagination.

## Use the shared footer

Pass `paginationConfig` to enable the built-in footer without copying selectors, pagination markup, or CSS. No pagination slot is required. `DataTableLazy` supports the same contract. Custom `#footer` content may add a summary or actions alongside the managed footer.

| Property | Purpose |
|---|---|
| `mode` | `offset` by default; `cursor` shows Previous/Next. |
| `itemsPerPage` | Required bounded page size. |
| `showPageSize` | Shows the 10/20/50/100 selector; defaults to false. |
| `loading` | Pending state; cursor navigation is disabled until the request settles. |
| `total` | Matching server count for offset pagination. Cursor does not require a total. |
| `hasNextPage` | Enables Next in cursor mode; defaults to false. |
| `rowCount` | Optional number of records on the current cursor page; defaults to `data.length`. |
| `floating` | Enables the mini pager; defaults to true. Offset supports it at all widths, cursor below 768px. |
| `to` | Optional page-link function for route-backed offset navigation. |

Offset uses `v-model:page`. Cursor binds the committed `:page` and handles `@update:page` with the requested page number. Both modes emit `page-size-change` with the selected size. The caller owns API requests and the current page of rows; DataTable does not fetch, append, or slice server records.

Both control modes occupy the same position: after the page-size selector on the right at desktop widths, and centered on a second row below 768px. The mobile row has a full-width separator and a horizontally scrollable control strip. Buttons remain visible without wrapping. Cursor buttons are 32px high, matching the page-size select. Raw `UPagination` outside DataTable retains Nuxt UI defaults.

The main footer stays in normal flow. When it leaves the viewport and its table remains visible, a fixed mini pager appears. Cursor uses this same handoff on mobile, displaying Previous/Next and the current range; offset uses numbered controls. Main and mini controls never appear together. The mini shares page, loading and end-of-data state with the main footer, and keeps controls left/range right on one row. Set `paginationConfig.floating: false` to disable it. Persist page size under one stable key per list if needed.

## Fetch numbered pages from the server

Request one bounded page and a count matching the same query. Use `meta.filterCount` for filtered lists and `meta.totalCount` otherwise. Keep the response unchanged; never apply client pagination to a server page. Reset to page 1 when filters, scope, or page size change. Replace `/work_items` and its fields with an existing route and readable columns.

```vue
<script setup>
const page = ref(1)
const pageSize = ref(10)
const records = ref([])
const total = ref(0)
const columns = [
  { accessorKey: 'id', header: 'ID' },
  { accessorKey: 'title', header: 'Title' },
]
const { pending, execute: fetchItems } = useApi('/work_items')
let requestId = 0

async function loadPage() {
  const currentRequest = ++requestId
  const response = await fetchItems({ query: {
    fields: 'id,title', sort: '-id', page: page.value,
    limit: pageSize.value, meta: 'totalCount',
  } })
  if (!response || currentRequest !== requestId) return
  records.value = response.data || []
  total.value = response.meta?.totalCount || 0
}
function setPageSize(size) {
  pageSize.value = size
  page.value = 1
}
watch([page, pageSize], () => { void loadPage() })
onMounted(() => { void loadPage() })
</script>

<template>
  <DataTable
    v-model:page="page"
    :data="records" :columns="columns" :loading="pending"
    :pagination-config="{ total, itemsPerPage: pageSize, showPageSize: true, loading: pending }"
    @page-size-change="setPageSize"
  />
</template>
```

Keep the last page visible during pending requests, but only commit the latest successful response. Surface request failures with a retry action; do not clear rows or change the count as if a failed request succeeded.

## Navigate cursor pages with Previous and Next

Cursor pagination fetches one server page at a time. Previous is disabled on page 1; Next is disabled when `hasNextPage` is false. Both buttons are disabled during a request. The range describes only the current page, for example `11–20`, without requiring a total count.

The local `page` is a position in visited cursor history, not a REST offset. For descending numeric IDs, request the first page without a cursor, save the last displayed ID, then fetch the next page with `filter: { id: { _lt: savedId } }`. Previous reuses the saved boundary for its target page. With page size 10, fetch 11 rows: display the first 10 and use the extra row only to decide whether Next is available.

### Complete extension example

Replace `/work_items` and its fields with an existing readable route. This example replaces rows on each successful page change, retains the last page on failure, and rejects stale responses. It uses the injected extension APIs without static imports.

```vue
<script setup>
const page = ref(1)
const pageSize = ref(10)
const records = ref([])
const hasNextPage = ref(false)
const columns = [
  { accessorKey: 'id', header: 'ID', size: 88, minSize: 88, maxSize: 88 },
  { accessorKey: 'title', header: 'Title' },
]
const { pending, error, execute: fetchItems } = useApi('/work_items')
let cursors = [null]
let requestId = 0

async function loadPage(targetPage, reset = false) {
  if (!Number.isInteger(targetPage) || targetPage < 1) return false
  if (!reset && (pending.value || targetPage > cursors.length)) return false
  const run = ++requestId
  const size = pageSize.value
  const cursor = reset ? null : cursors[targetPage - 1]
  const response = await fetchItems({ query: {
    fields: 'id,title', sort: '-id', limit: size + 1,
    ...(cursor != null ? { filter: { id: { _lt: cursor } } } : {}),
  } })
  if (!response || run !== requestId || size !== pageSize.value) return false
  const incoming = response.data || []
  const nextRows = incoming.slice(0, size)
  if (targetPage > page.value && nextRows.length === 0) {
    hasNextPage.value = false
    cursors = cursors.slice(0, page.value)
    return false
  }
  const nextCursors = reset ? [null] : cursors.slice(0, targetPage)
  const hasNext = incoming.length > size
  if (hasNext) nextCursors.push(nextRows[nextRows.length - 1].id)
  cursors = nextCursors
  records.value = nextRows
  page.value = targetPage
  hasNextPage.value = hasNext
  return true
}

function setPage(targetPage) {
  if (targetPage === page.value) return
  return loadPage(targetPage)
}

function setPageSize(size) {
  if (size === pageSize.value || ![10, 20, 50, 100].includes(size)) return
  pageSize.value = size
  return loadPage(1, true)
}

onMounted(() => { void loadPage(1, true) })
</script>

<template>
  <div class="space-y-4">
    <UAlert v-if="error" color="error" title="Unable to load records">
      <template #actions>
        <UButton type="button" label="Retry" color="neutral" variant="outline"
          :loading="pending" @click="loadPage(page)" />
      </template>
    </UAlert>
    <DataTable
      :page="page"
      :data="records" :columns="columns" :loading="pending"
      :pagination-config="{ mode: 'cursor', itemsPerPage: pageSize, showPageSize: true, hasNextPage, loading: pending }"
      @update:page="setPage"
      @page-size-change="setPageSize"
    />
  </div>
</template>
```

### Reset and request rules

- Keep a stable sort and unique cursor field. Use the last displayed row as the boundary, not the lookahead row; using the extra row would skip a record. For non-unique sort values, use the API's supported unique tie-breaker and matching cursor filter.
- Commit `records`, `page`, `hasNextPage` and cursor history together after a successful response. A failed Next or Previous request must retain the prior page and rows; `useApi.pending` releases when the request settles.
- Keep a request identifier or cancellation guard so an older response cannot overwrite a newer page. The example also verifies that the page size still matches the request.
- Page-size and filter changes start again at the first cursor and rebuild history. For a different dataset or tenant, invalidate pending commits and clear stale rows immediately before loading that scope.
- To refresh the current page, call `loadPage(page.value)`. To refresh from the first page or after changing a same-dataset filter, call `loadPage(1, true)` after updating the query state.
- Previous requires boundaries saved in the current extension instance. A browser reload starts at the first page; arbitrary jumps to an unvisited cursor page require an API that can supply that boundary.
- If rows disappear while Next is loading and the response is empty, keep the populated page and disable Next. Previous remains usable on a final page.

### Update a cursor extension after upgrading

The supported cursor contract is `hasNextPage`, optional current-page `rowCount`, and `update:page`. Extensions must adopt this contract when upgrading; there is no compatibility layer for the Load more API.

1. Bind the committed `:page` and handle `@update:page="setPage"`.
2. Replace `hasMore` with `hasNextPage`, determined by one-row lookahead.
3. Replace `@load-more` and append logic with a handler that selects the requested cursor and replaces the page of rows.
4. Remove `loadedCount`. Let the footer use `data.length`, or provide `rowCount` for the current page only.
5. Keep cursor boundaries for Previous and reset them when the query scope or size changes.
6. Check first-page Previous, last-page Next, failures, reload, page-size changes and the mobile mini pager. A clickable button alone does not prove the query changed.

## Customize columns and selection

Columns visibility is enabled by default, including in extensions. Pass all column definitions and bind `v-model:column-visibility` when the page must retain hidden columns. Built-in Settings tables opt out; `/data/<table>` persists visibility per table.

`useDataTableColumns().buildActionsColumn({ actions })` builds a row `…` menu from a fixed action list or a function of the record. For selection, use native `v-model:row-selection`, a checkbox column, and `getRowId` based on a stable identifier; never infer selection from row position.

On built-in `/data/<table>`, `metadata.tableCell.formatter` accepts a restricted function string with `(metadata, value)` and returns text, a number, a boolean, or `{ text, color?, variant? }`. Empty/invalid output falls back to the ordinary value (`_` for empty cells). The column editor supplies an example and a 1–4096-character display limit (80 by default). This changes presentation only, not stored data or responses. Extensions use column `cell` renderers or named slots.

## Troubleshoot

- Missing pages: verify `paginationConfig.total`, `itemsPerPage`, and the count returned by the active query.
- Next always disabled: set `mode: 'cursor'` and update `hasNextPage` from a request for `itemsPerPage + 1` rows.
- Clicking Next fetches the same records: inspect the query in the page-change handler. A changed local page number does not select a cursor; send the saved boundary for that page.
- Previous cannot return: retain earlier cursor boundaries rather than resetting history on every request.
- Cursor mini pager absent: it appears only below 768px while the main footer is off screen, the table is visible, navigation is available and `floating` is true.
- Footer absent: pass `paginationConfig` to `DataTable` or `DataTableLazy`, not to `UTable`, and ensure the running app supports that prop.
- Empty rows: confirm the response has a `data` array and fields match column accessors.
- A stale response replaces a newer page: guard response commits and cancel superseded reads.
- Style differs in an extension: remove copied pagination/selector CSS and enable the built-in footer with `paginationConfig`. Raw `UPagination` is intentionally not styled as a table footer.
