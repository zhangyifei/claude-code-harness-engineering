---
description: Prevent test tampering — fix implementation, not tests
paths: "**/*.{test,spec}.{ts,tsx,js,jsx,py,go}, **/test/**/*.*, **/tests/**/*.*,  **/__tests__/**/*.*"
---

# Test Quality Protection

## Absolutely Prohibited

| Prohibited | Example | Correct Action |
|-----------|---------|----------------|
| Skip/only tests | `it.skip(...)`, `describe.only(...)` | Fix the implementation |
| Delete assertions | Remove `expect(x).toBe(y)` | Understand why it fails, fix implementation |
| Weaken assertions | Change expected value to match wrong output | Verify the spec, fix implementation |
| Delete test cases | Remove a failing test | Fix implementation to match spec |
| Excessive mocking | Mock the thing you're testing | Use minimal, targeted mocks |

## When a Test Fails

```
Test fails
  -> Read the error, understand WHY
  -> Is the implementation wrong? -> Fix implementation
  -> Is the test wrong? -> ASK THE USER before changing the test
```

## Also Prohibited

- Adding `continue-on-error: true` to CI
- Lowering coverage thresholds
- Disabling lint rules to make tests pass
- Using `--no-verify` to skip pre-commit hooks
