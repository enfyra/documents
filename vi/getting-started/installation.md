---
slug: cai-dat
---

# Hướng dẫn cài đặt Enfyra

## Điều kiện cần có

- **Node.js** >= 24.0.0 nếu cài thủ công
- **Package manager** (`npm`, `yarn` hoặc `pnpm`)
- **Database server** (MySQL, PostgreSQL hoặc MongoDB — MariaDB dùng protocol `mysql://`)
- **Redis server**

Hoặc dùng Docker để có bộ all-in-one hoàn chỉnh, được khuyến nghị khi bắt đầu nhanh.

## Các cách cài đặt

### Lựa chọn 0: Enfyra Cloud (managed hosting)

Nếu không muốn tự vận hành server, database, Redis, TLS, reverse proxy hoặc pipeline triển khai, hãy dùng Enfyra Cloud. Cloud tạo project Enfyra trên dashboard; mỗi project có ranh giới runtime riêng. Cloud quản lý năng lực của database, edge và các dịch vụ hỗ trợ dùng chung bằng ngưỡng dự phòng và cơ chế bảo vệ. Gói cao hơn dành mức năng lực vận hành lớn hơn cho từng project.

Dùng Cloud khi muốn xây app Enfyra mà không quản lý hạ tầng. Dùng Docker hoặc cài thủ công bên dưới nếu cần tự sở hữu runtime environment. Xem [Enfyra Cloud](../cloud/README.md) để biết mô hình hosting, luồng tạo project, checkout và khác biệt với self-hosting.

### Lựa chọn 1: Docker (khuyến nghị để bắt đầu nhanh)

Docker image all-in-one gồm backend (Awilix + Express 5), frontend (Nuxt), PostgreSQL/MySQL nhúng tùy chọn và Redis nhúng tùy chọn.

#### Khởi động nhanh với Docker

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

Lệnh này chạy backend ở cổng 1105 nội bộ, frontend ở cổng 3000, PostgreSQL nhúng nếu chưa cấu hình DB và Redis nhúng nếu chưa có `REDIS_URI`.

- Frontend: http://localhost:3000
- API qua app proxy: http://localhost:3000/api/me

Server cũng lắng nghe `1105` **trong container**, nhưng quick-start không publish cổng này ra host. Đây là chủ ý: browser và app client nên dùng proxy `/api` cùng origin. Nếu cần raw server để tích hợp/diagnostic cục bộ, thêm `-p 1105:1105` và gọi `http://localhost:1105` rõ ràng.

**Thông tin mặc định:** email `enfyra@admin.com`, password `1234`.

Để thay đổi admin credential:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e ADMIN_EMAIL=myadmin@example.com \
  -e ADMIN_PASSWORD=secure_password_123 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

Để publish database/Redis nhúng cho công cụ bên ngoài:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -p 5432:5432 \
  -p 6379:6379 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

#### Trước khi dùng Docker ở production

Không triển khai với image default. Đặt admin password riêng và `SECRET_KEY` dài, ổn định trước lần chạy đầu. Lưu key trong secret manager: key ký session và cần để giải mã giá trị có `isEncrypted=true`.

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e ADMIN_EMAIL=admin@example.com \
  -e ADMIN_PASSWORD='use-a-unique-long-password' \
  -e SECRET_KEY='store-a-long-random-secret-outside-this-command-in-production' \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

Sau khi container khởi động, mở `http://localhost:3000`, đăng nhập và xác nhận ứng dụng hoạt động bình thường trước khi kết nối client bên ngoài. Xem [Docker](../docker/README.md) để biết TLS, backup, database/Redis bên ngoài và reverse proxy cho môi trường production.

#### Docker với MySQL nhúng

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e EMBEDDED_DB=mysql \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

#### Docker với database và Redis bên ngoài

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@my-postgres-host:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  -e REDIS_RUNTIME_CACHE=true \
  -e REDIS_USER_CACHE_LIMIT_MB=30 \
  enfyra/enfyra:latest
```

Nếu database và Redis chạy trên máy local, không dùng `localhost` trong `DB_URI` hoặc `REDIS_URI`: từ trong container nó là container đó. Dùng `host.docker.internal`:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

Trên Linux, thêm host gateway nếu Docker không tự cung cấp `host.docker.internal`:

```bash
docker run -d \
  --name enfyra \
  --add-host=host.docker.internal:host-gateway \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

Đảm bảo database và Redis local chấp nhận kết nối từ Docker, không chỉ `127.0.0.1`. Nếu password có ký tự đặc biệt, URL-encode chúng, ví dụ `p@ssw0rd` thành `p%40ssw0rd`.

`REDIS_RUNTIME_CACHE=true` lưu runtime definition snapshot trong Redis để instance cùng `NODE_NAME` dùng chung namespace runtime cache. Dữ liệu người dùng `$cache` / `@CACHE` được giới hạn riêng bởi `REDIS_USER_CACHE_LIMIT_MB`, mặc định 30 MB.

