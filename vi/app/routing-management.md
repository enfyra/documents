---
slug: ung-dung/quan-ly-dieu-huong
---

# Quản lý định tuyến

Quản lý định tuyến cho phép bạn tạo các điểm cuối API tùy chỉnh được **máy chủ phụ trợ** của bạn phục vụ. Theo mặc định, Enfyra tự động tạo điểm cuối API REST cho mọi bảng bạn tạo (như `/enfyra_table`). Trình quản lý định tuyến cho phép bạn tạo **điểm cuối tùy chỉnh** như `/products` hoặc `/users` cho bất kỳ mục đích nào bạn cần.

**Quan trọng**: Tất cả các tuyến được tạo và phân phát bởi **máy chủ phụ trợ**, không phải ứng dụng giao diện người dùng. Giao diện người dùng sử dụng các điểm cuối API này thông qua các yêu cầu HTTP.

## Thuộc tính tuyến đường

### Cấu hình cơ bản
- **Đường dẫn**: Đường dẫn điểm cuối tùy chỉnh (ví dụ: `/assets/:id` thay vì `/enfyra_file`)
- **Biểu tượng**: Mã định danh trực quan cho tuyến đường (sử dụng biểu tượng Lucide)
- **Mô tả**: Mô tả mà con người có thể đọc được về mục đích của tuyến đường
- **Trạng thái**: Tuyến đường được bật hay tắt
- **Tuyến hệ thống**: Cho biết đây có phải là tuyến hệ thống cốt lõi hay không

### Cấu hình nâng cao
- **Bảng chính**: Bảng chính mà tuyến đường này phục vụ (xem bên dưới để biết chi tiết)
- **Các phương thức có sẵn**: Phương thức ghi lại tuyến đường này hỗ trợ. Nhãn phương thức tồn tại trong `enfyra_method.name`; quản lý bản ghi phương pháp và màu huy hiệu từ **Cài đặt > Phương thức**.
- **Quyền định tuyến**: Quy tắc kiểm soát quyền truy cập cho điểm cuối này
- **Trình xử lý**: Logic xử lý yêu cầu tùy chỉnh (xem [Trình xử lý tùy chỉnh](hooks-handlers/custom-handlers.md))
- **Hooks**: Các sự kiện trong vòng đời và quá trình xử lý tùy chỉnh (Pre-Hooks và Post-Hooks)
- **Phương thức công khai**: Xác định phương thức HTTP nào là công khai (không yêu cầu xác thực) so với phương thức riêng tư (được bảo vệ bằng vai trò)

## Tạo tuyến đường tùy chỉnh

### Bước 1: Truy cập Route Manager
1. Điều hướng đến **Cài đặt > Tuyến đường**
2. Nhấp vào nút **"Tạo lộ trình"**

### Bước 2: Cấu hình cơ bản
Định cấu hình các thuộc tính tuyến đường thiết yếu:
- **Đường dẫn**: Đặt điểm cuối tùy chỉnh của bạn (ví dụ: `/v1/products`, `/user-profiles`)
- **Biểu tượng**: Chọn biểu tượng Lucide để nhận dạng trực quan
- **Mô tả**: Thêm mô tả rõ ràng về mục đích của tuyến đường
- **Đã bật**: Bật tắt để kích hoạt tuyến đường
- **Các phương thức có sẵn**: Chọn các phương thức mà tuyến đường hỗ trợ. Nếu thiếu một phương thức, hãy mở **Cài đặt > Phương thức** hoặc sử dụng hành động chọn `+` để tạo phương thức đó.

Xem [Quản lý phương pháp](./method-management.md) để biết bản ghi phương pháp và cấu hình màu huy hiệu.

### Bước 3: Link tới Main Table
Bước quan trọng nhất là kết nối tuyến đường của bạn với nguồn dữ liệu:
1. Nhấp vào **bộ chọn quan hệ** (biểu tượng bút chì) bên cạnh **Bảng chính**
2. Tìm kiếm và chọn bảng mà tuyến đường này sẽ phục vụ
3. **Hoạt động CRUD tự động**: Sau khi được liên kết, tuyến tùy chỉnh của bạn có thể cung cấp:
   - `GET /your-route` - Liệt kê tất cả các bản ghi
   - `POST /your-route` - Tạo bản ghi mới
   - `PATCH /your-route/:id` - Cập nhật bản ghi
   - `DELETE /your-route/:id` - Xóa bản ghi

