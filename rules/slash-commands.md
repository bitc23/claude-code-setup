# Custom Slash Commands

A custom slash command is a Markdown file in `.claude/commands/`. Anything you find yourself asking Claude to do repeatedly — "review this PR", "summarise the WORKLOG", "scaffold a new module" — is a candidate.

## Where to Put Them

- `~/.claude/commands/<name>.md` — available globally, in every project
- `<project>/.claude/commands/<name>.md` — only in this repo

The filename (without `.md`) becomes the command. `~/.claude/commands/review.md` → `/review`.

## Minimal Example

`~/.claude/commands/changelog.md`:

```markdown
---
description: Draft a CHANGELOG entry from this week's commits
---

Look at the last 7 days of git log on the current branch. Group commits by `type:` prefix (feat, fix, refactor, docs, chore). Write a concise CHANGELOG entry using the format already in `CHANGELOG.md`. Only show me the proposed entry — do not edit the file yet.
```

Invoke with `/changelog`. The body becomes the prompt.

## Frontmatter Fields

| Field | Purpose |
|-------|---------|
| `description` | One-line summary shown in `/help` and the command picker |
| `argument-hint` | Hint shown when the user starts typing `/<cmd>` (e.g. `<branch-name>`) |
| `model` | Pin the command to a specific model (e.g. `claude-haiku-4-5` for cheap one-shots) |
| `allowed-tools` | Restrict tool surface for this command |

## Arguments

Whatever the user types after `/<cmd>` is available as `$ARGUMENTS` in the prompt body:

```markdown
---
description: Generate release notes for a specific tag
argument-hint: <tag>
---

Find git log entries between $ARGUMENTS and the previous tag. Format as release notes.
```

Invoke with `/release-notes v0.3.0`.

## Bash Inside Commands

You can embed bash that runs *before* the prompt is sent, and inject the output:

```markdown
---
description: Summarise today's work
---

Today's commits:

!`git log --since=midnight --oneline`

Write a 3-bullet summary suitable for a standup.
```

The `!` ` ` ` ` ` ` ` block runs the command on disk and substitutes the output before Claude sees the prompt.

## When *Not* to Make a Slash Command

- Don't create a command for something you'll do twice. Wait until you've manually done it 3+ times.
- Don't bake project-specific paths into a global command — split into a global stub that calls a project-local command, or just keep it project-scoped.

## Related

- `rules/sub-agents.md` — for repeating *workflows* with isolated context, not just prompts
- `rules/orchestration.md` — for chained command → agent → skill patterns (rarely worth it for personal use)
