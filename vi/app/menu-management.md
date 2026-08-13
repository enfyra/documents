---
slug: ung-dung/quan-ly-menu
---

# Quản lý menu

Quản lý menu cung cấp giao diện trực quan để tạo và sắp xếp menu điều hướng trong ứng dụng. Bạn có thể xây dựng cấu trúc menu phân cấp, kéo thả để thay đổi thứ tự, cấu hình quyền và quản lý các mục menu ngay trong trình chỉnh sửa trực quan.

## Truy cập trang Quản lý menu

1. Trên thanh bên, đi đến **Cài đặt > Menu**.
2. Trang hiển thị toàn bộ menu dưới dạng cây trực quan.
3. Các menu hệ thống như Trang tổng quan và Cài đặt được bảo vệ, không thể xóa.

## Các loại menu

### Menu

- **Mục đích**: Là mục menu riêng lẻ dùng để mở một trang cụ thể hoặc làm mục cha chứa các mục con.
- **Cách hiển thị**: Là mục có thể nhấn trên thanh bên.
- **Có thể là**:
  - Menu đơn có tuyến trực tiếp để điều hướng đến một trang.
  - Nhóm menu có các mục con và có thể mở rộng.
- **Có thể chứa**: Các mục menu con (không bắt buộc).

### Menu thả xuống

- **Mục đích**: Nhóm các mục menu liên quan trong một phần có thể thu gọn.
- **Cách hiển thị**: Là một phần có thể mở rộng, kèm biểu tượng mũi tên.
- **Có thể chứa**: Các mục menu riêng lẻ hoặc menu thả xuống khác.
- **Cách hoạt động**: Nhấn để mở rộng hoặc thu gọn và hiển thị các mục con.

## Trình chỉnh sửa menu trực quan

Trình chỉnh sửa menu trực quan cung cấp giao diện kéo thả thuận tiện để quản lý menu:

- **Dạng cây**: Hiển thị cấu trúc menu theo thứ bậc.
- **Kéo thả**: Kéo các menu trong cây để thay đổi thứ tự.
- **Di chuyển giữa các cấp**: Kéo menu sang mục cha khác hoặc đưa về cấp gốc.
- **Phân cấp trực quan**: Thể hiện rõ quan hệ cha-con.
- **Thao tác nhanh**: Sửa, xóa, bật hoặc tắt ngay trong cây.

## Tạo menu mới

### Bước 1: Bắt đầu tạo

1. Nhấn nút **"Tạo menu"** trong khu vực thao tác ở đầu trang.
2. Biểu mẫu tạo menu sẽ mở trong một hộp thoại.

### Bước 2: Nhập thông tin cơ bản

- **Nhãn**: Tên hiển thị của mục menu.
- **Mô tả**: Nội dung mô tả chức năng của menu, không bắt buộc.
- **Biểu tượng**: Chọn một biểu tượng Lucide (mặc định là `menu`).
- **Đường dẫn**: Tuyến URL mà menu sẽ mở, ví dụ `/my-custom-page`.
  - **Quan trọng**: Đường dẫn phải bắt đầu bằng `/`.
  - Để trống nếu menu chỉ dùng làm mục chứa các mục con.

### Bước 3: Chọn loại menu

Chọn loại trong danh sách thả xuống:

- **Menu**: Tạo một mục menu; mục này có thể có tuyến hoặc chứa các mục con.
- **Menu thả xuống**: Tạo một phần có thể thu gọn để nhóm các mục liên quan.
  - Menu thả xuống thường không có tuyến vì chỉ đóng vai trò mục chứa.

### Bước 4: Thiết lập phân cấp (không bắt buộc)

**Đối với Menu:**

- **Mục cha**: Dùng bộ chọn quan hệ để chọn một menu cha, có thể là Menu thả xuống hoặc Menu có mục con.
- Nếu không chọn mục cha, menu sẽ xuất hiện ở cấp cao nhất.
- **Đường dẫn**: Nhập đường dẫn tuyến nếu menu cần điều hướng đến một trang.

**Đối với Menu thả xuống:**

- **Mục cha**: Dùng bộ chọn quan hệ để chọn một menu cha, nếu cần.
- Nếu không chọn mục cha, menu thả xuống sẽ xuất hiện ở cấp cao nhất.
- Menu thả xuống thường không có tuyến vì dùng để chứa các mục khác.

