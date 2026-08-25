# คู่มือการทำงานแบบ Solo Fullstack Dev สำหรับ Internal Tools & Business Workflow
> ย้ายงานใหม่ / รับโจทย์โปรเจกต์ภายใน (Internal Tools) ให้ทำงานแบบครบวงจร (End-to-End Solo Developer) ด้วยพลังของ AI

---

## 🎯 คำตอบสำหรับคำถามหลักของคุณ: "โจทย์มาลอยๆ เราคิด Business Logic เองแล้วใช้ AI ช่วยยังไง?"

> **คำตอบคือ: ถูกต้อง 100%!** 
> ในการพัฒนาโปรเจกต์ภายใน (Internal Product) หัวหน้าหรือฝ่ายบริหารมักจะให้มาแค่ **"ชื่อโปรเจกต์ + ปัญหาคร่าวๆ (Concept)"** เช่น *"อยากได้ระบบเบิกของออฟฟิศ/สต๊อกที่ไม่งง"* หรือ *"อยากให้ฝ่ายบัญชีดึงรายงานเร็วขึ้น"*
>
> ฐานะ Solo Fullstack Developer คุณคือ **Product Owner + System Architect + Developer ในคนเดียว**:
> 1. คุณจะใช้ `/grill-me` ให้ AI สวมบทบาทเป็นที่ปรึกษา ช่วยตั้งคำถามเพื่อ **แกะความต้องการ (Requirements), วิเคราะห์ Business Flow เดิม vs ใหม่ และดักจับ Edge Cases**
> 2. คุณใส่ Business Context / Logic และความเป็นไปได้ในทางปฏิบัติของบริษัทตอบกลับไป
> 3. AI สรุปเป็น **Spec & Acceptance Criteria**
> 4. คุณเริ่มพัฒนาด้วย AI (Test-First / Phase-by-phase) และจัดทำ **เอกสารส่งมอบ (Handover / User Manual)** ให้พนักงานและหัวหน้า

---

## 🗺️ End-to-End Lifecycle ของ Solo Dev (6 ขั้นตอน ตั้งแต่วันแรกจนส่งมอบ)

```
[ 1. รับ Concept ลอยๆ ]
         │
         ▼
[ 2. AI Brainstorm & Grill ] ──► ใช้ /grill-me ถามเจาะลึก Business Flow, Role, Data, Edge Cases
         │
         ▼
[ 3. Design & Baseline ]    ──► ทำ Spec, DB Schema, RBAC, Wireframe/Prototype
         │
         ▼
[ 4. Build & TDD Loop ]     ──► ซอยตั๋วงานย่อย (/to-tickets) → Implement ทีละ Ticket (/tdd)
         │
         ▼
[ 5. Delivery & Handover ]  ──► จัดทำ Quick User Guide, Deployment Guide, Demo Walkthrough
         │
         ▼
[ 6. Track & Report ]       ──► รายงานความคืบหน้า + สรุปผลลัพธ์ (Time Saved / Impact) ให้หัวหน้า
```

---

## 📌 ขั้นตอนที่ 1: การเปลี่ยนไอเดียลอยๆ ให้กลายเป็น Business Logic ด้วย `/grill-me`

เมื่อหัวหน้าสั่งงานมาสั้นๆ เช่น *"อยากได้ระบบจัดการสินทรัพย์และเบิกอุปกรณ์ไอทีภายใน"*

### 📋 Prompt เริ่มต้นสำหรับ Grill-me:
```text
ใช้ grill-me:
หัวหน้าให้โจทย์ทำโปรเจกต์ภายใน: "ระบบจัดการและเบิกอุปกรณ์ IT ภายในบริษัท"
เป้าหมาย: ช่วยให้พนักงานเบิกของง่ายขึ้น และแอดมิน IT เช็กสต๊อกไม่ตกหล่น

ช่วยซักถาม/สัมภาษณ์ผมแบบเจาะลึกทีละข้อ โดยครอบคลุม:
1. User Roles & Permissions (ใครมีสิทธิ์ทำอะไรบ้าง: พนักงาน, หัวหน้าอนุมัติ, IT Admin)
2. Workflow ปัจจุบัน (Pain point เดิม) vs Workflow ใหม่ที่ควรจะเป็น
3. สถานะและ Life cycle ของข้อมูล (Draft -> Pending -> Approved -> Delivered -> Returned)
4. ข้อมูลและ Fields ที่จำเป็นต้องเก็บ
5. Edge Cases (ของหมดสต๊อก, ยกเลิกคำขอ, คืนของชำรุด)
6. ความปลอดภัย (ใครเห็นข้อมูลใครได้บ้าง)
```

