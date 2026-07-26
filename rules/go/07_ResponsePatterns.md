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
        "page_size": 10,
        "total_data": 42,
        "total_page": 5,
        "has_next_page": true,
        "has_previous_page": false
    }
}
```

## 3. Error Response

```json
{
    "code": 404,
    "message": "user not found"
}
```

## 4. Conflict Response (version mismatch)

```json
{
    "code": 409,
    "message": "version mismatch",
    "data": { ... }
}
```

## 5. HTTP Status Mapping

| Error Code | HTTP Status | Description |
|---|---|---|
| `NOT_FOUND` | 404 | Resource not found |
| `ALREADY_EXISTS` | 409 | Duplicate resource |
| `BAD_REQUEST` | 400 | Invalid input |
| `UNAUTHORIZED` | 401 | Not authenticated |
| `FORBIDDEN` | 403 | Not authorized |
| `INACTIVE` | 403 | Account inactive |
| `INTERNAL` | 500 | Server error |
| `CONFLICT` | 409 | Version mismatch |

## 6. Response Helpers

```go
// Echo
response.HandleSuccess(c, http.StatusOK, "success", map[string]any{"data": result})
response.HandleError(c, err)  // auto-maps apperror code to HTTP status

// ConnectRPC
connectutil.FromAppError(err)  // converts apperror to connect.Error
```
