# opencode skills

## Skills ที่ติดตั้งอยู่ตอนนี้ — Core Set (10 ตัว)

คัดเลือกเฉพาะชุดที่จำเป็น ครอบคลุมทั้ง Main Flow (วางแผน → แตกงาน → โค้ด → ตรวจ) และ Tech Stack สำคัญ ไม่ซ้ำซ้อน:

| # | Skill | ผู้พัฒนา | หน้าที่หลัก |
|---|---|---|---|
| 1 | **grill-me** | mattpocock/skills | **สัมภาษณ์/ซักถาม Requirements & Edge cases ก่อนเริ่มเขียน Spec** |
| 2 | **to-spec** | mattpocock/skills | **สังเคราะห์ข้อสรุปจากการคุยเป็น Spec/PRD อย่างเป็นทางการ** |
| 3 | **to-tickets** | mattpocock/skills | **ซอย Spec ออกเป็น Tracer-bullet tickets ย่อยๆ (Atomic tasks)** |
| 4 | **tdd** | mattpocock/skills | **เขียนโค้ดแบบ Test-Driven Development (Red-Green-Refactor)** |
| 5 | **code-review** | mattpocock/skills | **Dual Review: ตรวจเทียบ Spec Match + Standards (`rules/`, `AGENTS.md`)** |
| 6 | **diagnosing-bugs** | mattpocock/skills | **สืบหา Root Cause ของบั๊กอย่างเป็นวิทยาศาสตร์ ก่อนเริ่มแตะโค้ด** |
| 7 | **handoff** | mattpocock/skills | **สรุป Context ย่อส่วนข้าม Session หรือส่งต่องานให้อีก Agent** |
| 8 | **shadcn** | shadcn/ui | **จัดการ UI components, presets, styling (Frontend)** |
| 9 | **supabase-postgres-best-practices** | supabase/agent-skills | **Best practices & Query performance ของ PostgreSQL (Backend/DB)** |
| 10 | **vercel-react-best-practices** | vercel-labs/agent-skills | **React/Next.js performance patterns & best practices (Frontend)** |

Symlink / Global Path: `~/.agents/skills/` และ `~/.claude/skills/`

---

## ตอบคำถาม: ควรใช้แบบ "ต่อ Session" หรือ "แยก Session"?

> **หลักการที่ดีที่สุดคือ: "แยก Session ตาม Phase ใหญ่ แต่ใช้ Session เดียวกันในลูปย่อย"**

### ทำไมไม่ควรใช้ Session เดียวยาวตั้งแต่ต้นจนจบโปรเจกต์?
1. **Context Window บวม**: ยิ่งคุยยาว AI จะเริ่มหลง ลืมกฎใน `AGENTS.md` หรือแก้โค้ดหลุดสโคป
2. **เปลือง Token**: ทุกครั้งที่ส่ง prompt ใหม่ AI ต้องอ่านประวัติแชทเก่าซ้ำทั้งหมด
3. **เสี่ยง Hallucination**: AI จะเริ่มสับสนระหว่าง "สิ่งที่เพิ่งคุย" กับ "สิ่งที่ทำเสร็จไปแล้ว"

---

## 🎯 แนะนำรูปแบบ Session Lifecycle (มาตรฐาน)

### 🔹 Session ที่ 1: ช่วง Planning & Spec (สัมภาษณ์ $\rightarrow$ สเปก $\rightarrow$ แตกตั๋ว)
*ใช้ Session เดียวกันได้เพราะเป็นการต่อยอดข้อมูลความคิด:*
1. รัน `/grill-me` (AI ซักถามคุณจนตกผลึก)
2. รัน `/to-spec` (AI สรุปเป็นไฟล์ `spec/...`)
3. รัน `/to-tickets` (AI แตกเป็นตั๋ว tracer tickets)
4. คุณสร้าง/อัปเดต `ARCHITECTURE.md` + `AGENTS.md` $\rightarrow$ **Commit เข้า baseline ด้วย `jj`**
5. **🎯 จบ Session 1 (ขึ้นแชทใหม่)**

---

### 🔹 Session ที่ 2 เป็นต้นไป: ช่วง Implementation (1 Ticket หรือ 1 Feature = 1 Session)
*เปิด Session ใหม่ทุกครั้งที่เริ่ม Ticket/Module ใหม่:*
1. เปิดแชทใหม่ (Context สะอาด 100%)
2. Prompt สั้นๆ: `"ทำ ticket 01 ตาม AGENTS.md ด้วย tdd"`
3. AI โค้ดเสร็จ $\rightarrow$ รัน `/code-review`
4. รันเทส/ความปลอดภัย $\rightarrow$ **Commit ด้วย `jj describe -m "feat: ..."`**
5. **🎯 จบ Session $\rightarrow$ เปิดแชทใหม่ทำ Ticket ถัดไป**

---

### 🔹 Session พิเศษ: เมื่อเจองานด่วน / บั๊ก / แก้ Legacy
*เปิด Session แยกเฉพาะกิจ:*
1. เปิดแชทใหม่
2. เรียก `/diagnosing-bugs` วิเคราะห์หาสาเหตุ
3. รัน `/tdd` เขียน failing test ดักบั๊ก $\rightarrow$ แก้โค้ดจนผ่าน
4. รัน `/code-review` ตรวจสอบว่าไม่กระทบจุดอื่น $\rightarrow$ **Commit Fix**

---

### 🔹 ถ้าแชทยาวเกินไปแล้วยังไม่จบ?
* เรียกใช้ `/handoff`
* AI จะสร้างเอกสารสรุปสถานะล่าสุด (`handoff.md`)
* ให้คุณ copy สรุปนั้นไปเปิดใน Session ใหม่แล้วสั่งทำต่อได้ทันทีโดยไม่เสียบริบทเดิม

---

## วิธีติดตั้ง / ถอนการติดตั้ง Skills

### ติดตั้งเพิ่ม
```bash
npx -y skills add <owner/repo>@<skill-name> -g -y
```

### ถอนการติดตั้ง (ลบโฟลเดอร์)
```bash
rm -rf ~/.agents/skills/<skill-name>
```
