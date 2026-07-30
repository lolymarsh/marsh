# Roadmap: Versus Thailand ERP + AI Chatbot

> บริษัท Versus Thailand — 500 สาขาทั่วไทย
>
> เป้าหมาย: Web ERP ที่มี AI Chatbot ถาม-ตอบข้อมูล พร้อม export หลาย format

---

## ภาพรวมระบบ

```
┌─────────────────┐      ┌───────────────────┐      ┌───────────────┐
│   Chat UI       │─────▶│   API Server      │─────▶│   LLM API     │
│   (React/Next)  │      │   (Nest.js)       │      │   (Claude)    │
│                 │◀─────│                   │◀─────│               │
└─────────────────┘      └───────┬───────────┘      └───────┬───────┘
                                 │                          │
                          ┌──────▼───────────┐     Function Calling
                          │   PostgreSQL     │◀──────────────┘
                          │   (500 สาขา)     │
                          └──────────────────┘
```

**User ถาม → AI เข้าใจ → Query DB → แปลง format → ส่งกลับ**

---

## Tech Stack ที่แนะนำ

| Layer | Technology | เหตุผล |
|-------|-----------|--------|
| Frontend | Next.js + TypeScript | SSR, ใช้ได้ทั้ง ERP และ Chat |
| Backend | Nest.js (Node) | Structure ชัด, เหมาะกับระบบใหญ่ |
| Database | PostgreSQL | รองรับ complex queries, 500 สาขา |
| ORM | Prisma | Type-safe, migration ง่าย |
| AI | Claude API (Anthropic) | Function calling ดีที่สุดตอนนี้ |
| Export | exceljs, docx, pdfkit | สร้าง xlsx, docs, pdf |
| Auth | NextAuth / JWT | Multi-branch, role-based |
| Deploy | Docker + VPS/Cloud | ควบคุมได้ |

---

## Phase 0: Planning & Setup (สัปดาห์ที่ 1)

### 0.1 ออกแบบ Database Schema

**สิ่งที่ต้องคิดก่อนเขียนโค้ดบรรทัดเดียว**

```
branches (500 สาขา)
├── id, name, address, region, manager_id
│
users (พนักงานทุกสาขา)
├── id, name, branch_id, role (admin/manager/staff)
│
products / inventory
├── id, name, sku, price, category
│
stock_levels (สาขา x สินค้า)
├── branch_id, product_id, quantity, last_updated
│
sales
├── id, branch_id, product_id, quantity, amount, date
│
purchase_orders
├── id, branch_id, supplier_id, status, items[]
│
transfers (โอนสินค้าระหว่างสาขา)
├── id, from_branch, to_branch, product_id, quantity, status
```

**Key decisions:**
- Multi-tenancy: ทุกสาขา share DB เดียว หรือแยก?
- Partitioning: partition ตาราง sales ตามเดือน/สาขา
- Indexing: index ที่ branch_id, date, product_id

### 0.2 ตั้ง Project

```bash
# Monorepo
mkdir versus-erp && cd versus-erp
pnpm init
mkdir apps/web apps/api packages/shared

# Backend
cd apps/api
npx @nestjs/cli new . --package-manager pnpm
pnpm add @prisma/client prisma
pnpm add @anthropic-ai/sdk

# Frontend
cd apps/web
npx create-next-app@latest . --typescript --tailwind

# Export libs
pnpm add exceljs docx pdfkit
```

### 0.3 ตั้ง Dev Environment

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: versus_erp
      POSTGRES_USER: admin
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

---

## Phase 1: Core ERP (สัปดาห์ที่ 2-4)

### 1.1 Auth & Branch System

```
POST /auth/login          → JWT token (with branch_id + role)
GET  /auth/me             → current user + branch info

Middleware:
- attachBranch()          → inject branch_id from token
- requireRole(['admin'])  → role-based access
```

### 1.2 CRUD หลัก

```
# Branch management
GET    /branches          → list ทุกสาขา (filter by region)
GET    /branches/:id      → รายละเอียดสาขา
POST   /branches          → สร้างสาขา (admin only)

# Products
GET    /products           → list สินค้า (filter, search, paginate)
POST   /products           → เพิ่มสินค้า
PATCH  /products/:id       → แก้ไข

# Inventory (per branch)
GET    /branches/:id/stock → สต็อกสาขา
POST   /branches/:id/stock/adjust → ปรับสต็อก

# Sales
POST   /sales              → บันทึกขาย
GET    /sales/report        → รายงาน (filter by branch, date, product)

# Transfers
POST   /transfers          → ขอโอนสินค้า
PATCH  /transfers/:id      → อัพเดทสถานะ
```

