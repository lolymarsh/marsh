---
trigger: always_on
---

# Python Project Structure

## 1. Directory Layout

```
{project}/
├── pyproject.toml                  # Project config (PEP 621)
├── requirements.txt / poetry.lock  # Dependencies
├── Dockerfile
├── docker-compose.yml
├── .env / _env_example
├── Makefile                        # lint/test/run
│
├── src/
│   └── {package}/
│       ├── __init__.py
│       ├── main.py                 # Entry point
│       ├── config/
│       │   ├── __init__.py
│       │   ├── settings.py         # Pydantic Settings
│       │   └── logger.py           # Logging config
│       ├── domain/{module}/
│       │   ├── __init__.py
│       │   ├── entity.py           # Pydantic models / SQLAlchemy models
│       │   ├── repo.py             # Repository interface + impl
│       │   ├── service.py          # Service interface + impl
│       │   └── schema.py           # Request/Response schemas
│       ├── api/
│       │   ├── __init__.py
│       │   ├── deps.py             # FastAPI dependencies (DI)
│       │   ├── router.py           # Central router
│       │   └── {module}.py         # Route definitions
│       ├── infrastructure/
│       │   ├── __init__.py
│       │   ├── database.py         # SQLAlchemy engine/session
│       │   ├── redis.py            # Redis client
│       │   └── rabbitmq.py         # RabbitMQ client
│       └── shared/
│           ├── __init__.py
│           ├── errors.py           # Custom error classes
│           ├── response.py         # Unified response helpers
│           └── middleware.py       # Auth, audit, etc.
│
├── migrations/                     # Alembic migrations
│   ├── alembic.ini
│   ├── env.py
│   └── versions/
│
├── tests/
│   ├── conftest.py                 # Shared fixtures
│   ├── domain/{module}/
│   │   ├── test_service.py
│   │   └── test_repo.py
│   └── api/
│       └── test_{module}.py
│
└── scripts/                        # Utility scripts
```

## 2. Module Structure

| File | Purpose |
|---|---|
| `entity.py` | SQLAlchemy model / Pydantic data model |
| `schema.py` | Pydantic request/response schemas |
| `repo.py` | Repository class (SQLAlchemy queries) |
| `service.py` | Service class (business logic) |

## 3. Naming Conventions (Go-style)

| Item | Convention | Example |
|---|---|---|
| Files | snake_case | `user_service.py` |
| Classes | PascalCase | `UserService`, `UserRepository` |
| Public methods | PascalCase | `FindById`, `CreateUser`, `Update` |
| Private methods | `_camelCase` | `_toResponse`, `_validateInput` |
| Variables | snake_case | `user_id`, `first_name` |
| Constants | UPPER_SNAKE | `ROLE_ADMIN` |
| DB tables | snake_case | `users` |
| DB columns | snake_case | `first_name` |
