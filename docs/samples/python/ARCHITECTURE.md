# Architecture

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Python 3.11+ |
| Framework | FastAPI |
| Testing | pytest |
| Linting | ruff |
| Type checking | mypy (strict) |
| Formatting | ruff format |
| Package management | uv / pip |

## Module Structure

```
src/
├── api/             ← route handlers (FastAPI routers)
├── core/            ← config, settings, shared dependencies
├── models/          ← Pydantic models (request/response schemas)
├── services/        ← business logic
├── repositories/    ← data access layer
├── utils/           ← shared helpers
└── main.py          ← app entry point, middleware setup
tests/
├── conftest.py      ← shared fixtures
├── test_api/        ← route integration tests
├── test_services/   ← unit tests for business logic
└── test_repositories/ ← data layer tests
```

## Data Flow

```
Request → Middleware (auth, CORS) → Router → Service → Repository → Database
                                                                       ↓
Response ← Exception Handler ← Router ← Service ← Repository ← Result
```

## Key Dependencies

| Package | Purpose |
|---------|---------|
| pydantic | Data validation and serialization |
| sqlalchemy | Database ORM |
| alembic | Database migrations |
| httpx | Async HTTP client |
| structlog | Structured logging |

## API Routes

| Method | URL | Handler | Description |
|--------|-----|---------|-------------|
| GET | `/health` | `api/health.py` | Health check |
| GET | `/api/v1/users/{id}` | `api/users.py` | Get user by ID |
| POST | `/api/v1/users` | `api/users.py` | Create user |
