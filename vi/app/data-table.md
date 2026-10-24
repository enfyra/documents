---
slug: ung-dung/bang-du-lieu
---

# Hiển thị bản ghi bằng DataTable

Dùng `DataTable` cho các bản ghi có cấu trúc trong page hoặc widget của Enfyra Admin. Đây là wrapper của `UTable` thuộc Nuxt UI: cột, sắp xếp, ẩn/hiện cột, chọn dòng, loading và slot của ô/tiêu đề theo mô hình bảng gốc. Dùng card grid cho nội dung thiên về hình ảnh, không thay thế danh sách thông tin bằng card.

## Tạo bảng

Extension được cấp sẵn `DataTable` và `useDataTableColumns()`. Không import chúng trong SFC của extension.

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

Mỗi `accessorKey` phải khớp field được trả về. Dùng renderer `cell` hoặc slot như `#status-cell="{ row }"` với `row.original` để hiển thị badge và text tùy biến. Giới hạn giá trị dài bằng ellipsis/tooltip thay vì kéo rộng bảng. Badge phải nằm trọn trong ô, chỉ chữ bên trong bị rút gọn. `DataTable` luôn hiển thị một bảng gốc ở mọi kích thước màn hình, kể cả điện thoại, và đã có vùng cuộn ngang. Bảng không chuyển sang card theo thiết bị hoặc user agent. Không bọc thêm scroller hoặc card trang trí.

Bind `loading` với trạng thái request cho cả lần tải đầu và refresh. Bảng dùng thanh tiến trình native của Nuxt UI dưới header, và skeleton từng hàng khi pending mà chưa có dữ liệu (theo kích thước trang, hoặc 10 dòng). Slot `#loading` riêng thay thế skeleton mặc định. Thông báo rỗng chỉ xuất hiện khi tải xong. Giữ các dòng thành công gần nhất khi đổi trang hoặc filter trong cùng dataset; ẩn dữ liệu cũ khi chuyển sang dataset khác. Có thể dùng slot `#loading` nếu nghiệp vụ cần nội dung riêng.

Bảng tự quản lý border trung tính và radius chung của app. Tiêu đề và nội dung trang cùng nằm trong container 75rem (1200px) được căn giữa của shell. Các class giới hạn trang dùng cùng chiều rộng này để bảng thẳng hàng với tiêu đề. Sự kiện click dòng trả bản ghi gốc để mở chi tiết; đặt hành động xóa trong menu `…`, kiểm tra quyền backend và xác nhận trước khi xóa.

Browse all data tại `/data` dùng cùng bảng để hiển thị các collection mà role của bạn được xem, gồm tên, API path, mô tả, loại Single/Multiple và nút ghim. Tìm theo tên collection, tên bảng hoặc API path; chọn A–Z hay Recent để sắp xếp rồi click dòng để mở dữ liệu. Collection đã ghim đứng đầu và vẫn xuất hiện trong sidebar. Danh sách mặc định 20 dòng mỗi trang và nhớ kích thước trang bạn chọn; nó phân trang trên catalog đã tải, còn danh sách bản ghi bên trong collection vẫn phân trang qua server.

## Bố trí header của bảng

Hàng header luôn hiển thị tiêu đề dataset bắt buộc và số record ở bên trái, cụm action tùy biến ở bên phải; Columns native đứng cuối và chỉ hiện icon. Search, khi bật, là hàng riêng do bảng quản lý ngay bên dưới header. Không lặp lại tiêu đề trang cạnh Search của một dataset.

| Prop | Mục đích |
|---|---|
| `headerName` | Tiêu đề dataset, bắt buộc. Mọi DataTable đều cần prop này. |
| `headerDescription` | Dòng mô tả phụ dưới tiêu đề và số record, tùy chọn. |
| `reload` | Handler reload tùy chọn. Mặc định tắt; nút reload icon-only chỉ xuất hiện khi truyền prop này, đứng sau Columns. |
| `reloadLoading` | Trạng thái pending của nút reload; nếu không truyền thì dùng `loading`. |
| `searchable` | Hiện ô search do bảng quản lý; mặc định tắt. |
| `searchPlaceholder` | Placeholder và label truy cập cho ô search của bảng. |
| `searchLabel` | Label truy cập riêng cho ô search của bảng. |

