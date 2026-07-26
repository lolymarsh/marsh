---
trigger: always_on
---

# Testing Standards

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
    app      *echo.Echo     // or httptest.Server for ConnectRPC
    client   *testutil.TestClient
    fixtures *testutil.Fixtures
}

func (s *Suite) SetupSuite() {
    // Load test config, setup DB, run migrations, wire DI
}

func (s *Suite) SetupTest() {
    // Truncate tables, create fixtures, generate tokens
}

func (s *Suite) TestMethod_Success() {
    resp := s.client.GET("/api/...").Do()
    s.Equal(http.StatusOK, resp.StatusCode())
}

func TestSuite(t *testing.T) {
    suite.Run(t, new(Suite))
}
```

## 3. No Mocks

- **Integration tests only** — real DB via testcontainers
- No mocking for repo/service
- Unit tests only for pure functions (e.g., snowflake, string utils)

## 4. Assertions

```go
// Echo (REST)
resp := s.client.GET("/api/profile").Do()
s.Equal(http.StatusOK, resp.StatusCode())
var result UserResponse
s.NoError(resp.BindJSON(&result))
s.Equal("test@email.com", result.Email)

// ConnectRPC
var connectErr *connect.Error
require.ErrorAs(t, err, &connectErr)
assert.Equal(t, connect.CodeUnauthenticated, connectErr.Code())
```

## 5. Test Helpers

| File | Purpose |
|---|---|
| `testutil/database.go` | SetupPostgresDB, SetupMySQLDB, SetupSQLiteDB |
| `testutil/fixtures.go` | CreateUser, CreateAuditLog, etc. |
| `testutil/http_client.go` | TestClient (GET/POST/PUT/DELETE) |
| `testutil/auth.go` | GenerateTestToken helpers |
| `testutil/config.go` | LoadTestConfig |
