---
trigger: always_on
---

# Service Layer Patterns

## 1. Interface + Implementation

```go
type Service interface {
    GetProfile(ctx context.Context, userID, role string) (*UserResponse, error)
    PatchProfile(ctx context.Context, req *PatchProfileRequest) (*UserResponse, error)
}

type service struct {
    repo         Repository
    auditService auditlog.Service
    logger       *zap.Logger
}

func NewService(repo Repository, logger *zap.Logger, auditService auditlog.Service) Service {
    svcLogger := logger.With(zap.String("layer", "module.service"))
    return &service{repo: repo, auditService: auditService, logger: svcLogger}
}
```

## 2. CRUD Pattern

```go
func (s *service) PatchProfile(ctx context.Context, req *PatchProfileRequest) (*UserResponse, error) {
    // Guard: auth/access check
    if req.UserID != "" && !authz.IsAdmin(role) {
        return nil, apperrors.Forbidden("admin only")
    }

    // Get existing
    user, err := s.repo.GetByID(ctx, targetID)
    if err != nil {
        return nil, apperrors.NotFound("user not found")
    }

    // Deep copy before mutation
    oldUser := *user

    // Apply changes
    if req.FirstName != nil {
        user.FirstName = *req.FirstName
    }

    // Update with optimistic locking
    if err := s.repo.Update(ctx, user); err != nil {
        return nil, err // already apperrors.Conflict from repo
    }

    // Audit log
    s.auditService.Log(ctx, &auditlog.Entry{Action: constants.AuditActionUpdate})

    return toResponse(user), nil
}
```

## 3. Transaction Pattern

```go
func (s *service) CreateOrder(ctx context.Context, req *CreateOrderRequest) error {
    return database.ExecuteInTransaction(ctx, s.db, func(ctx context.Context, tx bun.Tx) error {
        if err := s.repo.InsertOrder(ctx, &tx, order); err != nil {
            return apperrors.Internal("failed to insert order", err)
        }
        if err := s.repo.InsertItems(ctx, &tx, items); err != nil {
            return apperrors.Internal("failed to insert items", err)
        }
        return nil // auto-commit
    })
}
```

## 4. Filter/Pagination Pattern

```go
func (s *service) FilterUsers(ctx context.Context, req *request.FilterRequest) ([]*UserResponse, *response.PaginationResponse, error) {
    users, pagination, err := s.repo.FilterUsers(ctx, req)
    if err != nil {
        return nil, nil, apperrors.Internal("failed to filter users", err)
    }
    return mapToResponses(users), pagination, nil
}
```

## 5. Rules

- Guard clauses first (auth → existence → mutate)
- Deep copy before mutation: `oldUser := *user`
- Audit log after every mutation
- Public funcs ≤ 50 lines
- Interface + unexported struct
