# Effective Go Compliance Checklist

Use this checklist when reviewing `.go` files in a PR. Flag violations with the specific rule and suggest the idiomatic fix. Reference: https://go.dev/doc/effective_go

## Formatting

- Code should be formatted with `gofmt`. If it's not, flag it as the first item.
- Indentation uses tabs, not spaces.
- No unnecessary parentheses in control structures (`if`, `for`, `switch`).
- Opening brace of control structures must be on the same line as the statement, never on the next line.

## Naming

### Package Names
- Package names should be lowercase, single-word, no underscores, no mixedCaps.
- Package names should be short, concise, and evocative.
- Exported names should not stutter with the package name (e.g., `bufio.Reader` not `bufio.BufReader`).
- A constructor for the sole exported type may be called `New` (e.g., `ring.New`), not `NewRing`.

### Getters and Setters
- Getter methods should NOT have `Get` prefix. A field `owner` → getter `Owner()`, not `GetOwner()`.
- Setter methods should use `Set` prefix: `SetOwner()`.

### Interface Names
- One-method interfaces use the method name + `-er` suffix: `Reader`, `Writer`, `Formatter`, `Stringer`.
- Don't name a method `Read`, `Write`, `Close`, `String`, etc. unless it has the canonical signature and meaning.

### General Naming
- Use `MixedCaps` or `mixedCaps`, never underscores for multiword names.
- Acronyms should be all caps: `URL`, `HTTP`, `ID` — not `Url`, `Http`, `Id`.

## Semicolons

- No explicit semicolons except in `for` loop clauses or multi-statement lines.
- Opening braces must not be on a new line (the lexer inserts semicolons that break things).

## Control Structures

### If
- Omit `else` when the `if` body ends with `break`, `continue`, `goto`, or `return`.
- Use initialization statements in `if` where appropriate: `if err := f(); err != nil { ... }`.
- Error cases should return early; the happy path flows down the page.

### For
- Use `range` for iterating over slices, maps, strings, and channels.
- Use blank identifier `_` to discard unwanted loop variables.
- Parallel assignment for multiple loop variables (no comma operator).

### Switch
- Prefer `switch` over `if-else-if-else` chains.
- No automatic fall through — cases can be comma-separated.
- Use labeled breaks to exit surrounding loops from within a switch.
- Use type switches for interface type discovery with `.(type)`.

## Functions

### Multiple Return Values
- Use multiple return values instead of in-band error signaling or pointer parameters.
- Common pattern: `(result, error)`.

### Named Return Values
- Use named returns when they clarify what is being returned, especially for documentation.
- Avoid naked returns in long functions — they reduce readability.

### Defer
- Use `defer` for resource cleanup (closing files, unlocking mutexes).
- Place `defer` immediately after the resource is acquired.
- Remember: deferred function arguments are evaluated at the `defer` statement, not at function exit.
- Deferred calls execute in LIFO order.

## Data

### new vs make
- `new(T)` allocates zeroed memory and returns `*T`. Use for value types.
- `make(T, args)` initializes (not just zeros) slices, maps, and channels. Returns `T`, not `*T`.
- Don't confuse the two — `make` is only for slices, maps, and channels.

### Composite Literals
- Use composite literals to create instances: `&File{fd: fd, name: name}`.
- Field names in composite literals are optional if all fields are provided in order (but named fields are clearer).

### Arrays and Slices
- Slices are the common case; arrays are rarely used directly.
- Use `append` for growing slices.
- Be mindful that slices reference underlying arrays — large underlying arrays can be kept alive by small slices.

### Maps
- Use comma-ok idiom: `val, ok := m[key]` to distinguish missing keys from zero values.
- Maps are not safe for concurrent use — protect with `sync.Mutex` or `sync.RWMutex` or use `sync.Map`.

## Initialization

- Use `init()` functions sparingly, only for setup that cannot be expressed as declarations.
- Package-level variables should use simple declarations or composite literals where possible.

## Methods

### Pointer vs Value Receivers
- Methods that modify the receiver must use pointer receivers.
- Value receivers are safe for concurrent use; pointer receivers are not.
- Be consistent: if some methods have pointer receivers, all should (for interface satisfaction).

## Interfaces

- Keep interfaces small — one or two methods is ideal.
- Accept interfaces, return structs.
- Don't define interfaces preemptively — let them emerge from usage.

## Errors

### Error Handling
- Always check errors. Never ignore returned errors with `_`.
- Handle errors with early returns: check error → return → continue happy path.
- Don't use `panic` for normal error handling.

### Error Strings
- Error strings should not be capitalized (unless beginning with proper nouns/acronyms) and should not end with punctuation.
- Use `fmt.Errorf("something failed: %w", err)` to wrap errors for context.

### Custom Error Types
- Implement the `error` interface for domain-specific errors.
- Use `errors.Is` and `errors.As` for error checking (not type assertions).

## Concurrency

### Goroutines
- Don't leak goroutines — ensure every goroutine has a clear exit path.
- Use `context.Context` for cancellation and timeouts.

### Channels
- Prefer communicating via channels over sharing memory with mutexes.
- The sender should close the channel, never the receiver.
- Use buffered channels only when you have a specific reason.

### sync Package
- Use `sync.WaitGroup` for waiting on multiple goroutines.
- Use `sync.Once` for one-time initialization.
- Protect shared state with `sync.Mutex` or `sync.RWMutex` when channels aren't appropriate.

## Embedding
- Use embedding for composition, not inheritance simulation.
- Be aware that embedding promotes methods — this can cause unintended interface satisfaction.
- Embedding a type that has a `String()` method will change how your type prints.

## Blank Identifier
- Use `_` to ignore unwanted return values.
- Use `var _ Interface = (*Type)(nil)` as a compile-time interface check.
- Use blank imports `import _ "pkg"` only for side effects (e.g., database drivers, image codecs).
