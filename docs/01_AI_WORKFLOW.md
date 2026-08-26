# 01_AI_WORKFLOW.md — คู่มือ AI Coding Workflow สำหรับ SWE

> **One-Stop Practical Guide**: ลำดับการทำงานร่วมกับ AI ตั้งแต่ตั้งโจทย์ → วาง Architecture → สเปก → เขียนโค้ดแบบ TDD → ตรวจสอบคุณภาพ
> อ้างอิง Best Practices จาก `aihero.dev/skills` (Matt Pocock) ผสาน System Design & Fullstack SWE จริง

---

## 🗺️ ภาพรวม Workflow หลัก (The 5-Phase Lifecycle)

```
[ 1. REQUIREMENTS & ARCHITECTURE ] ──► บอก Requirement สั้นๆ → AI Grill ตีกรอบ Architecture
          │
          ▼
[ 2. SPEC & BASELINE ] ──────────────► AI สังเคราะห์เป็น Spec + วาง Data Flow & DB Schema
          │
          ▼
[ 3. VERTICAL TICKETS ] ─────────────► แตกตั๋วงานย่อยแบบ Tracer-bullet (Vertical Slice)
          │
          ▼ 🎯 [ จบ Session วางแผน / เปิด Session ใหม่ ]
[ 4. IMPLEMENTATION (TDD) ] ─────────► 1 Ticket = 1 Session สะอาด (Failing Test → โค้ดจนผ่าน)
          │
          ▼
[ 5. DUAL REVIEW & COMMIT ] ─────────► ตรวจ Spec Match + Coding Standards ก่อน Commit
```

---

## 🧭 Architecture Decision Guide (กรอบคิด Global ก่อนลงลึก)

ก่อนเริ่มลงมือ ให้เช็ก Decision Matrix นี้เพื่อบอกกรอบให้ AI ตั้งแต่แรก ไม่ให้เลือกเครื่องมือผิดขนาด:

```
[ โจทย์ของระบบ / ขนาดโปรเจกต์ ]
         │
         ├─► [ งานเล็ก / Internal Tools / Solo Dev ]
         │         └── 💡 Tier 1: Postgres-Only Lean Stack
         │               ├── DB: PostgreSQL (ACID)
         │               ├── Queue: Postgres SKIP LOCKED (River / pg-boss)
         │               ├── Realtime: Postgres LISTEN/NOTIFY + SSE
         │               └── Cache: Postgres UNLOGGED TABLE
         │
         ├─► [ งานขนาดกลาง / โหลดอ่านสูง ]
         │         └── 🚀 Tier 2: Hybrid Stack
         │               ├── DB: PostgreSQL
         │               ├── Cache & Queue: Redis (Cache-Aside + Streams)
         │               └── Realtime: Server-Sent Events (SSE)
         │
         └─► [ งาน Enterprise / โหลดหนัก / ซับซ้อน ]
                   └── 🏢 Tier 3: Distributed Stack
                         ├── DB: PostgreSQL + Read Replicas
                         ├── Cache & Lock: Redis Cluster
                         ├── Queue: RabbitMQ (DLQ + Topic Routing)
                         └── Realtime: WebSockets (Full-duplex)
```

### 💡 The "Postgres-Only" Lean Stack (สำหรับงานเล็ก / Internal Tools)
สำหรับงานขนาดเล็กถึงปานกลาง หรือโปรเจกต์ภายในบริษัท **PostgreSQL ตัวเดียวทำได้ครบทุกอย่าง** โดยไม่ต้องเปิด Redis หรือ RabbitMQ ให้เปลือง Infra:

