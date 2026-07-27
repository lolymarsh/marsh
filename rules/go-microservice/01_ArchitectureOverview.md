---
trigger: always_on
---

# Microservice Architecture (Go-style)

> **Languages**: Go (primary), TypeScript, Python
> **Service-to-Service**: ConnectRPC (gRPC), RabbitMQ (event-driven)
> **API Gateway**: Traefik / Nginx
> **DB per Service**: PostgreSQL / MySQL
> **Events**: RabbitMQ + outbox pattern

---

## 1. Core Principles

### Database-per-Service
- **Each service owns its own database** — never share DBs across services
- **No cross-service joins** — use API calls or events instead
- **Saga pattern** for distributed transactions

### Communication Styles

| Use Case | Method | Protocol |
|---|---|---|
| Sync (request-reply) | ConnectRPC / gRPC | HTTP/2, protobuf |
| Async (event-driven) | RabbitMQ | AMQP 0-9-1 |
| Query (read model) | CQRS + Materialized View | Eventual consistency |

### Service Boundaries

Split services by **business domain** (DDD bounded context):

```
user-service        → users, auth, roles
customer-service    → customers, vehicles
inventory-service   → products, categories, stock
order-service       → invoices, quotations, payments
notification-service → email, LINE, push
audit-service       → audit logs (MongoDB)
```

---

## 2. Service Template

```
{service-name}/
├── main.go                          # Entry point
├── go.mod                           # module {domain}-service
├── Makefile                         # build/lint/test
├── Dockerfile                       # Multi-stage
├── .env
│
├── internal/
│   ├── server/
│   │   ├── server.go                # HTTP/mux setup, graceful shutdown
│   │   └── di.go                    # DI container
│   ├── {module}/
│   │   ├── entity.go                # Domain model
│   │   ├── repo.go                  # Repository (interface + impl)
│   │   ├── service.go               # Business logic
│   │   ├── handler.go               # ConnectRPC handler
│   │   └── event.go                 # Event publishers/consumers
│   ├── client/                      # External service clients
│   │   ├── order_client.go          # ConnectRPC client → order-service
│   │   └── notification_client.go   # ConnectRPC client → notification-service
│   └── consumer/                    # RabbitMQ consumers
│       └── order_consumer.go
│
├── proto/                           # Service proto definitions
│   └── {service}/v1/*.proto
├── gen/                             # Generated code
├── migrations/
│   └── *.sql
│
└── pkg/
    ├── apperrors/                   # Error types
    ├── configs/                     # Config structs
    ├── infrastructure/              # DB, Redis, RabbitMQ init
    ├── logger/                      # Zap logger
    └── util/                        # Shared utilities
```

**Differences from monolith:**
- No `internal/router/` — routes registered directly on handlers
- `internal/client/` added — ConnectRPC clients for other services
- `internal/consumer/` added — RabbitMQ message consumers
- Proto definitions live in this service or shared repo depending on org
