---
trigger: always_on
---

# Go Coding Standards (Shared)

## 1. General

- **Go 1.25+** — use latest stable
- **gofmt -s** + **goimports** with `-local {module}` (local imports grouped last)
- Import order: stdlib → external → local (`{module}/...`)

## 2. Function Length

- Public functions: **max 50 lines**
- Cyclomatic complexity: **max 15**
- Decompose into private helpers when exceeding limits

## 3. Guard Clauses First

Handle errors and edge cases at the top, keep happy path unindented:

```go
func (s *service) GetProfile(ctx context.Context, id string) (*UserResponse, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        return nil, apperrors.Internal("failed to get user", err)
    }
    return toResponse(user), nil
}
```

## 4. Interface + Unexported Struct

Always return interface from constructor:

```go
type Repository interface { ... }
type repository struct { db *bun.DB }
func NewRepository(db *bun.DB) Repository { return &repository{db: db} }
```

## 5. Error Handling Rules

- **Service layer**: Use `pkg/apperrors` exclusively in public methods
- **Repo layer**: `fmt.Errorf("...: %w", err)` + sentinel errors (`var ErrNotFound = errors.New("...")`)
- **Private helpers**: `fmt.Errorf`
- **Transaction callbacks**: Use `apperrors`

## 6. No Hardcoding

- Roles → `pkg/authz/`
- Audit actions → `pkg/constants/audit.go`
- Table names → `pkg/constants/table.go`
- Error messages → inline in `apperrors.NotFound("message")`

## 7. Context Propagation

All public methods take `ctx context.Context` as first argument.

## 8. Logger Per Layer

```go
svcLogger := logger.With(zap.String("layer", "user.service"))
```

## 9. Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Files | `snake_case.go` | `user_handler.go` |
| Packages | lowercase | `auditlog`, `authz` |
| Structs | PascalCase | `UserModel` |
| Interfaces | PascalCase | `Service`, `Repository` |
| Exported funcs | PascalCase | `GetProfile` |
| Private funcs | camelCase | `validateInput` |
| Constants | PascalCase | `RoleAdmin` |
| Variables | short camelCase | `req`, `ctx`, `svc` |
| DB columns | snake_case | `user_id` |
