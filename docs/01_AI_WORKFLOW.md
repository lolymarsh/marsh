# todo.md — AI (OpenCode/Claude) for Development

> Flow การทำงานกับ AI ตั้งแต่ไอเดีย → Spec → Rule → Implement → Test
> อ่านก่อนเริ่มงานทุกครั้ง

---

## TL;DR — Flow การใช้งาน AI ที่ผสาน The Main Flow (AI Hero / Matt Pocock)

```
[ไอเดีย / Requirement ใหม่]
         │
         ▼ (Agent & You)
1. /grill-me หรือ /grill-with-docs ───► AI ยิงคำถามสัมภาษณ์ สรุป Edge Cases & Trade-offs
         │
         ▼ (AI)
2. /to-spec (หรือ plan.md) ──────────► สังเคราะห์การคุยเป็น Spec ชัดเจน
         │
         ▼ (AI & You)
3. กำหนด ARCHITECTURE.md + AGENTS.md ─► ดึง coding rules จาก rules/ เป็น Baseline
         │
         ▼ (AI)
4. /to-tickets ──────────────────────► ซอยงานเป็น Tracer-bullet tickets (Atomic tasks เล็กๆ)
         │
         ▼ (AI)
5. /implement หรือ /tdd ─────────────► Implement ทีละ ticket ด้วย Test-First
         │
         ▼ (AI)
6. /code-review ─────────────────────► Dual Review: เช็ค Spec Match + AGENTS.md Standards
         │
         ▼ (You)
7. jj describe -m "feat: ..." ────────► Commit ผ่าน jj ทีละชิ้นงานย่อย
```

---

## เริ่มงานตามสถานการณ์โปรเจกต์ (4 กรณี) + Skill Map

| สถานการณ์ | สิ่งแรกที่ต้องทำ | Skills ที่แนะนำให้เรียกใช้ | เอกสารที่ต้องสร้าง/ใช้ |
|---|---|---|---|
| **1. เริ่มโปรเจกต์ใหม่** | สัมภาษณ์ความต้องการ → วาง Spec + Rules | `/grill-me` → `/to-spec` → `/to-tickets` → `/tdd` → `/code-review` | ✅ spec เต็ม (plan + phases + AGENTS.md) |
| **2. เข้าไปกลางโปรเจกต์** (มีโค้ดแล้ว บางส่วนเสร็จ) | สำรวจภาพรวม + ห้ามเขียนทับโค้ดเดิม | `/improve-codebase-architecture` (สำรวจ) → `/to-tickets` (ซอยงานที่เหลือ) | ⚠️ mini-spec ต่อ feature (1 หน้า) |
| **3. โปรเจกต์ค้าง กลับมาทำต่อ** (ยังไม่จบ) | เช็คสถานะปัจจุบัน vs spec เดิม | `/ask-matt` (หา flow) → `/handoff` (สรุป context ต่อช่วง) | ⚠️ checklist สถานะปัจจุบัน (`status.md`) |
| **4. โปรเจกต์เก่า เข้าไปแก้** (production / bug) | หา Root Cause + ห้ามแตะส่วนอื่น | `/diagnosing-bugs` (วิเคราะห์) → `/tdd` (เขียน failing test) → `/code-review` | ❌ ไม่ต้อง spec (ใช้ failing test + prompt เจาะจง) |

### กรณี 1: เริ่มโปรเจกต์ใหม่ (The Main Flow)
1. **สัมภาษณ์ & สรุปความต้องการ**: ใช้ `/grill-me` หรือ `/grill-with-docs` ให้ AI ยิงคำถามเพื่อเคลียร์ความคลุมเครือ
2. **สร้าง Spec & Baseline**: สังเคราะห์เป็น spec ด้วย `/to-spec` และสร้าง `ARCHITECTURE.md` + `AGENTS.md`
3. **แตกงานย่อย**: สั่ง `/to-tickets` ซอยเป็น tracer-bullet tasks เล็กๆ
4. **Implement**: ทำทีละ ticket ด้วย `/implement` หรือ `/tdd` (Red-Green-Refactor)
5. **Review & Commit**: รัน `/code-review` ตรวจ 2 มิติ (Spec Match + Architecture Standard) ก่อน commit

