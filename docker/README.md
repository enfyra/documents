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
  -e DB_TYPE=postgres \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

 Runs with:
- **Server** (port 1105 internal)
- **App** (port 3000, exposed)
- **Embedded PostgreSQL** (if no DB config)
- **Embedded Redis** (if no REDIS_URI)

### With MySQL:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_TYPE=mysql \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

### With external database and Redis:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@my-postgres-host:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

**Or with legacy vars (backward compatible):**

---

##  2. Run Server Only (Backend)

### Run server only, no app:

```bash
docker run -d \
  --name enfyra-server \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_TYPE=postgres \
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
  -e DB_TYPE=postgres \
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
  -e DB_TYPE=postgres \
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
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@shared-postgres:5432/enfyra \
  -e REDIS_URI=redis://shared-redis:6379/0 \
  dothinh115/enfyra:latest

# Server node 2
docker run -d \
  --name enfyra-server-2 \
  -p 1106:1105 \
  -e ENFYRA_MODE=server \
  -e DB_TYPE=postgres \
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

### Required
- `DB_TYPE`: `postgres`, `mysql`, or `mongodb` (default: `postgres`)

### Mode
- `ENFYRA_MODE`: `all`, `server`, or `app` (default: `all`)

### Database (if not set  uses embedded)

**Primary (Prisma-style URI - Recommended):**
- `DB_URI`: Database connection URI
  - PostgreSQL: `postgresql://user:password@host:port/database`
  - MySQL: `mysql://user:password@host:port/database`
  - Example: `postgresql://enfyra:secret@my-postgres:5432/enfyra`
  - **Note**: URL-encode special characters in password if needed (`@`  `%40`, `:`  `%3A`, `/`  `%2F`, `%`  `%25`, `#`  `%23`, `?`  `%3F`, `&`  `%26`)

**Replica Configuration (Optional):**
- `DB_REPLICA_URIS`: Comma-separated replica URIs
  - Example: `postgresql://user:pass@replica1:5432/db,postgresql://user:pass@replica2:5432/db`
- `DB_READ_FROM_MASTER`: Include master in read pool (`true`/`false`, default: `false`)
  - `false`: Read queries only use replicas (master only for writes)
  - `true`: Read queries use master + replicas (round-robin)

**MongoDB:**
- `MONGO_URI` (only when `DB_TYPE=mongodb`)
  - Format: `mongodb://user:password@host:port/database?authSource=admin`

**Connection Pool (Optional):**
- `DB_POOL_MIN_SIZE`: Minimum pool size (default: `2`)
- `DB_POOL_MAX_SIZE`: Maximum pool size (default: `100`)
- `DB_POOL_MASTER_RATIO`: Master pool ratio 0.0-1.0 (default: `0.6`)
  - Example: `DB_POOL_MAX_SIZE=100`, `DB_POOL_MASTER_RATIO=0.6`, 1 master + 2 replicas
  -  Master: 60 connections, Each replica: 20 connections
- `DB_ACQUIRE_TIMEOUT`: Connection acquisition timeout in ms (default: `60000`)
- `DB_IDLE_TIMEOUT`: Idle connection timeout in ms (default: `30000`)

### Redis (if not set  uses embedded)
- `REDIS_URI`: Redis connection string
  - Format: `redis://user:pass@host:port/db` or `redis://host:port/db`
- `DEFAULT_TTL`: Default TTL for cache entries in seconds (default: `5`)

### Server
- `PORT`: Server port (default: `1105`)
- `ENFYRA_SERVER_WORKERS`: Number of workers for cluster (default: `1`)
- `SECRET_KEY`: JWT secret key (default: `enfyra_secret_key_change_in_production`)
- `BACKEND_URL`: Backend URL for Swagger/docs (default: `http://localhost:1105`)
- `NODE_NAME`: Node instance name for logs/cluster (default: `enfyra_docker`)
- `DEFAULT_HANDLER_TIMEOUT`: Handler execution timeout in ms (default: `20000`)
- `DEFAULT_PREHOOK_TIMEOUT`: Prehook timeout in ms (default: `20000`)
- `DEFAULT_AFTERHOOK_TIMEOUT`: Afterhook timeout in ms (default: `20000`)

### Auth (Optional)
- `SALT_ROUNDS`: bcrypt salt rounds (default: `10`)
- `ACCESS_TOKEN_EXP`: Access token expiration (default: `15m`)
- `REFRESH_TOKEN_NO_REMEMBER_EXP`: Refresh token expiration without remember (default: `1d`)
- `REFRESH_TOKEN_REMEMBER_EXP`: Refresh token expiration with remember (default: `7d`)

### App
- `ENFYRA_APP_PORT`: App port (default: `3000`)
- `API_URL`: Backend API URL (automatically set if `ENFYRA_MODE=all`)

### Handler Executor (Optional)
- `HANDLER_EXECUTOR_MAX_MEMORY`: Max memory per child process in MB (default: `512`)
- `HANDLER_EXECUTOR_POOL_MIN`: Minimum pool size (default: `2`)
- `HANDLER_EXECUTOR_POOL_MAX`: Maximum pool size (default: `4`)

### Package Manager (Optional)
- `PACKAGE_MANAGER`: Package manager `yarn`, `npm`, or `pnpm` (default: `yarn`)

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
     -e DB_TYPE=postgres \
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
     -e DB_TYPE=mysql \
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
     -e DB_TYPE=postgres \
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
   # Or legacy vars:
   -e DB_HOST=your-db-host
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

