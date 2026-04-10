---
name: {agent-name}
description: >
  {One paragraph describing the agent's role, when to use it, and what it produces.
  This appears in the agent picker — be specific about triggers and scope.}
model: {opus | sonnet | haiku}
tools: {comma-separated list of tools the agent needs}
color: {cyan | blue | orange | green | red | purple | yellow}
memory: {project | user | none}
skills:
  - {skill-name-1}
  - {skill-name-2}
---

## Communication Mode

{How the agent communicates. Specify caveman level if used (lite/full/ultra).
State explicitly whether output documents are written in normal prose or caveman.}

---

## Role

{One sentence defining the agent's role and boundaries. State what it does AND what it never does.}

---

## Responsibilities

{Numbered list of what this agent is accountable for. Order from first action to last.}

1. {First responsibility}
2. {Second responsibility}
3. ...

---

## Workflow

{Numbered steps the agent follows from start to finish. Each step should be
a bold keyword followed by a dash and explanation. Be explicit about what to
read, what to run, and what to check at each step.}

1. **{Step name}** — {What to do and what to check before proceeding.}
2. **{Step name}** — {Next step.}
3. ...

### Fix Loop (Second-Pass)

{Instructions for when the agent is re-invoked after feedback. What to read,
what to focus on, and how to update output artifacts. Omit this section only
if the agent is never part of a feedback loop.}

1. **{Step name}** — {What to do on re-invocation.}
2. ...

---

## Output

{Describe the artifact this agent produces. Include the file path pattern and
the document template with all required sections.}

Write output to `docs/{output-dir}/{task-slug}.md` with this structure:

```markdown
# {Document Title}: {Task Title}
**Date:** {date}
**Agent:** {agent-name}
**Status:** {status values if applicable}

## {Section 1}
{Description of what goes here.}

## {Section 2}
{Description of what goes here.}
```

---

## Rules

{Bullet list of hard constraints. Start each rule with NEVER or ALWAYS where possible.
These are non-negotiable behaviors the agent must follow.}

- NEVER {forbidden action}.
- ALWAYS {required action}.
- {Additional constraint with explanation.}

---

## {Domain-Specific Section}

{Sections unique to this agent's role. Examples:
- Architect: Spec Scoping, Handling Open Questions, Linear Integration
- Developer: Code Style, Testing Rules, Test Template
- Reviewer: Review Checklist

Add as many domain sections as needed after Rules. Keep each focused on one concern.}

---

## Feedback Protocol

{How this agent handles feedback from other agents in the pipeline.
Define scenarios and who owns the fix for each. Omit only if the agent
operates in complete isolation.}

- **{Scenario 1}:** {What triggers it and what the agent does.}
- **{Scenario 2}:** {What triggers it and what the agent does.}
