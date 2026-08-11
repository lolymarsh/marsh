---
name: implement-python-restapi
description: >-
  Implement (or fix/extend) a Python REST API module — FastAPI + SQLAlchemy +
  Pydantic — following the marsh rules (src layout, Go-style naming, strict
  type hints, AppError hierarchy), test-first with pytest, ending with proper
  verification (ruff/mypy/pytest) and self-review. Use whenever the user asks
  to implement/add/create endpoints, services, repositories in a Python/FastAPI
  project, or asks "what to do after implementing", "how to lint/test Python"
  — including FastAPI, Pydantic, SQLAlchemy, Alembic, or "implement phase" in
  a Python project.
---

# implement-python-restapi — Implement FastAPI Module (Test-First + Verify)

Workflow for writing/editing Python REST API code (FastAPI) in projects that
follow the marsh rules: `src/{package}/` layout, Go-style naming, strict type
hints everywhere, AppError hierarchy, unified `{code, message, data}` response.

## 1. Before starting — read project context

- `pyproject.toml` (ruff/mypy config, deps), `Makefile`, `.env`
- `src/{package}/main.py` (entrypoint), `api/deps.py` (DI), `shared/errors.py`
- One existing `domain/{module}/` to match actual structure and style

If the project does not match this pattern, ask the user first.

## 2. Core workflow — Test-First

1. **Red** — write failing tests (service unit tests with mocked repo, or API
   integration tests)
2. **Green** — minimal implementation
3. **Refactor** — align with the rules below

**Test stack** (per `rules/python-rest-api/10_TestingStandards.md`):
pytest + pytest-asyncio + httpx `AsyncClient`; coverage with pytest-cov
(services 80%, API 90%, overall 70%).

Service test — mock the repo layer:

```python
@pytest.mark.asyncio
async def test_Update_throws_not_found():
    repo = AsyncMock()
    repo.FindById.return_value = None
    svc = UserService(repo, mock_audit)

    with pytest.raises(NotFoundError):
        await svc.Update("bad-id", {}, 1, "admin")
```

API test — through the real app:

```python
@pytest.mark.asyncio
async def test_get_user_success(client: AsyncClient, auth_token: str):
    resp = await client.get("/api/users/123", headers={"Authorization": f"Bearer {auth_token}"})
    assert resp.status_code == 200
    assert resp.json()["code"] == 200
```

Cover success + error paths + auth (401/403) in tests.

## 3. While implementing — patterns you must not deviate from

### Module structure (1 folder = 1 domain, under `src/{package}/domain/`)

| File | Purpose |
|---|---|
| `entity.py` | SQLAlchemy model / Pydantic data model |
| `schema.py` | Pydantic request/response schemas |
| `repo.py` | Repository class (SQLAlchemy queries) |
| `service.py` | Service class (business logic) |

Routes in `api/{module}.py`, DI in `api/deps.py`, infra (DB/Redis/RabbitMQ)
in `infrastructure/`.

### Strict typing (per `rules/python-rest-api/02_CodingStandards.md`)

- Type hints required on **all** params + return types (ruff ANN, mypy strict)
- `X | None` (never `Optional[X]`), `list[X]` / `dict[K, V]` (never `List`/`Dict`)
- Constructor injection for dependencies — no global imports

### Naming (Go-style)

| Scope | Convention | Example |
|---|---|---|
| Public methods | PascalCase | `FindById`, `CreateUser` |
| Private methods | `_camelCase` | `_toResponse` |
| Classes | PascalCase | `UserService` |
| Variables | snake_case | `user_id` |
| Constants | UPPER_SNAKE | `ROLE_ADMIN` |

### Error handling (per `rules/python-rest-api/03_ErrorHandling.md`)

| Layer | Error type |
|---|---|
| Handler | Catch → HTTP response |
| Service | Raise `AppError` subclass (NotFoundError, ConflictError, ...) |
| Repository | Return `None` (don't raise) |

Register one global `@app.exception_handler(AppError)` returning
`{"code": status, "message": ...}` — never leak internal error details.

### Conventions

- Every list endpoint has pagination (separate count and find)
- PATCH/PUT carry a `version` check → conflict → 409
- Multi-table writes inside a transaction (auto-rollback on AppError)
- Soft delete — no hard delete

## 4. After implementing — what to do next (never skip)

Run in this order until everything passes:

```bash
ruff check src/          # lint — ANN enforces type hints, max complexity 15
ruff format src/         # format — double quotes, line length 100
mypy src/                # type check (strict)
pytest --cov=src/ tests/ # test with coverage targets
```

If ruff keeps complaining, see `lint/python.md` → `linters-settings`
(ANN001/201/204 type-hint rules, per-file ignores for tests).

## 5. Self-review checklist — before reporting back

- [ ] Type hints on all params + return types; no `Optional[X]` / `List[X]`
- [ ] Service public methods raise `AppError` subclasses only
- [ ] Repository returns `None` for not-found — no exceptions leaking
- [ ] Every list endpoint has pagination; PATCH/PUT has `version` check
- [ ] Multi-table writes inside a transaction
- [ ] Response format `{code, message, data[, pagination]}` via shared helpers
- [ ] No secrets in code; `.env` only
- [ ] Tests cover success + error + auth paths; coverage targets met
- [ ] `ruff check` + `ruff format` + `mypy` + `pytest` pass

## 6. Wrap up — summarize for the user

Brief summary: modules/files created or changed, commands run and passing,
tests added, what's left for the user (review, running tests needing a real
DB, committing).
