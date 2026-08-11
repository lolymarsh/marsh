# Prompt Templates — ใช้คู่กับ Skills ที่ติดตั้งไว้

> รวม prompt template สำหรับงานแต่ละประเภท พร้อมแมปกับ **skill** ที่ควรใช้
> (ทั้ง 4 project skills + 15 global skills ที่ ~/.agents/skills/)
>
> **วิธีใช้**: คัดลอก template → เติม `[วงเล็บเหลี่ยม]` → ส่งให้ AI
> ถ้าอยากบังคับใช้ skill ให้ **เอ่ยชื่อ skill ใน prompt** ตาม format ข้างล่าง

---

## 0. ตารางเลือก skill ให้ตรงกับงาน (อ่านก่อน!)

| งาน | Skill ที่ใช้ | Template # |
|---|---|---|
| เขียนแผน/PRD ใหม่ | `writing-plans` + `to-spec` | #1 |
| Implement Go REST API | `implement-go-restapi` | #2 |
| Implement React (Vite MVC) | `implement-fe-react` + `vercel-react-best-practices` | #3 |
| Implement Next.js (App Router) | `implement-fe-nextjs` + `vercel-react-best-practices` | #4 |
| Implement Python FastAPI | `implement-python-restapi` | #5 |
| แก้บั๊ก (debug) | `diagnosing-bugs` + project skill ตาม stack | #6 |
| Review โค้ด / ตรวจงาน AI | `code-review` (แยกผู้ตรวจ read-only) | #7 |
| เก็บข้อมูล/ส่งต่องานระหว่าง session | `handoff` | #8 |
| ออกแบบ UI/ปรับหน้าตา | `frontend-design` + `web-design-guidelines` + `shadcn` | #9 |
| งาน PostgreSQL/SQL | `supabase-postgres-best-practices` | #10 |
| ทำ TDD เข้มๆ | `tdd` (ใช้คู่กับทุก implement template) | ใช้เป็น prefix |
| ปรับปรุงสถาปัตยกรรม | `improve-codebase-architecture` | #11 |
| อยากได้ skill เพิ่ม | `find-skills` | — |

**Format การบังคับใช้ skill** (เพิ่มไว้ต้น prompt):
```
ใช้ [skill-name]:
<งานที่ต้องการ>
```

---

## #1 — เขียน Plan / Spec / PRD

```text
ใช้ writing-plans + to-spec:

ช่วยวางแผน [ชื่อโปรเจกต์/ฟีเจอร์] ให้หน่อย

บริบท:
- เป้าหมาย: [อยากได้อะไร สําหรับใคร แก้ปัญหาอะไร]
- stack: [เช่น Go + Echo + MySQL / React 19 + Vite + TS]
- ข้อจำกัด: [เช่น ไม่มี deploy เอาไว้ศึกษา / ใช้ MySQL แทน Postgres]
- deadline/ขนาด: [เช่น 2 สัปดาห์ / MVP ก่อน]

สิ่งที่อยากได้:
1. plan.md — master plan: modules, phases, dependencies, timeline
2. แยกเป็น phase เล็กๆ ที่ทำทีละอันได้ (1 phase = 1 commit)
3. แต่ละ phase มี: scope + ไม่เอา + acceptance criteria + คำสั่ง verify
```

**ผลลัพธ์ที่ควรได้**: `plan.md` + `spec/YYYY-MM-DD_<track>/` ตาม convention ใน `01_AI_WORKFLOW.md`

---

## #2 — Implement Go REST API Module

```text
ใช้ implement-go-restapi:

implement phase [NN] ตาม spec/[path]/[NN_NAME].md
ตาม AGENTS.md rules + rules/go-rest-api/* + ARCHITECTURE.md patterns

สิ่งที่ต้องการ:
- module: [เช่น customer] → internal/customer/ (entity, repo, service, handler, request, route)
- test-first: เขียน integration test ที่ fail ก่อน (success + error + wrong role → 403)
- เดินสาย DI ใน internal/server/di.go

ห้าม:
- ใช้ concrete type ใน layer — ต้องเป็น interface
- ละ pagination / version check / transaction

เสร็จแล้วรันให้ครบแล้วรายงานผล: make fmt && make vet && make lint && go test ./...
```

---

