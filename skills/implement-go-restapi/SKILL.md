---
name: implement-go-restapi
description: >-
  Implement (or fix/extend) a Go REST API module — Echo + Bun or ConnectRPC —
  test-first, ending with proper verification and self-review following the
  repo owner's workflow (marsh rules). Use whenever the user asks to
  implement/add/create a module, handler, service, repository, endpoint, or
  asks "what do I do after implementing", "how to verify", "how to lint/test"
  — whether they mention Go, Echo, ConnectRPC, bun, or just say "implement
  phase" in a Go project, use this skill.
---

# implement-go — Implement Go REST API Module (Test-First + Verify)

Workflow for writing/editing Go REST API code (Echo + Bun or ConnectRPC) in
projects that follow the marsh rules pattern: `internal/{module}/`,
interface-based DI, AppError, unified response `{code, message, data, pagination}`.

## 1. Before starting — read project context

Confirm the project matches the pattern this skill targets:

- `internal/{module}/` + `pkg/` per `rules/go-rest-api/01_ProjectStructure.md`
- `Makefile` with targets: `fmt`, `vet`, `lint`, `test`, `pre-commit`
- `.golangci.yml` per `lint/go.md` (golangci-lint v2)
- `internal/testutil/` test helpers

Always read before writing code: `go.mod` (module name), `Makefile`,
`.golangci.yml`, `internal/server/di.go` (to wire new dependencies), and one
similar existing module to match the project's actual structure — real code
trumps generic rules.

If the project does not match this pattern, ask the user instead of forcing it.

## 2. Core workflow — Test-First

Always write tests before code (red → green):

1. **Red** — write integration tests that fail (capturing the desired behavior)
2. **Green** — minimal implementation to make them pass
3. **Refactor** — align with the rules below (interfaces, AppError, response)

**Test patterns to follow** (per `rules/go-rest-api/08_TestingStandards.md`):

```go
//go:build integration

type Suite struct {
    suite.Suite
    db       *testutil.TestDB
    app      *echo.Echo  // or httptest.Server for ConnectRPC
    client   *testutil.TestClient
    fixtures *testutil.Fixtures
}

func (s *Suite) SetupSuite() { /* config, DB, migrations, DI */ }
func (s *Suite) SetupTest()  { /* truncate, fixtures, tokens */ }
```

- **Integration tests only** — real DB via testutil, no repo/service mocking
- Unit tests only for pure functions (snowflake, pagination, string utils)
- Every feature needs at least: success path, error path, wrong role → 403
- Table-driven tests for multi-case logic

## 3. While implementing — patterns you must not deviate from

### Per-module structure (1 folder = 1 domain)

| File | Responsibility |
|---|---|
| `entity.go` | Bun model + tags |
| `repo.go` | Interface + unexported impl |
| `service.go` | Interface + unexported impl |
| `handler.go` | Struct + constructor (no methods) |
| `*_handler.go` | Handler methods (split user/admin) |
| `request.go` | Request/Response DTOs |
| `route.go` | Route registration |

### Layer rules

- Handler → Service → Repository → DB; cross-module calls happen at the
  Service layer
- Handler depends on Service **interface**, Service depends on Repository
  **interface**
- Repository owns DB concerns: soft-delete filter (`deleted_at`),
  optimistic locking

### Error handling (per `rules/go-shared/02_ErrorHandling.md`)

| Layer | Error type |
|---|---|
| Repo | Sentinel errors, `fmt.Errorf("...: %w", err)` |
| Service public | **`apperrors.*` only** |
| Service private | `fmt.Errorf` |
| Handler | `response.HandleError(c, err)` — auto-maps code → HTTP status |

### Response format (per `rules/go-rest-api/05_ResponsePatterns.md`)

- Success: `{code, message, data}`
- List: `{code, message, data[], pagination{page, page_size, total_data, total_page, has_next_page, has_previous_page}}`
- Error: `{code, message}`; version conflict → 409 + data
- Always through helpers: `response.HandleSuccess` / `response.HandleError`
  (Echo), `connectutil.FromAppError` (ConnectRPC)

### Non-negotiable project conventions

- **Every list endpoint has pagination** — separate count and find (service
  exposes distinct methods)
- **PATCH/PUT carry a `version` check** — `WHERE version = ?` → conflict → 409
- **Multi-table writes run inside a transaction** — `ExecuteInTransaction` helper
- Soft delete — `deleted_at = 0`, no hard delete

## 4. After implementing — what to do next (never skip)

Run in this order until everything passes before reporting done:

```bash
make fmt            # gofmt -s + goimports -local {module}
make vet            # go vet ./...
make lint           # golangci-lint v2 — funlen 40, gocyclo/gocognit 15, nestif 5
go test ./...       # unit + pure function tests
go test -tags integration ./...   # integration tests (needs DB/testcontainers)
govulncheck ./...   # if present in Makefile
```

If a linter keeps complaining, check `lint/go.md` → `linters-settings`
(e.g. forbidigo bans `fmt.Print`/`log.`, and `== ""` — use `strings.TrimSpace`).

## 5. Self-review checklist — before reporting back to the user

- [ ] Every list endpoint has pagination
- [ ] PATCH/PUT schema has `version` + check on update
- [ ] Multi-table writes are inside a transaction
- [ ] Service public methods return `apperrors.*` only — no raw `fmt.Errorf`
      leaking out
- [ ] Response format is `{code, message, data[, pagination]}` via helpers
- [ ] No `any` leaking where a type-safe signature is expected
- [ ] No `fmt.Print` / `log.` / `== ""` in new code
- [ ] Handler → Service interface → Repo interface (no concrete types)
- [ ] Tests cover success + error + wrong role → 403
- [ ] `make fmt` + `make vet` + `make lint` + `go test ./...` pass

## 6. Wrap up — summarize for the user

Brief summary: modules/files created or changed, commands run and passing
(fmt/vet/lint/test), tests added, and what's left for the user (e.g. review,
running integration tests that need a DB, committing).
