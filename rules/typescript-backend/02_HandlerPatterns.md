---
trigger: always_on
---

# Handler Layer Patterns (Backend)

## 1. Handler Class

```typescript
export class UserHandler {
  constructor(private svc: IUserService) {}

  Login = async (req: Request, res: Response): Promise<void> => {
    try {
      // 1. Validate input with Zod
      const input = loginSchema.parse(req.body);

      // 2. Call service
      const result = await this.svc.Login(input);

      // 3. Send success response
      SendSuccess(res, 200, 'success', { data: result });
    } catch (err: unknown) {
      // Known app errors
      if (err instanceof AppError) {
        SendError(res, err.statusCode, err.message);
        return;
      }
      // Validation errors
      if (err instanceof ZodError) {
        SendError(res, 400, formatZodError(err));
        return;
      }
      // Unexpected errors
      logger.error({ err }, 'Login failed');
      SendError(res, 500, 'Internal server error');
    }
  };
}
```

## 2. Standard Flow

```
1. Parse input (Zod schema.parse)
2. Validate (auto from Zod)
3. Call service method
4. Send response (SendSuccess / SendError)
```

## 3. Error Handling Order

```
1. AppError (and subclasses) → map status code from error
2. ZodError → 400 Bad Request
3. Unknown → 500 Internal Server Error + log
```

## 4. Filter Handler

```typescript
FindFiltered = async (req: Request, res: Response): Promise<void> => {
  try {
    const input = filterSchema.parse(req.body);
    const result = await this.svc.FindFiltered(input, req.user!.userId, req.auditMeta);
    SendSuccess(res, 200, 'success', result);
  } catch (err: unknown) {
    if (err instanceof AppError) {
      SendError(res, err.statusCode, err.message);
      return;
    }
    if (err instanceof ZodError) {
      SendError(res, 400, formatZodError(err));
      return;
    }
    logger.error({ err }, 'Filter failed');
    SendError(res, 500, 'Internal server error');
  }
};
```

## 5. Route Registration

```typescript
export function RegisterUserRoutes(handler: UserHandler, auth: AuthMiddleware): Router {
  const router = Router();

  router.post('/auth/login', handler.Login);
  router.get('/users', auth('ADMIN'), handler.FindFiltered);
  router.patch('/users/:id', auth('ADMIN'), handler.Update);

  return router;
}
```

## 6. Middleware Pattern

```typescript
// Curried auth middleware
export type AuthMiddleware = (allowedRoles?: string[]) => RequestHandler;

export function CreateAuthMiddleware(redis: Redis): AuthMiddleware {
  return (allowedRoles?: string[]) => async (req, res, next) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    if (!token) return SendError(res, 401, 'Unauthorized');

    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    req.user = payload;

    if (allowedRoles && !allowedRoles.includes(payload.role)) {
      return SendError(res, 403, 'Forbidden');
    }
    next();
  };
}
```
