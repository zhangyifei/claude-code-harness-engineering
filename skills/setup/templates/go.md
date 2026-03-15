# Go Project Template

## CLAUDE.md

```markdown
# CLAUDE.md

[Project name] — Go application.

## Commands
- `go build ./...` — build
- `go test ./...` — run tests
- `go vet ./...` — static analysis
- `golangci-lint run` — linting (if installed)

## Repository Map
```
cmd/               ← main entry points
internal/          ← private packages
pkg/               ← public packages (if any)
docs/              ← architecture, conventions, quality
scripts/           ← build/validation scripts
go.mod             ← module definition
```

## Key Invariants
1. `go build ./...` must pass before completing work
2. `go test ./...` must pass before completing work
3. `go vet ./...` must pass before completing work

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
| Language | Go 1.22+ |
| Framework | [stdlib / gin / chi / echo] |
| Testing | stdlib testing + testify |
| Linting | golangci-lint |

## Package Layout
- `cmd/` — main packages, one per binary
- `internal/` — private application code
- `pkg/` — public library code (if applicable)

## Data Flow
[Describe how data flows through the application]
```

## docs/CONVENTIONS.md

```markdown
# Conventions

## Golden Principles
1. **Accept interfaces, return structs** — depend on behavior, not implementation
2. **Errors are values** — always handle, never ignore with `_`
3. **Keep packages small** — one package, one clear purpose
4. **Table-driven tests** — use `[]struct{ name, input, want }` pattern
5. **No globals** — pass dependencies explicitly via constructors

## Naming
- Packages: lowercase single word (`scoring`, `handler`)
- Exported: PascalCase (`CalculateScore`)
- Unexported: camelCase (`parseInput`)
- Interfaces: `-er` suffix when possible (`Reader`, `Scorer`)
```

## verify.sh

```bash
#!/bin/bash
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

## settings.json permissions.allow

```json
[
  "Bash(go build*)",
  "Bash(go test*)",
  "Bash(go vet*)",
  "Bash(go run*)",
  "Bash(golangci-lint*)"
]
```
