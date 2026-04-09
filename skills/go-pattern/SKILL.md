---
name: go-pattern
description: >
  Go implementation patterns and idioms. Use this skill whenever writing Go code,
  implementing features, creating constructors, structuring packages, or following
  Go best practices. Use when the user mentions "implement", "build", "code",
  "write", "constructor", "dependency injection", or "functional options".
---

# Go Implementation Patterns

## Constructor Pattern
Always use the `NewX` constructor pattern:

```go
type Server struct {
    addr    string
    handler http.Handler
    logger  *slog.Logger
    timeout time.Duration
}

func NewServer(addr string, handler http.Handler, opts ...ServerOption) (*Server, error) {
    if addr == "" {
        return nil, fmt.Errorf("NewServer: %w", ErrEmptyAddr)
    }
    s := &Server{
        addr:    addr,
        handler: handler,
        logger:  slog.Default(),
        timeout: 30 * time.Second,
    }
    for _, opt := range opts {
        opt(s)
    }
    return s, nil
}
```

## Functional Options
```go
type ServerOption func(*Server)

func WithLogger(l *slog.Logger) ServerOption {
    return func(s *Server) { s.logger = l }
}

func WithTimeout(d time.Duration) ServerOption {
    return func(s *Server) { s.timeout = d }
}
```

## Interface Compliance Check
```go
var _ io.ReadCloser = (*myReader)(nil)
```
Place at package level to get compile-time interface checks.

## Graceful Shutdown
```go
func (s *Server) Run(ctx context.Context) error {
    srv := &http.Server{Addr: s.addr, Handler: s.handler}

    errCh := make(chan error, 1)
    go func() { errCh <- srv.ListenAndServe() }()

    select {
    case err := <-errCh:
        return fmt.Errorf("server exited: %w", err)
    case <-ctx.Done():
        shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        return srv.Shutdown(shutdownCtx)
    }
}
```

## Repository / Store Pattern
```go
type UserStore interface {
    Get(ctx context.Context, id string) (*User, error)
    List(ctx context.Context, filter UserFilter) ([]User, error)
    Create(ctx context.Context, u *User) error
    Update(ctx context.Context, u *User) error
    Delete(ctx context.Context, id string) error
}
```
- Interface at consumption site
- Context is always first param
- Return concrete errors, not generic ones

## Middleware Pattern (HTTP)
```go
func LoggingMiddleware(logger *slog.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            next.ServeHTTP(w, r)
            logger.Info("request",
                "method", r.Method,
                "path", r.URL.Path,
                "duration", time.Since(start),
            )
        })
    }
}
```

## Package Organization Rules
1. `cmd/` — main packages only, thin wiring
2. `internal/` — private packages, business logic
3. `pkg/` — public API (use sparingly)
4. One package per domain concept
5. No circular imports (if needed, introduce an interface package)

## Naming Conventions
- Packages: `userstore`, not `user_store` or `userStore`
- Interfaces: name after behavior (`Reader`, `Processor`), not `IReader`
- Receivers: 1-2 letters, first letter of type (`func (s *Server)`)
- Exported: doc comment required, starts with the name (`// Server handles...`)
- Unexported: no doc comment needed unless complex
- Acronyms: `HTTPServer`, `userID`, not `HttpServer`, `userId`