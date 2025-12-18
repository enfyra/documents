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
  -e DB_HOST=my-postgres-host \
  -e DB_PORT=5432 \
  -e DB_USERNAME=enfyra \
  -e DB_PASSWORD=secret \
  -e DB_NAME=enfyra \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

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
  -e DB_HOST=my-postgres \
  -e REDIS_URI=redis://my-redis:6379/0 \
  dothinh115/enfyra:latest
```

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
- [Docker README](../../docker/README.md)
- [Docker Usage Guide](../../docker/USAGE.md)

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

### 2. Install and run the frontend app
```bash
npx @enfyra/create-app <project-name>
cd <project-name>
npm run dev
```
**Frontend runs at http://localhost:3000** - This app consumes APIs from your backend URL

- For detailed instructions: [@enfyra/create-app](https://www.npmjs.com/package/@enfyra/create-app)

## Connection Flow

**Important**: The frontend app is a client that connects to your backend server:

```
Database  Backend APIs (1105) ← Frontend App (3000)
```

1. **Backend** generates REST & GraphQL APIs from your database schema
2. **Frontend** connects to backend URL (`BACKEND_URL`) and makes HTTP requests
3. **All data operations** flow through: Frontend  Backend  Database

**No API exists on the frontend** - it's purely a client consuming backend APIs.

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

**The backend will run at http://localhost:1105 by default.**

After you finish answering the prompts, the CLI will:

* Validate database and Redis connections.
* Generate an `.env` file with your answers.
* Ask for confirmation before scaffolding the project.

Then run your new backend:

```bash
cd <project-name>
npm run start
```
(or use `yarn start:dev` / `bun run start:dev` depending on the package manager you selected.)

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