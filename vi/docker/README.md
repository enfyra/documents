---
slug: docker
---

# Enfyra Docker – Hướng dẫn sử dụng

> **Mới dùng Enfyra?** Xem [Hướng dẫn cài đặt](../getting-started/installation.md) để biết hướng dẫn thiết lập đầy đủ, bao gồm các tùy chọn cài đặt bằng Docker và cài đặt thủ công.

## Tổng quan

Image `enfyra/enfyra` có thể chạy ở **3 mode khác nhau**:

1. **`all`** (mặc định) - Chạy server + app + các service nhúng
2. **`server`** - Chỉ chạy backend server
3. **`app`** - Chỉ chạy frontend app

Tất cả đều dùng **cùng một image**, chỉ khác nhau ở các biến môi trường!

---

## 1. Chạy all-in-one (Server + App + các service nhúng)

### Cách đơn giản nhất (một lệnh):

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

Lệnh này chạy:

- **Server** (cổng 1105 nội bộ)
- **App** (cổng 3000, được expose)
- **PostgreSQL nhúng** (nếu không có cấu hình DB)
- **Redis nhúng** (nếu không có `REDIS_URI`)

### Với MySQL nhúng:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e EMBEDDED_DB=mysql \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

### Với database và Redis bên ngoài:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@my-postgres-host:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  enfyra/enfyra:latest
```

Nếu database và Redis đang chạy trên máy local để test, hãy dùng `host.docker.internal` thay cho `localhost`. Bên trong container, `localhost` trỏ đến chính container đó:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

Trên Linux, thêm host gateway mapping nếu cần:

```bash
docker run -d \
  --name enfyra \
  --add-host=host.docker.internal:host-gateway \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

Database và Redis trên máy local phải chấp nhận kết nối từ Docker, không chỉ từ `127.0.0.1`.

> **Lưu ý:** Database engine được tự động nhận diện từ protocol prefix của `DB_URI` (`mysql://`, `postgres://`, `mongodb://`). Không cần biến môi trường `DB_TYPE` riêng.

---

## 2. Chỉ chạy server (Backend)

### Chỉ chạy server, không chạy app:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  enfyra/enfyra:latest
```

**Với các database replica:**

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@master-postgres:5432/enfyra \
  -e DB_REPLICA_URIS=postgresql://enfyra:secret@replica1:5432/enfyra,postgresql://enfyra:secret@replica2:5432/enfyra \
  -e DB_READ_FROM_MASTER=false \
  -e REDIS_URI=redis://my-redis:6379/0 \
  enfyra/enfyra:latest
```

### Server với các service nhúng:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -v enfyra-server-data:/app/data \
  enfyra/enfyra:latest
```

**Lưu ý**: Khi `ENFYRA_MODE=server`, app sẽ không khởi động, chỉ có backend API.

---

## 3. Chỉ chạy app (Frontend)

### Chỉ chạy app, yêu cầu `API_URL` trỏ đến server bên ngoài:

```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://your-server-host:1105/ \
  enfyra/enfyra:latest
```

### App với backend HTTPS:

```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=https://api.your-domain.com/ \
  enfyra/enfyra:latest
```

**Lưu ý**:

- Khi `ENFYRA_MODE=app`, **bắt buộc** đặt `API_URL`
- Server và các service nhúng sẽ không khởi động

---

## So sánh mode

| Mode | Server | App | Redis nhúng | DB nhúng | Trường hợp sử dụng |
|------|--------|-----|-------------|----------|--------------------|
| `all` (mặc định) | Có | Có | Có (nếu không có `REDIS_URI`) | Có (nếu không có cấu hình DB) | Một container, đơn giản |
| `server` | Có | Không | Có (nếu không có `REDIS_URI`) | Có (nếu không có cấu hình DB) | Chỉ backend API, cluster |
| `app` | Không | Có | Không | Không | Chỉ frontend, scale riêng |

---

## Scale với nhiều container

### Scale server (backend cluster):

```bash
# Server node 1
docker run -d \
  --name enfyra-server-1 \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@shared-postgres:5432/enfyra \
  -e REDIS_URI=redis://shared-redis:6379/0 \
  enfyra/enfyra:latest

# Server node 2
docker run -d \
  --name enfyra-server-2 \
  -p 1106:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@shared-postgres:5432/enfyra \
  -e REDIS_URI=redis://shared-redis:6379/0 \
  enfyra/enfyra:latest

