---
trigger: always_on
---

# Testing Standards

## 1. Test Pyramid

| Level | Tool | Scope |
|---|---|---|
| Unit | Jest / Vitest | Services, repos (mocked), schemas |
| Integration | Supertest + Testcontainers | API endpoints with real DB |
| E2E | Playwright | Critical user flows |

## 2. Backend Unit Tests (Jest)

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

## 3. Frontend Unit Tests (Vitest + Testing Library)

```typescript
// Controller test
const mockApi = vi.hoisted(() => ({ post: vi.fn(), get: vi.fn() }));
vi.mock('axios', () => ({ default: { create: () => mockApi } }));

describe('useCustomerList', () => {
  it('fetches customers on mount', async () => {
    mockApi.post.mockResolvedValue({ data: { data: { data: [], pagination: {} } } });
    const { result } = renderHook(() => useCustomerList());
    await act(async () => { await new Promise(r => setTimeout(r, 0)); });
    expect(result.current.customers).toEqual([]);
  });
});
```

```typescript
// View test (mocked controller)
vi.mock('./controller', () => ({
  useCustomerList: () => ({
    customers: [], loading: false, error: null,
    pagination: null, refetch: vi.fn(), setSearch: vi.fn(), setPage: vi.fn(),
  }),
}));

it('renders empty state', () => {
  render(<CustomerListRoute />);
  expect(screen.getByText('No customers found')).toBeInTheDocument();
});
```

## 4. E2E Tests (Playwright)

```typescript
test('login flow', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="username"]', 'admin');
  await page.fill('[name="password"]', 'admin123');
  await page.click('button[type="submit"]');
  await page.waitForURL('/');
  await expect(page.getByText('Dashboard')).toBeVisible();
});
```

## 5. Coverage Targets

- Backend services: 80%
- Backend API endpoints: 90%
- Frontend: 70% (statements, lines)
- E2E: 5-8 critical scenarios
