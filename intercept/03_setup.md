# 03 — การติดตั้งและตั้งค่า RTK

> วิธีติดตั้ง RTK + ตั้งค่า intercept สำหรับ AI tool แต่ละตัว
> อ่านก่อน: `01_mechanism.md`

## 1. ติดตั้งตัว binary

```bash
# Homebrew (macOS — แนะนำ)
brew install rtk

# หรือ quick install (Linux/macOS)
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
```

ตรวจสอบ:

```bash
rtk --version     # ควรขึ้น rtk 0.45.x หรือใหม่กว่า
rtk gain          # dashboard สถิติประหยัด tokens
```

> ⚠️ **Name collision**: มีอีกโปรเจกต์ชื่อ "rtk" (Rust Type Kit) บน crates.io —
> ถ้า `rtk gain` ไม่ทำงาน แสดงว่าได้ตัวผิด ให้ใช้ `cargo install --git` หรือ brew แทน

## 2. ตั้งค่า hook ตาม AI tool

```bash
# Claude Code / Copilot (default)
rtk init -g

# OpenCode (plugin TS — tool.execute.before)
rtk init -g --opencode

# Cursor
rtk init -g --agent cursor

# Gemini CLI
rtk init -g --gemini

# Codex (AGENTS.md + RTK.md)
rtk init -g --codex

# Google Antigravity (rules ใน ~/.agents/rules/)
rtk init --agent antigravity

# ตรวจสอบว่า install อะไรไว้แล้วบ้าง
rtk init --show
```

สิ่งที่ `rtk init -g` สร้าง (กรณี Claude Code):
- Hook: `rtk hook claude` (native binary command)
- `~/.claude/RTK.md` — เอกสารให้ agent อ่าน
- `~/.claude/CLAUDE.md` — เพิ่มบรรทัด `@RTK.md` reference
- `settings.json` — ลงทะเบียน hook

## 3. ตั้งค่า Freebuff / agent ที่ไม่มี hook (Instruction mode)

สร้าง rules ไฟล์ที่ agent อ่านแล้วต้องทำตาม:

```bash
# Global rules — agent ทุกตัวบนเครื่อง
# ~/.agents/AGENTS.md  (มีแล้วบนเครื่องนี้ — สร้างโดย Freebuff 2026-08-11)

# Rules เฉพาะ Freebuff (format เดียวกับ antigravity)
# ~/.agents/rules/freebuff-rtk-rules.md  (มีแล้วบนเครื่องนี้)
```

## 4. Config เพิ่มเติม

ไฟล์ config: `~/.config/rtk/config.toml`

```toml
[hooks]
exclude_commands = ["curl", "playwright"]   # ข้าม rewrite สำหรับคำสั่งนี้

[tee]
enabled = true                              # บันทึก raw output เมื่อคำสั่ง fail
mode = "failures"                           # "failures" | "always" | "never"
```

Telemetry (ค่าเริ่มต้น **ปิด** — ต้อง opt-in):

```bash
rtk telemetry status     # ดูสถานะ
rtk telemetry enable     # ยินยอมให้เก็บ (anonymous aggregate)
rtk telemetry disable    # ถอนความยินยอม
```

## 5. ตรวจสอบ + ใช้งานจริง

```bash
rtk gain              # สถิติรวม: commands, tokens saved, efficiency %
rtk gain --graph      # กราฟ 30 วัน
rtk discover          # หาโอกาสประหยัดที่ยังไม่ได้ใช้
rtk session           # การ adoption ล่าสุด
```

ตัวอย่างผลจริงบนเครื่องนี้ (2026-08-11):
```
Total commands:    6374
Input tokens:      31.3M
Tokens saved:      23.7M (75.6%)
Efficiency meter:  ██████████████████░░░░░░ 75.6%
```

## 6. ถอนการติดตั้ง

```bash
rtk init -g --uninstall     # เอา hook + RTK.md + settings.json ออก
brew uninstall rtk          # หรือ cargo uninstall rtk
```
