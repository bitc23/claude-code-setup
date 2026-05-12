# Claude Code Setup

Drop this file at `~/.claude/CLAUDE.md` and copy the `rules/` and `setup/` directories alongside it. You should end up with:

```
~/.claude/
├── CLAUDE.md       ← this file (loaded every session)
├── rules/          ← referenced; read on demand
└── setup/          ← referenced; read only during setup tasks
```

On first use, tell Claude **"run first-time setup"** and it will walk through `setup/first-time-setup.md`. After that, the rules below apply automatically to every session.

The `rules/` and `setup/` files are *not* auto-loaded — they're referenced by name in the index sections below, and Claude reads the relevant ones when needed. This keeps the per-session token cost low.

---

## User Profile

Fill in once when you copy this to `~/.claude/CLAUDE.md`. Anything in here applies to every session.

- GitHub: **<your-github>** — email `<your-email>`
- Hardware: <e.g. iMac primary + MacBook synced>
- Secrets storage: <e.g. Apple Keychain only>
- Personal stack: <e.g. Terminal.app, Xcode, Claude Code under `~/Developer/`>
- Work stack: <e.g. VS Code, kept strictly separate from personal>

---

## Always-On Rules

The minimum a session needs to know without reading any other file. Detailed expansions live in `~/.claude/rules/`.

**Code style**

- Follow existing patterns. Edit existing files; don't create new ones unless required.
- No emojis in code, comments, docs, or commits.
- Prefer immutability — never mutate in place.
- → full: `rules/code-quality.md`

**Testing**

- TDD: red → green → refactor. Tests describe behaviour, not implementation.
- → full: `rules/testing.md`

**Git**

- Conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, `perf:`, `ci:`).
- Never commit to `main`. Use branches and PRs.
- Never add `Co-Authored-By: Claude` or similar attribution — not in commits, not in PRs, not anywhere.
- → full: `rules/git.md`

**Security**

- Personal infra: security is paramount. Default to private repos, never `curl … | sh`, secrets in Keychain only.
- Validate all external input at boundaries. Parameterized queries only.
- → full: `rules/security.md`

**Behaviour**

- Responses short and concise. No emojis unless explicitly asked.
- Never give time estimates.
- Stay in scope: don't fix unrelated things silently.
- Ask before making significant architectural changes or touching files outside the stated scope.
- Strict separation of personal (`~/Developer/`) and work tooling. Never mix configs, repos, or identities across that boundary.

**Workflow**

- Start every non-trivial task with **Plan Mode** (Shift+Tab).
- Run independent agents in parallel.
- Use `/compact` when context hits 50%.
- → full: `rules/workflow.md`, `rules/session-discipline.md`

---

## Detailed Rules — `~/.claude/rules/`

Loaded on demand. Read whichever is relevant to the current task.

| File | Purpose |
|------|---------|
| `rules/code-quality.md` | KISS/DRY/YAGNI, file org, immutability, naming, error handling |
| `rules/testing.md` | TDD discipline, AAA structure, naming |
| `rules/git.md` | Commit format, PR workflow |
| `rules/security.md` | Personal-infra security posture, code-level rules |
| `rules/workflow.md` | Dev workflow, model selection, code review, cross-model usage |
| `rules/memory.md` | Persistent memory at `~/.claude/projects/.../memory/` |
| `rules/session-discipline.md` | Start/middle/end of session, handoff notes |
| `rules/tips.md` | Slash commands, keybindings, plan mode, headless usage |
| `rules/cli-flags.md` | Claude Code CLI flag reference |
| `rules/hooks-reference.md` | Hook event one-pager |
| `rules/slash-commands.md` | How to author a custom `/command` |
| `rules/sub-agents.md` | How to author a custom sub-agent |
| `rules/mcp.md` | MCP server setup |
| `rules/orchestration.md` | Multi-agent orchestration (rarely worth it) |

---

## Setup — `~/.claude/setup/`

Loaded only when you're actually setting things up.

| File | When to read |
|------|-------------|
| `setup/first-time-setup.md` | When the user says "run first-time setup" — Step A (manual plugin installs) + Step B (settings.json merge) + token-budget guidance |
| `setup/project-scaffolding.md` | When the user says "set up a new project called X" — file checklist + standard docs |
| `setup/security-hooks.md` | When wiring `security-check.py` + `stop-check.py` into a project |
| `setup/agentshield-weekly.md` | When installing the weekly launchd security scan |
| `setup/memory-digest.md` | When scaffolding the nightly memory digest |
| `setup/swift.md` | Swift / Xcode / macOS rules + the global `block-xcodebuild-test.sh` hook |

---

## Project CLAUDE.md Template

When starting a new project, copy `PROJECT-CLAUDE-TEMPLATE.md` (in this repo) to the project root as `CLAUDE.md` and fill it in. Keep project CLAUDE.md files concise — long ones get ignored; short ones get followed.
