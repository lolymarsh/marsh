# Marsh — Full-Stack Developer

> 🔥 **Backend (Go)** · 🟢 **Backend (Node.js/Express)** · ⚡ **Frontend (Next.js + React)** · 🔌 **WordPress / WooCommerce** · 🤖 **AI/LLM + SSE** · 🐳 **DevOps** · 💾 **Backup & Recovery**

ออกแบบและพัฒนาระบบตั้งแต่ต้นจนจบ: Database → API (REST/RPC) → Frontend → Deploy → Monitor → Backup — รวมถึงดูแล Production จริง

---

## Backend Development (Go)

**ภาษา**: Go 1.25 (หลัก), TypeScript, JavaScript, Python, SQL
**Framework**: Echo v4, Fiber, Express, Fastify
**RPC Framework**: Connect RPC (Protobuf, buf, connect-go), gRPC
**WebSocket**: gorilla/websocket, real-time communication
**ORM / Database**: Bun ORM (MySQL, PostgreSQL, SQLite), GORM, SQLx, Squirrel
**Migration**: goose
**Database Transactions**: Multi-DB transaction management, ExecuteInTransaction helper
**Optimistic Locking**: version column + `WHERE version = ?` ป้องกัน data race

### Authentication & Authorization
- JWT (golang-jwt v5) — token-based auth
- OAuth2 Multi-Provider (Google, GitHub) — CSRF state, token exchange
- Role-Based Access Control (3-tier: ADMIN > STAFF > USER)
- Cloudflare Turnstile — bot protection middleware

### Integration
- LINE Messaging API — Push Flex Messages
- LINE LIFF v2 SDK — Social Login บน WordPress
- WooCommerce REST API — Custom Client (orders, products, coupons, customers, reports)
- AWS SDK Go v2 — S3, STS, SSO
- Cloudflare R2 — S3-compatible object storage
- SMTP / Email — ส่งอีเมลผ่าน SMTP server, Email notification

### Message Queue & Cache
- Redis — Cache, OAuth state, Queue (งานไม่หนัก)
- RabbitMQ — Message Queue
- Go Routine — Async worker (Audit Log, LINE Notify, WooCommerce Sync)
- conc (sourcegraph/conc) — Structured concurrency, worker pool

### Utility
- go-playground/validator — Struct tag validation
- go.uber.org/zap — Structured logging
- signintech/gopdf — PDF generation (certificates)
- skip2/go-qrcode — QR Code generation
- golang-jwt/jwt — JWT
- bwmarrin/snowflake — Distributed unique ID
- segmentio/ksuid — K-ordered unique ID
- golang.org/x/crypto — Password hashing
- golang.org/x/time — Rate limiting
- godotenv — Environment config
- copier — Deep copy between structs

### Architecture & Design
- Layered Architecture — Handler → Service → Repository → DB
- Dependency Injection — Manual DI container
- Interface-based Design — exported interfaces, unexported implementations
- Clean Architecture principles
- Error Handling — Custom AppError, sentinel errors, HTTP status mapping
- Unified JSON Response — `{code, message, data, pagination}`
- Soft Delete — `deleted_at = 0`, no hard delete
- Optimistic Locking — `version` column with `WHERE version = ?`
- Multi-DB Transactions — จัดการ transaction ข้ามหลาย database
- Swagger / OpenAPI — Auto-generated docs via swaggo, ReDoc UI

---

## Backend Development (Node.js / Express / TypeScript)

> ศึกษาและประยุกต์ Go-style architecture บน Node.js — Handler → Service → Repository → DB

**ภาษา**: TypeScript (strict), Node.js
**Framework**: Express.js
**ORM / Database**: Drizzle ORM (MySQL, PostgreSQL), Mongoose (MongoDB)
**Message Queue & Cache**: Redis (ioredis), RabbitMQ (amqplib)
**Validation**: Zod — runtime type inference + schema validation
**Authentication**: JWT + bcrypt + Redis sessions

### Architecture (Go-Style Domain Modules)
- **1 folder = 1 business domain** — `modules/{module}/` เลียนแบบ Go `internal/{module}/`
- แต่ละ module ประกอบด้วย 6 files:
  `entity.ts` · `schema.ts` · `handler.ts` · `service.ts` · `repo.ts` · `route.ts`
