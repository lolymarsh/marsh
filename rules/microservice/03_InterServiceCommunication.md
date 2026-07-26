---
trigger: always_on
---

# Inter-Service Communication

---

## 1. Sync: ConnectRPC (Recommended)

ConnectRPC is the primary choice for sync communication — protocol buffer, code-gen, HTTP/2:

```go
// internal/client/inventory_client.go
type InventoryClient struct {
    client inventoryv1connect.InventoryServiceClient
}

func NewInventoryClient(addr string) *InventoryClient {
    return &InventoryClient{
        client: inventoryv1connect.NewInventoryServiceClient(
            http.DefaultClient, addr,
        ),
    }
}

func (c *InventoryClient) ReserveStock(ctx context.Context, req *inventoryv1.ReserveStockRequest) error {
    resp, err := c.client.ReserveStock(ctx, connect.NewRequest(req))
    if err != nil {
        return connectutil.FromAppError(err)
    }
    if !resp.Msg.Success {
        return apperrors.BadRequest("insufficient stock", nil)
    }
    return nil
}
```

**DI in service layer:**

```go
type OrderService struct {
    repo         OrderRepository
    inventoryCli *InventoryClient
    paymentCli   *PaymentClient
}
```

---

## 2. Async: RabbitMQ Events

Use RabbitMQ for event-driven communication (saga, notification, audit):

```go
// internal/eventbus/publisher.go
type Publisher struct {
    ch *amqp.Channel
}

func (p *Publisher) Publish(ctx context.Context, exchange, key string, event any) error {
    body, _ := json.Marshal(event)
    return p.ch.PublishWithContext(ctx, exchange, key, false, false, amqp.Publishing{
        ContentType: "application/json",
        Body:       body,
    })
}
```

**Event naming convention** — past tense, domain-specific:
```
order.created
order.confirmed
order.cancelled
inventory.stock.reserved
inventory.stock.released
payment.charged
payment.refunded
```

---

## 3. Async: Consumer Pattern

```go
// internal/consumer/order_consumer.go
type OrderConsumer struct {
    inventorySvc *inventory.Service
    logger       *zap.Logger
}

func (c *OrderConsumer) HandleOrderCreated(ctx context.Context, event *OrderCreatedEvent) error {
    if err := c.inventorySvc.Reserve(ctx, event.Items); err != nil {
        c.logger.Error("failed to reserve stock", zap.Error(err))
        return err // nack → re-queue
    }
    return nil // ack
}

func RegisterConsumers(mq *infrastructure.RabbitMQClient, c *OrderConsumer) {
    mq.Consume("saga.order.created", func(msg amqp.Delivery) {
        var event OrderCreatedEvent
        json.Unmarshal(msg.Body, &event)
        if err := c.HandleOrderCreated(context.Background(), &event); err != nil {
            msg.Nack(false, true) // re-queue
            return
        }
        msg.Ack(false)
    })
}
```

---

## 4. Service Discovery

| Environment | Method |
|---|---|
| **Docker Compose** | Service name as hostname (`http://inventory-service:8000`) |
| **Kubernetes** | DNS-based (`inventory-service.namespace.svc.cluster.local`) |
| **Production** | Service mesh (Istio/Linkerd) |

```go
func NewInventoryClient(conf *configs.Config) *InventoryClient {
    addr := fmt.Sprintf("http://%s:%d", conf.InventoryService.Host, conf.InventoryService.Port)
    return &InventoryClient{
        client: inventoryv1connect.NewInventoryServiceClient(http.DefaultClient, addr),
    }
}
```

---

## 5. CQRS Pattern (Read Model)

For use cases that need to query data across services without sync calls every time:

```
order-service (write DB) ──event──→ report-service (read DB)
customer-service (write DB) ──event──→ report-service (read DB)

report-service has materialized views for dashboard/report purposes
```

No cross-service joins — denormalize data in the read model instead.
