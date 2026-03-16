---
description: "Bootstrap harness engineering structure for a new or existing project. Trigger: user says 'setup', 'init harness', 'bootstrap project', or starts a new project."
---

# /claude-harness-setup — Project Harness Bootstrap

Scaffolds the harness engineering structure (CLAUDE.md, docs/, agents, hooks, rules) for the current project. Adapts to the detected or specified project type.

## Step 1: Detect Project Type

Read the current directory for signals:

| Signal | Type |
|--------|------|
| `package.json` + `next.config.*` | **nextjs** |
| `package.json` (no next) | **node** |
| `pyproject.toml` or `setup.py` or `requirements.txt` | **python** |
| `go.mod` | **go** |
| None of the above | **generic** |

If ambiguous, ask the user which type to use.

## Step 2: Choose Agent Roles

Ask the user which **extended roles** to include (in addition to the 3 core roles that are always created).

### Core Roles (always included)
- **Research** — read-only codebase investigation
- **Implementation** — single code change + verify
- **Testing** — write and run tests

### Extended Roles (user selects)
- **Architect** — system design, module boundaries, coupling analysis. Updates docs/ARCHITECTURE.md. Best for: backend services, complex systems, microservices.
- **Designer** — UI/UX, component hierarchy, accessibility, responsive design. Best for: web apps, mobile apps, any project with a UI.
- **Codex-Review** — external second opinion via Codex CLI. Best for: teams, open source, high-stakes code.
- **Codex-Fix** — auto-fix issues flagged by Codex review. Best for: paired with Codex-Review.

Present these as a multi-select question. Pre-select **Architect** for all types, and additionally **Designer** for nextjs/node (frontend) types.

Read selected agent templates from `${CLAUDE_SKILL_DIR}/agents/[name].md` and copy them into the project's `.claude/agents/` directory.

## Step 3: Read the Sources

For the detected project type, read **two** sources:

### 3a. Template (skeleton with placeholders)

Contains CLAUDE.md template, minimal docs stubs, verify.sh script, and permissions.allow list:

- `${CLAUDE_SKILL_DIR}/templates/nextjs.md`
- `${CLAUDE_SKILL_DIR}/templates/node.md`
- `${CLAUDE_SKILL_DIR}/templates/python.md`
- `${CLAUDE_SKILL_DIR}/templates/go.md`
- `${CLAUDE_SKILL_DIR}/templates/generic.md`

### 3b. Sample docs (rich reference for docs/ files)

Contains fully fleshed-out ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md for the project type. Use these to generate richer `docs/` content than the template stubs alone:

- Node.js / Next.js: `docs/samples/node/` in the harness-engineering repo
- Python: `docs/samples/python/` in the harness-engineering repo
- Go: `docs/samples/go/` in the harness-engineering repo

**How these two sources work together:**
- **CLAUDE.md** → generate from template (it has the right commands and repo map for the type)
- **docs/ARCHITECTURE.md** → use sample doc as the starting point, adapt `[fill in]` sections based on what you observe in the project
- **docs/CONVENTIONS.md** → use sample doc as the starting point (golden principles are type-specific)
- **docs/QUALITY.md** → use sample doc structure, start all grades at "?" pending assessment
- **verify.sh** → use verbatim from template (it has the right build/lint/test commands)
- **permissions.allow** → use verbatim from template (it has the right allow list)

## Step 4: Create Files

Create all files listed below. Do NOT overwrite existing files — if a file exists, skip it and note it was skipped. This lets users pre-create any file to override the default.

### Override rule

The agent skips any file that already exists. To customize:
- **Before running `/claude-harness-setup`**: create the file yourself (e.g., write your own `docs/CONVENTIONS.md`), and setup will skip it
- **After running `/claude-harness-setup`**: edit any generated file directly — they're yours now, not managed by the framework

### Phase 1: Foundation (always create)

1. **`CLAUDE.md`** — from template `## CLAUDE.md` section, customized for the project
2. **`docs/ARCHITECTURE.md`** — from sample docs, adapted to what's observed in the project
3. **`docs/CONVENTIONS.md`** — from sample docs (type-specific golden principles)
4. **`docs/QUALITY.md`** — from sample docs structure, all grades start at "?"
5. **`.claude/settings.json`** — hooks (Phase 4) + permissions.allow from template
6. **`.claude/scripts/verify.sh`** — verbatim from template `## verify.sh` section

### Phase 2: Agents (always create core + selected extended)

7. **`.claude/agents/research.md`** — from `${CLAUDE_SKILL_DIR}/agents/research.md`
8. **`.claude/agents/implementation.md`** — from `${CLAUDE_SKILL_DIR}/agents/implementation.md`
9. **`.claude/agents/testing.md`** — from `${CLAUDE_SKILL_DIR}/agents/testing.md`
10. **Selected extended agents** — from `${CLAUDE_SKILL_DIR}/agents/[name].md`

### Phase 3: Rules (always create)

11. **`.claude/rules/test-quality.md`** — from `${CLAUDE_SKILL_DIR}/rules/test-quality.md`
12. **`.claude/rules/implementation-quality.md`** — from `${CLAUDE_SKILL_DIR}/rules/implementation-quality.md`

### Phase 4: Hook Configuration

The `.claude/settings.json` must include the hooks block below **plus** the type-specific `permissions.allow` and `verify.sh` from the template read in Step 3.

**Hooks (same for all project types):**

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

**Permissions and verify.sh (type-specific — read from template):**

Each template file contains a `## verify.sh` section and a `## settings.json permissions.allow` section. Use these verbatim:

| Type | verify.sh runs | permissions.allow |
|------|---------------|-------------------|
| **node / nextjs** | `npm run build` + `npm run lint` + `npm run test` | `Bash(npm run build)`, `Bash(npm run lint)`, `Bash(npm run test)`, etc. |
| **python** | `ruff check .` + `python -m mypy .` + `python -m pytest` | `Bash(python -m pytest*)`, `Bash(ruff check*)`, `Bash(python -m mypy*)`, etc. |
| **go** | `go build ./...` + `go vet ./...` + `go test ./...` + `golangci-lint run` | `Bash(go build*)`, `Bash(go test*)`, `Bash(go vet*)`, `Bash(golangci-lint*)`, etc. |
| **generic** | Empty (user must fill in) | `[]` |

**IMPORTANT**: Do not use the placeholder `"(project-type-specific ...)"`. Always extract the actual commands from the template.

## Step 5: Make verify.sh Executable

```bash
chmod +x .claude/scripts/verify.sh
```

## Step 6: Report

```
HARNESS SETUP COMPLETE
Type: [detected type]

Core Agents: research, implementation, testing
Extended Agents: [selected roles]
Hooks: Stop (verify.sh), PostToolUse (haiku quality check)

Created:
- CLAUDE.md
- docs/ARCHITECTURE.md, CONVENTIONS.md, QUALITY.md
- .claude/settings.json (hooks + permissions)
- .claude/scripts/verify.sh (back-pressure)
- .claude/agents/ ([list all])
- .claude/rules/ (test-quality, implementation-quality)

Skipped (already existed):
- [list]

Next steps:
1. Review CLAUDE.md and customize the repository map
2. Fill in docs/ARCHITECTURE.md with your actual architecture
3. Run /claude-harness-plan to start planning your first task
```
