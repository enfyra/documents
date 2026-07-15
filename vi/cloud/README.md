---
slug: cloud
---

# Enfyra Cloud

Enfyra Cloud là lựa chọn managed hosting cho Enfyra. Dùng Cloud khi bạn muốn đưa một Enfyra project lên mạng mà không phải tự provision server, database, Redis, reverse proxy, TLS và deployment pipeline.

Self-hosted Enfyra và Enfyra Cloud cùng dùng một product model: Enfyra runtime, admin app, REST API tự sinh, GraphQL tùy chọn, realtime event, flow, extension, user, role, file và schema management theo metadata. Khác biệt là quyền vận hành: self-hosting tự sở hữu hạ tầng; Cloud vận hành runtime boundary, shared service, deployment, provisioning có payment gate và bảo trì platform.

## Khi nào dùng Cloud

Chọn Enfyra Cloud khi bạn muốn:

- Bắt đầu bằng Enfyra instance hosted thay vì cài Docker hoặc quản lý VPS.
- Tạo project trong Cloud dashboard và nhận URL project cùng admin credential đầu tiên sau provisioning.
- Dùng Enfyra cho production app nhưng không để đội ngũ phải vận hành hạ tầng.
- Có lộ trình rõ ràng từ managed hosting sang self-hosting khi nhu cầu hạ tầng chuyên biệt hơn.

Chọn self-hosting khi cần tự kiểm soát server, database, Redis, storage, network và deployment; chạy trong private network/compliance boundary không thể dùng managed hosting; hoặc tùy biến hạ tầng vượt mô hình gói Cloud.

## Cô lập project

Mỗi Cloud project chạy trong runtime boundary riêng. App logic, cấu hình, credential, tenant metadata và quyền truy cập project được cô lập theo project.

Cloud dùng chung một số dịch vụ để vận hành hiệu quả hơn. Nền tảng điều phối tài nguyên database, edge và dịch vụ hỗ trợ theo nhu cầu thực tế, thay vì dành sẵn một phần tài nguyên không dùng đến cho từng project. Khi các project khác đang nhàn, project có tải có thể sử dụng phần năng lực còn trống.

Nền tảng luôn duy trì năng lực dự phòng và các giới hạn bảo vệ ở edge/runtime để một project không làm ảnh hưởng đến project khác. Gói cao hơn có ít project cùng chia sẻ tài nguyên hơn, đồng thời chịu tải tăng tốt hơn cho từng project.

Trang gói Cloud mô tả giới hạn dành cho khách hàng như storage, transfer, region, checkout và support. Chi tiết raw host package, provider location code, CPU/RAM placement nội bộ không thuộc public plan contract.

## Tạo Cloud project

1. Mở Cloud dashboard tại `https://cloud.enfyra.io`.
2. Đăng nhập hoặc tạo tài khoản.
3. Xác minh email nếu luồng tạo project yêu cầu.
4. Chọn gói và region đang khả dụng.
5. Nhập tên project, subdomain và `SECRET_KEY` của project.
6. Xem lại chi tiết.
7. Hoàn tất checkout khi online payment khả dụng.

Sau xác nhận thanh toán, Cloud provision project bất đồng bộ. Project URL, email đăng nhập và one-time admin password được gửi vào email tài khoản khi provisioning thành công.

Nếu checkout tạm dừng, dùng luồng early-access hoặc contact ở public site thay vì tạo paid project trên dashboard.

## Billing và checkout

Cloud checkout có payment gate. Tạo hoặc nâng cấp project trước hết tạo payment order; project chỉ được provision sau khi payment provider xác nhận order.

Trang giá hiển thị plan subtotal trước buyer-country tax, trừ khi checkout provider thể hiện khác. Thuế và chi tiết thanh toán do provider xử lý trong checkout. Dùng Orders page của Cloud dashboard cho billing record, khôi phục checkout pending và order cũ.

Cloud không cần một customer billing dashboard riêng khi payment provider sở hữu checkout và renewal management. Billing detail của project vẫn gắn với surface project và order trong Cloud dashboard.

## Region

Region được chọn từ catalog khả dụng trong dashboard. Nhãn region dành cho khách hàng mô tả vị trí bằng ngôn ngữ dễ hiểu; provider name, host location code và server package detail không xuất hiện trong customer UI.

Region chưa có năng lực có thể hiện future hoặc disabled. Project chỉ tạo được ở region đang available.

## Cloud và open-source runtime

Từ góc nhìn builder, Cloud project là Enfyra project bình thường. Khi provision xong, bạn dùng Enfyra admin app giống self-hosted:

- Tạo table và relation.
- Cấu hình role, route permission, hook, handler, flow, file và extension.
- Dùng REST API tự sinh, GraphQL tùy chọn và Socket.IO event.
- Kết nối Nuxt, Next.js, SvelteKit, Remix, mobile hoặc server client theo cùng API pattern trong repository này.

Để phát triển local, dùng Docker hoặc tự sở hữu hạ tầng, bắt đầu ở [Hướng dẫn cài đặt](../getting-started/installation.md). Để dùng managed hosting, bắt đầu với Enfyra Cloud.