# App (pointing to load balancer or server nodes)
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://load-balancer:1105/ \
  enfyra/enfyra:latest
```

---

## Tham chiếu biến môi trường

### Mode

- `ENFYRA_MODE`: `all`, `server` hoặc `app` (mặc định: `all`)

### Database (nếu không đặt, hệ thống dùng database nhúng)

- `EMBEDDED_DB`: Loại database nhúng khi không đặt `DB_URI`: `postgres` hoặc `mysql` (mặc định: `postgres`). Không có tác dụng khi đã cung cấp `DB_URI`.

**Primary (khuyến nghị):**

- `DB_URI`: URI kết nối database — loại database được **tự động nhận diện** từ protocol của URI nên không cần đặt `DB_TYPE`.
  - PostgreSQL: `postgresql://user:password@host:port/database`
  - MySQL: `mysql://user:password@host:port/database`
  - MongoDB: `mongodb://user:password@host:port/database?authSource=admin`
  - Ví dụ: `postgresql://enfyra:secret@my-postgres:5432/enfyra`
  - **Lưu ý**: URL-encode các ký tự đặc biệt trong password nếu cần (`@` → `%40`, `:` → `%3A`, `/` → `%2F`, `%` → `%25`, `#` → `%23`, `?` → `%3F`, `&` → `%26`)

**Cấu hình replica (tùy chọn):**

- `DB_REPLICA_URIS`: Các URI replica, ngăn cách bằng dấu phẩy
  - Ví dụ: `postgresql://user:pass@replica1:5432/db,postgresql://user:pass@replica2:5432/db`
- `DB_READ_FROM_MASTER`: Đưa master vào read pool (`true`/`false`, mặc định: `false`)
  - `false`: Các read query chỉ dùng replica (master chỉ dành cho write)
  - `true`: Các read query dùng master + replica (round-robin)

### Redis (nếu không đặt, hệ thống dùng Redis nhúng)

- `REDIS_URI`: Chuỗi kết nối Redis
  - Định dạng: `redis://user:pass@host:port/db` hoặc `redis://host:port/db`
- `DEFAULT_TTL`: TTL mặc định của cache entry, tính bằng giây (mặc định: `5`)
- `REDIS_RUNTIME_CACHE`: Lưu snapshot định nghĩa runtime của Enfyra trong Redis thay vì memory của từng instance (mặc định: `false`)
- `REDIS_USER_CACHE_LIMIT_MB`: Mức phân bổ mềm cho dữ liệu người dùng `$cache` / `@CACHE` (mặc định: `30`)
- `REDIS_USER_CACHE_MAX_VALUE_BYTES`: Giới hạn tùy chọn cho từng value của `$cache` / `@CACHE`; `0` tắt giới hạn

### Server

- `PORT`: Cổng server (mặc định: `1105`)
- `ENFYRA_SERVER_WORKERS`: Số worker cho cluster (mặc định: `1`)
- `SECRET_KEY`: Secret của server dùng để ký auth token và mã hóa field có `isEncrypted=true` (mặc định: `enfyra_secret_key_change_in_production`)
- `NODE_NAME`: Tên node instance dùng cho log/cluster (mặc định: UUID tự động tạo, bảo đảm tính duy nhất giữa các node)

Đặt `SECRET_KEY` rõ ràng, có entropy cao cho production và giữ giá trị này ổn định. Backup an toàn và dùng cùng một giá trị trên mọi server instance của cùng một Enfyra app. Việc thay đổi hoặc làm mất key sẽ vô hiệu hóa các auth token hiện có và khiến các giá trị field đã mã hóa không thể giải mã.

Tạo một key trước khi deploy Docker lên production:

```bash
openssl rand -hex 32
```

### Auth (tùy chọn)

- `SALT_ROUNDS`: Số vòng salt của bcrypt (mặc định: `10`)
- `ACCESS_TOKEN_EXP`: Thời hạn access token (mặc định: `15m`)
- `REFRESH_TOKEN_NO_REMEMBER_EXP`: Thời hạn refresh token khi không ghi nhớ đăng nhập (mặc định: `1d`)
- `REFRESH_TOKEN_REMEMBER_EXP`: Thời hạn refresh token khi ghi nhớ đăng nhập (mặc định: `7d`)

