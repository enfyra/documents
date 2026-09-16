---
slug: ung-dung/hook-va-handler/quan-ly-package
---

# Quản lý gói

Quản lý gói cho phép bạn cài đặt các gói NPM trực tiếp từ giao diện Enfyra và sử dụng chúng trong các trình xử lý và hook tùy chỉnh của bạn. Thay vì quản lý các phần phụ thuộc theo cách thủ công, bạn có thể tìm kiếm, cài đặt và định cấu hình các gói thông qua giao diện người dùng - chúng tự động có sẵn trong mã tùy chỉnh của bạn dưới dạng `$ctx.$pkgs.packagename`.

**Tài liệu liên quan**: [Trình xử lý tùy chỉnh](./custom-handlers.md) | [Móc và Trình xử lý](../../server/hooks-handlers/) | [Ví dụ về đăng ký người dùng](../../examples/user-registration-example.md)

## Tại sao nên sử dụng Quản lý gói?

- **Mở rộng chức năng**: Thêm các thư viện mạnh mẽ như axios, lodash, moment.js vào trình xử lý của bạn
- **Không có cấu hình**: Các gói có sẵn ngay lập tức trong mã tùy chỉnh của bạn mà không cần thiết lập
- **Giao diện trực quan**: Tìm kiếm và cài đặt các gói mà không cần chạm vào package.json
- **Ngữ cảnh handler được cô lập**: Các Server package đã cài đặt được cung cấp qua package handle theo từng task trong isolated executor. Dynamic script không nhận raw host stream, socket hoặc object nội bộ của package child.

## Cài đặt gói

### Bước 1: Truy cập cài đặt gói
1. Điều hướng đến **Gói** trong thanh bên
2. Nhấp vào nút **Cài đặt gói**

### Bước 2: Chọn loại gói hàng
Bạn sẽ thấy hai tùy chọn loại gói:
- **Gói máy chủ**: Để sử dụng trong các trình xử lý và hook tùy chỉnh (phía máy chủ)
- **Gói ứng dụng**: Để sử dụng trong các tiện ích mở rộng (trang, widget) trong trình duyệt

Chọn loại thích hợp cho trường hợp sử dụng của bạn.

### Bước 3: Tìm kiếm gói NPM
Bắt đầu nhập vào trường tìm kiếm để tìm các gói:
- **Tìm kiếm theo thời gian thực** với các đề xuất tự động khi bạn nhập
- **Thông tin gói phong phú** bao gồm mô tả và số phiên bản
- **Huy hiệu phiên bản** hiển thị phiên bản mới nhất hiện có
- **Biểu tượng gói** để nhận dạng trực quan

**Ví dụ về gói phổ biến:**
```
axios          # HTTP requests and API calls
lodash         # Utility functions for data manipulation
moment         # Date and time handling
uuid           # Generate unique identifiers
joi            # Data validation schemas
```
### Bước 4: Chọn và cấu hình
1. **Nhấp vào gói** từ kết quả thả xuống
2. **Tự động điền biểu mẫu** với tên gói, phiên bản và mô tả
3. **Tùy chỉnh chi tiết** nếu cần (mô tả, cờ, v.v.)
4. **Nhấp vào Cài đặt** để thêm gói vào dự án của bạn

**Quan trọng**: Trường tên gói được tự động điền và không thể chỉnh sửa - điều này đảm bảo độ phân giải gói chính xác.

## Sử dụng các gói đã cài đặt

Sau khi được cài đặt, các gói sẽ tự động có sẵn trong trình xử lý tùy chỉnh của bạn và nối thông qua đối tượng `$ctx.$pkgs`. Không cần nhập - chỉ cần truy cập chúng trực tiếp.

### Mẫu sử dụng cơ bản
```javascript
// In any custom handler or hook
export default async function handler({ $ctx }) {
  // Access installed packages via $ctx.$pkgs
  const packageName = $ctx.$pkgs.packagename;

  // Use the package normally
  return packageName.someMethod();
}
```
### Ví dụ thực tế

