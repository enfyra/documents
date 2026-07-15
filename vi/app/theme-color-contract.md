---
slug: ung-dung/hop-dong-mau-sac-theme
---

# Hướng dẫn về theme cho ứng dụng

Hãy dùng hướng dẫn này khi xây dựng trang, widget hoặc extension cho Enfyra App để giao diện đồng bộ với trang quản trị ở cả chế độ sáng và tối.

Nguyên tắc ngắn gọn là: chọn màu theo ý nghĩa, không chọn theo tên bảng màu. Đừng dùng `green`, `violet`, `cyan`, mã màu hex hoặc biến CSS thô chỉ vì chúng trông gần giống màu cần dùng. Hãy dùng các class của Enfyra và màu ngữ nghĩa của Nuxt UI để giao diện tự thích ứng với theme hiện tại của ứng dụng.

## Quy tắc nhanh

- Dùng `color="primary"` cho hành động chính hoặc đối tượng đang được chọn trên màn hình hiện tại.
- Chỉ dùng `color="success"`, `color="warning"`, `color="error"` hoặc `color="info"` khi màu sắc thể hiện trạng thái.
- Giữ các panel lớn ở màu trung tính. Chỉ dùng màu trạng thái cho badge nhỏ, biểu tượng hoặc đoạn văn bản trạng thái ngắn bên trong panel.
- Dùng các class `eapp-surface-*` cho card, hàng và panel.
- Dùng các class `eapp-text-*` để thể hiện cấp bậc văn bản.
- Dùng các class `eapp-primary-*` cho mục đang chọn, phần tiến độ đang hoạt động, biểu tượng chính và điểm nhấn nhận diện.
- Dùng `eapp-icon-tile` cho ô biểu tượng hình vuông thay vì tạo ô bằng padding.
- Dùng các prop footer của `CommonModal` và `CommonDrawer` cho những hành động Hủy, Lưu và Xóa theo chuẩn.
- Dùng `UTabs` cho các khu vực dạng tab thay vì tự tạo thanh tab.

## Lựa chọn thường dùng

| Thành phần cần xây dựng | Nên dùng |
| --- | --- |
| Nút lưu/tạo/chạy chính | `<UButton color="primary" variant="solid" />` |
| Quay lại, làm mới, lọc, đóng hoặc hành động phụ | `<UButton color="neutral" variant="outline" />` hoặc `variant="ghost"` |
| Hành động phá hủy dữ liệu | `<UButton color="error" />` |
| Badge trạng thái | `<UBadge color="success|warning|error|info|neutral" variant="soft" />` |
| Card hoặc panel thông thường | `eapp-surface-card` |
| Bề mặt lõm bên trong hoặc nền của thanh đo | `eapp-surface-muted` |
| Hiệu ứng hover cho hàng/card có thể nhấp | `eapp-surface-hover` |
| Tiêu đề hoặc giá trị chính | `eapp-text-primary` |
| Nội dung chính | `eapp-text-secondary` |
| Nhãn, văn bản hướng dẫn, metadata | `eapp-text-tertiary` |
| Khối đại diện cho đối tượng đang chọn/hiện tại | `eapp-primary-surface` kết hợp với kiểu khung card thông thường như `border` và `eapp-radius-panel` |
| Chip nhỏ đang chọn hoặc ô biểu tượng | `eapp-primary-soft eapp-icon-tile` |
| Biểu tượng chính hoặc văn bản chính nội dòng | `eapp-primary-text` |
| Phần đã hoàn thành của thanh tiến độ hoặc thanh đo đang hoạt động | `eapp-primary-solid` |
| Đường phân cách giữa các hàng | `eapp-divide-y` |

## Hành động trong modal và drawer

Với footer thông thường của modal hoặc drawer, hãy truyền các hành động qua prop. Cách này giúp nút Hủy, Lưu, Xóa cùng trạng thái loading, disabled và khoảng cách luôn nhất quán trong toàn ứng dụng.

```vue
<CommonModal
  v-model:open="open"
  :cancel-action="{ label: 'Hủy', onClick: () => (open = false) }"
  :primary-action="{ label: 'Lưu', loading: saving, disabled: !canSave, onClick: save }"
>
  <template #header>
    <h3 class="text-lg font-semibold eapp-text-primary">Chỉnh sửa cài đặt</h3>
  </template>

  <template #body>
    <FormEditor v-model="form" table-name="settings" />
  </template>
</CommonModal>
```

Hãy dùng các prop hành động sau:

| Prop | Mục đích sử dụng |
| --- | --- |
| `cancelAction` | Hủy, bỏ thay đổi hoặc thoát chế độ chỉnh sửa |
| `primaryAction` | Hành động an toàn chính như Lưu, Tạo, Cập nhật hoặc Chạy |
| `dangerAction` | Xóa hoặc thực hiện hành động phá hủy không thể hoàn tác |
| `leadingActions` | Các hành động phụ hiển thị ở phía trái footer |
| `footerHint` | Thông báo hướng dẫn ngắn bên cạnh các hành động trong footer |

