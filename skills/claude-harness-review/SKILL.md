---
description: "Multi-perspective code review. Trigger: user says 'review', 'check my code', 'review changes', or wants quality feedback."
---

# /claude-harness-review — Parallel Code Review

Review recent changes from 4 perspectives simultaneously: security, performance, quality, accessibility.

## Step 1: Identify Changes

Determine what to review:
1. If there are uncommitted changes: `git diff` + `git diff --staged`
2. If user specifies files: review those files
3. If working from `Plans.md`: review files changed by completed tasks

Collect the list of changed files and their diffs. This context will be passed to all 4 review agents.

## Step 2: Read Shared Context

Read these files and include their content in each agent's prompt:
1. `docs/CONVENTIONS.md` — coding patterns to check against
2. `docs/ARCHITECTURE.md` — architectural boundaries
3. `docs/QUALITY.md` — known gaps

## Step 3: Spawn 4 Review Agents in Parallel

**CRITICAL**: All 4 agents MUST be spawned in a single message block so they run simultaneously.

```
# In a SINGLE message, spawn all 4 review agents:

Agent(
  prompt: "SECURITY REVIEW of the following changes: [diff/files].
    Project conventions: [from CONVENTIONS.md].
    Check for:
    - SQL injection, XSS, command injection
    - Hardcoded secrets, API keys, credentials
    - Authentication/authorization gaps
    - Input validation at system boundaries
    - Path traversal, SSRF, insecure deserialization
    Return: list of issues with severity (critical/major/minor) and file:line references.
    If no issues, return 'PASS: no security issues found'.",
  agent: ".claude/agents/research.md"
)

Agent(
  prompt: "PERFORMANCE REVIEW of the following changes: [diff/files].
    Project conventions: [from CONVENTIONS.md].
    Check for:
    - N+1 queries or unnecessary loops
    - Memory leaks (unclosed resources, growing collections)
    - Unnecessary re-renders (React) or recomputation
    - Missing caching or memoization where beneficial
    - Bundle size impact (large imports, tree-shaking)
    Return: list of issues with severity and file:line references.
    If no issues, return 'PASS: no performance issues found'.",
  agent: ".claude/agents/research.md"
)

Agent(
  prompt: "QUALITY REVIEW of the following changes: [diff/files].
    Project conventions: [from CONVENTIONS.md].
    Architecture: [from ARCHITECTURE.md].
    Known gaps: [from QUALITY.md].
    Check for:
    - Follows project conventions and golden principles
    - Clear naming, single responsibility
    - Error handling (no swallowed errors, no bare catch)
    - Test coverage for new logic
    - Dead code, unused imports, leftover debug statements
    Return: list of issues with severity and file:line references.
    If no issues, return 'PASS: no quality issues found'.",
  agent: ".claude/agents/research.md"
)

Agent(
  prompt: "ACCESSIBILITY REVIEW of the following changes: [diff/files].
    Check for (skip if project has no UI layer):
    - ARIA labels on interactive elements (buttons, links, inputs)
    - Keyboard navigation support (focus management, tab order)
    - Color contrast (WCAG 2.1 AA minimum 4.5:1 text, 3:1 large text)
    - Screen reader compatibility (alt text, semantic HTML, live regions)
    - Form labels and error announcements
    Return: list of issues with severity and file:line references.
    If no UI changes or no issues, return 'PASS: no accessibility issues found'.",
  agent: ".claude/agents/research.md"
)
```

Note: Review agents use the **research** agent template (read-only). They must NOT modify any files.

## Step 4: Collect and Merge Results

After all 4 agents return, merge their findings into a single verdict:

```
REVIEW: [APPROVE | REQUEST_CHANGES]

Security:  [PASS | N issues]
- [critical] [file:line] [description]

Performance:  [PASS | N issues]
- [major] [file:line] [description]

Quality:  [PASS | N issues]
- [minor] [file:line] [description]

Accessibility:  [PASS | N issues]
- [minor] [file:line] [description]

Summary:
[1-2 sentence overall assessment]
```

## Verdict Logic

```
Any critical issue?
├── Yes → REQUEST_CHANGES
└── No → Any major issue?
    ├── Yes → REQUEST_CHANGES
    └── No → APPROVE (with minor issues noted)
```

Security vulnerabilities are always **critical**. Performance and quality issues start at **major** unless they're cosmetic. Accessibility issues start at **minor** unless they block core functionality.

## Step 5: Codex CLI Review (if available)

After the 4-perspective review, check if Codex CLI is installed:

```bash
command -v codex >/dev/null 2>&1
```

If Codex is available, run an **external second opinion** as an additional review pass:

1. Spawn the `codex-review` agent (if `.claude/agents/codex-review.md` exists) or run directly:
   ```bash
   codex review --base main
   ```
2. Include Codex findings in the final report under a separate **Codex** section
3. Codex critical/major issues contribute to the verdict (can upgrade APPROVE → REQUEST_CHANGES)

If Codex is NOT available, skip this step silently — the 4-perspective review is sufficient on its own.

```
Codex CLI:  [PASS | N issues | SKIPPED (not installed)]
- [severity] [description]
```

## After REQUEST_CHANGES

If the verdict is REQUEST_CHANGES, suggest running `/claude-harness-work` to fix the issues:
- List each issue as a task
- Group independent fixes as parallel
- If the project has a `codex-fix` agent (`.claude/agents/codex-fix.md`), use it for Codex-specific issues
- Run `/claude-harness-review` again after fixes