#### Docker mode

Image hỗ trợ ba mode:

1. **`all`** (mặc định): chạy server + app + embedded service.
2. **`server`**: chỉ chạy backend server.
3. **`app`**: chỉ chạy frontend app.

Ví dụ server-only:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://user:password@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  -e REDIS_RUNTIME_CACHE=true \
  -e REDIS_USER_CACHE_LIMIT_MB=30 \
  enfyra/enfyra:latest
```

Ví dụ app-only:

```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://your-backend:1105/ \
  enfyra/enfyra:latest
```

### Lựa chọn 2: Cài thủ công

#### Thiết lập nhanh

Dùng Enfyra create CLI để scaffold một workspace gồm backend server và Nuxt admin app:

```bash
npx @enfyra/create <project-name>
cd <project-name>
npm run dev
```

Workspace được tạo có cấu trúc:

```text
<project-name>/
|-- app/
|   `-- .env
|-- server/
|   `-- .env
|-- scripts/
`-- package.json
```

`app` và `server` là hai ứng dụng riêng trong cùng project. Mỗi thư mục có dependency và lockfile riêng; package root chỉ cung cấp convenience script.

**Server:** http://localhost:1105  
**App:** http://localhost:3000

Server dùng cố định cổng `1105`; CLI kiểm tra trước và hỏi trước khi kill process đang dùng cổng. Nuxt app cấu hình `3000`; khi cổng bận trong development, Nuxt có thể chọn cổng trống khác.

- Package: [@enfyra/create](https://www.npmjs.com/package/@enfyra/create)
- Xem [Tổng quan kiến trúc](../architecture-overview.md) để hiểu backend và frontend phối hợp.

Admin app là client kết nối đến backend. Backend tự sinh API từ metadata database; frontend dùng URL `app/.env` để gọi backend. Luồng dữ liệu là Frontend -> Backend -> Database; frontend không sở hữu generated API.

```text
Database -> Backend APIs (1105) <- Frontend App (3000)
```

#### Các câu hỏi cấu hình

```bash
npx @enfyra/create <project-name>
```

CLI hỏi package manager, tên project, loại/URI database (MySQL, PostgreSQL hoặc MongoDB), tùy chọn SQL read replica và URI replica, Redis URI, admin email và admin password. CLI kiểm tra kết nối database và Redis trước khi tạo project; nếu thất bại bạn có thể nhập lại, vẫn tiếp tục hoặc thoát.

Sau xác nhận, CLI tải Enfyra app/server, tạo `app/.env` và `server/.env`, nối app với local server URL, cài dependency riêng cho `server/` và `app/`, đồng thời tạo root convenience script.

> **Quan trọng – `SECRET_KEY`:** CLI tạo `SECRET_KEY` ngẫu nhiên trong `server/.env`. Giữ giá trị này ổn định và backup cho production. Nó ký auth token, tạo encryption key cho cột `isEncrypted=true`; đổi hoặc mất key làm token cũ mất hiệu lực và không thể giải mã dữ liệu đã mã hóa. Mọi server instance của cùng Enfyra app phải dùng cùng `SECRET_KEY`.

> **Password có ký tự đặc biệt:** URL-encode `@`, `:`, `/`, `%`, `#`, `?`, `&` trong database URI. Ví dụ `p@ssw0rd` thành `p%40ssw0rd`; `pass:word` thành `pass%3Aword`.

> **Database replication, tùy chọn cho SQL:** thêm replica URI khi setup để query đọc round-robin qua replica khỏe; ghi vẫn dùng main database URI.

#### Chạy workspace

Từ workspace root:

```bash
npm run dev
```

Lệnh này chạy server trước, chờ cổng `1105` nhận kết nối rồi chạy app. Dùng lệnh tương ứng nếu đã chọn Yarn hoặc pnpm:

```bash
yarn dev
pnpm run dev
```

Bạn cũng có thể chạy từng phía:

```bash
npm run dev:server
npm run dev:app
```

#### Build và start

Build cả hai ứng dụng từ workspace root:

```bash
npm run build
```

Root build script build server trước rồi tới app. Để chạy output production:

```bash
npm run start
```

Start script cũng chạy server trước và chỉ chạy app sau khi server port sẵn sàng.

## Bước tiếp theo

- [Bắt đầu](./getting-started.md) – làm quen giao diện và tạo bảng đầu tiên.
- [Tạo bảng](./table-creation.md) – tạo bảng với mọi field type.
- [Quản lý dữ liệu](./data-management.md) – quản lý record và relation.
- [Tài liệu App](../app/README.md) – frontend và tùy biến.
- [Tài liệu Server](../server/README.md) – backend nâng cao và phát triển API.
