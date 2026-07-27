---
trigger: always_on
---

# Go Testing Standards (Shared)

## 1. Framework

- **testify** (`suite.Suite`, `assert`, `require`)
- Build tag: `//go:build integration` for integration tests
- Test package: external (`package user_test`)

## 2. Test Structure

```go
//go:build integration

type Suite struct {
    suite.Suite
    db       *testutil.TestDB
    fixtures *testutil.Fixtures
}

func (s *Suite) SetupSuite() {
    // Load test config, setup DB, run migrations
}

func (s *Suite) SetupTest() {
    // Truncate tables, create fixtures
}

func TestSuite(t *testing.T) {
    suite.Run(t, new(Suite))
}
```

## 3. Unit Tests for Pure Functions

```go
func TestCalculatePagination(t *testing.T) {
    tests := []struct {
        name     string
        page     int
        pageSize int
        total    int
        want     PaginationResponse
    }{
        {"first page", 1, 10, 25, PaginationResponse{TotalPage: 3, HasNextPage: true}},
        {"last page", 3, 10, 25, PaginationResponse{TotalPage: 3, HasPreviousPage: true}},
        {"empty", 1, 10, 0, PaginationResponse{TotalPage: 0}},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := CalculatePagination(tt.page, tt.pageSize, tt.total)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

## 4. Test Helpers

| File | Purpose |
|---|---|
| `testutil/database.go` | SetupPostgresDB, SetupMySQLDB, SetupSQLiteDB |
| `testutil/fixtures.go` | CreateUser, CreateAuditLog, etc. |
| `testutil/config.go` | LoadTestConfig |
