# Enfyra Installation Guide

## Prerequisites

- **Node.js** >= 24.0.0 for manual installation
- **Package manager** (npm, yarn, or pnpm)
- **Database server** (MySQL, PostgreSQL, or MongoDB — MariaDB works via the `mysql://` protocol)
- **Redis server**

**OR** use Docker for a complete all-in-one setup (recommended for quick start)

---

## Installation Methods

### Option 0: Enfyra Cloud (Managed Hosting)

If you do not want to run your own server, database, Redis, TLS, reverse proxy, or deployment pipeline, use Enfyra Cloud instead of a self-hosted installation.

Cloud creates hosted Enfyra projects from the Cloud dashboard. Each project has its own Enfyra runtime boundary, while shared database, edge, and supporting services are capacity-managed with headroom and guardrails. Higher plans reserve more operating room per project.

Use Cloud when you want to start building in Enfyra without managing infrastructure. Use the Docker or manual installation paths below when you need to own the runtime environment yourself.

See [Enfyra Cloud](../cloud/README.md) for the managed hosting model, project creation flow, checkout behavior, and how Cloud differs from self-hosting.

### Option 1: Docker (Recommended for Quick Start)

The easiest way to get started with Enfyra is using the all-in-one Docker image, which includes:
- Backend server (Awilix + Express 5)
- Frontend app (Nuxt)
- Embedded PostgreSQL/MySQL (optional)
- Embedded Redis (optional)

#### Quick Start with Docker

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

This single command will:
- Start the backend server (port 1105, internal)
- Start the frontend app (port 3000, exposed)
- Start embedded PostgreSQL (if no DB config provided)
- Start embedded Redis (if no REDIS_URI provided)

**Access the application:**
- Frontend: http://localhost:3000
- Backend API: http://localhost:1105

**Default credentials:**
- Admin Email: `enfyra@admin.com`
- Admin Password: `1234`

**Customize admin credentials:**
```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e ADMIN_EMAIL=myadmin@example.com \
  -e ADMIN_PASSWORD=secure_password_123 \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

**Expose embedded services (optional):**
If you need to connect to the embedded database or Redis from external tools:
```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -p 5432:5432 \  # PostgreSQL
  -p 6379:6379 \  # Redis
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

#### Docker with MySQL (Embedded)

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e EMBEDDED_DB=mysql \
  -v enfyra-data:/app/data \
  enfyra/enfyra:latest
```

#### Docker with External Database and Redis

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

If your database and Redis are running on your local machine, do not use `localhost` in `DB_URI` or `REDIS_URI`. Inside the container, `localhost` means the container itself. Use `host.docker.internal` to connect back to the host:

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

On Linux, add the host gateway mapping if your Docker version does not provide `host.docker.internal` automatically:

```bash
docker run -d \
  --name enfyra \
  --add-host=host.docker.internal:host-gateway \
  -p 3000:3000 \
  -e DB_URI=postgresql://enfyra:secret@host.docker.internal:5432/enfyra \
  -e REDIS_URI=redis://host.docker.internal:6379/0 \
  enfyra/enfyra:latest
```

Make sure your local database and Redis accept connections from Docker, not only from `127.0.0.1`.

> **Note**: If your password contains special characters, URL-encode them. Example: password `p@ssw0rd`  use `p%40ssw0rd` in the URI.

`REDIS_RUNTIME_CACHE=true` stores Enfyra runtime definition snapshots in Redis so instances with the same `NODE_NAME` share the same runtime cache namespace. `$cache` / `@CACHE` user data is separately limited by `REDIS_USER_CACHE_LIMIT_MB`, which defaults to `30` MB.

#### Docker Modes

The Docker image supports 3 modes:

1. **`all`** (default) - Run both server + app + embedded services
2. **`server`** - Run backend server only
3. **`app`** - Run frontend app only

Example - Server only:
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

> **Note**: If your password contains special characters like `@`, `:`, `/`, etc., URL-encode them in the URI (e.g., `p@ssw0rd`  `p%40ssw0rd`).

Example - App only:
```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://your-backend:1105/ \
  enfyra/enfyra:latest
```

For detailed Docker documentation, see:
- [Docker](../docker/README.md) - Complete Docker setup guide with all configuration options

> **Next Steps**: After installation, see [Getting Started Guide](./getting-started.md) to learn how to create your first table and manage data.

---

### Option 2: Manual Installation

#### Quick Setup (Manual Installation)

Use the Enfyra create CLI to scaffold one workspace that contains both the backend server and the Nuxt admin app:

```bash
npx @enfyra/create <project-name>
cd <project-name>
npm run dev
```

The generated workspace has this structure:

```text
<project-name>/
|-- app/
|   `-- .env
|-- server/
|   `-- .env
|-- scripts/
`-- package.json
```

