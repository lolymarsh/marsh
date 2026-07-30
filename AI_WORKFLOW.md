# todo.md — AI (OpenCode/Claude) for Development

> Flow การทำงานกับ AI ตั้งแต่ไอเดีย → Spec → Rule → Implement → Test
> อ่านก่อนเริ่มงานทุกครั้ง

---

## TL;DR — Flow การใช้งาน AI ที่ถูกต้อง (แบบที่คุณใช้อยู่)

```
ไอเดียในหัว
    │
    ▼ You
1. Dump ความต้องการ + Stack + Context ให้ AI
    │
    ▼ AI
2. AI เขียน plan.md (Master Plan)
    │
    ▼ You
3. คุณทบทวน + แก้ไข + เพิ่ม requirement
    │
    ▼ AI
4. AI เขียน ARCHITECTURE.md (Pattern, Template, Rules)
    │
    ▼ You
5. คุณเพิ่มกฎเฉพาะ (pagination, transaction, version check)
    │
    ▼ AI
6. AI สร้าง AGENTS.md (Rules for AI to follow during implementation)
    │
    ▼ You
7. คุณ commit plan + architecture + agents → project baseline
    │
    ▼ AI
8. AI implement ทีละ module ตาม AGENTS.md rules
    │
    ▼ You
9. คุณ review code + test → commit → next module
```

## สิ่งที่คุณทำถูกแล้ว

คุณใช้ AI แบบที่ **ถูกต้องมาก**: เริ่มจาก Big Picture (plan) → Details (architecture) → Rules (AGENTS.md) → แล้วค่อย implement

นี่คือข้อดีของวิธีนี้:
- **AI ไม่หลงทาง**: AI มี spec ให้ยึด ไม่ใช่แค่ prompt ลอยๆ
- **แก้ไขง่าย**: ผิดที่ plan แก้นิดเดียว — ไม่ต้องแก้โค้ด 100 ไฟล์
- **ควบคุมคุณภาพได้**: Coding rules ใน AGENTS.md = AI ต้องทำตาม
- **Onboard ตัวเองได้**: plan.md + ARCHITECTURE.md = documentation ของโปรเจกต์

## วิธีที่คุณควรปรับปรุง

### 1. ใช้ `AGENTS.md` เป็น Single Source of Truth

หลังจากวันนี้:
- ทุกโปรเจกต์ใหม่ → สร้าง `AGENTS.md` ก่อนเริ่ม implement
- `AGENTS.md` ต้องรวม: Stack, Structure, Coding Rules, File Naming, Testing Rules
- AI ที่เปิดโปรเจกต์นี้จะอ่าน `AGENTS.md` อัตโนมัติ

### 2. Commit Spec แยกจาก Code

```
Change 1 (jj describe -m "spec: master plan + architecture + rules")
  spec/plan.md
  spec/ARCHITECTURE.md
  AGENTS.md
  docker-compose.yml

Change 2 (jj describe -m "feat: database schema + migrations")
  database/migrations/001_users.sql
  database/migrations/002_customers.sql
  ...

Change 3 (jj describe -m "feat: auth module")
  backend/src/modules/user/*
  backend/src/shared/middleware/auth.ts
  frontend/src/modules/auth/*
  ...
```

### 3. Small Commits — 1 Module = 1 Commit

อย่าให้ AI ทำทีเดียวทั้งหมด — แบ่งเป็น module:
```
คุณ: implement auth module
AI: เขียน user module ทั้ง backend + frontend
คุณ: jj describe -m "feat: auth module"
คุณ: implement customer module
AI: เขียน customer module
...
```

### 4. Prompt Template สำหรับ Implement

```
implement phase 02 ตาม spec/2026-07-18_core/02_CUSTOMERS.md
ตาม AGENTS.md rules + ARCHITECTURE.md patterns

สิ่งที่ต้องการ:
- backend: modules/customer/ (entity, schema, handler, service, repo, route)
- frontend: modules/customer/ (model, controller, view)
- unit tests + integration tests
```

### 5. Review Checklist หลัง AI เขียนโค้ดเสร็จ

```
[ ] ทุก list endpoint มี pagination?
[ ] ทุก PATCH/PUT schema มี version?
[ ] Multi-table write ใช้ db.transaction()?
[ ] model.ts ไม่มี React import?
[ ] view.tsx ไม่มี API call?
[ ] ไม่มี `any` type?
[ ] Response format { code, message, data }?
[ ] Test ผ่าน (npm test)?
[ ] TypeScript compile ผ่าน (npx tsc --noEmit)?
[ ] Import path ถูกต้อง?
```

