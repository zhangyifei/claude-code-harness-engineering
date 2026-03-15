---
description: "Multi-perspective code review. Trigger: user says 'review', 'check my code', 'review changes', or wants quality feedback."
---

# /review — Code Review

Review recent changes from 4 perspectives: security, performance, quality, accessibility.

## Step 1: Identify Changes

Determine what to review:
1. If there are uncommitted changes: `git diff` + `git diff --staged`
2. If user specifies files: review those files
3. If working from `Plans.md`: review files changed by completed tasks

## Step 2: Read Context

1. Read `docs/CONVENTIONS.md` for coding patterns to check against
2. Read `docs/ARCHITECTURE.md` for architectural boundaries
3. Read `docs/QUALITY.md` for known gaps

## Step 3: Review from 4 Perspectives

### Security
- SQL injection, XSS, command injection
- Hardcoded secrets or credentials
- Authentication/authorization gaps
- Input validation at system boundaries

### Performance
- N+1 queries or unnecessary loops
- Memory leaks (unclosed resources, growing collections)
- Unnecessary re-renders (React) or recomputation
- Missing caching where beneficial

### Quality
- Follows project conventions (from `docs/CONVENTIONS.md`)
- Clear naming, single responsibility
- Error handling (no swallowed errors)
- Test coverage for new logic

### Accessibility (for UI projects)
- ARIA labels on interactive elements
- Keyboard navigation support
- Color contrast
- Screen reader compatibility

## Step 4: Verdict

```
REVIEW: [APPROVE | REQUEST_CHANGES]

Critical Issues:
- [severity: critical/major/minor] [file:line] [description]

Recommendations (non-blocking):
- [file:line] [suggestion]

Summary:
[1-2 sentence overall assessment]
```

## Judgment Criteria

- **APPROVE**: No critical or major issues. Minor issues noted but non-blocking.
- **REQUEST_CHANGES**: Any critical or major issue. Security vulnerabilities are always critical.
