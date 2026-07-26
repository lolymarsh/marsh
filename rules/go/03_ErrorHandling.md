---
trigger: always_on
---

# Error Handling Patterns

## 1. AppError System

Two-tier error system:

**Tier 1 — Domain errors** (`pkg/apperrors/errors.go`):

```go
type AppError struct {
    Code    string  // "NOT_FOUND", "FORBIDDEN", etc.
    Message string
    Err     error   // wrapped original
    Data    any     // optional payload (conflict data)
}
```

**Factory functions:**

| Function | HTTP Status | When to use |
|---|---|---|
| `apperrors.NotFound(msg)` | 404 | Resource not found |
| `apperrors.AlreadyExists(msg)` | 409 | Duplicate resource |
| `apperrors.BadRequest(msg)` | 400 | Invalid input |
| `apperrors.Unauthorized(msg)` | 401 | Not authenticated |
| `apperrors.Forbidden(msg)` | 403 | Not authorized |
| `apperrors.Inactive(msg)` | 403 | Account disabled |
| `apperrors.Internal(msg, err)` | 500 | Unexpected error |
| `apperrors.Conflict(msg, data)` | 409 | Version mismatch |

## 2. Per-Layer Rules

| Layer | Error Type | Wrapping Style |
|---|---|---|
| **Repo public** | Sentinel errors | `fmt.Errorf("...: %w", err)` |
| **Repo private** | `fmt.Errorf` | `fmt.Errorf("...: %w", err)` |
| **Service public** | **`apperrors` ONLY** | `apperrors.NotFound("...")` |
| **Service private** | `fmt.Errorf` | `fmt.Errorf("...: %w", err)` |
| **Handler** | Convert from service | `response.HandleError(c, err)` |
| **Transaction cb** | `apperrors` | `apperrors.Internal("...", err)` |

## 3. ConnectRPC Error Mapping

```go
func FromAppError(err error) error {
    var appErr *apperrors.AppError
    if !errors.As(err, &appErr) {
        return connect.NewError(connect.CodeInternal, err)
    }
    switch appErr.Code {
    case "NOT_FOUND":
        return connect.NewError(connect.CodeNotFound, errors.New(appErr.Message))
    case "ALREADY_EXISTS":
        return connect.NewError(connect.CodeAlreadyExists, errors.New(appErr.Message))
    // ...
    }
}
```

## 4. Check Functions

```go
apperrors.IsNotFound(err)
apperrors.IsAlreadyExists(err)
apperrors.IsUnauthorized(err)
apperrors.IsForbidden(err)
apperrors.IsConflict(err)
```
