---
trigger: always_on
---

# Observability

---

## 1. Structured Logging (Zap)

- JSON format with `service` field
- Trace ID propagation across services

```go
logger := zap.NewProduction().With(
    zap.String("service", "order-service"),
    zap.String("trace_id", traceID),
)
```

---

## 2. Distributed Tracing

- **OpenTelemetry** — standard tracing protocol
- Trace ID passed via header: `traceparent`
- Each service creates its own span

```go
func (c *InventoryClient) ReserveStock(ctx context.Context, req *inventoryv1.ReserveStockRequest) error {
    ctx, span := tracer.Start(ctx, "inventory.ReserveStock")
    defer span.End()

    // span propagates via HTTP header to inventory-service
    resp, err := c.client.ReserveStock(ctx, connect.NewRequest(req))
    if err != nil {
        span.RecordError(err)
        return err
    }
    return nil
}
```

---

## 3. Health Checks

Every service must expose:

```
GET /healthz    → OK (liveness)
GET /readyz     → OK (readiness — checks DB, Redis, RabbitMQ connection)
```

```go
func HealthHandler(c echo.Context) error {
    if err := s.db.Ping(); err != nil {
        return c.String(http.StatusServiceUnavailable, "db: unavailable")
    }
    if err := s.redis.Ping(); err != nil {
        return c.String(http.StatusServiceUnavailable, "redis: unavailable")
    }
    return c.String(http.StatusOK, "ok")
}
```

---

## 4. Metrics

- Prometheus metrics endpoint: `GET /metrics`
- Track: request count, latency, error rate, goroutine count
- One Grafana dashboard per service

---

## 5. Centralized Logging

- **Loki** — log aggregation (Grafana Stack)
- Each service sends JSON logs to stdout → Promtail → Loki
- Search with `{service="order-service"} |= "error"`
