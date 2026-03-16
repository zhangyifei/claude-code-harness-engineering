# Architecture

## Tech Stack

| Layer | Choice |
|-------|--------|
| Runtime | Node.js 20+ |
| Language | TypeScript (strict) |
| Framework | Express / Fastify |
| Testing | Vitest |
| Linting | ESLint |
| Formatting | Prettier |

## Module Structure

```
src/
├── routes/          ← HTTP route handlers
├── services/        ← business logic
├── models/          ← data types and validation (Zod schemas)
├── middleware/       ← auth, logging, error handling
├── utils/           ← shared helpers
└── index.ts         ← entry point, server setup
```

## Data Flow

```
Request → Middleware (auth, validation) → Route Handler → Service → Database
                                                                      ↓
Response ← Middleware (error handling) ← Route Handler ← Service ← Result
```

## Key Dependencies

| Package | Purpose |
|---------|---------|
| zod | Runtime input validation |
| pino | Structured logging |
| prisma / drizzle | Database ORM |

## Routing

| Method | URL | Handler | Description |
|--------|-----|---------|-------------|
| GET | `/api/health` | `routes/health.ts` | Health check |
| GET | `/api/users/:id` | `routes/users.ts` | Get user by ID |
| POST | `/api/users` | `routes/users.ts` | Create user |
