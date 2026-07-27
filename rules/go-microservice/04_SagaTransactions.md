---
trigger: always_on
---

# Saga Transactions

> **Problem**: Business transaction that spans multiple services (different DBs)
> **Solution**: Saga — sequence of local transactions + compensating actions

---

## 1. Saga Pattern Selection

| Pattern | Pros | Cons | When to use |
|---|---|---|---|
| **Choreography** | No coordinator, simple, event-driven | Hard to trace, cyclic events | Few services, simple flows |
| **Orchestration** | Central coordinator, traceable, testable | Single point of failure, more code | Complex flows, many services |

**Recommendation**: Use **Orchestration Saga** — better traceability and fits structured coding style.

---

## 2. Orchestration Saga (Recommended)

### Structure

```
order-service (Orchestrator)
  │
  ├── Step 1: Create PENDING order (local DB)
  ├── Step 2: Call inventory-service → ReserveStock (sync RPC)
  ├── Step 3: Call payment-service → Charge (sync RPC)
  ├── Step 4: Update order → CONFIRMED (local DB)
  │
  │  If Step 3 fails:
  ├── Rollback Step 2: Call inventory-service → ReleaseStock (compensating)
  └── Rollback Step 1: Update order → FAILED
```

### Implementation (Go)

```go
// internal/service/order_saga.go
type CreateOrderSaga struct {
    orderSvc     *OrderService
    inventoryCli *inventory.Client
    paymentCli   *payment.Client
    logger       *zap.Logger
}

func (s *CreateOrderSaga) Execute(ctx context.Context, req *CreateOrderRequest) error {
    // Step 1: Create order (PENDING)
    order, err := s.orderSvc.CreatePending(ctx, req)
    if err != nil {
        return apperrors.Internal("failed to create order", err)
    }

    // Step 2: Reserve stock (sync RPC)
    if err := s.inventoryCli.ReserveStock(ctx, &inventoryv1.ReserveStockRequest{
        OrderId: order.ID, Items: req.Items,
    }); err != nil {
        s.compensate(ctx, order, "inventory")
        return apperrors.Internal("failed to reserve stock", err)
    }

    // Step 3: Charge payment (sync RPC)
    if err := s.paymentCli.Charge(ctx, &paymentv1.ChargeRequest{
        OrderId: order.ID, Amount: order.Total,
    }); err != nil {
        s.compensate(ctx, order, "payment")
        return apperrors.Internal("failed to charge payment", err)
    }

    // Step 4: Confirm order
    if err := s.orderSvc.Confirm(ctx, order.ID); err != nil {
        s.compensate(ctx, order, "confirm")
        return apperrors.Internal("failed to confirm order", err)
    }

    return nil
}

func (s *CreateOrderSaga) compensate(ctx context.Context, order *Order, failedStep string) {
    switch failedStep {
    case "payment":
        s.inventoryCli.ReleaseStock(ctx, &inventoryv1.ReleaseStockRequest{OrderId: order.ID})
        fallthrough
    case "inventory":
        s.orderSvc.MarkFailed(ctx, order.ID)
    case "confirm":
        s.inventoryCli.ReleaseStock(ctx, &inventoryv1.ReleaseStockRequest{OrderId: order.ID})
        s.orderSvc.MarkFailed(ctx, order.ID)
    }
}
```

### With RabbitMQ (async steps)

```go
func (s *CreateOrderSaga) ExecuteAsync(ctx context.Context, req *CreateOrderRequest) error {
    order, err := s.orderSvc.CreatePending(ctx, req)
    if err != nil {
        return err
    }
    return s.eventBus.Publish(ctx, "saga.order.created", &OrderCreatedEvent{
        OrderID: order.ID, Items: req.Items,
    })
}

// Consumer in inventory-service
func (h *OrderConsumer) HandleOrderCreated(ctx context.Context, event *OrderCreatedEvent) error {
    // Reserve stock
    // Success → publish "saga.inventory.reserved"
    // Fail → publish "saga.inventory.failed"
}

// Consumer in order-service listens for responses
func (h *SagaConsumer) HandleInventoryReserved(ctx context.Context, event *InventoryReservedEvent) error {
    // Proceed to next step: publish "saga.payment.charge"
}

func (h *SagaConsumer) HandleInventoryFailed(ctx context.Context, event *InventoryFailedEvent) error {
    // Mark order as FAILED
}
```

---

## 3. Choreography Saga (Event-driven)

```
order-service:     create order → publish "order.created"
inventory-service: consume "order.created" → reserve stock → publish "stock.reserved"
payment-service:   consume "stock.reserved" → charge → publish "payment.charged"
order-service:     consume "payment.charged" → confirm order → publish "order.confirmed"

Rollback:
inventory-service: consume "payment.failed" → release stock → publish "stock.released"
```

**Pros**: No central orchestrator, loosely coupled
**Cons**: Hard to trace, each service needs saga state machine

---

## 4. Outbox Pattern (Data Consistency)

> **Problem**: Local DB transaction + RabbitMQ publish must be atomic
> **Solution**: Outbox table + relay

```sql
-- In the service that publishes events
CREATE TABLE outbox (
    id          UUID PRIMARY KEY,
    aggregate_type VARCHAR(100),
    aggregate_id   VARCHAR(36),
    event_type     VARCHAR(100),
    payload     JSONB,
    created_at  TIMESTAMP,
    published   BOOLEAN DEFAULT FALSE
);
```

```go
// Service layer: insert to outbox in same DB transaction
func (s *OrderService) CreatePending(ctx context.Context, req *CreateOrderRequest) (*Order, error) {
    var order *Order
    err := infrastructure.ExecuteInTransaction(ctx, s.db, func(ctx context.Context, tx bun.Tx) error {
        order, err := s.repo.InsertOrder(ctx, &tx, req)
        if err != nil {
            return apperrors.Internal("failed to insert order", err)
        }

        // Insert outbox event in same tx
        if err := s.repo.InsertOutbox(ctx, &tx, &OutboxEvent{
            AggregateType: "order",
            AggregateID:   order.ID,
            EventType:     "order.created",
            Payload:       order,
        }); err != nil {
            return apperrors.Internal("failed to insert outbox", err)
        }
        return nil
    })
    return order, err
}

// Outbox relay: background job that publishes + marks as published
func (s *OutboxRelay) Start(ctx context.Context) {
    for {
        events, _ := s.repo.GetUnpublishedEvents(ctx, 100)
        for _, event := range events {
            if err := s.eventBus.Publish(ctx, event.EventType, event.Payload); err != nil {
                s.logger.Error("failed to publish event", zap.Error(err))
                continue
            }
            s.repo.MarkPublished(ctx, event.ID)
        }
        time.Sleep(1 * time.Second)
    }
}
```

---

## 5. Anti-patterns

| Anti-pattern | Why not |
|---|---|
| **Distributed transaction (2PC)** | Not suitable for microservices — coordinator is SPOF, blocking |
| **Shared database** | High coupling, no scalability |
| **Synchronous chain** | Service A → B → C → D = latency accumulates, low fault tolerance |
| **Eventual consistency without compensation** | Data corruption on partial failure |
| **Cascading rollback via try-catch** | Rollback logic scattered, hard to test |