4. **Kiểm soát phương thức công khai**: Các phương thức HTTP được chỉ định trong **Phương thức công khai** trở thành **công khai** (khách có thể truy cập mà không cần xác thực). Các phương thức không được liệt kê vẫn **riêng tư** và yêu cầu quyền xác thực và vai trò phù hợp.

Tuyến kế thừa tất cả các trường, quy tắc xác thực và mối quan hệ của bảng mà không yêu cầu cấu hình bổ sung.

Xem [Hệ thống chọn quan hệ](relation-picker.md) để biết cách sử dụng chi tiết giao diện lựa chọn quan hệ.

### Bước 4: Lưu và kiểm tra
1. Nhấp vào **Lưu** để tạo lộ trình
2. Điểm cuối tùy chỉnh sẽ hoạt động ngay lập tức
3. Tất cả các trang quản lý dữ liệu đều tự động sử dụng điểm cuối mới
4. Lệnh gọi API trên toàn hệ thống chuyển sang đường dẫn tùy chỉnh

### Bước 5: Định cấu hình luồng thực thi (Trình xử lý & Móc)Sau khi tạo tuyến, bạn có thể định cấu hình luồng thực thi bằng trình xử lý và hook. Phần **Hình ảnh hóa luồng thực thi** cung cấp bản trình bày trực quan về cách xử lý các yêu cầu cho từng phương thức HTTP.

#### Tìm hiểu luồng thực thi

Luồng thực thi hiển thị trình tự:
1. **Pre-Hooks** (được thực thi trước trình xử lý, theo thứ tự ưu tiên)
2. **Trình xử lý** (logic xử lý chính hoặc CRUD mặc định nếu không có trình xử lý tùy chỉnh)
3. **Post-Hooks** (được thực thi sau trình xử lý, theo thứ tự ưu tiên)

#### Bảng chính và Trình xử lý mặc định

Khi một tuyến đường có **Bảng chính** được định cấu hình:
- Nếu không có trình xử lý tùy chỉnh nào tồn tại cho một phương thức HTTP cụ thể, hệ thống sẽ tự động sử dụng **trình xử lý mặc định** cung cấp các thao tác CRUD tiêu chuẩn
- Trình xử lý mặc định xuất hiện trong phần trực quan hóa luồng thực thi cho tất cả các phương thức không có trình xử lý tùy chỉnh
- Bạn có thể click vào trình xử lý mặc định để tạo trình xử lý tùy chỉnh cho phương thức đó

#### Thêm trình xử lý

1. Trong trang chi tiết tuyến đường, cuộn đến phần **Quy trình thực thi**
2. Nhấp vào nút **"Thêm trình xử lý"**
3. Cấu hình trình xử lý:
   - **Tên**: Tên mô tả cho trình xử lý
   - **Phương thức**: Chọn phương thức HTTP ghi lại các quy trình xử lý này. Các phương thức CRUD tích hợp là `GET`, `POST`, `PATCH` và `DELETE`; các phương thức tùy chỉnh như `PUT` trước tiên phải tồn tại trong **Cài đặt > Phương thức**.
   - **Logic**: Viết mã JavaScript tùy chỉnh của bạn
   - **Đã bật**: Chuyển đổi để kích hoạt/hủy kích hoạt
4. Nhấp vào **Lưu** để tạo trình xử lý

**Lưu ý**: Khi một trình xử lý tùy chỉnh tồn tại cho một phương thức, nó sẽ thay thế trình xử lý CRUD mặc định cho phương thức cụ thể đó.

#### Thêm móc

Hook cho phép bạn thêm logic xử lý trước (Pre-Hooks) hoặc sau (Post-Hooks) việc thực thi trình xử lý.

