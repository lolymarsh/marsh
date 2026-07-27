---
trigger: always_on
---

# Error Handling Patterns (Shared)

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

## 2. HTTP Status Mapping

| Error Class | HTTP Status |
|---|---|
| `NotFoundError` | 404 |
| `BadRequestError` | 400 |
| `UnauthorizedError` | 401 |
| `ForbiddenError` | 403 |
| `ConflictError` | 409 |
| `AppError` (generic) | 500 |

## 3. Zod Error Formatting

```typescript
import { ZodError } from 'zod';

export function formatZodError(err: ZodError): string {
  return err.issues.map((i) => `${i.path.join('.')}: ${i.message}`).join(', ');
}
```
