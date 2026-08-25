# 02_prompt-templates.md — Prompt Templates สำเร็จรูป (Core 7 Templates)

> รวม Prompt Templates หลักที่จำเป็น แมปกับ **Core 10 Skills** และ Session Lifecycle 
> 
> **วิธีใช้**: คัดลอก template → แก้ไขข้อความใน `[วงเล็บเหลี่ยม]` → ส่งให้ AI

---

## 📌 ตารางเลือก Template ตามงาน

| Phase | # | งานที่ต้องการทำ | Skills ที่ใช้ | ใช้ใน Session ไหน |
|---|---|---|---|---|
| **Plan** | **#1** | สัมภาษณ์ → สร้าง Spec → แตกตั๋ว | `grill-me` + `to-spec` + `to-tickets` | 🟢 Session 1 (Planning) |
| **Code** | **#2** | ทำตั๋วงานย่อยด้วย TDD (ราย Ticket) | `tdd` | 🔵 Session 2..N (1 Ticket = 1 Session) |
| **Code** | **#3** | Implement เจาะจง Stack (Go / React / Python) | `rules/*` + `vercel-react-best-practices` | 🔵 Session 2..N (เมื่อไม่มีตั๋วแยก) |
| **Verify** | **#4** | Review โค้ด 2 มิติ (Spec + Standards) | `code-review` | 🔵 ท้าย Session ก่อน Commit |
| **Support** | **#5** | แก้บั๊ก & สืบหา Root Cause | `diagnosing-bugs` + `tdd` | 🔴 Session แยก (Bug Fix) |
| **Support** | **#6** | Handoff สรุปสถานะย่อข้าม Session | `handoff` | 🧰 เมื่อจบรอบงาน / context ใกล้เต็ม |
| **Support** | **#7** | PostgreSQL Schema & Query Optimization | `supabase-postgres-best-practices` | 🔵 เมื่อทำงานเกี่ยวกับ DB |

---

## #1 — วางแผน → สร้าง Spec → แตก Tickets (The Main Flow)

> 🟢 **รันใน Session 1 (คุยต่อเนื่องในแชทเดิม)**

### Step 1.1: สัมภาษณ์ความต้องการ (Grill)
```text
ใช้ grill-me:

อยากทำระบบ/ฟีเจอร์ [ชื่อเรื่อง]
บริบท:
- เป้าหมาย: [อยากได้อะไร สำหรับใคร แก้ปัญหาอะไร]
- Stack: [เช่น Go + Echo + PostgreSQL / React 19 + Vite + TS]
- ขอบเขต: [เช่น Core modules: Auth, Customer, Invoice]
- ข้อจำกัด: [เช่น ไม่มี deploy เอาไว้ศึกษา / ใช้ Zod validate ทุกชั้น]

ช่วยซักถาม/สัมภาษณ์ผมแบบเจาะลึกเพื่อเก็บ Requirement, Edge Cases และ Trade-offs ทั้งหมดก่อนเริ่มทำ Spec
```

### Step 1.2: สังเคราะห์เป็น Spec แยกโฟลเดอร์ (To Spec)
```text
ใช้ to-spec:

สรุปข้อตกลงทั้งหมดลงโฟลเดอร์ spec ตาม convention:
1. สร้าง spec/plan.md (Master Plan ภาพรวม 1 หน้า)
2. สร้างโฟลเดอร์ track: spec/YYYY-MM-DD_[track_name]/
3. แยกไฟล์ spec ตามโมดูลย่อย (ห้ามรวมเป็นไฟล์เดียว):
   - spec/YYYY-MM-DD_[track]/01_FOUNDATION.md
   - spec/YYYY-MM-DD_[track]/02_[MODULE_NAME].md
   ...

แต่ละไฟล์ต้องมี: Scope, Schemas/Data Flow, API/UI Components, Acceptance Criteria และ Verification Commands
```

