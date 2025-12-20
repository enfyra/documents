# Enfyra Docker

All-in-one Docker image for Enfyra platform, including Server + App + Embedded Redis + Embedded Database.

> **New to Enfyra?** See the [Installation Guide](../getting-started/installation.md) for complete setup instructions, including Docker and manual installation options.

## 🚀 Quick Start

### Simplest way (single command)

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  dothinh115/enfyra:latest
```

→ Runs with embedded PostgreSQL and Redis, no additional configuration needed!

### With MySQL

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=mysql \
  dothinh115/enfyra:latest
```

### With MongoDB (requires MONGO_URI)

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=mongodb \
  -e MONGO_URI=mongodb://user:pass@host:27017/dbname \
  dothinh115/enfyra:latest
```

## 🧹 Clean Up Old Images

To remove old Docker images and free up disk space:

```bash
cd docker

# List all Enfyra images
./cleanup-images.sh --list

# Keep only 5 latest images (default)
./cleanup-images.sh --keep 5

# Keep only 3 latest images
./cleanup-images.sh --keep 3

# Remove only dangling images
./cleanup-images.sh --dangling

# Remove ALL Enfyra images (⚠️ careful!)
./cleanup-images.sh --all
```

## 📦 Build and Push Image

### Build and push with automatic version (from server/package.json)

```bash
cd docker
./build-and-push.sh
```

### Build and push with specific version

```bash
cd docker
./build-and-push.sh 1.2.5
```

### Use environment variable to override namespace

```bash
DOCKER_NAMESPACE=enfyra ./build-and-push.sh
```

## ⚙️ Configuration

### Environment Variables

#### Required
- `DB_TYPE`: Database type (`postgres`, `mysql`, `mongodb`) - **default: `postgres`**

#### Database Configuration

**Primary (Prisma-style URI - Recommended):**
- `DB_URI`: Database connection URI (if not set → uses embedded DB)
  - PostgreSQL: `postgresql://user:password@host:port/database`
  - MySQL: `mysql://user:password@host:port/database`
  - Example: `postgresql://enfyra:secret@my-postgres:5432/enfyra`
  - **Note**: If password contains special characters (`@`, `:`, `/`, etc.), URL-encode them:
    - `@` → `%40`, `:` → `%3A`, `/` → `%2F`, `%` → `%25`, `#` → `%23`, `?` → `%3F`, `&` → `%26`

**Replica Configuration (Optional):**
- `DB_REPLICA_URIS`: Comma-separated replica URIs (e.g., `postgresql://user:pass@replica1:5432/db,postgresql://user:pass@replica2:5432/db`)
- `DB_READ_FROM_MASTER`: Include master in read pool (`true`/`false`, default: `false`)

**Legacy (Backward Compatible):**
- `DB_HOST`: Database host (if `DB_URI` not set)
- `DB_PORT`: Database port
- `DB_USERNAME`: Database username
- `DB_PASSWORD`: Database password
- `DB_NAME`: Database name

**MongoDB:**
- `MONGO_URI`: MongoDB connection string (only when `DB_TYPE=mongodb`)

#### Redis Configuration
- `REDIS_URI`: Redis connection string (if not set → uses embedded Redis)

#### Mode Configuration
- `ENFYRA_MODE`: Run mode (`all`, `server`, `app`) - **default: `all`**
  - `all`: Run both server + app (default)
  - `server`: Run backend server only
  - `app`: Run frontend app only

#### Database Connection Pool (Optional)
- `DB_POOL_MIN_SIZE`: Minimum connection pool size - **default: `2`**
- `DB_POOL_MAX_SIZE`: Maximum connection pool size - **default: `100`**
- `DB_POOL_MASTER_RATIO`: Master pool ratio (0.0-1.0) - **default: `0.6`** (60% master, 40% replicas)
- `DB_ACQUIRE_TIMEOUT`: Connection acquisition timeout (ms) - **default: `60000`**
- `DB_IDLE_TIMEOUT`: Idle connection timeout (ms) - **default: `30000`**

#### Server Configuration
- `PORT`: Server port - **default: `1105`**
- `ENFYRA_SERVER_WORKERS`: Number of worker processes for cluster - **default: `1`**
- `SECRET_KEY`: JWT secret key - **default: `enfyra_secret_key_change_in_production`**
- `BACKEND_URL`: Backend URL (for Swagger/docs) - **default: `http://localhost:1105`**
- `NODE_NAME`: Node instance name (for logs/cluster) - **default: `enfyra_docker`**
- `DEFAULT_HANDLER_TIMEOUT`: Handler execution timeout (ms) - **default: `20000`**
- `DEFAULT_PREHOOK_TIMEOUT`: Prehook timeout (ms) - **default: `20000`**
- `DEFAULT_AFTERHOOK_TIMEOUT`: Afterhook timeout (ms) - **default: `20000`**