| ความสามารถ | วิธีทำใน PostgreSQL | จุดเด่น / เครื่องมือแนะนำ |
|---|---|---|
| **Primary Database** | Relational Tables + JSONB | ACID 100%, Flexible Schema |
| **Transactional Queue** | `SELECT ... FOR UPDATE SKIP LOCKED` | **Transactional Job**: สร้างข้อมูลและลง Queue ใน DB Transaction เดียวกันได้ทันที ไม่มีปัญหา Dual-write หลุด!<br>*(Go: `riverqueue/river`, Node: `pg-boss` / `graphile-worker`)* |
| **Realtime / PubSub** | `LISTEN` / `NOTIFY` | ยิง Event เตือน Backend แล้วแปลงเป็น **SSE** ส่งหา Client ได้ทันทีโดยไม่ต้องมี Broker เพิ่ม |
| **Cache / Temporary** | `UNLOGGED TABLE` + Index | ไม่เขียน Disk WAL ทำให้เร็วใกล้เคียง In-Memory สำหรับ Session หรือ Cache ชั่วคราว |

---

### 📋 Cheatsheet สรุปการเลือกใช้ตามขนาดงาน:

1. **Tier 1: Lean Stack (PostgreSQL เดี่ยวๆ)**:
   - เหมาะกับ: Internal Tools, MVP, Solo Dev, ผู้ใช้งาน < 1,000 Concurrent
   - ข้อดี: Infra กล่องเดียวจบ Backup ง่าย ไม่ต้องต่อหลาย Service
2. **Tier 2: Hybrid Stack (PostgreSQL + Redis + SSE)**:
   - เหมาะกับ: Production App ทั่วไป, มีการอ่านข้อมูลซ้ำๆ บ่อย (Cache-aside), งาน Realtime ไม่หนัก
3. **Tier 3: Distributed Stack (PostgreSQL + Redis + RabbitMQ + WebSocket)**:
   - เหมาะกับ: งาน Enterprise, ต้องการ Dead Letter Queue (DLQ) ซับซ้อน, Multi-worker Routing, หรือแชทโต้ตอบ Realtime 2 ทาง

---

## 🎯 เลือกทำตามสถานการณ์ (5 กรณี) + Prompt สำเร็จรูป

---

### 🟢 กรณี 1: เริ่มโปรเจกต์ใหม่ / ฟีเจอร์ใหม่ (The Main Flow)

#### ลำดับขั้นตอน:
1. **เปิด Session 1 (Planning)**: คุยในแชทเดียวจนได้ Spec และ Tickets
2. **2-Phase Grill**: ให้ AI ซักถาม 2 ระดับ:
   - *Phase A (Global Architecture)*: สรุปเลือก Tools & Boundaries (DB, Queue, Realtime)
   - *Phase B (Technical Details)*: เจาะลึก Schema, API Contract, Event Payload, FE State
3. **to-spec**: ให้ AI สรุปเป็นไฟล์ Spec และ ARCHITECTURE.md
4. **to-tickets**: แตกตั๋วงานย่อยแบบ **Vertical Slice** (ไม่ใช่แยก Backend/Frontend ทั้งหมด)
5. **Commit Baseline** $\rightarrow$ **ปิด Session 1**
6. **เปิด Session 2..N (Implement)**: รันทีละตั๋ว (1 Ticket = 1 Session) ด้วย `tdd` $\rightarrow$ `code-review`

---

#### 📋 Prompt Templates สำหรับกรณี 1 (ก๊อปปี้ไปปรับใช้):

#### 🔹 [Step 1: สั่ง Grill-me แบบ Global-to-Specific]
```text
ใช้ grill-me:
ผมต้องการทำระบบ/ฟีเจอร์: [เช่น ระบบจองห้องประชุมพร้อมแจ้งเตือน Realtime]

เป้าหมาย & ปัญหาที่ต้องการแก้:
- [อธิบายสั้นๆ 2-3 บรรทัด เช่น พนักงานกดจองห้อง มีการแจ้งเตือนหน้าจอทันที และมี worker ส่งอีเมลยืนยัน]

Tech Stack ที่ต้องการใช้:
- Strategy: [เลือก: 1) Postgres-Only Lean Stack หรือ 2) Hybrid Postgres+Redis หรือ 3) Distributed Enterprise]
- Database: PostgreSQL
- Cache / In-Memory: [Postgres UNLOGGED / Redis / None]
- Queue / Worker: [Postgres SKIP LOCKED (River/pg-boss) / Redis Streams / RabbitMQ / None]
- Realtime: [Postgres LISTEN/NOTIFY + SSE / WebSocket / None]
- Frontend: [เช่น Next.js App Router + TanStack Query + Tailwind + shadcn]
- Backend: [เช่น Go REST API หรือ NestJS]

ช่วยสัมภาษณ์ผมแบบเจาะลึก 2 ขั้นตอน (ห้ามข้าม):
1. Global Architecture & Boundaries: จุดไหน Sync/Async, SLA & Concurrency, Failure recovery
2. Technical Details: Postgres Schema + Indexes, Caching/Keys, Event Payloads + DLQ, Realtime Channels, API DTO, Frontend State Reconciliation
```

