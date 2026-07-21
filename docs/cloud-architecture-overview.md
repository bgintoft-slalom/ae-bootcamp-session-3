# Cloud Architecture Overview

## System Context

The application is a monorepo consisting of a React frontend and an Express.js backend API, backed by an in-memory data store. There is no external database or cloud storage in the current architecture.

```mermaid
graph TD
    User["User<br/>(Browser)"]
    Frontend["React Frontend<br/>packages/frontend"]
    Backend["Express API<br/>packages/backend"]
    Store["In-Memory Store<br/>(better-sqlite3, ':memory:')"]

    User -->|"HTTP (UI interactions)"| Frontend
    Frontend -->|"REST calls: /api/tasks"| Backend
    Backend -->|"SQL queries"| Store
```

## Components

- **React Frontend** (`packages/frontend`): Renders the TODO UI and calls the backend REST API for all task operations (create, read, update, delete).
- **Express API** (`packages/backend`): Exposes `/api/tasks` endpoints and handles request validation, filtering, and sorting logic.
- **In-Memory Store**: A `better-sqlite3` database instantiated with `:memory:`, scoped to the lifetime of the running backend process. Data does not persist across server restarts and no external database or cloud service is used.

## Sequence: Creating a TODO

```mermaid
sequenceDiagram
    actor User
    participant Frontend as React Frontend<br/>(TaskForm.js)
    participant Backend as Express API<br/>(POST /api/tasks)
    participant Store as In-Memory Store<br/>(better-sqlite3)

    User->>Frontend: Fill in title/description/due date & submit
    Frontend->>Frontend: Validate title is non-empty
    Frontend->>Backend: POST /api/tasks (title, description, due_date)
    Backend->>Backend: Validate title is required
    Backend->>Store: INSERT INTO tasks (...)
    Store-->>Backend: New task row (id, created_at, ...)
    Backend-->>Frontend: 201 Created (new task JSON)
    Frontend->>Backend: GET /api/tasks (refresh list)
    Backend->>Store: SELECT * FROM tasks
    Store-->>Backend: Task rows
    Backend-->>Frontend: 200 OK (task list)
    Frontend-->>User: Show updated task list
```