### Step 1.3: แตก Spec เป็น Tracer Tickets (To Tickets)
```text
ใช้ to-tickets:

ช่วยแตกตั๋ว tracer tickets จากไฟล์ spec:
spec/YYYY-MM-DD_[track]/[NN_MODULE].md

ข้อกำหนด:
1. แตกเป็น Vertical Slices (แต่ละตั๋วทำครบ Schema → API → UI → Test)
2. ระบุ Blocked by ชัดเจน (ตั๋วไหนต้องทำก่อน-หลัง)
3. บันทึกไฟล์ตั๋วลงโฟลเดอร์: tickets/YYYY-MM-DD_[track]/
```

---

## #2 — Implement รายตั๋วด้วย TDD (1 Ticket = 1 Session)

> 🔵 **เปิดแชทใหม่ (Clean Context 100%) ทุกครั้งที่เริ่ม Ticket ใหม่**

```text
อ่าน AGENTS.md และทำตาม ticket:
tickets/YYYY-MM-DD_[track]/[NN_TASK].md

ใช้ skill: tdd

ขั้นตอน:
1. ทำความเข้าใจ Seam (Interface boundary) และ Acceptance Criteria ในตั๋ว
2. เขียน Failing Test ดักจับพฤติกรรมก่อน (Red)
3. เขียน Implementation โค้ดจนกว่า Test จะผ่าน (Green)
4. รันคำสั่ง Verify ตามที่ระบุในตั๋ว + ตรวจ Lint/Typecheck
5. เสร็จแล้วรัน /code-review ตรวจสอบความถูกต้อง
```

---

## #3 — Implement Stack เจาะจง (เมื่อไม่ได้แตกตั๋ว)

### 3.1 Go REST API
```text
ใช้ tdd:

implement โมดูล [ชื่อโมดูล] ตาม spec/[path].md
ตาม AGENTS.md rules + rules/go-rest-api/* + ARCHITECTURE.md

สิ่งที่ต้องการ:
- internal/[module]/: entity, repo, service, handler, request, route
- test-first: integration test (success + error + auth 403)
- เดินสาย DI ใน internal/server/di.go
- ห้ามละ pagination / version check / transaction

เสร็จแล้วรัน: make fmt && make vet && make lint && go test ./...
```

### 3.2 React (Vite MVC) / Next.js
```text
ใช้ tdd + vercel-react-best-practices:

implement ฟีเจอร์ [ชื่อฟีเจอร์] ตาม spec/[path].md
ตาม rules/typescript-frontend-react/* (React MVC Pattern หรือ Next.js RSC)

สิ่งที่ต้องการ:
- modules/[domain]/: model.ts (API + Zod) → controller.ts (Hook) → view.tsx (Pure UI)
- test-first: controller test mock API, view test mock hook
- ห้ามใช้ `any` และห้าม view.tsx เรียก API ตรงๆ

เสร็จแล้วรัน: npm run typecheck && npm run lint && npm run test
```

---

## #4 — Review โค้ด & ตรวจสอบคุณภาพ 2 มิติ

> 🔵 **ใช้ตรวจงานก่อน Commit (แนะนำให้แยกเป็นคนละ Agent/Session กับคนเขียน)**

```text
ใช้ code-review:

ตรวจสอบโค้ดที่เพิ่งแก้ไขล่าสุดแบบเป็นฝ่ายจับผิด (คุณไม่ใช่คนเขียน):
1. ตรวจสอบ Spec Match: เทียบกับ ticket หรือ spec/[path].md ว่าทำครบจริงไหม
2. ตรวจสอบ Standards: ตรวจเทียบกับ AGENTS.md และ rules/*
3. หาบั๊ก/Edge Cases: null check, error handling, race conditions, memory leaks
4. รันคำสั่ง Verify จริงแล้วรายงานผล
5. ห้ามแก้โค้ดเอง — ให้สรุปเป็นลิสต์ [CRITICAL] / [MAJOR] / [MINOR] พร้อมไฟล์:บรรทัด และวิธีแก้
```