### กรณี 2: เข้าไปกลางโปรเจกต์ (มีโค้ดอยู่แล้ว)
1. **สำรวจสถาปัตยกรรมเดิม**: รัน `/improve-codebase-architecture` ให้ AI วิเคราะห์โครงสร้างและ pattern เดิม
2. **เช็คสถานะ**: ดูว่าอะไรเสร็จแล้ว อะไรค้างอยู่ (จาก code, TODOs, commits)
3. **Mini-spec & Tickets**: เขียน mini-spec 1 หน้า หรือใช้ `/to-tickets` สำหรับ feature เฉพาะที่ต้องทำ
4. **Implement ทีละนิด**: ทำตาม mini-spec → รัน `/code-review` เทียบกับโค้ดเดิม → commit
5. **ข้อควรระวัง**: ห้ามให้ AI "เดาและทำต่อเอง" โดยไม่สแกนโครงสร้างเดิมก่อน

### กรณี 3: โปรเจกต์ค้าง กลับมาทำต่อ
1. **ฟื้น Context**: อ่าน spec/plan เดิม หรือใช้ `/handoff` เพื่อดูสรุปช่วงการทำงาน
2. **สร้างสถานะปัจจุบัน**: สร้าง checklist: ✅ เสร็จแล้ว / 🔄 กำลังทำ / ⬜ ยังไม่ทำ
3. **ตรวจสอบความสอดคล้อง**: เช็คว่าโค้ดจริงกับ spec เดิมตรงกันไหม ถ้าไม่ตรงให้อัปเดต spec ก่อน
4. **ทำต่อทีละ Ticket**: แตกงานย่อยต่อจากจุดที่ค้างด้วย `/to-tickets`
5. **ส่งต่องาน**: เมื่อจบรอบการทำงาน ให้เรียก `/handoff` สรุปสถานะทิ้งไว้สำหรับ session ถัดไป

### กรณี 4: โปรเจกต์เก่า เข้าไปแก้ (Production / Bug Fix / Legacy)
1. **เข้าใจ Root Cause ก่อน**: เรียก `/diagnosing-bugs` ให้ AI สืบหาต้นตออย่างเป็นระบบ ห้ามแก้ตามอาการ
2. **เขียน Failing Test ดักก่อน**: รัน `/tdd` เขียน test เคสที่พังเพื่อยืนยันว่าเกิดบั๊กจริง
3. **แก้เฉพาะจุด (Isolated Fix)**: ให้ AI แก้โค้ดจน failing test ผ่าน โดยสั่งห้าม refactor ส่วนอื่น
4. **Regression & Security Check**: รัน test ทั้งหมด + รัน `/code-review` เช็คว่ากระทบจุดอื่นไหม
5. **Commit**: `jj describe -m "fix(module): bug explanation"`

**หลักคิดรวมทุกกรณี:**
- สเปกคือ "สัญญากับ AI" — ใหญ่แค่ไหน ขึ้นกับขนาดงาน ไม่ใช่ทุกงานต้อง spec เต็ม
- ก่อน AI เขียนอะไร → ต้องรู้สถานะปัจจุบันของโปรเจกต์ก่อนเสมอ
- งานบั๊ก/งานแก้จุดเล็ก $\rightarrow$ `/diagnosing-bugs` + `/tdd`
- งานฟีเจอร์ใหม่/งานใหญ่ $\rightarrow$ The Main Flow (`/grill-me` $\rightarrow$ `/to-spec` $\rightarrow$ `/to-tickets` $\rightarrow$ `/implement` $\rightarrow$ `/code-review`)

---

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
[ ] Security Scan ผ่าน (gitleaks + semgrep ไม่พบ High/Critical)?
[ ] ไม่มี Hardcoded API Keys / Passwords หลุดในโค้ด?
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

## Security Scanning Workflow (ความปลอดภัยระหว่างการพัฒนา)

> ⚠️ **คำถามพบบ่อย**: ควรจะสแกน Security แค่ตอนจบโปรเจกต์ใช่ไหม?
> **คำตอบคือ ไม่ควรสแกนแค่ตอนจบโปรเจกต์!** ต้องใช้แนวคิด **Shift-Left Security** (สแกนเรื่อยๆ ระหว่างเขียนโค้ด)

### จังหวะการสแกนความปลอดภัย (Security Gates)

1. **Every Commit (ทุกครั้งที่ Commit — อัตโนมัติ)**:
   - ใช้ `pre-commit` รัน **Gitleaks** (ดักจับ Key/Secret หลุด) และ **Semgrep** (สแกนโค้ดสั้นๆ)
   - ใช้เวลาเพียง < 2 วินาที ดักจับปัญหาก่อนโค้ดเข้า Git