### Tài khoản admin (tùy chọn)

- `ADMIN_EMAIL`: Email admin mặc định (mặc định: `enfyra@admin.com`)
- `ADMIN_PASSWORD`: Password admin mặc định (mặc định: `1234`)

**Ví dụ:**

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e ADMIN_EMAIL=myadmin@example.com \
  -e ADMIN_PASSWORD=secure_password_123 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

Đổi credential mặc định trước khi public instance. Xem [Cài đặt](../getting-started/installation.md) để biết cách quick start và [Docker](../getting-started/installation.md#docker-modes) để biết hướng dẫn tích hợp.

### App

- `ENFYRA_APP_PORT`: Cổng app (mặc định: `3000`)
- `API_URL`: URL backend API mà Nuxt app gọi đến (được tự động đặt nếu `ENFYRA_MODE=all`)

### Môi trường

- `NODE_ENV`: Môi trường `development`, `production` hoặc `test` (mặc định: `production`)

---

## Mẹo

1. **Lưu trữ dữ liệu bền vững**: Luôn dùng volume để duy trì dữ liệu:

   ```bash
   -v enfyra-data:/app/data
   ```

2. **Production**: Nên dùng DB và Redis bên ngoài cho production

3. **Development**: Có thể dùng các service nhúng để test nhanh

4. **Xung đột cổng**:

   - Container sẽ tự động kiểm tra cổng trước khi khởi động các service nhúng
   - Nếu cổng đã được sử dụng, hệ thống sẽ hiện cảnh báo nhưng vẫn tiếp tục
   - **Giải pháp**: Đặt `REDIS_URI` hoặc `DB_HOST` để dùng service bên ngoài
   - **Lưu ý**: Nếu dùng `--network host`, có thể xảy ra xung đột với cổng trên host

5. **Kết nối tới database nhúng từ DBeaver/công cụ bên ngoài**:

   **PostgreSQL** (cổng 5432):

   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 5432:5432 \  # ← Expose PostgreSQL port
     -v enfyra-data:/app/data \
     enfyra/enfyra:latest
   ```

   Sau đó kết nối bằng DBeaver:

   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `enfyra`
   - **Username**: `enfyra`
   - **Password**: `enfyra_password_123`

   **MySQL** (cổng 3306):

   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 3306:3306 \  # ← Expose MySQL port
     -e EMBEDDED_DB=mysql \
     -v enfyra-data:/app/data \
     enfyra/enfyra:latest
   ```

   Sau đó kết nối bằng DBeaver:

   - **Host**: `localhost`
   - **Port**: `3306`
   - **Database**: `enfyra`
   - **Username**: `enfyra`
   - **Password**: `enfyra_password_123`

   **Redis** (cổng 6379):

   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 6379:6379 \  # ← Expose Redis port
     -v enfyra-data:/app/data \
     enfyra/enfyra:latest
   ```

   **Người dùng admin mặc định** (được tự động tạo khi khởi tạo):

   - **Email**: `enfyra@admin.com`
   - **Password**: `1234`

6. **Log**: Xem log của từng service:

   ```bash
   docker exec enfyra cat /var/log/supervisor/server.log
   docker exec enfyra cat /var/log/supervisor/app.log
   docker exec enfyra cat /var/log/supervisor/redis.log
   docker exec enfyra cat /var/log/supervisor/postgres.log
   ```

## Xử lý sự cố

### Cảnh báo xung đột cổng

Nếu thấy cảnh báo về xung đột cổng:

```
  WARNING: Port 6379 is already in use!
   This may cause conflicts when starting embedded Redis.
```

**Giải pháp:**

1. **Dùng service bên ngoài** (khuyến nghị):

   ```bash
   -e REDIS_URI=redis://your-redis-host:6379/0
   -e DB_URI=postgresql://user:pass@your-db-host:5432/dbname
   ```

2. **Dừng service trên host** (nếu không cần):

   ```bash
   # Stop local Redis
   sudo systemctl stop redis
   
   # Stop local PostgreSQL
   sudo systemctl stop postgresql
   ```

3. **Dùng cổng khác** (không khuyến nghị vì các service nhúng dùng cổng cố định):

   - Nên dùng service bên ngoài thay vì đổi cổng