**Pre-Hooks** - Thực thi trước trình xử lý:
1. Nhấp vào nút **"Thêm móc trước"**
2. Cấu hình hook:
   - **Tên**: Tên mô tả
   - **Là toàn cầu**: Chọn mục này để áp dụng hook cho tất cả các tuyến đường (không chỉ tuyến đường này)
   - **Phương thức**: Chọn bản ghi phương thức HTTP nào mà hook này áp dụng (có thể chọn nhiều)
   - **Ưu tiên**: Các số thấp hơn thực hiện trước (ví dụ: số 1 thực hiện trước số 2)
   - **Logic**: Viết mã JavaScript của bạn
   - **Đã bật**: Chuyển đổi để kích hoạt/hủy kích hoạt
3. Nhấp vào **Lưu**

**Post-Hooks** - Thực thi sau trình xử lý:
1. Nhấp vào nút **"Thêm bài đăng"**
2. Cấu hình các trường tương tự như Pre-Hooks
3. Nhấp vào **Lưu**

**Móc toàn cầu**:
- Khi **Is Global** được bật, hook sẽ áp dụng cho **tất cả các tuyến** trong hệ thống
- Các móc nối chung xuất hiện trong luồng thực thi để khớp với các phương thức định tuyến
- Sử dụng móc nối chung cho các mối quan tâm xuyên suốt như ghi nhật ký, kiểm tra xác thực hoặc thông báo sẽ chạy cho mọi yêu cầu

#### Luồng thực thi trực quan

Trực quan hóa luồng thực thi:
- Thực thi nhóm theo phương thức HTTP (`GET`, `POST`, `PATCH`, `DELETE`)
- Hiển thị trình tự Pre-Hooks > Handler > Post-Hooks cho từng phương thức
- Hiển thị số ưu tiên móc
- Cho phép nhấp vào bất kỳ nút nào để chỉnh sửa nó
- Hiển thị trạng thái bật/tắt

**Quy tắc hiển thị móc**:
- **Móc toàn cầu** (isGlobal = true) xuất hiện để khớp với các phương thức định tuyến
- **Móc dành riêng cho phương pháp** chỉ xuất hiện cho các phương thức đã chọn của chúng
- **Trình xử lý mặc định** xuất hiện cho tất cả các phương thức không có trình xử lý tùy chỉnh (khi Bảng chính được định cấu hình)

#### Chỉnh sửa và quản lý

- **Nhấp vào bất kỳ nút nào** trong hình ảnh trực quan để chỉnh sửa nó
- **Xóa các nút** trực tiếp từ giao diện chỉnh sửa
- **Bật/Tắt** móc mà không cần chỉnh sửa chúng
- Các thay đổi có hiệu lực ngay lập tức### Bước 6: Định cấu hình quyền định tuyến (Tùy chọn)
Sau khi tạo tuyến đường, bạn có thể thiết lập kiểm soát truy cập chi tiết:

1. **Truy cập trang chỉnh sửa** của tuyến đường mới tạo của bạn
2. **Cuộn đến phần Quyền định tuyến** (chỉ có trên trang chỉnh sửa, không có trong quá trình tạo)
3. **Nhấp vào "Thêm quyền"** để mở ngăn cấu hình quyền
4. **Định cấu hình cài đặt quyền:**
   - **Vai trò**: Chọn vai trò nào có quyền truy cập (lựa chọn duy nhất thông qua bộ chọn quan hệ)
   - **Phương thức**: Chọn phương thức REST mà vai trò này có thể truy cập (nhiều lựa chọn): `GET`, `POST`, `PATCH`, `DELETE`
   - **Người dùng được phép**: Chọn những người dùng cụ thể bỏ qua các hạn chế về vai trò (nhiều lựa chọn thông qua bộ chọn quan hệ)
   - **Đã bật**: Chuyển đổi để kích hoạt/hủy kích hoạt quy tắc cấp phép này
   - **Mô tả**: Ghi lại mục đích của quyền này

5. **Lưu quyền** - quyền này có hiệu lực ngay lập tức

**Quan trọng**: Quyền của Tuyến đường hoạt động độc lập với cấu hình tuyến đường chính. Các thay đổi về quyền (tạo, chỉnh sửa, xóa) được áp dụng ngay lập tức mà không cần nút "Lưu" trong hành động tiêu đề.

