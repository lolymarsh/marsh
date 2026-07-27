---
trigger: always_on
---

# Go REST API Project Structure

## 1. Directory Layout

```
{project}/
├── main.go                          # Entry point
├── go.mod                           # module loly_service
├── Makefile                         # build/run/lint/test/docker
├── Dockerfile                       # Multi-stage build
├── docker-compose.yml               # Dev/test infra
├── .env / _env_example
├── .golangci.yml                    # Linter config (golangci-lint v2)
│
├── migrations/
│   ├── migrations.go                # Embedded SQL runner (goose v3)
│   └── {driver}/*.sql               # DB-specific SQL migrations
│
├── internal/                        # Private application code
│   ├── server/
│   │   ├── {echo|connect}Server.go  # Server setup, middleware, graceful shutdown
│   │   └── di.go                    # Manual DI container
│   ├── {module}/                    # One folder = one business domain
│   │   ├── entity.go                # Bun model struct
│   │   ├── repo.go                  # Repository interface + impl
│   │   ├── service.go               # Service interface + impl
│   │   ├── handler.go               # Handler struct + constructor
│   │   ├── *handler.go              # Handler methods (split by user/admin)
│   │   ├── request.go               # Request/Response DTOs
│   │   ├── route.go                 # Route registration
│   │   └── *_test.go                # Integration tests
│   └── testutil/                    # Test helpers
│
├── pkg/                             # Shared libraries
│   ├── apperrors/                   # Custom error types
│   ├── authz/                       # Roles + permissions
│   ├── common/                      # Context helpers, validator, pagination
│   ├── configs/                     # Config structs (one file per section)
│   ├── constants/                   # Constants (table names, context keys)
│   ├── infrastructure/              # DB/Redis/RabbitMQ init + factories
│   ├── logger/                      # Global logger (zap)
│   ├── mapper/                      # Generic copier (MapJSON[T])
│   ├── middleware/                  # Echo middleware (Echo only)
│   ├── connectutil/                 # ConnectRPC interceptors + errors
│   ├── response/                    # HandleError, HandleSuccess
│   ├── token/                       # JWT token store + validator
│   └── util/                        # UUID, JWT, password, snowflake
│
├── proto/                           # Protobuf definitions (ConnectRPC only)
└── internal/gen/                    # Generated code (ConnectRPC only)
```

---

## 2. Module Structure

Every business module follows identical layout:

| File | Responsibility |
|---|---|
| `entity.go` | Bun model struct with `bun` tags |
| `repo.go` | Repository interface + unexported impl |
| `service.go` | Service interface + unexported impl |
| `handler.go` | Handler struct + constructor (no methods) |
| `*_handler.go` | Actual handler methods |
| `request.go` | Request/Response DTOs |
| `route.go` | Route registration function |

---

## 3. Constants Organization

```
pkg/constants/
├── table.go        # TableUsers, TableAuthUsers
├── auth.go         # Context keys, Redis key prefixes
├── audit.go        # AuditActionCreate, AuditActionUpdate
└── server.go       # MODE_SERVER_DEV, MODE_SERVER_PROD
```

---

## 4. Layer Responsibilities

| Layer | Responsibility |
|---|---|
| **Handler** | Bind request, validate, extract user context, call service, return response |
| **Service** | Business logic, authorization checks, call repository, audit logging |
| **Repository** | Database operations (bun), soft-delete filter, optimistic locking |

## 5. Architecture Rules

```
Handler → Service → Repository → DB
        → Service → Service (cross-module via interface)
```

- Handler depends on Service interface (never concrete type)
- Service depends on Repository interface (never concrete type)
- Cross-module calls happen at Service layer, injected via constructor