#### 🔹 [Step 2: สังเคราะห์เป็น Spec]
```text
ใช้ to-spec:
สรุปข้อตกลงทั้งหมดลงใน spec/plan.md และแยกรายละเอียดย่อยลง spec/[module]/ (เช่น 01_plan.md, 02_api.md, 03_schema.md, 04_frontend.md) โดยระบุ:
1. Topology Diagram & Data Flow
2. Database Schema (DDL + Constraints + Index)
3. Queue / Event Payload Schema (ถ้ามี)
4. API Contract & Realtime Protocol
5. Frontend UI/UX Flow & Optimistic Updates
```

#### 🔹 [Step 3: แตกตั๋วแบบ Vertical Slice]
```text
ใช้ to-tickets:
ช่วยแตกตั๋วงานย่อยจาก spec/[module]/ ลงใน spec/[module]/04_ticket.md (หรือแยก 04_ticket01.md, 04_ticket02.md... ถ้าตั๋วเยอะ)
โดยจัดเรียงตั๋วแบบ Vertical Slice (Tracer-bullet) ให้แต่ละตั๋วเทส End-to-End ได้:
- Ticket 01: Core Ingestion (DB Migration + CRUD API)
- Ticket 02: Async Pipeline (Queue Setup + Worker + Idempotency Test)
- Ticket 03: Realtime Propagation (SSE/WS Gateway + Events)
- Ticket 04: Frontend Client & Live UI (Hook + Optimistic UI)
```

#### 🔹 [Step 4: เปิด Session ใหม่ เพื่อเริ่มโค้ด (1 Ticket = 1 แชทใหม่)]
```text
อ่าน AGENTS.md และทำตามตั๋วใน: spec/[module]/04_ticket.md (หรือ 04_ticket01.md)
ใช้ skill: tdd
เขียน Failing Test ก่อนเสมอ แล้วค่อยเขียนโค้ดจนผ่าน
```

#### 🔹 [Step 5: ตรวจสอบก่อน Commit]
```text
ใช้ code-review:
ตรวจโค้ดที่เพิ่งเขียนเทียบกับตั๋วใน spec/[module]/04_ticket.md (หรือ 04_ticket01.md) และกฎใน AGENTS.md
พร้อมเช็ก Security (SQL Injection, IDOR, Unhandled Error)
```

---

### 🟡 กรณี 2: ฟีเจอร์เดิมเปลี่ยน Flow / เปลี่ยน Requirement / เปลี่ยน Tech Stack

```
[ Session วางแผน ] ──► อัปเดตไฟล์ spec/[module]/ เดิม (In-place) → รัน to-tickets อัปเดต 04_ticket.md
          │
          ▼ 🎯 [ จบ Session / เปิดแชทใหม่ ]
[ Session โค้ด ]   ──► เปิดแชทใหม่ รัน tdd (แก้ Test เดิมให้ Red → โค้ดจน Green)
```

#### 📋 Prompt Templates สำหรับกรณี 2:

##### 🔹 [2.1 เปลี่ยน Business Flow / Requirements]
```text
อัปเดตไฟล์ spec/[module]/01_plan.md:

เรามีการเปลี่ยน Flow การทำงาน:
- Flow เดิม: [เช่น ส่งอีเมลแบบ Sync ภายใน API Request]
- Flow ใหม่: [เช่น ยิง Job ลง Redis Queue ให้ Worker ส่งเบื้องหลัง และตอบ 202 Accepted]
- เหตุผล: [เช่น ลด Response Time ของหน้าบ้านจาก 3s เหลือ < 100ms]

สิ่งที่ต้องทำ:
1. ปรับปรุง Architecture, Schema, และ Sequence ในไฟล์ spec เดิม (ห้ามสร้างไฟล์ใหม่)
2. รัน to-tickets เพื่ออัปเดตตั๋วงานย่อยชุดใหม่ลงใน spec/[module]/04_ticket.md
```

##### 🔹 [2.2 เปลี่ยน Tech Stack / Architecture กระทันหัน]
```text
อัปเดตไฟล์ spec/[module]/01_plan.md และ spec/[module]/ (In-place):

เรามีการเปลี่ยน Tech Stack กระทันหัน:
- Stack เดิม: [เช่น Redis Streams สำหรับ Async Queue + Node.js Worker]
- Stack ใหม่: [เช่น Postgres SKIP LOCKED ผ่าน River Queue + Go Worker เพื่อลด Infra]
- สาเหตุ & ขอบเขต: [เช่น ปรับตาม Lean Stack เพื่อลด Dependency ภายนอกและรับประกัน Transactional Safety]

สิ่งที่ต้องทำ:
1. ปรับปรุง Architecture Diagram, Data Flow, Sequence, Config/ENV และ Dependencies ในไฟล์ spec เดิมทั้งหมด
2. รัน to-tickets เพื่อ Re-generate ตั๋วงานย่อยชุดใหม่ลงใน spec/[module]/04_ticket.md (ปรับ Acceptance Criteria และ Tooling ให้เข้ากับ Stack ใหม่)
```

---

### 🔴 กรณี 3: งานแก้บั๊ก / Hotfix / ปัญหา Production

> ⚠️ **ไม่ต้องทำ Spec** — เปิดแชทใหม่แบบ Clean Context 100% ทันที

```
[ Error Log / บั๊ก ] ──► /diagnosing-bugs (หา Root Cause) 
                             │
                             ▼
                        /tdd (เขียน Failing Test ยืนยันว่าพังจริง)
                             │
                             ▼
                        แก้โค้ดเฉพาะจุดจนผ่าน → /code-review → Commit
```

#### 📋 Prompt Template สำหรับกรณี 3:
```text
มีบั๊กในระบบ: [อธิบายอาการ เช่น พนักงานกดยกเลิกออเดอร์แต่สต็อกไม่คืน หรือ SSE หลุดแล้วไม่ Reconnect]
Log / ข้อมูลเพิ่มเติม: [แนบ Error Log หรือ Step to reproduce]

คำสั่ง:
1. ใช้ diagnosing-bugs: วิเคราะห์หาสาเหตุที่แท้จริง (Root Cause) จากโค้ดปัจจุบัน
2. ใช้ tdd: เขียน Failing Unit/Integration Test เพื่อจำลองบั๊กนี้ให้เห็นชัดเจนก่อน
3. แก้ไขโค้ดเฉพาะจุดที่ผิดพลาดจน Test ผ่าน (ห้าม Refactor หรือแตะโค้ดส่วนอื่นที่ไม่เกี่ยวข้อง)
4. ใช้ code-review ตรวจสอบ Side-effects ก่อนจบงาน
```

---

### 🟣 กรณี 4: เข้าไปแทรกงานกลางโปรเจกต์ / โค้ดคนอื่น

```
[ สำรวจระบบเดิม ] ──► ให้ AI อ่านโค้ดและสรุป Architecture → เขียน Mini-spec 1 หน้า → แตกตั๋ว
```

