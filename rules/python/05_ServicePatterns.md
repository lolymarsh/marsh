---
trigger: always_on
---

# Service Layer Patterns

## 1. Class Structure

```python
from typing import Optional
from src.domain.user.entity import UserEntity
from src.domain.user.repo import UserRepository
from src.domain.user.schema import UserResponse
from src.shared.errors import NotFoundError, ConflictError

class UserService:
    def __init__(
        self,
        repo: UserRepository,
        audit_service: AuditLogService,
    ) -> None:
        self._repo = repo
        self._audit_service = audit_service

    async def Update(
        self,
        user_id: str,
        input: dict,
        version: int,
        admin_id: str,
    ) -> UserResponse:
        existing = await self._repo.FindById(user_id)
        if not existing:
            raise NotFoundError(f"User {user_id} not found")

        updated = await self._repo.Update(user_id, input, version)
        if not updated:
            raise ConflictError("Version mismatch")

        await self._audit_service.Log(
            action="UPDATE", table="users", record_id=user_id,
            user_id=admin_id, old=existing, new=updated,
        )

        return self._toResponse(updated)

    def _toResponse(self, entity: UserEntity) -> UserResponse:
        return UserResponse(
            id=entity.id,
            username=entity.username,
            display_name=entity.display_name,
            created_at=entity.created_at.isoformat(),
        )
```

## 2. CRUD Pattern

- Guard clauses first (existence → permission → mutate)
- Audit after every mutation
- Private helpers with `_` prefix
- Public methods ≤ 50 lines

## 3. Transaction Pattern

```python
from sqlalchemy.ext.asyncio import AsyncSession

async def CreateInvoice(
    self,
    input: CreateInvoiceInput,
    user_id: str,
) -> InvoiceResponse:
    async with self._db.begin() as tx:
        product = await self._repo.GetProductForUpdate(tx, input.product_id)
        if not product or product.stock < input.quantity:
            raise BadRequestError("Insufficient stock")

        invoice = await self._repo.CreateInvoice(tx, input, user_id)
        await self._repo.UpdateStock(tx, product.id, product.stock - input.quantity)

    return self._toResponse(invoice)
```

## 4. Filter/Pagination Pattern

Separate `CountByFilter` and `FindByFilter` — service layer combines them only when pagination is needed:

```python
import asyncio
import math

async def FilterUsers(
    self,
    input: FilterInput,
) -> tuple[list[UserResponse], Pagination]:
    data, total = await asyncio.gather(
        self._repo.FindByFilter(
            search=input.search,
            role=input.role,
            page=input.page,
            page_size=input.page_size,
        ),
        self._repo.CountByFilter(
            search=input.search,
            role=input.role,
        ),
    )
    pagination = Pagination(
        page=input.page,
        page_size=input.page_size,
        total_data=total,
        total_page=math.ceil(total / input.page_size),
    )
    return [self._toResponse(u) for u in data], pagination
```