---

## #5 — แก้บั๊ก & สืบหา Root Cause (Bug Fix)

> 🔴 **เปิด Session แยกเฉพาะกิจ (ห้ามรวมกับแชทฟีเจอร์)**

```text
ใช้ diagnosing-bugs + tdd:

พบปัญหา/บั๊ก: [อาการ เช่น customer list กดเปลี่ยนหน้าแล้ว error 500]
ไฟล์/โมดูลที่เกี่ยวข้อง: [ถ้ารู้ ระบุที่นี่]

ขั้นตอนที่ต้องการ:
1. หา Root Cause ก่อน — วิเคราะห์ Data Flow และอ่านโค้ด ห้ามเดาสุ่มแก้ตามอาการ
2. เขียน Failing Test ที่ reproduce บั๊กนี้ให้เห็นว่า fail จริง
3. แก้ไขโค้ดเฉพาะจุดจนกว่า test จะผ่าน — ห้ามแตะต้องไฟล์นอก scope หรือ refactor ส่วนอื่น
4. รัน Regression Test ทั้งหมด และสรุปผล: Root Cause คืออะไร + แก้ที่ไหน + เพิ่ม Test อะไร
```

---

## #6 — Handoff: เก็บสถานะ / ส่งต่องาน

> 🧰 **ใช้เมื่อจบรอบการทำงาน, Context ใกล้เต็ม, หรือจะส่งต่องานให้อีก Agent**

```text
ใช้ handoff:

สรุปสถานะงานปัจจุบันเป็น handoff doc ลงไฟล์: docs/handoff-[YYYY-MM-DD].md
- 1. งานที่ทำเสร็จแล้ว (พร้อม commit/ticket id)
- 2. งานที่กำลังทำค้างอยู่ (ติดปัญหาตรงไหน อะไรยังไม่เสร็จ)
- 3. งานถัดไปที่ต้องทำต่อ (Next Steps ตาม ticket ไหน)
- 4. ข้อควรระวัง / traps ที่ค้นพบในรอบนี้
- ห้ามแปะ diff โค้ดทั้งก้อน — ให้สรุปเป็นข้อความกระชับ

(Session ถัดไปจะเปิดอ่านไฟล์นี้เพื่อทำต่อได้ทันที)
```

---

## #7 — PostgreSQL Schema & Query Optimization

```text
ใช้ supabase-postgres-best-practices:

ช่วยออกแบบ/ปรับปรุง Query หรือ Schema สำหรับ:
- ตาราง / งานที่ต้องการ: [เช่น ตาราง invoices หรือ query ยอดขายรายเดือน]
- ปัญหา / เป้าหมาย: [เช่น query ช้า, ต้องการทำ partition, หรือหา index ที่เหมาะสม]
- ขอ: ผลวิเคราะห์ EXPLAIN + คำแนะนำ Index + Migration SQL Script ที่ปลอดภัย
```

---

## ⚠️ กฎสำคัญ 4 ข้อในการใช้ Prompt Templates
1. **เอ่ยชื่อ Skill เสมอ**: การใส่ `ใช้ <skill-name>:` ไว้ต้น prompt ช่วยกระตุ้นให้ AI ดึง Best Practice ประจำตัวมาใช้
2. **Test-First เป็นวินัยหลัก**: ทุกการ Implement ต้องเขียน failing test ก่อนเริ่มเขียนโค้ดจริง
3. **1 Ticket = 1 Session**: ปิดแชทเดิม เปิดแชทใหม่ทุกครั้งที่ขึ้นตั๋วงานใหม่
4. **แยกผู้เขียนกับผู้ตรวจ**: ตอนรัน `#4 code-review` ให้ AI สวมบทบาทเป็น Reviewer ตรวจจับผิดอย่างเดียว
