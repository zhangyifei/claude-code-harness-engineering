# Claude Code Harness Engineering

A reusable framework for configuring Claude Code as a disciplined development partner. Provides project scaffolding, agent roles, hook-based enforcement, and structured workflows that work across any project type.

Synthesized from:
- [OpenAI "Harness Engineering"](https://openai.com/index/harness-engineering/) — CLAUDE.md as map, docs/ as system of record, mechanical enforcement
- [HumanLayer "Skill Issue"](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — Skills, sub-agents, hooks, context firewall
- [Chachamaru127/claude-code-harness](https://github.com/Chachamaru127/claude-code-harness) — 5-verb workflow, agent teams, SSOT pattern

## Quick Start

### 1. Install user-level config

Copy the user-level files to your `~/.claude/` directory:

```bash
# Copy the 5-verb skills (available in every project)
cp -r skills/ ~/.claude/skills/

# Copy or merge user-level CLAUDE.md
cp user-config/CLAUDE.md ~/.claude/CLAUDE.md

# Merge permissions into your existing ~/.claude/settings.json
# (see user-config/settings.json for the deny/ask lists)
```

### 2. Bootstrap a new project

```bash
mkdir my-project && cd my-project
git init
# Set up your project (npm init, go mod init, etc.)
claude
```

Then in Claude Code:
```
/claude-harness-setup
```

This auto-detects your project type (Next.js, Node, Python, Go) and creates:
- `CLAUDE.md` — project map (~40 lines)
- `docs/` — ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md
- `.claude/settings.json` — Stop hook + PostToolUse haiku hook + permissions
- `.claude/scripts/verify.sh` — back-pressure (build + lint + test)
- `.claude/agents/` — core + selected extended agents
- `.claude/rules/` — test-quality, implementation-quality

### 3. Use the workflow

```
/claude-harness-plan     → Structure tasks with acceptance criteria
/claude-harness-work     → Implement tasks (with sub-agents for parallel work)
/claude-harness-review   → 4-perspective code review (security, performance, quality, accessibility)
/claude-harness-release  → Changelog, version bump, tag preparation
```

## What's Inside

```
├── README.md                          ← you are here
├── docs/
│   ├── best-practices.md             ← comprehensive guide (all principles + patterns)
│   └── samples/                      ← sample docs/ files for each project type
│       ├── node/                     ← ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md
│       ├── python/                   ← ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md
│       └── go/                       ← ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md
├── skills/                            ← 5-verb workflow skills (copy to ~/.claude/skills/)
│   ├── claude-harness-setup/
│   │   ├── SKILL.md                  ← /claude-harness-setup — project bootstrap with role selection
│   │   ├── templates/                ← project-type configs
│   │   │   ├── nextjs.md
│   │   │   ├── node.md
│   │   │   ├── python.md
│   │   │   ├── go.md
│   │   │   └── generic.md
│   │   ├── agents/                   ← agent templates (copied into projects)
│   │   │   ├── research.md           ← core: read-only investigation
│   │   │   ├── implementation.md     ← core: code change + verify
│   │   │   ├── testing.md            ← core: write + run tests
│   │   │   ├── architect.md          ← extended: system design + coupling analysis
│   │   │   └── designer.md           ← extended: UI/UX + accessibility
│   │   └── rules/                    ← rule templates (copied into projects)
│   │       ├── test-quality.md       ← prevent test tampering
│   │       └── implementation-quality.md ← prevent hollow implementations
│   ├── claude-harness-plan/SKILL.md                 ← /claude-harness-plan — structured planning
│   ├── claude-harness-work/SKILL.md                 ← /claude-harness-work — implement from plan
│   ├── claude-harness-review/SKILL.md               ← /claude-harness-review — multi-perspective review
│   └── claude-harness-release/SKILL.md              ← /claude-harness-release — changelog + version + tag
└── user-config/                       ← user-level Claude Code config
    ├── CLAUDE.md                      ← baseline preferences for ~/.claude/CLAUDE.md
    └── settings.json                  ← permission deny/ask lists for ~/.claude/settings.json
```

## Core Concepts

### CLAUDE.md as a Map (not a manual)

~40 lines max. Answers three questions: What commands? Where is the code? What invariants? Points to `docs/` for everything else.

### How `docs/` Works in the Framework

Every project gets a `docs/` folder with three files. These are not just documentation — they're **active context** that agents read before every task:

```
docs/
├── ARCHITECTURE.md       ← agents read before implementation (tech stack, boundaries, data flow)
├── CONVENTIONS.md        ← agents read before writing code (naming, patterns, golden rules)
├── QUALITY.md            ← agents read before review (known gaps, grade per area)
└── design-docs/          ← agents read on-demand when CLAUDE.md points here
```

**Who reads what:**

| Workflow | Reads |
|----------|-------|
| `/claude-harness-plan` | ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md — to scope tasks correctly |
| `/claude-harness-work` | CONVENTIONS.md — each implementation agent reads before coding |
| `/claude-harness-review` | All three — review agents check code against conventions and known gaps |
| **Architect agent** | ARCHITECTURE.md — updates it when boundaries change |

**Sample docs** for Node.js, Python, and Go are in [`docs/samples/`](docs/samples/). Use them as a starting point — `/claude-harness-setup` generates these tailored to your project.

### Progressive Disclosure

```
CLAUDE.md (always loaded, ~40 lines)
    ↓ "read docs/ARCHITECTURE.md"
docs/ (on-demand, detailed)
    ↓ "read docs/design-docs/auth.md"
design-docs/ (deep-dive, rare)
```

### Back-Pressure Hooks

The Stop hook (`verify.sh`) runs build + lint + test on every task completion. Silent on success (saves context), exit 2 on failure (feeds error back to agent for auto-fix).

### Context Firewall

Sub-agents (research, implementation, testing) isolate intermediate noise. The parent agent only sees condensed results, not raw file reads and search output.

### Agent Roles

| Role | Tools | Constraint |
|------|-------|------------|
| **Research** | Read, Grep, Glob | Read-only |
| **Implementation** | Read, Write, Edit, Bash, Grep, Glob | Must verify build |
| **Testing** | All except Agent | No Task recursion |
| **Architect** | Read, Grep, Glob + Write to docs/ only | No source code changes |
| **Designer** | Read, Grep, Glob | Analysis only |

### Defense in Depth (4 layers)

1. **Permission deny/ask lists** — block `sudo`, `rm -rf`, secret file reads
2. **PreToolUse hooks** — guard writes to protected paths
3. **PostToolUse hooks** — haiku agent checks every Write/Edit for secrets and stubs
4. **Rules** — always-active guardrails (test-quality, implementation-quality)

## Detailed Documentation

For the full guide covering all principles, patterns, and configuration details:

→ [docs/best-practices.md](docs/best-practices.md)

## License

MIT
