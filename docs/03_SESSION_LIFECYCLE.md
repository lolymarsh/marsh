# 04_SESSION_LIFECYCLE.md — แผนการแบ่ง Session AI ในทางปฏิบัติ

> คู่มือการบริหารจัดการ AI Context & Session: เมื่อไหร่ควรคุยต่อในแชทเดิม เมื่อไหร่ต้องขึ้นแชทใหม่ เพื่อคุณภาพโค้ดสูงสุดและประหยัด Token

---

## 💡 กฎเหล็กของ Session Management

> **"1 Session ไม่ควรยาวเกิน 1 Phase หรือ 1 Ticket ใหญ่"**
> **"ห้ามใช้ Session เดียวตั้งแต่คิดไอเดียจนเขียนโค้ดเสร็จทั้งโปรเจกต์"**

### ทำไมต้องแบ่ง Session?
1. **Context Window Degradation**: ยิ่งบทสนทนายาวเกิน 30-50 ข้อความ AI จะเริ่มหลงลืมกฎใน `AGENTS.md` และสร้างโค้ดที่มี Hallucination สูงขึ้น
2. **Context Pollution (มลพิษทางบริบท)**: การคุยเรื่อง Requirement/Discussion ค้างไว้ในแชท จะทำให้ AI สับสนกับตอนเขียนโค้ดจริง
3. **Token & Cost Efficiency**: ประหยัด Token มหาศาล ไม่ต้องเสียเวลาส่งประวัติแชทเก่าๆ วนกลับไปทุกคำสั่ง

---

## 🗺️ ภาพรวม Session Lifecycle (Idea → Ship)

```
[🟢 Session 1: Planning & Setup (คุยต่อในแชทเดิมได้ตลอดช่วงนี้)]
  ไอเดีย / โจทย์ใหม่
       │
       ▼
  1. /grill-me (AI ซักถาม Requirement & Edge cases)
       │
       ▼
  2. /to-spec (AI สรุปเป็นไฟล์ Spec ชัดเจน)
       │
       ▼
  3. /to-tickets (AI แตกเป็นตั๋วงานย่อย Tracer tasks)
       │
       ▼
  4. สร้าง ARCHITECTURE.md + AGENTS.md
       │
       ▼
  5. jj describe -m "spec: initial plan, architecture and tickets"
       │
═══════╪══════════════════════════════════════════════════════════
       ▼ 🎯 จบ Session 1 — [ ปิดแชทเดิม / เปิดแชทใหม่ ]
═══════╪══════════════════════════════════════════════════════════
       │
[🔵 Session 2: Implement Ticket 01 (Clean Context 100%)]
  1. เปิดแชทใหม่: "ทำ Ticket 01 ตาม AGENTS.md ด้วย skill tdd"
       │
       ▼
  2. /tdd (เขียน failing test ก่อน → โค้ดจนผ่าน)
       │
       ▼
  3. /code-review (ตรวจเทียบ Spec Match + Standards)
       │
       ▼
  4. jj describe -m "feat(module): ticket 01 description"
       │
═══════╪══════════════════════════════════════════════════════════
       ▼ 🎯 จบ Session 2 — [ ปิดแชทเดิม / เปิดแชทใหม่ ]
═══════╪══════════════════════════════════════════════════════════
       │
[🔵 Session 3..N: Implement Ticket 02..N (เปิดแชทใหม่ทุกตั๋ว)]
  ทำซ้ำลูปเดียวกับ Session 2 จนครบทุก Tickets
```

---

## 📋 รายละเอียดแต่ละ Session

---

### 🟢 Session 1: Phase วางแผนและสร้างมาตรฐาน (Planning & Baseline)

* **เป้าหมาย**: เปลี่ยนไอเดียดิบให้กลายเป็น **Spec + Architecture Rules + Tickets** ที่พร้อมนำไปเขียนโค้ด
* **ลักษณะ Session**: **คุยต่อเนื่องในแชทเดิมได้ตลอดทั้ง Phase นี้** เพราะต้องต่อยอดบทสนทนา

| ลำดับคำสั่ง | Action / Skill | ผลลัพธ์ที่ต้องได้ |
|---|---|---|
| 1 | `/grill-me` | AI ซักถามโจทย์, Non-functional requirements, Edge cases |
| 2 | `/to-spec` | ไฟล์ Spec เช่น `spec/2026-xx-xx_module/01_SPEC.md` |
| 3 | `/to-tickets` | รายการตั๋วงานย่อย เช่น `tickets/TICKET-01.md`, `02.md` |
| 4 | Manual Review | ตรวจสอบไฟล์ `ARCHITECTURE.md` และดึง `rules/` ลง `AGENTS.md` |
| 5 | **VCS Commit** | `jj describe -m "spec: initial plan, architecture and tickets"` |

