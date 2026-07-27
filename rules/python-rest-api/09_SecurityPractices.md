---
trigger: always_on
---

# Security Practices

## 1. Authentication

- JWT via `python-jose` or `PyJWT`
- Password hashing: `bcrypt` or `argon2`
- FastAPI `Depends(GetCurrentUser)` for auth protection
- Redis sessions for token blacklist/validation

```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def GetCurrentUser(
    token: str = Depends(security),
    redis = Depends(get_redis),
) -> UserEntity:
    payload = DecodeJWT(token.credentials)
    session = await redis.get(f"session:{payload['user_id']}")
    if not session:
        raise HTTPException(status_code=401, detail="Session expired")
    return payload
```

## 2. Authorization

- Role-based: `ADMIN > STAFF > USER`
- Custom dependency for role checks

```python
def RequireRole(allowed_roles: list[str]):
    async def checker(current_user: UserEntity = Depends(GetCurrentUser)):
        if current_user.role not in allowed_roles:
            raise HTTPException(status_code=403, detail="Forbidden")
        return current_user
    return checker

@router.get("/admin/users")
async def admin_list(
    current_user: UserEntity = Depends(RequireRole(["ADMIN"])),
):
    ...
```

## 3. Input Validation

- **Pydantic** schemas for all inputs (auto-validation)
- Never trust raw request data
- Use `model_validator` for cross-field validation

```python
from pydantic import BaseModel, model_validator

class CreateUserInput(BaseModel):
    password: str
    confirm_password: str

    @model_validator(mode="after")
    def _passwordsMatch(self) -> "CreateUserInput":
        if self.password != self.confirm_password:
            raise ValueError("Passwords do not match")
        return self
```

## 4. SQL Injection Prevention

- SQLAlchemy ORM (parameterized queries)
- No raw SQL string concatenation
- If raw SQL needed: use SQLAlchemy `text()` with bind params

## 5. Soft Delete

- `deleted_at: DateTime | None` — never hard delete
- Always filter `deleted_at.is_(None)` in queries
- Soft-deleted records excluded from all list/get operations

## 6. Sensitive Data

- `password_hash` excluded from Pydantic response schemas
- Don't expose internal error details
- Log full errors server-side, return generic messages
