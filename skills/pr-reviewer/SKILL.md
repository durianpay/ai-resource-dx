---
name: pr-reviewer
description: >
  Review GitHub pull requests using the GitHub CLI (gh). Use this skill whenever the user mentions
  PRs, pull requests, code review, reviewing changes, checking diffs, PR feedback, listing open PRs,
  or anything related to GitHub pull request workflows. This includes: listing PRs (open, closed, draft),
  checking out a PR branch locally, reviewing code changes in a PR, summarizing PR diffs, and analyzing
  code quality in pull requests. For Go files, the review enforces Effective Go compliance.
  The skill also pulls Linear ticket context (via MCP) from the PR title or comments to validate
  that the code changes match the ticket's requirements and acceptance criteria.
  Trigger even for casual mentions like "check that PR", "review #123", "what PRs are open",
  "look at the changes in PR 45", or "give me feedback on this pull request". Also trigger when the
  user says things like "review my code on GitHub", "check my open PRs", or "help me with code review".
---

# PR Reviewer Skill

Review GitHub pull requests from the command line using `gh` (GitHub CLI). This skill works globally — no need to be inside a specific repo directory since `gh` can target any repository via the `--repo` flag.

All review output goes directly to the user in the conversation. Do NOT post comments, reviews, or any feedback directly on the PR in GitHub. The user will decide what to do with the review.

## Prerequisites

- `gh` (GitHub CLI) must be installed and authenticated (`gh auth status` to verify)
- Git must be installed for checkout operations
- Linear MCP server must be connected in Claude Code for ticket lookups

## Core Workflows

### 1. List Pull Requests

List open PRs for a repository. Always start here if the user hasn't specified a PR number.

```bash
# List open PRs for a specific repo
gh pr list --repo <owner/repo>

# List with more detail
gh pr list --repo <owner/repo> --limit 20 --state open --json number,title,author,createdAt,headRefName,labels,reviewDecision

# Filter by state: open, closed, merged, all
gh pr list --repo <owner/repo> --state all

# Filter by author
gh pr list --repo <owner/repo> --author <username>

# Filter by label
gh pr list --repo <owner/repo> --label "bug"

# Search PRs with query
gh pr list --repo <owner/repo> --search "fix login"
```

If the user provides a repo URL like `https://github.com/owner/repo`, extract `owner/repo` from it.

If the user doesn't specify a repo and you're inside a git repository, `gh` will auto-detect it — you can omit `--repo`.

### 2. View PR Details

Get full context before reviewing.

```bash
# View PR summary (title, body, status, checks, reviewers)
gh pr view <number> --repo <owner/repo>

# View as JSON for structured data
gh pr view <number> --repo <owner/repo> --json title,body,author,baseRefName,headRefName,files,reviews,comments,state,additions,deletions,changedFiles

# View the full diff
gh pr diff <number> --repo <owner/repo>
```

### 3. Checkout a PR Locally

When you need to test or deeply inspect the code, check out the PR branch. This is especially useful for running linters, tests, or exploring files that aren't in the diff.

```bash
# If already inside a clone of the repo
gh pr checkout <number> --repo <owner/repo>

# If not inside a clone, clone first then checkout
gh repo clone <owner/repo> /tmp/<repo-name> -- --depth=1
cd /tmp/<repo-name>
gh pr checkout <number>
```

After checkout, you can run tests, lint, build, or explore the codebase directly.

### 4. Extract Linear Ticket Context

Before reviewing code, pull the Linear ticket to understand what the PR is supposed to accomplish. This gives you the requirements, acceptance criteria, and context to judge whether the code changes are correct and complete.

**Step A — Extract the ticket identifier from the PR.**

The PR title follows the pattern `[TICKET-ID]: description`. Extract the ticket ID using a regex match on the title. Common formats:

- `[DGAS-544]: add boolean flag...` → ticket ID is `DGAS-544`
- `[TEAM-123] fix login flow` → ticket ID is `TEAM-123`
- `DGAS-544: add boolean flag` → ticket ID is `DGAS-544` (brackets optional)

```bash
# Get the PR title
gh pr view <number> --repo <owner/repo> --json title --jq '.title'
```

