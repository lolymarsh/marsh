---
trigger: always_on
---

# Backend Project Structure (TypeScript)

## 1. Backend Layout

```
backend/
├── package.json                 # "type": "module"
├── tsconfig.json
├── src/
│   ├── app.ts                   # Entry point
│   ├── router.ts                # DI composition + route mounting
│   ├── config/
│   │   ├── database.ts          # Drizzle ORM client
│   │   ├── logger.ts            # Pino logger
│   │   ├── redis.ts             # ioredis
│   │   └── rabbitmq.ts          # amqplib
│   ├── modules/{domain}/
│   │   ├── entity.ts            # TypeScript interfaces (DB row shapes)
│   │   ├── schema.ts            # Zod validation schemas + inferred types
│   │   ├── repo.ts              # Interface + implementation
│   │   ├── service.ts           # Interface + implementation
│   │   ├── handler.ts           # Request handler
│   │   ├── route.ts             # Router factory
│   │   └── *.test.ts            # Co-located tests
│   ├── shared/
│   │   ├── errors/AppError.ts   # Error hierarchy
│   │   ├── response/            # Response helpers (SendSuccess, SendError)
│   │   ├── middleware/          # Auth, audit, validator
│   │   └── pagination/          # Zod schemas + helpers
│   └── workers/                 # RabbitMQ consumers
└── drizzle/                     # Auto-generated migrations
```

## 2. Module File Purposes

| File | Purpose |
|---|---|
| `entity.ts` | TypeScript interfaces (DB row shapes) |
| `schema.ts` | Zod validation schemas + inferred types |
| `repo.ts` | `I{Name}Repository` interface + class impl |
| `service.ts` | `I{Name}Service` interface + class impl |
| `handler.ts` | Express handler class |
| `route.ts` | `Register{Module}Routes()` factory |

## 3. DI Wiring (router.ts)

```typescript
// router.ts — Central DI composition
export function createRouter(db: MySql2Database, redis: Redis): Router {
  const userRepo = new UserRepository(db);
  const userService = new UserService(userRepo, redis);
  const userHandler = new UserHandler(userService);
  const auth = CreateAuthMiddleware(redis);

  const router = Router();
  router.use('/api', RegisterUserRoutes(userHandler, auth));
  return router;
}
```
