---
trigger: always_on
---

# Frontend Project Structure (TypeScript)

## 1. Frontend Layout

```
frontend/
├── package.json
├── vite.config.ts
├── src/
│   ├── main.tsx / App.tsx
│   ├── router.tsx               # Route components (composer)
│   ├── config/api.ts            # Axios instance
│   ├── stores/authStore.ts      # Zustand (auth only)
│   ├── modules/{domain}/
│   │   ├── model.ts             # Types + API calls (NO React)
│   │   ├── controller.ts        # useXxx hook
│   │   ├── view.tsx             # Components (props only)
│   │   └── *.test.ts/tsx
│   ├── shared/
│   │   ├── components/          # Reusable components
│   │   ├── hooks/               # Shared hooks
│   │   └── pages/               # Error pages
│   └── index.css                # Tailwind base
└── e2e/                         # Playwright
```

## 2. Module Structure (React MVC)

| File | Purpose |
|---|---|
| `model.ts` | Types + API functions + Zod (NO React imports) |
| `controller.ts` | `use{Action}` custom hook (state + logic) |
| `view.tsx` | React components (props only, no API calls) |

## 3. Layer Rules

| File | CAN import | CANNOT import |
|---|---|---|
| `model.ts` | axios, zod, shared types | React, components, hooks |
| `controller.ts` | model.ts, React, hooks | axios directly, view.tsx |
| `view.tsx` | controller.ts, React | axios, model.ts directly |

## 4. Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Model file | `model.ts` | `customer/model.ts` |
| Controller file | `controller.ts` | `customer/controller.ts` |
| View file | `view.tsx` | `customer/view.tsx` |
| Hooks | camelCase + use | `useCustomerList` |
| Components | PascalCase | `CustomerListView` |
| API objects | camelCase + Api | `customerApi` |
| Named exports | camelCase | `export const useCustomerList` |

## 5. Tech Stack

- React 19 + Vite + TypeScript
- MUI (Material UI) for components
- Zustand for auth state only
- Axios for API calls
- Tailwind CSS for utility styling
- React Hook Form + Zod for forms
- Playwright for E2E tests
