---
trigger: always_on
---

# Database Patterns

## 1. ORM: SQLAlchemy 2.0 (Async)

```python
from sqlalchemy import Column, String, Integer, DateTime, Boolean, Enum
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from datetime import datetime
import uuid

class Base(DeclarativeBase):
    pass

class UserModel(Base):
    __tablename__ = "users"

    id: Mapped[str] = mapped_column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    username: Mapped[str] = mapped_column(String(100), unique=True, nullable=False)
    password_hash: Mapped[str] = mapped_column(String(255), nullable=False)
    display_name: Mapped[str] = mapped_column(String(255), nullable=False)
    role: Mapped[str] = mapped_column(Enum("ADMIN", "STAFF", "USER"), default="USER")
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    version: Mapped[int] = mapped_column(Integer, default=1)
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
    updated_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    deleted_at: Mapped[datetime | None] = mapped_column(DateTime, nullable=True)
```

## 2. Conventions

| Item | Convention |
|---|---|
| PK | UUID (`String(36)`) |
| Timestamps | `DateTime` (UTC) |
| Soft delete | `deleted_at` nullable |
| Locking | `version` int default 1 |
| Money | `Numeric(12, 2)` |
| Text | `Text` for long strings |

## 3. Migrations (Alembic)

```bash
alembic init migrations
alembic revision --autogenerate -m "description"
alembic upgrade head
```

```python
# migrations/env.py
from src.domain import Base
target_metadata = Base.metadata
```

## 4. Database Session

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

engine = create_async_engine(settings.DATABASE_URL)
AsyncSessionLocal = async_sessionmaker(engine, class_=AsyncSession)

async def GetSession() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session
```

## 5. Multi-DB Support

| Database | Use Case |
|---|---|
| **PostgreSQL/MySQL** | Core business data (ACID) |
| **Redis** | Cache, sessions |
| **MongoDB** | Audit logs, event sourcing |
