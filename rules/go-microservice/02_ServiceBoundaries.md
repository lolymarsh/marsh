---
trigger: always_on
---

# Service Boundaries & Contracts

---

## 1. How to Split Services

Split by **business domain** + **data ownership**:

```
❌ Bad: order-service + order-item-service (split because of data size)
✅ Good: order-service + inventory-service (different domain, different DB)

❌ Bad: user-service + user-auth-service (same domain)
✅ Good: user-service (auth + profile + role together)
```

**Rule of thumb**: If 2 modules read/write the same tables → they belong in the same service.

---

## 2. Proto Contracts

Proto definitions live in the **owner service's repo**. Other services import them:

```
inventory-service/
├── proto/inventory/v1/
│   ├── inventory.proto   # InventoryService RPCs + messages
│   └── stock.proto       # Stock-related messages
│
order-service/
├── proto/order/v1/
│   └── order.proto       # imports inventory/v1/stock.proto
│
└── proto/buf.yaml        # dependency → inventory-service proto
```

**Versioning**: All changes must be backward-compatible — use `buf breaking` to enforce.

---

## 3. Service-to-Service Auth

- **mTLS** (production) — certificate-based, service mesh level
- **JWT with service role** (dev/staging) — service account JWT
- Never use plaintext API keys or shared secrets

```
Service A → mTLS → Service B
Service A → JWT (service-role) → API Gateway → Service B
```

---

## 4. Resilience Patterns

| Pattern | Implementation | When |
|---|---|---|
| **Retry with backoff** | `github.com/cenkalti/backoff/v4` | Transient failures |
| **Circuit breaker** | `github.com/sony/gobreaker` | Repeated failures, protect downstream |
| **Timeout** | Context timeout per RPC call | Always |
| **Bulkhead** | Goroutine pool per downstream | Isolate failures |
| **Fallback** | Cache / default response | Non-critical reads |

```go
func (c *InventoryClient) ReserveStock(ctx context.Context, req *inventoryv1.ReserveStockRequest) error {
    return backoff.Retry(func() error {
        _, err := c.client.ReserveStock(ctx, connect.NewRequest(req))
        if err != nil {
            if connect.CodeOf(err) == connect.CodeUnavailable {
                return err // retry
            }
            return backoff.Permanent(err) // don't retry
        }
        return nil
    }, backoff.NewExponentialBackOff())
}
```

---

## 5. Shared Libraries

| Share | Don't Share |
|---|---|
| `apperrors` | Entity structs (copy per service) |
| `infrastructure` (DB/Redis init) | Repository implementations |
| `logger`, `util` | Business logic |
| Proto definitions | Internal DTOs |
