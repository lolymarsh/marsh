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

## เริ่มงานตามสถานการณ์โปรเจกต์ (5 กรณี) + Skill Text Flow

---

### กรณี 1: เริ่มโปรเจกต์ใหม่ / ฟีเจอร์ใหม่ (The Main Flow)

```
[🟢 Session 1: Planning (คุยต่อในแชทเดิม)]
  ไอเดียใหม่ / Business Requirement
       │
       ▼
  1. /grill-me ──────────────► AI สัมภาษณ์เจาะลึก Edge Cases & Trade-offs
       │
       ▼
  2. /to-spec ───────────────► สรุปเป็น spec/plan.md + spec/YYYY-MM-DD_[track]/[NN].md
       │
       ▼
  3. สร้าง Baseline ──────────► วาง ARCHITECTURE.md + AGENTS.md (ดึงกฎจาก rules/)
       │
       ▼
  4. /to-tickets ────────────► แตกเป็นตั๋ว tracer tickets ลงใน tickets/
       │
       ▼
  5. jj describe -m "spec: initial plan, architecture and tickets"
       │
═══════╪═══════════════════════════════════════════════════════════════════════
       ▼ 🎯 จบ Session 1 — [ ปิดแชทเดิม / เปิดแชทใหม่ ]
═══════╪═══════════════════════════════════════════════════════════════════════
       │
[🔵 Session 2..N: Implement ราย Ticket (1 Ticket = 1 Session)]
  1. เปิดแชทใหม่: "อ่าน AGENTS.md และทำตาม ticket: tickets/.../01_task.md ด้วย tdd"
       │
       ▼
  2. /tdd ───────────────────► เขียน Failing Test ก่อน → Implement โค้ดจนผ่าน
       │
       ▼
  3. /code-review ───────────► ตรวจสอบ Spec Match + Standards
       │
       ▼
  4. jj describe -m "feat(module): ticket description" ──► [ จบ Session ]
```

#### 📋 Prompt สำเร็จรูป (ก๊อปปี้ไปใช้ได้ทันที):
* **Step 1 (Grill)**:
  ```text
  ใช้ grill-me:
  อยากทำระบบ/ฟีเจอร์ [ชื่อเรื่อง]
  บริบท: เป้าหมาย [รายละเอียด], Stack [เช่น Go + Echo / React + Vite], ขอบเขต [โมดูลหลัก]
  ช่วยซักถาม/สัมภาษณ์ผมแบบเจาะลึกเพื่อเก็บ Requirement และ Edge Cases ทั้งหมดก่อนเริ่มทำ Spec
  ```
* **Step 2 (To Spec)**:
  ```text
  ใช้ to-spec:
  สรุปทั้งหมดลง spec/plan.md และแยกไฟล์โมดูลลง spec/YYYY-MM-DD_[track]/NN_[MODULE].md ตาม convention
  ```
* **Step 3 (To Tickets)**:
  ```text
  ใช้ to-tickets:
  ช่วยแตก tracer tickets จากไฟล์ spec/YYYY-MM-DD_[track]/[NN_MODULE].md ลงโฟลเดอร์ tickets/YYYY-MM-DD_[track]/
  ```

---

### กรณี 2: ฟีเจอร์เดิมเปลี่ยน Flow / เปลี่ยน Requirement