**Yêu cầu HTTP với Axios**
```javascript
// Custom handler for external API integration
export default async function handler({ $ctx }) {
  const axios = $ctx.$pkgs.axios;

  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/users');
    return {
      success: true,
      users: response.data
    };
  } catch (error) {
    return {
      success: false,
      error: error.message
    };
  }
}
```
**Xử lý dữ liệu bằng Lodash**
```javascript
// Handler for complex data transformation
export default async function handler({ $ctx }) {
  const _ = $ctx.$pkgs.lodash;

  // Simulate processing user data
  const users = $ctx.$body.users || [];

  // Group users by department and get counts
  const grouped = _.groupBy(users, 'department');
  const summary = _.mapValues(grouped, group => ({
    count: group.length,
    active: _.filter(group, 'isActive').length
  }));

  return {
    departmentSummary: summary,
    totalUsers: users.length
  };
}
```
**Thao tác ngày với khoảnh khắc**
```javascript
// Handler for date-based operations
export default async function handler({ $ctx }) {
  const moment = $ctx.$pkgs.moment;

  const startDate = moment($ctx.$query.startDate);
  const endDate = moment($ctx.$query.endDate);

  return {
    period: {
      start: startDate.format('YYYY-MM-DD'),
      end: endDate.format('YYYY-MM-DD'),
      days: endDate.diff(startDate, 'days')
    },
    timestamps: {
      created: moment().unix(),
      formatted: moment().format('YYYY-MM-DD HH:mm:ss')
    }
  };
}
```

## Consume readable stream từ Server package

Một số Server package, chẳng hạn `undici`, trả về readable response body thay vì giá trị đã buffer. Enfyra cung cấp package-backed readable dưới dạng `AsyncIterable<Uint8Array>` chỉ có một consumer, cùng các helper `$ctx.$streams` để đọc an toàn trước response hoặc consume toàn bộ body.

Chọn đúng một cách consume cho mỗi body:

- Dùng `$ctx.$streams.preflight()` trước `@RES.stream()` khi handler phải chờ raw byte đầu tiên trước khi commit public response.
- Dùng `$ctx.$streams.readText()` hoặc `readBytes()` khi handler phải consume và validate toàn bộ upstream body trước khi trả JSON hoặc response đã buffer.
- Dùng `for await...of` khi cần tự xử lý theo cơ chế pull.
- Chỉ truyền body trực tiếp vào `@RES.stream()` khi có thể commit response ngay và handler không cần retry trường hợp upstream chưa trả byte nào.

Một body không thể được consume hai lần. Không preflight hoặc collect một body rồi truyền body gốc vào `@RES.stream()`.

### Preflight trước khi commit streaming response

`preflight()` chờ chunk raw đầu tiên không rỗng. Bất kỳ byte nào cũng được tính là activity, kể cả SSE event chỉ chứa reasoning hoặc thinking. Helper không parse SSE và không chờ visible text.

```js
const upstream = await @PKGS.undici.request("https://api.example.com/stream", {
  method: "POST",
  headers: {
    authorization: `Bearer ${@ENV.UPSTREAM_API_KEY}`,
    "content-type": "application/json"
  },
  body: JSON.stringify(@BODY)
})

const guarded = await $ctx.$streams.preflight(upstream.body, {
  timeoutMs: 15_000
})

await @RES.stream(guarded.stream, {
  statusCode: upstream.statusCode,
  mimetype: upstream.headers["content-type"] || "text/event-stream"
})
```

`guarded.stream` relay lại chunk đầu đúng một lần rồi pull các chunk còn lại theo đúng thứ tự. `guarded.firstChunk` là bản sao an toàn để kiểm tra; thay đổi giá trị này không làm thay đổi replay stream.

### Retry upstream không có byte trong cùng task

Preflight timeout chỉ áp dụng cho upstream stream đó. Enfyra đóng source bị timeout mà không abort executor task, nên code ứng dụng có thể tạo request mới. Kernel không tự động retry network request.

```js
let selected = null

for (let attempt = 0; attempt < 2; attempt++) {
  const upstream = await @PKGS.undici.request("https://api.example.com/stream", {
    method: "POST",
    body: JSON.stringify(@BODY)
  })

  try {
    const guarded = await $ctx.$streams.preflight(upstream.body, {
      timeoutMs: 15_000
    })
    selected = { upstream, guarded }
    break
  } catch (error) {
    const retryableNoByte =
      error.code === "ERR_PACKAGE_STREAM_TIMEOUT" ||
      error.code === "ERR_PACKAGE_STREAM_EMPTY"

    if (!retryableNoByte || attempt === 1) throw error
  }
}

if (!selected) @THROW.externalService("upstream", "No stream is available")

await @RES.stream(selected.guarded.stream, {
  statusCode: selected.upstream.statusCode,
  mimetype: "text/event-stream"
})
```

