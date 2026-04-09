# DX AI Resource

Claude Code agents, skills, and commands for Go development workflows. Provides a structured, multi-agent development pipeline with specialized Go knowledge baked into every step.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed and configured
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- [Go](https://go.dev/dl/) toolchain installed
- [golangci-lint](https://golangci-lint.run/welcome/install-local/) installed (for linting skill)
- Linear MCP server connected in Claude Code (for ticket lookups in PR reviews and architect)

## Installation

Copy or symlink the contents of this repo into your user-level Claude Code configuration at `~/.claude/`:

```bash
# Clone the repo
git clone <repo-url> ~/dx-ai-resource

# Symlink agents, skills, and commands into ~/.claude/
ln -s ~/dx-ai-resource/agents ~/.claude/agents
ln -s ~/dx-ai-resource/skills ~/.claude/skills
ln -s ~/dx-ai-resource/command ~/.claude/command
```

This makes the agents, skills, and commands available globally across all your projects.

Resulting structure:

```
~/.claude/
  agents/
    go-architect.md
    go-developer.md
    go-reviewer.md
  command/
    workflow.md
  skills/
    go-error-handling/
    go-pattern/
    golang-code-style/
    golang-concurrency/
    golang-data-structures/
    golang-design-patterns/
    golang-linter/
    golang-performance/
    golang-structs-interfaces/
    golang-testing/
    pr-reviewer/
    spec-writing/
```

## Quick Start

The fastest way to use this project is the `/workflow` command, which runs the full pipeline:

```
> /workflow Implement a user authentication service
```

This automatically runs: **Architect** -> **Developer** -> **Reviewer** (with fix loops if needed).

---

## Agents

Agents are specialized sub-agents that Claude Code spawns for specific roles. Each has its own tools, model, and skills.

### go-architect

| Property | Value |
|----------|-------|
| Model | Opus |
| Role | Plan and design |
| Output | `docs/specs/{task-slug}.md` |

**When to use:** Before any new feature, refactor, or significant code change.

**What it does:**
- Pulls Linear tickets for requirements (if given a ticket number)
- Analyzes requirements and breaks them into scoped tasks
- Defines package structure and boundaries
- Designs interfaces at the consumption site (Go idiom)
- Specifies error handling strategy per component
- Writes a structured spec to `docs/specs/`

**Skills loaded:** `spec-writing`, `go-pattern`, `go-error-handling`, `golang-design-patterns`

**Rules:** Never writes implementation code. Never modifies source files. Read-only.

---

### go-developer

| Property | Value |
|----------|-------|
| Model | Opus |
| Role | Implement and test |
| Output | Source code + `docs/test/{task-slug}.md` |

**When to use:** After the architect has produced a spec.

**What it does:**
1. Reads the spec from `docs/specs/`
2. Implements code following spec interfaces and signatures exactly
3. Writes comprehensive unit tests (table-driven, with `testify`)
4. Runs `go build`, `go vet`, `go test`, `go fmt`
5. Reports any deviations from spec

**Skills loaded:** `golang-data-structures`, `golang-performance`, `golang-design-patterns`, `golang-linter`, `go-pattern`, `golang-structs-interfaces`, `go-error-handling`, `golang-code-style`, `golang-testing`, `golang-concurrency`

**Rules:** Never deviates from spec without documenting why. Never makes design decisions. Flags ambiguities instead.

---

### go-reviewer

| Property | Value |
|----------|-------|
| Model | Sonnet |
| Role | Review and approve |
| Output | `docs/reviews/{task-slug}-review.md` |

**When to use:** After the developer has implemented and tested the code.

**What it does:**
1. Reads spec, implementation, and test code
2. Runs `go build`, `go vet`, `go test -race`
3. Evaluates against a checklist: spec compliance, Go best practices, security, performance, test quality
4. Writes review to `docs/reviews/`
5. Delivers verdict: **APPROVE** or **REQUEST_CHANGES**

**Skills loaded:** `go-error-handling`, `golang-data-structures`, `golang-performance`, `golang-design-patterns`, `golang-linter`, `go-pattern`, `golang-structs-interfaces`, `golang-code-style`, `golang-testing`, `golang-concurrency`

**Rules:** Never modifies source files. Never approves code that doesn't compile. References file:line in all feedback.

---

## Commands

### /workflow

Runs the full development pipeline end-to-end:

```
> /workflow <task description>
```

**Pipeline phases:**

| Phase | Agent | Output |
|-------|-------|--------|
| 1. Architecture | `go-architect` | `docs/specs/{task-slug}.md` |
| 2. Implementation | `go-developer` | Source code + `docs/test/{task-slug}.md` |
| 3. Review | `go-reviewer` | `docs/reviews/{task-slug}-review.md` |
| 4. Fix Loop | `go-developer` + `go-reviewer` | Iterates until APPROVED (max 2 rounds, then escalates to human) |

---

## Skills

Skills provide domain-specific knowledge that agents and conversations can invoke.

### Core Project Skills

| Skill | Triggers | Description |
|-------|----------|-------------|
| `spec-writing` | "spec", "design doc", "architecture", "planning" | Structured spec writing for Go projects with a defined template |
| `pr-reviewer` | "review PR", "check PR #123", "what PRs are open" | GitHub PR review with Effective Go compliance and Linear ticket validation |
| `go-error-handling` | "error", "sentinel", "wrap", "unwrap" | Error handling patterns: sentinel errors, custom types, wrapping rules |
| `go-pattern` | "implement", "build", "constructor", "functional options" | Go implementation patterns: constructors, middleware, repository pattern |

### Go Knowledge Skills (from [samber/cc-skills-golang](https://github.com/samber/cc-skills-golang))

| Skill | Focus Area |
|-------|------------|
| `golang-code-style` | Line length, formatting, comments, clarity conventions |
| `golang-concurrency` | Goroutines, channels, sync primitives, worker pools, pipelines |
| `golang-data-structures` | Slices, maps, arrays, generics, pointers, container packages |
| `golang-design-patterns` | Functional options, DI, graceful shutdown, clean/hexagonal architecture |
| `golang-linter` | golangci-lint configuration, nolint directives, CI setup |
| `golang-performance` | Allocation reduction, CPU efficiency, GC tuning, pooling, caching |
| `golang-structs-interfaces` | Composition, embedding, type assertions, interface segregation |
| `golang-testing` | Table-driven tests, testify, mocks, benchmarks, fuzzing, coverage |

### Invoking Skills Directly

User-invocable skills can be called with a slash command:

```
> /golang-testing
> /golang-linter
> /golang-code-style
> /golang-concurrency
> /golang-data-structures
> /golang-design-patterns
> /golang-performance
> /golang-structs-interfaces
```

Skills also activate automatically when you mention related topics in conversation.

---

## Project Output Structure

The agents produce artifacts in a standard directory layout:

```
docs/
  specs/          # Architecture specs (from go-architect)
  test/           # Test reports (from go-developer)
  reviews/        # Code reviews (from go-reviewer)
```

---

## Examples

### Full pipeline for a new feature

```
> /workflow Implement a rate limiter middleware for the API gateway
```

### Architecture only

```
> Use the go-architect agent to design a caching layer for the user service
```

### Review an existing GitHub PR

```
> /pr-reviewer Review PR #42 on owner/repo
```

### Get Go knowledge on a topic

```
> /golang-concurrency How should I structure a fan-out/fan-in pipeline?
```

```
> /golang-performance Profile shows high allocations in the request handler, what should I do?
```

### Lint setup for a project

```
> /golang-linter Set up golangci-lint for this project
```