#### Auth Configuration (Optional)
- `SALT_ROUNDS`: bcrypt salt rounds - **default: `10`**
- `ACCESS_TOKEN_EXP`: Access token expiration - **default: `15m`**
- `REFRESH_TOKEN_NO_REMEMBER_EXP`: Refresh token expiration (no remember) - **default: `1d`**
- `REFRESH_TOKEN_REMEMBER_EXP`: Refresh token expiration (remember) - **default: `7d`**

#### Redis Configuration
- `REDIS_URI`: Redis connection string (if not set → uses embedded Redis)
  - Format: `redis://user:pass@host:port/db` or `redis://host:port/db`
- `DEFAULT_TTL`: Default TTL for cache entries (seconds) - **default: `5`**

#### App Configuration
- `ENFYRA_APP_PORT`: App port - **default: `3000`**
- `API_URL`: Backend API URL (automatically set if `ENFYRA_MODE=all`)

#### Handler Executor (Optional)
- `HANDLER_EXECUTOR_MAX_MEMORY`: Max memory per child process (MB) - **default: `512`**
- `HANDLER_EXECUTOR_POOL_MIN`: Minimum pool size - **default: `2`**
- `HANDLER_EXECUTOR_POOL_MAX`: Maximum pool size - **default: `4`**

#### Package Manager (Optional)
- `PACKAGE_MANAGER`: Package manager (`yarn`, `npm`, `pnpm`) - **default: `yarn`**

#### Environment
- `NODE_ENV`: Environment (`development`, `production`, `test`) - **default: `production`**

### Usage Examples

#### 1. All-in-one with embedded services (simplest)

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

#### 2. Use external database and Redis (with DB_URI - Recommended)

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

**Or with legacy vars (backward compatible):**

```bash
docker run -d \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -e DB_HOST=my-postgres \
  -e DB_PORT=5432 \
  -e DB_USERNAME=enfyra \
  -e DB_PASSWORD=secret \
  -e DB_NAME=enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

#### 3. Run server only (for cluster)

```bash
docker run -d \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  -e ENFYRA_SERVER_WORKERS=4 \
  dothinh115/enfyra:latest
```

**With replica databases:**

```bash
docker run -d \
  -p 1105:1105 \
  -e ENFYRA_MODE=server \
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@master-postgres:5432/enfyra \
  -e DB_REPLICA_URIS=postgresql://enfyra:secret@replica1:5432/enfyra,postgresql://enfyra:secret@replica2:5432/enfyra \
  -e DB_READ_FROM_MASTER=false \
  -e REDIS_URI=redis://my-redis:6379/0 \
  -e ENFYRA_SERVER_WORKERS=4 \
  dothinh115/enfyra:latest
```

#### 4. Run app only (frontend only)

```bash
docker run -d \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=https://api.my-domain.com/ \
  dothinh115/enfyra:latest
```

## 🏗️ Architecture

This image contains:
- **Server**: NestJS backend (port 1105)
- **App**: Nuxt frontend (port 3000)
- **Redis**: Embedded Redis server (port 6379, internal - expose with `-p 6379:6379` if needed)
- **PostgreSQL**: Embedded database (port 5432, internal - expose with `-p 5432:5432` if needed)
- **MySQL**: Embedded database (port 3306, internal - expose with `-p 3306:3306` if needed)

All managed by **supervisor** in a single container.

### Default Credentials

When using embedded database, default credentials:
- **PostgreSQL/MySQL**:
  - Username: `enfyra`
  - Password: `enfyra_password_123`
  - Database: `enfyra`
- **Admin User** (auto-created):
  - Email: `enfyra@admin.com`
  - Password: `1234`

## 📝 Notes

- **Data persistence**: Use volume to persist embedded DB/Redis data:
  ```bash
  -v enfyra-data:/app/data
  ```

- **Production**: Recommended to use external database and Redis for production for better HA and backup.

- **Cluster**: Set `ENFYRA_SERVER_WORKERS > 1` to run backend cluster in container, or scale multiple containers with `ENFYRA_MODE=server`.

- **Environment Files**: `.env` files are automatically generated for both server and app based on `env_example` files. All env vars have reasonable defaults, but you can override by setting env vars when `docker run`.

- **Embedded Ports**: Embedded services use default ports. Expose them when running if you need external access:
  ```bash
  -p 5432:5432  # PostgreSQL
  -p 3306:3306  # MySQL
  -p 6379:6379  # Redis
  ```
  
  **Note**: If you already have these services running on your host, either:
  - Use external services (set `DB_URI` or `DB_HOST`, `REDIS_URI`)
  - Use different host ports: `-p 5433:5432` (maps host 5433 to container 5432)

## 🔧 Build Requirements

- Docker installed and running
- Logged in to Docker Hub: `docker login`
- Access to `server/` and `app/` directories
- Stable internet connection (for pulling base images)

### Troubleshooting Build Issues

**Network timeout when pulling base images:**
```bash
# Pre-pull base image to avoid timeout
docker pull node:20-alpine

# Then retry build
./build-and-push.sh
```

**If build still fails:**
- Check your internet connection
- Try again later (Docker Hub might be slow)
- The build script has automatic retry (3 attempts)


