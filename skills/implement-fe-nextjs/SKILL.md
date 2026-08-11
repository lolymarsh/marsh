---
name: implement-fe-nextjs
description: >-
  Implement (or fix/extend) a feature in a Next.js App Router project (React
  19, RSC, Tailwind, shadcn/MUI) — pages, layouts, route handlers, server
  actions, data fetching, forms, dashboards — test-first (Vitest + React
  Testing Library, Playwright E2E), ending with proper verification
  (lint/typecheck/build/test) and self-review. Use whenever the user asks to
  implement/build/add UI, pages, API routes, server actions in a Next.js
  project, or asks "what to do after implementing", "how to test Next.js" —
  including Next.js, App Router, RSC, server components, use client,
  Tailwind, shadcn.
---

# implement-fe-nextjs — Implement Next.js Feature (App Router + Verify)

Workflow for writing/editing Next.js (App Router) features following the
marsh conventions: server components by default, thin client boundary,
data fetching in RSC, forms with React Hook Form + Zod, E2E with Playwright.

## 1. Before starting — read project context

- `next.config.ts`, `package.json` (scripts, Next version), `app/` directory
  structure, `components.json` (shadcn) if present
- Read `app/` layout and one existing feature route to match actual style
- Confirm App Router (vs Pages Router) — this skill covers App Router

If the project is Pages Router or a different structure, ask the user first.

## 2. Testing stack

- **Unit/component**: Vitest + React Testing Library — client components
  (forms, tables, dialogs); server components are not unit-tested with RTL —
  verify them via build + E2E instead
- **API mocking**: `vi.mock('axios')` + `vi.hoisted()`, or MSW
- **E2E**: Playwright — critical flows (login, main CRUD)

## 3. Core workflow — Test-First

1. **Red** — write failing tests (client components) or define the route/UI
   contract first
2. **Green** — minimal implementation
3. **Refactor** — align with the RSC conventions below

Test patterns follow `rules/typescript-frontend-react/04_TestingStandards.md`:
controller tests mock the API layer (`vi.hoisted()` before `vi.mock`), view
tests mock hooks, `renderHook` for hooks, cover success + error + empty/loading.

## 4. While implementing — App Router conventions

### Component boundary (hard constraint)

- **Default to server components.** Only add `"use client"` when the file
  needs hooks, event handlers, or browser APIs
- Keep `"use client"` files thin — pass data down as props, don't fetch in
  them if avoidable
- Data fetching lives in server components / RSC / route handlers, not in
  client components

### Structure

```
app/
├── layout.tsx              # Shared shell (Root layout)
├── (route)/
│   ├── page.tsx            # Server component — fetch + render
│   ├── loading.tsx         # Suspense fallback
│   ├── error.tsx           # Error boundary
│   ├── route.ts            # API route (thin — proxy to backend, validate with Zod)
│   └── _components/        # Local components (or components/)
├── actions/                # Server actions (mutations via forms)
```

- `route.ts` / server actions stay thin: validate input with Zod → call the
  backend → return normalized data. Business logic does not live in Next.js
- Forms: React Hook Form + Zod; use server actions or API routes, never
  uncontrolled raw DOM forms without validation

## 5. After implementing — what to do next (never skip)

Run in this order until everything passes:

```bash
npm run lint           # eslint (eslint-config-next)
npx tsc --noEmit       # typecheck
npm run build          # CRITICAL — catches RSC/SSR errors, hydration issues,
                       # missing "use client" boundaries that lint/tests miss
npm run test           # vitest — unit + component
# E2E: npx playwright test  (needs dev server)
```

`npm run build` is the most important Next.js verification step — do not
report done until it passes.

## 6. Self-review checklist — before reporting back

- [ ] `"use client"` added only where truly needed — no fetch in client components
- [ ] Route handlers / server actions validate input with Zod
- [ ] Forms use React Hook Form + Zod
- [ ] `loading.tsx` / `error.tsx` present for data-loading routes
- [ ] No secrets/env vars in client components (only `NEXT_PUBLIC_*`)
- [ ] No `any` — use `unknown` / Zod `.parse()`
- [ ] Tests cover success + error + empty/loading states
- [ ] `lint` + `tsc --noEmit` + `build` + `test` pass

## 7. Wrap up — summarize for the user

Brief summary: routes/components/files created or changed, commands run and
passing, tests added, what's left for the user (review, E2E with dev server,
committing).
