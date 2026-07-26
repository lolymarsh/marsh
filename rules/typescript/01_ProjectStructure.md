---
trigger: always_on
---

# Project Structure (TypeScript Full-Stack)

## 1. Top-Level Layout

```
{project}/
├── AGENTS.md                        # AI rules (auto-read by OpenCode)
├── docker-compose.yml               # MySQL/MongoDB/Redis/RabbitMQ
├── spec/                            # Spec docs (plan.md + schema.md)
│
├── backend/                         # Express/Fastify + TypeScript
│   ├── package.json                 # "type": "module"
│   ├── tsconfig.json
│   ├── src/
│   │   ├── app.ts                   # Entry point
│   │   ├── router.ts                # DI composition + route mounting
│   │   ├── config/
│   │   │   ├── database.ts          # Drizzle ORM client
│   │   │   ├── logger.ts            # Pino logger
│   │   │   ├── redis.ts             # ioredis
│   │   │   └── rabbitmq.ts          # amqplib
│   │   ├── modules/{domain}/
│   │   │   ├── entity.ts            # TypeScript interfaces
│   │   │   ├── schema.ts            # Zod validation schemas
│   │   │   ├── repo.ts              # Interface + implementation
│   │   │   ├── service.ts           # Interface + implementation
│   │   │   ├── handler.ts           # Request handler
│   │   │   ├── route.ts             # Router factory
│   │   │   └── *.test.ts            # Co-located tests
│   │   ├── shared/
│   │   │   ├── errors/AppError.ts   # Error hierarchy
│   │   │   ├── response/            # Response helpers
│   │   │   ├── middleware/          # Auth, audit, validator
│   │   │   └── pagination/          # Zod schemas + helpers
│   │   └── workers/                 # RabbitMQ consumers
│   └── drizzle/                     # Auto-generated migrations
│
├── frontend/                        # React 19 + Vite + MUI
│   ├── package.json
│   ├── vite.config.ts
│   ├── src/
│   │   ├── main.tsx / App.tsx
│   │   ├── router.tsx               # Route components (composer)
│   │   ├── config/api.ts            # Axios instance
│   │   ├── stores/authStore.ts      # Zustand (auth only)
│   │   ├── modules/{domain}/
│   │   │   ├── model.ts             # Types + API calls (NO React)
│   │   │   ├── controller.ts        # useXxx hook
│   │   │   ├── view.tsx             # Components (props only)
│   │   │   └── *.test.ts/tsx
│   │   ├── shared/
│   │   │   ├── components/          # Reusable components
│   │   │   ├── hooks/               # Shared hooks
│   │   │   └── pages/               # Error pages
│   │   └── index.css                # Tailwind base
│   └── e2e/                         # Playwright
└── database/                        # Seed scripts
```

---

## 2. Backend Module Structure

| File | Purpose |
|---|---|
| `entity.ts` | TypeScript interfaces (DB row shapes) |
| `schema.ts` | Zod validation schemas + inferred types |
| `repo.ts` | `I{Name}Repository` interface + class impl |
| `service.ts` | `I{Name}Service` interface + class impl |
| `handler.ts` | Express handler class |
| `route.ts` | `Register{Module}Routes()` factory |

---

## 3. Frontend Module Structure (React MVC)

| File | Purpose |
|---|---|
| `model.ts` | Types + API functions + Zod (NO React imports) |
| `controller.ts` | `use{Action}` custom hook (state + logic) |
| `view.tsx` | React components (props only, no API calls) |

---

## 4. Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Files (backend) | kebab-case | `repo.ts`, `service.ts` |
| Files (frontend) | kebab-case | `controller.ts`, `view.tsx` |
| Classes | PascalCase | `UserRepository` |
| Interfaces | PascalCase + I prefix | `IUserRepository` |
| Entity types | PascalCase + Entity | `UserEntity` |
| Response types | PascalCase + Response | `UserResponse` |
| Public methods | PascalCase | `FindById`, `Update` |
| Private methods | camelCase | `toResponse` |
| Hooks | camelCase + use | `useCustomerList` |
| Components | PascalCase | `CustomerListView` |
| Schema vars | camelCase | `loginSchema` |
| API objects | camelCase + Api | `customerApi` |
