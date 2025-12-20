# Enfyra Installation Guide

## Prerequisites

- **Node.js** >= 20.0.0
- **Package manager** (npm ≥8.0.0, yarn ≥1.22.0, or bun ≥1.0.0)
- **Database server** (MySQL, MariaDB, PostgreSQL, or MongoDB)
- **Redis server**

**OR** use Docker for a complete all-in-one setup (recommended for quick start)

---

## Installation Methods

### Option 1: Docker (Recommended for Quick Start)

The easiest way to get started with Enfyra is using the all-in-one Docker image, which includes:
- Backend server (NestJS)
- Frontend app (Nuxt)
- Embedded PostgreSQL/MySQL (optional)
- Embedded Redis (optional)

#### Quick Start with Docker

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
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

**Expose embedded services (optional):**
If you need to connect to the embedded database or Redis from external tools:
```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -p 5432:5432 \  # PostgreSQL
  -p 6379:6379 \  # Redis
  -e DB_TYPE=postgres \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

#### Docker with MySQL

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_TYPE=mysql \
  -v enfyra-data:/app/data \
  dothinh115/enfyra:latest
```

#### Docker with External Database and Redis

```bash
docker run -d \
  --name enfyra \
  -p 3000:3000 \
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://enfyra:secret@my-postgres-host:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

> **Note**: If your password contains special characters, URL-encode them. Example: password `p@ssw0rd` → use `p%40ssw0rd` in the URI.

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
  -e DB_TYPE=postgres \
  -e DB_URI=postgresql://user:password@my-postgres:5432/enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

> **Note**: If your password contains special characters like `@`, `:`, `/`, etc., URL-encode them in the URI (e.g., `p@ssw0rd` → `p%40ssw0rd`).

Example - App only:
```bash
docker run -d \
  --name enfyra-app \
  -p 3000:3000 \
  -e ENFYRA_MODE=app \
  -e API_URL=http://your-backend:1105/ \
  dothinh115/enfyra:latest
```

For detailed Docker documentation, see:
- [Docker README](../docker/README.md) - Complete Docker setup guide with all configuration options
- [Docker Usage Guide](../docker/USAGE.md) - Detailed usage examples for all Docker modes

> **Next Steps**: After installation, see [Getting Started Guide](./getting-started.md) to learn how to create your first table and manage data.

---

### Option 2: Manual Installation

## Quick Setup (Manual Installation)

### 1. Install and run the backend

```bash
npx @enfyra/create-server <project-name>
cd <project-name>
npm run start
```
**Backend runs at http://localhost:1105** - This server generates and serves ALL API endpoints

- For detailed instructions: [@enfyra/create-server](https://www.npmjs.com/package/@enfyra/create-server)
- See [Architecture Overview](../architecture-overview.md) to understand how backend and frontend work together

### 2. Install and run the frontend app
```bash
npx @enfyra/create-app <project-name>
cd <project-name>
npm run dev
```
**Frontend runs at http://localhost:3000** - This app consumes APIs from your backend URL

- For detailed instructions: [@enfyra/create-app](https://www.npmjs.com/package/@enfyra/create-app)
- See [Architecture Overview](../architecture-overview.md) to understand the backend-first architecture

## Connection Flow

**Important**: The frontend app is a client that connects to your backend server:

```
Database  Backend APIs (1105) ← Frontend App (3000)
```

1. **Backend** generates REST & GraphQL APIs from your database schema
2. **Frontend** connects to backend URL (`BACKEND_URL`) and makes HTTP requests
3. **All data operations** flow through: Frontend  Backend  Database

**No API exists on the frontend** - it's purely a client consuming backend APIs.

> **Learn More**: 
> - [Architecture Overview](../architecture-overview.md) - Understand the system architecture
> - [Getting Started Guide](./getting-started.md) - Next steps after installation
> - [Table Creation Guide](./table-creation.md) - Create your first table

## Configuration Prompts

## 3. Backend Configuration prompts

When you run:

```bash
npx @enfyra/create-server <project-name>
```

the CLI will ask you a series of configuration questions. Enter the values that match your environment.

| Prompt                                | Description                                                         |
| ------------------------------------- | ------------------------------------------------------------------- |
| **Package manager**                   | Select the package manager you want to use (`npm`, `yarn`, `bun`)   |
| **Project name**                      | Name of the backend project (if not passed as a CLI argument)       |
| **Database type**                     | Type of database (`MySQL`, `PostgreSQL`, `MariaDB`, `MongoDB`)      |
| **Database host**                     | Hostname or IP address of your database                             |
| **Database port**                     | Port number of your database (e.g. `3306` for MySQL)                |
| **Database username**                 | Database user account                                               |
| **Database password**                 | Database password (can be left empty)                               |
| **Database name**                     | Name of the database to connect to / create                         |
| **Configure database pool settings?** | Whether to configure advanced connection-pool settings (Yes/No)     |
| **Redis URI**                         | URI of your Redis instance (e.g. `redis://user:pass@host:port`)     |
| **Application port**                  | Port where the Enfyra backend will run (default `1105`)             |
| **Admin email**                       | Initial admin email used to log into the dashboard                  |
| **Admin password**                    | Initial admin password                                              |

