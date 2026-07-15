---
slug: ung-dung/quan-ly-luu-tru
---

# Quản lý lưu trữ

Quản lý lưu trữ cho phép bạn sắp xếp và quản lý các tệp và thư mục trong ứng dụng của mình. Bạn có thể tải tệp lên, tạo thư mục và định cấu hình vị trí lưu trữ.

## Truy cập Quản lý lưu trữ

1. Điều hướng đến **Quản lý lưu trữ** trong thanh bên
2. Bạn sẽ thấy giao diện quản lý file với các thư mục và tập tin

## Tuyến đường

- **Quản lý lưu trữ**: `/storage/management` - Trang quản lý tập tin chính
- **Chi tiết tệp**: `/storage/management/file/:id` - Xem và chỉnh sửa thông tin tệp
- **Chi tiết thư mục**: `/storage/management/folder/:id` - Xem và quản lý nội dung thư mục
- **Cấu hình lưu trữ**: `/storage/config` - Quản lý cấu hình lưu trữ

## Tải tập tin lên

### Từ Trang quản lý lưu trữ

1. Nhấp vào nút **"Tải tệp lên"** ở tiêu đề (trên cùng bên phải)
2. Phương thức tải lên sẽ mở
3. **Tùy chọn**: Chọn vị trí lưu trữ từ danh sách thả xuống (nếu bạn có nhiều cấu hình lưu trữ)
4. Chọn tệp bằng một trong hai cách sau:
   - Nhấp vào nút **"Chọn tệp"** để duyệt máy tính của bạn
   - Kéo thả file vào vùng upload
5. Các tệp đã chọn sẽ xuất hiện trong danh sách bên dưới khu vực tải lên
6. Nhấp vào nút **"Tải lên"** để bắt đầu tải lên
7. Các tập tin sẽ được tải lên cấp gốc (không có trong bất kỳ thư mục nào)

### Từ trang chi tiết thư mục

1. Điều hướng đến một thư mục bằng cách nhấp vào nó
2. Nhấp vào nút **"Tải tệp lên"** trong tiêu đề
3. Thực hiện các bước tương tự như trên
4. Các tập tin sẽ được tải lên thư mục hiện tại

### Tính năng tải lên

- **Nhiều tệp**: Bạn có thể tải lên nhiều tệp cùng một lúc
- **Giới hạn kích thước tệp**: Kích thước tệp tối đa là 50MB mỗi tệp
- **Lựa chọn bộ nhớ**: Bạn có thể tùy ý chọn cấu hình bộ nhớ sẽ sử dụng để tải lên
- **Các loại tệp**: Tất cả các loại tệp đều được chấp nhận theo mặc định

## Tạo thư mục

### Từ Trang quản lý lưu trữ

1. Nhấp vào nút **"Thư mục mới"** trong tiêu đề (bên cạnh Tải tệp lên)
2. Một phương thức sẽ mở ra hỏi tên thư mục
3. Nhập tên thư mục
4. Nhấp vào **"Tạo"** để tạo thư mục ở cấp độ gốc

### Từ trang chi tiết thư mục

1. Điều hướng vào một thư mục
2. Nhấp vào nút **"Thư mục mới"**
3. Nhập tên thư mục
4. Thư mục mới sẽ được tạo bên trong thư mục hiện tại

## Xem tập tin và thư mục

### Chế độ xem lưới (Mặc định)

- Các tập tin và thư mục được hiển thị dưới dạng thẻ
- Mỗi thẻ thể hiện:
  - Icon (biểu tượng thư mục cho thư mục, biểu tượng loại file cho file)
  - Tên
  - Siêu dữ liệu bổ sung (cho tệp)

### Xem danh sách

- Nhấp vào nút **"Xem danh sách"** trong tiêu đề phụ để chuyển sang chế độ xem danh sách
- Các tập tin và thư mục được hiển thị dưới dạng bảng
- Hiển thị thông tin chi tiết hơn

### Chuyển đổi chế độ xem

- Sử dụng nút chuyển đổi chế độ xem ở tiêu đề phụ (bên trái)
- Chuyển đổi giữa Chế độ xem lưới và Chế độ xem danh sách

## Quản lý tập tin

### Xem chi tiết tệp

