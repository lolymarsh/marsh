---
trigger: always_on
---

# Go Handler Layer Patterns (REST API)

## 1. Handler Struct + Constructor

```go
type Handler struct {
    svc      Service
    validate *validator.Validate
    logger   *zap.Logger
}

func NewHandler(svc Service, validate *validator.Validate, logger *zap.Logger) *Handler {
    return &Handler{svc: svc, validate: validate, logger: logger}
}
```

## 2. Standard Handler Flow (Echo)

```go
func (h *Handler) GetProfile(c echo.Context) error {
    // 1. Bind request (if POST/PUT/PATCH)
    req := &RequestType{}
    if err := c.Bind(req); err != nil {
        return response.HandleError(c, err, http.StatusBadRequest)
    }

    // 2. Extract user context (if authenticated)
    userID := common.ExtractUserID(c)

    // 3. Validate request
    if err := h.validate.StructCtx(c.Request().Context(), req); err != nil {
        return response.HandleError(c, err, http.StatusBadRequest)
    }

    // 4. Call service
    result, err := h.svc.Method(ctx, req)
    if err != nil {
        h.logger.Error("message", zap.Error(err))
        return response.HandleError(c, err)
    }

    // 5. Return response
    return response.HandleSuccess(c, http.StatusOK, "success", map[string]any{"data": result})
}
```

## 3. ConnectRPC Handler

```go
func (h *UserConnectHandler) GetProfile(
    ctx context.Context,
    req *connect.Request[userv1.GetProfileRequest],
) (*connect.Response[userv1.GetProfileResponse], error) {
    uc := connectutil.GetUserContext(ctx)

    resp, err := h.svc.GetProfile(ctx, uc.UserID, uc.Role)
    if err != nil {
        return nil, connectutil.FromAppError(err)
    }
    return connect.NewResponse(&userv1.GetProfileResponse{User: mapToProto(resp)}), nil
}
```

## 4. Route Registration

```go
func RegisterRoutes(app *echo.Echo, h *Handler, mid *middleware.Middleware) {
    g := app.Group("/api/module", mid.IsHaveTokenMiddleware())
    {
        g.GET("/:id", h.GetByID)
        g.POST("/filter", h.Filter)
    }
}
```

## 5. Swagger Annotations

```go
// @Summary Get user profile
// @Description Get current user's profile
// @Tags User
// @Security BearerAuth
// @Success 200 {object} response.SuccessResponse
// @Router /api/user/profile [GET]
func (h *Handler) GetProfile(c echo.Context) error {
```

## 6. ConnectRPC Error Mapping

```go
func FromAppError(err error) error {
    var appErr *apperrors.AppError
    if !errors.As(err, &appErr) {
        return connect.NewError(connect.CodeInternal, err)
    }
    switch appErr.Code {
    case "NOT_FOUND":
        return connect.NewError(connect.CodeNotFound, errors.New(appErr.Message))
    case "ALREADY_EXISTS":
        return connect.NewError(connect.CodeAlreadyExists, errors.New(appErr.Message))
    // ...
    }
}
```
