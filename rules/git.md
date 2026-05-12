# Git Workflow

## Commit Message Format

```
<type>: <description>

<optional body>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

## Rules

- **Never** commit directly to `main` — use branches and PRs
- **Never** add `Co-Authored-By: Claude…` or "Generated with Claude Code" footers — not in commits, not in PRs, not anywhere
- **One change, one commit.** A bug fix doesn't need surrounding cleanup; a one-shot operation doesn't need a helper
- Fix order when resolving issues: formatting first, then type errors, then linting

## Pull Requests

Run `git diff [base]...HEAD` to see all changes across all commits before writing the PR summary. Include a test plan checklist.
