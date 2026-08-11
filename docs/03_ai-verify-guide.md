# คู่มือ: ห้ามให้ AI ตรวจงานของตัวเอง (Independent Verification)

> ปัญหา: ถ้าปล่อยให้ AI ที่เขียนโค้ด เป็นคนตรวจโค้ดของตัวเอง → AI มัก "ยืนยันว่างานถูกต้อง"
> ทั้งที่จริงมีบั๊ก (confirmation bias — มองไม่เห็นจุดบอดของตัวเอง เพราะมันเห็น context เดียวกันกับตอนเขียน)
>
> วิธีแก้: **แยกคนเขียน กับ คนตรวจ** ให้เป็นคนละ "บทบาท" (คนละ agent / คนละ session / หรือคนละ tool)
>
> สรุปจาก best practice ของ Claude Code / OpenCode community (2025–2026)

---

## 1. ทำไม "AI ตรวจงานตัวเอง" ใช้ไม่ได้

| ปัญหา | คำอธิบาย |
|---|---|
| **Confirmation bias** | AI รู้อยู่แล้วว่ามัน "ตั้งใจ" จะเขียนอะไร → มองข้ามจุดที่มันคิดผิดตั้งแต่แรก |
| **Context เดียวกัน** | ผู้เขียนกับผู้ตรวจเห็นไฟล์/ประวัติเดียวกัน → จุดบอดซ้ำกัน (blind spot ซ้ำ) |
| **ยืนยันแบบผิวเผิน** | AI มักตอบ "ดูแล้วถูกต้อง" โดยไม่ได้รัน test จริง หรือไม่ได้ไล่ทุก edge case |
| **ไม่มีอำนาจโต้แย้ง** | agent ตัวเดียวกันไม่ค่อย "เถียง" งานตัวเอง — มันอยากจบงาน |

**หลักการ**: คนตรวจต้องเป็นคนละตัวกับคนเขียน — มี optimization goal ต่างกัน
(ผู้เขียน = "ทำให้ครบ/เร็ว" / ผู้ตรวจ = "จับผิดให้ได้")

---

## 2. วิธีแยกผู้เขียนกับผู้ตรวจ (ทำได้ 3 ระดับ)

### ระดับ 1: แยก session / แยก agent ตัว (ง่ายสุด — ใช้ได้ทุกที่)

```
Session A (ผู้เขียน):
  "implement phase 02 ตาม spec/... — เขียน test + โค้ดให้ครบ"

Session B (ผู้ตรวจ) — เปิด session ใหม่ / spawn agent ใหม่:
  "ตรวจงานใน session ก่อนหน้าแบบจับผิด:
   - ไม่เชื่อว่ามันถูก ให้หา bug ให้เจอ
   - เทียบกับ spec ทีละข้อ ว่าทำครบจริงมั้ย
   - รัน test + typecheck + lint เอง
   - ห้ามแก้โค้ด เอาแค่รายงานปัญหา"
```

- ผู้ตรวจ **ห้ามแก้โค้ด** — เอาแค่รายงาน (read-only)
- สลับ model ก็ได้ (เช่น เขียนด้วย Claude → ตรวจด้วย GPT/DeepSeek) — จุดบอดคนละแบบ

### ระดับ 2: ใช้ sub-agent เฉพาะทาง (ใน Freebuff / Claude Code / OpenCode)

หลายเครื่องมือมี sub-agent/command สำหรับ review โดยเฉพาะ:

| เครื่องมือ | วิธี |
|---|---|
| **Freebuff** | สั่ง "spawn code-reviewer-deepseek-flash ตรวจ diff ที่เพิ่งเขียน" — agent แยกตัว + model คนละตัว |
| **Claude Code** | `/review` หรือ spawn subagent "verifier" ที่มี `Read` เท่านั้น (ไม่มี write) |
| **OpenCode** | ตั้ง agent profile แยก: `implementer` (เขียนได้) vs `reviewer` (read-only + มี test tools) |
| **GitHub/GitLab** | PR review: ให้ AI ตัวอื่น (Copilot review / บอท) review ก่อนคน |

**กุญแจสำคัญ**: ผู้ตรวจต้องมี **permission ต่างจากผู้เขียน** — อย่างน้อยห้าม write ไฟล์โค้ด
ให้เขียนได้แค่ report

### ระดับ 3: ให้ "เครื่องจักร" ตรวจ (ไม่ใช่ AI — deterministic)

AI ตรวจแล้วยังไงก็พลาดได้ → ใช้เครื่องมือที่ตรวจได้ 100% แน่นอน:

| ตรวจอะไร | เครื่องมือ | คำสั่ง |
|---|---|---|
| Type/Compile | tsc / go build | `npx tsc --noEmit` / `go build ./...` |
| Lint / Format | eslint / golangci-lint / ruff / prettier | `npx eslint .` / `golangci-lint run` |
| Test | vitest / jest / pytest / go test | `npm test` / `go test ./...` |
| Security (secrets) | gitleaks | `gitleaks detect` |
| Security (code) | semgrep | `semgrep scan --config auto` |
| Vulnerabilities | trivy | `trivy fs .` |
| API contract | specmatic / zod validation | ตามโปรเจกต์ |

**หลัก**: AI (ตัวตรวจ) ทำหน้าที่ "คิดเชิงลึก + มองภาพรวม" แต่ verification ขั้นสุดท้าย
ต้องเป็นเครื่องมือที่ deterministic — รันแล้วผลตายตัว

---

## 3. Flow ปฏิบัติที่แนะนำ (สำหรับงานประจำ)

