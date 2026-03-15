---
description: "Bootstrap harness engineering structure for a new or existing project. Trigger: user says 'setup', 'init harness', 'bootstrap project', or starts a new project."
---

# /setup — Project Harness Bootstrap

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

## Step 3: Read the Template

Based on detected type, read the matching template file in this skill directory:

- `${CLAUDE_SKILL_DIR}/templates/nextjs.md`
- `${CLAUDE_SKILL_DIR}/templates/node.md`
- `${CLAUDE_SKILL_DIR}/templates/python.md`
- `${CLAUDE_SKILL_DIR}/templates/go.md`
- `${CLAUDE_SKILL_DIR}/templates/generic.md`

## Step 4: Create Files

Create all files listed in the template. Do NOT overwrite existing files — if a file exists, skip it and note it was skipped.

### Phase 1: Foundation (always create)

1. **`CLAUDE.md`** — project map (~40 lines), customized for the project type
2. **`docs/ARCHITECTURE.md`** — tech stack, data flow, component boundaries
3. **`docs/CONVENTIONS.md`** — 3-5 golden principles for this project type
4. **`docs/QUALITY.md`** — letter grades (start all at "?" pending assessment)
5. **`.claude/settings.json`** — hooks + permissions (see below for full config)
6. **`.claude/scripts/verify.sh`** — back-pressure hook (build + lint + test, adapted to project type)

### Phase 2: Agents (always create core + selected extended)

7. **`.claude/agents/research.md`** — from `${CLAUDE_SKILL_DIR}/agents/research.md`
8. **`.claude/agents/implementation.md`** — from `${CLAUDE_SKILL_DIR}/agents/implementation.md`
9. **`.claude/agents/testing.md`** — from `${CLAUDE_SKILL_DIR}/agents/testing.md`
10. **Selected extended agents** — from `${CLAUDE_SKILL_DIR}/agents/[name].md`

### Phase 3: Rules (always create)

11. **`.claude/rules/test-quality.md`** — from `${CLAUDE_SKILL_DIR}/rules/test-quality.md`
12. **`.claude/rules/implementation-quality.md`** — from `${CLAUDE_SKILL_DIR}/rules/implementation-quality.md`

### Phase 4: Hook Configuration

The `.claude/settings.json` must include:

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
  },
  "permissions": {
    "allow": [
      "(project-type-specific build/lint/test commands)"
    ]
  }
}
```

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
3. Run /plan to start planning your first task
```