2. **Every Module / Feature (เมื่อ AI เขียนจบ 1 Module)**:
   - สั่ง agy cli / opencode รัน `semgrep scan --config auto` เพื่อเช็กความเสี่ยง Injection, Auth, หรือ Logic ผิดพลาด
   - ให้ AI ช่วยแก้ปัญหานั้นทันทีขณะที่บริบทของโมดูลยังสดใหม่อยู่
3. **Final Gate (ก่อน Deploy / ส่งมอบงานจบโปรเจกต์)**:
   - รัน Full Audit ด้วย **Trivy** เพื่อตรวจหา Vulnerability ทั้งหมดใน Dependencies, Package, Config และ Docker Images
   - ยิงทดสอบ DAST (ถ้าเป็น Backend API)

📖 *ดูคู่มือฉบับเต็มและคำสั่งติดตั้งได้ที่: `tools/security-scanning.md`*

---

## โปรแกรมแนะนำสำหรับเพิ่มประสิทธิภาพ (Recommended Tools)

### 🚀 RTK — Intercept (rtk-ai/rtk)

📚 **รายละเอียดฉบับเต็ม (กลไก hook, flow input→output, rules, setup):** ดูที่ [`intercept/`](intercept/README.md) ในโปรเจกต์นี้

### 🔀 เครื่องมือ Intercept อื่นๆ ที่น่าสนใจ (ทางเลือก/เสริม RTK)

> ค้นจาก GitHub (2026-08-11) — กลุ่มเครื่องมือแนวเดียวกัน: ดักจับ/บีบอัด output ของคำสั่ง shell ก่อนเข้าสู่ context ของ AI agent

| เครื่องมือ | สแตก | จุดเด่น | เหมาะกับ |
|---|---|---|---|
| [**kuro-lean** (`kt`)](https://github.com/kurovu146/kuro-lean) | Bun + Claude Code | บีบอัด shell output + **Guard** บล็อกคำสั่งที่เผา token ก่อนรัน (เช่น `find /`, cat ไฟล์ 5MB) + **แสดงราคาจริง** ต่อ session + **กู้ session** ที่ prompt cache หมดอายุแล้ว (~2.5k tokens) | คนที่ใช้ Claude Code และอยากเห็นว่าเงินหายไปกับอะไร (cache read/write ~89% ของบิล ไม่ใช่ output) |
| [**aivenv / aienv**](https://github.com/xmonader/aivenv) | Go | AI-optimized virtual environment — บีบอัด CLI output 40–90% | ใช้ wrapper แทน shell ปกติ |
| [**token-trim**](https://github.com/Junr-Studio/token-trim) | TypeScript | บีบอัด output ของ shell command ที่ agent รัน เพื่อตัด token ใน context | ฝั่ง agent อื่นๆ (ไม่พึ่ง Claude Code hooks) |
| [**bitrun**](https://github.com/BitSec01/bitrun) | Rust | shell-output token reducer — บีบอัดทั้งคำสั่งและอ่านไฟล์ใหญ่ก่อนเข้าสู่ agent | คล้าย RTK แต่โฟกัสอ่านไฟล์ใหญ่ |

**บทเรียนจาก kuro-lean (มีข้อมูลวัดจริง 93k+ messages):**
- **ค่าใช้จ่ายส่วนใหญ่ (~89%) อยู่ที่ cache read + cache write** ไม่ใช่ output — การบีบอัด output อย่างเดียวช่วยได้จำกัด
- ค่า token 1 ตัวที่ load เข้า context = 2× input (cache write) + 0.1× input × ทุกรอบที่เหลือ → **การไม่ load ข้อมูลที่ไม่จำเป็น (หรือ `/clear`/แยก session)** ประหยัดกว่า compression มาก
- session ยาว 600–3000 turns (1% ของ session) คิดเป็น 43.7% ของบิล — แยก session ตามงาน = ลดได้ ~14%

**คำแนะนำของผม:**
- **RTK** = ตัวหลักที่ใช้อยู่ (ติดตั้งแล้ว, hook ครบ Claude Code/OpenCode) — ใช้ต่อ
- **kuro-lean** = น่าลองที่สุดถ้าใช้ Claude Code เพราะวัด cost จริง + Guard บล็อกคำสั่งก่อนรัน (เสริม RTK ได้ ไม่ต้องเลือกอย่างใดอย่างหนึ่ง)
- สิ่งที่ได้ผลจริงโดยไม่ต้องติดตั้งอะไร: `/compact` หรือ `/clear` ระหว่างงาน, แยก session ยาวๆ, ใช้ subagent ที่ context ตายพร้อมตัวมันเอง

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
