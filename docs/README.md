# docs/ — เอกสาร AI Workflow (เรียงตาม priority ที่ควรอ่าน)

> โฟลเดอร์รวมเอกสารคู่มือการใช้ AI ของ marsh
> **อ่านตามลำดับเลขหน้าชื่อไฟล์** — ยิ่งเลขน้อย ยิ่งต้องอ่านก่อน

## 📖 ลำดับการอ่าน (Priority Order)

| ลำดับ | ไฟล์ | เนื้อหา | อ่านเมื่อไหร่ |
|---|---|---|---|
| **01** | [`01_AI_WORKFLOW.md`](01_AI_WORKFLOW.md) | Flow หลัก: idea → plan → spec → ARCHITECTURE → AGENTS.md → implement + **วิธีเริ่มงาน 4 สถานการณ์** (ใหม่/กลาง/ค้าง/เก่า) | ⭐ **ก่อนเริ่มงานทุกครั้ง** — บังคับอ่าน |
| **02** | [`02_prompt-templates.md`](02_prompt-templates.md) | 11 prompt templates ที่แมปกับ skill (implement Go/React/Next/Python, แก้บั๊ก, review, handoff ฯลฯ) | เมื่อจะสั่งงาน AI — เลือก template ตามงาน |
| **03** | [`03_ai-verify-guide.md`](03_ai-verify-guide.md) | วิธีแยกผู้เขียน/ผู้ตรวจ — ห้ามให้ AI ตรวจงานตัวเอง + checklist | หลัง AI ทำเสร็จ — ก่อน commit |

## 🔁 Flow การใช้เอกสาร (สรุป)

```
1. อ่าน 01_AI_WORKFLOW.md   → รู้วิธีเริ่มงานตามสถานการณ์ (ใหม่/กลาง/ค้าง/เก่า)
2. เลือก 02_prompt-templates.md → เอา template ไปใช้กับงาน (พร้อม skill)
3. หลัง AI ทำเสร็จ → อ่าน 03_ai-verify-guide.md → ตรวจแบบแยกผู้ตรวจ + checklist
```

## 📚 เอกสารอื่นในโปรเจกต์ (อ้างอิงเสริม)

| ตำแหน่ง | เนื้อหา |
|---|---|
| `../AGENTS.md` | ภาพรวมโปรเจกต์ marsh (อ่านก่อนทุกอย่างเหมือนเดิม) |
| `../rules/` | Coding rules แยกตาม stack (go/typescript/python/devops) |
| `../skills/` | Project skills (implement-*) + รายการ global skills |
| `../intercept/` | เอกสาร RTK — กลไก intercept/bีบอัด output (ดู `intercept/README.md`) |

---

> กฎการตั้งชื่อ: ไฟล์ใน docs/ ขึ้นต้นด้วย `NN_` เสมอ = ลำดับ priority ที่ควรอ่าน
> เพิ่มไฟล์ใหม่ → ต้องระบุ priority ให้ชัดเจน และอัปเดตตารางนี้
