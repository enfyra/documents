# Enfyra Docker - Usage Guide

> **New to Enfyra?** See the [Installation Guide](../getting-started/installation.md) for complete setup instructions, including Docker and manual installation options.

##  Overview

The `dothinh115/enfyra` (or `enfyra/enfyra`) image can run in **3 different modes**:

1. **`all`** (default) - Run both server + app + embedded services
2. **`server`** - Run backend server only
3. **`app`** - Run frontend app only

All from the **same image**, just different environment variables!

---

##  1. Run All-in-One (Server + App + Embedded Services)

### Simplest way (single command):

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

 Runs with:
- **Server** (port 1105 internal)
- **App** (port 3000, exposed)
- **Embedded PostgreSQL** (if no DB config)
- **Embedded Redis** (if no REDIS_URI)

### With MySQL (Embedded):

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e EMBEDDED_DB=mysql \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

### With external database and Redis:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@my-postgres-host:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

> **Note:** The database engine is auto-detected from the `DB_URI` protocol prefix (`mysql://`, `postgres://`, `mongodb://`, `sqlite://`). No separate `DB_TYPE` env var is needed.

---

##  2. Run Server Only (Backend)

### Run server only, no app:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

**With replica databases:**

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@master-postgres:5432/enfyra \
  -e DB_REPLICA_URIS=postgresql://enfyra:secret@replica1:5432/enfyra,postgresql://enfyra:secret@replica2:5432/enfyra \
  -e DB_READ_FROM_MASTER=false \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

### Server with embedded services:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -v enfyra-server-data:/app/data \
  dothinh115/enfyra:latest
```

**Note**: When `ENFYRA_MODE=server`, app will not be started, only backend API.

---

##  3. Run App Only (Frontend)

### Run app only, requires API_URL pointing to external server:

```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://your-server-host:1105/ \
  dothinh115/enfyra:latest
```

### App with HTTPS backend:

```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=https://api.your-domain.com/ \
  dothinh115/enfyra:latest
```

**Note**: 
- When `ENFYRA_MODE=app`, **must** set `API_URL`
- Server and embedded services will not be started

---

##  Mode Comparison

| Mode | Server | App | Embedded Redis | Embedded DB | Use Case |
|------|--------|-----|----------------|-------------|----------|
| `all` (default) |  |  |  (if no REDIS_URI) |  (if no DB config) | Single container, simple |
| `server` |  |  |  (if no REDIS_URI) |  (if no DB config) | Backend API only, cluster |
| `app` |  |  |  |  | Frontend only, scale separately |

---

##  Scaling with Multiple Containers

### Scale Server (Backend cluster):

```bash
# Server node 1
docker run -d \
  --name enfyra-server-1 \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@shared-postgres:5432/enfyra \
  -e REDIS_URI=redis://shared-redis:6379/0 \
  dothinh115/enfyra:latest

# Server node 2
docker run -d \
  --name enfyra-server-2 \
  -p 1106:1105 \
  -e ENFYRA_MODE=server \
  -e DB_URI=postgresql://enfyra:secret@shared-postgres:5432/enfyra \
  -e REDIS_URI=redis://shared-redis:6379/0 \
  dothinh115/enfyra:latest

# App (pointing to load balancer or server nodes)
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://load-balancer:1105/ \
  dothinh115/enfyra:latest
```

---

##  Environment Variables Reference

### Mode
- `ENFYRA_MODE`: `all`, `server`, or `app` (default: `all`)

### Database (if not set  uses embedded)

- `EMBEDDED_DB`: Embedded database type when no `DB_URI` is set: `postgres` or `mysql` (default: `postgres`). Has no effect when `DB_URI` is provided.

**Primary (Recommended):**
- `DB_URI`: Database connection URI — the database type is **auto-detected** from the URI protocol, so `DB_TYPE` does not need to be set.
  - PostgreSQL: `postgresql://user:password@host:port/database`
  - MySQL: `mysql://user:password@host:port/database`
  - MongoDB: `mongodb://user:password@host:port/database?authSource=admin`
  - Example: `postgresql://enfyra:secret@my-postgres:5432/enfyra`
  - **Note**: URL-encode special characters in password if needed (`@`  `%40`, `:`  `%3A`, `/`  `%2F`, `%`  `%25`, `#`  `%23`, `?`  `%3F`, `&`  `%26`)

**Replica Configuration (Optional):**
- `DB_REPLICA_URIS`: Comma-separated replica URIs
  - Example: `postgresql://user:pass@replica1:5432/db,postgresql://user:pass@replica2:5432/db`