## #3 — Implement React Feature (Vite + MVC)

```text
ใช้ implement-fe-react + vercel-react-best-practices:

implement feature [ชื่อ] ตาม spec/[path].md
ตาม rules/typescript-frontend-react/* (React MVC pattern)

สิ่งที่ต้องการ:
- modules/[domain]/: model.ts (API + types + Zod) → controller.ts (useXxx hook) → view.tsx (props only)
- test-first: controller test mock API layer (vi.hoisted ก่อน vi.mock), view test mock hook
- cover: success + error + empty/loading

ห้าม:
- model.ts import React / view.tsx เรียก API ตรงๆ
- ใช้ any / as — ใช้ unknown + Zod .parse()

เสร็จแล้วรัน: npm run typecheck && npm run lint && npm run format && npm run test
```

---

## #4 — Implement Next.js Feature (App Router)

```text
ใช้ implement-fe-nextjs + vercel-react-best-practices:

implement [หน้า/ฟีเจอร์] ตาม spec/[path].md
ตาม rules/typescript-frontend-react/*

สิ่งที่ต้องการ:
- server components โดย default, "use client" เฉพาะที่จำเป็น (thin)
- data fetching ใน RSC / server component — client components รับ props
- forms: React Hook Form + Zod
- loading.tsx + error.tsx สำหรับ route ที่ fetch ข้อมูล

ห้าม:
- fetch ใน client component
- ใส่ secret/env ใน client (ยกเว้น NEXT_PUBLIC_*)

เสร็จแล้วรัน: npm run lint && npx tsc --noEmit && npm run build && npm run test
⚠️ npm run build สำคัญสุด — จับ RSC/SSR/hydration error ที่ lint/test ไม่เจอ
```

---

## #5 — Implement Python FastAPI Module

```text
ใช้ implement-python-restapi:

implement module [ชื่อ] ตาม spec/[path].md
ตาม rules/python-rest-api/*

สิ่งที่ต้องการ:
- src/[pkg]/domain/[module]/: entity.py, schema.py, repo.py, service.py + routes ใน api/[module].py
- test-first: service test mock repo + API test ผ่าน AsyncClient
- type hints ครบทุก param/return (ruff ANN + mypy strict)
- cover: success + error + auth (401/403)

ห้าม:
- Optional[X] / List[X] — ใช้ X | None / list[X]
- repository raise exception — return None สำหรับ not-found

เสร็จแล้วรัน: ruff check src/ && ruff format src/ && mypy src/ && pytest --cov=src/ tests/
```

---

## #6 — แก้บั๊ก (Bug Fix)

```text
ใช้ diagnosing-bugs + [implement-go-restapi / implement-fe-react / ...]:

มีบั๊ก: [อาการ เช่น customer list กดหน้าถัดไปแล้ว error 500 / ข้อมูลไม่อัปเดต]
ที่ไฟล์: [ไฟล์/โมดูลที่เกี่ยวข้อง ถ้ารู้]

ขั้นตอนที่ต้องการ:
1. หา root cause ก่อน — วิเคราะห์ flow + อ่านโค้ดที่เกี่ยวข้อง อย่าเพิ่งแก้
2. เขียน failing test ที่ reproduce บั๊ก ให้เห็นว่า fail จริง
3. แก้ให้ test ผ่านเท่านั้น — ห้าม refactor อย่างอื่น / ห้ามแตะไฟล์นอก scope
4. รัน regression ทั้งหมด + รายงาน: root cause + อะไรแก้ + test ไหนเพิ่ม

ห้าม: แก้ตามอาการโดยไม่รู้ root cause / แก้กว้างเกิน scope
```

---

## #7 — Review โค้ด / ตรวจงาน AI (แยกผู้ตรวจ)

```text
ใช้ code-review:

ตรวจสอบงานล่าสุดแบบเป็นฝ่ายจับผิด (คุณไม่ใช่คนเขียน):
1. เทียบกับ spec/[path].md ทีละข้อ — ทำครบจริงมั้ย
2. หา bug/edge case: null / error path / race / auth
3. รัน verify จริง: [คำสั่งตาม stack] แล้วรายงานผล
4. ห้ามแก้โค้ด — รายงานเป็น list:
   [CRITICAL]/[MAJOR]/[MINOR] + ไฟล์:บรรทัด + เหตุผล + วิธีแก้ที่แนะนำ
```

