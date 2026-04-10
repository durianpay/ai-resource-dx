---
description: "{Short one-line description of what the command does}"
---

# {Command Title}

{One sentence explaining the command's purpose and when to use it.}

## Setup

{Pre-execution steps: derive slugs, create directories, validate prerequisites.
Omit this section if the command has no setup requirements.}

1. **{Setup step}** — {What to prepare before the main phases.}

## Phase 1: {Phase Name}

{Each phase describes one logical unit of work. Specify:
- Which agent or tool to use
- What context to pass (file paths, prior phase artifacts)
- What to verify before proceeding to the next phase}

Use the `{agent-name}` sub-agent to:
1. {Action}
2. {Action}

Wait for the agent to finish. Verify:
- {Verification check}
- {Verification check}

**If {error condition}:** {What to do — escalate, retry, or skip.}

## Phase 2: {Phase Name}

{Next phase. Include what context to pass from the previous phase.}

Use the `{agent-name}` sub-agent. In the agent prompt, include:
- {Context item from prior phase}
- {Context item from prior phase}

The agent will:
1. {Action}
2. {Action}

## Phase N: Fix Loop (if needed)

{Describe the feedback loop if the command supports iteration between agents.
Specify: max iterations, how to pass feedback between agents, and when to
escalate to the user.}

If {agent} returns {failure condition}:

1. **{Corrective action}** — Re-invoke `{agent}` with:
   - {Context to pass}
   - {Instructions for the re-invocation}
2. **{Re-validation}** — Re-invoke `{validator-agent}` with the same paths.
3. **Iterate** — Repeat steps 1-2 until {success condition} (max {N} iterations). If not resolved, escalate to the user.

## Completion

{What to report to the user when the command finishes successfully.}

When complete, summarize:
- {Summary item}
- {Summary item}