---

## 📌 ขั้นตอนที่ 2: สังเคราะห์เป็น Spec & Baseline Architecture

หลังตอบคำถามจนตกผลึก ให้ AI รัน `/to-spec` เพื่อสร้างเอกสารโครงสร้างระบบ

### สิ่งที่ระบบ Internal Tools ควรมีเป็น Baseline เสมอ:
1. **Authentication & RBAC (Role-Based Access Control)**:
   - Login ผ่าน Google Workspace SSO / Microsoft Office 365 หรือ Internal Email
   - Role ชัดเจน (เช่น `EMPLOYEE`, `MANAGER`, `ADMIN`, `SUPER_ADMIN`)
2. **Audit Logging & Activity History**:
   - ระบบภายใน "ต้องรู้ว่าใครแก้/ลบ/อนุมัติอะไร เมื่อไหร่" (CreatedBy, UpdatedBy, Timestamp, Action Log)
3. **Data Export & Search / Filter**:
   - พนักงานชอบ Excel/CSV เสมอ ต้องออกแบบให้ Export ได้ง่าย และค้นหาเร็ว
4. **Simple Status & Notifications**:
   - แจ้งเตือนผ่าน LINE Notify, Discord Webhook, หรือ Email เมื่อมีการร้องขอ/อนุมัติ

---

## 📌 ขั้นตอนที่ 3: ซอยตั๋วและเขียนโค้ด (The Implementation Flow)

ทำงานแบบ 1 Ticket = 1 Session ตามหลัก **The Main Flow**:

1. รัน `/to-tickets` เพื่อแตก Spec ออกเป็นไฟล์ย่อยใน `tickets/`
2. เปิด Session ใหม่ รันคำสั่ง:
   ```text
   อ่าน AGENTS.md และทำตาม ticket: tickets/.../01_auth_rbac.md ด้วย skill: tdd
   ```
3. รัน `/code-review` เพื่อตรวจเช็คความถูกต้องและ Security (ป้องกัน Hardcoded Secret / Data Leak)
4. บันทึกประวัติด้วย `jj describe -m "feat(asset): implement requisition approval flow"`

---

## 📦 ขั้นตอนที่ 4: เอกสารที่ต้องทำส่งมอบ (Handover Deliverables Checklist)

สำหรับงาน Internal Tools เอกสารไม่ต้องยาวเป็นร้อยหน้า แต่ต้อง **"ตรงจุด ใช้งานเป็น ดูแลต่อได้"**:

### Checklist 4 เอกสารสำคัญที่ต้องมี:

| เอกสาร | กลุ่มเป้าหมาย | ความยาว | สิ่งที่ต้องมี |
|---|---|---|---|
| **1. Quick User Manual** (คู่มือพนักงาน) | พนักงานทั่วไป / End Users | 1-2 หน้า (มีภาพ/GIF ประกอบ) | - วิธี Login ครั้งแรก<br>- ขั้นตอนทำรายการหลัก 3-4 ขั้นตอน<br>- FAQ ปัญหาที่พบบ่อย |
| **2. Admin & Ops Guide** (คู่มือแอดมิน) | IT Admin / ฝ่ายที่ดูแลระบบ | 2-3 หน้า | - วิธีเพิ่ม/ลดสิทธิ์ผู้ใช้<br>- วิธีจัดการ Master Data / แก้ไขข้อมูลที่ผิด<br>- วิธี Export ข้อมูลและดู Audit Log |
| **3. Technical & Runbook** (คู่มือทางเทคนิค) | คุณในอนาคต / Dev คนถัดไป | 1-2 หน้า (`README.md`) | - Architecture Diagram & Tech Stack<br>- วิธี Run Local & Environment Variables (`.env.example`)<br>- Database Migration & Backup/Restore Procedure<br>- Production Deployment Steps (Docker / CI/CD) |
| **4. Release Notes & Demo** | หัวหน้า / ผู้บริหาร | 1 หน้าสรุป + Demo 5 นาที | - สรุปฟีเจอร์ที่ปล่อยในเวอร์ชันนี้<br>- สิ่งที่ระบบจะช่วยประหยัดเวลา/ลดข้อผิดพลาด |

