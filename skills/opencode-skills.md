# opencode skills

## Skills ที่ติดตั้งอยู่ตอนนี้ (15 ตัว)

| # | Skill | ผู้พัฒนา | การใช้งาน |
|---|---|---|---|
| 1 | find-skills | vercel-labs/skills | ค้นหา skill อื่นๆ |
| 2 | frontend-design | anthropics/skills | ออกแบบ UI/Frontend |
| 3 | tdd | mattpocock/skills | Test-Driven Development |
| 4 | code-review | mattpocock/skills | Review โค้ด |
| 5 | handoff | mattpocock/skills | ส่งต่องานระหว่าง agent |
| 6 | diagnosing-bugs | mattpocock/skills | วิเคราะห์บั๊ก |
| 7 | improve-codebase-architecture | mattpocock/skills | ปรับปรุงโครงสร้างโค้ด |
| 8 | skill-creator | anthropics/skills | สร้าง skill เอง |
| 9 | shadcn | shadcn/ui | UI components |
| 10 | supabase-postgres-best-practices | supabase/agent-skills | ฐานข้อมูล PostgreSQL (backend) |
| 11 | vercel-react-best-practices | vercel-labs/agent-skills | React/Next.js best practices (frontend) |
| 12 | web-design-guidelines | vercel-labs/agent-skills | แนวทางการออกแบบเว็บ (frontend) |
| 13 | webapp-testing | anthropics/skills | ทดสอบ web application (testing) |
| 14 | writing-plans | obra/superpowers | เขียนแผน/PRD |
| 15 | **to-spec** | **mattpocock/skills** | **สรุปงานที่คุยกันเป็น spec/PRD (ชื่อเดิม to-prd ถูก rename)** |

Symlink จาก `~/.claude/skills/` → `~/.agents/skills/`

## วิธีติดตั้ง

```bash
npx skills add <owner/repo>@<skill-name> -g -y
```

- `-g` = global (อยู่ที่ `~/.claude/skills/` ซึ่ง opencode อ่านได้)
- `-y` = ข้ามยืนยัน

### ติดตั้ง manual (ไม่มี npx)

สร้างโฟลเดอร์ `SKILL.md` ใน path ใด path หนึ่ง:
- `~/.config/opencode/skills/<name>/SKILL.md`
- `.opencode/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`

## แนะนำ skills ที่น่าสนใจ

### จาก `mattpocock/skills`
| Skill | คำสั่งติดตั้ง |
|---|---|
| tdd | `npx skills add mattpocock/skills@tdd -g -y` |
| code-review | `npx skills add mattpocock/skills@code-review -g -y` |
| handoff | `npx skills add mattpocock/skills@handoff -g -y` |
| diagnosing-bugs | `npx skills add mattpocock/skills@diagnosing-bugs -g -y` |
| improve-codebase-architecture | `npx skills add mattpocock/skills@improve-codebase-architecture -g -y` |
| to-spec (เดิมชื่อ to-prd) | `npx skills add mattpocock/skills@to-spec -g -y` |

### จาก `anthropics/skills`
| Skill | คำสั่งติดตั้ง |
|---|---|
| frontend-design | `npx skills add anthropics/skills@frontend-design -g -y` |
| skill-creator | `npx skills add anthropics/skills@skill-creator -g -y` |
| webapp-testing | `npx skills add anthropics/skills@webapp-testing -g -y` |

### จาก `vercel-labs/agent-skills`
| Skill | คำสั่งติดตั้ง |
|---|---|
| vercel-react-best-practices | `npx skills add vercel-labs/agent-skills@vercel-react-best-practices -g -y` |
| web-design-guidelines | `npx skills add vercel-labs/agent-skills@web-design-guidelines -g -y` |

### จาก `obra/superpowers`
| Skill | คำสั่งติดตั้ง |
|---|---|
| systematic-debugging | `npx skills add obra/superpowers@systematic-debugging -g -y` |
| test-driven-development | `npx skills add obra/superpowers@test-driven-development -g -y` |
| writing-plans | `npx skills add obra/superpowers@writing-plans -g -y` |

### สำหรับวางแผน (Planning)
| Skill | คำสั่งติดตั้ง |
|---|---|
| writing-plans | `npx skills add obra/superpowers@writing-plans -g -y` |

### อื่นๆ
| Skill | คำสั่งติดตั้ง |
|---|---|
| shadcn (shadcn/ui) | `npx skills add shadcn/ui@shadcn -g -y` |
| supabase-postgres-best-practices | `npx skills add supabase/agent-skills@supabase-postgres-best-practices -g -y` |
| sentry-cli | `npx skills add sentry/dev@sentry-cli -g -y` |

## ค้นหา skills เพิ่ม

```bash
npx skills find <keyword>
```

หรือดู leaderboard: https://skills.sh/

## วิธีใช้งาน skills

### แบบอัตโนมัติ (แนะนำ)

opencode agent จะเห็นรายการ skills ที่มี และ load skill ที่เกี่ยวข้องให้อัตโนมัติตาม context ของงาน — ไม่ต้อง prompt พิเศษ

เช่น ถ้าสั่ง:
```
implement customer module with tests
```
agent จะเห็นว่า `tdd`, `code-review`, `supabase-postgres-best-practices` เกี่ยวข้อง → load ให้เอง

### แบบ Manual (บังคับใช้)

แค่ reference ชื่อ skill ใน prompt:
```
ใช้ tdd skill: implement customer module, เขียน test ก่อน code
```

### ตัวอย่างการเรียกใช้ใน workflow ของคุณ

แทน prompt ปกติ:
```
implement phase 02 ตาม spec/02_CUSTOMERS.md
```

เติม skill hint ไป:
```
ใช้ tdd + supabase-postgres-best-practices:
implement phase 02 ตาม spec/02_CUSTOMERS.md
ตาม AGENTS.md rules + ARCHITECTURE.md patterns
```

### ตรวจสอบว่า skill ถูกใช้มั้ย

ถ้าไม่แน่ใจว่ามีการ load skill หรือเปล่า — ถาม agent ตรงๆ:
```
ใช้ find-skills skill หา skill ที่เหมาะกับงาน frontend ให้หน่อย
```

หรือดูใน context ว่า agent พูดถึง skill หรือ mention skill ชื่อนั้นๆ

### สรุป

| วิธี | ทำยังไง |
|---|---|
| อัตโนมัติ | แค่ทำงานปกติ — agent เลือก skill ให้เอง |
| Manual | พิมพ์ `ใช้ <skill-name>:` นำหน้า prompt |
| ค้นหา skill เพิ่ม | `ใช้ find-skills: หา skill ที่เหมาะกับ <งาน>` |

## ตำแหน่งที่ opencode ค้นหา skills

- `~/.config/opencode/skills/<name>/SKILL.md`
- `.opencode/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`
- `.claude/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
