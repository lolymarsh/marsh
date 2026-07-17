# Marsh — Full-Stack Developer

> 🔥 **Backend (Go)** · ⚡ **Frontend (Next.js)** · 🔌 **WordPress / WooCommerce** · 🐳 **DevOps** · 💾 **Backup & Recovery**

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

## Frontend Development

**Framework**: Next.js 16 (App Router), React 19 (Server Components)
**ภาษา**: TypeScript 5 (strict mode), JavaScript
**State Management**: Zustand 5
**UI / Styling**:
- Tailwind CSS v4 — Utility-first CSS
- ShadCN UI — Component library (Radix/base-ui primitives)
- MUI (Material UI)
- Lucide React — Icon library
- Recharts — Dashboard charts

**Forms & Validation**:
- React Hook Form
- Zod Schema Validation

**HTTP Client**: Axios — interceptors (auth token, 401 redirect, error normalization)
**API Service Layer**: Custom wrapper functions (GET, POST, PATCH, PUT, DELETE, Upload)
**Testing**: Playwright — E2E testing
**Other**: CSV Export (BOM สำหรับภาษาไทย), Mock Data for development

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
- MariaDB 11.4, MySQL, PostgreSQL
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

---

## Tooling & Automation

- Makefile — Build, test, lint, migrate, docker, swagger
- golangci-lint v2 — 25+ linters (gosec, errcheck, staticcheck, gocyclo, etc.)
- govulncheck — Vulnerability scanning
- go vet / gofmt / goimports
- Air — Hot reload (development)
- ESLint 9 + eslint-config-next
- PostCSS with Tailwind
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
> **Key Strengths**: Go Backend, Next.js Frontend, WordPress/WooCommerce, Docker/K8s Production Ops, Backup & Recovery Automation
