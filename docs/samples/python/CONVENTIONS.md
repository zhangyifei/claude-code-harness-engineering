# Conventions

## Golden Principles

1. **Type hints everywhere** — all function signatures fully annotated; no bare `dict` or `list`; use `mypy --strict`
2. **Fail loudly** — raise domain-specific exceptions; never return `None` to signal errors
3. **Validate at boundaries** — Pydantic models parse all external input; internal code trusts typed data
4. **One module, one responsibility** — keep files under 300 lines; split when a module has multiple concerns
5. **Test behavior, not implementation** — prefer integration tests with real fixtures over mocks

## Naming

- Modules: snake_case (`data_loader.py`)
- Classes: PascalCase (`QuizParser`)
- Functions: snake_case (`calculate_score`)
- Constants: UPPER_SNAKE (`MAX_RETRIES`)
- Private: leading underscore (`_parse_row`)

## File Organization

- Mirror `src/` structure in `tests/`
- One class per file for models; group related helpers in a single module
- Shared fixtures live in `conftest.py` at the closest common ancestor directory

## Error Handling

- Define domain exceptions in `src/core/exceptions.py` (`NotFoundError`, `ValidationError`)
- Route handlers use FastAPI exception handlers to map domain exceptions to HTTP responses
- Never bare `except:` — always catch specific exception types
- Log context with structured logging before raising