- `DB_READ_FROM_MASTER`: Include master in read pool (`true`/`false`, default: `false`)
  - `false`: Read queries only use replicas (master only for writes)
  - `true`: Read queries use master + replicas (round-robin)

### Redis (if not set  uses embedded)
- `REDIS_URI`: Redis connection string
  - Format: `redis://user:pass@host:port/db` or `redis://host:port/db`
- `DEFAULT_TTL`: Default TTL for cache entries in seconds (default: `5`)

### Server
- `PORT`: Server port (default: `1105`)
- `ENFYRA_SERVER_WORKERS`: Number of workers for cluster (default: `1`)
- `SECRET_KEY`: JWT secret key (default: `enfyra_secret_key_change_in_production`)
- `NODE_NAME`: Node instance name for logs/cluster (default: auto-generated UUID - ensures 100% uniqueness across nodes)

### Auth (Optional)
- `SALT_ROUNDS`: bcrypt salt rounds (default: `10`)
- `ACCESS_TOKEN_EXP`: Access token expiration (default: `15m`)
- `REFRESH_TOKEN_NO_REMEMBER_EXP`: Refresh token expiration without remember (default: `1d`)
- `REFRESH_TOKEN_REMEMBER_EXP`: Refresh token expiration with remember (default: `7d`)

### Admin Account (Optional)
- `ADMIN_EMAIL`: Default admin email (default: `enfyra@admin.com`)
- `ADMIN_PASSWORD`: Default admin password (default: `1234`)

**Example:**
```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e ADMIN_EMAIL=myadmin@example.com \
  -e ADMIN_PASSWORD=secure_password_123 \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

### App
- `ENFYRA_APP_PORT`: App port (default: `3000`)
- `API_URL`: Backend API URL the Nuxt app calls (automatically set if `ENFYRA_MODE=all`)

### Environment
- `NODE_ENV`: Environment `development`, `production`, or `test` (default: `production`)

---

##  Tips

1. **Data persistence**: Always use volume to persist data:
   ```bash
   -v enfyra-data:/app/data
   ```

2. **Production**: Recommended to use external DB and Redis for production

3. **Development**: Can use embedded services for quick testing

4. **Port Conflict**: 
   - Container will automatically check port before starting embedded services
   - If port is already in use, will show warning but continue
   - **Solution**: Set `REDIS_URI` or `DB_HOST` to use external service
   - **Note**: If using `--network host`, port conflict may occur with host

5. **Connect to Embedded Database from DBeaver/External Tools**:
   
   **PostgreSQL** (port 5432):
   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 5432:5432 \  # ← Expose PostgreSQL port
     -v enfyra-data:/app/data \
     dothinh115/enfyra:latest
   ```
   
   Then connect with DBeaver:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `enfyra`
   - **Username**: `enfyra`
   - **Password**: `enfyra_password_123`
   
   **MySQL** (port 3306):
   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 3306:3306 \  # ← Expose MySQL port
     -e EMBEDDED_DB=mysql \
     -v enfyra-data:/app/data \
     dothinh115/enfyra:latest
   ```
   
   Then connect with DBeaver:
   - **Host**: `localhost`
   - **Port**: `3306`
   - **Database**: `enfyra`
   - **Username**: `enfyra`
   - **Password**: `enfyra_password_123`
   
   **Redis** (port 6379):
   ```bash
   docker run -d \
     --name enfyra \
     -p 3000:3000 \
     -p 6379:6379 \  # ← Expose Redis port
     -v enfyra-data:/app/data \
     dothinh115/enfyra:latest
   ```
   
   **Default Admin User** (auto-created on init):
   - **Email**: `enfyra@admin.com`
   - **Password**: `1234`

6. **Logs**: View logs for each service:
   ```bash
   docker exec enfyra cat /var/log/supervisor/server.log
   docker exec enfyra cat /var/log/supervisor/app.log
   docker exec enfyra cat /var/log/supervisor/redis.log
   docker exec enfyra cat /var/log/supervisor/postgres.log
   ```

##  Troubleshooting

### Port Conflict Warning

If you see a warning about port conflict:

```
  WARNING: Port 6379 is already in use!
   This may cause conflicts when starting embedded Redis.
```

**Solutions:**
1. **Use external service** (recommended):
   ```bash
   -e REDIS_URI=redis://your-redis-host:6379/0
   -e DB_URI=postgresql://user:pass@your-db-host:5432/dbname
   ```

2. **Stop service on host** (if not needed):
   ```bash
   # Stop local Redis
   sudo systemctl stop redis
   
   # Stop local PostgreSQL
   sudo systemctl stop postgresql
   ```

3. **Use different port** (not recommended, as embedded services use fixed ports):
   - Should use external service instead of changing port

