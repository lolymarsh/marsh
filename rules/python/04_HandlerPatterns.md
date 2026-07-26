---
trigger: always_on
---

# Handler / API Layer Patterns

## 1. FastAPI Router (Recommended)

```python
# src/api/user.py
from fastapi import APIRouter, Depends, status
from src.domain.user.service import UserService
from src.domain.user.schema import (
    UserResponse, CreateUserInput, UpdateUserInput, FilterInput,
)
from src.api.deps import get_user_service, get_current_user

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/{user_id}", response_model=ApiResponse[UserResponse])
async def get_user(
    user_id: str,
    svc: UserService = Depends(get_user_service),
    current_user: UserEntity = Depends(get_current_user),
) -> ApiResponse[UserResponse]:
    result = await svc.GetProfile(user_id, current_user.role)
    return ApiResponse(code=200, message="success", data=result)


@router.post("/filter", response_model=ApiListResponse[UserResponse])
async def filter_users(
    input: FilterInput,
    svc: UserService = Depends(get_user_service),
    current_user: UserEntity = Depends(get_current_user),
) -> ApiListResponse[UserResponse]:
    data, pagination = await svc.FilterUsers(input.toDomain())
    return ApiListResponse(
        code=200, message="success", data=data, pagination=pagination,
    )


@router.patch("/{user_id}", response_model=ApiResponse[UserResponse])
async def update_user(
    user_id: str,
    input: UpdateUserInput,
    svc: UserService = Depends(get_user_service),
    current_user: UserEntity = Depends(get_current_user),
) -> ApiResponse[UserResponse]:
    result = await svc.Update(
        user_id, input.toDomain(), current_user.id, current_user.audit_meta,
    )
    return ApiResponse(code=200, message="success", data=result)


@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def soft_delete_user(
    user_id: str,
    input: DeleteInput,
    svc: UserService = Depends(get_user_service),
    current_user: UserEntity = Depends(get_current_user),
) -> None:
    await svc.SoftDelete(user_id, input.version, current_user.id)
```

## 2. Standard Handler Flow

```
1. Parse path/query/body params (FastAPI auto-validates with Pydantic)
2. Get dependencies (via Depends)
3. Call service method
4. Return response (FastAPI auto-serializes)
```

## 3. Router Registration

```python
# src/api/router.py
from fastapi import APIRouter
from src.api import user, auth, auditlog

router = APIRouter()
router.include_router(auth.router)
router.include_router(user.router)
router.include_router(auditlog.router)
```

## 4. Dependencies

```python
# src/api/deps.py
from fastapi import Depends, HTTPException, Header
from src.infrastructure.database import get_session
from src.domain.user.service import UserService
from src.domain.user.repo import UserRepository

async def get_user_service(
    db = Depends(get_session),
) -> UserService:
    repo = UserRepository(db)
    return UserService(repo, ...)
```
