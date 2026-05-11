# claude-code-setup

A drop-in Claude Code setup: plugins, hooks, rules, and project scaffolding.

## What you get

- **Plugins**: `superpowers` (skills system) + `everything-claude-code` (200+ specialized agents and skills)
- **Status line**: color-coded context %, 5-hour session limit, weekly limit
- **SessionStart hook**: auto-invokes the skills system every session
- **Rules**: coding style, TDD, git workflow, dev workflow, code review, security, model selection, agent orchestration, memory system
- **Project scaffolding**: `.gitignore`, `WORKLOG.md`, `TODO.md`, `ARCHITECTURE.md`, per-module READMEs, session handoff patterns
- **Xcode section** *(optional)*: Swift style, Swift Testing, xcodebuild Terminal-crash protection

## Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Global config — put this at `~/.claude/CLAUDE.md` |
| `PROJECT-CLAUDE-TEMPLATE.md` | Template — copy to each new project root as `CLAUDE.md` |

## How to set up

1. Copy `CLAUDE.md` to `~/.claude/CLAUDE.md`
2. Open Claude Code and say: **"run first-time setup"**
3. Claude walks you through the manual plugin steps and configures everything else automatically

## Starting a new project

1. Copy `PROJECT-CLAUDE-TEMPLATE.md` to your repo root as `CLAUDE.md`
2. Fill in the blanks (project name, stack, build commands, constraints)
3. Tell Claude: **"set up a new project called X"** — it creates `ARCHITECTURE.md`, `WORKLOG.md`, `TODO.md`, `CHANGELOG.md`, `.gitignore`, and `.claude/settings.json`
