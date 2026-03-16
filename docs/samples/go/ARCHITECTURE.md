# Architecture

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Go 1.22+ |
| Framework | stdlib `net/http` + chi router |
| Testing | stdlib `testing` + testify |
| Linting | golangci-lint |
| Database | sqlx (raw SQL + struct scanning) |
| Config | envconfig / viper |

## Package Layout

```
cmd/
└── server/
    └── main.go          ← entry point, wires dependencies
internal/
├── handler/             ← HTTP handlers (accept request, return response)
├── service/             ← business logic (pure, no HTTP awareness)
├── repository/          ← data access (database queries)
├── model/               ← domain types and value objects
├── middleware/           ← auth, logging, recovery
└── config/              ← environment-based configuration
pkg/                     ← public library code (if any)
migrations/              ← SQL migration files
docs/                    ← architecture, conventions, quality
```

## Data Flow

```
Request → Middleware (auth, logging, recovery) → Handler → Service → Repository → DB
                                                                                  ↓
Response ← Middleware (response logging) ← Handler ← Service ← Repository ← Rows
```

## Dependency Injection

Constructor-based injection, no globals:

```go
func NewUserService(repo UserRepository, logger *slog.Logger) *UserService
func NewUserHandler(svc UserService) *UserHandler
```

All dependencies wired in `cmd/server/main.go`.

## Key Dependencies

| Package | Purpose |
|---------|---------|
| chi | Lightweight HTTP router |
| sqlx | SQL query builder + struct mapping |
| slog | Structured logging (stdlib) |
| testify | Test assertions and mocks |
| golang-migrate | Database migrations |

## API Routes

| Method | URL | Handler | Description |
|--------|-----|---------|-------------|
| GET | `/healthz` | `handler/health.go` | Health/readiness check |
| GET | `/api/v1/users/{id}` | `handler/user.go` | Get user by ID |
| POST | `/api/v1/users` | `handler/user.go` | Create user |
