---
trigger: always_on
---

# Repository Layer Patterns

## 1. Interface + Implementation

```go
type Repository interface {
    GetByID(ctx context.Context, id string) (*Model, error)
    Insert(ctx context.Context, m *Model) error
    Update(ctx context.Context, m *Model) error
    SoftDelete(ctx context.Context, m *Model) error
}

type repository struct {
    db *bun.DB
}

func NewRepository(db *bun.DB) Repository {
    return &repository{db: db}
}

var ErrNotFound = errors.New("resource not found")
```

## 2. SELECT

```go
func (r *repository) GetByID(ctx context.Context, id string) (*Model, error) {
    var m Model
    err := r.db.NewSelect().Model(&m).
        Where("id = ?", id).
        Where("deleted_at = 0").
        Scan(ctx)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrNotFound
        }
        return nil, fmt.Errorf("failed to get by id: %w", err)
    }
    return &m, nil
}
```

## 3. INSERT

```go
func (r *repository) Insert(ctx context.Context, m *Model) error {
    _, err := r.db.NewInsert().Model(m).Exec(ctx)
    if err != nil {
        return fmt.Errorf("failed to insert: %w", err)
    }
    return nil
}
```

## 4. UPDATE with Optimistic Locking

```go
func (r *repository) Update(ctx context.Context, m *Model) error {
    currentVersion := m.Version
    m.Version++
    result, err := r.db.NewUpdate().Model(m).
        Column("field1", "field2", "version").
        Where("id = ?", m.ID).
        Where("version = ?", currentVersion).
        Where("deleted_at = 0").
        Exec(ctx)
    if err != nil {
        return fmt.Errorf("failed to update: %w", err)
    }
    rows, _ := result.RowsAffected()
    if rows == 0 {
        return apperrors.Conflict("version mismatch, please retry", nil)
    }
    return nil
}
```

## 5. SOFT DELETE

```go
func (r *repository) SoftDelete(ctx context.Context, m *Model) error {
    result, err := r.db.NewUpdate().Model(m).
        Column("deleted_at", "version").
        Where("id = ?", m.ID).
        Where("version = ?", m.Version).
        Where("deleted_at = 0").
        Exec(ctx)
    if err != nil {
        return fmt.Errorf("failed to soft delete: %w", err)
    }
    rows, _ := result.RowsAffected()
    if rows == 0 {
        return apperrors.Conflict("version mismatch", nil)
    }
    return nil
}
```

## 6. Filter with Pagination

Separate `Count` and `Find` into different methods — shared `buildFilterQuery` avoids duplication:

```go
func (r *repository) buildFilterQuery(req *FilterRequest) *bun.SelectQuery {
    query := r.db.NewSelect().Model(&models).
        Where("deleted_at = 0")

    if req.Search != "" {
        query = query.Where("name ILIKE ?", "%"+req.Search+"%")
    }

    return query
}

func (r *repository) CountByFilter(ctx context.Context, req *FilterRequest) (int, error) {
    return r.buildFilterQuery(req).Count(ctx)
}

func (r *repository) FindByFilter(ctx context.Context, req *FilterRequest) ([]Model, error) {
    query := r.buildFilterQuery(req)

    err := query.Order("created_at DESC").
        Limit(req.PageSize).
        Offset((req.Page - 1) * req.PageSize).
        Scan(ctx, &models)

    return models, err
}
```

## 7. Rules

- Error wrapping: `fmt.Errorf("...: %w", err)` — NOT `apperrors`
- Sentinel errors: `var ErrXxx = errors.New("...")`
- Map `sql.ErrNoRows` → sentinel error
- Always filter `deleted_at = 0`
- Check rows affected for optimistic locking
- Transactions: `infrastructure.ExecuteInTransaction(ctx, db, func(ctx, tx bun.Tx) error)`