```
┌─ STEP 1: ผู้เขียน (AI session A) ─────────────────────────┐
│  implement ตาม spec + เขียน test ก่อน (หรือคู่กัน)        │
│  → รัน test ผ่านเบื้องต้น                                  │
└──────────────────────────┬───────────────────────────────┘
                           ▼
┌─ STEP 2: เกราะเครื่องจักร (deterministic) ────────────────┐
│  typecheck + lint + test + gitleaks + semgrep            │
│  ❌ ตก → ส่งกลับผู้เขียนแก้                                │
└──────────────────────────┬───────────────────────────────┘
                           ▼
┌─ STEP 3: ผู้ตรวจ (AI ตัวใหม่ — read-only) ────────────────┐
│  ตรวจตาม spec ทีละข้อ / หา bug / ไล่ edge case            │
│  ❌ เจอ → ส่งกลับผู้เขียนแก้ → วน STEP 2-3 ใหม่            │
└──────────────────────────┬───────────────────────────────┘
                           ▼
┌─ STEP 4: คน (คุณ) — review ระดับแนวคิด ──────────────────┐
│  ตรวจ: spec ตรงมั้ย / architecture ถูกมั้ย / รับผิดชอบ 100%│
│  (ไม่ต้องไล่ทีละบรรทัด — อ่าน diff ย่อ + สรุปจากผู้ตรวจ)    │
└──────────────────────────┬───────────────────────────────┘
                           ▼
                     commit / merge
```

---

## 4. Prompt template ผู้ตรวจ (คัดลอกใช้ได้เลย)

### Template A — ตรวจตาม spec (ใช้กับงาน implement)

```
คุณเป็นผู้ตรวจสอบอิสระ (ไม่ใช่คนเขียนโค้ด) จงทำตัวเป็น "ฝ่ายจับผิด":

1. อ่าน spec/02_CUSTOMERS.md แล้วไล่ checklist ทีละข้อว่า implement ครบจริงมั้ย
2. หา bug ให้เจอ อย่างน้อย 3 จุด: edge case / null / error handling / race
3. อย่าเชื่อว่าโค้ดถูก ให้พยายามหักล้างให้ได้
4. รัน: npm test && npx tsc --noEmit && npx eslint .  แล้วรายงานผลจริง
5. ห้ามแก้ไฟล์โค้ดเด็ดขาด — ส่งรายงานเป็น list:
   [CRITICAL] / [MAJOR] / [MINOR] + ไฟล์:บรรทัด + เหตุผล + วิธีแก้ที่แนะนำ
```

### Template B — ตรวจ diff ที่เพิ่งเขียน (งานแก้บั๊ก)

```
ตรวจสอบ diff ล่าสุดแบบไม่เชื่อใจ:
- git diff ดูว่าอะไรเปลี่ยนบ้าง (ใช้ rtk git diff)
- แต่ละบรรทัดที่แก้: มันแก้ถูกจุดมั้ย? มีผลข้างเคียงกับที่อื่นมั้ย?
- มี code path ที่ test ยังไม่ครอบคลุมมั้ย?
- ห้ามแก้โค้ด — รายงานอย่างเดียว
```

### Template C — ตรวจความปลอดภัย

```
ตรวจ security ของงานนี้:
- ไล่หา: injection, auth bypass, hardcoded secret, path traversal, SSRF
- ตรวจว่า secret ไม่หลุดในโค้ด/commit (รัน gitleaks)
- ตรวจ permission ที่เกินจำเป็นใน docker/k8s config
- รายงาน: ระดับความเสี่ยง + ไฟล์:บรรทัด + วิธีแก้
```

---

## 5. Checklist ท้ายงาน (ใช้คู่กับผู้ตรวจ)

```
[ ] ผู้เขียน ≠ ผู้ตรวจ (คนละ session/agent/model)
[ ] ผู้ตรวจเป็น read-only (ไม่มีสิทธิ์แก้โค้ด)
[ ] typecheck ผ่าน
[ ] lint ผ่าน (0 error)
[ ] test ผ่าน (รันเอง ไม่เชื่อปาก AI)
[ ] gitleaks ไม่พบ secret
[ ] semgrep ไม่พบ High/Critical
[ ] ผู้ตรวจไล่ spec ครบทุกข้อ (acceptance criteria)
[ ] คุณ (คน) อ่าน diff ย่อ + สรุปผู้ตรวจ แล้ว approve เอง
[ ] โค้ดที่แก้มี test ครอบคลุมจุดที่แก้ (ไม่ใช่แค่ "รันผ่าน")
```

---

## 6. ข้อควรจำ

- **AI เป็นเครื่องมือ ไม่ใช่คนรับผิดชอบ** — คุณรับผิดชอบผลลัพธ์ 100% เสมอ
- ผู้ตรวจ AI ≠ เชื่อถือได้ 100% — มันคือ "คนอ่านคนที่ 2" ที่ช่วยจับจุดบอด
- เครื่องมือ deterministic (test/lint/typecheck) = เกราะจริง ผู้ตรวจ AI = เกราะเสริม
- งานเล็กมาก (แก้ typo, เพิ่ม 1 function) ไม่ต้องผ่านขั้นตอนนี้ครบ — ใช้ดุลยพินิจ
- งานที่แตะ auth / database / concurrency / production config → ต้องตรวจเข้มสุดเสมอ

---

> สร้าง: 2026-08-11 (Freebuff)
> อ้างอิง: Spec-Driven Development + Adversarial Agent Pattern (Implementor vs Verifier)
