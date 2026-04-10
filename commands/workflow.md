---
description: "Run the full development pipeline: go-architect → go-developer → go-reviewer"
---

# Development Pipeline

Execute the full agent pipeline for the given task. Follow these steps strictly in order.

## Token Efficiency
All agents in this pipeline use caveman mode (lite for architect/reviewer, ultra for developer) to minimize token usage. Prose is compressed; code blocks, technical terms, and document structure stay intact.

## Phase 1: Architecture
Use the `go-architect` sub-agent to:
1. Analyze the task requirements
2. Check existing codebase for relevant code
3. Produce a spec at `docs/specs/{task-slug}.md`
4. Ensure the spec has all required sections: Overview, Package Layout, Interfaces, Types, Function Signatures, Error Handling, Dependencies, Constraints

Wait for the architect to finish. Read the spec and verify it's complete before proceeding.

## Phase 2: Implementation
Use the `go-developer` sub-agent to:
1. Read the spec from Phase 1
2. Implement the code following spec exactly
3. Write the unit tests for code implemented
4. Report any deviations from spec
5. Produce a test report at `docs/test/{task-slug}.md`

Wait for the go-developer to finish. Read the test report before proceeding.

## Phase 3: Review
Use the `go-reviewer` sub-agent to:
1. Read the spec, implementation, and test report
2. Run the full review checklist
3. Write review to `docs/reviews/{task-slug}-review.md`
4. Deliver verdict: APPROVE or REQUEST_CHANGES

## Phase 4: Fix Loop (if needed)
If reviewer returns REQUEST_CHANGES:
1. Send the review feedback to the `go-developer` sub-agent for fixes
2. Re-run the `go-reviewer` sub-agent
3. Repeat until APPROVED (max 2 iterations, then escalate to human)

## Completion
When approved, summarize:
- What was built
- Key design decisions
- Test coverage
- Any open items or follow-ups