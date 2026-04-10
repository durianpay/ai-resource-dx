---
description: "Run the full development pipeline: go-architect → go-developer → go-reviewer"
---

# Development Pipeline

Execute the full agent pipeline for the given task. Follow these steps strictly in order.

## Token Efficiency
All agents in this pipeline use caveman mode (lite for architect/reviewer, ultra for developer) to minimize token usage. Prose is compressed; code blocks, technical terms, and document structure stay intact.

## Setup

1. **Derive task slug** — Convert the task description to a short, lowercase, hyphenated slug (e.g., "Implement rate limiter middleware" → `rate-limiter-middleware`). Use this slug consistently for all artifact paths.
2. **Create output directories** — Ensure `docs/specs/`, `docs/test/`, and `docs/reviews/` exist.

## Phase 1: Architecture

Use the `go-architect` sub-agent to:
1. Analyze the task requirements
2. Check existing codebase for relevant code
3. Produce a spec at `docs/specs/{task-slug}.md`
4. Ensure the spec has all required sections: Overview, Package Layout, Interfaces, Types, Function Signatures, Error Handling, Dependencies, Constraints

Wait for the architect to finish. Read the spec and verify:
- The spec exists at the expected path
- The `**Status:**` field is `final` (not `draft`)
- All required sections are present

**If the spec status is `draft`:** The architect has open questions that need human input. Surface the open questions to the user and wait for answers. Then re-run the architect to finalize the spec. Do NOT proceed to Phase 2 with a draft spec.

## Phase 2: Implementation

Use the `go-developer` sub-agent. In the agent prompt, include:
- The task description
- The spec path: `docs/specs/{task-slug}.md`

The developer will:
1. Read the spec from Phase 1
2. Implement the code following spec exactly
3. Write the unit tests for code implemented
4. Run build, test, vet, and fmt validation
5. Report any deviations from spec
6. Produce a test report at `docs/test/{task-slug}.md`

Wait for the developer to finish. Read the test report and verify all commands passed before proceeding.

## Phase 3: Review

Use the `go-reviewer` sub-agent. In the agent prompt, include:
- The task description
- The spec path: `docs/specs/{task-slug}.md`
- The test report path: `docs/test/{task-slug}.md`

The reviewer will:
1. Read the spec, implementation, and test report
2. Run the full review checklist
3. Write review to `docs/reviews/{task-slug}-review.md`
4. Deliver verdict: APPROVE or REQUEST_CHANGES

## Phase 4: Fix Loop (if needed)

If reviewer returns REQUEST_CHANGES:

1. **Send fixes to developer** — Re-invoke the `go-developer` sub-agent. In the prompt, include:
   - The spec path: `docs/specs/{task-slug}.md`
   - The review path: `docs/reviews/{task-slug}-review.md`
   - Instruction to read the review's "Approval Conditions" and fix each item
2. **Re-run reviewer** — Re-invoke the `go-reviewer` sub-agent with the same paths. Note in the prompt that this is pass 2 so the reviewer follows its second-pass protocol.
3. **Iterate** — Repeat steps 1-2 until APPROVED (max 2 iterations total). If still not approved after 2 rounds, escalate to the user with a summary of unresolved issues.

## Completion

When approved, summarize:
- What was built
- Key design decisions
- Test coverage
- Any open items or follow-ups
