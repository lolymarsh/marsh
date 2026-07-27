---
trigger: always_on
---

# Response Patterns

## 1. Unified Response Model

```python
from pydantic import BaseModel
from typing import Any, Optional

class ApiResponse(BaseModel):
    code: int
    message: str
    data: Optional[Any] = None

class Pagination(BaseModel):
    page: int
    page_size: int
    total_data: int
    total_page: int
    has_next_page: bool
    has_previous_page: bool

class ApiListResponse(BaseModel):
    code: int
    message: str
    data: list[Any]
    pagination: Pagination
```

## 2. Response Formats

**Success:**
```json
{ "code": 200, "message": "success", "data": { ... } }
```

**Paginated:**
```json
{
    "code": 200,
    "message": "success",
    "data": [ ... ],
    "pagination": {
        "page": 1,
        "pageSize": 20,
        "totalData": 100,
        "totalPage": 5,
        "hasNextPage": true,
        "hasPreviousPage": false
    }
}
```

**Error:**
```json
{ "code": 404, "message": "not found" }
```

## 3. HTTP Status Mapping

| Exception | Status |
|---|---|
| `NotFoundError` | 404 |
| `BadRequestError` | 400 |
| `UnauthorizedError` | 401 |
| `ForbiddenError` | 403 |
| `ConflictError` | 409 |
| `AppError` (generic) | 500 |