The `app` and `server` folders are separate applications inside one project folder. Each folder has its own dependencies and lockfile; the root package only provides convenience scripts.

**Server:** http://localhost:1105  
**App:** http://localhost:3000

The server port is fixed to `1105`. During setup, the CLI checks that port first and asks before killing any process already using it. The Nuxt app is configured with `3000`; if the port is busy during development, Nuxt can select another available port.

- For package details: [@enfyra/create](https://www.npmjs.com/package/@enfyra/create)
- See [Architecture Overview](../architecture-overview.md) to understand how the backend and frontend work together

#### Connection Flow

The admin app is a client that connects to your backend server:

```text
Database -> Backend APIs (1105) <- Frontend App (3000)
```

1. The backend generates REST APIs and the enabled GraphQL schema from your database metadata.
2. The frontend uses the generated `app/.env` API URL to call the backend.
3. All data operations flow through: Frontend -> Backend -> Database.

The frontend does not own the generated API. It consumes the backend API.

> **Learn More**:
> - [Architecture Overview](../architecture-overview.md) - Understand the system architecture
> - [Getting Started Guide](./getting-started.md) - Next steps after installation
> - [Table Creation Guide](./table-creation.md) - Create your first table

#### Configuration Prompts

When you run:

```bash
npx @enfyra/create <project-name>
```

the CLI asks for the settings needed to create both applications.

| Prompt | Description |
| ------ | ----------- |
| **Package manager** | Select the package manager to use for both `app` and `server` (`npm`, `yarn`, or `pnpm`). |
| **Project name** | Name of the generated workspace, if not passed as a CLI argument. |
| **Database type** | Type of database (`MySQL`, `PostgreSQL`, or `MongoDB`). MariaDB users select `MySQL`. |
| **Database URI** | SQL connection URI, such as `mysql://user:password@localhost:3306/enfyra` or `postgresql://postgres:password@localhost:5432/enfyra`. |
| **MongoDB URI** | MongoDB connection URI when MongoDB is selected. |
| **Setup read replica?** | Optional SQL read replica configuration. |
| **Replica URI** | SQL replica connection URI when read replicas are enabled. |
| **Redis URI** | URI of your Redis instance, such as `redis://localhost:6379`. |
| **Admin email** | Initial admin email used to log into the dashboard. |
| **Admin password** | Initial admin password. |

The CLI validates the database and Redis connections before creating the project. If a connection fails, it asks whether to re-enter the connection details, continue anyway, or exit setup.

After you confirm creation, the CLI will:

* Download the Enfyra app and server.
* Generate `app/.env` and `server/.env`.
* Connect the app to the local server URL.
* Install dependencies separately inside `server/` and `app/`.
* Generate root convenience scripts for local development and self-hosting.

> **Important - `SECRET_KEY`**: The CLI generates a random `SECRET_KEY` in `server/.env`. Keep this value stable and backed up for production. It signs auth tokens and derives the encryption key for columns marked `isEncrypted=true`; changing or losing it invalidates existing tokens and prevents Enfyra from decrypting stored encrypted field values. All server instances for the same Enfyra app must use the same `SECRET_KEY`.

> **Important - Password with Special Characters**: If your database password contains special characters (such as `@`, `:`, `/`, `%`, `#`, `?`, `&`), URL-encode them in the database URI. For example:
> - Password `p@ssw0rd` -> Use `p%40ssw0rd` in URI
> - Password `pass:word` -> Use `pass%3Aword` in URI
> - Common encodings: `@` = `%40`, `:` = `%3A`, `/` = `%2F`, `%` = `%25`, `#` = `%23`, `?` = `%3F`, `&` = `%26`

> **Database replication (optional, SQL only)**  
> Add replica URI values during setup so read queries can round-robin across healthy replicas. Writes still use the main database URI.

#### Running The Workspace

From the workspace root:

```bash
npm run dev
```

`npm run dev` starts the server first, waits for port `1105` to accept connections, and then starts the app. Use the matching package manager command if you selected Yarn or pnpm:

```bash
yarn dev
pnpm run dev
```

You can also run one side at a time:

```bash
npm run dev:server
npm run dev:app
```

#### Building And Starting

Build both applications from the workspace root:

```bash
npm run build
```

The root build script runs the server build first, then the app build. To run the production outputs:

```bash
npm run start
```

The start script also starts the server first and then the app after the server port is ready.

> **Next Steps**:
> - [Getting Started Guide](./getting-started.md) - Learn the interface and create your first table
> - [Table Creation Guide](./table-creation.md) - Complete guide to creating tables with all field types
> - [Data Management Guide](./data-management.md) - Learn to manage records and relationships
> - [App Documentation](../app/README.md) - Frontend features and customization guides
> - [Server Documentation](../server/README.md) - Advanced backend configuration and API development