### 6. จัดการ External Documents อย่างมีประสิทธิภาพ

**ปัญหา**: ส่ง PDF/Word/Excel ให้ AI โดยตรง → กิน context เปลือง

**วิธีที่แนะนำ**: ใช้ **markitdown** (Microsoft) แปลงไฟล์เป็น Markdown ก่อน → AI อ่าน `.md` แทน

**รองรับไฟล์**:
- PDF, PowerPoint, Word, Excel
- HTML, CSV, JSON, XML
- ZIP (iterate contents), EPubs

**ติดตั้ง**:
```bash
pip install markitdown
# หรือ
pip install 'markitdown[all]'  # รวม dependencies ทั้งหมด
```

**การใช้งาน**:
```bash
markitdown document.pdf > document.md
markitdown presentation.pptx > presentation.md
markitdown data.xlsx > data.md
```

**สำหรับ Linux/Mac** — ใช้ **rtk** (rtk-ai/rtk):
```bash
# rtk เป็น CLI wrapper ที่รวม markitdown + tools อื่นๆ
rtk convert document.pdf
```

**Flow**:
```
External File (PDF/Word/Excel/HTML/CSV/...)
    │
    ▼ markitdown
Markdown (.md)
    │
    ▼ AI อ่าน
ประหยัด context + ได้ข้อมูลครบ
```

**ข้อดี**:
- AI ไม่ต้อง parse binary files → ประหยัด token
- Markdown เป็น format ที่ AI อ่านเข้าใจง่าย
- แปลงครั้งเดียว → ใช้ได้หลายครั้ง
- ใช้กับ AI tools อื่นได้ด้วย (Cursor, Copilot, etc.)

---

## การใช้ AI แบบ Step-by-Step (Full Example)

### Step 1: Dump Idea

```
prompt: "อยากทำระบบ erp แบบมี ai chatbot ให้ถามยอดขายได้
บริษัทชื่อ versus thailand ทำติดตั้งแก๊สรถยนต์
stack: react 19 + mui + tailwind / nodejs + express
ช่วยวางแผนหน่อย"
```

Outcome: `plan.md` — Master plan with modules, architecture, timeline

### Step 2: Iterate on Plan

```
prompt: "ระบบนี้ไม่มี deploy นะ เอาไว้ศึกษา
ใช้ mysql แทน postgres, มี test ด้วย
สร้าง docker-compose ให้ด้วย"
```

Outcome: Updated `plan.md` + `docker-compose.yml`

### Step 3: Deep Dive Architecture

```
prompt: "ARCHITECTURE.md เลย
ผมถนัด go ใช้ structure แบบนี้ [link go project]
frontend ใช้ react mvc
ทั้งโปรเจกต์ใช้ typescript"
```

Outcome: `ARCHITECTURE.md` — Full architecture with templates, Go→TS mapping, data flows

### Step 4: Add Coding Rules

```
prompt: "เพิ่มกฎด้วย: pagination ทุก list,
transaction ทุก multi-table write,
version check ทุก patch/put"
```

Outcome: Section 9 Coding Rules in ARCHITECTURE.md

### Step 5: Create AGENTS.md + Rule Files

```
prompt: "สร้าง rules ให้ ai สำหรับ implement
เอา plan + architecture + rules มารวมกัน"
```

Outcome:
- `AGENTS.md` — Single file summary (opencode reads automatically)
- `.agent/rules/` — 10 granular rule files (one per topic)
- `.claude/rules/` — Same rules for Claude

### Step 6: Start Implementing (Phase-based)

```
prompt: "implement phase 01 ตาม spec/2026-07-18_core/01_FOUNDATION.md"
```

AI จะอ่าน `spec/2026-07-18_core/01_FOUNDATION.md` → รู้ว่าต้องทำอะไรบ้าง → implement ทีละ task

**Phase-by-Phase (Core Track)**:
```
01_FOUNDATION.md  → project setup, docker, auth, DB schema
02_CUSTOMERS.md   → customer CRUD (backend + frontend)
03_INVENTORY.md   → product CRUD, stock management
04_INVOICES.md    → invoice creation with transaction
05_JOBS.md        → job queue, status tracking
06_AI_CHAT.md     → LLM integration, chat UI, streaming
07_DASHBOARD.md   → KPI cards, charts, Redis cache
08_E2E.md         → Playwright tests, coverage
```