- Interface-based Dependency Injection — exported interfaces, class-based implementations
- Unified JSON Response — `{code, message, data, pagination}`
- Optimistic Locking — `version` column + `WHERE version = ?`
- Multi-DB Transactions — Drizzle `db.transaction()` + `FOR UPDATE` row locks
- Soft Delete — `deleted_at = null`, no hard delete
- Custom Error Classes — AppError, NotFoundError, ConflictError, UnauthorizedError
- RabbitMQ Workers — Async report generation, notifications, AI heavy queries, audit logging
- **Polyglot Persistence**:
  - **MySQL** — Core business data (ACID, Relations)
  - **MongoDB** — Chat history, activity/audit logs (high-write, flexible schema)
  - **Redis** — Cache, sessions, rate limiting, AI query cache

---

## Frontend Development

**Framework**: Next.js 16 (App Router), React 19 (Server Components + Vite)
**ภาษา**: TypeScript 5 (strict mode), JavaScript
**Build Tool**: Vite, Next.js
**Routing**: React Router v7, Next.js App Router
**State Management**: Zustand 5
**UI / Styling**:
- Tailwind CSS v4 — Utility-first CSS
- ShadCN UI — Component library (Radix/base-ui primitives)
- MUI (Material UI) v6 — DataGrid, Tables, Forms, Dialogs
- Lucide React — Icon library
- Recharts — Dashboard charts

**Forms & Validation**:
- React Hook Form
- Zod Schema Validation

**Architecture Pattern**:
- **MVC Pattern** (React) — `model.ts` (API calls + types) / `view.tsx` (presentation) / `controller.ts` (custom hook)
  - Model — Pure TypeScript, no React imports, API calls + type definitions
  - View — React component, presentation-only, receives props, no API calls
  - Controller — `useXxx()` custom hook, state + logic, orchestrates Model ↔ View

**HTTP Client**: Axios — interceptors (auth token, 401 redirect, error normalization)
**API Service Layer**: Custom wrapper functions (GET, POST, PATCH, PUT, DELETE, Upload)
**Testing**: Playwright — E2E testing
**Other**: CSV Export (BOM สำหรับภาษาไทย), Mock Data for development

---

## AI / LLM Integration & SSE Streaming

**LLM Providers**: OpenAI (GPT-4o / GPT-4o-mini), Google Gemini Flash, Anthropic Claude
**AI Capabilities**:
- Natural Language → SQL Query Generation (ถามภาษาคน → แปลงเป็น Query → execute)
- Intent Detection + Entity Extraction
- SQL Sanitizer — Read-only guard, ป้องกัน DROP/UPDATE/DELETE
- Response Formatter — Text, Table, CSV, HTML, JSON

**SSE (Server-Sent Events)**:
- SSE Streaming — ส่งข้อมูลจาก Backend → Frontend แบบ real-time สำหรับ AI Chat
- Chatbot stream response ทีละ chunk (Token Streaming)
- Async job status push — Backend → SSE → Frontend (polling fallback)
- Notification push via SSE / Long Polling

**AI Architecture Flow**:
```
User Input → Intent Detection (LLM) → SQL Generation → Sanitizer → Execute (MySQL)
→ Cache (Redis, 10min TTL) → Format Response → SSE Stream → User
(Heavy queries → RabbitMQ → AI Worker → Redis → Polling → Frontend)
```

**Cache Strategy**: Redis `ai:cache:{md5(question)}` — 10 min TTL, ถามซ้ำตอบจาก cache ไม่เรียก LLM

## MCP (Model Context Protocol) — กำลังศึกษา

- กำลังศึกษา MCP เพื่อทำความเข้าใจการสร้าง Tools/Resources สำหรับ AI Agent
- สนใจการ integrate MCP Server เข้ากับระบบ ERP เพื่อให้ AI เข้าถึงข้อมูลธุรกิจได้โดยตรง

---

## WordPress / WooCommerce

**Plugin Development**:
- WooCommerce Payment Gateway — Custom PromptPay QR (extends WC_Payment_Gateway)
- WooCommerce Blocks — Payment Method Type integration
- LINE LIFF v2 — Social Login, auto-create WP users
- Gutenberg Blocks — Custom dynamic blocks, server-side rendering
- Shortcodes — For Elementor/page builder compatibility
- Must-Use Plugin (MU) — Domain routing, auth header fixes

