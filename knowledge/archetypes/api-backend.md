# Archetype: API / Backend Service

## Default Stack Recommendation

| Layer | Default | Alternative | When to Switch |
|-------|---------|-------------|----------------|
| Runtime | Node.js (Hono) | Python (FastAPI), Go | Python for ML/data, Go for high-performance |
| Language | TypeScript | Python, Go | Based on team/domain |
| Framework | Hono | Express, Fastify, NestJS | Express for simplicity, NestJS for enterprise |
| Database | PostgreSQL (Neon/Supabase) | MongoDB, SQLite | Mongo for document-heavy, SQLite for embedded |
| ORM | Drizzle | Prisma, TypeORM | Prisma for DX, TypeORM for NestJS |
| Validation | Zod | Joi, class-validator | class-validator with NestJS |
| Auth | JWT (jose library) | Passport.js, API keys only | Passport for OAuth, API keys for B2B |
| Queue | BullMQ + Redis | SQS, Inngest | SQS for AWS, Inngest for serverless |
| Caching | Redis (Upstash) | In-memory LRU | In-memory for single instance |
| Docs | OpenAPI/Swagger | — | Always generate API docs |
| Testing | Vitest | Jest | — |
| Hosting | Railway | Fly.io, AWS ECS, Vercel (Edge) | Fly for global, AWS for enterprise |
| Package Manager | pnpm | npm | — |

## Default Directory Structure

```
src/
  routes/
    auth/
      index.ts               # POST /auth/login, /auth/register, /auth/refresh
      middleware.ts           # Auth middleware
    users/
      index.ts               # GET/PATCH /users/:id
    [feature]/
      index.ts               # CRUD routes for feature
      schema.ts              # Zod validation schemas
      service.ts             # Business logic
    index.ts                 # Route aggregator
  middleware/
    auth.ts                  # JWT verification
    rateLimit.ts             # Rate limiting
    errorHandler.ts          # Global error handler
    logger.ts                # Request logging
  db/
    schema.ts                # Drizzle schema definitions
    migrations/              # SQL migrations
    index.ts                 # Database client
    seed.ts                  # Seed data
  services/
    email.ts                 # Email sending (Resend)
    queue.ts                 # Job queue (BullMQ)
    cache.ts                 # Cache layer (Redis)
  lib/
    env.ts                   # Environment variable validation (zod)
    errors.ts                # Custom error classes
    utils.ts                 # Shared utilities
  types/
    index.ts                 # Shared types
  index.ts                   # App entry point
tests/
  routes/                    # Route integration tests
  services/                  # Service unit tests
  helpers/                   # Test utilities
drizzle.config.ts            # Drizzle Kit config
```

## Common Patterns

### Route Structure (Hono)
```typescript
// Each route file exports a Hono app
const app = new Hono()
  .get('/', listHandler)
  .get('/:id', getHandler)
  .post('/', createHandler)
  .patch('/:id', updateHandler)
  .delete('/:id', deleteHandler)
export default app
```

### Error Handling
- Custom error classes: `NotFoundError`, `ValidationError`, `UnauthorizedError`
- Global error middleware catches all, returns consistent JSON shape:
  `{ success: false, error: { code, message, details? } }`

### API Response Shape
```json
{
  "success": true,
  "data": { ... },
  "meta": { "page": 1, "total": 100 }
}
```

### Auth
- Login returns access token (15min) + refresh token (7d)
- Access token in Authorization header: `Bearer <token>`
- Refresh token in httpOnly cookie
- Auth middleware decodes token, attaches `ctx.user`

## Build Order

1. **Scaffolding**: Init project, TypeScript config, install Hono + Drizzle + Zod
2. **Config**: Environment validation with Zod, logger setup
3. **Database**: Schema definition, migration setup, database client
4. **Error handling**: Custom error classes, global error middleware
5. **Auth routes**: Register, login, refresh, auth middleware
6. **Core feature routes**: CRUD for the primary entity
7. **Validation**: Zod schemas for all inputs, middleware integration
8. **Background jobs** (if needed): BullMQ setup, worker process
9. **Caching** (if needed): Redis cache layer for hot paths
10. **API docs**: OpenAPI spec generation, Swagger UI
11. **Testing**: Integration tests for routes, unit tests for services
12. **Deploy**: Railway/Fly config, health check endpoint, monitoring

## Common Pitfalls

- **Don't skip input validation.** Every route input must be validated with Zod. No exceptions.
- **Don't return raw database errors.** Map them to user-friendly messages.
- **Don't forget rate limiting.** Add it from day one on auth routes at minimum.
- **Don't hardcode secrets.** All config via environment variables, validated at startup.
- **Don't skip health checks.** `/health` endpoint for load balancers and monitoring.

## Skills for Build Phase

| Skill | When |
|-------|------|
| `/deep-research` | Comparing API frameworks or infra choices |

## Alternative Stack: Python (FastAPI)

