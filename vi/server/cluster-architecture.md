---
slug: kien-truc-cum
---

# Kiến trúc cụm

Nhiều process Enfyra **server** có thể cùng chạy trên một database và Redis. Chúng phối hợp qua **Redis Pub/Sub** (tín hiệu nạp lại cache), **BullMQ dùng Redis** (tác vụ nền), **Redis cho Socket.IO** (`@socket.io/redis-adapter`) và **Redis lease** theo namespace cho provisioning hệ thống và đăng ký scheduler.

Nội dung trang này phản ánh server mã nguồn mở hiện tại trong repository [`server`](https://github.com/enfyra/server).

## Phần nào thực sự “stateless”

- **HTTP request** không dựa vào file session cục bộ: xác thực dùng JWT, còn session nằm trong database (`enfyra_session`).
- Mặc định, mỗi process giữ cache định nghĩa runtime trong bộ nhớ (metadata, route, dữ liệu liên quan GraphQL, package, cấu hình storage và OAuth, định nghĩa websocket, flow, cây thư mục...). Các cấu trúc này được dựng lại từ database sau khi khởi động hoặc khi instance khác phát tín hiệu reload.
- Khi `REDIS_RUNTIME_CACHE=true`, snapshot định nghĩa runtime được lưu trên Redis dưới namespace `NODE_NAME` hiện tại. Các instance cùng `NODE_NAME` đọc chung snapshot thay vì mỗi instance giữ một bản sao đầy đủ.
- Instance vẫn không phải “không có state trong RAM”: mỗi process giữ object runtime đang hoạt động, client, queue, worker và state theo request. Chúng có thể thay thế lẫn nhau nếu dùng chung DB + Redis và cùng namespace cluster.

## Định danh instance

- Khi khởi động, `InstanceService` gán **instance ID ngẫu nhiên gồm 32 ký tự hex**. ID này dùng để **bỏ qua thông điệp Pub/Sub do chính process phát ra**, tránh reload tự kích hoạt ngay lập tức.
- **`NODE_NAME` tách biệt** với instance ID. Nó **không** tự tạo UUID cho Pub/Sub.

## Đồng bộ cache trong thực tế

Khi một cache service reload (từ Admin, invalidation metadata...), mẫu xử lý thông thường của `BaseCacheService` / `MetadataCacheService` là:

1. Phát một JSON nhỏ trên Redis channel của cache: `{ instanceId, type: 'RELOAD_SIGNAL', timestamp }`.
2. Không phát toàn bộ payload cache qua Redis; các instance còn lại luôn **truy vấn lại database**.
3. Subscriber cùng channel đọc thông điệp. Nếu `instanceId` không phải của mình, nó tự **reload từ DB** và làm mới state trong bộ nhớ.

**Lưu ý:** Chế độ cache trong bộ nhớ có thể dựng lại định nghĩa độc lập trên nhiều instance. Chế độ Redis runtime-cache dùng refresh lock theo snapshot và snapshot dùng chung. Cả hai chế độ đều không bảo đảm side effect ứng dụng chạy đúng một lần.

## `NODE_NAME` và Redis channel

`RedisPubSubService` tạo tên channel thực tế như sau:

- `BASE_CHANNEL` khi chưa đặt `NODE_NAME`
- `BASE_CHANNEL:NODE_NAME` khi đã đặt `NODE_NAME`

**Mọi API instance cần dùng chung metadata/route/runtime cache đang hoạt động phải có cùng `NODE_NAME` (hoặc đều không đặt).** Nếu mỗi máy dùng `NODE_NAME` khác nhau, chúng subscribe vào các channel khác nhau và đọc namespace Redis runtime cache khác nhau.

`NODE_NAME` **không** nhằm làm giá trị duy nhất cho từng instance; đây là một **phân đoạn môi trường/deployment** tùy chọn trong tên channel.

## Redis runtime cache và user cache

`REDIS_RUNTIME_CACHE=true` bật snapshot định nghĩa runtime trên Redis. Đây là cache do hệ thống Enfyra sở hữu, tách biệt với dữ liệu `$cache` của ứng dụng.

`$ctx.$cache` và `@CACHE` dùng managed user cache. Người viết script dùng logical key như `user:123`; Enfyra lưu key dưới namespace app hiện tại là `NODE_NAME:user_cache:*`.

User cache được điều khiển bởi:

- `REDIS_USER_CACHE_LIMIT_MB`: mức phân bổ mềm cho giá trị user-cache, mặc định `30`.
- `REDIS_USER_CACHE_MAX_VALUE_BYTES`: giới hạn tùy chọn cho từng giá trị; `0` tắt giới hạn này.

Khi vượt mức phân bổ, Enfyra chỉ loại bỏ user-cache key ít được dùng gần đây nhất. System key không bị tính hoặc bị xóa theo giới hạn user-cache này.

Redis Admin gắn nhãn các key thuộc app hiện tại, gồm runtime cache, user cache, BullMQ, Socket.IO, runtime monitor và lock. Công cụ không đọc hoặc chỉnh sửa key ngoài namespace `NODE_NAME` hiện tại.

## BullMQ (tác vụ nền)

BullMQ dùng cùng Redis connection với app. **Key prefix** của queue là namespace `NODE_NAME` hiện tại nếu giá trị này được đặt.

- Trong **một cluster logic**, mọi Enfyra server instance nên dùng **cùng** `NODE_NAME` (hoặc đều không đặt để process nào cũng dùng prefix `bull`). Nếu không, mỗi instance chỉ xử lý queue cô lập của riêng nó; chẳng hạn session cleanup hoặc websocket worker có thể không chạy trên toàn cluster như kỳ vọng.

## Khởi tạo và nâng cấp hệ thống

Provisioning schema và migration schema/data đã khai báo chạy khi database chưa khởi tạo hoặc `enfyraVersion` đã lưu cũ hơn server đang chạy. Lease có gia hạn và kiểm tra chủ sở hữu phối hợp các replica; readiness chỉ được công bố sau khi trạng thái đích đã được xác minh. Database đã khởi tạo ở đúng phiên bản đang chạy bỏ qua bước này.

Tự động hóa ứng dụng nằm trong flow, route handler, hook và các lifecycle script được hỗ trợ. Startup và cache reload không thực thi một registry startup script do quản trị viên viết.

## Dọn dẹp session

Bản ghi `enfyra_session` hết hạn được xóa bởi repeatable job **BullMQ** trong queue `sys_session-cleanup` (`SYSTEM_QUEUES.SESSION_CLEANUP`), processor có **concurrency 1**, lịch **`0 2 * * *`** (mỗi ngày). Mỗi process chạy một worker; đăng ký scheduler theo namespace ngăn các replica tạo lại cùng scheduler. Concurrency của worker áp dụng cho từng process, không bảo đảm side effect ứng dụng chạy đúng một lần trên toàn cluster.

## WebSocket giữa các node

`DynamicWebSocketGateway` cấu hình **`@socket.io/redis-adapter`** để Socket.IO room và emit hoạt động qua nhiều Node process dùng chung Redis.

Client vẫn cần chiến lược load balancer tương thích WebSocket, ví dụ sticky session hoặc TCP pass-through. Redis adapter xử lý việc truyền sự kiện **giữa các server**, không xử lý HTTP stickiness.

## Tên Redis Pub/Sub channel (base key)

Cache builder định nghĩa các base channel; `RedisPubSubService` thêm hậu tố `NODE_NAME`:

| Base channel | Mục đích |
| --- | --- |
| `enfyra:metadata-cache-sync` | Cache metadata bảng/column/relation |
| `enfyra:route-cache-sync` | Route trie, handler, hook, permission |
| `enfyra:package-cache-sync` | Danh sách package server / phối hợp CDN cache |
| `enfyra:storage-config-cache-sync` | Cấu hình storage |
| `enfyra:oauth-config-cache-sync` | Cấu hình OAuth provider |
| `enfyra:websocket-cache-sync` | Định nghĩa websocket gateway/event |
| `enfyra:flow-cache-sync` | Định nghĩa flow cho scheduler/dispatch |
| `enfyra:folder-tree-cache-sync` | Cache cây thư mục |
| `enfyra:guard-cache-sync` | Định nghĩa và quy tắc guard |
| `enfyra:setting-cache-sync` | System setting (`maxQueryDepth`...) |

Reload schema GraphQL đi theo cùng pipeline invalidation metadata/route và `GraphqlService.reloadSchema()`; hằng số hiện không có Pub/Sub channel `enfyra:graphql-*` riêng.

Tại thời điểm viết tài liệu này, cây server mã nguồn mở **không** có channel `enfyra:ai-config-cache-sync`. Nếu deployment của bạn bổ sung cache cấu hình AI, hãy xem đó là phần riêng của sản phẩm.

## Invalidation nào reload cache nào

`RELOAD_CHAINS` tại `src/engines/cache/services/cache-orchestrator.service.ts` ánh xạ các bảng metadata tới cache chịu ảnh hưởng (metadata, helper, route, GraphQL, storage, websocket, package, OAuth, folder tree, flow, guard, setting). Sau một lần ghi phù hợp, orchestrator reload chuỗi phụ thuộc, kích hoạt snapshot rồi phát tín hiệu có version tới các instance khác. Reload thất bại ở instance nhận được thử lại trong giới hạn; Pub/Sub không phải log lưu bền để replay.

## Khả năng chịu lỗi trong thực tế

- **Redis không chạy khi khởi động:** khởi tạo Pub/Sub client có thể thất bại, process có thể không khởi động sạch; hãy kiểm tra log (`RedisPubSub`).
- **Mất Redis khi đang chạy:** cache trong bộ nhớ đã tải vẫn phục vụ dữ liệu **cũ** cho tới TTL hoặc khi vận hành reload. Tín hiệu reload mới không thể publish/subscribe đến khi Redis hoạt động lại.

## Checklist mở rộng theo chiều ngang

1. **Cùng database** cho mọi API instance (`DB_URI` / Mongo URI, cùng logical DB).
2. **Cùng Redis** cho Pub/Sub, BullMQ và Socket.IO adapter (`REDIS_URI`, hoặc host/port/password tương ứng nếu tách cấu hình).
3. **Cùng `NODE_NAME` trên mọi instance trong một cluster** (hoặc tất cả đều không đặt) để Pub/Sub channel, namespace Redis runtime cache, namespace user-cache và BullMQ prefix trùng nhau.
4. Đặt **load balancer** trước HTTP/WebSocket phù hợp với nền tảng của bạn.

Mật khẩu chứa ký tự đặc biệt trong URI phải được URL-encode, tương tự mọi JDBC/Redis URL.

**SQL read replica tùy chọn:** `DB_REPLICA_URIS`, `DB_READ_FROM_MASTER`; xem [Cài đặt](../getting-started/installation.md) để biết chi tiết.

## Lợi ích

- Mở rộng HTTP bằng cách thêm process phía sau load balancer.
- Redis adapter và DB dùng chung giúp hành vi websocket/API nhất quán giữa các node.
- Provisioning lease có gia hạn và scheduler được phối hợp giúp replica chia sẻ khởi tạo và tác vụ nền; side effect ứng dụng vẫn nên có tính idempotent.