Extract the ticket ID by matching the pattern `[A-Z]+-[0-9]+` from the title. If no ticket ID is found in the title, check the PR body and comments — a Linear bot often posts a comment with a link like `https://linear.app/team/issue/DGAS-544/...`.

```bash
# Get PR body and comments to look for Linear links
gh pr view <number> --repo <owner/repo> --json body,comments --jq '.body, .comments[].body'
```

Look for Linear URLs matching `linear.app/.*/issue/([A-Z]+-[0-9]+)` in the body or comments.

**Step B — Fetch the ticket from Linear via MCP.**

Use the Linear MCP tools to look up the ticket. The primary approach is to search by the ticket identifier (e.g., `DGAS-544`):

```
Use the Linear MCP to search for issue "DGAS-544"
```

From the ticket, extract and note:
- **Title**: What the ticket is about
- **Description**: Detailed requirements, context, and specs
- **Acceptance criteria**: What "done" looks like (if specified)
- **Status**: Current workflow state
- **Labels/Priority**: Severity and categorization
- **Comments**: Additional context, discussions, or decisions from the team

**Step C — Build a review context from the ticket.**

Before analyzing the code, synthesize the ticket into a review checklist:

1. What is the expected behavior change or feature being added?
2. Are there specific requirements or constraints mentioned in the ticket?
3. Are there acceptance criteria that the code must satisfy?
4. Are there edge cases or concerns raised in the ticket comments?

This checklist is used in Step C of the code review (section 5 below) to validate that the PR actually delivers what the ticket asks for.

### 5. Review the Code Changes

This is the core of the skill. All review output is presented directly to the user — never post anything to GitHub.

**Step A — Get the diff and changed files:**
```bash
gh pr diff <number> --repo <owner/repo>
gh pr view <number> --repo <owner/repo> --json files --jq '.files[].path'
```

**Step B — If the diff is large or you need full file context, checkout the PR:**
```bash
gh repo clone <owner/repo> /tmp/<repo-name> -- --depth=1
cd /tmp/<repo-name>
gh pr checkout <number>
```
Then read the full files to understand context beyond the diff hunks.

**Step C — Analyze the changes.** For each file, evaluate:

- **Correctness**: Logic errors, off-by-one mistakes, null/undefined risks, race conditions, unhandled edge cases.
- **Security**: SQL injection, XSS, hardcoded secrets, insecure dependencies, auth bypasses, path traversal.
- **Performance**: N+1 queries, unnecessary allocations in hot paths, missing indexes, O(n²) where O(n) is possible.
- **Readability**: Unclear naming, overly complex logic, missing comments for non-obvious code, dead code.
- **Testing**: Are new code paths covered by tests? Are edge cases handled? Are tests meaningful or just covering lines?
- **Best practices**: Does the code follow the project's conventions? Deprecated API usage? Code duplication?
- **Error handling**: Are errors caught and handled gracefully? Are error messages helpful and actionable?

Additionally, **validate the changes against the Linear ticket context** (from section 4):

- **Completeness**: Does the PR address everything the ticket requires? Are any acceptance criteria unmet?
- **Scope**: Does the PR introduce changes that go beyond what the ticket asks for? Are unrelated changes mixed in?
- **Intent alignment**: Does the implementation approach match what the ticket describes or what was discussed in ticket comments?
- **Missing pieces**: Are there requirements from the ticket that have no corresponding code change?

**Step D — For Go files (.go), also check Effective Go compliance.**

Read the reference checklist before reviewing Go code:
```
references/effective-go-checklist.md
```

Apply the Effective Go checklist to every `.go` file in the diff. Key areas to check:

