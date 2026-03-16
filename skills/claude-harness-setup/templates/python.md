# Python Project Template

## CLAUDE.md

```markdown
# CLAUDE.md

[Project name] — Python application.

## Commands
- `python -m pytest` — run tests
- `python -m mypy .` — type checking
- `ruff check .` — linting
- `ruff format .` — formatting

## Repository Map
```
src/               ← source code
tests/             ← test files
scripts/           ← build/validation scripts
docs/              ← architecture, conventions, quality
pyproject.toml     ← project config
```

## Key Invariants
1. `pytest` must pass before completing work
2. `ruff check` must pass before completing work
3. `mypy` must pass (if configured) before completing work

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
| Language | Python 3.11+ |
| Framework | [FastAPI / Click / None] |
| Testing | pytest |
| Linting | ruff |
| Type checking | mypy |

## Module Structure
[Describe how modules relate to each other]

## Data Flow
[Describe the data flow through the application]
```

## docs/CONVENTIONS.md

```markdown
# Conventions

## Golden Principles
1. **Type hints everywhere** — all function signatures, no bare `dict` or `list`
2. **Test behavior, not implementation** — prefer integration tests over mocks
3. **Fail loudly** — raise exceptions, don't return None for errors
4. **One module, one responsibility** — keep files under 300 lines
5. **Lint must pass** — `ruff check` clean before commit

## Naming
- Modules: snake_case (`data_loader.py`)
- Classes: PascalCase (`QuizParser`)
- Functions: snake_case (`calculate_score`)
- Constants: UPPER_SNAKE (`MAX_RETRIES`)
```

## verify.sh

```bash
#!/bin/bash
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

## settings.json permissions.allow

```json
[
  "Bash(python -m pytest*)",
  "Bash(python -m mypy*)",
  "Bash(ruff check*)",
  "Bash(ruff format*)",
  "Bash(python -m*)"
]
```
