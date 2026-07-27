---
trigger: always_on
---

# Go Error Handling Patterns (Shared)

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

| Function | Code | When to use |
|---|---|---|
| `apperrors.NotFound(msg)` | NOT_FOUND | Resource not found |
| `apperrors.AlreadyExists(msg)` | ALREADY_EXISTS | Duplicate resource |
| `apperrors.BadRequest(msg)` | BAD_REQUEST | Invalid input |
| `apperrors.Unauthorized(msg)` | UNAUTHORIZED | Not authenticated |
| `apperrors.Forbidden(msg)` | FORBIDDEN | Not authorized |
| `apperrors.Inactive(msg)` | INACTIVE | Account disabled |
| `apperrors.Internal(msg, err)` | INTERNAL | Unexpected error |
| `apperrors.Conflict(msg, data)` | CONFLICT | Version mismatch |

## 2. Check Functions

```go
apperrors.IsNotFound(err)
apperrors.IsAlreadyExists(err)
apperrors.IsUnauthorized(err)
apperrors.IsForbidden(err)
apperrors.IsConflict(err)
```

## 3. Per-Layer Rules

| Layer | Error Type | Wrapping Style |
|---|---|---|
| **Repo public** | Sentinel errors | `fmt.Errorf("...: %w", err)` |
| **Repo private** | `fmt.Errorf` | `fmt.Errorf("...: %w", err)` |
| **Service public** | **`apperrors` ONLY** | `apperrors.NotFound("...")` |
| **Service private** | `fmt.Errorf` | `fmt.Errorf("...: %w", err)` |
| **Transaction cb** | `apperrors` | `apperrors.Internal("...", err)` |

## 4. Sentinel Errors

```go
// In repo layer
var ErrNotFound = errors.New("resource not found")
var ErrConflict = errors.New("version mismatch")

// Map sql.ErrNoRows → sentinel
if errors.Is(err, sql.ErrNoRows) {
    return nil, ErrNotFound
}
```
