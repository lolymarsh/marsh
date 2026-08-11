# intercept/ — กลไกดักจับคำสั่ง shell ก่อนเข้าสู่ AI Context

> เอกสารอธิบายกลไก **Intercept** — การดักจับ/บีบอัด Output ของคำสั่ง Terminal
> ก่อนที่มันจะถูกส่งเข้า Context Window ของ AI Agent
> ตัวหลักที่ใช้คือ **RTK (rtk-ai/rtk)** — Rust Token Killer

## ทำไมต้องมี intercept

AI Agent (Freebuff, Claude Code, OpenCode, Cursor, Gemini CLI) ทำงานโดยรันคำสั่ง shell
เช่น `git status`, `grep`, `npm test` แล้ว **ผลลัพธ์ดิบ (raw output) ทุกบรรทัดจะถูกส่งเข้า
Context Window** ทุกรอบที่คุยกับ model — ถ้า output เยอะ (เช่น `cargo test` 200+ บรรทัด)
ก็จะเผา token เปล่าๆ ซ้ำแล้วซ้ำเล่า

**Intercept = ตัวกลางที่กรอง/บีบอัด output เหล่านั้นก่อนเข้าสู่ context**
→ ประหยัด token ได้ 60–90%

```
Agent ──รันคำสั่ง──▶ Shell ──output ดิบ──▶ [ ★ INTERCEPT ตรงนี้ ★ ] ──output บีบ──▶ Context ──▶ Model
```

## สารบัญ

| ไฟล์ | เนื้อหา |
|---|---|
| `01_mechanism.md` | กลไกการทำงาน: hook คืออะไร, อยู่ตรงไหนของ pipeline, แบบต่างๆ |
| `02_flow.md` | Flow เต็มตั้งแต่ input → output แบบ text (มี/ไม่มี RTK เปรียบเทียบกัน) |
| `03_setup.md` | วิธีติดตั้ง + ตั้งค่า RTK สำหรับแต่ละ AI tool |
| `04_rules.md` | Rules บังคับใช้ rtk + ตาราง command mapping (ต้องใช้กับทุกโปรเจกต์) |

## หลักการสำคัญ (อ่านก่อน)

1. **Intercept อยู่ที่ชั้น tool call ไม่ใช่ชั้น request ไป model** — ไม่แตะเส้นทาง `user → model`
2. **ทำงานเฉพาะฝั่ง output ของคำสั่ง** — ไม่ยุ่งกับ prompt, system prompt, หรือ response ที่ model ตอบ
3. **ประหยัดเพราะ**: output ที่บีบแล้วเข้าไปอยู่ใน context ของรอบถัดไป — model อ่านน้อยลง
4. **มี 2 โหมด**:
   - **Hook mode** (Claude Code / OpenCode / Cursor / Gemini) — ตัว hook แทรกกลาง bash tool call อัตโนมัติ
   - **Instruction mode** (Freebuff, agent อื่น) — agent อ่าน rules แล้วเรียก `rtk <cmd>` เอง

---

*อ้างอิง: `AI_WORKFLOW.md` → หัวข้อ "โปรแกรมแนะนำสำหรับเพิ่มประสิทธิภาพ (Recommended Tools)"*
*สร้าง: 2026-08-11*
