# Marsh Rules — AI Coding Rules Collection

> Rules สำหรับ AI Coding Tools (OpenCode, Claude Code, Codex) แยกตามภาษาและ domain

---

## โครงสร้าง

```
rules/
├── typescript-shared/              # TypeScript ใช้ร่วม FE + BE
├── typescript-backend/             # TypeScript Backend (Express/Fastify)
├── typescript-frontend-react/      # TypeScript Frontend (React + Vite)
│
├── go-shared/                      # Go ใช้ร่วมทุก project
├── go-rest-api/                    # Go REST API (Echo/ConnectRPC)
├── go-microservice/                # Go Microservice patterns
│
├── python-rest-api/                # Python REST API (FastAPI)
│
└── devops/                         # Docker, K8s, Nginx, CI/CD, Coolify
```

---

## วิธีใช้

### 1. OpenCode

คัดลอก rules ที่ต้องการไปไว้ใน `.opencode/rules/` ของ project:

```bash
# ตัวอย่าง: Go REST API + DevOps
cp -r rules/go-shared/. project/.opencode/rules/
cp -r rules/go-rest-api/. project/.opencode/rules/
cp -r rules/devops/. project/.opencode/rules/
```

หรือกำหนด path ใน `opencode.json`:

```json
{
  "rules": [
    "../rules/go-shared",
    "../rules/go-rest-api",
    "../rules/devops"
  ]
}
```

### 2. Claude Code (claude.ai/code)

คัดลอก rules ไปรวมใน `.claude/CLAUDE.md`:

```bash
# รวม rules เป็น CLAUDE.md
cat rules/go-shared/*.md rules/go-rest-api/*.md > project/.claude/CLAUDE.md
```

หรือแยกไฟล์ใน `.claude/rules/`:

```bash
mkdir -p project/.claude/rules
cp rules/go-shared/*.md project/.claude/rules/
cp rules/go-rest-api/*.md project/.claude/rules/
```

### 3. Codex (OpenAI)

คัดลอก rules ไปรวมใน `AGENTS.md`:

```bash
# รวม rules เป็น AGENTS.md
cat rules/go-shared/*.md rules/go-rest-api/*.md > project/AGENTS.md
```

---

## ตัวอย่างการใช้ตาม Stack

### Go REST API (Echo + Bun)

```bash
rules/go-shared/           # Coding standards, error handling, testing
rules/go-rest-api/         # Handler, service, repo, DB, response
rules/devops/              # Docker, K8s, deploy
```

### Go Microservice

```bash
rules/go-shared/
rules/go-rest-api/
rules/go-microservice/     # Architecture, service boundaries, saga
rules/devops/
```

### TypeScript Full-Stack (React + Express)

```bash
rules/typescript-shared/   # Coding standards, error handling, testing
rules/typescript-backend/  # Handler, service, repo, DB, response
rules/typescript-frontend-react/  # React MVC, components, state
rules/devops/
```

### Python REST API (FastAPI)

```bash
rules/python-rest-api/     # All Python REST API rules
rules/devops/
```

---

## หลักการแยก Rules

| หลักการ | เหตุผล |
|---|---|
| **แยก shared** | Coding standards, error handling, testing ใช้ร่วมได้ทุก project |
| **แยก frontend/backend** | AI ไม่สับสนระหว่าง Express handler กับ React component |
| **แยกตาม framework** | React ≠ Next.js, Echo ≠ Gin — patterns ต่างกัน |
| **DevOps แยกต่างหาก** | CORS, headers, rate limit ดักที่ gateway ไม่ต้องเขียนใน app |
| **trigger: always_on** | Rules ถูกโหลดตลอด — 适用于 coding standards ที่ต้องใช้เสมอ |

---

## Rules ที่มี

### TypeScript

| Folder | Files | เนื้อหา |
|---|---|---|
| `typescript-shared/` | 3 | Coding standards, error handling, testing |
| `typescript-backend/` | 8 | Project structure, handler, service, repo, response, DB, security, testing |
| `typescript-frontend-react/` | 4 | Project structure, component patterns, state management, testing |

### Go

| Folder | Files | เนื้อหา |
|---|---|---|
| `go-shared/` | 3 | Coding standards, error handling, testing |
| `go-rest-api/` | 8 | Project structure, handler, service, repo, response, DB, security, testing |
| `go-microservice/` | 6 | Architecture, service boundaries, inter-service communication, saga, API gateway, observability |

### Python

| Folder | Files | เนื้อหา |
|---|---|---|
| `python-rest-api/` | 10 | Project structure, coding standards, error handling, handler, service, repo, response, DB, security, testing |

### DevOps

| Folder | Files | เนื้อหา |
|---|---|---|
| `devops/` | 5 | Docker, Kubernetes, reverse proxy (Nginx/Traefik), CI/CD, Coolify |

---

## เพิ่ม Rules ใหม่

1. สร้าง folder ตาม pattern: `{language}-{scope}/`
2. สร้างไฟล์ `{序号}_{Name}.md`
3. เพิ่ม frontmatter `trigger: always_on` สำหรับ rules ที่ต้องใช้เสมอ
4. ใช้ code examples จาก project จริง (ไม่ใช่ generic)

---

## การใช้งานร่วมกับ DevOps Rules

**หลักการสำคัญ:** CORS, Security Headers, Rate Limiting → ดักที่ Reverse Proxy (Nginx/Traefik) ไม่ต้องเขียนใน application code

```
Request Flow:
Client → Reverse Proxy (CORS, headers, rate limit) → Application (auth, validation, business logic)
```

ดูรายละเอียดใน `devops/03_ReverseProxyPatterns.md`