> ใช้คู่กับ `03_ai-verify-guide.md` — ผู้ตรวจต้องเป็นคนละ agent/session กับผู้เขียน

---

## #8 — Handoff: เก็บสถานะ / ส่งต่องาน

```text
ใช้ handoff:

สรุปสถานะงานปัจจุบันเป็น handoff doc (docs/handoff-[วันที่].md):
- งานที่ทำเสร็จแล้ว (พร้อม commit/phase)
- งานที่กำลังทำ (ค้างตรงไหน อะไรยังไม่จบ)
- งานถัดไปที่ต้องทำ (ตาม spec phase ไหน)
- ไฟล์ที่เกี่ยวข้อง + traps/ข้อควรระวัง
- ห้ามเอาโค้ด/diff มาแปะทั้งก้อน — สรุปเป็นข้อความ

session ถัดไปจะเปิดอ่านไฟล์นี้แทนการคุย history ทั้งหมด
```

---

## #9 — ออกแบบ UI / ปรับหน้าตา

```text
ใช้ frontend-design + web-design-guidelines + shadcn:

ออกแบบ/ปรับปรุง UI ของ [หน้า/component]:
- เป้าหมาย: [เช่น เพิ่ม conversion / ทำให้อ่านง่าย / ตาม brand]
- stack: [shadcn/ui + Tailwind + MUI]
- โฟกัส: hierarchy, contrast, hover/micro-interaction, responsive
- ขอให้ใช้ component ที่มีอยู่ก่อน ถ้าไม่มีค่อยสร้างใหม่ (ห้ามสร้างซ้ำ)
```

---

## #10 — งาน PostgreSQL / SQL

```text
ใช้ supabase-postgres-best-practices:

ช่วยเขียน/ปรับ query หรือ schema สำหรับ:
- ตาราง/query: [เช่น dashboard ยอดขายรายเดือน]
- ปัญหา: [เช่น ช้า / index ไม่โดน / N+1]
- ขอ: EXPLAIN วิเคราะห์ + index ที่แนะนำ + migration SQL
```

---

## #11 — ปรับปรุงสถาปัตยกรรม

```text
ใช้ improve-codebase-architecture:

สแกนโปรเจกต์นี้ หาโอกาสปรับปรุงสถาปัตยกรรม:
1. สรุปจุดอ่อน/ความเสี่ยง (coupling, duplicated code, layer violation)
2. จัดลำดับตาม impact กับ effort
3. เลือก [อันที่ 1] → เสนอแผน refactor ทีละขั้น (เล็กลงจน commit ได้)
4. ห้าม refactor ทันที — ขอแผนก่อน ผม approve แล้วค่อยทำ
```

---

## ⚠️ กฎการใช้ template (สำคัญ)

1. **เอ่ยชื่อ skill ใน prompt** — นี่คือกลไกบังคับให้ agent โหลด skill (ไม่ใช่แค่ "ทำตามที่เขียน")
2. **ทุก implement ต้อง test-first** — ถ้าไม่เห็นคำว่า failing test ใน template ให้เติมเอง
3. **ทุกงานต้องมี verify** — คำสั่งท้าย template ห้ามตัด (lint/test/build คือเกราะจริง)
4. **ห้าม AI ตรวจงานตัวเอง** — ใช้ #7 แยก session/agent เสมอ (ดู `03_ai-verify-guide.md`)
5. **ปรับ stack ให้ตรงโปรเจกต์** — template #2–#5 เขียนตาม marsh rules ถ้าโปรเจกต์อื่นต่างจากนี้
   ให้บอก AI ให้อ่าน AGENTS.md ของโปรเจกต์นั้นก่อน (ดู 01_AI_WORKFLOW.md → "เริ่มงานตามสถานการณ์โปรเจกต์")
6. **skill บางตัวช่วยเลือกเองได้** — ถ้าไม่แน่ใจใช้ตัวไหน พิมพ์ "ใช้ find-skills: หา skill ที่เหมาะกับ [งาน]"

---

> สร้าง: 2026-08-11 (Freebuff)
> อ้างอิง: skills/ 4 project skills + ~/.agents/skills/ 15 global skills + 01_AI_WORKFLOW.md
