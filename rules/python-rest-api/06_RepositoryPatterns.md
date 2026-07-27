---
trigger: always_on
---

# Repository Layer Patterns

## 1. SQLAlchemy (Async)

```python
from sqlalchemy import select, update, func
from sqlalchemy.ext.asyncio import AsyncSession
from src.domain.user.entity import UserEntity

class UserRepository:
    def __init__(self, db: AsyncSession) -> None:
        self._db = db

    async def FindById(self, user_id: str) -> UserEntity | None:
        stmt = select(UserEntity).where(
            UserEntity.id == user_id,
            UserEntity.deleted_at.is_(None),
        )
        result = await self._db.execute(stmt)
        return result.scalar_one_or_none()

    async def Create(self, data: dict) -> UserEntity:
        entity = UserEntity(**data)
        self._db.add(entity)
        await self._db.flush()
        await self._db.refresh(entity)
        return entity

    async def Update(self, user_id: str, data: dict, version: int) -> UserEntity | None:
        stmt = (
            update(UserEntity)
            .where(UserEntity.id == user_id, UserEntity.version == version)
            .values(**data, version=version + 1)
            .returning(UserEntity)
        )
        result = await self._db.execute(stmt)
        return result.scalar_one_or_none()  # None = version mismatch

    async def SoftDelete(self, user_id: str, version: int) -> bool:
        from datetime import datetime
        stmt = (
            update(UserEntity)
            .where(UserEntity.id == user_id, UserEntity.version == version)
            .values(deleted_at=datetime.utcnow(), version=version + 1)
        )
        result = await self._db.execute(stmt)
        return result.rowcount > 0

    # --- Filter (separate count from data) ---

    async CountByFilter(
        self,
        search: str | None = None,
        role: str | None = None,
    ) -> int:
        stmt = select(func.count()).select_from(
            self._buildFilterQuery(search, role).subquery()
        )
        total = await self._db.scalar(stmt)
        return total or 0

    async FindByFilter(
        self,
        search: str | None = None,
        role: str | None = None,
        page: int = 1,
        page_size: int = 20,
    ) -> list[UserEntity]:
        stmt = (
            self._buildFilterQuery(search, role)
            .order_by(UserEntity.created_at.desc())
            .limit(page_size)
            .offset((page - 1) * page_size)
        )
        result = await self._db.execute(stmt)
        return result.scalars().all()

    def _buildFilterQuery(self, search: str | None = None, role: str | None = None):
        stmt = select(UserEntity).where(UserEntity.deleted_at.is_(None))
        if search:
            stmt = stmt.where(UserEntity.display_name.ilike(f"%{search}%"))
        if role:
            stmt = stmt.where(UserEntity.role == role)
        return stmt
```

## 2. Rules

- Return `None` on not found (don't raise)
- Return `None` on version mismatch (service layer raises ConflictError)
- Always filter `deleted_at.is_(None)` for soft delete
- Constructor injection: `__init__(self, db: AsyncSession)`
- No global `db` import
