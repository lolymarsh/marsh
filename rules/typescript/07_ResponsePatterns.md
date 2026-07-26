---
trigger: always_on
---

# Response Patterns

## 1. Success Response

```json
{
    "code": 200,
    "message": "success",
    "data": { ... }
}
```

## 2. Paginated Response

```json
{
    "code": 200,
    "message": "success",
    "data": [ ... ],
    "pagination": {
        "page": 1,
        "pageSize": 20,
        "totalData": 100,
        "totalPage": 5,
        "hasNextPage": true,
        "hasPreviousPage": false
    }
}
```

## 3. Error Response

```json
{
    "code": 404,
    "message": "Resource not found"
}
```

## 4. Conflict Response (version mismatch)

```json
{
    "code": 409,
    "message": "Data modified by another user, please retry"
}
```

## 5. HTTP Status Mapping

| Error Class | HTTP Status |
|---|---|
| `NotFoundError` | 404 |
| `BadRequestError` | 400 |
| `UnauthorizedError` | 401 |
| `ForbiddenError` | 403 |
| `ConflictError` | 409 |
| `AppError` (generic) | 500 |

## 6. Response Helpers

```typescript
// Success
SendSuccess(res, 200, 'success', { data: result });
SendSuccess(res, 201, 'created', { data: newResource });

// Error
SendError(res, 404, 'Resource not found');
SendError(res, 409, 'Data modified by another user');

// Pagination helper
calculatePagination(page, pageSize, total)
// → { page, pageSize, totalData, totalPage, hasNextPage, hasPreviousPage }
```
