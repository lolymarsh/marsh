---
trigger: always_on
---

# API Gateway

---

## 1. Gateway Pattern

```
Client → API Gateway → Service A
                     → Service B
                     → Service C
```

**Responsibilities:**
- Route requests to the correct service
- Auth (JWT validation) + rate limiting
- Request/response transformation
- Aggregation (avoid if possible — prefer BFF or client-side)
- CORS, TLS termination

---

## 2. Auth Flow

```
Client → Gateway (validate JWT)
         ├── Gateway injects user context in header
         ├── Forward to service
         └── Service reads user from header (no re-validation)
```

**Header propagation:**
```
X-User-ID: <user_id>
X-User-Role: <role>
X-Request-ID: <uuid>
```

Service reads headers middleware:

```go
func ServiceAuthMiddleware() echo.MiddlewareFunc {
    return func(next echo.HandlerFunc) echo.HandlerFunc {
        return func(c echo.Context) error {
            userID := c.Request().Header.Get("X-User-ID")
            role := c.Request().Header.Get("X-User-Role")
            if userID == "" || role == "" {
                return response.HandleError(c, apperrors.Unauthorized("missing auth headers"))
            }
            c.Set("user_id", userID)
            c.Set("role", role)
            return next(c)
        }
    }
}
```

---

## 3. Gateway Options

| Tool | Pros | Cons |
|---|---|---|
| **Traefik** | Auto service discovery, Let's Encrypt, Middleware chain | Complex config at scale |
| **Nginx** | Fast, production-ready | No native service discovery |
| **Kong** | Plugin ecosystem, DB-backed | Heavy |
| **Custom Go Gateway** | Full control, type-safe | Must maintain yourself |

**Recommendation**: **Traefik** for Kubernetes or Docker Compose.

---

## 4. What NOT to do in Gateway

- ❌ Don't write business logic in gateway
- ❌ Don't aggregate data in gateway (use BFF or client-side instead)
- ❌ Don't run cross-service transactions through gateway
- ❌ Don't share DBs with any service
