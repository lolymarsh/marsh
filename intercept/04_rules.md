# 04 — Rules: บังคับใช้ rtk กับทุกคำสั่ง shell

> กฎที่ agent (และมนุษย์) ต้องปฏิบัติเมื่อรันคำสั่ง terminal
> ใช้กับทุกโปรเจกต์บนเครื่องนี้ — บังคับใช้ผ่าน global rules (`~/.agents/`)
> อ่านก่อน: `01_mechanism.md`, `02_flow.md`

## ข้อบังคับหลัก (R1–R4)

- **R1 — ห้ามรันคำสั่ง shell แบบ raw** สำหรับคำสั่งที่ RTK รองรับ ต้อง prefix `rtk` เสมอ
- **R2 — อ่านไฟล์ผ่าน shell** ให้ใช้ `rtk read <file>` แทน `cat` ทั้งไฟล์
- **R3 — ค้นหาโค้ดผ่าน shell** ให้ใช้ `rtk grep` / `rtk find` แทน `grep` / `find` ตรงๆ
- **R4 — คำสั่งที่ RTK ไม่รองรับ** ใช้ `rtk proxy <cmd>` (passthrough) หรือ `rtk err <cmd>` / `rtk summary <cmd>` ช่วยกรอง output ยาว

## Command Mapping (ตารางหลัก)

### ไฟล์ & การค้นหา

| แทนที่จะใช้ | ให้ใช้ | ผลลัพธ์ |
|---|---|---|
| `ls` / `tree` | `rtk ls <dir>` | tree compact พร้อมจำนวนไฟล์ |
| `cat file` | `rtk read file` | อ่านแบบมีโครงสร้าง ประหยัด tokens |
| `cat file` (อยากได้แค่ signature) | `rtk read file -l aggressive` | เฉพาะโครงสร้าง ไม่เอา body |
| `grep` / `rg` | `rtk grep "pattern" <dir>` | จัดกลุ่มผลลัพธ์ตามไฟล์ |
| `find` | `rtk find "*.rs" .` | ผลลัพธ์ compact |
| `diff a b` | `rtk diff a b` | condensed diff |

### Git

| แทนที่จะใช้ | ให้ใช้ | ผลลัพธ์ |
|---|---|---|
| `git status` | `rtk git status` | 1-2 บรรทัด |
| `git log -n 10` | `rtk git log -n 10` | hash + author + subject |
| `git diff` | `rtk git diff` | ลด context, ตัด header |
| `git add` | `rtk git add` | → `ok` |
| `git commit -m "msg"` | `rtk git commit -m "msg"` | → `ok abc1234` |
| `git push` | `rtk git push` | → `ok main` |
| `git pull` | `rtk git pull` | → `ok 3 files +10 -2` |

### Test / Build / Lint

| แทนที่จะใช้ | ให้ใช้ | ผลลัพธ์ |
|---|---|---|
| `pytest` | `rtk pytest` | failures only (-90%) |
| `go test` | `rtk go test` | NDJSON, failures only (-90%) |
| `cargo test` | `rtk cargo test` | failures only (-90%) |
| `jest` | `rtk jest` | failures only |
| `vitest` | `rtk vitest` | failures only |
| `npm test` / คำสั่ง test ใดๆ | `rtk test <cmd>` | generic wrapper, failures only |
| คำสั่งไหนอยากได้แค่ error | `rtk err <cmd>` | กรองเฉพาะ error |
| `npx tsc` | `rtk tsc` | errors จัดกลุ่มตามไฟล์ |
| `eslint` | `rtk lint` | จัดกลุ่มตาม rule/file |
| `cargo build` | `rtk cargo build` | (-80%) |
| `cargo clippy` | `rtk clippy` | (-80%) |
| `ruff check` | `rtk ruff check` | JSON, (-80%) |

### Docker / อื่นๆ

| แทนที่จะใช้ | ให้ใช้ | ผลลัพธ์ |
|---|---|---|
| `docker ps` | `rtk docker ps` | เฉพาะ field สำคัญ |
| `docker logs <c>` | `rtk docker logs <c>` | deduplicated |
| `gh pr list` | `rtk gh pr list` | compact |
| `gh issue list` | `rtk gh issue list` | compact |
| `gh run list` | `rtk gh run list` | workflow status |
| `kubectl pods` | `rtk kubectl pods` | compact |
| `curl <url>` | `rtk curl <url>` | truncate + save full |
| `log` ไฟล์ยาว | `rtk log app.log` | deduplicated |
| `pip list` | `rtk pip list` | compact |

## คำสั่ง meta ที่ใช้บ่อย

```bash
rtk gain              # สถิติประหยัด
rtk gain --graph      # กราฟ 30 วัน
rtk discover          # หาโอกาสประหยัดที่พลาด
rtk session           # adoption ล่าสุด
rtk proxy <cmd>       # รันแบบ raw ไม่กรอง (debug)
rtk summary <cmd>     # สรุปคำสั่งยาว
```

## ตัวอย่างก่อน/หลัง (เห็นภาพ)

```
# git status — 45 บรรทัด → 1-2 บรรทัด
❌ git status                          ✅ rtk git status
On branch main                          * main...origin/main
Your branch is up to date...            ?? msg.md
Changes not staged...
    modified:   ea/SimpleMA/SimpleMA.mq5
...

# cargo test — 200+ บรรทัด → ~20 บรรทัด
❌ cargo test                          ✅ rtk cargo test
running 15 tests                        FAILED: 2/15 tests
test utils::test_parse ... ok           test_edge_case: assertion failed
...
```

## หมายเหตุ

- **Hook mode** (Claude Code/OpenCode/Cursor) rewrite อัตโนมัติ — ไม่ต้องจำตาราง
- **Instruction mode** (Freebuff) — agent อ่านกฎนี้แล้วเรียก `rtk` prefix เอง
- **Built-in tools** (Read/Grep/Glob ของ Claude Code/Freebuff) ไม่ผ่าน shell hook —
  ถ้าอยากได้ output แบบบีบให้ใช้ shell command (`rtk read`, `rtk grep`) แทน
- โปรเจกต์นี้ใช้ `jj` (git ถูก wrap) — `rtk git status` ยังอ่านผล jj ได้ถูกต้อง
