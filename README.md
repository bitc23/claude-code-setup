# claude-code-setup

A drop-in `CLAUDE.md` that sets up Claude Code with plugins, hooks, and best-practice rules.

## What you get

- **Plugins**: `superpowers` (skills system) + `everything-claude-code` (200+ specialized agents)
- **Status line**: color-coded context %, 5-hour session limit, weekly limit
- **SessionStart hook**: auto-invokes the `superpowers:using-superpowers` skill every session
- **Rules**: coding style, TDD, git workflow, code review, security, model selection, agent orchestration, memory system
- **Xcode section** *(optional)*: Swift coding style, Swift Testing, xcodebuild Terminal-crash protection

## How to use

1. Copy `CLAUDE.md` to `~/.claude/CLAUDE.md`
2. Open Claude Code and say: **"run first-time setup"**
3. Claude will walk you through the manual plugin steps and configure the rest automatically

## After setup

Drop a project-level `CLAUDE.md` in each repo root using the template at the bottom of the file. Keep it short — long CLAUDE.md files get ignored, short ones get followed.