Lỗi xảy ra sau chunk đầu là stream failure thông thường. Enfyra chuyển lỗi qua response-stream error path hiện có và không tự động retry.

### Consume và validate trước khi trả buffered response

`readText()` và `readBytes()` consume package stream mà không khởi động `@RES`. Cả hai mặc định giới hạn 8 MiB, áp dụng `maxBytes` trên raw bytes và reject source error thay vì trả partial success. `readText()` giữ nguyên ký tự UTF-8 bị tách qua nhiều chunk.

```js
const upstream = await @PKGS.undici.request("https://api.example.com/result", {
  method: "POST",
  body: JSON.stringify(@BODY)
})

const text = await $ctx.$streams.readText(upstream.body, {
  timeoutMs: 60_000,
  maxBytes: 8 * 1024 * 1024
})

const result = JSON.parse(text)
if (!result.id) @THROW.http(502, "Upstream response is missing id")

return { result }
```

Với dữ liệu binary, dùng `readBytes()` và xử lý `Uint8Array` trả về. Stream rỗng trả về chuỗi rỗng hoặc byte array có độ dài bằng 0.

### Lỗi stream và deadline

| Điều kiện | Kết quả |
|---|---|
| Không có chunk không rỗng trước `timeoutMs` | `ERR_PACKAGE_STREAM_TIMEOUT`; target stream bị đóng và task có thể tiếp tục |
| EOF trước chunk không rỗng đầu tiên | `ERR_PACKAGE_STREAM_EMPTY`; task có thể tiếp tục |
| Vượt `maxBytes` | `ERR_PACKAGE_STREAM_MAX_BYTES`; target stream bị đóng |
| Body bị consume nhiều hơn một lần | `ERR_PACKAGE_STREAM_ALREADY_CONSUMED` |
| Gọi `next()` đồng thời | `ERR_PACKAGE_STREAM_CONCURRENT_NEXT` |
| Handler/flow-step timeout hoặc client disconnect | Enclosing task và mọi stream thuộc task đều bị cancel |

Timeout của stream helper không thể kéo dài deadline của handler hoặc flow step. Hãy đặt handler timeout đủ cho toàn bộ upstream request, preflight hoặc collect, transform và response relay.

### Observer và transform là callback sau commit

`observer` và `transform` thuộc response relay của `@RES.stream()`. Chúng không phải first-byte hoặc pre-response primitive. Giá trị text là fragment đã decode UTF-8 tùy ý, không phải SSE event hoàn chỉnh; hãy buffer đến blank-line event boundary trước khi parse.

```js
await @RES.stream(guarded.stream, {
  observer: async (text, kind) => {
    // Best-effort inspection. Errors are ignored.
  },
  transform: async (text, kind) => {
    if (kind === "end") return undefined
    return text
  }
})
```

Trả về string từ `transform` để thay fragment, `null` để loại bỏ hoặc `undefined` để giữ nguyên bytes gốc. Transform error làm stream thất bại.

## Quản lý các gói đã cài đặt

### Xem tất cả các gói
1. Điều hướng đến **Gói** trong thanh bên
2. Nhấp vào **Gói máy chủ** để xem tất cả các gói máy chủ đã cài đặt

**Thông tin thẻ gói:**
- **Tên gói** có biểu tượng npm
- Huy hiệu **Phiên bản hiện tại**
- **Mô tả** và ngày cài đặt
- **Thao tác nhanh** (xem chi tiết, gỡ cài đặt)

### Cập nhật thông tin gói hàng
1. **Nhấp vào bất kỳ thẻ gói nào** để mở chế độ xem chi tiết
2. **Sửa đổi thông tin** như mô tả hoặc cờ
3. **Nhấp vào Lưu** để cập nhật siêu dữ liệu gói

> **Lưu ý**: Tên gói sẽ bị khóa sau khi cài đặt để tránh phá vỡ các tham chiếu mã hiện có.