**Optimization**:
- Redis Object Cache — WordPress integration
- OPcache JIT — PHP JIT compilation
- WP-Cron Disabled — Use system cron instead

---

## DevOps & Infrastructure

**Container / Orchestration**:
- Docker — Multi-service compose (7 services), Multi-stage builds
- Kubernetes (K8s, k3s)

**Reverse Proxy / Ingress**:
- Traefik v2 — Let's Encrypt, Docker provider, TLS config
- Nginx — SSL termination, HTTP/2, Rate limiting, Security headers, CORS

**Database**:
- MariaDB 11.4, MySQL 8.4, PostgreSQL
- MongoDB 7 — NoSQL (logs, chat history, audit trail)
- Custom config tuning (my.cnf)
- Health checks for every service

**Security**:
- Cloudflare Zero Trust — Tunnel, Access, Local Network
- Cloudflare Turnstile — CAPTCHA replacement
- VPN
- Self-signed Certificate — Auto-generation via OpenSSL
- Docker socket mounting — For backup agent docker commands

**Monitoring & Maintenance**:
- ดูแลเซิร์ฟเวอร์ Production จริง ทั้ง Dev และ Production Environment
- Healthcheck endpoints ทุก service
- Structured logging (Zap)

---

## Backup & Recovery

**Scheduling**: Go cron (robfig/cron) — runs daily 03:00 AM (configurable)

**Backup Types**:
- Database dump (mysqldump → gzip)
- File archive (tar.gz)
- S3 Offsite Storage — AWS SDK v2 (R2/MinIO compatible)

**Verification**:
- Gunzip integrity check
- Restore to test database
- Table count validation

**Restore**: Safety backup → Drop all tables → Restore → Verify → Restart services
**Custom Restore UI**: REST API + Interface สำหรับ restore MySQL & PostgreSQL
**Notification**: LINE Notify — Backup summary report
**Retention**: Configurable retention days, automatic cleanup

---

## Testing

- Table-driven tests (Go standard)
- Integration tests — Real PostgreSQL/MySQL
- Seed factories — Comprehensive test data (1100+ lines)
- Playwright — E2E (login, navigation, payments, reports)
- Hurl — E2E HTTP API testing
- testutil — JWT token generation, DB setup/truncation
- **Vitest + React Testing Library** — Frontend unit + component tests
- **MSW (Mock Service Worker)** — Mock API responses in frontend tests
- **Jest + ts-jest** — Backend unit tests (services, utils, middleware)
- **Supertest + Testcontainers** — Backend integration tests with real throwaway DBs
- **Contract Testing** — API response schema validation (Zod)

---

## Tooling & Automation

- Makefile — Build, test, lint, migrate, docker, swagger
- golangci-lint v2 — 25+ linters (gosec, errcheck, staticcheck, gocyclo, etc.)
- govulncheck — Vulnerability scanning
- go vet / gofmt / goimports
- Air — Hot reload (development)
- ESLint 9 + eslint-config-next
- PostCSS with Tailwind
- Vite — Build tool (development + production)
- tsx — TypeScript execution (Node.js hot reload)
- Version Control: Git / Jujutsu (jj)

---

## การออกแบบระบบ

- ออกแบบ Database Schema — MySQL, PostgreSQL (Tables, Indexes, ENUMs, Relations)
- ออกแบบ REST API / RPC Endpoints
- ออกแบบ Folder Structure และ Software Architecture
- ออกแบบ Flow Chart และ Business Logic
- Code Refactoring — ยกระดับระบบจากภาษาเดิม สู่ Go
- API Documentation — Swagger, Postman

---

> **สามารถออกแบบและพัฒนาระบบตั้งแต่ต้นจนจบ**: Database → API (REST/RPC) → Frontend → Deploy → Monitor → Backup — รวมถึงดูแล Production จริง
>
> **Key Strengths**: Go Backend, Node.js/Express Backend, Next.js Frontend (MVC + RSC), React MVC Frontend, WordPress/WooCommerce, Docker/K8s Production Ops, AI/LLM Integration + SSE Streaming, Backup & Recovery Automation
