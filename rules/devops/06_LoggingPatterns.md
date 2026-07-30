---
trigger: always_on
---

# Logging Patterns (DevOps)

> App server เขียน log ลง stdout เท่านั้น — DevOps จัดการ collection, storage, rotation, alert ทั้งหมด

## 1. Principle: Logs are Infrastructure Concern

| Concern | Where to handle | Why |
|---|---|---|
| **Log Format** | App (JSON to stdout) | Structured data 便于 query |
| **Log Collection** | DevOps | External tools collect from container stdout |
| **Log Storage** | DevOps | Centralized storage (ELK/Loki/CloudWatch) |
| **Log Rotation** | DevOps | External tool manages retention |
| **Log Search** | DevOps | Kibana/Grafana/CloudWatch Insights |
| **Log Alerting** | DevOps | Alert on error patterns |
| **Log Retention** | DevOps | Policy-based retention |

### ทำไม App ไม่ควรเขียน log ลง file

1. **Container filesystem เป็น ephemeral** — restart แล้ว log หาย
2. **I/O overhead** — เขียน file เพิ่ม latency ให้ app server
3. **Disk space** — container disk เล็ก ไม่เหมาะเก็บ log
4. **Log rotation** — ต้องจัดการเอง เพิ่ม complexity
5. **Centralized** — หลาย container ต้องรวม log ที่เดียว

## 2. App-Side: stdout JSON Format

App server เขียน log ลง stdout เป็น JSON:

```go
// Go slog — stdout JSON handler
handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
})
logger := slog.New(handler)
```

### JSON Log Format

```json
{
  "time": "2026-07-29T10:00:00Z",
  "level": "INFO",
  "msg": "user registered",
  "request_id": "REQ_abc123",
  "user_id": "U_000000000000000001",
  "layer": "auth.service"
}
```

### Required Fields

| Field | Source | Example |
|---|---|---|
| `time` | slog auto | `2026-07-29T10:00:00Z` |
| `level` | slog auto | `INFO`, `ERROR`, `WARN`, `DEBUG` |
| `msg` | slog auto | `"user registered"` |
| `request_id` | Echo middleware | `REQ_abc123` |
| `layer` | Service/handler | `"auth.service"`, `"user.handler"` |

## 3. Docker: Log Driver Configuration

### docker-compose.yml

```yaml
services:
  app:
    image: loly_service:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"    # Max size per log file
        max-file: "3"      # Max number of log files
    # stdout is automatically collected by Docker
```

### Docker Daemon Config (/etc/docker/daemon.json)

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

## 4. Kubernetes: Log Collection

### Fluent Bit (DaemonSet)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  template:
    spec:
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:3.0
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: containers
              mountPath: /var/lib/docker/containers
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: containers
          hostPath:
            path: /var/lib/docker/containers
```

### Fluent Bit Config (parsers + filters)

```ini
[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    Parser            docker
    Tag               kube.*
    Refresh_Interval  5

[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc:443
    Kube_Tag_Prefix     kube.var.log.containers.
    Merge_Log           On
    K8S-Logging.Parser  On
    K8S-Logging.Exclude Off

[OUTPUT]
    Name            es
    Match           *
    Host            elasticsearch.logging.svc
    Port            9200
    Index           app-logs
    Type            _doc
```

## 5. GCP: Cloud Logging

### Cloud Run / GKE

Cloud Logging อัตโนมัติ collects stdout/stderr จาก containers

```yaml
# No config needed — GCP auto-collects from stdout
# Use structured JSON logs for best results
```

### Log Query (Cloud Logging Insights)

```
jsonPayload.layer="auth.service"
jsonPayload.level="ERROR"
jsonPayload.request_id="REQ_abc123"
```

## 6. AWS: CloudWatch Logs

### ECS Task Definition

```json
{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/loly-service",
      "awslogs-region": "ap-southeast-1",
      "awslogs-stream-prefix": "ecs"
    }
  }
}
```

### Log Query (CloudWatch Insights)

```sql
fields @timestamp, @message, level, request_id, layer
| filter level = "ERROR"
| sort @timestamp desc
| limit 100
```

## 7. ELK Stack (Self-Hosted)

### Architecture

```
┌──────────┐     ┌───────────┐     ┌───────────────┐     ┌──────────┐
│   App    │────▶│  Filebeat │────▶│ Elasticsearch │────▶│ Kibana   │
│ (stdout) │     │           │     │               │     │          │
└──────────┘     └───────────┘     └───────────────┘     └──────────┘
```

### Filebeat Config

```yaml
filebeat.inputs:
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'

processors:
  - decode_json_fields:
      fields: ["message"]
      target: "json"
      overwrite_keys: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  indices:
    - index: "app-logs-%{+yyyy.MM.dd}"
```

## 8. Loki + Grafana (Lightweight)

### Architecture

```
┌──────────┐     ┌───────────┐     ┌──────────┐     ┌──────────┐
│   App    │────▶│  Promtail │────▶│   Loki   │────▶│ Grafana  │
│ (stdout) │     │           │     │          │     │          │
└──────────┘     └───────────┘     └──────────┘     └──────────┘
```

### Promtail Config

```yaml
scrape_configs:
  - job_name: containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: containerlogs
          __path__: /var/lib/docker/containers/*/*.log
    pipeline_stages:
      - docker: {}
      - json:
          expressions:
            level: level
            msg: msg
            request_id: request_id
      - labels:
          level:
          request_id:
```

### LogQL Query Examples

```logql
# All errors
{job="containerlogs"} | json | level="ERROR"

# Specific service
{job="containerlogs"} | json | layer="auth.service"

# Search by request ID
{job="containerlogs"} | json | request_id="REQ_abc123"

# Count errors per minute
count_over_time({job="containerlogs"} | json | level="ERROR" [1m])
```

## 9. Log Retention Policy

| Environment | Retention | Storage |
|---|---|---|
| Production | 30 days | Hot storage |
| Production | 90 days | Cold storage (S3/GCS) |
| Staging | 7 days | Hot storage only |
| Development | 3 days | Local only |

## 10. Alerting Rules

### Error Rate Alert

```yaml
# Prometheus AlertManager
groups:
  - name: app-alerts
    rules:
      - alert: HighErrorRate
        expr: sum(rate({job="containerlogs"} | json | level="ERROR" [5m])) > 10
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
```

### Loki Alert (Grafana)

```yaml
# Grafana Alert Rule
expr: |
  sum(count_over_time({job="containerlogs"} | json | level="ERROR" [5m])) > 10
for: 2m
labels:
  severity: critical
annotations:
  summary: "High error rate in {{ $labels.job }}"
```

## 11. Checklist for DevOps

- [ ] App writes logs to stdout in JSON format
- [ ] Log collector deployed (Fluent Bit/Promtail/Filebeat)
- [ ] Log storage configured (ELK/Loki/CloudWatch)
- [ ] Log retention policy set
- [ ] Log search/dashboard created (Kibana/Grafana/CloudWatch Insights)
- [ ] Error rate alerts configured
- [ ] Request ID propagated in all logs
- [ ] Log format tested and parseable
