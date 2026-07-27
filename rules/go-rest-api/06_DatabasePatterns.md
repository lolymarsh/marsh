---
trigger: always_on
---

# Go Database Patterns (REST API)

## 1. ORM: Uptrace Bun

- `github.com/uptrace/bun` with dialect support (PostgreSQL, MySQL, SQLite)
- Bun wraps `*sql.DB` with fluent query builder
- `bundebug` hook for query logging (development)

## 2. Entity Conventions

```go
type Model struct {
    bun.BaseModel `bun:"table:models"`
    ID            string `json:"id" bun:"id,pk"`
    Field1        string `json:"field1" bun:"field1"`
    CreatedAt     int64  `json:"created_at" bun:"created_at"`
    UpdatedAt     int64  `json:"updated_at" bun:"updated_at"`
    DeletedAt     int64  `json:"-" bun:"deleted_at,default:0"`
    Version       int    `json:"version" bun:"version,default:1"`
    TokenVersion  int    `json:"-" bun:"token_version,default:0"`
}
```

| Field | Type | Purpose |
|---|---|---|
| `ID` | `string` (UUID) | Primary key |
| `CreatedAt` | `int64` | Epoch ms |
| `UpdatedAt` | `int64` | Epoch ms |
| `DeletedAt` | `int64` (default:0) | Soft delete |
| `Version` | `int` (default:1) | Optimistic locking |
| `TokenVersion` | `int` (default:0) | Token revocation |

## 3. Multi-DB Support

```go
func InitDatabase(conf *configs.Config) (*bun.DB, error) {
    switch conf.Database.Driver {
    case "postgres": return InitDatabasePostgres(conf)
    case "mysql":    return InitDatabaseMySQL(conf)
    case "sqlite":   return InitDatabaseSQLite(conf)
    }
}
```

## 4. Migrations (Goose v3)

- SQL files embedded via `//go:embed`
- DB-specific directories: `migrations/postgres/`, `migrations/mysql/`
- Naming: `YYYYMMDDHHMMSS_description.sql`
- Auto-run on startup in `InitDatabase()`
- Each file has `-- +goose Up` / `-- +goose Down` sections

## 5. Transactions

```go
infrastructure.ExecuteInTransaction(ctx, db, func(ctx context.Context, tx bun.Tx) error {
    // pass &tx to repo methods
    if err := repo.Insert(ctx, &tx, model); err != nil {
        return apperrors.Internal("failed", err)
    }
    return nil
})
```

## 6. Query Patterns

| Operation | Pattern |
|---|---|
| SELECT single | `db.NewSelect().Model(&m).Where("id = ?", id).Where("deleted_at = 0").Scan(ctx)` |
| INSERT | `db.NewInsert().Model(m).Exec(ctx)` |
| UPDATE | `db.NewUpdate().Model(m).Column("fields").Where("id = ?", id).Where("version = ?", v).Exec(ctx)` |
| SOFT DELETE | `db.NewUpdate().Model(m).Column("deleted_at","version").Where(...).Exec(ctx)` |
| FILTER | Build query dynamically with `Where`, `Order`, `Limit`, `Offset` |
| COUNT | `.Count(ctx)` before paginated query |
