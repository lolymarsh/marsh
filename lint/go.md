# Go — Linter Config Reference

Created from: `be-go-echo`, `be-connectrpc-go`

## Tool: golangci-lint v2

```yaml
# .golangci.yml
linters:
  enable:
    # Security
    - gosec
    - noctx
    # Error handling
    - errcheck
    - errorlint
    # Bugs
    - govet
    - staticcheck
    - nilnil
    - copyloopvar
    # Code quality
    - funlen
    - gocyclo
    - gocognit
    - nestif
    - gocritic
    # Performance
    - prealloc
    - makezero
    # Closed resources
    - bodyclose
    -rowserrcheck
    - sqlclosecheck
    - unused
    - ineffassign
    # Custom
    - forbidigo

linters-settings:
  funlen:
    lines: 40
    statements: 30

  gocyclo:
    min-complexity: 15

  gocognit:
    min-complexity: 15

  nestif:
    min-complexity: 5

  gocritic:
    enabled-tags:
      - diagnostic
      - performance
      - style

  forbidigo:
    forbid:
      - p: ^fmt\.Print(|f|ln)$
      - p: ^log\.
      - pattern: == ""
        msg: use strings.TrimSpace instead

  errcheck:
    check-type-assertions: true
    check-blank: true

## Commands

```bash
make lint           # golangci-lint run ./internal/... ./pkg/... (hides false positives)
make lint-quick     # max 5 issues/linter
make lint-fix       # auto-fix
make fmt            # gofmt -s + goimports -local {module}
make vet            # go vet ./...
make pre-commit     # fmt + vet + lint + vuln
```

## Key Rules

- Function length: max 40 lines
- Cyclomatic complexity: max 15
- Cognitive complexity: max 15
- Nested if depth: max 5
- No `== ""` — use `strings.TrimSpace` instead
- Error returns always checked
- SQL rows, HTTP bodies always closed
