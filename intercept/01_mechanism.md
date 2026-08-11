# 01 — กลไกการทำงานของ Intercept (Hook)

> อธิบายว่า "hook" คืออะไร, อยู่ตรงไหนของ pipeline, และมีกี่แบบ
> อ่านก่อน: `README.md`

## 1. Hook คืออะไร

**Hook** = ตัวดักจับ (interceptor) ที่ AI tool เปิดช่องให้โค้ดภายนอกแทรกเข้าไป
**ก่อน/หลัง** tool call แต่ละครั้ง เช่น ก่อนรัน bash command

RTK ใช้ช่องทางนี้ rewrite คำสั่ง:
```
git status            ──hook แปลง──▶   rtk git status
```
โดย agent ไม่รู้ตัว — พิมพ์ `git status` ตามปกติ แต่ที่รันจริงคือ `rtk git status`
และ output ที่ได้กลับมาคือเวอร์ชันบีบอัดแล้ว

## 2. ตำแหน่งของ intercept ใน pipeline

```
User input
    │
    ▼
Agent (LLM ตัดสินใจรันคำสั่ง)
    │  bash tool call
    ▼
HOOK (intercept) ──rewrite──▶ rtk <cmd>
    │                              │
    │                     รันคำสั่งจริง (git, grep, test...)
    │                              │
    │                     output ดิบ ──▶ กรอง/บีบ/จัดกลุ่ม
    ▼                              ▼
Agent context ◀─────── output บีบแล้ว (compact)
    │
    ▼
Request ไป Model (API ตรงๆ — RTK ไม่แตะเส้นนี้)
    │
    ▼
Response → User
```

**จุดที่ RTK แทรกได้ = เฉพาะขั้น "bash tool call"** ไม่ใช่ขั้น request ไป model

## 3. ประเภท hook ที่ RTK ใช้ (แยกตาม tool)

| AI Tool | กลไก | วิธีติดตั้ง |
|---|---|---|
| **Claude Code** | `PreToolUse` hook (native binary) | `rtk init -g` |
| **OpenCode** | Plugin TS (`tool.execute.before`) | `rtk init -g --opencode` |
| **Cursor** | `preToolUse` hook (hooks.json) | `rtk init -g --agent cursor` |
| **Gemini CLI** | `BeforeTool` hook | `rtk init -g --gemini` |
| **Codex** | AGENTS.md + RTK.md (instruction) | `rtk init -g --codex` |
| **Windsurf** | `.windsurfrules` (project-scoped) | `rtk init -g --agent windsurf` |
| **Cline / Roo** | `.clinerules` (project-scoped) | `rtk init --agent cline` |
| **Google Antigravity** | `.agents/rules/antigravity-rtk-rules.md` | `rtk init --agent antigravity` |
| **Freebuff / อื่นๆ** | ❌ ไม่มี hook — ใช้ **instruction mode** | อ่าน `04_rules.md` |

## 4. โหมดที่ไม่มี hook (สำคัญสำหรับ Freebuff)

Freebuff ยังไม่อยู่ในลิสต์ 16 tools ที่ RTK รองรับ hook → **ไม่มีตัวแทรกกลางอัตโนมัติ**

ทางเลือก = **Instruction mode**: วาง rules ไฟล์ที่ agent อ่านแล้วต้องทำตาม

| ตำแหน่ง | บทบาท |
|---|---|
| `~/.agents/AGENTS.md` | Global rules — ทุก agent บนเครื่องต้องใช้ `rtk` prefix |
| `~/.agents/rules/freebuff-rtk-rules.md` | Rules เฉพาะ Freebuff (format เดียวกับ antigravity) |

ข้อจำกัด: เป็นการ "บังคับด้วยคำสั่ง" ไม่ใช่ "บังคับด้วยกลไก" — ขึ้นกับว่า agent
อ่าน rules แล้วปฏิบัติตามครบถ้วนแค่ไหน

## 5. ตรงไหนที่ RTK ไม่ยุ่ง

- ❌ Request ไป API ของ model (เส้นทางตรง agent → model provider)
- ❌ System prompt / rules ที่ส่งเข้ารอบแรก
- ❌ Output ที่ model ตอบกลับมา
- ❌ ไฟล์ในเครื่อง — RTK ไม่แตะโค้ด อ่านอย่างเดียวผ่านคำสั่ง

## 6. กลยุทธ์ 4 อย่างที่ RTK ใช้บีบ output

| กลยุทธ์ | ตัวอย่าง |
|---|---|
| **Smart Filtering** — ตัดขยะ | ตัด comment, whitespace, boilerplate |
| **Grouping** — รวมกลุ่มคล้ายกัน | จัดกลุ่มไฟล์ตาม directory, error ตามประเภท |
| **Truncation** — ตัดส่วนเกิน | เก็บ context ที่สำคัญ ตัดความซ้ำซ้อน |
| **Deduplication** — รวมบรรทัดซ้ำ | `INFO log... x50` แทน 50 บรรทัด |
