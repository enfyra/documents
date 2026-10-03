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

Bind `loading` với trạng thái request cho cả lần tải đầu và refresh. Bảng dùng thanh tiến trình native của Nuxt UI dưới header, không có skeleton từng hàng mặc định, và chỉ hiện thông báo rỗng sau khi tải xong. Giữ các dòng thành công gần nhất khi đổi trang hoặc filter trong cùng dataset; ẩn dữ liệu cũ khi chuyển sang dataset khác. Có thể dùng slot `#loading` nếu nghiệp vụ cần nội dung riêng.

Bảng tự quản lý border trung tính và radius chung của app. Tiêu đề và nội dung trang cùng nằm trong container 75rem (1200px) được căn giữa của shell. Các class giới hạn trang dùng cùng chiều rộng này để bảng thẳng hàng với tiêu đề. Sự kiện click dòng trả bản ghi gốc để mở chi tiết; đặt hành động xóa trong menu `…`, kiểm tra quyền backend và xác nhận trước khi xóa.

Browse all data tại `/data` dùng cùng bảng để hiển thị các collection mà role của bạn được xem, gồm tên, API path, mô tả, loại Single/Multiple và nút ghim. Tìm theo tên collection, tên bảng hoặc API path; chọn A–Z hay Recent để sắp xếp rồi click dòng để mở dữ liệu. Collection đã ghim đứng đầu và vẫn xuất hiện trong sidebar. Danh sách mặc định 20 dòng mỗi trang và nhớ kích thước trang bạn chọn; nó phân trang trên catalog đã tải, còn danh sách bản ghi bên trong collection vẫn phân trang qua server.

## Dùng footer chung

Truyền `paginationConfig` để bật footer có sẵn mà không sao chép selector, markup phân trang hoặc CSS. Không cần slot phân trang. `DataTableLazy` có cùng contract. Slot `#footer` riêng vẫn dùng được để thêm nội dung tóm tắt hoặc hành động bên cạnh footer do bảng quản lý.

| Thuộc tính | Mục đích |
|---|---|
| `mode` | Mặc định `offset`; `cursor` hiện Previous/Next. |
| `itemsPerPage` | Kích thước trang có giới hạn, bắt buộc. |
| `showPageSize` | Hiện bộ chọn 10/20/50/100; mặc định false. |
| `loading` | Trạng thái pending; khóa điều hướng cursor đến khi request kết thúc. |
| `total` | Tổng số thuộc cùng query cho offset. Cursor không cần tổng số. |
| `hasNextPage` | Cho phép Next ở cursor; mặc định false. |
| `rowCount` | Số bản ghi của trang cursor hiện tại, tùy chọn; mặc định `data.length`. |
| `floating` | Bật thanh mini; mặc định true. Offset dùng ở mọi chiều rộng, cursor dùng dưới 768px. |
| `to` | Hàm tạo link trang, tùy chọn khi offset dùng router. |

Offset dùng `v-model:page`. Cursor bind `:page` của trang đã tải thành công và xử lý `@update:page` nhận số trang được yêu cầu. Cả hai chế độ phát `page-size-change` với kích thước đã chọn. Trang tự quản lý request API và các dòng của trang hiện tại; DataTable không tự fetch, nối thêm hay cắt bản ghi server.

Hai kiểu nút nằm cùng vị trí: sau selector số dòng ở bên phải trên desktop, và căn giữa trên hàng thứ hai khi màn hình dưới 768px. Hàng mobile có đường chia toàn chiều ngang và cụm nút cuộn ngang. Các nút luôn hiện đầy đủ, không xuống dòng. Hai nút cursor cao 32px, bằng select số dòng. `UPagination` dùng riêng bên ngoài DataTable giữ mặc định Nuxt UI.

Footer chính nằm trong luồng trang. Khi footer khuất khỏi màn hình nhưng bảng vẫn hiển thị, thanh mini cố định xuất hiện. Cursor dùng cùng cơ chế trên mobile với Previous/Next và khoảng bản ghi hiện tại; offset dùng các nút số trang. Footer chính và thanh mini không xuất hiện đồng thời. Thanh mini dùng chung page, loading và trạng thái hết dữ liệu, giữ nút bên trái/khoảng bản ghi bên phải trên một hàng. Đặt `paginationConfig.floating: false` để tắt thanh mini. Nếu cần lưu số dòng, dùng một key ổn định theo từng danh sách.

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

Columns được bật mặc định, kể cả trong extension. Truyền toàn bộ định nghĩa cột và dùng `v-model:column-visibility` nếu page cần giữ cột đã ẩn. Các bảng Settings built-in tắt menu này; `/data/<table>` lưu lựa chọn riêng theo bảng.

`useDataTableColumns().buildActionsColumn({ actions })` tạo menu `…` theo dòng từ mảng action cố định hoặc hàm nhận bản ghi. Để chọn dòng, dùng `v-model:row-selection` gốc, cột checkbox và `getRowId` dựa trên định danh ổn định; không suy ra lựa chọn từ vị trí dòng.

Ở `/data/<table>` built-in, `metadata.tableCell.formatter` nhận chuỗi function giới hạn với `(metadata, value)` và trả text, số, boolean hoặc `{ text, color?, variant? }`. Kết quả trống/sai trở về giá trị thông thường (`_` cho ô trống). Editor của cột có ví dụ và giới hạn 1–4096 ký tự, mặc định 80. Đây chỉ là cách trình bày, không đổi dữ liệu lưu hoặc phản hồi. Extension dùng renderer `cell` hoặc slot của ô.

## Xử lý lỗi hiển thị

- Thiếu trang: kiểm tra `paginationConfig.total`, `itemsPerPage` và tổng số của query đang dùng.
- Next luôn bị khóa: đặt `mode: 'cursor'` và cập nhật `hasNextPage` từ request lấy `itemsPerPage + 1` dòng.
- Bấm Next vẫn tải cùng dữ liệu: kiểm tra query trong handler đổi trang. Đổi số trang client chưa chọn cursor; phải gửi mốc đã lưu của trang đó.
- Previous không quay lại được: giữ các mốc cursor trước đó thay vì reset lịch sử ở mọi request.
- Không thấy thanh mini cursor: thanh chỉ hiện dưới 768px khi footer chính khuất màn hình, bảng còn hiển thị, còn điều hướng được và `floating` là true.
- Không có footer: truyền `paginationConfig` vào `DataTable` hoặc `DataTableLazy`, không phải `UTable`, và bảo đảm eApp đang chạy hỗ trợ prop đó.
- Bảng trống: xác nhận phản hồi có mảng `data` và field khớp accessor của cột.
- Phản hồi cũ ghi đè trang mới: chặn commit cũ và hủy request đọc đã bị thay thế.
- Style extension khác built-in: bỏ CSS phân trang/selector tự sao chép và bật footer có sẵn bằng `paginationConfig`. `UPagination` dùng riêng được giữ nguyên, không mang style footer của bảng.
