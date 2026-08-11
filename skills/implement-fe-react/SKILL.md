---
name: implement-fe-react
description: >-
  Implement (or fix/extend) a frontend feature/module in a React + Vite +
  TypeScript project following the React MVC pattern (model.ts / controller.ts
  / view.tsx), test-first with Vitest + React Testing Library, ending with
  proper verification (lint/format/typecheck/test/E2E) and self-review.
  Use whenever the user asks to implement/add/build UI, pages, forms, tables,
  dialogs, data fetching, or asks "how to test frontend", "test after
  implementing", "what to do after implementing" — including React, Next.js,
  Vite, shadcn/MUI, Tailwind, or "implement phase" in a frontend project.
---

# implement-fe-react — Implement React Feature (MVC + Test-First + Verify)

Workflow for writing/editing frontend code in React + Vite + TypeScript
projects that follow the React MVC pattern (`modules/{domain}/model.ts` →
`controller.ts` → `view.tsx`) from the marsh rules.

## 1. Before starting — read project context

Confirm the project matches this pattern:

- `src/modules/{domain}/` with `model.ts`, `controller.ts`, `view.tsx`
- `src/config/api.ts` (Axios instance) + `src/stores/authStore.ts` (Zustand)
- `vite.config.ts`, `vitest.config.ts` (or jest), `e2e/` for Playwright
- ESLint config per `lint/typescript.md`

Read before writing: `package.json` (scripts + test runner), one existing
`modules/{domain}/` module to match actual style, `src/config/api.ts` to see
how API calls are shaped. Real code trumps generic rules.

If the project does not match (e.g. no MVC split), ask the user first.

## 2. Testing stack (answer the "what do they test with" question)

- **Unit/component tests**: Vitest + React Testing Library
  (`@testing-library/react`, `renderHook`) — run with `npm test` or `npx vitest`
- **API mocking in unit tests**: `vi.mock('axios')` with `vi.hoisted()` +
  MSW (Mock Service Worker) where configured
- **E2E**: Playwright — critical user flows only (login, main CRUD flows)

## 3. Core workflow — Test-First

1. **Red** — write tests that fail (capturing behavior you want)
2. **Green** — minimal implementation
3. **Refactor** — align with MVC import rules below

**Test patterns** (per `rules/typescript-frontend-react/04_TestingStandards.md`):

Controller test — mock the API layer:

```typescript
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

View test — mock the controller hook, not the API:

```typescript
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

Rules: `vi.hoisted()` must come before `vi.mock`; view tests mock the
controller hook; controller tests mock the API layer; use `renderHook` for
hooks. Cover success + error + empty/loading states.

## 4. While implementing — MVC patterns you must not deviate from

### React MVC structure (1 folder = 1 domain)

| File | Purpose |
|---|---|
| `model.ts` | Types + API functions + Zod (NO React imports) |
| `controller.ts` | `use{Action}` custom hook (state + logic) |
| `view.tsx` | React components (props only, no API calls) |

### Import rules (hard constraints)

| File | CAN import | CANNOT import |
|---|---|---|
| `model.ts` | axios, zod, shared types | React, components, hooks |
| `controller.ts` | model.ts, React, hooks | axios directly, view.tsx |
| `view.tsx` | controller.ts, React | axios, model.ts directly |

### Conventions

- Hooks: camelCase + `use` prefix (`useCustomerList`)
- Components: PascalCase (`CustomerListView`)
- API objects: camelCase + `Api` suffix (`customerApi`)
- Named exports, camelCase
- Zustand for auth state only; all other state lives in controllers
- Forms: React Hook Form + Zod validation

## 5. After implementing — what to do next (never skip)

Run in this order until everything passes before reporting done:

```bash
npm run typecheck    # tsc --noEmit
npm run lint         # eslint src/ — no any, explicit return types, max 40 lines/func
npm run format       # prettier --write src/
npm run test         # vitest — unit + component
# E2E (ถ้ามี setup): npx playwright test
```

If lint keeps complaining, see `lint/typescript.md`: no `any` (use `unknown`),
no `as` (use Zod `.parse()`), return types required, PascalCase public
methods / camelCase private, max 5 params, max depth 5.

## 6. Self-review checklist — before reporting back

- [ ] `model.ts` has no React imports; `view.tsx` has no API calls
- [ ] State lives in the controller hook, not scattered in components
- [ ] Forms validated with Zod (no raw `as` casts)
- [ ] No `any` — use `unknown` or Zod `.parse()`
- [ ] Types/interfaces follow naming conventions (I-prefix interfaces per lint config)
- [ ] Tests cover success + error + empty/loading states
- [ ] Controller tests mock API layer; view tests mock controller hook
- [ ] `npm run typecheck` + `lint` + `format` + `test` pass

## 7. Wrap up — summarize for the user

Brief summary: module/files created or changed, commands run and passing
(typecheck/lint/format/test), tests added, and what's left for the user
(e.g. review, running Playwright E2E with dev server, committing).
