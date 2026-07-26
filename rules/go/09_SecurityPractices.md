---
trigger: always_on
---

# Security Practices

## 1. Authentication

- JWT with HMAC-SHA256 (golang-jwt v5)
- Token validation: verify signature + check expiry + validate `token_version` against Redis/DB
- Session-based invalidation via `token_version` column
- `Authorization: Bearer <token>` header

## 2. Authorization

- 3-tier RBAC: `ADMIN > STAFF > USER`
- Roles defined in `pkg/authz/roles.go`
- Permission checks in service layer (guard clauses)

```go
if !authz.IsAdmin(role) {
    return nil, apperrors.Forbidden("admin access required")
}
```

## 3. Input Validation

- `go-playground/validator` struct tags
- Validate in handler after bind: `h.validate.StructCtx(ctx, req)`
- Parameterized queries (Bun ORM — no raw SQL injection risk)

## 4. Password Security

- bcrypt: `golang.org/x/crypto` — `HashPassword`, `CheckPasswordHash`
- Never return `password_hash` in JSON responses (`json:"-"`)
- No password logging

## 5. Soft Delete

- `DeletedAt int64` — never hard delete
- Always filter `deleted_at = 0` in queries
- Soft-deleted records excluded from all list/get operations

## 6. Sensitive Data

- `password_hash`, `token_version` → `json:"-"`
- Don't expose internal error details to client (use generic messages in apperrors)
- Log full errors server-side, return generic messages

## 7. Additional

- Request timeout via `ContextTimeout` middleware
- Cloudflare Turnstile for bot protection (register/login)
- Rate limiting at reverse proxy level (Nginx)