```
[🟢 Session 1: Update Spec & Tickets]
  โจทย์เปลี่ยน Flow (เช่น รวม API อัปโหลดรูป + เพิ่ม Turnstile)
       │
       ▼
  1. อัปเดต Spec เดิม ────────► ส่ง Prompt แก้ไขไฟล์ spec/.../NN_MODULE.md (In-place)
       │
       ▼
  2. /to-tickets ────────────► AI แตกตั๋วงานย่อยชุดใหม่ลงใน tickets/
       │
       ▼
  3. jj describe -m "spec: update upload flow to single endpoint with turnstile"
       │
═══════╪═══════════════════════════════════════════════════════════════════════
       ▼ 🎯 จบ Session วางแผน — [ ปิดแชทเดิม / เปิดแชทใหม่ ]
═══════╪═══════════════════════════════════════════════════════════════════════
       │
[🔵 Session 2..N: Implement ตั๋วใหม่]
  1. เปิดแชทใหม่: สั่งทำตั๋วใหม่ด้วย skill: tdd
  2. /tdd ───────────────────► แก้ไข Test เดิมให้ตรง Flow ใหม่ (Test จะ Red) → แก้โค้ดจน Green
  3. /code-review ───────────► ตรวจสอบความถูกต้อง → jj commit
```

#### 📋 Prompt สำเร็จรูป (ก๊อปปี้ไปใช้ได้ทันที):
```text
อัปเดตไฟล์ spec/YYYY-MM-DD_[track]/[NN_MODULE].md:

เรามีการเปลี่ยน Flow การทำงานในโมดูลนี้:
- Flow เดิม: [เช่น แยก endpoint อัปโหลดรูป กับ บันทึกข้อมูล]
- Flow ใหม่ที่ต้องการ: [เช่น รวมเป็น endpoint เดียวผ่าน Multipart/form-data + ใส่ Turnstile validation]
- เหตุผล: [เช่น ป้องกัน bot และลด network round-trip]

สิ่งที่ต้องแก้ในไฟล์ spec เดิม:
1. ปรับปรุง Schemas, API Endpoints และ Acceptance Criteria ในไฟล์ spec เดิม (ห้ามสร้างไฟล์ใหม่)
2. เสร็จแล้วใช้ to-tickets แตกตั๋วงานย่อยชุดใหม่ลงใน tickets/YYYY-MM-DD_[track]/
```

---

### กรณี 3: งานแก้บั๊ก / Hotfix / ปัญหา Production

```
[🔴 Session เฉพาะกิจ (Clean Context 100% — ไม่ต้องทำ Spec)]
  พบอาการบั๊ก / Error Log จาก Production
       │
       ▼
  1. /diagnosing-bugs ───────► วิเคราะห์ Root Cause หาต้นตอของปัญหา (ห้ามแก้เดาสุ่ม)
       │
       ▼
  2. /tdd ───────────────────► เขียน Failing Test ดักจับบั๊ก (ยืนยันว่าพังจริงก่อนแก้)
       │
       ▼
  3. แก้ไขเฉพาะจุด ────────────► AI แก้ไขโค้ดจน Failing Test ผ่าน (ห้ามแตะไฟล์นอก scope)
       │
       ▼
  4. /code-review ───────────► รัน Regression Test + ตรวจสอบ Side-effects
       │
       ▼
  5. jj describe -m "fix(module): root cause and bug explanation" ──► [ จบงาน ]
```

#### 📋 Prompt สำเร็จรูป (ก๊อปปี้ไปใช้ได้ทันที):
```text
ใช้ diagnosing-bugs + tdd:

พบปัญหา/บั๊ก: [อาการ เช่น customer list กดเปลี่ยนหน้าแล้ว error 500 / build docker fail]
ไฟล์/โมดูลที่เกี่ยวข้อง: [ถ้ารู้ ระบุที่นี่ พร้อมแนบ error log]

ขั้นตอนที่ต้องการ:
1. หา Root Cause ก่อน — วิเคราะห์ flow และ log ห้ามเดาสุ่มแก้ตามอาการ
2. เขียน Failing Test ที่ reproduce บั๊กนี้ให้เห็นว่า fail จริง (กรณีโค้ดแอป)
3. แก้ไขโค้ดเฉพาะจุดจนกว่า test จะผ่าน — ห้ามแตะต้องไฟล์นอก scope หรือ refactor ส่วนอื่น
4. รัน Regression Test ทั้งหมด และสรุปผล: Root Cause คืออะไร + แก้ที่ไหน
```