---

## 📊 ขั้นตอนที่ 5: การติดตามงาน (Task Tracking) และการรายงานหัวหน้า (Management Reporting)

### 1. วิธี Track งานและ Feature เมื่อทำหลายโปรเจกต์คู่ขนาน:
ใช้ระบบไฟล์ Markdown หรือ Tool ง่ายๆ (เช่น Linear, Notion, GitHub Projects) โดยแบ่ง Track:
- `spec/[module]/` (เช่น `spec/core/01_plan.md`, `02_api.md`, `03_schema.md`, `04_ticket.md` หรือแยก `04_ticket01.md`, `04_ticket02.md` ถ้างานเยอะ) รวมจบในโฟลเดอร์เดียว ไม่ต้องสร้างโฟลเดอร์ tickets แยก

### 2. เทมเพลตรายงานหัวหน้า / ผู้บริหาร (Weekly / Milestone Update):

```markdown
### 🚀 สรุปความคืบหน้าโปรเจกต์: [ชื่อโปรเจกต์ เช่น Internal Asset Management]
**สถานะภาพรวม**: 🟢 On Track (พร้อมส่งมอบตามกำหนด)

#### 1. สิ่งที่ทำเสร็จแล้วในสัปดาห์นี้ (Completed):
- [x] ระบบ Login และกำหนดสิทธิ์ (Employee vs IT Admin)
- [x] หน้าฟอร์มเบิกอุปกรณ์พร้อมระบบเช็กสต๊อก Realtime
- [x] ระบบแจ้งเตือนการอนุมัติผ่าน LINE Notify

#### 2. สิ่งที่กำลังจะทำในสัปดาห์ถัดไป (Next Steps):
- [ ] ระบบ Export รายงานประจำเดือนเป็น Excel
- [ ] ทดสอบระบบร่วมกับฝ่าย IT และจัดทำ Quick User Manual

#### 3. สรุปผลลัพธ์และคุณค่าที่พนักงานจะได้รับ (Impact/Value):
- ช่วยลดเวลาในการเดินเอกสารเบิกของจากเดิม 2 วัน เหลือไม่เกิน 15 นาที
- ป้องกันปัญหาของในสต๊อกหายหรือไม่ตรงกับยอดจริง 100%

#### 4. ข้อติดขัด / สิ่งที่ต้องการความช่วยเหลือ (Blockers/Help Needed):
- ต้องการรายชื่อ Master Data หมวดหมู่อุปกรณ์จากพี่แอดมิน IT ภายในวันพุธนี้ครับ
```

---

## 💡 สรุป Mindset สำหรับ Solo Fullstack Dev
1. **เริ่มจากปัญหาผู้ใช้ (Pain Point) เสมอ**: อย่าเพิ่งคิดเรื่องโค้ด ให้ใช้ `/grill-me` คุยปัญหาให้ชัดก่อน
2. **ระบบภายในเน้น Speed to Value & Reliability**: UI คลีน ใช้งานง่าย ทำงานเสร็จไว ดีกว่า UI หรูหราแต่ใช้ยาก
3. **ใช้ AI เป็น Fullstack Pair-programmer**: ตั้งแต่ช่วยตั้งคำถาม (Grill) -> ออกแบบ DB -> เขียน Test -> ตรวจสอบ Security -> ช่วยร่างคู่มือ User Manual
4. **สื่อสารผลลัพธ์เป็นภาษาธุรกิจ**: เวลาคุยกับหัวหน้า ให้พูดถึง "ลดเวลาทำงานกี่นาที / ข้อมูลถูกต้องขึ้นแค่ไหน" แทนที่จะพูดแค่เรื่องภาษาหรือเฟรมเวิร์กที่ใช้
