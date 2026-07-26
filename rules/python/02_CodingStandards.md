---
trigger: always_on
---

# Python Coding Standards (Go-style)

## 1. General

- **Python 3.12+** — latest stable
- Type hints required on all functions (PEP 484)
- Use `pyproject.toml` for project config (PEP 621)

## 2. Strict Typing (Go/TS style)

**Type hints enforced everywhere** — no exceptions:

```python
# ✅ GOOD: fully typed
class UserService:
    def __init__(self, repo: UserRepository, redis: Redis) -> None:
        self._repo: UserRepository = repo
        self._redis: Redis = redis

    async def GetProfile(self, user_id: str) -> UserEntity | None:
        result: UserEntity | None = await self._repo.FindById(user_id)
        return result

    def _toResponse(self, entity: UserEntity) -> UserResponse:
        return UserResponse(id=entity.id, name=entity.display_name)

# ❌ BAD: missing types
class UserService:
    def __init__(self, repo, redis):  # no types
        self._repo = repo

    async def GetProfile(self, user_id):  # missing return type
        ...
```

**Rules:**
- All function params + return types: required
- All class attributes: `self._x: Type = ...`
- All variable assignments: annotated (optional but encouraged)
- Use `X | None` (3.10+) — never `Optional[X]`
- Use `list[X]`, `dict[K, V]` — never `List[X]`, `Dict[K, V]` (3.9+)

## 3. Function Length

- **max 50 lines** per function
- Cyclomatic complexity: **max 15**
- Break into smaller helper functions when exceeded

## 4. Guard Clauses First

```python
async def Update(self, user_id: str, input: UpdateInput, version: int) -> UserResponse:
    existing = await self._repo.FindById(user_id)
    if not existing:
        raise NotFoundError(f"User {user_id} not found")

    updated = await self._repo.Update(user_id, input, version)
    if not updated:
        raise ConflictError("Version mismatch")

    return self._toResponse(updated)
```

## 5. Dependency Injection

```python
class UserService:
    def __init__(
        self,
        repo: UserRepository,
        redis: Redis,
        audit_service: AuditLogService,
    ) -> None:
        self._repo = repo
        self._redis = redis
        self._audit_service = audit_service
```

- Constructor injection — no global imports
- Wire in central DI function or FastAPI `dependencies`

## 6. Error Handling

- Custom error hierarchy (see ErrorHandling.md)
- Service layer raises custom exceptions
- Handler layer catches and converts to HTTP responses
- Never expose internal error details to client

## 7. Naming (Go-style)

| Scope | Convention | Example |
|---|---|---|
| Public methods | PascalCase | `FindById`, `CreateUser` |
| Private methods | `_camelCase` | `_toResponse`, `_validateInput` |
| Classes | PascalCase | `UserService` |
| Variables | snake_case | `user_id` |
| Constants | UPPER_SNAKE | `ROLE_ADMIN` |