### Quản lý quyền của tuyến đường
Sau khi tạo quyền, chúng sẽ xuất hiện dưới dạng danh sách trong phần Quyền định tuyến:
- **Mục cấp phép** hiển thị mô tả (hoặc "Không có mô tả" nếu trống)
- **Chỉ báo trạng thái** hiển thị quyền được bật hay tắt
- **Hành động** bao gồm các tùy chọn chỉnh sửa và xóa cho mỗi quyền
- **Nhiều quyền** có thể được định cấu hình cho cùng một tuyến đường với các vai trò/phương thức khác nhau
- **Cập nhật tức thì**: Tất cả các thay đổi về quyền sẽ có hiệu lực ngay lập tức, tách biệt với các sửa đổi về tuyến đường

Xem [Hệ thống bộ chọn quan hệ](relation-picker.md) để biết chi tiết về cách chọn vai trò và người dùng thông qua giao diện quan hệ.

### Truy cập GraphQL

GraphQL được bật cho mỗi bảng từ trình chỉnh sửa bảng bằng nút chuyển đổi **GraphQL** để ghi bản ghi `enfyra_graphql` của bảng. Nó không được kích hoạt từ Quyền định tuyến.

Các yêu cầu GraphQL hiện yêu cầu mã thông báo Bearer. REST `publicMethods` và các phương thức cấp phép định tuyến không làm cho GraphQL ẩn danh. Các truy vấn và đột biến GraphQL vẫn sử dụng cùng một siêu dữ liệu bảng, trạng thái xuất bản trường, lược đồ đầu vào/đầu ra được tạo, bộ bảo vệ và hành vi của kho lưu trữ như thời gian chạy của máy chủ.

Để biết cú pháp truy vấn và bộ lọc, hãy xem **[Lọc truy vấn](../server/query-filtering.md)**.

## Tuyến đường hệ thống

Các tuyến hệ thống (được đánh dấu bằng huy hiệu "Tuyến hệ thống") là các điểm cuối Enfyra cốt lõi hỗ trợ giao diện quản trị. Các tuyến đường này được tạo tự động và thường không được sửa đổi trừ khi bạn hiểu được hàm ý.

Ví dụ bao gồm các tuyến đường cho:
- Quản lý tập tin và lưu trữ
- Quản lý định nghĩa tuyến đường
- Cấu hình thực đơn
- Hệ thống cấp phép và người dùng

## Phương pháp xuất bản và kiểm soát quyền truy cập

Trường **Phương thức công khai** kiểm soát các yêu cầu xác thực cho từng phương thức HTTP.

**Để biết thông tin chi tiết đầy đủ về quyền, vai trò và Người dùng được phép, hãy xem [Trình tạo quyền](./permission-builder.md).**

### Phương thức công khai và Phương thức riêng tư
- **Phương thức công khai**: Truy cập công khai, không cần xác thực
- **Phương pháp chưa được công bố**: Yêu cầu xác thực và quyền thích hợp

### Tham khảo nhanh
1. **Phương pháp công cộng**: Truy cập công khai
2. **Người dùng được phép**: Bỏ qua các hạn chế về vai trò  
3. **Quyền theo vai trò**: Quyền truy cập dựa trên vai trò tiêu chuẩn
4. **Không có quyền truy cập**: Bị từ chối

## Hành vi tuyến đường tùy chỉnh

### Hoạt động CRUD mặc địnhKhi bạn tạo một tuyến đường và liên kết nó với Bảng chính, Enfyra sẽ tự động cung cấp các hoạt động CRUD tiêu chuẩn:

**API REST:**
- `GET /your-route` - Liệt kê các bản ghi
- `POST /your-route` - Tạo bản ghi
- `PATCH /your-route/:id` - Cập nhật bản ghi
- `DELETE /your-route/:id` - Xóa bản ghi

**API GraphQL:**
- `truy vấn { your_table(...) }` - Bản ghi truy vấn
- `mutation { create_your_table(...) }` - Tạo bản ghi
- `mutation { update_your_table(...) }` - Cập nhật bản ghi
- `mutation { delete_your_table(...) }` - Xóa bản ghi

### Ghi đè trình xử lý tùy chỉnh
Bạn có thể thay thế bất kỳ thao tác mặc định nào bằng logic nghiệp vụ tùy chỉnh bằng cách tạo trình xử lý:

