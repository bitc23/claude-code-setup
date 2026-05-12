# Code Quality

## Immutability — Critical

Always create new objects; never mutate in place. Immutable data prevents hidden side effects and makes debugging easier.

## Core Principles

- **KISS**: simplest solution that works; optimize for clarity over cleverness
- **DRY**: extract when repetition is real, not speculative
- **YAGNI**: build what's needed now; refactor only when the pressure is real
- No half-finished abstractions, feature flags, or backwards-compat shims when you can just change the code

## Code Style

- Follow existing patterns before introducing new ones
- Prefer editing existing files over creating new ones
- Keep solutions simple — avoid over-engineering
- Avoid adding comments unless the logic is non-obvious
- No docstrings, type annotations, or error handling beyond what is necessary
- Remove unused code completely; no dead code, no `_` prefix workarounds
- No emojis in code, comments, or docs
- Prefer immutability — never mutate objects or arrays in place
- Use early returns to reduce nesting
- No `console.log` left in production code

## File Organization

Many small files over few large ones. Target 200–400 lines per file, 800 max. Organize by feature/domain, not type.

## Error Handling

Handle errors explicitly at every level. Never silently swallow.

- Catch specific exceptions; avoid bare `except Exception:` outside top-level handlers
- Use `logger.exception()` (not `logger.error()`) when catching exceptions, to preserve stack traces
- Always surface errors to the user in some form (message, log, or UI feedback)
- UI-facing code shows friendly messages; server-side logs full context with detail

## Naming

- Variables/functions: `camelCase`
- Booleans: `is`, `has`, `should`, `can` prefix
- Types/interfaces/components: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Custom hooks: `use` prefix + `camelCase`

## Code Smells to Avoid

- Deep nesting (>4 levels) — prefer early returns
- Magic numbers — use named constants
- Long functions (>50 lines) — split into focused pieces