| Layer | Technology | Why |
|-------|-----------|-----|
| Framework | FastAPI | Async, automatic OpenAPI docs, type validation via Pydantic |
| Language | Python 3.12+ | Type hints, async/await |
| ORM | SQLAlchemy 2.0 | Industry standard, async support, mature ecosystem |
| Migrations | Alembic | De facto migration tool for SQLAlchemy |
| Validation | Pydantic v2 | Built into FastAPI, fast, great DX |
| Auth | python-jose (JWT) | Lightweight JWT handling |
| Queue | Celery + Redis | Battle-tested, distributed task queue |
| Testing | pytest + httpx | pytest is the standard, httpx for async API testing |
| Hosting | Railway, Fly.io, AWS Lambda (Mangum) | Railway for simplicity, Lambda for serverless |

### Directory Structure

```
app/
  main.py                          # FastAPI app, CORS, middleware
  routers/
    auth.py                        # POST /auth/login, /auth/register
    users.py                       # GET/PATCH /users/:id
    [feature].py                   # CRUD routes per feature
  models/
    user.py                        # SQLAlchemy models
    [feature].py
  schemas/
    user.py                        # Pydantic request/response schemas
    [feature].py
  services/
    auth.py                        # JWT creation, verification
    email.py                       # Email sending
  db/
    session.py                     # Database session factory
    base.py                        # Base model class
  core/
    config.py                      # Settings from environment (pydantic-settings)
    security.py                    # Password hashing, token utilities
    exceptions.py                  # Custom exception handlers
  tasks/                           # Celery tasks (background jobs)
    email_tasks.py
alembic/
  versions/                        # Migration files
  env.py
tests/
  test_auth.py
  test_[feature].py
  conftest.py                      # Fixtures, test DB setup
requirements.txt
Dockerfile
```

### Build Order (Python)

1. **Scaffolding**: Create project, install FastAPI + Uvicorn + SQLAlchemy + Alembic + Pydantic
2. **Config**: pydantic-settings for env vars, logging setup
3. **Database**: SQLAlchemy models, Alembic init, first migration
4. **Auth**: JWT endpoints (register, login, refresh), middleware
5. **Core feature**: CRUD routes with Pydantic validation
6. **Background jobs** (if needed): Celery + Redis setup
7. **Testing**: pytest fixtures, httpx async tests for every route
8. **API docs**: FastAPI auto-generates OpenAPI — customize descriptions
9. **Docker**: Dockerfile + docker-compose.yml for local dev
10. **Deploy**: Railway or Fly.io with Dockerfile

## Alternative Stack: Go

| Layer | Technology | Why |
|-------|-----------|-----|
| Framework | Chi | Lightweight, idiomatic, stdlib-compatible middleware |
| Language | Go 1.22+ | Fast compilation, single binary, excellent concurrency |
| ORM / Query | sqlc | Type-safe SQL — write SQL, generate Go code |
| Migrations | golang-migrate | Simple, SQL-based migrations |
| Validation | go-playground/validator | Struct tag validation |
| Auth | golang-jwt | Standard JWT library |
| Queue | asynq + Redis | Simple, reliable, Go-native |
| Testing | go test + testify | Built-in testing + assertion library |
| Hosting | Fly.io, Railway, AWS ECS | Fly.io for global edge, Railway for simplicity |

### Directory Structure

```
cmd/
  server/
    main.go                        # Entry point, wire dependencies, start server
internal/
  handler/
    auth.go                        # Auth HTTP handlers
    user.go                        # User HTTP handlers
    [feature].go                   # Feature handlers
  service/
    auth.go                        # Auth business logic
    user.go                        # User business logic
  repository/
    user.go                        # Database queries (generated by sqlc)
    queries.sql                    # Raw SQL queries for sqlc
  middleware/
    auth.go                        # JWT middleware
    ratelimit.go                   # Rate limiting
    logger.go                      # Request logging
  model/
    user.go                        # Domain types
  config/
    config.go                      # Environment config
db/
  migrations/
    000001_init.up.sql
    000001_init.down.sql
  sqlc.yaml                        # sqlc configuration
tests/
  integration/
    auth_test.go
    [feature]_test.go
Dockerfile
go.mod
go.sum
```

### Build Order (Go)

1. **Scaffolding**: `go mod init`, install Chi + sqlc + golang-migrate
2. **Config**: Environment loading, structured logging (slog)
3. **Database**: SQL schema, migrations, sqlc codegen
4. **Router**: Chi router with middleware (logger, recoverer, CORS)
5. **Auth**: JWT endpoints, auth middleware
6. **Core feature**: CRUD handlers, wire to sqlc repository
7. **Validation**: Request validation with validator
8. **Testing**: Integration tests with test database
9. **Docker**: Multi-stage Dockerfile (build + run stages, ~20MB final image)
10. **Deploy**: Fly.io or Railway with Dockerfile

## See Also

- `knowledge/building-blocks/api-design-patterns.md` — REST conventions, response shapes, validation
- `knowledge/building-blocks/database-patterns.md` — Schema design, Drizzle setup, migrations
- `knowledge/building-blocks/auth-patterns.md` — JWT patterns, API key auth
- `knowledge/building-blocks/deployment-patterns.md` — Railway, Fly.io, Docker config
- `knowledge/building-blocks/testing-patterns.md` — Integration tests for every endpoint
- `knowledge/building-blocks/security-ops-patterns.md` — Rate limiting, input validation, CORS, monitoring