Số record hiển thị cạnh tiêu đề ở chế độ offset. Ở chế độ cursor, số này chỉ hiện khi bạn truyền tổng số từ server qua `recordCount` hoặc `paginationConfig.total`; hãy tải thêm `filterCount` (hoặc tương đương) cùng trang khi danh sách cursor có tổng số có ý nghĩa, và không tự suy ra tổng.

Search là prop của bảng, không phải markup của caller. Truyền `searchable` và bind `v-model:search` vào query ref của bạn; bảng render `CommonSearchField`, nên ô search giống Collections và mọi danh sách khác mà không cần tự style input, nút hay field group. Dùng `@search` khi việc submit mới kích hoạt request, và reset page về 1 trong handler đó. Chỉ dùng `#toolbar` cho control phụ như quick filter, không đặt ô search ở đó.

Toolbar không có padding ngang, nên đường chia giữa các tầng chạy hết chiều rộng bảng. Trên desktop, tiêu đề, số record cạnh đó và cụm action nằm chung hàng header; ô Search tùy chọn là một hàng riêng ngay bên dưới. Dưới `md`, tiêu đề giữ hàng riêng, số record chuyển sang trái của tầng thứ hai cùng cụm action bên phải, rồi đến hàng Search full width. Control tùy biến đặt trong `#toolbar-actions`; không đăng ký cạnh ô search và không dùng `#toolbar` cho chính ô search. Trang danh sách render bảng như bề mặt độc lập; không lồng bảng vào panel.

Hành động hàng loạt trên dòng đã chọn nằm trong `#toolbar-actions` ở mọi viewport và chỉ hiện icon: một nút thùng rác vuông kiểu error-outline, không có nút Clear và không có chữ đếm số dòng đã chọn trong cụm. Hiển thị số dòng đã chọn qua `#pagination-summary`.

Footer desktop giữ khoảng bản ghi bên trái, bộ chọn số dòng và điều hướng bên phải. Mobile giữ range/selector trên hàng đầu, điều hướng ở hàng thứ hai có đường phân cách. Xem [Tìm nhanh Settings](./settings-search.md) để biết field built-in và cách gửi tìm kiếm qua URL.

## Dùng footer chung

Truyền `paginationConfig` để bật footer có sẵn mà không sao chép selector, markup phân trang hoặc CSS. Không cần slot phân trang. `DataTableLazy` có cùng contract. Slot `#footer` riêng vẫn dùng được để thêm nội dung tóm tắt hoặc hành động bên cạnh footer do bảng quản lý.

| Thuộc tính | Mục đích |
|---|---|
| `mode` | Mặc định `offset`; `cursor` hiện Previous/Next. |
| `itemsPerPage` | Kích thước trang có giới hạn, bắt buộc. |
| `showPageSize` | Hiện bộ chọn 10/20/50/100; mặc định false. |
| `loading` | Trạng thái pending; khóa điều hướng cursor đến khi request kết thúc. |
| `total` | Tổng số thuộc cùng query. Bắt buộc cho đánh số offset; tùy chọn ở cursor, nơi nó bật số record trên header. |
| `hasNextPage` | Cho phép Next ở cursor; mặc định false. |
| `rowCount` | Số bản ghi của trang cursor hiện tại, tùy chọn; mặc định `data.length`. |
| `floating` | Bật thanh mini; mặc định true. Cả offset và cursor dùng ở mọi chiều rộng. |
| `to` | Hàm tạo link trang, tùy chọn khi offset dùng router. |

