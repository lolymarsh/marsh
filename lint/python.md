# Python — Linter Config Reference

## Tool: Ruff (recommended)

```toml
# pyproject.toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = [
    "E",    # pycodestyle
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "W",    # pycodestyle warning
    "UP",   # pyupgrade
    "ANN",  # annotations (strict type hints)
    "ARG",  # unused arguments
    "B",    # flake8-bugbear
    "C4",   # flake8-comprehensions
    "SIM",  # flake8-simplify
    "PL",   # pylint
    "RUF",  # Ruff-specific
]
# ANN rules enforce type hints everywhere:
# ANN001: missing type annotation for function argument
# ANN201: missing return type annotation for public function
# ANN202: missing return type annotation for private function
# ANN204: missing return type annotation for special method (__init__)
# ANN205: missing return type annotation for static method
# ANN206: missing return type annotation for class method
ignore = ["ANN101", "ANN102"]  # skip self/cls type annotation

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["ANN"]

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.ruff.format]
quote-style = "double"

[tool.ruff.lint.mccabe]
max-complexity = 15  # max cyclomatic complexity
```

## Tool: mypy (Type checking)

```toml
[tool.mypy]
python_version = "3.12"
strict = true              # enforce type hints everywhere (Go/TS style)
ignore_missing_imports = true
explicit_override = true   # require @override on all overridden methods
warn_unreachable = true    # warn on unreachable code
disallow_untyped_defs = true        # forbid functions without type annotations
disallow_untyped_calls = true       # forbid calling untyped functions
disallow_any_unimported = true      # forbid Any from unimported modules
disallow_incomplete_defs = true     # forbid incomplete type hints
```

## Commands

```bash
ruff check src/              # Lint
ruff format src/             # Format
ruff check --fix src/        # Auto-fix
mypy src/                    # Type check
pytest --cov=src/ tests/     # Test with coverage
```

## Key Rules

- Type hints required on all functions (`ANN` — error level)
- No unused imports/variables (`F401`, `F841`)
- Proper naming: `PascalCase` public methods, `_camelCase` private methods, `snake_case` vars, `UPPER_SNAKE` constants
- pyupgrade for modern syntax (3.12+)
- Max complexity: 15
- Max line length: 100
- Max lines per function: 40
- Google-style docstrings
