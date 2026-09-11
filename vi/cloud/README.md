---
slug: cloud
---

# Enfyra Cloud

Enfyra Cloud là control plane quản lý các Enfyra project chạy trong tài khoản hạ tầng do chính bạn sở hữu. Enfyra kết nối với provider được hỗ trợ qua OAuth, tạo và quản lý tài nguyên đã chọn cho từng project, đồng thời cung cấp một console chung cho provisioning, deployment, update, credential, domain, backup và thao tác runtime.

Enfyra không bán lại compute. Thuê bao quản lý có giá `$11.99` cho mỗi project mỗi tháng. Provider hạ tầng thu trực tiếp trên tài khoản provider của bạn cho compute, memory, storage, network traffic, backup và các feature của provider.

Railway là provider đầu tiên được hỗ trợ. Mô hình Cloud project không khóa vào provider, nên có thể bổ sung các provider hỗ trợ OAuth khác mà không thay đổi contract sở hữu.

## Khi nào dùng Cloud

Chọn Enfyra Cloud khi bạn muốn:

- Vận hành Enfyra mà không phải quản lý VPS hoặc tự xây deployment control plane.
- Giữ hạ tầng, mức sử dụng, dữ liệu và billing của provider trong tài khoản của chính mình.
- Để Enfyra quản lý provisioning, deployment, cập nhật runtime đã pin, restart, environment variable, domain, credential và backup.
- Chọn provider được hỗ trợ cho từng project thay vì khóa mọi project vào một nhà cung cấp hạ tầng.

Chọn self-hosting khi bạn cần toàn quyền kiểm soát host, network, deployment pipeline và mọi backing service mà không cấp quyền quản lý cho Enfyra.

## Quyền sở hữu project và provider

Enfyra Cloud project là đơn vị quản lý duy nhất. Mỗi project chọn một provider được hỗ trợ và ánh xạ tới một provider project. Bạn cấp quyền cho Enfyra qua OAuth của provider; bạn không phải dán personal access token vào Enfyra Cloud.

Tài nguyên provider vẫn nằm trong tài khoản của bạn. Disconnect OAuth, dừng gia hạn subscription, mất management entitlement hoặc xóa Cloud management record không tự dừng hay xóa provider service và cũng không dừng billing của provider. Thao tác phá hủy tài nguyên provider là luồng riêng và luôn cần xác nhận rõ ràng.

## Tạo project

1. Mở `https://cloud.enfyra.io` và đăng nhập.
2. Tạo Enfyra Cloud project record.
3. Chọn provider hạ tầng được hỗ trợ.
4. Kết nối provider qua OAuth nếu chưa kết nối.
5. Cấu hình provider project, region, subdomain, administrator email và runtime option.
6. Xem lại trách nhiệm billing tách biệt giữa Enfyra và provider.
7. Kích hoạt thuê bao quản lý Enfyra `$11.99` qua PayPal.
8. Bắt đầu provisioning.

Thanh toán là gate cuối cùng của quá trình setup. Enfyra chỉ provision tài nguyên provider sau khi cấu hình project hoàn tất và management subscription đã active.

## Topology Railway hiện tại

Với Railway, Enfyra tạo:

- Một Railway project cho một Enfyra Cloud project.
- Một Railway PostgreSQL service luôn bật với persistent database volume riêng.
- Một Enfyra service kết nối PostgreSQL qua private network của Railway.
- Redis embedded trong Enfyra service, với runtime data được giữ bền vững tại `/app/data`.
- Một Railway service domain và một custom domain do Enfyra quản lý.

Production database không nằm embedded trong Enfyra container. Restart hoặc update runtime không tháo PostgreSQL volume.

## File storage

File upload bắt buộc dùng external object storage. Sau khi provisioning, hãy cấu hình Amazon S3, Cloudflare R2, Google Cloud Storage hoặc backend tương thích S3 được hỗ trợ. Railway runtime volume và database volume không phải nơi lưu file upload lâu dài.

## Railway Serverless

Railway Serverless là tùy chọn cho Enfyra runtime service. PostgreSQL luôn bật.

Bật hoặc tắt Serverless đều cần một Railway deployment mới thì cấu hình mới có hiệu lực. Enfyra cảnh báo trước khi áp dụng và bắt đầu redeploy sau khi bạn xác nhận. Runtime đang sleep có thể làm request đầu tiên chậm hơn hoặc tạm nhận provider error trong lúc container thức dậy. Outbound traffic, kết nối WebSocket hoặc database còn mở và background work cũng có thể làm service không sleep.

## Billing và hủy gia hạn

Enfyra subscription và bill của provider độc lập với nhau:

- PayPal xử lý `$11.99` mỗi tháng cho mỗi managed project để sử dụng dịch vụ quản lý của Enfyra.
- Provider đã kết nối thu trực tiếp chi phí sử dụng hạ tầng.
- Enfyra không cộng phụ phí vào mức sử dụng provider và không bao gồm compute trong giá quản lý.

Cloud cung cấp **Dừng gia hạn**, không cung cấp hoàn tiền tự nguyện. Dừng gia hạn ngăn khoản thu ở kỳ tiếp theo và giữ quyền quản lý tới hết kỳ đã thanh toán. Kỳ subscription đã thanh toán không được hoàn hoặc chia theo thời gian sử dụng, trừ khi pháp luật áp dụng bắt buộc phải điều chỉnh.

Khi management access kết thúc, các thao tác managed của Enfyra bị vô hiệu hóa. Tài nguyên provider tiếp tục nằm trong tài khoản provider của bạn cho tới khi bạn thay đổi hoặc xóa chúng ở provider, hoặc dùng một thao tác tài nguyên riêng đã được xác nhận rõ ràng.

## Backup và drift

Khả năng backup phụ thuộc vào provider và account tier đã chọn. Trên Railway, built-in volume backup yêu cầu workspace Pro hoặc Enterprise. Workspace khác phải dùng external PostgreSQL backup target, chẳng hạn chạy `pg_dump` theo lịch rồi upload vào S3-compatible storage.

Restore thay dữ liệu database hiện tại nên luôn cần xác nhận rõ ràng.

Thay đổi trực tiếp trong provider console được phát hiện khi reconciliation. Service, volume, mount path, domain, image hoặc region bị thiếu hay thay đổi sẽ xuất hiện dưới dạng drift. Thao tác managed không an toàn bị vô hiệu hóa cho tới khi mapping được kiểm tra; Enfyra không tự ghi đè hoặc tạo lại tài nguyên của khách hàng.

## Sử dụng Enfyra runtime

Sau khi provisioning, mở URL project và dùng Enfyra bình thường:

- Tạo table và relation.
- Cấu hình role, route permission, hook, handler, flow và extension.
- Dùng REST API tự sinh, GraphQL tùy chọn và Socket.IO event.
- Tạo API token cho application mà không làm lộ Cloud management credential.

Để tự vận hành runtime local, dùng Docker hoặc toàn quyền sở hữu hạ tầng, bắt đầu ở [Hướng dẫn cài đặt](../getting-started/installation.md).
