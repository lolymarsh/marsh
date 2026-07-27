---
trigger: always_on
---

# Docker Patterns (DevOps)

## 1. Dockerfile — Multi-Stage Build

### Go Backend
```dockerfile
# Stage 1: Build
FROM golang:1.25 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/server .

# Stage 2: Runtime
FROM alpine:latest
RUN apk add --no-cache tzdata ca-certificates
COPY --from=builder /app/server /server
EXPOSE 8000
CMD ["/server"]
```

### Next.js Frontend
```dockerfile
# Stage 1: Dependencies
FROM node:22-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Stage 2: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx next build

# Stage 3: Runtime
FROM node:22-alpine AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

### Node.js Backend (Express/Fastify)
```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 appuser
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
USER appuser
EXPOSE 3000
CMD ["node", "dist/app.js"]
```

## 2. Dockerfile Rules

| Rule | Reason |
|---|---|
| Always multi-stage | Smaller image, no build tools in production |
| Alpine base | Minimal attack surface |
| Non-root user | Security best practice |
| `COPY` package files first | Layer caching for dependencies |
| No `COPY . .` before `npm ci` | Breaks layer caching |
| `EXPOSE` before `CMD` | Documentation |
| Health check | `HEALTHCHECK` directive or compose healthcheck |

## 3. Docker Compose — Local Dev

```yaml
services:
  api:
    build: ./backend
    ports:
      - "127.0.0.1:3000:3000"  # localhost only
    environment:
      - NODE_ENV=development
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  web:
    build: ./frontend
    ports:
      - "127.0.0.1:5173:5173"
    environment:
      - VITE_API_BASE_URL=http://localhost:3000
    networks:
      - web

  db:
    image: mysql:8.0
    ports:
      - "127.0.0.1:3306:3306"  # localhost only
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "127.0.0.1:6379:6379"  # localhost only
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    networks:
      - backend

networks:
  web:
    driver: bridge
  backend:
    driver: bridge

volumes:
  db_data:
```

## 4. Docker Compose — Production (Server)

```yaml
services:
  traefik:
    image: traefik:v3.0
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - web

  api:
    image: ${API_IMAGE:-myapp-api:latest}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.services.api.loadbalancer.server.port=3000"
    env_file:
      - .env
    networks:
      - web
      - backend
    restart: unless-stopped

  web:
    image: ${WEB_IMAGE:-myapp-web:latest}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`example.com`)"
      - "traefik.http.routers.web.entrypoints=websecure"
      - "traefik.http.routers.web.tls.certresolver=letsencrypt"
    networks:
      - web
    restart: unless-stopped
```

## 5. Environment Variable Rules

| Rule | Example |
|---|---|
| `.env.example` committed | Template with placeholder values |
| `.env` gitignored | Actual secrets |
| Secrets in compose: use `env_file` | Not inline `environment` for sensitive data |
| Image tags overrideable | `${API_IMAGE:-myapp-api:latest}` |
| DB ports: localhost only | `127.0.0.1:3306:3306` |
