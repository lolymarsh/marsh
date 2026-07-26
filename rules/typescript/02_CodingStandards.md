---
trigger: always_on
---

# TypeScript Coding Standards

## 1. General

- **TypeScript strict mode** — `strict: true` in tsconfig
- **ESM** — `"type": "module"` in package.json
- Latest stable TS version
- ESLint + Prettier for formatting

## 2. TypeScript Rules

- **No `any`** — use `unknown` + type guard
- **No `as`** — use Zod `.parse()` for runtime validation
- **Return types required** on all functions
- Prefer `interface` over `type` for object shapes
- Use `type` for unions, intersections, and primitives

```typescript
// ✅ GOOD
const input = loginSchema.parse(req.body); // validates + types

// ❌ BAD
const input = req.body as LoginInput; // no validation
```

## 3. Function Length

- Public methods: **max 50 lines**
- Cyclomatic complexity: **max 15**
- Decompose into private helpers when exceeding limits

## 4. Guard Clauses First

```typescript
async Update(id: string, input: UpdateInput, adminId: string): Promise<UserResponse> {
  const existing = await this.repo.FindById(id);
  if (!existing) throw new NotFoundError('User not found');

  const updated = await this.repo.Update(id, input, input.version);
  if (!updated) throw new ConflictError('Version mismatch');

  return this.toResponse(updated);
}
```

## 5. Interface + Class Pattern

```typescript
export interface IUserRepository {
  FindById(id: string): Promise<UserEntity | null>;
  Update(id: string, data: Partial<UserEntity>, version: number): Promise<UserEntity | null>;
}

export class UserRepository implements IUserRepository {
  constructor(private db: MySql2Database) {}
  // ...
}
```

## 6. Dependency Injection

- Constructor injection — no global imports
- Wire in central `router.ts` / DI container
- Dependencies passed as constructor arguments

## 7. Imports Order

```
1. Node built-ins (fs, path, etc.)
2. Third-party (express, zod, drizzle-orm)
3. Project shared (src/shared/...)
4. Project modules (src/modules/...)
```

## 8. Frontend Rules

- `model.ts`: NO React imports (pure TypeScript)
- `view.tsx`: NO API calls (data via props only)
- `controller.ts`: NO direct DB queries
- Named exports only (no `export default` except `App`)
