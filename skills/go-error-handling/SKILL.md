---
name: go-error-handling
description: >
  Go error handling patterns and best practices. Use this skill whenever writing
  error handling code, wrapping errors, creating sentinel errors, implementing
  custom error types, or reviewing error handling. Use when the user mentions
  "error", "error handling", "sentinel", "wrap", or "unwrap".
---

# Go Error Handling Patterns

## The Three Error Categories

### 1. Sentinel Errors — caller checks identity
```go
var (
    ErrNotFound    = errors.New("not found")
    ErrConflict    = errors.New("conflict")
    ErrUnauthorized = errors.New("unauthorized")
)

// Caller checks with errors.Is:
if errors.Is(err, ErrNotFound) {
    // handle not found
}
```
Use when: the caller needs to branch on error identity.

### 2. Custom Error Types — caller needs structured data
```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation: %s: %s", e.Field, e.Message)
}

// Caller checks with errors.As:
var ve *ValidationError
if errors.As(err, &ve) {
    // access ve.Field, ve.Message
}
```
Use when: the caller needs more than just identity.

### 3. Opaque Errors — caller only checks nil/non-nil
```go
return fmt.Errorf("parsing config: %w", err)
```
Use when: the caller doesn't need to inspect the error type.

## Wrapping Rules

### Always wrap with context
```go
// GOOD — adds context about what operation failed
func (s *Server) loadConfig(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return fmt.Errorf("loadConfig %s: %w", path, err)
    }
    return nil
}

// BAD — loses context
func (s *Server) loadConfig(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return err  // where did this come from?
    }
    return nil
}
```

### Wrapping format convention
```
functionName: %w           // simple case
functionName [detail]: %w  // when extra context helps
```

### When NOT to wrap
- At package boundaries where you want to hide internals:
  ```go
  return fmt.Errorf("store: %v", err)  // %v breaks the chain intentionally
  ```

## Error Handling Anti-Patterns

### Don't ignore errors
```go
// BAD
json.Unmarshal(data, &v)

// GOOD
if err := json.Unmarshal(data, &v); err != nil {
    return fmt.Errorf("unmarshal: %w", err)
}
```

### Don't panic in library code
```go
// BAD
func Parse(s string) Config {
    c, err := parse(s)
    if err != nil {
        panic(err)  // caller can't recover gracefully
    }
    return c
}

// GOOD
func Parse(s string) (Config, error) {
    c, err := parse(s)
    if err != nil {
        return Config{}, fmt.Errorf("Parse: %w", err)
    }
    return c, nil
}
```

### Don't over-wrap
```go
// BAD — redundant wrapping
if err != nil {
    return fmt.Errorf("failed to process: error processing item: %w", err)
}

// GOOD — single clear context
if err != nil {
    return fmt.Errorf("processItem %s: %w", item.ID, err)
}
```

## HTTP Error Mapping
```go
func httpStatusFromError(err error) int {
    switch {
    case errors.Is(err, ErrNotFound):
        return http.StatusNotFound
    case errors.Is(err, ErrConflict):
        return http.StatusConflict
    case errors.Is(err, ErrUnauthorized):
        return http.StatusUnauthorized
    default:
        var ve *ValidationError
        if errors.As(err, &ve) {
            return http.StatusBadRequest
        }
        return http.StatusInternalServerError
    }
}
```