# Harness Engineering Best Practices

General-purpose guide for configuring Claude Code (and Codex CLI) as a disciplined development partner. Synthesized from three sources:

1. **OpenAI "Harness Engineering"** — CLAUDE.md as map, docs/ as system of record, mechanical enforcement, progressive disclosure
2. **HumanLayer "Skill Issue"** — Skills, sub-agents, hooks, context firewall pattern
3. **Chachamaru127/claude-code-harness** — 5-verb workflow, TypeScript guardrail engine, agent teams, SSOT pattern

---

## Table of Contents

1. [Core Principles](#1-core-principles)
2. [CLAUDE.md Design](#2-claudemd-design)
3. [Documentation Architecture](#3-documentation-architecture)
4. [Agent Architecture](#4-agent-architecture)
5. [Hook System](#5-hook-system)
6. [Skills (Task Workflows)](#6-skills-task-workflows)
7. [Rules (Guardrails)](#7-rules-guardrails)
8. [Permission & Safety Model](#8-permission--safety-model)
9. [Workflow Loop](#9-workflow-loop)
10. [Project Bootstrap Checklist](#10-project-bootstrap-checklist)
11. [User-Level Configuration](#11-user-level-configuration)

---

## 1. Core Principles

### From OpenAI: Structure Over Prompting

| Principle | Meaning |
|-----------|---------|
| **CLAUDE.md is a map, not a manual** | ~40 lines max. Points to docs/ for depth. Agent reads this first. |
| **docs/ is the system of record** | ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md hold truth |
| **Progressive disclosure** | Agent reads CLAUDE.md → follows pointers → reads design-docs/ only when needed |
| **Mechanical enforcement** | Validation scripts catch errors that prompts can't prevent |
| **Back-pressure** | Hooks that are silent on success, re-engage agent on failure with remediation hints |

### From HumanLayer: Context Firewalling

| Principle | Meaning |
|-----------|---------|
| **Sub-agents isolate noise** | Research, implementation, testing agents keep intermediate output out of parent context |
| **Skills are task workflows** | Structured prompts that guide the agent through multi-step processes |
| **Hooks enforce at runtime** | Pre/PostToolUse hooks guard execution, not just prompt compliance |

### From claude-code-harness: Operational Discipline

| Principle | Meaning |
|-----------|---------|
| **5-verb workflow** | Plan → Work → Review → Release → Setup — one clear path |
| **3 agent roles** | Worker (implement), Reviewer (read-only critique), Scaffolder (setup) |
| **SSOT pattern** | Decisions and patterns stored in versioned files, not scattered in chat |
| **Test tampering prevention** | Rules that explicitly prohibit `it.skip`, assertion deletion, config relaxation |
| **Implementation quality rules** | Prohibit hardcoded returns, stub implementations, swallowed errors |

---

## 2. CLAUDE.md Design

Keep CLAUDE.md under ~40 lines. It should answer three questions:
1. What are the commands to build/test/lint?
2. Where is the code?
3. What invariants must always hold?

### Template

```markdown
# CLAUDE.md

[Project name] — [one-line description]

## Commands
- `npm run dev` — dev server
- `npm run build` — production build (verify before completing work)
- `npm run lint` — linter
- `npm run test` — tests
- Use `--registry https://registry.npmjs.org/` for all npm install commands

## Repository Map
[tree of key directories with one-line descriptions]

## Key Invariants (enforced mechanically)
1. [invariant checked by script]
2. [invariant checked by script]

## When You Need More Context
- Architecture & data flow → read `docs/ARCHITECTURE.md`
- Coding patterns → read `docs/CONVENTIONS.md`
- Quality status → read `docs/QUALITY.md`
- [task-specific] → read `docs/design-docs/[topic].md`
```

### Anti-patterns

- Don't put the full architecture in CLAUDE.md — point to docs/ARCHITECTURE.md
- Don't list every file — show the top 2 levels of directories
- Don't duplicate what the code says — CLAUDE.md describes *where* to look, not *what* the code does
- Don't put session-specific instructions — those belong in skills or agent prompts

---

## 3. Documentation Architecture

```
docs/
├── ARCHITECTURE.md       # Tech stack, data flow, component boundaries, routing
├── CONVENTIONS.md        # Golden principles, naming, patterns (max 5 principles)
├── QUALITY.md            # Letter grades per area, known gaps, test coverage
└── design-docs/          # One file per design decision or workflow
    ├── [feature].md
    └── [workflow].md
```

### ARCHITECTURE.md

Should contain:
- Tech stack table (framework, language, styling, key libraries)
- Data flow diagram (text-based, ASCII or Mermaid)
- Component boundaries (what is server vs client, what talks to what)
- Routing table (URL → page → data source)

### CONVENTIONS.md

Maximum 5 "golden principles" — rules that prevent the most common bugs. Examples:
- "Data files are the source of truth; always validate after modification"
- "Parse external input at the boundary; internal code trusts typed data"
- "All math content uses `$...$` LaTeX delimiters — no HTML entities"
- "Components are pure functions of props; state lives in pages"
- "Never commit with failing build or lint"

### QUALITY.md

Letter grades (A-F) for each area. Agent reads this to know where quality is weak.

```markdown
| Area | Grade | Notes |
|------|-------|-------|
| Data validation | A | Mechanical checks pass |
| Type safety | B | Strict mode, but some `any` in parsing |
| Test coverage | F | No tests yet |
| Error handling | D | No error boundaries |
| Accessibility | D | No ARIA labels |
```

---

## 4. Agent Architecture

### Minimum Viable Agents (3-role model)

Based on claude-code-harness v3's consolidation of 11 agents → 3:

| Agent | Tools | Purpose | Key Constraint |
|-------|-------|---------|----------------|
| **Worker** | Read, Write, Edit, Bash, Grep, Glob | Implement → self-review → build verify → commit | No Task tool (prevent recursion) |
| **Reviewer** | Read, Grep, Glob (read-only) | Multi-perspective code review | No Write, Edit, Bash |
| **Scaffolder** | Read, Write, Edit, Bash, Grep, Glob | Project analysis, setup, state updates | Used only during setup |

### Extended Agents (add as needed)

| Agent | When to Add | Purpose |
|-------|-------------|---------|
| **Research** | Codebase exploration | Read-only investigation, returns condensed findings |
| **Testing** | When test framework exists | Write and run tests in isolation |
| **Codex-Review** | When Codex CLI installed | External second-opinion via `codex review` |
| **Codex-Fix** | When Codex-Review is used | Fix issues flagged by Codex review |

### Agent File Format

```markdown
# [Agent Name]

[One-line description of what this agent does]

## Instructions
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Response Format
[Structured output template the agent should follow]

## What NOT to do
- [Constraint 1]
- [Constraint 2]
```

### Context Firewall Pattern

Agents return condensed results to the parent. The parent never sees intermediate file reads or search results — only the conclusion.

```
User Request
    ↓
Main Agent (parent)
    ├── spawn Research Agent → "Found 3 relevant files: ..."
    ├── spawn Worker Agent → "Implemented feature, changed 2 files, build passes"
    └── spawn Reviewer Agent → "APPROVE — no critical issues"
    ↓
Main Agent summarizes to user
```

### Parallel Execution

Claude Code supports true parallelism when multiple Agent calls are made **in a single message**. This is the key mechanism — agents spawned across separate messages run sequentially.

#### Worktree Isolation

Use `isolation: "worktree"` for parallel implementation agents. Each agent gets its own copy of the repo, so file writes cannot conflict:

```
# SINGLE message with 3 parallel Agent calls:
Agent(task: "Task 1", agent: "implementation.md", isolation: "worktree")
Agent(task: "Task 2", agent: "implementation.md", isolation: "worktree")
Agent(task: "Task 3", agent: "implementation.md", isolation: "worktree")
# All 3 run simultaneously, each in its own repo copy
```

After all agents return, merge their worktree branches into the main working tree.

#### Read-only agents don't need worktrees

Review agents (read-only) can run in parallel without worktree isolation since they don't modify files:

```
# SINGLE message with 4 parallel review agents:
Agent(task: "Security review", agent: "research.md")
Agent(task: "Performance review", agent: "research.md")
Agent(task: "Quality review", agent: "research.md")
Agent(task: "Accessibility review", agent: "research.md")
```

### Team Composition (for parallel work)

From claude-code-harness:
- Lead (main agent) orchestrates — never writes code during delegation
- Workers (1-3) implement in parallel, each in isolated worktrees
- Reviewer (1) does read-only review after all workers complete
- On REQUEST_CHANGES: Lead creates fix tasks → Workers fix → Reviewer re-reviews
- Optimal: 3-5 teammates total, 5-6 tasks per teammate

### Execution Decision Tree

```
1 task                → run directly (no worktree)
2+ independent tasks  → parallel group with worktree isolation
2+ mixed tasks        → group into parallel phases + sequential steps
Review                → always 4 parallel read-only agents
```

---

## 5. Hook System

Hooks enforce behavior at runtime — they're the mechanical enforcement layer.

### Essential Hooks

| Event | What It Does | Pattern |
|-------|-------------|---------|
| **Stop** | Back-pressure: validate + build + lint | Silent on success, exit 2 on failure with hints |
| **PreToolUse** (Write/Edit) | Guardrail: block writes to protected paths | Deny writes to .env, .git/, secrets |
| **PostToolUse** (Write/Edit) | Quality check: detect test tampering | Warn on `it.skip`, assertion removal |
| **PreToolUse** (Bash) | Safety: block destructive commands | Deny `sudo`, `rm -rf /`, `git push --force` |

### Back-Pressure Hook Pattern (verify.sh)

The most impactful hook — runs on Stop, silent when green, re-engages on red. Examples for each project type:

#### Node.js

```bash
#!/bin/bash
# .claude/scripts/verify.sh — back-pressure hook (Node.js)
set -euo pipefail

errors=()

if ! npm run build --silent 2>/dev/null; then
  errors+=("Build failed — run 'npm run build' and fix TypeScript errors")
fi

if ! npm run lint --silent 2>/dev/null; then
  errors+=("Lint failed — run 'npm run lint' and fix warnings")
fi

if npm run test --silent 2>/dev/null; then
  : # tests pass
elif [ $? -ne 0 ] 2>/dev/null; then
  errors+=("Tests failed — run 'npm run test' and fix failures")
fi

if [ ${#errors[@]} -gt 0 ]; then
  echo "=== VERIFICATION FAILED ==="
  for e in "${errors[@]}"; do echo "- $e"; done
  echo ""
  echo "Fix these issues before completing work."
  exit 2  # exit 2 = feedback to agent
fi
# Silent on success — don't waste context
```

#### Python

```bash
#!/bin/bash
# .claude/scripts/verify.sh — back-pressure hook (Python)
set -euo pipefail

errors=()

if command -v ruff &> /dev/null; then
  if ! ruff check . 2>/dev/null; then
    errors+=("Lint failed — run 'ruff check .' and fix issues")
  fi
fi

if command -v mypy &> /dev/null; then
  if ! python -m mypy . 2>/dev/null; then
    errors+=("Type check failed — run 'python -m mypy .' and fix errors")
  fi
fi

if [ -f "pyproject.toml" ] || [ -d "tests" ]; then
  if ! python -m pytest --tb=short -q 2>/dev/null; then
    errors+=("Tests failed — run 'python -m pytest' and fix failures")
  fi
fi

if [ ${#errors[@]} -gt 0 ]; then
  echo "=== VERIFICATION FAILED ==="
  for e in "${errors[@]}"; do echo "- $e"; done
  echo ""
  echo "Fix these issues before completing work."
  exit 2
fi
```

#### Go

```bash
#!/bin/bash
# .claude/scripts/verify.sh — back-pressure hook (Go)
set -euo pipefail

errors=()

if ! go build ./... 2>/dev/null; then
  errors+=("Build failed — run 'go build ./...' and fix errors")
fi

if ! go vet ./... 2>/dev/null; then
  errors+=("Vet failed — run 'go vet ./...' and fix issues")
fi

if ! go test ./... -count=1 -short 2>/dev/null; then
  errors+=("Tests failed — run 'go test ./...' and fix failures")
fi

if command -v golangci-lint &> /dev/null; then
  if ! golangci-lint run 2>/dev/null; then
    errors+=("Lint failed — run 'golangci-lint run' and fix issues")
  fi
fi

if [ ${#errors[@]} -gt 0 ]; then
  echo "=== VERIFICATION FAILED ==="
  for e in "${errors[@]}"; do echo "- $e"; done
  echo ""
  echo "Fix these issues before completing work."
  exit 2
fi
```

### Agent Hooks (LLM-powered)

From claude-code-harness — use a cheap model (haiku) for lightweight checks:

```json
{
  "type": "agent",
  "prompt": "Review the code change for: (1) hardcoded secrets, (2) TODO stubs without implementation, (3) security vulnerabilities. If issues found, return {permissionDecision: 'deny', permissionDecisionReason: '...'}. If ok, return nothing.",
  "model": "haiku",
  "timeout": 30
}
```

### Hook Configuration (.claude/settings.json)

The `hooks` block is the same for all project types — only `permissions.allow` changes.

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "bash .claude/scripts/verify.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "agent",
        "prompt": "Review this code change for: (1) hardcoded secrets or API keys, (2) TODO/FIXME stubs without implementation, (3) obvious security issues (SQL injection, XSS, command injection). If issues found, include them in systemMessage as warnings. If clean, return nothing. Input: $ARGUMENTS",
        "model": "haiku",
        "timeout": 30
      }]
    }]
  }
}
```

#### Node.js permissions.allow

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run build)",
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run start)",
      "Bash(npm run dev)"
    ]
  }
}
```

#### Python permissions.allow

```json
{
  "permissions": {
    "allow": [
      "Bash(python -m pytest*)",
      "Bash(python -m mypy*)",
      "Bash(ruff check*)",
      "Bash(ruff format*)",
      "Bash(python -m*)"
    ]
  }
}
```

#### Go permissions.allow

```json
{
  "permissions": {
    "allow": [
      "Bash(go build*)",
      "Bash(go test*)",
      "Bash(go vet*)",
      "Bash(go run*)",
      "Bash(golangci-lint*)"
    ]
  }
}
```

### Permission Deny List (user-level baseline)

From claude-code-harness's settings.json — safe defaults for any project:

```json
{
  "permissions": {
    "deny": [
      "Bash(sudo:*)",
      "Bash(rm -rf:*)",
      "Bash(rm -fr:*)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(**/*.pem)",
      "Read(**/*.key)",
      "Read(**/*id_rsa*)",
      "Read(**/*id_ed25519*)",
      "Read(**/.aws/**)",
      "Read(**/.ssh/**)"
    ],
    "ask": [
      "Bash(rm -r:*)",
      "Bash(git reset --hard:*)",
      "Bash(git push -f:*)",
      "Bash(git push --force:*)",
      "Bash(git clean -f:*)",
      "Bash(git clean -fdx:*)",
      "Bash(git rebase:*)"
    ]
  }
}
```

---

## 6. Skills (Task Workflows)

Skills are structured prompts for recurring multi-step tasks. They live in `.claude/skills/[name]/SKILL.md` or `docs/design-docs/[name].md` depending on whether they need to be invocable.

### When to Use Skills vs Design Docs

| Content Type | Location | Trigger |
|-------------|----------|---------|
| **Invocable task workflow** (parse PDF, create component) | `.claude/skills/[name]/SKILL.md` | `/skill-name` command |
| **Reference knowledge** (how scoring works, architecture decisions) | `docs/design-docs/[topic].md` | Agent reads when CLAUDE.md points there |
| **Operational procedure** (deploy, release) | `.claude/skills/[name]/SKILL.md` | `/skill-name` command |

### Skill File Format

```markdown
---
description: "One-line trigger description for when this skill should activate"
---

# Skill: [Name]

## When to Use
[Trigger conditions]

## Steps
1. [Step with specific instructions]
2. [Step with specific instructions]
3. [Step with specific instructions]

## Verification
- [ ] [Check 1]
- [ ] [Check 2]
```

### 5-Verb Skill Model (from claude-code-harness)

For larger projects, organize skills around 5 verbs:

| Verb | Purpose | Consolidates |
|------|---------|-------------|
| **Plan** | Ideas → structured plan document | planning, task breakdown, scope analysis |
| **Work** | Parallel implementation | implementation, TDD, error recovery |
| **Review** | Multi-perspective code review | security, performance, quality, accessibility |
| **Release** | Changelog, tag, handoff | versioning, release notes, deployment |
| **Setup** | Project initialization | init, config, maintenance |

---

## 7. Rules (Guardrails)

Rules are always-active instructions that apply to matching file patterns. They live in `.claude/rules/`.

### Essential Rules

#### test-quality.md
Prevents test tampering — the most common agent failure mode:

- **Prohibit**: `it.skip()`, `describe.only()`, assertion deletion, test case removal
- **Prohibit**: Relaxing eslint/tsconfig/CI config to make tests pass
- **Require**: Fix implementation, not tests. Ask user if test seems wrong.

#### implementation-quality.md
Prevents hollow implementations:

- **Prohibit**: Hardcoded return values that match test expectations
- **Prohibit**: Stub implementations (`return null`, `return []`, `TODO`)
- **Prohibit**: Error swallowing (`catch { return null }`)
- **Require**: Self-check — "Would this work for inputs *not* in the tests?"

### Rule File Format

```markdown
---
description: [What this rule enforces]
paths: "**/*.{ts,tsx,js,jsx}"
---

# [Rule Name]

## Prohibited
- [Pattern 1]
- [Pattern 2]

## Required
- [Pattern 1]
- [Pattern 2]
```

---

## 8. Permission & Safety Model

### Defense in Depth (4 layers)

| Layer | What | How |
|-------|------|-----|
| 1. **Permission deny/ask lists** | Block dangerous commands | `settings.json` deny/ask arrays |
| 2. **PreToolUse hooks** | Guard writes to protected paths | Shell/TS scripts checking stdin JSON |
| 3. **PostToolUse hooks** | Detect test tampering, quality issues | Shell/TS/agent hooks checking output |
| 4. **Rules** | Always-active behavioral constraints | `.claude/rules/*.md` with path matchers |

### For Sub-agents / Agent Teams

- Workers: restrict to Read/Write/Edit/Bash/Grep/Glob — no Task tool (prevents recursion)
- Reviewers: Read/Grep/Glob only — no write capability at all
- Lead: during delegation phase, only uses Task/SendMessage — no direct file edits
- Error recovery: max 3 retries before escalation to human

---

## 9. Workflow Loop

### Minimum Viable Loop (solo developer)

```
1. Write code
2. Stop hook runs: validate + build + lint
   ├── All pass (silent) → done
   └── Failure → agent gets feedback, fixes, re-runs
```

### Standard Loop (with parallel agents)

```
1. Plan: structure tasks into parallel groups + sequential steps
2. Work: for each group, spawn N worker agents in parallel (worktree isolation)
   ├── Merge worktree branches
   ├── Run sequential tasks
   └── Repeat for next parallel group
3. Verify: Stop hook runs validate + build + lint + test
4. Review: 4 review agents in parallel (security, performance, quality, accessibility)
   ├── APPROVE → commit
   └── REQUEST_CHANGES → spawn fix agents (parallel) → re-review
5. Commit
```

### Full Loop (with Codex CLI)

The `/review` skill automatically detects Codex CLI. If `codex` is on `$PATH`, it adds an external second opinion as Step 5:

```
1-3. [Same as standard loop]
4. Review: 4 parallel agents + Codex CLI (if available)
   ├── 4 internal agents: security, performance, quality, accessibility
   ├── Codex CLI: `codex review --base main` (external second opinion)
   ├── Merge all findings into single verdict
   ├── APPROVE → commit + push
   └── REQUEST_CHANGES → spawn fix agents (parallel, codex-fix for Codex issues) → re-review
5. Commit + push
```

If Codex is not installed, Step 4 silently skips the Codex pass — the 4-perspective internal review is sufficient on its own.

---

## 10. Project Bootstrap Checklist

When setting up a new project, create these files in order:

### Phase 1: Foundation (required)

- [ ] `CLAUDE.md` — map (~40 lines)
- [ ] `docs/ARCHITECTURE.md` — tech stack, data flow, boundaries
- [ ] `docs/CONVENTIONS.md` — 3-5 golden principles
- [ ] `docs/QUALITY.md` — letter grades per area
- [ ] `.claude/settings.json` — hooks + permissions
- [ ] `.claude/scripts/verify.sh` — back-pressure hook (validate + build + lint)

### Phase 2: Agents (recommended)

- [ ] `.claude/agents/research.md` — read-only investigation
- [ ] `.claude/agents/implementation.md` — single code change + verify
- [ ] `.claude/agents/testing.md` — write and run tests

### Phase 3: Extensions (as needed)

- [ ] `.claude/agents/codex-review.md` — external review via Codex CLI
- [ ] `.claude/agents/codex-fix.md` — fix Codex review issues
- [ ] `.claude/skills/[task]/SKILL.md` — recurring task workflows
- [ ] `.claude/rules/test-quality.md` — test tampering prevention
- [ ] `.claude/rules/implementation-quality.md` — hollow implementation prevention
- [ ] `docs/design-docs/` — design decisions and deep-dive docs

### Phase 4: Team (for parallel work)

- [ ] Agent Teams configuration (Worker + Reviewer + Lead)
- [ ] Worktree isolation for parallel writes
- [ ] SubagentStart/Stop hooks for monitoring

---

## 11. User-Level Configuration

Settings that apply across all projects on this machine.

### ~/.claude/settings.json

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1"
  },
  "permissions": {
    "deny": [
      "Bash(sudo:*)",
      "Bash(rm -rf:*)",
      "Bash(rm -fr:*)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(**/*.pem)",
      "Read(**/*.key)",
      "Read(**/*id_rsa*)",
      "Read(**/*id_ed25519*)",
      "Read(**/.aws/**)",
      "Read(**/.ssh/**)"
    ],
    "ask": [
      "Bash(rm -r:*)",
      "Bash(git reset --hard:*)",
      "Bash(git push -f:*)",
      "Bash(git push --force:*)",
      "Bash(git clean -f:*)",
      "Bash(git rebase:*)"
    ]
  }
}
```

### ~/.claude/CLAUDE.md (user-level)

```markdown
# User Preferences

## npm Registry
Always use `--registry https://registry.npmjs.org/` for npm/npx commands.
Project should have local `.npmrc` with `registry=https://registry.npmjs.org/`.

## Workflow
- Always run build + lint before completing work
- Use sub-agents for isolated tasks (research, testing, implementation)
- Prefer mechanical checks over prompt-based compliance

## Code Style
- TypeScript strict mode
- Prefer functional patterns
- Minimal dependencies
```

---

## Quick Reference Card

| Concept | File | Purpose |
|---------|------|---------|
| Project map | `CLAUDE.md` | Entry point (~40 lines) |
| Architecture | `docs/ARCHITECTURE.md` | Tech stack, data flow |
| Conventions | `docs/CONVENTIONS.md` | Golden principles |
| Quality | `docs/QUALITY.md` | Letter grades, gaps |
| Hooks | `.claude/settings.json` | Runtime enforcement |
| Back-pressure | `.claude/scripts/verify.sh` | Silent success, loud failure |
| Agents | `.claude/agents/*.md` | Context-isolated workers |
| Skills | `.claude/skills/*/SKILL.md` | Task workflows |
| Rules | `.claude/rules/*.md` | Always-active guardrails |
| Design docs | `docs/design-docs/*.md` | Deep-dive references |
| Decisions | `.claude/memory/decisions.md` | Why we chose X (SSOT) |
| Patterns | `.claude/memory/patterns.md` | How we do X (SSOT) |