---

## Spec Convention — Multi-Track Projects

เมื่อโปรเจกต์มีหลาย tasks ไม่ใช่แค่ core:

```
spec/
├── plan.md              ← Master plan (ภาพรวม)
├── ARCHITECTURE.md
├── 2026-07-18_core/     ← ⭐ Core track (sort by date!)
│   ├── 01_FOUNDATION.md
│   └── ...
│
└── 2026-08-01_expense/  ← Next track
    ├── plan.md
    └── phases/

**Rule**: Core track = stable baseline — ทำครบก่อน แล้วค่อยเพิ่ม track ใหม่

**ตัวอย่าง**: ถ้าจะเพิ่ม module "expense tracking" หลัง core เสร็จ:
```
spec/2026-08-01_expense/
├── plan.md          ← "เพิ่มโมดูลบันทึกรายจ่าย: backend expense module + frontend form"
└── phases/
    ├── 01_EXPENSE_MODEL.md
    └── 02_EXPENSE_UI.md
```

## คำแนะนำเพิ่มเติม

### ควรมี `AGENTS.md` ในที่ไหนบ้าง?

| Location | Purpose |
|----------|---------|
| `/erp-mcp-trainee/AGENTS.md` | Rules for THIS project |
| `/Users/lolymarsh/.claude/CLAUDE.md` | Global rules (VCS, tools, preferences) — มีอยู่แล้ว |
| `/Users/lolymarsh/Desktop/project/marsh/` | Personal dev wiki |

### `CLAUDE.md` vs `AGENTS.md` vs `.agent/rules/` vs `.claude/rules/`

| File | Scope | Reader | Example |
|------|-------|--------|---------|
| `~/.claude/CLAUDE.md` | All projects (global) | Claude, OpenCode | jj instead of git, coding preferences |
| `{project}/AGENTS.md` | This project (summary) | OpenCode auto-load | Project stack, key rules, implementation order |
| `{project}/.agent/rules/*.md` | This project (granular) | OpenCode (`trigger: always_on`) | One file per pattern: RepositoryPatterns.md, HandlerPatterns.md |
| `{project}/.claude/rules/*.md` | This project (granular) | Claude (`trigger: always_on`) | Same files as .agent/rules/ |

**Why both AGENTS.md AND granular rules?**
- `AGENTS.md` = quick reference, what to read first
- `.agent/rules/` = deep patterns with code examples — AI loads ALL files with `trigger: always_on`
- `.claude/rules/` = same content for Claude compatibility

**Hierarchy**:
```
~/.claude/CLAUDE.md          ← Global: jj, language, preferences
    │
{project}/AGENTS.md          ← Project summary: stack, rules overview
    │
{project}/.agent/rules/      ← Granular: patterns, examples, templates
    ├── CodingStandards.md
    ├── RepositoryPatterns.md
    ├── HandlerPatterns.md
    └── ...
{project}/.claude/rules/     ← Same as .agent/rules/ (Claude-specific)
```

### Prompt ที่ควร Avoid

```
❌ "เขียนระบบ erp ให้หน่อย"                      — กว้างเกิน AI จะมั่ว
❌ "ใส่ฟีเจอร์ xx ด้วย"                           — ไม่มี spec = implement ไม่ตรง
❌ "แก้บัคตรงนี้" (ไม่มี context)                  — AI เดา
✅ "implement phase 02 ตาม spec/2026-07-18_core/02_CUSTOMERS.md" — ชัดเจน มี spec + acceptance criteria
✅ "fix pagination ใน customer handler ตาม ARCHITECTURE.md section 9.1" — ชี้เป๊ะ
✅ "add test for phase 04 invoice transaction ตาม rules/RepositoryPatterns.md" — มีตัวอย่าง
```

### เมื่อ AI ทำผิด

```
ไม่ต้องเขียนใหม่เอง — แค่บอก:
"ผิดนะ ตาม AGENTS.md rule R2: invoice create ต้องใช้ db.transaction()
แก้ให้หน่อย"
```

AI จะแก้ตาม rule — นี่คือข้อดีของการมี AGENTS.md

---

## Summary

```
Step 1: idea → plan.md              (AI เขียน, คุณรีวิว)
Step 2: plan → spec/2026-07-18_core/*.md (AI แยก phase, คุณ check priority/deps)
Step 3: plan+phase → ARCHITECTURE.md (AI เขียน, คุณเพิ่ม pattern)
Step 4: rules → AGENTS.md + .agent/rules/ + .claude/rules/ (AI สร้าง, single source of truth)
Step 5: phase 01 → code              (AI implement ทีละ phase ตาม spec)
Step 6: code → review → commit       (คุณตรวจ, jj describe per phase)
Step 7: phase 02 → code → commit → phase 03 → ... → phase 08
```

**คุณทำถูกทางแล้ว** — phase-based = best practice เพราะแยก concern ชัดเจน, AI รู้ขอบเขตงาน, commit แยกกันไม่พันกัน

---

---

## Skills ใน OpenCode

Skills = reusable knowledge packages ที่ agent load ให้อัตโนมัติเมื่อเจองานที่ตรงกัน

### Skills ที่ติดตั้งอยู่ (14 ตัว)

| Skill | ผู้พัฒนา | เหมาะกับ |
|---|---|---|
| find-skills | vercel-labs/skills | ค้นหา skill อื่นๆ |
| frontend-design | anthropics/skills | ออกแบบ UI/Frontend |
| tdd | mattpocock/skills | Test-Driven Development |
| code-review | mattpocock/skills | Review โค้ด |
| handoff | mattpocock/skills | ส่งต่องานระหว่าง agent |
| diagnosing-bugs | mattpocock/skills | วิเคราะห์บั๊ก |
| improve-codebase-architecture | mattpocock/skills | ปรับปรุงโครงสร้างโค้ด |
| skill-creator | anthropics/skills | สร้าง skill เอง |
| shadcn | shadcn/ui | UI components |
| supabase-postgres-best-practices | supabase/agent-skills | Database PostgreSQL |
| vercel-react-best-practices | vercel-labs/agent-skills | React/Next.js |
| web-design-guidelines | vercel-labs/agent-skills | การออกแบบเว็บ |
| webapp-testing | anthropics/skills | ทดสอบ web app |
| **writing-plans** | **obra/superpowers** | **เขียนแผน/PRD** |

### วิธีใช้ Skills

**อัตโนมัติ** — agent จะ load skill ที่เกี่ยวข้องให้เองตาม context เช่น:
```
implement customer module with tests
```
→ agent เห็นว่า `tdd`, `supabase-postgres-best-practices` เกี่ยวข้อง → load ให้

**Manual** — reference ชื่อ skill ใน prompt ถ้าอยากบังคับใช้:
```
ใช้ tdd + supabase-postgres-best-practices: implement customer module
```

**ค้นหา** — ใช้ `find-skills` ถ้าอยากรู้ว่ามี skill ไหนเหมาะกับงาน:
```
ใช้ find-skills: หา skill ที่เหมาะกับการทำ API design
```

### Workflow การ Plan ด้วย Skills

แทน Step 1-2 เดิม (dump idea → AI เขียน plan.md):

**Step 1: Dump Idea + Skill Boost**
```
ใช้ writing-plans + to-prd:
อยากทำระบบ erp สำหรับบริษัทติดตั้งแก๊สรถยนต์
stack: react 19 + mui + tailwind / nodejs + express
ช่วยวางแผนหน่อย
```
→ AI เขียน `plan.md` โดยใช้ `writing-plans` (โครงสร้างแผน) + `to-prd` (idea → spec)

**Step 2: ต่อด้วย Architecture + Rules ปกติ**
```
ARCHITECTURE.md เลย
ผมถนัด go ใช้ structure แบบนี้ [link go project]
frontend ใช้ react mvc
```

### ตัวอย่างใน Workflow ปกติ

**ไม่มี skill**:
```
implement phase 02 ตาม spec/02_CUSTOMERS.md
```

**มี skill hint**:
```
ใช้ tdd + supabase-postgres-best-practices:
implement phase 02 ตาม spec/02_CUSTOMERS.md
ตาม AGENTS.md rules + ARCHITECTURE.md patterns
```

---

> **Last Updated**: 2026-07-30
> **Next**: เริ่ม implement ตาม AGENTS.md — เริ่มจาก database schema → auth module