#### 📋 Prompt Template สำหรับกรณี 4:
```text
สำรวจโปรเจกต์นี้ก่อน:
1. อ่านโครงสร้างโปรเจกต์, README, และ AGENTS.md แล้วสรุป Tech Stack + Data Flow สั้นๆ
2. เราต้องการต่อเติมฟีเจอร์: [เช่น เพิ่มระบบ Export ข้อมูลเป็น Excel แบบ Async]
3. ช่วยเขียน Mini-Spec 1 หน้าลงใน spec/[module]/01_plan.md และแตกตั๋วด้วย to-tickets
(ห้ามแตะต้องหรือ Refactor โค้ดเดิมของระบบ)
```

---

### 🧰 กรณี 5: โปรเจกต์ค้าง / Context ใกล้เต็ม / ส่งต่องานข้ามกะ

```
[ แชทเดิม ]   ──► รัน /handoff สรุปสถานะลง docs/handoff-[วันที่].md → ปิดแชท
          │
          ▼
[ แชทใหม่ ]   ──► เปิดแชทใหม่ สั่งอ่าน handoff.md และทำ Next Step ต่อทันที
```

#### 📋 Prompt Template สำหรับกรณี 5:
* **ตอนจบรอบ (ในแชทเดิม)**:
  ```text
  ใช้ handoff:
  ช่วยสรุปสถานะงานลง docs/handoff-[YYYY-MM-DD].md: สิ่งที่เสร็จแล้ว, สิ่งที่ค้างอยู่, และ Next Step ข้อถัดไป
  ```
* **ตอนเริ่มรอบใหม่ (เปิดแชทใหม่)**:
  ```text
  อ่าน docs/handoff-[YYYY-MM-DD].md และ AGENTS.md แล้วทำงานในข้อ Next Step ถัดไป ด้วย skill: tdd
  ```

---

## ⚡ Skills Reference Cheatsheet

| หมวด | Skill | หน้าที่หลัก | คำสั่งเรียกใช้เร็ว |
|---|---|---|---|
| **Main Flow** | `grill-me` | สัมภาษณ์ Requirements & Architecture | `ใช้ grill-me: ...` |
| | `to-spec` | สรุปเป็นเอกสาร Spec & Data Flow | `ใช้ to-spec: ...` |
| | `to-tickets` | แตกตั๋วงานย่อย Tracer-bullets | `ใช้ to-tickets: ...` |
| | `tdd` | บังคับเขียน Failing Test ก่อนโค้ด | `ใช้ tdd: ...` |
| | `code-review` | ตรวจ Spec Match + Coding Standards | `ใช้ code-review: ...` |
| | `diagnosing-bugs` | สืบหา Root Cause ของบั๊กก่อนแก้ | `ใช้ diagnosing-bugs: ...` |
| | `handoff` | บันทึกย่อ Context ส่งต่องานข้าม Session | `ใช้ handoff: ...` |
| **Tech Boosters** | `supabase-postgres-best-practices` | Best practices & Index tuning ของ Postgres | `ใช้ supabase-postgres-best-practices: ...` |
| | `shadcn` | ติดตั้ง/จัดแต่ง UI Component (React/Next) | `ใช้ shadcn: ...` |
| | `vercel-react-best-practices` | Optimize Frontend & Next.js Performance | `ใช้ vercel-react-best-practices: ...` |

---

## 🔒 กฎเหล็กประจำตัว (Rules of Thumb)

1. **ห้ามเริ่มโค้ดถ้ายังไม่มีตั๋ว/Spec**: เสียเวลาคุย Architecture 10 นาที ประหยัดเวลาแก้โค้ด 3 วัน
2. **1 Ticket = 1 Session สะอาดเสมอ**: เมื่อตั๋วเสร็จ $\rightarrow$ Commit $\rightarrow$ ปิดแชท $\rightarrow$ ขึ้นแชทใหม่
3. **อย่าให้ AI ตรวจงานตัวเองในรอบเดียวกัน**: ให้ใช้ `code-review` หรือตรวจแยก session เพื่อหลีกเลี่ยง Confirmation Bias
4. **ความปลอดภัยเป็นอันดับหนึ่ง**: ห้าม Hardcode Secrets, ใช้ Parameterized Query เสมอ, และเช็ก RBAC ทุก Endpoint
