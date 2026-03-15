# Next.js Project Template

## CLAUDE.md

```markdown
# CLAUDE.md

[Project name] — Next.js web application.

## Commands
- `npm run dev` — dev server
- `npm run build` — production build (verify before completing work)
- `npm run lint` — ESLint
- `npm run test` — tests
- Use `--registry https://registry.npmjs.org/` for all npm install commands

## Repository Map
```
src/app/           ← pages and layouts (App Router)
src/app/components/ ← React components
src/app/lib/       ← shared utilities, types, data loading
public/            ← static assets
docs/              ← architecture, conventions, quality
scripts/           ← build/validation scripts
```

## Key Invariants
1. `npm run build` must pass before completing work
2. `npm run lint` must pass before completing work

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
| Framework | Next.js (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Testing | [TBD — Vitest recommended] |

## Data Flow
[Describe how data flows: API → server components → client components → user]

## Component Boundaries
- Server components: pages, layouts, data fetching
- Client components: interactive UI (`"use client"`)

## Routing
| URL | Page | Data Source |
|-----|------|-------------|
| `/` | Home | [describe] |
```

## docs/CONVENTIONS.md

```markdown
# Conventions

## Golden Principles
1. **Server by default** — use `"use client"` only when needed (event handlers, hooks, browser APIs)
2. **Type everything** — no `any`, no `as unknown`, strict mode
3. **Components are pure** — props in, JSX out; state lives in pages
4. **Build must pass** — never commit with TypeScript errors
5. **Lint must pass** — fix warnings, don't disable rules

## Naming
- Components: PascalCase (`QuestionCard.tsx`)
- Utilities: camelCase (`scoring.ts`)
- Pages: `page.tsx` in route directory
- Types: PascalCase interfaces, no `I` prefix
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
  "Bash(npm run dev)",
  "Bash(npm run test)"
]
```