---

### กรณี 4: เข้าไปแทรกงานกลางโปรเจกต์ / โค้ดคนอื่น

```
[🟢 Session 1: สำรวจสถาปัตยกรรม & แตกตั๋ว]
  เข้าไปโปรเจกต์ที่มีโค้ดอยู่แล้วบางส่วน
       │
       ▼
  1. สำรวจโค้ดเดิม ───────────► ให้ AI อ่านโครงสร้าง + เอกสารเดิม ห้ามเขียนทับโค้ด
       │
       ▼
  2. ทำ Mini-spec ───────────► สรุปสโคปเฉพาะฟีเจอร์ที่ต้องต่อเติม (1 หน้า)
       │
       ▼
  3. /to-tickets ────────────► แตกตั๋วงานย่อยที่ต้องทำต่อ
       │
═══════╪═══════════════════════════════════════════════════════════════════════
       ▼ 🎯 จบการสำรวจ — [ ปิดแชทเดิม / เปิดแชทใหม่ ]
═══════╪═══════════════════════════════════════════════════════════════════════
       │
[🔵 Session 2..N: Implement]
  รันทีละ Ticket ด้วย /tdd → /code-review → jj commit
```

#### 📋 Prompt สำเร็จรูป (ก๊อปปี้ไปใช้ได้ทันที):
```text
สำรวจโปรเจกต์นี้ก่อน:
1. อ่านโครงสร้างโค้ด, README, และ AGENTS.md แล้วสรุป architecture ให้ฟัง
2. เราจะเพิ่มฟีเจอร์: [ชื่อฟีเจอร์ เช่น export excel]
3. ช่วยเขียน mini-spec 1 หน้าลงใน spec/YYYY-MM-DD_track/NN_[FEATURE].md และแตกเป็น tickets ด้วย to-tickets
(ห้ามแตะต้องหรือ refactor โค้ดเดิมที่มีอยู่แล้ว)
```

---

### กรณี 5: โปรเจกต์ค้าง กลับมาทำต่อ / ส่งต่องานข้ามกะ

```
[🧰 จบรอบงานเดิม / Context ใกล้เต็ม]
  1. /handoff ───────────────► สร้าง docs/handoff-[YYYY-MM-DD].md สรุปสิ่งที่เสร็จ + สิ่งที่ค้าง
       │
═══════╪═══════════════════════════════════════════════════════════════════════
       ▼ 🎯 พักรอบงาน — [ ปิดแชทเดิม ]
═══════╪═══════════════════════════════════════════════════════════════════════
       │
[🔵 เริ่มรอบงานใหม่ (Session ใหม่)]
  2. เปิดแชทใหม่ ────────────► สั่ง: "อ่าน handoff.md และ AGENTS.md แล้วทำต่องานข้อถัดไป"
  3. รันด้วย /tdd ────────────► ทำงานต่อได้ทันทีโดย Context ไม่บวมและไม่หลุดบริบท
```

#### 📋 Prompt สำเร็จรูป (ก๊อปปี้ไปใช้ได้ทันที):
* **ตอนจบรอบ (Handoff)**:
  ```text
  ใช้ handoff:
  สรุปสถานะงานปัจจุบันลง docs/handoff-[YYYY-MM-DD].md: อะไรเสร็จแล้ว, อะไรค้างอยู่, Next Step ต้องทำอะไรต่อ
  ```
* **ตอนเปิดแชทใหม่ทำต่อ**:
  ```text
  อ่าน docs/handoff-[YYYY-MM-DD].md และ AGENTS.md แล้วทำงานในข้อ Next Step ถัดไป ด้วย skill: tdd
  ```

---

## 📌 สรุปกฎเหล็ก (Rule of Thumb) — งานแบบไหนต้องทำอะไร

