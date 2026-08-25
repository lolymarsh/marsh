# docs/ — เอกสาร AI Workflow (เรียงตาม priority ที่ควรอ่าน)

> โฟลเดอร์รวมเอกสารคู่มือการใช้ AI ของ marsh
> **อ่านตามลำดับเลขหน้าชื่อไฟล์** — ยิ่งเลขน้อย ยิ่งต้องอ่านก่อน

## 📖 ลำดับการอ่าน (Priority Order)

| ลำดับ | ไฟล์ | เนื้อหา | อ่านเมื่อไหร่ |
|---|---|---|---|
| **01** | [`01_AI_WORKFLOW.md`](01_AI_WORKFLOW.md) | Flow หลัก (Plan → Spec → Tickets → Code) + **Architecture Decision + Prompt สำเร็จรูป 5 สถานการณ์** | ⭐ **ก่อนเริ่มงานทุกครั้ง** — บังคับอ่าน (One-Stop Guide) |
| **02** | [`02_ai-verify-guide.md`](02_ai-verify-guide.md) | วิธีแยกผู้เขียน/ผู้ตรวจ — ห้ามให้ AI ตรวจงานตัวเอง + Checklist ความปลอดภัย | หลัง AI ทำเสร็จ — ก่อน commit |
| **03** | [`03_SESSION_LIFECYCLE.md`](03_SESSION_LIFECYCLE.md) | รายละเอียดเชิงลึกการแบ่ง Session AI: เมื่อไหร่คุยต่อแชทเดิม / เมื่อไหร่ขึ้นแชทใหม่ (1 Ticket = 1 Session) | วางแผนรอบการทำงาน (Context Management) |
| **04** | [`04_SOLO_DEV_INTERNAL_TOOLS_WORKFLOW.md`](04_SOLO_DEV_INTERNAL_TOOLS_WORKFLOW.md) | End-to-End Workflow สำหรับ Solo Fullstack Dev (Internal Tools, เอกสารส่งมอบ, รายงานผู้บริหาร) | เมื่อรับโจทย์โปรเจกต์ภายในบริษัท |

## 🔁 Flow การใช้เอกสาร (สรุป)

```
1. เปิด 01_AI_WORKFLOW.md   → ดูกรณีที่ตรงกับงาน (1-5) แล้วก๊อปปี้ Prompt ไปใช้
2. ดู 03_SESSION_LIFECYCLE.md → เช็ควิธีตัดรอบ Session (Planning แชทเดียว, Implement แยกทีละตั๋ว)
3. หลัง AI ทำเสร็จ          → อ่าน 02_ai-verify-guide.md → ตรวจแบบแยกผู้ตรวจ + checklist
```

## 📚 เอกสารอื่นในโปรเจกต์ (อ้างอิงเสริม)

| ตำแหน่ง | เนื้อหา |
|---|---|
| `../AGENTS.md` | ภาพรวมโปรเจกต์ marsh (อ่านก่อนทุกอย่าง) |
| `../rules/` | Coding rules แยกตาม stack (go/typescript/python/devops) — Single Source of Truth สำหรับโค้ด |
| `../skills/` | รายการ Core Global skills (10 ตัว) |
| `../intercept/` | เอกสาร RTK — กลไก intercept/บีบอัด output (ดู `intercept/README.md`) |

---

> กฎการตั้งชื่อ: ไฟล์ใน docs/ ขึ้นต้นด้วย `NN_` เสมอ = ลำดับ priority ที่ควรอ่าน
