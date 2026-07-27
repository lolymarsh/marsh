---
trigger: always_on
---

# Reverse Proxy Patterns (DevOps)

> CORS, Security Headers, Rate Limiting ทั้งหมดดักที่ Reverse Proxy layer (Nginx/Traefik) — ไม่ต้องเขียนใน application code

## 1. Principle: Handle at Gateway

| Concern | Where to handle | Why |
|---|---|---|
| **CORS** | Reverse Proxy | Single point of control, ไม่ต้อง config ทุก service |
| **Security Headers** | Reverse Proxy | HSTS, XSS, Content-Type — ใช้ได้ทุก response |
| **Rate Limiting** | Reverse Proxy | ดักก่อนถึง app, ป้องกัน DDoS |
| **SSL/TLS** | Reverse Proxy | Termination ที่ edge |
| **HTTP → HTTPS redirect** | Reverse Proxy | 301 redirect |
| **Body Size Limit** | Reverse Proxy | ป้องกัน upload ใหญ่เกิน |
| **Request Timeout** | Reverse Proxy | ป้องกัน slow loris |

## 2. Traefik Middlewares (K8s)

### CORS Headers (Reusable)

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: cors-headers
  namespace: default
spec:
  headers:
    accessControlAllowCredentials: true
    accessControlAllowOriginList:
      - "https://dev.example.com"
      - "https://staging.example.com"
      - "https://example.com"
    accessControlAllowMethods:
      - GET
      - POST
      - PUT
      - PATCH
      - DELETE
      - OPTIONS
    accessControlAllowHeaders:
      - Authorization
      - Content-Type
      - X-Request-ID
    accessControlMaxAge: 3600
```

### Security Headers

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: security-headers
spec:
  headers:
    frameDeny: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsSeconds: 31536000
    stsIncludeSubdomains: true
    stsPreload: true
    referrerPolicy: strict-origin-when-cross-origin
    customFrameOptionsValue: SAMEORIGIN
    permissionsPolicy: "camera=(), microphone=(), geolocation=()"
```

### Rate Limiting

```yaml
# Strict (auth endpoints)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ratelimit-strict
spec:
  rateLimit:
    average: 100
    burst: 50

# Normal (standard API)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ratelimit-normal
spec:
  rateLimit:
    average: 200
    burst: 100

# Relaxed (read-heavy endpoints)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ratelimit-relaxed
spec:
  rateLimit:
    average: 500
    burst: 200
```

### Body Size Limit

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: body-size-10mb
spec:
  buffering:
    maxRequestBodyBytes: 10485760  # 10MB
```

### HTTPS Redirect

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect-https
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

### IP Whitelist (Internal only)

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist-vpc
spec:
  ipWhiteList:
    sourceRange:
      - "10.104.0.0/20"   # VPC
      - "10.8.8.0/24"     # VPN
```

## 3. Nginx (Docker Compose)

### Main Config (`nginx.conf`)

```nginx
# Rate limiting zones
limit_zone api_zone $binary_remote_addr zone=api_limit:10m rate=60r/s;
limit_zone wp_zone $binary_remote_addr zone=wp_limit:10m rate=100r/s;

# SSL settings
ssl_protocols TLSv1.2 TLSv1.3;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;

# HTTP → HTTPS
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

### Per-Service Config (`conf.d/api.conf`)

```nginx
upstream api_backend {
    server api:3000;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL
    ssl_certificate /etc/nginx/certs/api.crt;
    ssl_certificate_key /etc/nginx/certs/api.key;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # CORS
    set $cors_origin "";
    if ($http_origin ~* "^https://(dev\.|staging\.)?example\.com$") {
        set $cors_origin $http_origin;
    }
    add_header Access-Control-Allow-Origin $cors_origin always;
    add_header Access-Control-Allow-Credentials "true" always;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Authorization, Content-Type" always;

    # CORS preflight
    if ($request_method = 'OPTIONS') {
        add_header Access-Control-Allow-Origin $cors_origin;
        add_header Access-Control-Max-Age 3600;
        add_header Content-Length 0;
        return 204;
    }

    # Rate limiting
    limit_req zone=api_limit burst=20 nodelay;

    # Proxy
    location / {
        proxy_pass http://api_backend;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        # Timeouts
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;

        # Body size
        client_max_body_size 10m;
    }
}
```

## 4. Per-App Middleware Strategy

### Option A: Combined (Recommended for small apps)

```yaml
# Single middleware with CORS + security headers
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: {app}-combined
spec:
  headers:
    # Security
    frameDeny: true
    contentTypeNosniff: true
    browserXssFilter: true
    # CORS
    accessControlAllowOriginList:
      - "https://{app}.example.com"
    accessControlAllowMethods: ["GET", "POST", "PUT", "DELETE"]
    accessControlAllowHeaders: ["Authorization", "Content-Type"]
```

### Option B: Separated (Recommended for production)

```yaml
# Chain multiple middlewares in IngressRoute
middlewares:
  - name: {app}-cors          # CORS only
  - name: security-headers    # Shared security headers
  - name: ratelimit-normal    # Shared rate limit
  - name: body-size-10mb      # Shared body limit
```

## 5. Application Code Rules

| Concern | App Code | Reverse Proxy |
|---|---|---|
| CORS headers | ✗ Don't add | ✓ Handle here |
| Security headers | ✗ Don't add | ✓ Handle here |
| Rate limiting | ✗ Don't add | ✓ Handle here |
| Request validation | ✓ Zod schema | ✗ |
| Auth verification | ✓ JWT middleware | ✗ |
| Business logic auth | ✓ Service layer | ✗ |
