# Tài liệu Enfyra

> **Lưu ý**  
> Đây là repository tài liệu. Nếu Enfyra hữu ích với bạn, hãy star repository chính tại [github.com/enfyra/enfyra](https://github.com/enfyra/enfyra).

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Nuxt](https://img.shields.io/badge/Nuxt-4-green.svg)](https://nuxt.com/)
[![Vue](https://img.shields.io/badge/Vue-3-green.svg)](https://vuejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue.svg)](https://www.typescriptlang.org/)

## Enfyra là gì?

Enfyra là backend framework tạo API từ database và metadata của dự án. Bạn có thể tạo bảng, quan hệ, phân quyền và route bằng giao diện quản trị; Enfyra cung cấp REST API ngay sau đó, đồng thời có thể bật GraphQL theo từng bảng.

Khi nghiệp vụ phức tạp hơn, bạn có thể bổ sung handler, hook, flow, WebSocket event hoặc extension mà không phải fork phần lõi.

**Bản demo:** [demo.enfyra.io](https://demo.enfyra.io/)

## Bắt đầu theo mục tiêu

Chọn lộ trình gần nhất với việc bạn cần làm:

- **Tạo ứng dụng đầu tiên:** đọc [Xây dựng ứng dụng đầu tiên](./getting-started/first-app.md), sau đó học [Tạo bảng](./getting-started/table-creation.md) và [Quản lý dữ liệu](./getting-started/data-management.md).
- **Kết nối frontend hoặc mobile app:** bắt đầu từ [Enfyra SDK](./sdk/README.md), rồi dùng [Tài liệu API](./api-reference/README.md) để tra REST contract nền tảng.
- **Dùng trợ lý lập trình AI:** đọc [Sử dụng Enfyra với trợ lý lập trình AI](./integrations/mcp-server.md).
- **Tự vận hành Enfyra:** đọc [Cài đặt](./getting-started/installation.md), sau đó đến [Enfyra Docker](./docker/README.md).
- **Dùng dịch vụ được quản lý:** đọc [Enfyra Cloud](./cloud/README.md).
- **Viết logic nghiệp vụ:** đi theo [Handler và Hook](./app/hooks-handlers/README.md), [Repository](./server/repository-methods/README.md) và [Context](./server/context-reference/README.md).
- **Xây dựng hệ thống multi-tenant:** đọc [Guard](./server/guards.md), [Field permission](./server/field-permissions.md) và [Ví dụ RLS multi-tenant](./examples/multi-tenant-rls-example.md).

## Enfyra phù hợp với

- Nền tảng thương mại điện tử: sản phẩm, đơn hàng, khách hàng và tồn kho.
- Website nội dung: blog, tin tức và cổng tài liệu.
- Ứng dụng nghiệp vụ: CRM, quản lý dự án và công cụ nội bộ.
- Backend cho mobile app: người dùng, đồng bộ dữ liệu, file và thông báo realtime.
- Hệ thống SaaS nhiều tenant với phân quyền theo dữ liệu.
- API tùy chỉnh và workflow cần mở rộng dần theo sản phẩm.

## Bản đồ tài liệu

Tên file được giữ thống nhất giữa bản EN và VI để việc đối chiếu source dễ dàng. Mỗi file tiếng Việt có slug riêng dành cho URL công khai.

```text
vi/
├── README.md
├── architecture-overview.md
├── getting-started/
│   ├── README.md
│   ├── installation.md
│   ├── first-app.md
│   ├── getting-started.md
│   ├── table-creation.md
│   └── data-management.md
├── cloud/
│   └── README.md
├── docker/
│   └── README.md
├── api-reference/
│   ├── README.md
│   ├── overview.md
│   ├── authentication.md
│   ├── crud-operations.md
│   ├── query-parameters.md
│   └── file-storage.md
├── integrations/
│   ├── README.md
│   ├── ssr-frameworks.md
│   └── mcp-server.md
├── sdk/
│   ├── README.md
│   ├── core-client.md
│   ├── authentication.md
│   ├── data-queries.md
│   ├── custom-http.md
│   ├── storage.md
│   ├── realtime.md
│   ├── nuxt.md
│   ├── next.md
│   ├── vue.md
│   └── react.md
├── app/
│   ├── README.md
│   ├── api-integration.md
│   ├── form-system.md
│   ├── filter-system.md
│   ├── relation-picker.md
│   ├── permission-builder.md
│   ├── permission-components.md
│   ├── column-rules.md
│   ├── routing-management.md
│   ├── hooks-handlers/
│   │   ├── README.md
│   │   ├── hooks.md
│   │   ├── custom-handlers.md
│   │   └── package-management.md
│   ├── flows.md
│   ├── storage-management.md
│   ├── menu-management.md
│   ├── theme-color-contract.md
│   ├── extension-system.md
│   ├── header-actions.md
│   ├── page-header.md
│   ├── runtime-monitor.md
│   ├── log-viewing.md
│   └── method-management.md
├── server/
│   ├── README.md
│   ├── api-lifecycle.md
│   ├── query-filtering.md
│   ├── repository-methods/
│   │   ├── README.md
│   │   ├── find.md
│   │   ├── create-update-delete.md
│   │   └── patterns.md
│   ├── context-reference/
│   │   ├── README.md
│   │   ├── repositories.md
│   │   ├── request-data.md
│   │   ├── helpers-cache.md
│   │   ├── logging-errors.md
│   │   └── advanced.md
│   ├── hooks-handlers/
│   │   ├── README.md
│   │   ├── prehooks.md
│   │   ├── posthooks.md
│   │   ├── custom-handlers.md
│   │   └── patterns.md
│   ├── template-syntax.md
│   ├── guards.md
│   ├── field-permissions.md
│   ├── flows.md
│   ├── websocket.md
│   ├── file-handling.md
│   ├── cache-operations.md
│   ├── cluster-architecture.md
│   ├── schema-migration-preview.md
│   └── error-handling.md
└── examples/
    ├── README.md
    ├── api-examples.md
    ├── script-examples.md
    ├── app-examples.md
    ├── user-registration-example.md
    ├── multi-tenant-rls-example.md
    ├── third-party-chat-app.md
    ├── crud-apps/
    │   ├── README.md
    │   ├── todo-app.md
    │   ├── blog-comments.md
    │   ├── catalog-orders.md
    │   └── graphql-read-api.md
    ├── auth-permissions/
    │   ├── README.md
    │   ├── profile-owner-scope.md
    │   ├── team-workspace-rls.md
    │   └── public-contact-form.md
    ├── files-realtime/
    │   ├── README.md
    │   ├── file-attachments.md
    │   ├── avatar-upload.md
    │   ├── realtime-notifications.md
    │   └── activity-feed.md
    ├── automation-integrations/
    │   ├── README.md
    │   ├── webhook-ingest.md
    │   ├── scheduled-cleanup.md
    │   ├── rate-limited-public-api.md
    │   └── outbound-sync.md
    └── admin-operations/
        ├── README.md
        ├── moderation-console.md
        ├── operator-queue.md
        └── settings-page.md
```

## Lộ trình học chi tiết

### Lộ trình nhanh: tạo kết quả đầu tiên

1. Mở [bản demo](https://demo.enfyra.io/) hoặc hoàn tất [cài đặt](./getting-started/installation.md).
2. Làm theo [Xây dựng ứng dụng đầu tiên](./getting-started/first-app.md).
3. Tạo thêm field và relation bằng [Tạo bảng](./getting-started/table-creation.md).
4. Thêm, sửa và liên kết bản ghi theo [Quản lý dữ liệu](./getting-started/data-management.md).
5. Gọi API đã sinh bằng [CRUD](./api-reference/crud-operations.md).

Sau năm bước này, bạn đã có một mô hình dữ liệu hoạt động và có thể truy cập qua API.

### Giai đoạn 1: nền tảng

1. [Tổng quan kiến trúc](./architecture-overview.md) — hiểu app, server, metadata, database, auth, realtime và flow liên kết với nhau thế nào.
2. [Cài đặt](./getting-started/installation.md) — chọn Enfyra Cloud, Docker hoặc workspace tự vận hành.
3. [Bắt đầu](./getting-started/getting-started.md) — làm quen giao diện quản trị.
4. [Tạo bảng](./getting-started/table-creation.md) — field, index, constraint và relation.
5. [Quản lý dữ liệu](./getting-started/data-management.md) — thao tác bản ghi và dữ liệu liên quan.

### Giai đoạn 2: API và xác thực

6. [Tổng quan API](./api-reference/overview.md) — base URL, header và định dạng response.
7. [Xác thực](./api-reference/authentication.md) — đăng nhập, refresh, OAuth và `/me`.
8. [CRUD](./api-reference/crud-operations.md) — đọc và thay đổi dữ liệu.
9. [Tham số truy vấn](./api-reference/query-parameters.md) — filter, fields, sort, page, limit và deep relation.
10. [Tệp và lưu trữ](./api-reference/file-storage.md) — asset, folder và tải file lên.

Nếu xây frontend riêng, đọc [Enfyra SDK](./sdk/README.md) và [SSR framework](./integrations/ssr-frameworks.md) khi ứng dụng có server rendering. Tài liệu `app/` dành cho extension chạy trong Enfyra Admin.

### Giai đoạn 3: giao diện quản trị

11. [Hệ thống form](./app/form-system.md) — field renderer, validation và giá trị form.
12. [Hệ thống bộ lọc](./app/filter-system.md) — tìm kiếm và lọc dữ liệu.
13. [Bộ chọn relation](./app/relation-picker.md) — chọn bản ghi liên quan.
14. [Trình tạo quyền](./app/permission-builder.md) — xây điều kiện phân quyền.
15. [Quản lý menu](./app/menu-management.md) — tổ chức điều hướng.
16. [Extension](./app/extension-system.md) — tạo page, widget và tính năng riêng.
17. [Theme](./app/theme-color-contract.md) — dùng đúng token và surface của Enfyra.
18. [Header action](./app/header-actions.md) và [đầu trang](./app/page-header.md) — tích hợp UI vào shell quản trị.

### Giai đoạn 4: logic nghiệp vụ

19. [Quản lý route](./app/routing-management.md) — tạo endpoint và thiết lập quyền truy cập.
20. [Hook](./app/hooks-handlers/hooks.md) — validate hoặc biến đổi request/response.
21. [Custom handler](./app/hooks-handlers/custom-handlers.md) — triển khai nghiệp vụ thay cho CRUD mặc định.
22. [Repository](./server/repository-methods/README.md) — đọc, tạo, cập nhật và xóa dữ liệu trong script.
23. [Context](./server/context-reference/README.md) — request, user, repository, helper và package có sẵn.
24. [Vòng đời API](./server/api-lifecycle.md) — thứ tự route, hook, handler và response.
25. [Xử lý lỗi](./server/error-handling.md) — trả lỗi rõ ràng và giữ thông tin cần thiết.

### Giai đoạn 5: phân quyền và multi-tenant

26. [Guard](./server/guards.md) — kiểm soát truy cập ở route và runtime.
27. [Field permission](./server/field-permissions.md) — giới hạn field được đọc hoặc thay đổi.
28. [Thành phần phân quyền](./app/permission-components.md) — kiểm soát UI theo quyền.
29. [Ví dụ profile owner scope](./examples/auth-permissions/profile-owner-scope.md) — chỉ cho phép chủ sở hữu truy cập.
30. [Ví dụ team workspace RLS](./examples/auth-permissions/team-workspace-rls.md) — cô lập dữ liệu theo nhóm.
31. [Ví dụ SaaS multi-tenant](./examples/multi-tenant-rls-example.md) — kết hợp relation, hook và extension.

### Giai đoạn 6: tự động hóa và realtime

32. [Flow](./server/flows.md) — trigger, step, retry và lịch chạy.
33. [WebSocket](./server/websocket.md) — gateway, event và kết nối realtime.
34. [Cache](./server/cache-operations.md) — cache phân tán và distributed lock.
35. [File handling](./server/file-handling.md) — xử lý upload trong runtime.
36. [Ví dụ webhook](./examples/automation-integrations/webhook-ingest.md) và [scheduled cleanup](./examples/automation-integrations/scheduled-cleanup.md).

### Giai đoạn 7: production và mở rộng

37. [Enfyra Docker](./docker/README.md) — cấu hình container và dịch vụ phụ thuộc.
38. [Cluster](./server/cluster-architecture.md) — nhiều instance và phối hợp qua Redis.
39. [Schema migration preview](./server/schema-migration-preview.md) — xem tác động trước khi đổi schema.
40. [Runtime monitor](./app/runtime-monitor.md) và [xem log](./app/log-viewing.md) — kiểm tra hệ thống khi vận hành.

## Lộ trình theo loại sản phẩm

- **MVP CRUD:** Giai đoạn 1 → 2 → form và permission ở Giai đoạn 3.
- **Dashboard nội bộ:** [API integration](./app/api-integration.md) → [Extension](./app/extension-system.md) → [Permission component](./app/permission-components.md).
- **Ứng dụng bên thứ ba:** [Enfyra SDK](./sdk/README.md) → [Xác thực bằng SDK](./sdk/authentication.md) → [Query dữ liệu](./sdk/data-queries.md).
- **Ứng dụng SSR bên ngoài:** [Nuxt SDK](./sdk/nuxt.md) hoặc [Next.js SDK](./sdk/next.md) → [SSR framework](./integrations/ssr-frameworks.md) → [Third-party chat](./examples/third-party-chat-app.md).
- **API nghiệp vụ:** [Routing](./app/routing-management.md) → [Custom handler](./app/hooks-handlers/custom-handlers.md) → [Context](./server/context-reference/README.md).
- **SaaS multi-tenant:** [Guard](./server/guards.md) → [Field permission](./server/field-permissions.md) → [Multi-tenant RLS](./examples/multi-tenant-rls-example.md).
- **Hệ thống realtime:** [WebSocket](./server/websocket.md) → [Realtime notification](./examples/files-realtime/realtime-notifications.md) → [Activity feed](./examples/files-realtime/activity-feed.md).
- **Tự động hóa tích hợp:** [Flow](./server/flows.md) → [Webhook ingest](./examples/automation-integrations/webhook-ingest.md) → [Outbound sync](./examples/automation-integrations/outbound-sync.md).
- **Phát triển với AI:** [Enfyra MCP](./integrations/mcp-server.md) → yêu cầu trợ lý đọc project → thực hiện từng thay đổi có kiểm chứng.

## Cài đặt nhanh

Enfyra cần backend server và admin app. Trình cài đặt tạo cả hai trong cùng workspace:

```bash
npx @enfyra/create ten-project
```

Sau đó chọn package manager, database, Redis và thông tin admin theo hướng dẫn. Xem [Hướng dẫn cài đặt](./getting-started/installation.md) để biết đầy đủ các phương án.

## Đóng góp

Khi cập nhật tài liệu:

1. Sửa nội dung tiếng Anh trong `en/` nếu đó là thay đổi nguồn.
2. Cập nhật file tương ứng trong `vi/` và giữ nguyên đường dẫn tương đối.
3. Giữ code, identifier, URL và tên API chính xác với implementation.
4. Viết theo hướng sử dụng và kết quả cần đạt, không biến tài liệu người dùng thành danh sách contract nội bộ.
5. Kiểm tra link, heading, code fence và độ phủ EN/VI trước khi gửi thay đổi.

## Hỗ trợ

- [Repository chính](https://github.com/enfyra/enfyra)
- [Issues](https://github.com/dothinh115/enfyra/issues)
- [Discussions](https://github.com/dothinh115/enfyra/discussions)

Tài liệu được phát hành theo giấy phép MIT. Lưu ý: lõi Enfyra (`server/` và `kernel/`) được phát hành theo [Elastic License 2.0](https://www.elastic.co/licensing/elastic-license); các package khác (app, SDK, MCP server, cloud) theo giấy phép MIT.