Offset dùng `v-model:page`. Cursor bind `:page` của trang đã tải thành công và xử lý `@update:page` nhận số trang được yêu cầu. Cả hai chế độ phát `page-size-change` với kích thước đã chọn. Trang tự quản lý request API và các dòng của trang hiện tại; DataTable không tự fetch, nối thêm hay cắt bản ghi server.

Hai kiểu nút nằm cùng vị trí: sau selector số dòng ở bên phải trên desktop, và căn giữa trên hàng thứ hai khi màn hình dưới 768px. Hàng mobile có đường chia toàn chiều ngang và cụm nút cuộn ngang. Các nút luôn hiện đầy đủ, không xuống dòng. Hai nút cursor cao 32px, bằng select số dòng. `UPagination` dùng riêng bên ngoài DataTable giữ mặc định Nuxt UI.

Footer chính nằm trong luồng trang. Khi footer khuất khỏi màn hình nhưng bảng vẫn hiển thị, thanh mini cố định xuất hiện. Cursor dùng cùng cơ chế ở mọi chiều rộng với Previous/Next và khoảng bản ghi hiện tại; offset dùng các nút số trang. Footer chính và thanh mini không xuất hiện đồng thời. Thanh mini dùng chung page, loading và trạng thái hết dữ liệu, giữ nút bên trái/khoảng bản ghi bên phải trên một hàng. Đặt `paginationConfig.floating: false` để tắt thanh mini. Nếu cần lưu số dòng, dùng một key ổn định theo từng danh sách.

## Fetch trang dạng số từ server

Yêu cầu một trang có giới hạn và tổng số thuộc cùng query. Dùng `meta.filterCount` khi có filter, `meta.totalCount` khi không có filter. Giữ nguyên phản hồi; không phân trang client lần nữa trên một trang server. Reset về trang 1 khi đổi filter, phạm vi hoặc kích thước trang. Thay `/work_items` và các field bằng route đã tồn tại cùng các cột có quyền đọc.

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

Giữ trang gần nhất trong lúc pending, nhưng chỉ áp dụng phản hồi thành công mới nhất. Khi request lỗi, cho phép retry; không xóa dòng hay đổi tổng số như thể request đã thành công.

## Chuyển trang cursor bằng Previous và Next

Cursor pagination tải từng trang từ server. Previous bị khóa ở trang 1; Next bị khóa khi `hasNextPage` là false. Cả hai nút bị khóa trong lúc request chạy. Khoảng bản ghi chỉ mô tả trang hiện tại, ví dụ `11–20`, không yêu cầu tổng số.

`page` là vị trí trong lịch sử cursor ở phía client, không phải offset gửi lên REST. Với ID số giảm dần, tải trang đầu không có cursor, lưu ID cuối đang hiển thị, rồi tải trang kế tiếp bằng `filter: { id: { _lt: savedId } }`. Previous dùng lại mốc đã lưu của trang đích. Nếu mỗi trang có 10 dòng, fetch 11 dòng: hiển thị 10 dòng đầu và chỉ dùng dòng dư để xác định còn trang kế tiếp hay không.

Chỉ yêu cầu `meta` ở trang đầu. Query cursor tự thêm điều kiện biên, nên xin lại `filterCount` ở trang 2 sẽ chỉ đếm phần dòng nằm sau mốc đó và badge trên header nhỏ dần sau mỗi lần Next. Lấy tổng số một lần từ request đầu tiên không có biên rồi giữ nguyên cho hết phiên phân trang; reset cùng lúc với lịch sử cursor khi đổi filter, phạm vi hoặc kích thước trang. `totalCount` không thay thế được — nó bỏ qua toàn bộ filter, nên sai ngay khi có filter.

### Ví dụ extension hoàn chỉnh

Thay `/work_items` và các field bằng route đã tồn tại và có quyền đọc. Ví dụ thay các dòng sau mỗi lần chuyển trang thành công, giữ trang gần nhất khi request lỗi và chặn phản hồi cũ. Code dùng API được inject sẵn, không có static import.

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

