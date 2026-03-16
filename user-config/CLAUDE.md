# User Preferences

## Workflow — 5 Verb Skills
Available in every project via `~/.claude/skills/`:

| Command | What It Does |
|---------|-------------|
| `/claude-harness-setup` | Bootstrap harness structure (CLAUDE.md, docs/, agents, hooks, rules). Auto-detects project type: Next.js, Node, Python, Go. |
| `/claude-harness-plan` | Create structured plan with tasks, acceptance criteria, dependencies |
| `/claude-harness-work` | Implement tasks from plan (or direct request). Self-review + verify. |
| `/claude-harness-review` | 4-perspective code review: security, performance, quality, accessibility |
| `/claude-harness-release` | Changelog, version bump, tag preparation |

For a new project, run `/claude-harness-setup` first — it creates everything else.

## Principles
- Run build + lint before completing work
- Use sub-agents for isolated tasks (research, testing, implementation)
- Prefer mechanical checks (scripts, hooks) over prompt-based compliance
- Back-pressure pattern: hooks silent on success, re-engage on failure

## Harness Engineering Reference
Full guide: see `docs/best-practices.md` in the claude-code-harness-engineering repo.