**Lưu ý**: Bạn có thể lồng menu bên dưới Menu hoặc Menu thả xuống để tạo cấu trúc phân cấp. Sau khi tạo, hãy dùng trình chỉnh sửa trực quan để sắp xếp lại.

### Bước 5: Cấu hình

- **Thứ tự**: Số xác định thứ tự hiển thị; số nhỏ hơn xuất hiện trước.
- **Đã bật**: Bật hoặc tắt mục menu.
- **Quyền**: Nhấn **"Cấu hình quyền"** để thiết lập quy tắc truy cập; xem phần bên dưới.

### Bước 6: Lưu

Nhấn **"Tạo"** để lưu mục menu mới.

## Quản lý menu bằng trình chỉnh sửa trực quan

### Sắp xếp lại bằng thao tác kéo thả

Trình chỉnh sửa trực quan giúp bạn dễ dàng tổ chức lại menu:

1. **Sắp xếp các mục cùng cấp**: Kéo mục menu lên hoặc xuống trong cấp hiện tại.
2. **Chuyển sang mục cha khác**: Kéo mục menu vào một menu cha khác.
3. **Đưa về cấp gốc**: Dùng tùy chọn **"Đưa về cấp gốc"** cho menu đang có mục cha.
4. **Phản hồi trực quan**: Khi kéo, giao diện cho biết vị trí menu sẽ được đặt vào.

### Thao tác nhanh trong dạng cây

Mỗi mục menu trong cây có các thao tác nhanh sau:

- **Sửa**: Nhấn biểu tượng sửa để thay đổi mục menu.
- **Xóa**: Nhấn biểu tượng xóa; thao tác này không áp dụng cho menu hệ thống.
- **Bật/Tắt**: Nhấn công tắc để bật hoặc tắt mục menu.
- **Thêm mục con**: Tạo mục menu mới và tự động đặt menu hiện tại làm mục cha.
- **Thêm tiện ích mở rộng**: Liên kết một tiện ích mở rộng với menu nếu phù hợp.

### Sửa menu

1. **Nhấn vào một mục menu** trong cây trực quan để mở hộp thoại chỉnh sửa.
2. **Thay đổi các trường cần thiết** như nhãn, biểu tượng, đường dẫn hoặc quyền.
3. **Nhấn "Lưu"** để áp dụng thay đổi.

**Lưu ý**: Các thay đổi về thuộc tính menu, ngoại trừ thứ tự và mục cha, được lưu ngay. Để đổi thứ tự, hãy dùng thao tác kéo thả.

### Xóa menu

1. **Nhấn biểu tượng xóa** trên mục menu cần xóa; menu hệ thống không có thao tác này.
2. **Xác nhận xóa** trong hộp thoại xác nhận.
3. **Các menu con** sẽ tự động được chuyển lên mục cha của menu vừa xóa hoặc về cấp gốc.

**Cảnh báo**: Việc xóa menu cũng ảnh hưởng đến các menu con phụ thuộc vào menu đó với vai trò mục cha. Hệ thống sẽ tự động sắp xếp lại cấu trúc phân cấp.

## Hiển thị menu

Hiển thị menu chỉ là contract điều hướng, tách biệt với quyền route và không cấp quyền gọi API.

### Menu public

Menu mới mặc định là private (`isPublic: false`), ngoại trừ Dashboard tích hợp (`/dashboard`) được public ngay từ lần cài đầu. Chỉ bật **Public** (`isPublic`) cho menu khác khi mọi role đều phải thấy. Đây là trạng thái tường minh; không được tự hiểu “không có role row” là public đối với menu private. Ở lần cài mới, role không phải root sẽ chưa thấy các menu khác cho đến khi được thêm role rule hoặc menu được bật public.

### Menu giới hạn theo role

1. Tắt **Public** (`isPublic: false`) trong trình chỉnh sửa menu.
2. Trong phần **Role visibility**, nhấn **Add role**.
3. Chọn từng role được phép thấy menu rồi lưu rule.
4. Tắt hoặc xóa role rule khi role đó không còn được thấy menu.

Menu private không có role rule đang bật sẽ không hiển thị với role thông thường nào. Root admin vẫn thấy menu đang bật.

### Menu cha

Menu cha vẫn hiển thị nếu còn ít nhất một menu con mà role được phép thấy, để người dùng giữ được đường dẫn điều hướng tới menu con đó.

### Quyền API và action

