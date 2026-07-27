---
trigger: always_on
---

# Go REST API Testing Standards

## 1. Integration Tests (testify + testcontainers)

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

func (s *Suite) TestGetProfile_Success() {
    resp := s.client.GET("/api/profile").WithToken(s.userToken).Do()
    s.Equal(http.StatusOK, resp.StatusCode())
    var result UserResponse
    s.NoError(resp.BindJSON(&result))
    s.Equal("test@email.com", result.Email)
}

func (s *Suite) TestGetProfile_NotFound() {
    resp := s.client.GET("/api/profile/invalid-id").WithToken(s.adminToken).Do()
    s.Equal(http.StatusNotFound, resp.StatusCode())
}

func TestSuite(t *testing.T) {
    suite.Run(t, new(Suite))
}
```

## 2. ConnectRPC Tests

```go
func (s *Suite) TestGetProfile_ConnectRPC() {
    client := userv1connect.NewUserServiceClient(s.httpClient, s.server.URL)
    resp, err := client.GetProfile(context.Background(), connect.NewRequest(&userv1.GetProfileRequest{}))
    s.NoError(err)
    s.Equal("test@email.com", resp.Msg.User.Email)
}

func (s *Suite) TestGetProfile_Unauthorized() {
    _, err := client.GetProfile(context.Background(), connect.NewRequest(&userv1.GetProfileRequest{}))
    var connectErr *connect.Error
    require.ErrorAs(s.T(), err, &connectErr)
    assert.Equal(s.T(), connect.CodeUnauthenticated, connectErr.Code())
}
```

## 3. Test Rules

- **Integration tests only** — real DB via testcontainers
- No mocking for repo/service
- Unit tests only for pure functions (e.g., snowflake, string utils, pagination)
- Test both success and error paths
- Test authorization (wrong role → 403)