1. Bấm vào thẻ hoặc tên file bất kỳ
2. Bạn sẽ được đưa đến trang chi tiết tệp
3. Bạn có thể thấy:
   - Xem trước tập tin (đối với hình ảnh)
   - Biểu tượng tập tin (đối với các loại tập tin khác)
   - Mẫu thông tin tập tin
   - Phần quyền tập tin

### Chỉnh sửa thông tin file

1. Điều hướng đến trang chi tiết tập tin
2. Chỉnh sửa bất kỳ trường nào trong biểu mẫu
3. Nhấp vào nút **"Lưu"** trong tiêu đề để lưu thay đổi
4. Nhấp vào **"Đặt lại"** để hủy các thay đổi

### Thay thế một tập tin

1. Điều hướng đến trang chi tiết tập tin
2. Nhấp vào nút **"Thay thế tệp"** trong tiêu đề phụ
3. Phương thức tải lên sẽ mở ra
4. Chọn file mới để thay thế file hiện tại
5. Bấm vào **"Tải lên"** để thay thế
6. **Cảnh báo**: File cũ sẽ bị mất vĩnh viễn

### Xóa tập tin

**Từ Trang chi tiết tệp:**1. Điều hướng đến chi tiết tập tin
2. Nhấp vào nút **"Xóa"** trong tiêu đề (nút màu đỏ)
3. Xác nhận xóa

**Từ Trình quản lý tệp:**
1. Nhấp chuột phải vào một tập tin
2. Chọn **"Xóa"** từ menu ngữ cảnh
3. Xác nhận xóa

**Nhiều tệp:**
1. Vào chế độ lựa chọn (nhấp vào "Chọn mục" trong tiêu đề phụ)
2. Chọn nhiều tệp bằng cách nhấp vào chúng
3. Sử dụng tác vụ xóa từ menu ngữ cảnh hoặc thanh công cụ

## Quản lý thư mục

### Xem nội dung thư mục

1. Nhấp vào thư mục bất kỳ
2. Bạn sẽ được đưa đến trang chi tiết thư mục
3. Bạn có thể thấy:
   - Tất cả các thư mục con
   - Tất cả các tập tin trong thư mục
   - Thông tin thư mục

### Chỉnh sửa thông tin thư mục

1. Điều hướng đến trang chi tiết thư mục
2. Chỉnh sửa các trường thư mục trong biểu mẫu
3. Nhấp vào **"Lưu"** để lưu thay đổi

### Xóa thư mục

**Từ Chi tiết Thư mục:**
1. Nhấp chuột phải vào thư mục
2. Chọn **"Xóa"** từ menu ngữ cảnh
3. Xác nhận xóa

**Cảnh báo**: Xóa một thư mục cũng sẽ xóa tất cả các tệp và thư mục con bên trong nó.

## Cấu hình lưu trữ

### Xem cấu hình lưu trữ

1. Điều hướng đến **Cấu hình lưu trữ** trong thanh bên
2. Bạn sẽ thấy danh sách tất cả các cấu hình lưu trữ
3. Mỗi cấu hình hiển thị:
   - Tên
   - Loại lưu trữ (Amazon S3, Google Cloud Storage, Cloudflare R2, Local Storage)
   - Trạng thái (Hoạt động/Không hoạt động)
   - Mô tả
   - Màu huy hiệu biểu thị loại kho lưu trữ:
     - **Xanh lam (Chính)**: Amazon S3
     - **Cyan (Thông tin)**: Google Cloud Storage
     - **Cam (Cảnh báo)**: Cloudflare R2
     - **Xám (Trung tính)**: Bộ nhớ cục bộ

### Tạo cấu hình lưu trữ

1. Vào trang Cấu hình lưu trữ
2. Nhấp vào nút **"Tạo bộ nhớ"** trong tiêu đề
3. Ban đầu, biểu mẫu sẽ chỉ hiển thị các trường sau:
   - **Tên**: Tên mô tả cho cấu hình lưu trữ này
   - **Nhóm**: Tên nhóm/vùng chứa lưu trữ
   - **Mô tả**: Mô tả tùy chọn
   - **Loại**: Chọn loại bộ nhớ (Amazon S3, Google Cloud Storage hoặc Cloudflare R2)
     - **Lưu ý**: Không thể tạo Bộ nhớ cục bộ thông qua giao diện người dùng