### Quy tắc reset và xử lý request

- Giữ thứ tự sắp xếp ổn định và field cursor unique. Lấy dòng cuối đang hiển thị làm mốc, không lấy dòng lookahead; lấy dòng dư sẽ bỏ sót một bản ghi. Nếu giá trị sort không unique, dùng field phân định duy nhất và filter cursor tương ứng mà API hỗ trợ.
- Chỉ cập nhật `records`, `page`, `hasNextPage` và lịch sử cursor cùng nhau sau phản hồi thành công. Next hoặc Previous thất bại phải giữ nguyên trang và dữ liệu trước đó; `useApi.pending` được giải phóng khi request kết thúc.
- Dùng mã request hoặc cơ chế hủy để phản hồi cũ không ghi đè trang mới. Ví dụ còn kiểm tra kích thước trang vẫn khớp với request.
- Đổi kích thước trang hoặc filter phải tải lại từ cursor đầu và tạo lại lịch sử. Khi chuyển dataset hoặc tenant, vô hiệu hóa commit đang chờ và xóa ngay dữ liệu cũ trước khi tải scope mới.
- Để refresh trang hiện tại, gọi `loadPage(page.value)`. Để tải lại từ đầu hoặc sau khi đổi filter trong cùng dataset, cập nhật query rồi gọi `loadPage(1, true)`.
- Previous cần các mốc được lưu trong instance của extension. Reload trình duyệt bắt đầu lại từ trang đầu; muốn nhảy đến trang cursor chưa đi qua thì API phải cung cấp được mốc của trang đó.
- Nếu dữ liệu bị xóa trong lúc Next đang tải và phản hồi rỗng, giữ trang còn dữ liệu rồi khóa Next. Previous vẫn dùng được ở trang cuối.

### Cập nhật extension cursor khi nâng phiên bản

Contract cursor được hỗ trợ là `hasNextPage`, `rowCount` tùy chọn cho trang hiện tại và `update:page`. Extension phải áp dụng contract này khi nâng phiên bản; không có lớp tương thích với API Load more.

1. Bind `:page` của trang đã tải thành công và xử lý `@update:page="setPage"`.
2. Thay `hasMore` bằng `hasNextPage`, xác định qua một dòng lookahead.
3. Thay `@load-more` và logic nối dòng bằng handler chọn cursor của trang được yêu cầu rồi thay dữ liệu của trang.
4. Bỏ `loadedCount`. Để footer dùng `data.length`, hoặc truyền `rowCount` chỉ cho trang hiện tại.
5. Lưu mốc cursor cho Previous và reset khi đổi scope query hoặc kích thước trang.
6. Kiểm tra Previous ở trang đầu, Next ở trang cuối, request lỗi, reload, đổi số dòng và thanh mini mobile. Nút bấm được chưa chứng minh query đã thay đổi.

## Tùy biến cột và chọn dòng

Columns được bật mặc định, kể cả trong extension. Truyền toàn bộ định nghĩa cột và dùng `v-model:column-visibility` nếu page cần giữ cột đã ẩn. Các bảng Settings built-in cũng bật Columns; `/data/<table>` lưu lựa chọn riêng theo bảng.

`useDataTableColumns().buildActionsColumn({ actions })` tạo menu `…` theo dòng từ mảng action cố định hoặc hàm nhận bản ghi.

Bật `enable-row-selection` để `DataTable` tự quản lý cột checkbox và trạng thái chọn. Bảng nhận diện bản ghi bằng `id` hoặc `_id`; chỉ truyền `getRowId` khi dữ liệu dùng định danh ổn định khác. Bên gọi không cần tự dựng cột checkbox hay giữ thêm map chọn dòng gốc.

```vue
<DataTable
  :data="records"
  :columns="columns"
  header-name="Records"
  enable-row-selection
  v-model:selected-row-ids="selectedIds"
  @update:selected-rows="selectedRows = $event"
/>
```

