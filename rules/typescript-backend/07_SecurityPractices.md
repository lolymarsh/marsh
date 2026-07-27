---
trigger: always_on
---

# Security Practices (Backend)

## 1. Authentication

- JWT (`jsonwebtoken`) with strong secret via env var
- Redis sessions: `session:{userId}` — validate on each request
- Session invalidation on logout/deactivation (delete Redis key)
- `Authorization: Bearer <token>` header
- Auth middleware: curried `auth(roles?)` for per-route access control

## 2. Authorization

- Role-based: `ADMIN > STAFF > USER`
- Middleware checks role against allowed roles list
- Service layer re-checks authorization for sensitive operations

```typescript
router.get('/admin/users', auth('ADMIN'), handler.FindFiltered);
```

## 3. Input Validation

- **Always use Zod** — never trust raw request data
- `.parse()` validates + types in one step
- Never use `as` for type casting from request body

```typescript
const input = loginSchema.parse(req.body);  // ✅ validates at runtime
const input = req.body as LoginInput;       // ❌ no validation
```

## 4. Password Security

- bcrypt with cost factor 12
- Never return `passwordHash` in responses
- No password logging

## 5. Soft Delete

- `deletedAt: Date | null` — never hard delete
- Always filter `deletedAt = null` in queries
- Soft-deleted records excluded from all list/get operations

## 6. SQL Injection Prevention

- Drizzle ORM parameterized queries (no raw SQL string concatenation)
- If raw SQL needed: use `sql\`...\`` template literals (Drizzle-safe)

## 7. Backend API Security

- Request timeout (Axios: 30s, Express: configurable)
- Rate limiting at reverse proxy level (Nginx/Traefik) — see devops rules
- CORS handled at reverse proxy level — see devops rules
- No sensitive data in logs (passwords, tokens)

## 8. Audit Logging

Audit failures are fire-and-forget (never fail the main operation):

```typescript
setImmediate(async () => {
  try {
    await this.repo.Insert(doc);
  } catch (err: unknown) {
    logger.error({ err }, 'Failed to insert audit log');
  }
});
```
