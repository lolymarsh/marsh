---
trigger: always_on
---

# Backend Testing Standards

## 1. Unit Tests (Jest)

```typescript
// Service test (mocked repo)
describe('UserService', () => {
  let svc: UserService;
  let repo: jest.Mocked<IUserRepository>;

  beforeEach(() => {
    repo = { FindById: jest.fn(), Update: jest.fn(), /* ... */ };
    svc = new UserService(repo, mockRedis, mockAudit);
  });

  it('throws NotFoundError when user not found', async () => {
    repo.FindById.mockResolvedValue(null);
    await expect(svc.Update('bad-id', input, 'admin'))
      .rejects.toThrow(NotFoundError);
  });

  it('throws ConflictError on version mismatch', async () => {
    repo.FindById.mockResolvedValue(mockUser);
    repo.Update.mockResolvedValue(null);
    await expect(svc.Update('id', input, 'admin'))
      .rejects.toThrow(ConflictError);
  });
});
```

```typescript
// Handler test (mocked service)
import { mockReqRes } from './test-utils';

describe('UserHandler', () => {
  it('returns 200 on successful login', async () => {
    svc.Login.mockResolvedValue(mockResult);
    const { req, res } = mockReqRes({ body: { username: 'admin', password: 'pass' } });
    await handler.Login(req, res);
    expect(res.status).toHaveBeenCalledWith(200);
  });
});
```

## 2. Integration Tests (Supertest + Testcontainers)

```typescript
describe('POST /auth/login', () => {
  it('returns token on valid credentials', async () => {
    const res = await request(app)
      .post('/auth/login')
      .send({ username: 'admin', password: 'admin123' })
      .expect(200);

    expect(res.body.data.token).toBeDefined();
  });

  it('returns 401 on invalid credentials', async () => {
    await request(app)
      .post('/auth/login')
      .send({ username: 'admin', password: 'wrong' })
      .expect(401);
  });
});
```

## 3. Test Rules

- Mock all external dependencies (Redis, RabbitMQ, external APIs)
- Use Testcontainers for integration tests with real DB
- Test error paths (not found, conflict, validation errors)
- Test authorization (wrong role → 403)
