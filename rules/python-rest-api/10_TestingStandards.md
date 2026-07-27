---
trigger: always_on
---

# Testing Standards

## 1. Framework

- **pytest** — primary test framework
- **pytest-asyncio** — for async tests
- **httpx** — for FastAPI test client
- Coverage via `pytest-cov`

## 2. Test Structure

```
tests/
├── conftest.py                    # Shared fixtures (DB session, client, auth)
├── domain/{module}/
│   ├── test_service.py            # Service unit tests (mocked repo)
│   └── test_repo.py               # Repository tests (real DB via testcontainers)
└── api/
    └── test_{module}.py           # API integration tests
```

## 3. Service Test Pattern

```python
import pytest
from unittest.mock import AsyncMock
from src.domain.user.service import UserService
from src.shared.errors import NotFoundError, ConflictError

@pytest.mark.asyncio
async def test_Update_throws_not_found():
    repo = AsyncMock()
    repo.FindById.return_value = None
    svc = UserService(repo, mock_audit)

    with pytest.raises(NotFoundError):
        await svc.Update("bad-id", {}, 1, "admin")

@pytest.mark.asyncio
async def test_Update_throws_conflict():
    repo = AsyncMock()
    repo.FindById.return_value = mock_user()
    repo.Update.return_value = None
    svc = UserService(repo, mock_audit)

    with pytest.raises(ConflictError):
        await svc.Update("id", {}, 1, "admin")
```

## 4. API Test Pattern

```python
@pytest.mark.asyncio
async def test_get_user_success(client: AsyncClient, auth_token: str):
    resp = await client.get(
        "/api/users/123",
        headers={"Authorization": f"Bearer {auth_token}"},
    )
    assert resp.status_code == 200
    assert resp.json()["code"] == 200

@pytest.mark.asyncio
async def test_get_user_unauthorized(client: AsyncClient):
    resp = await client.get("/api/users/123")
    assert resp.status_code == 401
```

## 5. Coverage Targets

- Services: 80%
- API endpoints: 90%
- Overall: 70%