Theo mặc định, nút Hủy dùng kiểu viền trung tính. Hãy dùng `dangerAction` cho hành động phá hủy không thể hoàn tác như xóa, thu hồi hoặc loại bỏ vĩnh viễn.

```vue
:cancel-action="{ label: 'Đóng', tone: 'neutral', onClick: () => (open = false) }"
```

Trong hộp thoại xác nhận bỏ thay đổi, hành động an toàn nên là hành động chính:

```vue
:cancel-action="{ label: 'Tiếp tục chỉnh sửa', tone: 'primary', onClick: () => (discardOpen = false) }"
```

Chỉ dùng `#footer` tùy chỉnh khi footer cần bố cục riêng hoặc chứa nội dung không phải nút.

Với nội dung giữ chỗ trong lúc tải, hãy dùng `USkeleton` hoặc các component loading dùng chung. Ứng dụng ánh xạ màu skeleton trên toàn hệ thống để nội dung giữ chỗ luôn dễ nhìn ở cả chế độ sáng và tối, đồng thời tự theo màu nhấn hiện tại mà không cần hardcode màu từ bảng màu.

## Tab

Dùng `UTabs` cho các khu vực của trang, panel cài đặt và biểu mẫu nhiều bước. Enfyra định kiểu chung cho chỉ báo tab đang hoạt động và chưa hoạt động, nên tab trong extension sẽ tự động đồng bộ với các trang hệ thống.

```vue
<UTabs
  v-model="activeTab"
  :items="[
    { label: 'Tổng quan', icon: '', value: 'overview' },
    { label: 'Hoạt động', icon: '', value: 'activity' }
  ]"
/>
```

Tránh tự tạo thanh tab trừ khi bố cục thực sự không phải là giao diện dạng tab.

## Card và danh sách

Dùng bề mặt trung tính cho nội dung thông thường:

```vue
<article class="eapp-surface-card p-4">
  <div class="flex items-start justify-between gap-3">
    <div>
      <p class="text-sm eapp-text-tertiary">Dự án</p>
      <p class="mt-2 text-2xl font-semibold eapp-text-primary">{{ total }}</p>
    </div>

    <span class="eapp-primary-soft eapp-icon-tile">
      
    </span>
  </div>
</article>
```

Với các hàng:

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
      {{ item.enabled ? 'Đã bật' : 'Đã tắt' }}
    </UBadge>
  </button>
</section>
```

## Mục được chọn hoặc đang hoạt động

Chỉ dùng bề mặt primary khi toàn bộ khối đang được chọn, là đối tượng hiện tại hoặc đang hoạt động. `eapp-primary-surface` cung cấp màu nền nhận diện cho trạng thái được chọn; đồng thời vẫn giữ kiểu khung card thông thường như `border` và `eapp-radius-panel` trên phần tử:

```vue
<article class="eapp-primary-surface eapp-radius-panel border p-4">
  <div class="flex items-center gap-3">
    <span class="eapp-primary-soft eapp-icon-tile">
      
    </span>
    <div>
      <p class="font-semibold eapp-text-primary">{{ name }}</p>
      <p class="text-sm eapp-text-tertiary">Đang được chọn</p>
    </div>
  </div>
</article>
```

Không dùng `eapp-primary-surface` cho mọi card trong một lưới. Nếu mọi thứ đều được làm nổi bật thì sẽ không còn gì thực sự nổi bật.

## Trạng thái

Màu trạng thái nên được dùng ở phạm vi nhỏ và cho mục đích cụ thể:

```vue
<section class="eapp-surface-card p-4">
  <div class="flex items-center justify-between gap-3">
    <p class="font-semibold eapp-text-primary">Đối soát</p>
    <UBadge color="warning" variant="soft">Cần xem lại</UBadge>
  </div>
</section>
```

Tránh tô toàn bộ panel thành màu xanh lá, vàng hoặc đỏ. Hãy dùng panel trung tính với một badge trạng thái bên trong.

## Tiến độ

Dùng nền thanh trung tính và phần tiến độ màu primary:

```vue
<div class="h-1.5 overflow-hidden eapp-radius-pill eapp-surface-muted">
  <div class="h-full eapp-primary-solid" :style="{ width: progressWidth }"></div>
</div>
```

## Những điều cần tránh

Không viết:

```vue
<UButton color="violet">Lưu</UButton>
<UBadge color="green">Đang hoạt động</UBadge>
<div class="bg-slate-950 text-gray-200 border-gray-700"></div>
<div class="text-[var(--some-color)] bg-[var(--some-bg)]"></div>
<div style="color: #14b8a6"></div>
```

Hãy viết:

```vue
<UButton color="primary">Lưu</UButton>
<UBadge color="success">Đang hoạt động</UBadge>
<div class="eapp-surface-card eapp-text-primary"></div>
```

Nhờ đó, cùng một extension vẫn hiển thị chính xác khi người vận hành đổi màu nhấn của ứng dụng hoặc chuyển giữa chế độ sáng và tối.
