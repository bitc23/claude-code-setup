# Session & Project Discipline

Good session discipline is what makes Claude useful across many sessions on the same codebase — without it, every session starts cold and wastes time re-reading files.

## At Session Start

1. Run `git pull` — never report status or TODOs from a stale checkout
2. Read `WORKLOG.md` — understand what was last done
3. Read `TODO.md` — know current outstanding work; bugs are priority 1
4. Read the memory index (`~/.claude/projects/…/memory/MEMORY.md`) for any relevant context

## During Work

**Stay in scope.** Work on the one thing you were asked. If you discover something that needs fixing elsewhere, add it to `todo/tech-debt.md` and continue. Don't fix unrelated things silently.

**List cross-cutting changes upfront.** If a task genuinely requires touching multiple modules, name every file you intend to change before touching any of them. Give the user a chance to redirect.

**Read only what you need.** For any folder you haven't visited this session, read its `README.md` first. Only open source files whose README tells you you need them. This prevents re-reading the entire codebase every session.

**Phased development.** Work through the phases defined in `ARCHITECTURE.md` in order. Each phase ends with a runnable, testable build. Don't start phase N+1 until phase N's Definition of Done is met.

**Library/platform first.** Before writing any helper or utility, check if the language standard library or platform already provides it. Hand-rolled solutions are maintenance debt.

## After Each Unit of Work

Append 2–3 bullets to `WORKLOG.md`:

```
## 2026-05-11
- fixed null-deref in TokenValidator when header is missing
- added unit test covering the empty-header case
```

Update `TODO.md` / `todo/*.md` if you completed a carry-over item or found a new one.

## When Context Gets Low (~50%+)

Use `/compact` to compress conversation history. When approaching the end of context, write a handoff note:

```
## Handoff — 2026-05-11 16:30
Done: implemented email validation in UserService; tests passing
Blocked: integration test needs a running DB — deferred
Next: wire up the validation to the API route in routes/users.ts
Gotcha: the `validateEmail` helper in utils/ has a known false-positive on .museum TLDs — do not remove the regex workaround
```

This note goes in `WORKLOG.md`. The next session resumes from it without re-reading everything.

## Before Reporting Work Complete

Invoke `superpowers:verification-before-completion` to catch gaps — missed edge cases, incomplete tests, forgotten WORKLOG updates.