`PermissionGate` bên trong page tiếp tục kiểm soát button, form, tab và action bằng điều kiện route/method. Quyền route phía backend vẫn là authority, nên menu có thể hiện nhưng API vẫn trả `403` nếu role chưa được cấp quyền. Cấu hình các gate này độc lập; không đặt JSON route/method vào cấu hình hiển thị menu.

Field JSON `enfyra_menu.permission` cũ không còn được dùng để quyết định menu hiển thị. Dùng `isPublic` và các role visibility rule thay thế.

## Ví dụ về cấu trúc menu

### Menu cấp cao nhất đơn giản

1. Tạo **Menu** có nhãn "Báo cáo" và đường dẫn `/reports`.
2. Menu xuất hiện dưới dạng một mục có thể nhấn để mở trang báo cáo.

### Menu có mục con

1. Tạo **Menu** có nhãn "Quản lý", không nhập đường dẫn hoặc đặt một đường dẫn mặc định.
2. Tạo các mục **Menu** và đặt "Quản lý" làm mục cha:
   - "Danh sách người dùng" (`/management/users`).
   - "Vai trò người dùng" (`/management/roles`).
3. Menu Quản lý trở thành mục có thể mở rộng và hiển thị các mục con khi được nhấn.

### Cấu trúc menu thả xuống

1. Tạo **Menu thả xuống** có nhãn "Quản lý người dùng", không cần nhập đường dẫn.
2. Tạo các mục **Menu** và đặt "Quản lý người dùng" làm mục cha:
   - "Danh sách người dùng" (`/management/users`).
   - "Vai trò người dùng" (`/management/roles`).
3. Menu thả xuống xuất hiện dưới dạng một phần có thể thu gọn, kèm biểu tượng mũi tên.

### Cấu trúc lồng nhau phức tạp

1. Tạo **Menu** có nhãn "Cài đặt" ở cấp cao nhất.
2. Tạo **Menu thả xuống** có nhãn "Quản lý người dùng" và đặt "Cài đặt" làm mục cha.
3. Tạo các mục **Menu** và đặt "Quản lý người dùng" làm mục cha:
   - "Danh sách người dùng" (`/settings/users`).
   - "Vai trò người dùng" (`/settings/roles`).
4. Kết quả là cấu trúc: Cài đặt > Quản lý người dùng > (Danh sách người dùng, Vai trò người dùng).

**Mẹo**: Hãy tạo cấu trúc ban đầu trong trình chỉnh sửa trực quan, sau đó kéo thả để sắp xếp lại khi cần.

## Tích hợp tiện ích mở rộng

Menu có thể liên kết với **Tiện ích mở rộng** để cung cấp nội dung hoặc chức năng động:

1. **Tạo hoặc chọn một menu** trong trình chỉnh sửa trực quan.
2. **Nhấn "Thêm tiện ích mở rộng"** hoặc dùng tùy chọn tiện ích mở rộng.
3. **Chọn một tiện ích mở rộng** trong bộ chọn quan hệ.
4. Khi được nhấn, menu sẽ hiển thị nội dung của tiện ích mở rộng.

**Lưu ý**: Tiện ích mở rộng cung cấp nội dung trang hoặc chức năng tùy chỉnh. Xem [Hệ thống mở rộng](./extension-system.md) để biết chi tiết.

## Lưu ý quan trọng

### Menu hệ thống

- **Được bảo vệ**: Không thể xóa các menu hệ thống như Trang tổng quan, Dữ liệu, Bộ sưu tập, Cài đặt và Lưu trữ.
- **Giới hạn chỉnh sửa**: Bạn có thể thay đổi nhãn và biểu tượng, nhưng chức năng cốt lõi được bảo vệ.
- **Cách nhận biết**: Menu hệ thống có nhãn **"Hệ thống"**.

### Yêu cầu đối với đường dẫn