1. **Formatting**: Is the code `gofmt`-formatted? Tabs for indentation? Opening braces on the same line?
2. **Naming**: No stuttering with package names? No `Get` prefix on getters? MixedCaps not underscores? Proper acronym casing (URL, HTTP, ID)?
3. **Error handling**: Early returns for error cases? No ignored errors? Error strings lowercase without trailing punctuation? Errors wrapped with `%w`?
4. **Control flow**: `else` omitted after return/break/continue? `switch` preferred over if-else chains? `range` used for iteration?
5. **Functions**: Multiple return values used properly? `defer` for cleanup placed right after resource acquisition?
6. **Data**: Correct use of `new` vs `make`? Composite literals used? Comma-ok idiom for map access?
7. **Concurrency**: No goroutine leaks? Channels used correctly (sender closes)? Shared state protected? Context used for cancellation?
8. **Interfaces**: Small interfaces? Accept interfaces, return structs? No premature interface definitions?
9. **Methods**: Consistent receiver types (pointer vs value)? Pointer receivers for mutations?

When flagging Effective Go violations, cite the specific rule and show the idiomatic fix.

**Step E — Produce the review.** Present the output directly to the user in this structure:

```
## PR Review: #<number> — <title>

**Author**: <author> | **Branch**: <head> → <base> | **Changed files**: <count> | **+<additions> -<deletions>**
**Linear Ticket**: <TICKET-ID> — <ticket title> (<status>)

### Summary
<1-2 sentence overview of what the PR does>

### Ticket Alignment
<Does the PR satisfy the ticket requirements? Call out:>
- ✅ Requirements met: <list what's covered>
- ❌ Requirements missing: <list what's not addressed, if any>
- ⚠️ Scope concerns: <unrelated changes or scope creep, if any>

### Overall Assessment
<✅ Looks good / ⚠️ Needs work / 🚨 Major concerns>

### Critical Issues (must fix before merge)
- **[file:line]** Description of the issue
  - Why it matters
  - Suggested fix

### Effective Go Violations (for .go files)
- **[file:line]** <Rule violated>
  - Current: `<current code>`
  - Idiomatic: `<fixed code>`
  - Reference: <specific Effective Go section>

### Suggestions (non-blocking improvements)
- **[file:line]** Description and reasoning

### Positive Notes
- Call out good patterns, clean code, or thoughtful design decisions

### Files Reviewed
- <file path> — <one-line summary of changes>
```

If there are no issues in a category, omit that section. If there are no Go files, omit the Effective Go section. If no Linear ticket was found, omit the Ticket Alignment section and note that no ticket was detected.

## Working Globally (No Repo Directory Required)

This skill is designed to work from anywhere. The `--repo <owner/repo>` flag lets most `gh pr` commands run without being in a project directory.

For checkout operations (running tests, deep file inspection), clone to a temp directory:

```bash
gh repo clone <owner/repo> /tmp/<repo-name> -- --depth=1
cd /tmp/<repo-name>
gh pr checkout <number>
```

Use shallow clones (`--depth=1`) to keep things fast when full history isn't needed.

## Handling Ambiguity

- If the user says "review PR 42" but hasn't specified a repo, check if you're in a git repo first (`git remote -v`). If not, ask which repository they mean.
- If the user says "check my PRs", ask if they mean PRs they authored or PRs assigned to them for review, and which repo(s).
- If a diff is very large (50+ files), summarize at a high level first and ask if the user wants a deep dive into specific files or areas.
- If no Linear ticket ID is found in the title, body, or comments, proceed with the review without ticket context but mention to the user that no ticket was linked.

## Important Rules

1. **Never comment on the PR directly.** All feedback goes to the user in the conversation output. The user decides what action to take.
2. **Always fetch the Linear ticket when available.** Extract the ticket ID from the PR title or comments and use Linear MCP to get the full ticket context before reviewing.
3. **Always check Effective Go compliance for .go files.** Read `references/effective-go-checklist.md` before reviewing any Go code.
4. **Show code examples.** When suggesting fixes, show the current code and the corrected version.
5. **Be specific with locations.** Always reference `file:line` so the user can find the exact spot.
6. **Prioritize findings.** Critical issues first, then ticket alignment, then Effective Go violations, then suggestions.

## Tips

- For large diffs, focus on specific files: `gh pr diff <number> --repo <owner/repo> -- <specific-file>`
- Use `--json` and `--jq` flags to extract structured data for programmatic use
- Check CI status: `gh pr checks <number> --repo <owner/repo>`
- View existing review comments: `gh pr view <number> --repo <owner/repo> --comments`