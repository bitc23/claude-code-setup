# claude-code-setup

A drop-in Claude Code setup: a slim global CLAUDE.md plus on-demand `rules/` and `setup/` files, and a project scaffolding template.

> **Disclaimer.** This is a personal project and a work in progress. It reflects how *I* like to use Claude Code on my own machines — not a vetted, comprehensive, or professional reference. Conventions here are opinionated, the setup evolves often, and pieces of it may be wrong, stale, or only suit my own workflow. Use it as a starting point, not as authoritative guidance.

## What you get

- **A slim core CLAUDE.md** — only the always-on rules and a pointer index. Keeps per-session token cost low.
- **`rules/`** — detailed rules read on demand: code quality, testing, git, security, workflow, memory, session discipline, tips, CLI flags, hooks reference, slash commands, sub-agents, MCP, orchestration.
- **`setup/`** — heavy setup material loaded only when actively setting things up: first-time setup, project scaffolding, security hooks, AgentShield weekly scan, nightly memory digest, Xcode/Swift rules + the global `block-xcodebuild-test.sh` hook.
- **`PROJECT-CLAUDE-TEMPLATE.md`** — a per-project template you fill in for each new repo.
- **Status line** — color-coded context %, 5-hour session limit, weekly limit (configured during first-time setup).

The `superpowers` plugin is the only required plugin. `everything-claude-code` is optional and disabled by default — it injects ~4,000 tokens per session, only worth it if you actually use >20% of its skills.

## Layout

```
claude-code-setup/
├── CLAUDE.md                        # slim core: always-on rules + index
├── PROJECT-CLAUDE-TEMPLATE.md       # per-project template
├── rules/                           # on-demand
│   ├── code-quality.md
│   ├── testing.md
│   ├── git.md
│   ├── security.md
│   ├── workflow.md
│   ├── memory.md
│   ├── session-discipline.md
│   ├── tips.md
│   ├── cli-flags.md
│   ├── hooks-reference.md
│   ├── slash-commands.md
│   ├── sub-agents.md
│   ├── mcp.md
│   └── orchestration.md
└── setup/                           # only read when setting up
    ├── first-time-setup.md
    ├── project-scaffolding.md
    ├── security-hooks.md
    ├── agentshield-weekly.md
    ├── memory-digest.md
    └── swift.md
```

## How to set up

1. Copy this repo's `CLAUDE.md`, `rules/`, and `setup/` into `~/.claude/`:

   ```bash
   cp CLAUDE.md ~/.claude/CLAUDE.md
   cp -r rules ~/.claude/rules
   cp -r setup ~/.claude/setup
   ```

2. Fill in the **User Profile** section at the top of `~/.claude/CLAUDE.md`.
3. Open Claude Code and say: **"run first-time setup"**. Claude will read `setup/first-time-setup.md` and walk through the manual plugin steps, then merge the required `settings.json` block.

## Starting a new project

1. Copy `PROJECT-CLAUDE-TEMPLATE.md` to the project root as `CLAUDE.md` and fill in the blanks (project name, stack, build commands, constraints).
2. Tell Claude: **"set up a new project called X"** — Claude reads `setup/project-scaffolding.md` and creates `ARCHITECTURE.md`, `WORKLOG.md`, `TODO.md`, `CHANGELOG.md`, `.gitignore`, and `.claude/settings.json`.
