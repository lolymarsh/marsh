---
trigger: always_on
---

# Kubernetes Patterns (DevOps)

## 1. Directory Structure (Per-Environment)

```
k8s/
├── base/                         # Shared infrastructure
│   ├── traefik/                  # API Gateway (Traefik)
│   │   ├── values.yaml           # Helm values
│   │   ├── middlewares/          # Reusable TraefikMiddleware CRDs
│   │   └── examples/             # IngressRoute templates
│   └── registry/
│
├── environments/
│   ├── dev/                      # Development
│   │   └── {project}/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── configmap.yaml
│   │       ├── secrets.yaml          # gitignored
│   │       ├── secrets_example.yaml  # template
│   │       ├── ingressroute.yaml
│   │       └── middleware.yaml
│   ├── staging/
│   └── production/
```

## 2. Standard Manifests Per App

| File | Purpose |
|---|---|
| `deployment.yaml` | Pod spec, replicas, resources, probes |
| `service.yaml` | ClusterIP service |
| `configmap.yaml` | Non-secret env vars |
| `secrets.yaml` | Sensitive env vars (gitignored) |
| `secrets_example.yaml` | Template for onboarding |
| `ingressroute.yaml` | Traefik routing + TLS |
| `middleware.yaml` | App-specific CORS, rate limit, headers |

## 3. Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {app}-{env}
  namespace: {namespace}
  labels:
    app: {app}
    env: {env}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {app}
      env: {env}
  strategy:
    type: RollingUpdate
    maxSurge: 1
    maxUnavailable: 0
  template:
    metadata:
      labels:
        app: {app}
        env: {env}
    spec:
      imagePullSecrets:
        - name: registry-secret
      containers:
        - name: {app}
          image: registry.unicon.site/{app}:{env}-{commitSHA}
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: {app}-{env}-config
            - secretRef:
                name: {app}-{env}-secrets
          resources:
            requests:
              cpu: 50m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          securityContext:
            runAsNonRoot: true
            runAsUser: 1000
            readOnlyRootFilesystem: true
```

## 4. IngressRoute (Traefik CRD)

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: {app}-{env}-route
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`{subdomain}.example.com`)
      kind: Rule
      middlewares:
        - name: {app}-{env}-cors
        - name: {app}-{env}-security-headers
        - name: ratelimit-normal
      services:
        - name: {app}-{env}-service
          port: 3000
  tls:
    certResolver: letsencrypt
---
# HTTP redirect
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: {app}-{env}-route-http
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`{subdomain}.example.com`)
      kind: Rule
      middlewares:
        - name: redirect-https
      services:
        - name: {app}-{env}-service
          port: 3000
```

## 5. Naming Conventions

| Resource | Pattern | Example |
|---|---|---|
| Deployment | `{app}-{env}` | `songnai-backend-dev` |
| Service | `{app}-{env}-service` | `songnai-backend-dev-service` |
| ConfigMap | `{app}-{env}-config` | `songnai-backend-dev-config` |
| Secret | `{app}-{env}-secrets` | `songnai-backend-dev-secrets` |
| IngressRoute | `{app}-{env}-route` | `songnai-backend-dev-route` |
| Middleware | `{app}-{env}-cors` | `songnai-backend-dev-cors` |

## 6. Environment Differences

| Setting | Dev | Staging | Production |
|---|---|---|---|
| Replicas | 1 | 1-2 | 2-3 |
| CPU request | 50m | 100m | 300m |
| Memory request | 128Mi | 256Mi | 512Mi |
| Subdomain | `dev.*` | `staging.*` | bare domain |
| PDB | No | No | Yes |
| Security context | Basic | Basic | Full (drop ALL caps) |

## 7. Secrets Rules

- `secrets.yaml` is **gitignored** (`**/secrets.yaml`)
- `secrets_example.yaml` is committed as template
- Use `envFrom: [secretRef]` for bulk injection
- Use `secretKeyRef` for individual keys
- Rotate secrets quarterly
