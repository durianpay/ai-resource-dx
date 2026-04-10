---
name: {skill-name}
description: >
  {One paragraph describing what the skill provides and when to trigger it.
  Include keyword triggers (e.g., "Use when the user mentions X, Y, or Z").
  This is used for auto-activation — be generous with trigger phrases.}
user-invocable: {true | false}
---

# {Skill Title}

## Purpose

{One or two sentences explaining what this skill helps with and the outcome it produces.}

## {Core Section 1}

{The main content of the skill. Structure depends on skill type:

- **Pattern/reference skills** (e.g., error handling, design patterns):
  Use categorized subsections with code examples.

- **Workflow/process skills** (e.g., PR review, linting):
  Use numbered steps with bash commands and decision points.

- **Knowledge skills** (e.g., concurrency, data structures):
  Use concept → code example → rules/gotchas structure.

Include code blocks with language tags (```go, ```bash, etc.).
Show both good and bad patterns where relevant.}

### {Subsection}

```go
// Code example with comments explaining intent
```

## {Core Section 2}

{Continue with additional sections as needed. Each section should cover
one coherent topic. Keep sections scannable — prefer code examples over
long prose.}

## {Rules / Anti-Patterns / Checklist}

{End with actionable constraints or a checklist. Use bullet points or
checkboxes. This section anchors the skill's guidance into concrete actions.}

- {Rule or checklist item}
- {Rule or checklist item}