1. **Tạo tuyến** với cấu hình Bảng chính
2. **Thêm trình xử lý tùy chỉnh** thông qua phần Luồng thực thi để ghi đè các phương thức HTTP cụ thể
3. **Trình xử lý được ưu tiên** - khi trình xử lý tồn tại cho một tuyến+phương thức, nó sẽ thực thi thay vì CRUD mặc định

Để biết ví dụ và cách tạo trình xử lý chi tiết, hãy xem [Trình xử lý tùy chỉnh](hooks-handlers/custom-handlers.md).

### Trạng thái tuyến đường
- **Đã bật/Tắt**: Kiểm soát xem tuyến đường có hoạt động hay không
- **Tích hợp hệ thống**: Các tuyến hệ thống được tích hợp tự động với giao diện quản trị

## Lợi ích điểm cuối tùy chỉnh

### Tính nhất quán của API
Tạo các điểm cuối RESTful phù hợp với quy ước đặt tên của ứng dụng của bạn:
- `/products` thay vì `/enfyra_table`
- `/orders` thay vì `/order_table`

### Hỗ trợ phiên bản
Triển khai phiên bản API:
- `/v1/người dùng`
- `/v2/người dùng`

### Tích hợp logic nghiệp vụ
Các tuyến đường có thể được tăng cường với:
- Xác thực tùy chỉnh thông qua trình xử lý
- Chuyển đổi dữ liệu qua hook
- Quy định cấp phép chuyên ngành

## Tác động đến các trang dữ liệu

Khi bạn tạo các tuyến đường tùy chỉnh, tất cả chức năng liên quan sẽ tự động điều chỉnh:
- **Trang quản lý dữ liệu** sử dụng điểm cuối tùy chỉnh
- **Bộ chọn quan hệ** tôn trọng các tuyến tùy chỉnh
- **Hệ thống cấp phép** áp dụng cho đường dẫn tùy chỉnh
- **Các lệnh gọi API** trên toàn hệ thống sử dụng các điểm cuối mới

Các thay đổi đối với tuyến đường có hiệu lực ngay lập tức mà không yêu cầu khởi động lại ứng dụng.

## Các phương pháp hay nhất

### Quy ước đặt tên
- Sử dụng đường dẫn viết thường, phân cách bằng dấu gạch nối: `/user-profiles`
- Bao gồm tiền tố phiên bản để ổn định API: `/v1/`
- Mang tính mô tả nhưng ngắn gọn: `/product-categories` không phải `/pc`

### Tổ chức tuyến đường
- Nhóm các tuyến liên quan theo tiền tố chung
- Sử dụng các mẫu nhất quán trên API của bạn
- Ghi lại mục đích của từng tuyến đường tùy chỉnh

### Căn chỉnh quyền
- Đảm bảo quyền của tuyến đường phù hợp với quyền của bảng
- Kiểm tra các tuyến tùy chỉnh với các vai trò người dùng khác nhau
- Xem xét ý nghĩa bảo mật của đường dẫn tùy chỉnh

### Tổ chức luồng thực thi
- Sử dụng **Pre-Hooks** để xác thực, kiểm tra xác thực hoặc chuẩn bị dữ liệu
- Sử dụng **Trình xử lý** cho logic nghiệp vụ chính
- Sử dụng **Post-Hooks** để ghi nhật ký, thông báo hoặc dọn dẹp
- Đặt các giá trị **ưu tiên** thích hợp để kiểm soát thứ tự thực hiện
- Sử dụng **Global Hooks** một cách tiết kiệm cho các mối quan tâm xuyên suốt áp dụng cho tất cả các tuyến đường

## Tài liệu liên quan

- **[Trình xử lý tùy chỉnh](hooks-handlers/custom-handlers.md)** - Viết logic JavaScript cho các điểm cuối tùy chỉnh
- **[Hooks System](hooks-handlers/hooks.md)** - Thêm xác thực và thông báo
- **[Quản lý gói](hooks-handlers/package-management.md)** - Cài đặt gói NPM để tích hợp bên ngoài

## Ví dụ thực tế

- **[Ví dụ đăng ký người dùng](../examples/user-registration-example.md)** - Hướng dẫn đầy đủ bao gồm tạo tuyến đường, trình xử lý tùy chỉnh và tích hợp trình gửi nút
