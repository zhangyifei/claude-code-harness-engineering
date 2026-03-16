# Conventions

## Golden Principles

1. **Accept interfaces, return structs** — depend on behavior, not implementation; define interfaces at the call site
2. **Errors are values** — always handle errors; never discard with `_`; wrap with `fmt.Errorf("context: %w", err)`
3. **Keep packages small** — one package, one clear purpose; avoid `utils` or `common` packages
4. **Table-driven tests** — use `[]struct{ name string; input; want }` pattern for all unit tests
5. **No globals** — pass dependencies explicitly via constructors; no `init()` for business logic

## Naming

- Packages: lowercase single word (`scoring`, `handler`, `user`)
- Exported: PascalCase (`CalculateScore`, `UserService`)
- Unexported: camelCase (`parseInput`, `validateAge`)
- Interfaces: `-er` suffix when single-method (`Reader`, `Scorer`); descriptive name otherwise (`UserRepository`)
- Files: snake_case (`user_handler.go`, `score_test.go`)
- Test files: `*_test.go` in the same package

## File Organization

- One primary type per file (`user.go` contains `User` struct and methods)
- Handler, service, and repository for each domain in separate packages
- Test helpers in `testutil/` package or `_test.go` files
- Keep `main.go` thin — only wiring and startup

## Error Handling

- Define sentinel errors in package scope: `var ErrNotFound = errors.New("not found")`
- Wrap with context at every layer: `fmt.Errorf("get user %d: %w", id, err)`
- Handlers map domain errors to HTTP status codes using `errors.Is()`
- Never `log.Fatal` except in `main.go`