| ประเภทงาน | ตัวอย่างงาน | ต้องทำ Spec มั้ย? | Session ที่ใช้ | Skills หลักที่ใช้ |
|---|---|---|---|---|
| **1. เริ่มโปรเจกต์ใหม่ / ฟีเจอร์ใหม่** | ระบบ Auth, Customer CRUD, Chatbot | ✅ **ทำ Spec เต็ม** (`spec/...`) | 🟢 Session 1 (Plan) → 🔵 Session 2..N (Code) | `/grill-me` → `/to-spec` → `/to-tickets` → `/tdd` |
| **2. ฟีเจอร์เดิมเปลี่ยน Flow** | รวม API อัปโหลดรูป, เพิ่ม Turnstile | 🔄 **อัปเดตทับ Spec เดิม** | 🟢 Session Update Spec → 🔵 Session Code | `to-spec` (in-place) → `/to-tickets` → `/tdd` |
| **3. งานแก้บั๊ก / Hotfix** | บั๊ก 500, ข้อมูลไม่อัปเดต, Logic ผิด | ❌ **ไม่ต้องทำ Spec** | 🔴 Session แยก (Clean) | `/diagnosing-bugs` + `/tdd` (Failing test) |
| **4. แทรกงานกลางโปรเจกต์ / โค้ดคนอื่น** | รับงานต่อ, เพิ่มโมดูลในระบบเดิม | ⚠️ **Mini-spec เฉพาะจุด** | 🟢 Session สำรวจ → 🔵 Session Code | `to-tickets` → `/tdd` → `/code-review` |
| **5. งานค้าง / ส่งต่องานข้ามกะ** | Context เต็ม, ทำงานข้ามวัน | 🔄 **ใช้ handoff.md นำทาง** | 🧰 Session สรุป → 🔵 Session ใหม่ | `/handoff` → `/tdd` |

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

### 🧰 Skill เสริมเฉพาะทางตาม Tech Stack (Tech Boosters)

สกิลกลุ่มนี้ AI จะ **โหลดให้อัตโนมัติ** ตามบริบทของงาน แต่ถ้าต้องการเรียกใช้แบบเจาะจง สามารถใช้ prompt สั้นๆ ดังนี้:

#### 1. งาน Database / SQL Optimization (`supabase-postgres-best-practices`)
* **เมื่อไหร่ที่ใช้**: ออกแบบตารางซับซ้อน, Query ช้า, หรือจะทำ Index/Partition
* **Prompt**:
  ```text
  ใช้ supabase-postgres-best-practices:
  ช่วยวิเคราะห์ query [ระบุ query] ทำไมถึงช้า ขอคำแนะนำ Index + Migration script ที่ปลอดภัย
  ```

#### 2. งาน UI Components & Design System (`shadcn`)
* **เมื่อไหร่ที่ใช้**: ติดตั้ง component ใหม่, หา UI pattern, หรือจัดการ styling
* **Prompt**:
  ```text
  ใช้ shadcn:
  เพิ่ม component [เช่น data-table, dialog, form] พร้อมตัวอย่างการใช้งานแบบ type-safe
  ```

#### 3. งาน Optimize Frontend / React Performance (`vercel-react-best-practices`)
* **เมื่อไหร่ที่ใช้**: หน้าเว็บโหลดช้า, มี Re-render ซ้ำซ้อน, หรือสลับ Client/Server Component
* **Prompt**:
  ```text
  ใช้ vercel-react-best-practices:
  ช่วย Audit หน้านี้เพื่อลด bundle size และ optimize การ fetch ข้อมูลใน Server Component
  ```

---

> **Tip สรุป**:
> - **Main Flow** (`grill-me` → `to-spec` → `to-tickets` → `tdd` → `code-review`) = ตัวคุมรอบการทำงาน (Workflow)
> - **Tech Boosters** (`shadcn`, `supabase-postgres`, `vercel-react`) = คลังความรู้คอยเสริมให้โค้ดคุณภาพสูงขึ้นอัตโนมัติ