- **Phải bắt đầu bằng /**: Mọi đường dẫn đều phải bắt đầu bằng dấu gạch chéo `/`.
- **Đường dẫn duy nhất**: Mỗi menu nên có đường dẫn riêng để tránh xung đột, nếu có khai báo đường dẫn.
- **Khớp tuyến**: Đảm bảo đường dẫn tương ứng với một tuyến thực tế trong ứng dụng.
- **Đường dẫn trống**: Được phép với menu chỉ dùng làm mục chứa, không điều hướng trực tiếp.

### Thứ tự và cách tổ chức

- **Sắp xếp bằng số**: Số nhỏ hơn xuất hiện trước, chẳng hạn 0, 1, 2.
- **Phân cấp trực quan**: Menu có thể chứa Menu thả xuống hoặc các mục Menu khác.
- **Nhóm hợp lý**: Gom các chức năng liên quan vào cùng một menu cha hoặc menu thả xuống.
- **Vị trí**: Dùng trường vị trí để đặt menu ở phần `top` hoặc `bottom` của thanh bên.
- **Kéo thả**: Trình chỉnh sửa trực quan cho phép đổi thứ tự mà không cần sửa từng menu.

### Lợi ích của trình chỉnh sửa trực quan

- **Tổ chức trực quan**: Quan sát toàn bộ cấu trúc menu trong một màn hình.
- **Dễ sắp xếp lại**: Kéo menu để tổ chức lại mà không cần sửa từng mục.
- **Thao tác nhanh**: Sửa, xóa, bật hoặc tắt ngay trong cây.
- **Hiển thị phân cấp**: Thể hiện rõ quan hệ cha-con.

## Khắc phục sự cố

### Menu không xuất hiện

- Kiểm tra **Đã bật** có đang được bật hay không.
- Xác minh người dùng có đủ quyền cần thiết.
- Đảm bảo loại menu và cấu trúc phân cấp được thiết lập đúng.
- Kiểm tra menu cha cũng đang được bật và hiển thị.

### Vấn đề về quyền

- Xác nhận các quy tắc quyền đã được cấu hình đúng.
- Kiểm tra vai trò của người dùng có các quyền tuyến cần thiết.
- Thử dùng **"Cho phép tất cả"** để xác định lỗi nằm ở quyền hay thành phần khác.
- Lưu ý rằng menu cha sẽ bị ẩn nếu không có mục con nào mà người dùng được phép truy cập.

### Vấn đề về phân cấp

- **Menu**: Có thể nằm dưới một Menu hoặc Menu thả xuống, hoặc ở cấp cao nhất.
- **Menu thả xuống**: Có thể nằm dưới một Menu hoặc Menu thả xuống, hoặc ở cấp cao nhất.
- **Tham chiếu vòng**: Hệ thống ngăn menu chọn chính nó làm mục cha.
- **Xung đột đường dẫn**: Mỗi menu có tuyến nên dùng một đường dẫn duy nhất.
- **Menu mất mục cha**: Menu có mục cha đã bị xóa sẽ tự động được đưa về cấp gốc.

### Vấn đề khi kéo thả

- Đảm bảo bạn đang kéo menu vào vùng thả hợp lệ, chẳng hạn một menu khác hoặc vùng cấp gốc.
- Phản hồi trực quan cho biết vị trí menu sẽ được đặt vào.
- Nếu menu được di chuyển sai vị trí, hãy dùng **"Đưa về cấp gốc"** rồi sắp xếp lại.
- Kiểm tra mục cha đích có thể chứa mục con hay không; một số loại menu có thể bị giới hạn.

## Thực hành tốt

### Tổ chức menu

- **Nhóm các mục liên quan**: Dùng Menu thả xuống để gom các chức năng liên quan.
- **Phân cấp hợp lý**: Xây dựng cấu trúc điều hướng rõ ràng, dễ hiểu.
- **Đặt tên nhất quán**: Dùng nhãn rõ nghĩa và có tính mô tả.
- **Giới hạn độ sâu**: Tránh lồng quá sâu; tối đa 3-4 cấp giúp trải nghiệm sử dụng tốt hơn.

### Quyền

- **Đặc quyền tối thiểu**: Chỉ cấp quyền cho những người dùng thực sự cần.
- **Theo vai trò**: Dùng vai trò cho các mẫu quyền phổ biến.
- **Kiểm thử kỹ**: Kiểm tra khả năng hiển thị menu với nhiều vai trò người dùng.
- **Mô tả quy tắc phức tạp**: Thêm nội dung mô tả vào cấu hình quyền.

### Sử dụng trình chỉnh sửa trực quan

- **Lên cấu trúc trước**: Phác thảo cấu trúc menu trước khi tạo.
- **Dùng kéo thả**: Sắp xếp menu trực quan thay vì sửa từng số thứ tự.
- **Rà soát định kỳ**: Thường xuyên xem lại và tối ưu cấu trúc menu.
- **Sao lưu trước thay đổi lớn**: Cân nhắc ghi lại cấu trúc trước khi tổ chức lại trên diện rộng.
