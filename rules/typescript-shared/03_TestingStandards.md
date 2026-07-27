---
trigger: always_on
---

# Testing Standards (Shared)

## 1. Test Pyramid

| Level | Tool | Scope |
|---|---|---|
| Unit | Jest / Vitest | Services, repos (mocked), schemas |
| Integration | Supertest + Testcontainers | API endpoints with real DB |
| E2E | Playwright | Critical user flows |

## 2. Coverage Targets

- Backend services: 80%
- Backend API endpoints: 90%
- Frontend: 70% (statements, lines)
- E2E: 5-8 critical scenarios

## 3. General Rules

- Test file co-located with source: `*.test.ts` or `*.test.tsx`
- One `describe` block per function/method
- `it` descriptions must be specific and actionable
- Use `beforeEach` for setup, avoid `beforeAll` unless necessary
- Mock external dependencies, never hit real APIs/DB in unit tests
- Use `vi.hoisted()` (Vitest) or `jest.fn()` (Jest) for mocks