4. Sau khi chọn loại lưu trữ, các trường bổ sung sẽ xuất hiện dựa trên loại:

   **Đối với Amazon S3:**
   - **ID khóa truy cập**: ID khóa truy cập AWS của bạn
   - **Khóa truy cập bí mật**: Khóa truy cập bí mật AWS của bạn
   - **Nhóm**: Tên nhóm S3
   - **Khu vực**: Khu vực AWS (ví dụ: us-east-1, eu-west-1)

   **Đối với Google Cloud Storage:**
   - **Thông tin xác thực**: Thông tin xác thực JSON cho tài khoản dịch vụ GCS
   - **Nhóm**: Tên nhóm GCS

   **Đối với Cloudflare R2:**
   - **ID khóa truy cập**: ID khóa truy cập R2 của bạn
   - **Khóa truy cập bí mật**: Khóa truy cập bí mật R2 của bạn
   - **ID tài khoản**: ID tài khoản Cloudflare của bạn
   - **Nhóm**: Tên nhóm R2

5. Điền vào tất cả các trường bắt buộc cho loại bộ nhớ bạn đã chọn
6. Nhấp vào **"Lưu"** để tạo cấu hình

### Chỉnh sửa cấu hình lưu trữ

1. Nhấp vào bất kỳ thẻ cấu hình lưu trữ nào
2. Biểu mẫu sẽ hiển thị tất cả các trường có liên quan dựa trên loại lưu trữ
3. **Lưu ý**: Trường **Type** bị vô hiệu hóa và không thể thay đổi sau khi tạo
4. Chỉnh sửa các trường bạn muốn sửa đổi
5. Nhấp vào **"Lưu"** để lưu thay đổi
6. Nhấp vào **"Đặt lại"** để hủy các thay đổi

### Bật/Tắt bộ nhớ

1. Trên trang danh sách Cấu hình lưu trữ
2. Chuyển đổi công tắc trên bất kỳ thẻ cấu hình nào
3. Cấu hình hoạt động có thể được sử dụng để tải tệp lên
4. Không thể chọn cấu hình không hoạt động trong quá trình tải lên

### Xóa cấu hình lưu trữ

1. Trên trang danh sách Cấu hình lưu trữ
2. Nhấp vào nút **"Xóa"** trên thẻ cấu hình
3. Xác nhận xóa

## Tác vụ tệp### Thao tác trên menu ngữ cảnh

Nhấp chuột phải vào bất kỳ tệp nào để xem các hành động có sẵn:
- **Xem**: Mở file trong tab mới
- **Tải xuống**: Tải file xuống
- **Sao chép URL**: Sao chép URL tệp vào clipboard
- **Chi tiết**: Điều hướng đến trang chi tiết tập tin
- **Delete**: Xóa file (nếu có quyền)

### Chế độ lựa chọn

1. Nhấp vào nút **"Chọn mục"** trong tiêu đề phụ
2. Bấm vào tập tin/thư mục để chọn chúng
3. Các mục được chọn sẽ được đánh dấu
4. Sử dụng các thao tác hàng loạt (xóa, di chuyển, v.v.)
5. Nhấp vào **"Hủy lựa chọn"** để thoát khỏi chế độ lựa chọn

## Điều hướng

### Mẩu bánh mì

- Sử dụng đường dẫn ở trên cùng để điều hướng trở lại thư mục mẹ
- Bấm vào tên thư mục bất kỳ trong đường dẫn để đến thư mục đó

### Điều hướng quay lại

- Sử dụng nút quay lại trình duyệt
- Hoặc bấm vào thư mục mẹ trong đường dẫn

## Phân trang

Khi bạn có nhiều tệp hoặc thư mục:
- Điều khiển phân trang xuất hiện ở phía dưới
- Phân trang riêng biệt cho các thư mục và tập tin
- Sử dụng số trang để điều hướng qua các trang

## Quyền

- **Tải tệp lên**: Yêu cầu quyền tạo trên `/enfyra_file`
- **Tạo thư mục**: Yêu cầu quyền tạo trên `/enfyra_folder`
- **Chỉnh sửa tệp**: Yêu cầu quyền cập nhật trên `/enfyra_file`
- **Xóa tệp**: Yêu cầu quyền xóa trên `/enfyra_file`
- **Quản lý cấu hình lưu trữ**: Yêu cầu quyền trên `/enfyra_storage_config`

Các hành động sẽ chỉ xuất hiện nếu bạn có các quyền cần thiết.
