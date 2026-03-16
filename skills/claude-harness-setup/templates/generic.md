# Generic Project Template

Fallback template when project type can't be detected.

## CLAUDE.md

```markdown
# CLAUDE.md

[Project name] — [description].

## Commands
- [build command]
- [test command]
- [lint command]

## Repository Map
```
src/               ← source code
tests/             ← test files
docs/              ← architecture, conventions, quality
scripts/           ← build/validation scripts
```

## Key Invariants
1. Build must pass before completing work
2. Lint must pass before completing work

## When You Need More Context
- Architecture & data flow → read `docs/ARCHITECTURE.md`
- Coding patterns → read `docs/CONVENTIONS.md`
- Quality status → read `docs/QUALITY.md`
```

## docs/ARCHITECTURE.md

```markdown
# Architecture

## Tech Stack
| Layer | Choice |
|-------|--------|
| Language | [fill in] |
| Framework | [fill in] |
| Testing | [fill in] |
| Linting | [fill in] |

## Module Structure
[Describe how modules relate to each other]

## Data Flow
[Describe the data flow through the application]
```

## docs/CONVENTIONS.md

```markdown
# Conventions

## Golden Principles
1. **Type safety** — use the strongest type system available
2. **Test behavior** — test inputs and outputs, not implementation details
3. **Fail loudly** — surface errors, don't swallow them
4. **Keep it simple** — one file, one purpose
5. **Verify before done** — build + lint + test must pass

## Naming
[Define naming conventions for this language/framework]
```

## verify.sh

```bash
#!/bin/bash
set -euo pipefail
errors=()

echo "NOTE: Edit .claude/scripts/verify.sh to add your project's build/lint/test commands"

if [ ${#errors[@]} -gt 0 ]; then
  echo "=== VERIFICATION FAILED ==="
  for e in "${errors[@]}"; do echo "- $e"; done
  echo ""
  echo "Fix these issues before completing work."
  exit 2
fi
```

## settings.json permissions.allow

```json
[]
```
