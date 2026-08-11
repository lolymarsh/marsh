# คู่มือการติดตั้งและใช้งาน Security Scanning Tools สำหรับ AI Workflow (agy cli & opencode)

เอกสารสรุปเครื่องมือและขั้นตอนการสแกนความปลอดภัย (Security Scanning) ทั้งฝั่ง **Frontend** และ **Backend** เพื่อช่วยคัดกรองโค้ดที่สร้างจาก AI อย่างรวดเร็วและปลอดภัย ก่อนทำการ Commit หรือ Deploy

---

## 1. เครื่องมือหลัก 3 ตัวที่ต้องมี (Core Security Stack)

| เครื่องมือ | ประเภทการสแกน | จุดเด่น | คำสั่งติดตั้ง (Mac) |
| :--- | :--- | :--- | :--- |
| **Gitleaks** | Secret / API Key Scanning | รันเร็วมาก ตรวจจับ API Key, Token, Password ที่เผลอหลุดในโค้ดและ Git History | `brew install gitleaks` |
| **Semgrep** | SAST (Static Application Security Testing) | รัน Local ได้ทันที สแกนหาช่องโหว่ OWASP Top 10, Injection, Logic หลุด รองรับหลายภาษา | `brew install semgrep` |
| **Trivy** | SCA (Dependency & Container Scanning) | สแกนช่องโหว่ใน Library, Package, Dockerfile, และ IaC Config | `brew install trivy` |

---

## 2. การสแกนแยกตามส่วนประกอบ (Frontend vs Backend)

### 🎨 ฝั่งหน้าบ้าน (Frontend Focus)
* **ความเสี่ยงหลัก**: Secrets หลุดใน Bundle/Env, XSS (Cross-Site Scripting), Third-party Packages ที่มีช่องโหว่
* **เครื่องมือสแกน**:
  * `gitleaks detect` — ดักจับ Key หลุดก่อน Build
  * `semgrep scan --config auto` — สแกนหาจุดเสี่ยง XSS / Dangerous DOM Manipulations
  * `npm audit` / `pnpm audit` / `yarn audit` — ตรวจสอบแพ็กเกจหน้าบ้าน

### ⚙️ ฝั่งหลังบ้าน (Backend Focus)
* **ความเสี่ยงหลัก**: SQL/NoSQL Injection, SSRF, RCE, Authentication/Authorization Bypass, Data Leakage
* **เครื่องมือสแกน**:
  * `semgrep scan --config p/owasp-top-10` — สแกนหาแนวทางการโจมตีฝั่ง Backend
  * **Language-Specific SAST Tools** (เลือกใช้ตามภาษาหลักของ Backend):
    * **Python (FastAPI / Django / Flask)**: `bandit -r ./app`
    * **Go (Golang)**: `gosec ./...`
    * **Node.js (Express / NestJS)**: `eslint-plugin-security`
    * **C / C++ / MQL5**: `cppcheck` หรือ `flawfinder`
  * **Dynamic / API Scan (DAST)**: ใช้ **OWASP ZAP** หรือ **Nuclei** ยิงทดสอบ Endpoint ขณะ Server รันอยู่

---

## 3. การตั้งค่าระบบอัตโนมัติผ่าน Git Pre-commit Hook

เพื่อไม่ให้ลืมสแกนในเวลาที่ต้องส่งงานเร่งด่วน แนะนำให้ใช้ `pre-commit` เพื่อให้สแกนอัตโนมัติทุกครั้งก่อน `git commit`:

1. **ติดตั้ง pre-commit**:
   ```bash
   brew install pre-commit
   ```

2. **สร้างไฟล์ `.pre-commit-config.yaml` ใน Root ของโปรเจกต์**:
   ```yaml
   repos:
     - repo: https://github.com/gitleaks/gitleaks
       rev: v8.18.2
       hooks:
         - id: gitleaks

     - repo: https://github.com/returntocorp/semgrep
       rev: 'v1.65.0'
       hooks:
         - id: semgrep
           args: ['--config', 'auto']
   ```

3. **เปิดใช้งานในโปรเจกต์**:
   ```bash
   pre-commit install
   ```

---

## 4. Cheat Sheet คำสั่งสแกนด่วน (Quick Scan Commands)

```bash
# 1. สแกนหา Secret / API Key ในโปรเจกต์
gitleaks detect --verbose

# 2. สแกนหาช่องโหว่โค้ดในโปรเจกต์ด้วย Semgrep (Auto Config)
semgrep scan --config auto

# 3. สแกนหาช่องโหว่โค้ดเน้นหนัก OWASP Top 10
semgrep scan --config p/owasp-top-10

# 4. สแกนโฟลเดอร์โครงการด้วย Trivy (สแกนทั้ง Dependencies และ Config)
trivy fs .
```

---

## 5. สรุปคำแนะนำสำหรับการทำงานร่วมกับ agy cli / opencode

1. **เปิดใช้ Gitleaks + Semgrep ใน Pre-commit Hook** เป็นบรรทัดฐานขั้นต่ำสุดของทุกโปรเจกต์
2. **ฝั่ง Backend** ให้เพิ่มการสแกนด้วยเครื่องมือเฉพาะภาษา (เช่น Bandit / Gosec) หรือรัน SonarLint ใน IDE
3. **เมื่อใช้งาน agy cli / opencode** สามารถสั่งคำสั่งสแกนผ่าน Terminal เพื่อให้ AI ช่วยสรุปจุดแก้ไขที่เป็นความเสี่ยงระดับ `High` หรือ `Critical` ได้ทันที
