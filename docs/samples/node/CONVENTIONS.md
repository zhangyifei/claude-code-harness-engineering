# Conventions

## Golden Principles

1. **TypeScript strict mode** — no `any`, no `as unknown`, enable all strict checks
2. **Validate at boundaries** — parse external input with Zod at route handlers; internal code trusts typed data
3. **Explicit error handling** — never swallow errors; use typed Result patterns or throw structured errors
4. **One module, one responsibility** — keep files focused and under 300 lines
5. **Test behavior, not implementation** — test inputs and outputs; minimize mocks

## Naming

- Files: camelCase (`userService.ts`)
- Classes: PascalCase (`UserService`)
- Functions: camelCase (`calculateScore`)
- Constants: UPPER_SNAKE (`MAX_RETRIES`)
- Types/Interfaces: PascalCase, no `I` prefix (`UserPayload`, not `IUserPayload`)

## File Organization

- One exported thing per file when possible
- Co-locate tests: `src/services/scoring.ts` → `src/services/scoring.test.ts`
- Barrel exports (`index.ts`) only at package boundaries, not within modules

## Error Handling

- Route handlers catch and map errors to HTTP status codes
- Services throw domain-specific error classes (`NotFoundError`, `ValidationError`)
- Never `catch {}` — always log or rethrow
