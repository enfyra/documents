---
slug: ung-dung/bang-du-lieu
---

# Hiển thị bản ghi bằng DataTable

Dùng `DataTable` để hiển thị các bản ghi có cấu trúc trong page hoặc widget của Enfyra Admin. Đây là wrapper của ứng dụng quanh `UTable` của Nuxt UI: định nghĩa cột, sắp xếp, ẩn/hiện cột, chọn dòng, trạng thái tải và các slot của ô/tiêu đề đều theo mô hình bảng của Nuxt UI. Nếu đang trình bày một dashboard gồm các thẻ nội dung khác nhau thay vì các bản ghi, hãy dùng bố cục card phù hợp.

## Tạo bảng

Extension của Enfyra Admin được cấp sẵn `DataTable` và `useDataTableColumns()`. Không cần import chúng trong SFC của extension. Truyền danh sách bản ghi và định nghĩa cột:

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

Mỗi `accessorKey` phải khớp với một field trong bản ghi trả về. Dùng `cell` hoặc slot của ô khi không muốn hiển thị giá trị thô. Chẳng hạn, thể hiện trạng thái bằng `UBadge` và giá trị tùy chọn còn trống bằng `_`. Với đường dẫn hoặc mô tả dài, hãy giới hạn chiều rộng cột và dùng dấu ba chấm/tooltip thay vì để một giá trị kéo giãn cả bảng. `DataTable` đã có vùng cuộn riêng; đừng bọc thêm vùng cuộn ngang hoặc thay thế toàn bộ style bảng mặc định.

`DataTable` hiển thị skeleton khi tải lần đầu mà chưa có dòng và hiển thị empty state khi không có dữ liệu. Trong lúc refresh, giữ trang bản ghi gần nhất thay vì thay bằng skeleton mới. Nhấn vào dòng để mở trang chi tiết hoặc editor; đặt Enable/Disable và hành động có thể xóa dữ liệu trong menu `…`, còn trạng thái hiện tại thể hiện bằng badge ở cột thông thường. Kiểm tra quyền backend trước khi cho phép thao tác và xác nhận trước hành động phá hủy.

## Phân trang có giới hạn

Với danh sách có thể tăng, hãy yêu cầu một trang dữ liệu có giới hạn từ server cùng tổng số tương ứng. Truyền nguyên trang nhận được vào `DataTable`: wrapper **không** cắt dữ liệu server hay tự phân trang ở client. Đặt điều khiển phân trang dưới bảng và tải lại khi trang hoặc số dòng mỗi trang đổi. Nếu query có filter, dùng `meta.filterCount`; chỉ dùng `meta.totalCount` khi query không có filter. Trở về trang 1 khi đổi filter, phạm vi hoặc số dòng mỗi trang. Tránh `limit: 0` cho danh sách có thể tăng, kể cả bản ghi built-in/system.

Các danh sách Settings tích hợp dùng `DataTableSettingsTable` để bổ sung thao tác theo dòng, bộ chọn 10/20/50/100 dòng mỗi trang và `CommonPaginationBar`. Adapter này dành cho các trang tích hợp; extension có thể ghép `DataTable` với điều khiển phân trang theo server của riêng mình. Kích thước trang được lưu riêng theo từng danh sách Settings và khôi phục sau khi refresh.

## Tùy biến cột và chọn dòng

`useDataTableColumns().buildActionsColumn({ actions })` tạo cột menu `…`. `actions` có thể là mảng cố định hoặc hàm nhận bản ghi hiện tại. Chỉ trả về các thao tác mà bản ghi và quyền truy cập cho phép. Để chọn dòng, dùng `v-model:row-selection` gốc của bảng, cột checkbox và `getRowId` dựa trên định danh ổn định. Không suy ra trạng thái chọn từ vị trí dòng hoặc phân trang lần nữa trên kết quả đã phân trang.

Ở trang `/data/<table>` tích hợp, một cột có thể khai báo `metadata.tableCell.formatter` dưới dạng chuỗi chứa hàm `(metadata, value) => expression` trong phạm vi biểu thức được hỗ trợ. Kết quả có thể là chuỗi, số, boolean hoặc `{ text, color?, variant? }` để hiển thị; kết quả không hợp lệ/trống trở về giá trị ô thông thường (`_` nếu giá trị rỗng). Formatter chỉ chạy ở eApp và không thay đổi dữ liệu lưu trữ hay phản hồi của server. Trong extension, dùng renderer `cell` của cột để định dạng theo nhu cầu riêng.

## Khi bảng hiển thị không đúng

- Bảng trống: kiểm tra phản hồi có mảng `data` và các field được yêu cầu khớp với `accessorKey`.
- Sai tổng số hoặc thiếu trang: xác nhận filter đang dùng và `meta.filterCount` thuộc cùng một query trên server.
- Dòng biến mất khi refresh: giữ trang thành công gần nhất cho đến khi có phản hồi mới; chỉ dùng skeleton lúc tải đầu tiên.
- Thiếu một thao tác: kiểm tra quyền trên route backend và điều kiện của bản ghi trước khi thay đổi UI.
