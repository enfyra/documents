# Architecture Overview

This diagram shows how Enfyra's two-component system works.

> **New to Enfyra?** Start with the [Installation Guide](./getting-started/installation.md) to set up your backend and frontend.

## System Architecture Diagram

```
┌─────────────────────┐    HTTP Requests    ┌─────────────────────┐
│                     │ ────────────────► │                     │
│   Frontend App      │                    │   Backend Server    │
│   (Port 3000)       │ ◄──────────────── │   (Port 1105)       │
│                     │    JSON Response   │                     │
└─────────────────────┘                    └─────────────────────┘
         │                                           │
         │                                           │
         │                                           ▼
         │                                  ┌─────────────────────┐
         │                                  │                     │
         │                                  │      Database       │
         │                                  │  (MySQL/PostgreSQL/ │
         │                                  │  MariaDB/MongoDB)   │
         │                                  └─────────────────────┘
         │                                           │
         │                                           │
         ▼                                           ▼
┌─────────────────────┐                    ┌─────────────────────┐
│                     │                    │                     │
│   Admin Interface   │                    │   REST & GraphQL    │
│   Forms, Tables,    │                    │   API Generation    │
│   Data Management   │                    │   Custom Handlers   │
│                     │                    │   Hooks, Permissions │
└─────────────────────┘                    └─────────────────────┘
```

## Component Responsibilities

### Backend Server (Port 1105)
- **Database Management**: Direct connection to MySQL/PostgreSQL/MariaDB/MongoDB
- **API Generation**: Automatically creates REST & GraphQL APIs from database schema
- **Business Logic**: Custom handlers, hooks, and validation
- **Security**: Authentication, authorization, and permissions
- **Schema Management**: Table creation, relationships, and migrations

### Frontend App (Port 3000)
- **User Interface**: Admin panel, forms, tables, dashboards
- **API Consumer**: Makes HTTP requests to backend endpoints
- **State Management**: Handles UI state, caching, and user sessions
- **Extensions**: Custom Vue components and pages
- **No Database Access**: Never directly connects to database

## Data Flow

1. **User Action**: User interacts with frontend admin interface
2. **HTTP Request**: Frontend makes API call to backend server
3. **Processing**: Backend processes request through hooks/handlers
4. **Database Operation**: Backend performs database operation
5. **Response**: Backend returns JSON response to frontend
6. **UI Update**: Frontend updates interface with new data

## Key Points

- **All APIs originate from backend**: Frontend is purely a client
- **Single Database Connection**: Only backend connects to database
- **Stateless Frontend**: No server-side logic in frontend
- **API-First Design**: Backend serves APIs, frontend consumes them
- **Independent Deployment**: Backend and frontend can be deployed separately

## Development Workflow

```
1. Create/Modify Tables (Backend Admin Interface)
2.  Backend generates API endpoints automatically
3.  Frontend consumes APIs via HTTP requests
4.  Zero additional configuration needed
```

## Production Deployment

```
Backend Server (1105)  API Endpoints  Frontend App (3000)  User Browser
       ↓
   Database Server
       ↓
   Redis (Synchronization)
```

## Related Documentation

- **Server Documentation**: `./server/README.md` – Server architecture, APIs, repositories, and context object
- **App Documentation**: `./app/README.md` – Frontend/admin app behavior, extensions, forms, and permissions
