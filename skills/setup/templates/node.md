# Node.js Project Template

## CLAUDE.md

```markdown
# CLAUDE.md

[Project name] — Node.js application.

## Commands
- `npm run build` — compile TypeScript
- `npm run start` — run the application
- `npm run test` — run tests
- `npm run lint` — ESLint
- Use `--registry https://registry.npmjs.org/` for all npm install commands

## Repository Map
```
src/               ← source code
tests/             ← test files
scripts/           ← build/validation scripts
docs/              ← architecture, conventions, quality
```

## Key Invariants
1. `npm run build` must pass before completing work
2. `npm run lint` must pass before completing work
3. `npm run test` must pass before completing work

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
| Runtime | Node.js 20+ |
| Language | TypeScript (strict) |
| Framework | [Express / Fastify / None] |
| Testing | [Vitest / Jest] |
| Linting | ESLint |

## Module Structure
[Describe how modules relate to each other]

## Data Flow
[Describe the data flow through the application]
```

## docs/CONVENTIONS.md

```markdown
# Conventions

## Golden Principles
1. **TypeScript strict mode** — no `any`, no `as unknown`, enable all strict checks
2. **Explicit error handling** — never swallow errors, use typed Result patterns
3. **One module, one responsibility** — keep files focused
4. **Build must pass** — never commit with TypeScript errors
5. **Test behavior** — test inputs and outputs, minimize mocks

## Naming
- Files: camelCase (`userService.ts`)
- Classes: PascalCase (`UserService`)
- Functions: camelCase (`calculateScore`)
- Constants: UPPER_SNAKE (`MAX_RETRIES`)
- Types/Interfaces: PascalCase, no `I` prefix
```

## verify.sh

```bash
#!/bin/bash
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
  exit 2
fi
```

## settings.json permissions.allow

```json
[
  "Bash(npm run build)",
  "Bash(npm run lint)",
  "Bash(npm run test)",
  "Bash(npm run start)",
  "Bash(npm run dev)"
]
```
