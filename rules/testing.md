# Testing

**TDD is mandatory:**

1. Write test first (RED — it must fail before you write anything)
2. Write minimal implementation (GREEN — make the test pass)
3. Refactor (IMPROVE)
4. Verify 80%+ coverage

**Required test types:** unit, integration, E2E for critical user flows.

## Principles

- Write tests that cover behavior, not implementation details
- Prefer function-based tests over class-based test suites
- Avoid fixed-duration sleeps in async tests; use events or `fail_after(timeout)` instead
- Run tests before committing

## Test Structure — AAA

```
// Arrange — set up state
// Act — call the thing
// Assert — check the result
```

## Naming

Describe the behavior, not the implementation:

- `"returns empty array when no results match query"`
- `"throws error when API key is missing"`
