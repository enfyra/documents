---
slug: may-chu-mcp
---

# Sử dụng Enfyra với trợ lý lập trình AI

Enfyra MCP kết nối dự án Enfyra với các trợ lý lập trình hỗ trợ MCP như Codex, Claude Code, Cursor, VS Code với GitHub Copilot và Google Antigravity. Sau khi kết nối, bạn có thể yêu cầu trợ lý tìm hiểu hoặc thay đổi dự án bằng ngôn ngữ tự nhiên, không cần tự sao chép schema, response API hay cấu hình qua lại giữa các công cụ.

## Bạn có thể làm gì?

Enfyra MCP có thể hỗ trợ bạn:

- Tìm hiểu bảng, quan hệ, route, phân quyền, flow, extension và cấu hình runtime.
- Tạo hoặc cập nhật mô hình dữ liệu mà không làm ảnh hưởng cấu trúc hiện có.
- Xây dựng và kiểm thử API tùy chỉnh, handler, hook và tác vụ tự động.
- Tạo trang quản trị hoặc widget phù hợp với theme hiện tại của Enfyra.
- Điều tra lỗi phân quyền, request thất bại, log và hành vi runtime.
- Kiểm tra thay đổi trực tiếp trên Enfyra trước khi hoàn tất công việc.

Trợ lý đọc trạng thái thực tế của dự án đang kết nối, vì vậy kết quả không phụ thuộc vào một bản mô tả schema đã cũ.

## Chuẩn bị

Bạn cần:

1. Một Enfyra instance có thể mở trên trình duyệt.
2. Programmatic API token tại **Tài khoản → API token** trong Enfyra Admin.
3. Một công cụ lập trình hỗ trợ MCP.
4. Thư mục project nơi trợ lý sẽ làm việc.

Chỉ cấp cho token những quyền thực sự cần thiết. Nên dùng token riêng cho môi trường development và production.

## Kết nối project

Mở terminal tại thư mục project rồi chạy:

```bash
npx @enfyra/mcp-server config
```

Trình cấu hình sẽ hỏi URL Enfyra và API token, sau đó tạo cấu hình cục bộ phù hợp với công cụ lập trình đang dùng.

Nếu cần cấu hình tự động:

```bash
npx @enfyra/mcp-server config --yes \
  --app-url https://demo.enfyra.io \
  --api-token efy_pat_your-token
```

Hãy dùng URL bạn thường mở để truy cập Enfyra Admin. Sau khi cấu hình, khởi động lại hoặc reload công cụ lập trình để công cụ nhận MCP server mới.

## Kiểm tra kết nối

Bắt đầu bằng một yêu cầu chỉ đọc:

> Kiểm tra Enfyra instance đang kết nối, sau đó liệt kê các bảng chính trong project. Không thay đổi dữ liệu hay cấu hình.

Hãy chắc chắn URL và các bảng được trả về thuộc đúng môi trường trước khi yêu cầu trợ lý thực hiện thay đổi.

## Các tình huống sử dụng phổ biến

### Tìm hiểu một project có sẵn

Nên yêu cầu tổng quan trước khi thay đổi một project chưa quen thuộc:

> Rà soát project Enfyra này và giải thích các bảng, quan hệ, public API, phân quyền và tác vụ tự động chính. Chỉ ra những điểm cần lưu ý nhưng chưa thay đổi gì.

### Tạo mô hình dữ liệu

Mô tả nghiệp vụ và ràng buộc quan trọng thay vì tự chỉ định metadata cấp thấp:

> Thêm `projects` và `project_members` để một project có thể có nhiều thành viên. Không cho phép trùng thành viên, giữ nguyên dữ liệu hiện có và kiểm tra lại các quan hệ sau khi áp dụng.

Trợ lý có thể đọc schema hiện tại, chọn kiểu dữ liệu tương thích, thực hiện thay đổi và xác minh kết quả.

### Xây dựng API

Nêu rõ ai được gọi API và họ được phép truy cập những bản ghi nào:

> Tạo endpoint trả về các project đang hoạt động của người dùng đã đăng nhập. Người dùng chỉ được thấy project mà họ là thành viên. Kiểm thử cả trường hợp hợp lệ và trường hợp không có quyền.

Phân quyền là một phần của tính năng. Với dữ liệu thuộc người dùng, nhóm hoặc tenant, chỉ mở quyền truy cập route là chưa đủ.

### Tạo tác vụ tự động

Mô tả trigger, hành động và cách xử lý khi có lỗi:

> Khi đơn hàng chuyển sang trạng thái đã thanh toán, tạo bản ghi hóa đơn và thông báo cho nhóm vận hành. Flow phải an toàn khi chạy lại. Cho tôi xem cấu trúc cuối cùng trước khi bật flow.

### Thêm trang quản trị hoặc widget

Tập trung vào công việc mà giao diện cần hỗ trợ:

> Thêm trang quản trị để duyệt các đơn hàng đang chờ xử lý. Có bộ lọc theo ngày và trạng thái, dùng theme hiện tại của Enfyra và chỉ hiển thị những thao tác người dùng có quyền thực hiện.

### Điều tra lỗi

Cung cấp URL bị lỗi, thông báo nhìn thấy và hành vi mong đợi:

> Request đến `/api/projects` trả 403 với role editor. Kiểm tra route, quyền của role và điều kiện truy cập bản ghi, sau đó giải thích nguyên nhân. Chưa thay đổi quyền cho đến khi xác định được nguyên nhân.

## Cách viết yêu cầu hiệu quả

Một yêu cầu rõ ràng nên nêu:

- Kết quả cần đạt được.
- Người dùng hoặc role liên quan.
- Dữ liệu và hành vi hiện có cần được giữ nguyên.
- Ranh giới phân quyền hoặc tenant quan trọng.
- Cách kiểm tra kết quả.
- Trợ lý được phép áp dụng thay đổi hay chỉ cần chuẩn bị phương án.

Với công việc lớn, hãy yêu cầu trợ lý đọc project trước. Chỉ nên áp dụng thay đổi sau khi trợ lý xác định được các bảng, route và quyền sẽ bị ảnh hưởng.

## Làm việc an toàn

- Xác nhận đúng môi trường trước mỗi thay đổi trên production.
- Dùng cấu hình cục bộ riêng cho từng project hoặc môi trường.
- Yêu cầu xem trước khi xóa bảng, field, route hoặc dữ liệu.
- Sao lưu trước khi thay đổi schema có tính phá vỡ hoặc chạy data migration lớn.
- Chỉ cấp cho production token những quyền cần thiết.
- Thu hồi token ngay nếu token xuất hiện trong log, source control hoặc nội dung trao đổi.
- Luôn kiểm thử bằng cả người dùng được phép và người dùng phải bị từ chối.

## Chuyển môi trường

Cấu hình MCP được lưu trong project hiện tại. Muốn chuyển sang Enfyra instance khác, hãy chạy lại lệnh cấu hình với URL và token mới, sau đó khởi động lại công cụ lập trình.

Không nên dùng chung một production token cho nhiều project cục bộ không liên quan.

## Xử lý sự cố

### Không thấy công cụ Enfyra

Khởi động lại công cụ lập trình sau khi chạy lệnh cấu hình. Kiểm tra file cấu hình MCP đã được tạo trong đúng project và Node.js có thể chạy `npx`.

### Xác thực thất bại

Tạo programmatic API token mới trong Enfyra Admin rồi chạy lại lệnh cấu hình. URL Enfyra phải thuộc đúng instance đã cấp token.

### Không có quyền đọc hoặc thay đổi

Kết nối đã hoạt động nhưng người dùng hoặc role của token chưa có quyền với route hay bảng cần thiết. Chỉ bổ sung quyền còn thiếu rồi thử lại yêu cầu ban đầu.

### Trợ lý vẫn thấy schema cũ

Yêu cầu trợ lý tải lại metadata của bảng hoặc route liên quan. Nếu vừa thay đổi cấu hình MCP, hãy khởi động lại công cụ lập trình trước khi thử lại.

### Công việc lớn khó kiểm soát

Chia yêu cầu theo kết quả: schema, hành vi API, flow, giao diện và cuối cùng là kiểm tra end-to-end. Mỗi phần nên có thể kiểm thử độc lập.

## Cấu hình nâng cao

Lệnh cấu hình sẽ tự quản lý các giá trị sau:

| Thiết lập | Mục đích |
|---|---|
| `ENFYRA_APP_URL` | URL Enfyra bạn mở trên trình duyệt. |
| `ENFYRA_API_URL` | API base được suy ra từ URL của ứng dụng. |
| `ENFYRA_API_TOKEN` | Programmatic token dùng để xác thực kết nối. |
| `ENFYRA_MCP_TOOLSET` | Chế độ hiển thị công cụ; nên giữ chế độ guided mặc định. |

Chỉ dùng full toolset khi cần debug nâng cao hoặc xử lý vấn đề tương thích. Chế độ này hiển thị thêm các thao tác cấp thấp thường không cần thiết trong quá trình phát triển thông thường.
