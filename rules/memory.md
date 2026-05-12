# Memory System

Claude Code maintains persistent memory at `~/.claude/projects/<encoded-path>/memory/`. Build it up over time so future sessions don't start cold.

## Four Memory Types

- `user/` — who you're working with, their role, preferences, expertise level
- `feedback/` — corrections and confirmed approaches; lead with the rule, add **Why:** and **How to apply:** lines
- `project/` — ongoing goals, decisions, deadlines (convert relative dates to absolute)
- `reference/` — pointers to external systems (Linear boards, Grafana dashboards, Slack channels)

## Index

`MEMORY.md` is the index — one line per entry, under 200 lines total. Each memory lives in its own file with YAML frontmatter (`name`, `description`, `type`).

## When to Save

**Save immediately** when the user:

- Corrects your approach
- Confirms a non-obvious approach worked
- Shares context about the project, the team, or themselves

## Mid-Session Shortcut

Type `# remember: <fact>` mid-conversation to drop a quick note into memory without breaking flow.

## Cautions

- Memory decays fast. Verify a stored fact against current reality before acting on it
- Don't store anything derivable from the code, git log, or CLAUDE.md
- Don't store ephemeral session state (in-progress work, current task)
- Don't store solution recipes — the fix is in the code; the commit message has the why