`update:selectedRowIds` phát danh sách ID đã chọn; `update:selectedRows` phát các bản ghi đã chọn có trên trang hiện tại khi lựa chọn thay đổi. Bind `v-model:selected-row-ids` để giữ ID qua các trang cursor; dùng danh sách ID này cho thao tác trên nhiều trang vì bảng không tự tải bản ghi ngoài trang hiện tại. Nếu không bind, bảng tự quản lý lựa chọn và bỏ các ID không còn trên trang. `DataTableLazy` hỗ trợ cùng contract.

Dùng `row-selection-mode="single"` để chọn một bản ghi, không có checkbox chọn tất cả; `selection-disabled` để khóa thay đổi lựa chọn; và `select-on-row-click` để bật chọn bằng cách bấm dòng. Bấm checkbox không kích hoạt action bấm dòng thông thường; `/data/<table>` vẫn mở trang chi tiết khi bấm dòng.

Bật chọn dòng không đồng nghĩa với bật quyền xóa. Trang sử dụng bảng quyết định bulk action, quyền truy cập, bản ghi được phép thao tác, xác nhận và cách tải lại dữ liệu. Các trang quản trị CRUD built-in chỉ cung cấp xóa hàng loạt khi có quyền DELETE tương ứng; không thể chọn system record được bảo vệ, tài khoản root admin hay route gắn collection để xóa. Khi một phần thao tác xóa thất bại, danh sách được tải lại và chỉ những bản ghi đã xác nhận xóa thành công mới được bỏ chọn.

API Tester, Audit, Logs, OAuth Accounts, relation picker và các bảng điều hướng hoặc lịch sử runtime không có generic bulk delete. Collections giữ luồng xác nhận xóa schema riêng ở trang chi tiết; Packages giữ luồng Uninstall. Chọn bản ghi trong relation picker chỉ thay đổi giá trị quan hệ của biểu mẫu, không xóa bản ghi liên quan.

Ở `/data/<table>` built-in, `metadata.tableCell.formatter` nhận chuỗi function giới hạn với `(metadata, value)` và trả text, số, boolean hoặc `{ text, color?, variant? }`. Kết quả trống/sai trở về giá trị thông thường (`_` cho ô trống). Editor của cột có ví dụ và giới hạn 1–4096 ký tự, mặc định 80. Đây chỉ là cách trình bày, không đổi dữ liệu lưu hoặc phản hồi. Extension dùng renderer `cell` hoặc slot của ô.

## Xử lý lỗi hiển thị

- Thiếu trang: kiểm tra `paginationConfig.total`, `itemsPerPage` và tổng số của query đang dùng.
- Next luôn bị khóa: đặt `mode: 'cursor'` và cập nhật `hasNextPage` từ request lấy `itemsPerPage + 1` dòng.
- Bấm Next vẫn tải cùng dữ liệu: kiểm tra query trong handler đổi trang. Đổi số trang client chưa chọn cursor; phải gửi mốc đã lưu của trang đó.
- Previous không quay lại được: giữ các mốc cursor trước đó thay vì reset lịch sử ở mọi request.
- Không thấy thanh mini cursor: thanh hiện ở mọi chiều rộng khi footer chính khuất màn hình, bảng còn hiển thị, còn điều hướng được và `floating` là true.
- Không có footer: truyền `paginationConfig` vào `DataTable` hoặc `DataTableLazy`, không phải `UTable`, và bảo đảm eApp đang chạy hỗ trợ prop đó.
- Bảng trống: xác nhận phản hồi có mảng `data` và field khớp accessor của cột.
- Phản hồi cũ ghi đè trang mới: chặn commit cũ và hủy request đọc đã bị thay thế.
- Style extension khác built-in: bỏ CSS phân trang/selector tự sao chép và bật footer có sẵn bằng `paginationConfig`. `UPagination` dùng riêng được giữ nguyên, không mang style footer của bảng.
