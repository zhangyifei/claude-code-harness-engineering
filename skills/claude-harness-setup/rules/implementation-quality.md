---
description: Prevent hollow implementations — code must work for real inputs, not just test cases
paths: "**/*.{ts,tsx,js,jsx,py,go,rs,java,rb}"
---

# Implementation Quality

## Absolutely Prohibited

| Prohibited | Example | Why |
|-----------|---------|-----|
| Hardcoded returns | Return a dict of test expected values | Breaks on any other input |
| Stub implementations | `return null`, `return []`, `pass` | Not actually implemented |
| Test-case-only logic | if/elif chain matching each test input | No generalization |
| Error swallowing | `except: return None` | Hides real problems |

## Self-Check Before Completing

- Would this work for inputs NOT in the tests?
- Can another developer understand the logic by reading the code?
- Am I handling edge cases (empty input, null, boundary values)?
- Am I surfacing errors, not hiding them?

## When Stuck

If you can't implement something correctly, report honestly:
- What you tried
- What's blocking you
- Proposed alternatives

Never disguise a stub as a real implementation.