### Cập nhật phiên bản gói
Để cập nhật gói lên một phiên bản cụ thể:

1. **Điều hướng đến trang chi tiết gói**
2. **Chỉnh sửa trường phiên bản** trong biểu mẫu để chỉ định phiên bản mong muốn
3. **Nhấp vào Lưu** để cài đặt phiên bản được chỉ định
4. **Xác minh bản cập nhật** - huy hiệu phiên bản sẽ hiển thị số phiên bản mới

**Quy trình cập nhật phiên bản mẫu:**
```
Current: lodash v4.17.20
↓ Edit version field to "4.17.21"
↓ Click Save
Updated: lodash v4.17.21
```
**Ghi chú cập nhật phiên bản:**
- **Thông số kỹ thuật thủ công**: Bạn phải nhập chính xác số phiên bản mà bạn muốn
- **Xác thực phiên bản**: Đảm bảo phiên bản tồn tại trên NPM trước khi cập nhật
- **Khả năng tương thích ngược**: Kiểm tra trình xử lý của bạn sau khi cập nhật phiên bản, đặc biệt là những thay đổi lớn về phiên bản
- **Kiểm tra NPM**: Truy cập [npmjs.com/package/packagename](https://npmjs.com) để xem các phiên bản có sẵn

### Xóa gói
1. **Mở chi tiết gói** bằng cách nhấp vào thẻ gói
2. **Nhấp vào nút Gỡ cài đặt** trong tiêu đề (biểu tượng thùng rác màu đỏ)
3. **Xác nhận xóa** khi được nhắc

**Cảnh báo**: Việc gỡ cài đặt sẽ xóa gói khỏi tất cả các trình xử lý và hook ngay lập tức. Trước tiên hãy đảm bảo rằng không có mã hoạt động nào phụ thuộc vào nó.

## Các trường hợp sử dụng gói phổ biến

### Gói thiết yếu cho hầu hết các dự án

**Tích hợp HTTP & API**
```javascript
// axios - HTTP requests
$ctx.$pkgs.axios.get('https://api.example.com/data')

// node-fetch - Alternative HTTP client
$ctx.$pkgs.fetch('https://api.example.com')
```
**Thao tác dữ liệu**
```javascript
// lodash - Comprehensive utilities
$ctx.$pkgs.lodash.groupBy(data, 'category')

// ramda - Functional programming
$ctx.$pkgs.ramda.filter(R.propEq('active', true))
```
**Xử lý ngày và giờ**
```javascript
// moment - Full-featured date library
$ctx.$pkgs.moment().format('YYYY-MM-DD')

// dayjs - Lightweight alternative
$ctx.$pkgs.dayjs().add(1, 'month')
```
**Xác thực & Bảo mật**
```javascript
// joi - Schema validation
$ctx.$pkgs.joi.object({ email: joi.string().email() })

// bcrypt - Password hashing
$ctx.$pkgs.bcrypt.hash(password, 10)
```
**Tiện ích**
```javascript
// uuid - Generate unique IDs
$ctx.$pkgs.uuid.v4()

// slugify - URL-friendly strings
$ctx.$pkgs.slugify('Hello World', { lower: true })
```
## Khắc phục sự cố

### Không tìm thấy gói
```javascript
//  Wrong - package not installed
const axios = $ctx.$pkgs.axios; // undefined

//  Correct - install package first via UI
// Then access as shown above
```
**Giải pháp:**
1. Xác minh gói được cài đặt trong **Gói máy chủ gói**
2. Kiểm tra chính tả tên gói (phân biệt chữ hoa chữ thường)
3. Đảm bảo bạn đang sử dụng cú pháp `$ctx.$pkgs.exactname`

### Vấn đề tìm kiếm gói
- **Nhập ít nhất 2 ký tự** trước khi kích hoạt tìm kiếm
- **Thử từ khóa khác** nếu không có kết quả nào xuất hiện
- **Kiểm tra kết nối internet** để biết các truy vấn NPM bên ngoài

### Sự cố cài đặt
- **Xác minh gói tồn tại** trên [npmjs.com](https://npmjs.com)
- **Thử xóa bộ nhớ cache của trình duyệt** và làm mới
- **Kiểm tra chính tả tên gói** trong tìm kiếm

### Vấn đề cập nhật phiên bản
- **Kiểm tra tính tương thích của gói** trước khi cập nhật các phiên bản chính
- **Kiểm tra trình xử lý/hook** sau khi cập nhật phiên bản để đảm bảo tính tương thích
- **Hoàn nguyên nếu cần** bằng cách gỡ cài đặt và cài đặt lại phiên bản trước

## Đề xuất gói

**Starter Pack** - Các gói cần thiết cho hầu hết các dự án:
- `axios` - Yêu cầu HTTP và tích hợp API
- `lodash` - Thao tác và tiện ích dữ liệu
- `khoảnh khắc` - Thao tác ngày/giờ
- `uuid` - Tạo mã định danh duy nhất
- `joi` - Xác thực đầu vào

**Gói xử lý dữ liệu**:
- `csv-parser` - Phân tích tệp CSV
- `fast-xml-parser` - Phân tích cú pháp XML
- `jsonwebtoken` - Xử lý mã thông báo JWT
- `bcrypt` - Mã hóa mật khẩu

**Gói tích hợp**:
- `nodemailer` - Gửi email
- `twilio` - SMS và liên lạc
- `sọc` - Xử lý thanh toán
- `aws-sdk` - Dịch vụ web của Amazon

**Bắt đầu với các gói Starter Pack** - chúng đáp ứng 80% nhu cầu phụ trợ thông thường và phối hợp tốt với nhau.

## Các phương pháp hay nhất

### Quản lý gói
- **Kiểm tra các bản cập nhật**: Thường xuyên kiểm tra NPM để biết các phiên bản mới hơn của các gói đã cài đặt của bạn
- **Kiểm soát phiên bản thủ công**: Chỉ định phiên bản chính xác khi cập nhật để duy trì tính nhất quán
- **Kiểm tra sau khi cập nhật**: Luôn kiểm tra trình xử lý và hook của bạn sau khi cập nhật phiên bản
- **Dọn dẹp các gói không sử dụng**: Loại bỏ các gói không còn được sử dụng để giữ cho hệ thống gọn gàng

### Chiến lược phiên bản
- **Cập nhật được kiểm soát**: Chỉ định phiên bản theo cách thủ công thay vì tự động cập nhật để tránh làm hỏng các thay đổi
- **Phiên bản nghiên cứu**: Kiểm tra ghi chú phát hành gói và nhật ký thay đổi trước khi cập nhật
- **Thử nghiệm trong môi trường chạy thử**: Đối với hệ thống sản xuất, trước tiên hãy cập nhật phiên bản thử nghiệm trong môi trường chạy thử
- **Ghim các phiên bản quan trọng**: Giữ các phiên bản ổn định cho các chức năng quan trọng
- **Phụ thuộc tài liệu**: Theo dõi trình xử lý/hook nào sử dụng gói và phiên bản nào

---

## Sử dụng gói ứng dụng trong tiện ích mở rộng

Gói ứng dụng cho phép bạn sử dụng gói npm trực tiếp trong Tiện ích mở rộng của mình (trang tùy chỉnh, tiện ích con và tiện ích mở rộng shell toàn cầu). Không giống như Gói phụ trợ chạy trên máy chủ, Gói ứng dụng chạy trên trình duyệt và hoàn hảo cho:
- Biểu đồ và trực quan hóa (chart.js, echarts)
- Tiện ích (lodash, dayjs, axios)
- Thành phần giao diện người dùng
- Thư viện xử lý dữ liệu

### Cài đặt gói ứng dụng

1. Điều hướng đến **Gói** trong thanh bên
2. Nhấp vào **Cài đặt gói**
3. Chọn **Gói ứng dụng** làm loại
4. Tìm kiếm và chọn gói của bạn
5. Nhấp vào **Cài đặt**

### Sử dụng gói trong tiện ích mở rộng

Sau khi cài đặt, các gói sẽ có sẵn thông qua hàm `getPackages()`:

**Phương pháp 1: Phá hủy (Khuyến nghị)**
```vue
<script setup>
onMounted(async () => {
  const { chartjs, dayjs } = await getPackages();

  const formattedDate = dayjs().format('YYYY-MM-DD');
  console.log('Date:', formattedDate);
});
</script>
```
**Phương pháp 2: Mảng gói**
```vue
<script setup>
onMounted(async () => {
  const packages = await getPackages(['chartjs', 'dayjs']);

  const chart = packages.chartjs;
  const date = packages.dayjs().format('YYYY-MM-DD');
});
</script>
```
**Cách 3: Tất cả các gói**
```vue
<script setup>
onMounted(async () => {
  const packages = await getPackages();

  const chart = packages.chartjs;
  const date = packages.dayjs().format('YYYY-MM-DD');
});
</script>
```
### Ví dụ hoàn chỉnh: Biểu đồ trong phần mở rộng
```vue
<template>
  <UCard>
    <template #header>
      <h3>Sales Chart</h3>
    </template>

    <canvas ref="chartCanvas"></canvas>
  </UCard>
</template>

<script setup>
const chartCanvas = ref(null);

onMounted(async () => {
  const { Chart } = await getPackages(['chart.js']);

  const ctx = chartCanvas.value.getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: ['Jan', 'Feb', 'Mar'],
      datasets: [{
        label: 'Sales',
        data: [12, 19, 3],
        backgroundColor: 'rgba(59, 130, 246, 0.5)'
      }]
    }
  });
});
</script>
```
### Gói ứng dụng phổ biến

**Biểu đồ & Trực quan hóa:**
- `chart.js` - Biểu đồ và đồ thị
- `vue-chartjs` - Trình bao bọc Vue cho Chart.js
- `echarts` - Biểu đồ doanh nghiệp
- `apexcharts` - Thư viện biểu đồ hiện đại

**Tiện ích:**
- `dayjs` - Thao tác ngày tháng (nhẹ)
- `lodash` - Các hàm tiện ích
- `axios` - Yêu cầu HTTP
- `uuid` - Tạo ID duy nhất

**Dữ liệu & Biểu mẫu:**
- `vuedraggable` - Kéo và thả
- `sortablejs` - Danh sách có thể sắp xếp
- `papaparse` - Phân tích cú pháp CSV

### Cách thức hoạt động

**Tự động phát hiện:**
Hệ thống tự động phát hiện gói bạn sử dụng bằng cách quét mã:
- Quét các lệnh gọi `getPackages()`
- Xác định tên gói từ việc phá hủy
- Chỉ tải các gói bạn thực sự sử dụng

**Tải Lười:**
- Tải gói khi truy cập lần đầu
- Các cuộc gọi tiếp theo sử dụng phiên bản được lưu trong bộ nhớ đệm
- Tải trang ban đầu nhanh hơn

**Quản lý phụ thuộc:**
- Phụ thuộc tải tự động
- Đảm bảo đúng đơn hàng
- Không cần thiết lập thủ công

### Các phương pháp hay nhất

1. **Chỉ định rõ ràng các gói** - Hiệu suất tốt hơn tải tất cả   
   ```javascript
   onMounted(async () => {
     const { chartjs } = await getPackages();
   });
   ```
2. **Tải onMounted** - Đảm bảo thành phần đã sẵn sàng   
   ```javascript
   onMounted(async () => {
     const packages = await getPackages();
   });
   ```
3. **Kiểm tra tính khả dụng của gói**   
   ```javascript
   const packages = await getPackages(['chartjs']);
   if (!packages.chartjs) {
     toast.add({
       title: 'Package not installed',
       description: 'Install chart.js from Packages',
       color: 'red'
     });
   }
   ```
### Khắc phục sự cố

**Gói không tải:**
- Xác minh gói được cài đặt dưới dạng **Gói ứng dụng** (không phải Phần cuối)
- Kiểm tra bảng điều khiển trình duyệt để tìm lỗi
- Đảm bảo tên gói khớp với cài đặt

**Nhiều phiên bản:**
- Chỉ có một phiên bản cho mỗi tên gói
- Cập nhật phiên bản trong cài đặt gói nếu cần

**Hiệu suất:**
- Các gói được lưu trữ sau lần tải đầu tiên
- Sử dụng các gói cụ thể để có hiệu suất tốt hơn

**Để biết thêm ví dụ, hãy xem [Hệ thống tiện ích mở rộng](../extension-system.md#su-dung-goi-npm-trong-tien-ich-mo-rong)**
