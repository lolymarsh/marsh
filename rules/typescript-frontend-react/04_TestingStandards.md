---
trigger: always_on
---

# Frontend Testing Standards

## 1. Unit Tests (Vitest + Testing Library)

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

## 2. E2E Tests (Playwright)

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

## 3. Test Rules

- Use `vi.hoisted()` for API mocks (must be before `vi.mock`)
- View tests mock the controller hook
- Controller tests mock the API layer
- E2E tests cover critical user flows only (login, CRUD main flows)
- Use `@testing-library/react` for component rendering
- Use `renderHook` for controller hook tests