> 🛑 **จุดตัด (Boundary)**: หลัง Commit Baseline เสร็จ **ให้ปิดแชทนี้ทันที** อย่าเขียนโค้ดต่อในแชทนี้

---

### 🔵 Session 2..N: Phase เขียนโค้ดรายตั๋ว (1 Ticket = 1 Session)

* **เป้าหมาย**: สร้างโค้ดที่ถูกต้องตามแบบแผนของ 1 Ticket ย่อย
* **ลักษณะ Session**: **เปิดแชทใหม่ทุกครั้ง (Clean Context 100%)**

#### Template คำสั่งเปิดแชทใหม่:
```markdown
อ่าน AGENTS.md และทำตาม ticket: tickets/TICKET-01.md
ใช้ skill: tdd
เขียน unit test ก่อน แล้วค่อย implement โค้ด
```

| ลำดับขั้นตอน | Action / Skill | รายละเอียด |
|---|---|---|
| 1 | **Implement with TDD** | AI สร้าง Failing Test $\rightarrow$ เขียนโค้ดจน Test ผ่าน (Red-Green-Refactor) |
| 2 | `/code-review` | ให้ AI สวมบทบาท Reviewer ตรวจเทียบ 2 มิติ: <br>1) ตรงกับ Ticket ไหม? <br>2) ตรงตาม `AGENTS.md` & `rules/` ไหม? |
| 3 | **Verify & Commit** | รัน linter/test จริง $\rightarrow$ `jj describe -m "feat(module): ticket description"` |

> 🛑 **จุดตัด (Boundary)**: เมื่อ Ticket นั้นเสร็จและ commit แล้ว **ให้ปิดแชททันที** เพื่อเริ่ม Ticket ถัดไปในแชทใหม่

---

### 🔴 Session พิเศษ: การแก้บั๊ก / ปรับปรุงงานด่วน (Bug Fix & Hotfix)

* **เป้าหมาย**: แก้ปัญหาโดยไม่สร้าง Side Effect และไม่แตะต้องโค้ดส่วนอื่น
* **ลักษณะ Session**: **เปิดแชทแยกต่างหากทันที**

| ขั้นตอน | Skill / Action | แนวทางปฏิบัติ |
|---|---|---|
| 1. วิเคราะห์ | `/diagnosing-bugs` | บังคับให้ AI ตั้งสมมติฐานและหา Root Cause จาก log/trace ก่อน ห้ามเดาสุ่ม |
| 2. ดักบั๊ก | `/tdd` | เขียน Regression Test ที่ยืนยันว่าเกิดบั๊กจริง (Test ต้อง Fail ก่อนแก้) |
| 3. แก้ไข | Prompt: "แก้เฉพาะ X ห้าม refactor ส่วนอื่น" | แก้ไขโค้ดจน failing test ผ่าน |
| 4. ตรวจสอบ | `/code-review` + Regression Scan | ตรวจสอบว่าไม่กระทบฟังก์ชันเดิม |
| 5. Commit | `jj describe -m "fix(module): bug summary"` | จบงาน ปิดแชท |

---

## 🧰 เครื่องมือช่วยเชื่อมต่อข้าม Session (`/handoff`)

เมื่อไหร่ที่ต้องใช้ `/handoff`:
1. ทำงานติดพันแต่ **Context ใกล้เต็ม** หรือ AI เริ่มตอบช้า/วน
2. **หมดเวลาทำงาน** ต้องพักกะ แล้วจะกลับมาทำต่อวันพรุ่งนี้
3. ต้องส่งต่องานให้อีก Agent / Model อื่นทำต่อ

### วิธีใช้งาน `/handoff`:
1. พิมพ์สั่ง: `/handoff` ในแชทปัจจุบัน
2. AI จะสรุปสถานะปัจจุบัน:
   - สิ่งที่ทำเสร็จแล้ว
   - สิ่งที่ค้างอยู่ (Next Steps)
   - ปัญหาหรือข้อตกลงที่ค้นพบ
3. เซฟเป็นไฟล์ `handoff.md`
4. เมื่อเปิด Session ใหม่ ให้เริ่มด้วย:
   ```markdown
   อ่าน handoff.md และ AGENTS.md แล้วทำต่องานในข้อถัดไป
   ```
