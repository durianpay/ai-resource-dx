---
name: spec-writing
description: >
  Write structured technical specs for Go projects. Use this skill whenever
  designing a new feature, planning a refactor, defining interfaces, or creating
  architecture documents. Also use when the user mentions "spec", "design doc",
  "architecture", "planning", or "interface design".
---

# Spec Writing for Go Projects

## Purpose
Produce clear, implementable specs that a developer agent can execute without ambiguity.

## Spec Structure

Every spec MUST include these sections in order:

### 1. Overview
- What are we building and why?
- What problem does this solve?
- One paragraph max.

### 2. Package Layout
```
internal/
  feature/
    feature.go       // Core types and interfaces
    feature_impl.go  // Implementation
    option.go        // Functional options (if needed)
pkg/
  featureapi/        // Public API surface (if needed)
```
- Justify every new package
- Prefer adding to existing packages when the feature is small

### 3. Interface Definitions
Write actual Go code blocks:
```go
// Processor handles incoming events.
type Processor interface {
    // Process handles a single event. Returns ErrInvalidEvent if
    // the event cannot be processed.
    Process(ctx context.Context, event Event) error
}
```
Rules:
- Interfaces at the consumption site, not the declaration site
- 1-3 methods max per interface (Go idiom: small interfaces)
- Document the contract, not the implementation
- Specify which errors each method can return

### 4. Type Definitions
```go
// Event represents an incoming message from the queue.
type Event struct {
    ID        string    `json:"id"`
    Payload   []byte    `json:"payload"`
    CreatedAt time.Time `json:"created_at"`
}
```
- Use value types for small immutable data
- Use pointer types for large or mutable data
- Always include JSON/DB tags if the type is serialized

### 5. Constructor & Function Signatures
```go
// NewProcessor creates a Processor with the given options.
// Returns an error if required options are missing.
func NewProcessor(opts ...Option) (*processor, error)
```
- Always return `error` from constructors
- Use functional options for configurable types
- Context as first param for anything that does I/O

### 6. Error Strategy
```go
var (
    ErrInvalidEvent = errors.New("invalid event")
    ErrTimeout      = errors.New("operation timed out")
)
```
- Define sentinel errors for conditions callers need to check
- Use `fmt.Errorf("context: %w", err)` for wrapping
- Document which functions return which sentinel errors

### 7. Dependencies
- List external packages with versions
- List internal packages being imported
- Flag any new dependencies that need team approval

### 8. Constraints & Non-Goals
- Performance targets (latency, throughput)
- Backward compatibility requirements
- What this feature explicitly does NOT do

### 9. Open Questions
- Numbered list of decisions requiring human input
- Include your recommendation for each

## Quality Checklist
Before finalizing a spec:
- [ ] Can a developer implement this without asking clarifying questions?
- [ ] Are all interfaces small enough to mock easily?
- [ ] Are error paths explicitly defined?
- [ ] Is the package layout justified?
- [ ] Are there no implementation details leaking into the spec?