### 1.3 Database Query Layer สำหรับ AI

**สำคัญ: สร้าง query functions ที่ AI เรียกได้**

```typescript
// src/queries/branch-queries.ts
export async function getBranchSales(branchId: string, dateFrom: string, dateTo: string) {
  return prisma.sales.aggregate({
    where: { branchId, date: { gte: dateFrom, lte: dateTo } },
    _sum: { amount: true, quantity: true },
    _count: true,
  });
}

export async function getLowStock(branchId: string, threshold: number = 10) {
  return prisma.stockLevel.findMany({
    where: { branchId, quantity: { lt: threshold } },
    include: { product: true },
  });
}

export async function getTopProducts(branchId: string, limit: number = 10) {
  // ... query top selling products
}
```

---

## Phase 2: AI Chatbot (สัปดาห์ที่ 5-6)

### 2.1 เรียนก่อนลงมือ (1-2 วัน)

**อ่าน 2 อย่างนี้ก่อน:**
- [Anthropic: Tool Use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
- [Anthropic: Prompt Engineering](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

### 2.2 สร้าง AI Service

```typescript
// src/ai/ai.service.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// Tools ที่ AI สามารถเรียกได้
const tools = [
  {
    name: 'query_branch_sales',
    description: 'ดึงข้อมูลยอดขายของสาขาที่ระบุ ในช่วงวันที่กำหนด',
    input_schema: {
      type: 'object' as const,
      properties: {
        branch_id: { type: 'string', description: 'รหัสสาขา' },
        date_from: { type: 'string', description: 'วันที่เริ่มต้น (YYYY-MM-DD)' },
        date_to: { type: 'string', description: 'วันที่สิ้นสุด (YYYY-MM-DD)' },
      },
      required: ['branch_id', 'date_from', 'date_to'],
    },
  },
  {
    name: 'query_stock_level',
    description: 'ดึงข้อมูลสต็อกสินค้าของสาขา หรือดูสินค้าที่สต็อกต่ำ',
    input_schema: {
      type: 'object' as const,
      properties: {
        branch_id: { type: 'string', description: 'รหัสสาขา (ไม่ใส่ = ทุกสาขา)' },
        low_stock_only: { type: 'boolean', description: 'แสดงเฉพาะสินค้าสต็อกต่ำ' },
        product_name: { type: 'string', description: 'ชื่อสินค้า (ค้นหา)' },
      },
      required: [],
    },
  },
  {
    name: 'query_branch_comparison',
    description: 'เปรียบเทียบยอดขายระหว่างสาขา หรือระหว่าง region',
    input_schema: {
      type: 'object' as const,
      properties: {
        compare_by: { type: 'string', enum: ['branch', 'region'], description: 'เปรียบเทียบตามอะไร' },
        date_from: { type: 'string' },
        date_to: { type: 'string' },
        metric: { type: 'string', enum: ['revenue', 'quantity', 'transactions'] },
      },
      required: ['compare_by', 'date_from', 'date_to'],
    },
  },
  {
    name: 'export_data',
    description: 'ส่งออกข้อมูลเป็นไฟล์ excel, word, pdf หรือ html',
    input_schema: {
      type: 'object' as const,
      properties: {
        format: { type: 'string', enum: ['xlsx', 'docs', 'pdf', 'html', 'plain'] },
        data_query: { type: 'string', description: 'คำอธิบายข้อมูลที่ต้องการ export' },
      },
      required: ['format'],
    },
  },
];

// System prompt
const SYSTEM_PROMPT = `คุณเป็น AI ผู้ช่วยของระบบ ERP บริษัท Versus Thailand (500 สาขา)

หน้าที่:
- ตอบคำถามเกี่ยวกับข้อมูลบริษัท (ยอดขาย, สต็อก, สาขา)
- ใช้ tools เพื่อดึงข้อมูลจริงจากฐานข้อมูล
- ถามผู้ใช้ว่าต้องการ format ไหน (plain text, HTML, Excel, Word, PDF)
- ตอบเป็นภาษาไทย unless specified otherwise

กฎ:
- ใช้ tool เพื่อดึงข้อมูลเสมอ อย่าเดา
- ถ้าไม่แน่ใจว่าสาขาไหน ให้ถาม
- สรุปข้อมูลให้อ่านง่าย พร้อมตัวเลข`;

export class AiService {
  async chat(messages: any[], userId: string, branchId: string) {
    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 4096,
      system: SYSTEM_PROMPT,
      tools,
      messages,
    });

    // ถ้า AI ต้องการเรียก tool
    if (response.stop_reason === 'tool_use') {
      const toolUse = response.content.find((c) => c.type === 'tool_use');
      const result = await this.executeTool(toolUse.name, toolUse.input, branchId);

      // ส่งผลลัพธ์กลับให้ AI สรุป
      const followUp = await client.messages.create({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4096,
        system: SYSTEM_PROMPT,
        tools,
        messages: [
          ...messages,
          { role: 'assistant', content: response.content },
          {
            role: 'user',
            content: [{ type: 'tool_result', tool_use_id: toolUse.id, content: JSON.stringify(result) }],
          },
        ],
      });

      return followUp;
    }

    return response;
  }

  private async executeTool(name: string, input: any, branchId: string) {
    switch (name) {
      case 'query_branch_sales':
        return getBranchSales(input.branch_id, input.date_from, input.date_to);
      case 'query_stock_level':
        return getStockLevel(input.branch_id || branchId, input.low_stock_only, input.product_name);
      case 'query_branch_comparison':
        return compareBranches(input.compare_by, input.date_from, input.date_to, input.metric);
      case 'export_data':
        return { needs_format_choice: true, query: input.data_query };
      default:
        return { error: 'Unknown tool' };
    }
  }
}
```

### 2.3 Export Service

```typescript
// src/export/export.service.ts
import ExcelJS from 'exceljs';
import { Document, Packer, Paragraph, Table } from 'docx';
import PDFDocument from 'pdfkit';

export class ExportService {
  async toXlsx(data: any[], columns: string[]): Promise<Buffer> {
    const workbook = new ExcelJS.Workbook();
    const sheet = workbook.addWorksheet('Report');

    sheet.columns = columns.map((col) => ({ header: col, key: col }));
    data.forEach((row) => sheet.addRow(row));

    return workbook.xlsx.writeBuffer() as Promise<Buffer>;
  }

  async toDocs(title: string, data: any[]): Promise<Buffer> {
    const doc = new Document({
      sections: [{ children: [new Paragraph({ text: title, heading: 'Heading1' }), ...this.dataToParagraphs(data))] }],
    });
    return Packer.toBuffer(doc);
  }

  async toPdf(data: any[]): Promise<Buffer> {
    return new Promise((resolve) => {
      const doc = new PDFDocument();
      const chunks: Buffer[] = [];
      doc.on('data', (chunk) => chunks.push(chunk));
      doc.on('end', () => resolve(Buffer.concat(chunks)));
      data.forEach((row) => doc.text(JSON.stringify(row)));
      doc.end();
    });
  }

  async toHtml(data: any[], columns: string[]): Promise<string> {
    const headers = columns.map((c) => `<th>${c}</th>`).join('');
    const rows = data.map((r) => `<tr>${columns.map((c) => `<td>${r[c]}</td>`).join('')}</tr>`).join('');
    return `<table><thead><tr>${headers}</tr></thead><tbody>${rows}</tbody></table>`;
  }
}
```

### 2.4 Chat API Endpoint

```
POST /chat
Body: { message: "ยอดขายสาขา กทม. เดือน ม.ค. เท่าไหร่", format?: "xlsx" }
Response: { reply: "สรุป...", file?: { url, type, name } }

GET  /chat/history/:sessionId
Response: { messages: [...] }
```

---

## Phase 3: Format Selection UX (สัปดาห์ที่ 6)

### Chat UI Flow

```
User:  "ยอดขายรวมทุกสาขาเดือนนี้เท่าไหร่"

AI:    "ดึงข้อมูลให้ครับ... ยอดขายรวมทุกสาขาเดือน ม.ค. 2026
        อยู่ที่ ฿45,230,000 (12,340 รายการ)

        ต้องการข้อมูลในรูปแบบไหนครับ?
        [Plain Text] [HTML] [Excel] [Word] [PDF]"

User:  [กด Excel]

AI:    "ส่งออก Excel ให้แล้วครับ 📎 sales-report-jan2026.xlsx"
```

---

## Phase 4: Hardening (สัปดาห์ที่ 7-8)

### 4.1 Security

- Row-level security: user เห็นเฉพาะสาขาตัวเอง (ยกเว้น admin)
- SQL injection: ใช้ Prisma/ORM เท่านั้น ไม่ raw query จาก AI
- Rate limiting: จำกัดจำนวน chat ต่อนาที
- Audit log: บันทึกทุกคำถามและการ query

### 4.2 Performance

- Cache ข้อมูล branch summary ด้วย Redis (TTL 5 นาที)
- Pagination ทุก query ที่อาจ return เยอะ
- Database connection pooling

### 4.3 AI Guardrails

```typescript
// ไม่ให้ AI query ข้อมูลเกินขอบเขต
const ALLOWED_TABLES = ['sales', 'stock_levels', 'products', 'branches'];
const MAX_ROWS_RETURNED = 1000;

// ตรวจสอบ SQL ที่ AI สร้างก่อน execute
function validateQuery(sql: string): boolean {
  const forbidden = ['DROP', 'DELETE', 'UPDATE', 'INSERT', 'ALTER', 'TRUNCATE'];
  return !forbidden.some((cmd) => sql.toUpperCase().includes(cmd));
}
```

---

## สิ่งที่ต้องเรียน (ตามลำดับ)

| ตอนไหน | เรียนอะไร | ใช้เวลา | จากที่ไหน |
|--------|----------|---------|----------|
| ก่อนเริ่ม | DB Schema Design | 1-2 วัน | YouTube: "database design for ERP" |
| Phase 1 | Nest.js + Prisma | 2-3 วัน | [Nest.js docs](https://docs.nestjs.com/) |
| Phase 2 | Claude Function Calling | 1-2 วัน | [Anthropic docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use) |
| Phase 2 | Prompt Engineering | 1 วัน | [Anthropic prompt guide](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview) |
| Phase 3 | Export libs (exceljs, docx) | 1 วัน | npm docs + examples |
| Phase 4 | Docker + Deployment | 1-2 วัน | [Docker Get Started](https://docs.docker.com/get-started/) |

**รวม: ~2 สัปดาห์เรียน + 6 สัปดาห์ทำ = 8 สัปดาห์**

---

## Checklist

### Phase 0: Planning
- [ ] Design DB schema (branches, users, products, stock, sales, transfers)
- [ ] Set up monorepo (apps/web, apps/api)
- [ ] Docker compose (postgres + redis)
- [ ] Set up Prisma + initial migration

### Phase 1: Core ERP
- [ ] Auth system (JWT + branch-based)
- [ ] Branch CRUD
- [ ] Product CRUD
- [ ] Inventory management (per branch)
- [ ] Sales recording + reports
- [ ] Transfer between branches
- [ ] Query functions สำหรับ AI

### Phase 2: AI Chatbot
- [ ] Claude API integration
- [ ] Define tools (sales query, stock query, comparison, export)
- [ ] Tool execution layer
- [ ] Chat endpoint
- [ ] Chat history (session management)

### Phase 3: Export & Format
- [ ] Excel export (exceljs)
- [ ] Word export (docx)
- [ ] PDF export (pdfkit)
- [ ] HTML table export
- [ ] Format selection in chat UI

### Phase 4: Production Ready
- [ ] Row-level security
- [ ] Rate limiting
- [ ] Audit logging
- [ ] Redis caching
- [ ] AI guardrails (validate queries)
- [ ] Docker production build
- [ ] Monitoring & health checks

---

## ไม่ต้องเรียน (สำหรับ project นี้)

- Machine Learning ❌
- Deep Learning ❌
- Neural Networks ❌
- Statistics & Probability ❌
- Linear Algebra ❌
- Calculus ❌
- Reinforcement Learning ❌
- 90% ของ lachinemearning course ❌

**สิ่งที่ต้องทำคือ: ลงมือทำ ERP + อ่าน Anthropic docs เรื่อง function calling**

---

*สร้างสำหรับ: Versus Thailand ERP Project*
