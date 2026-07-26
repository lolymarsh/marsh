---
trigger: always_on
---

# Error Handling Patterns

## 1. Error Hierarchy

```python
class AppError(Exception):
    def __init__(self, status_code: int, message: str, details: Any = None) -> None:
        self.status_code = status_code
        self.details = details
        super().__init__(message)

class NotFoundError(AppError):
    def __init__(self, message: str = "Resource not found") -> None:
        super().__init__(404, message)

class ConflictError(AppError):
    def __init__(self, message: str = "Version mismatch") -> None:
        super().__init__(409, message)

class BadRequestError(AppError):
    def __init__(self, message: str = "Invalid input") -> None:
        super().__init__(400, message)

class UnauthorizedError(AppError):
    def __init__(self, message: str = "Unauthorized") -> None:
        super().__init__(401, message)

class ForbiddenError(AppError):
    def __init__(self, message: str = "Forbidden") -> None:
        super().__init__(403, message)
```

## 2. Per-Layer Rules

| Layer | Error Type |
|---|---|
| **Handler** | Catch → return HTTP response |
| **Service** | Raise `AppError` subclass |
| **Repository** | Return `None` (don't raise) |
| **Transaction** | Raise `AppError` (auto-rollback) |

## 3. FastAPI Error Handler

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from src.shared.errors import AppError

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError) -> JSONResponse:
    return JSONResponse(
        status_code=exc.status_code,
        content={"code": exc.status_code, "message": exc.message},
    )
```

## 4. Repository → Service Flow

```
Repo.FindById → returns None    → Service raises NotFoundError
Repo.Update   → returns None    → Service raises ConflictError
Repo.FindById → returns entity  → Service continues
```
