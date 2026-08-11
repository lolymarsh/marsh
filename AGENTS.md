# AGENTS.md

Personal dev knowledge base + AI coding-rules collection. **Not a software project** — no package manifests, no build/test/lint/run commands. All work here is Markdown edits.

## Layout

- `rules/` — the main deliverable: reusable AI coding rules split into `{language}-{scope}/` folders (`go-shared`, `go-rest-api`, `go-microservice`, `typescript-shared`, `typescript-backend`, `typescript-frontend-react`, `python-rest-api`, `devops`). Each folder has numbered `NN_Name.md` files.
- `rules/README.md` — how rules get consumed (copied into a project's `.opencode/rules/`, concatenated into `AGENTS.md`/`CLAUDE.md`, or referenced via `opencode.json` `"rules"` array) and how to add new ones. **Read before editing rules.**
- `lint/` — linter config references extracted from real projects: `lint/{go,python,typescript}.md` (golangci-lint v2, ESLint 9, ruff).
- `skills/opencode-skills.md` — inventory of installed OpenCode skills (15).
- `docs/01_AI_WORKFLOW.md` — the owner's plan → spec → ARCHITECTURE.md → AGENTS.md → phase-based implementation workflow (per-module commits via jj). Read before anything about workflow/spec conventions.
- `ERP-AI-ROADMAP.md` — roadmap for the Versus Thailand ERP + AI chatbot project.
- `README.md` — owner's tech profile; the source of the stack conventions that `rules/` encodes.

## Conventions

- Content is written in Thai; keep Thai when editing or summarizing existing Thai sections.
- Rule files must follow `NN_Name.md` naming and use `trigger: always_on` frontmatter for standards that must always load; use code examples from real projects, not generic ones (see `rules/README.md` → "เพิ่ม Rules ใหม่").
- VCS is `jj` (Jujutsu), not `git` — the global `~/.claude/CLAUDE.md` wraps git to jj; `jj status` / `jj describe -m "msg"`.