>  If any of the database or Redis connection details are invalid, the CLI will prompt you to re-enter them or cancel setup.

> **Note**: The CLI will generate a `DB_URI` connection string in your `.env` file (e.g., `mysql://user:pass@host:port/database`). You can also manually set `DB_URI` instead of using separate host/port/username/password/name fields.

> **Important - Password with Special Characters**: If your database password contains special characters (such as `@`, `:`, `/`, `%`, `#`, `?`, `&`), you must URL-encode them in the `DB_URI`. For example:
> - Password `p@ssw0rd` → Use `p%40ssw0rd` in URI
> - Password `pass:word` → Use `pass%3Aword` in URI
> - Common encodings: `@` = `%40`, `:` = `%3A`, `/` = `%2F`, `%` = `%25`, `#` = `%23`, `?` = `%3F`, `&` = `%26`

> **Database Replication (Optional)**: To enable read replicas, add `DB_REPLICA_URIS` (comma-separated) to your `.env`. Connection pool is automatically distributed between master and replicas. Read queries use round-robin routing across replicas. Set `DB_READ_FROM_MASTER=true` to include master in the round-robin pool for reads.

**The backend will run at http://localhost:1105 by default.**

After you finish answering the prompts, the CLI will:

* Validate database and Redis connections.
* Generate an `.env` file with your answers (using `DB_URI` format).
* Ask for confirmation before scaffolding the project.

Then run your new backend:

```bash
cd <project-name>
npm run start
```
(or use `yarn start:dev` / `bun run start:dev` depending on the package manager you selected.)

> **Next Steps**: 
> - [Getting Started Guide](./getting-started.md) - Learn the interface and create your first table
> - [Table Creation Guide](./table-creation.md) - Complete guide to creating tables with all field types
> - [Server Documentation](../server/README.md) - Advanced backend configuration and API development

## 4. Frontend Configuration Prompts

When you run:

```bash
npx @enfyra/create-app <project-name>
```

the CLI will ask you a series of configuration questions for the frontend application. Enter the values that match your environment.

| Prompt                                | Description                                                         |
| ------------------------------------- | ------------------------------------------------------------------- |
| **Package manager**                   | Select the package manager you want to use (`npm`, `yarn`, `pnpm`, `bun`) |
| **API base URL**                      | **CRITICAL**: Base URL of your backend API server that generates all APIs (must include `http://` or `https://`) |
| **App port**                          | Port where the Enfyra frontend will run (default `3000`)           |

>  **Important**: The API base URL must point to your **backend server** (usually `http://localhost:1105`). The frontend app will make HTTP requests to this URL to consume APIs. All REST & GraphQL endpoints are served by your backend, not the frontend.

After you finish answering the prompts, the CLI will:

* Validate the API URL format.
* Generate an `.env` file with your configuration.
* Scaffold the frontend project with your selected package manager.

Then run your new frontend:

```bash
cd <project-name>
npm run dev
```

(or use `yarn dev` / `pnpm dev` / `bun dev` depending on the package manager you selected.)

> **Next Steps**: 
> - [Getting Started Guide](./getting-started.md) - Learn the interface and create your first table
> - [Table Creation Guide](./table-creation.md) - Complete guide to creating tables with all field types
> - [Data Management Guide](./data-management.md) - Learn to manage records and relationships
> - [App Documentation](../app/README.md) - Frontend features and customization guides