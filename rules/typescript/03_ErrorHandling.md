---
trigger: always_on
---

# Error Handling Patterns

## 1. Error Hierarchy

```typescript
export class AppError extends Error {
  constructor(public statusCode: number, message: string, public details?: any) {
    super(message);
  }
}

export class NotFoundError extends AppError {
  constructor(message = 'Resource not found') { super(404, message); }
}

export class ConflictError extends AppError {
  constructor(message = 'Data modified by another user, please retry') { super(409, message); }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Please login') { super(401, message); }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Access denied') { super(403, message); }
}

export class BadRequestError extends AppError {
  constructor(message = 'Invalid input') { super(400, message); }
}
```

## 2. Per-Layer Rules

| Layer | Error Type |
|---|---|
| **Handler** | Catch errors → `SendError(res, status, message)` |
| **Service** | Throw `AppError` subclass |
| **Repository** | Return `null` (not throw) |
| **Transaction** | Throw `AppError` subclass (auto-rollback) |

## 3. Repository → Service Flow

```
Repository.FindById → returns null        → Service throws NotFoundError
Repository.Update   → returns null        → Service throws ConflictError
Repository.FindById → returns entity      → Service continues
```

## 4. Handler Error Handling

```typescript
try {
  const input = schema.parse(req.body);
  const result = await this.svc.Method(input);
  SendSuccess(res, 200, 'success', { data: result });
} catch (err: unknown) {
  if (err instanceof AppError) {
    SendError(res, err.statusCode, err.message);  // Known: map status from error
    return;
  }
  if (err instanceof ZodError) {
    SendError(res, 400, formatZodError(err));      // Validation: always 400
    return;
  }
  logger.error({ err }, 'Method failed');           // Unexpected: 500 + log
  SendError(res, 500, 'Internal server error');
}
```

## 5. Audit Errors

Audit failures are fire-and-forget (never fail the main operation):

```typescript
setImmediate(async () => {
  try {
    await this.repo.Insert(doc);
  } catch (err: unknown) {
    logger.error({ err }, 'Failed to insert audit log');
  }
});
```
