# Starting a New Project

When you start a new project, tell Claude **"set up a new project called X"** and it will create the structure below. Here is what gets created and why.

## File Checklist

```
my-project/
├── .gitignore              ← keep secrets and junk out of git
├── .claude/
│   └── settings.json       ← project-level hooks (e.g. kill app before build)
├── CLAUDE.md               ← filled-in copy of PROJECT-CLAUDE-TEMPLATE.md
├── ARCHITECTURE.md         ← source of truth for structure and phased plan
├── WORKLOG.md              ← running log: what changed and where, per session
├── TODO.md                 ← index only; real entries live in todo/
├── todo/
│   ├── bugs.md             ← bugs are priority 1
│   ├── features.md
│   └── tech-debt.md
├── CHANGELOG.md            ← user-facing version history
└── src/                    ← your actual code
    └── module-name/
        └── README.md       ← ≤15 lines: public types and their purpose
```

## .gitignore

Every project needs one. Claude will create this at project setup:

```gitignore
# OS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Thumbs.db
desktop.ini

# Editors
.vscode/
.idea/
*.swp
*.swo
*~
.vim/

# Environment & secrets — NEVER commit these
.env
.env.*
!.env.example
*.pem
*.key
*.p12
*.cert
secrets/
credentials.json

# Node
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.npm
.yarn/cache
dist/
build/
.next/
.nuxt/

# Python
__pycache__/
*.py[cod]
.venv/
venv/
env/
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
.ruff_cache/
htmlcov/
.coverage

# Go
vendor/
*.exe
*.test

# Rust
target/

# Java / Kotlin / Android
*.class
*.jar
*.aar
.gradle/
build/
local.properties

# Swift / Xcode
*.xcworkspace/xcuserdata/
*.xcodeproj/xcuserdata/
DerivedData/
*.ipa
*.dSYM.zip
*.dSYM
Pods/

# Logs & temp
*.log
*.tmp
*.temp
.cache/

# Claude Code — session state, not source
.claude/tasks/
.claude/sessions/
.claude/session-data/
.claude/shell-snapshots/
.claude/file-history/
```

## Standard Project Docs

**ARCHITECTURE.md** — the structural source of truth. Describes what the system does, how modules are organised, and the phased implementation plan. Claude reads this first when starting any non-trivial task. Template:

```markdown
# ARCHITECTURE.md — [Project Name]

## Overview
What this system does and for whom (2–3 sentences).

## Module Map
| Module | Path | Responsibility |
|--------|------|----------------|
| …      | …    | …              |

## Key Design Decisions
- Why X over Y (include the constraint or reason)

## Implementation Phases
### Phase 1: [Name]
Goal: …
DoD (Definition of Done): …

### Phase 2: [Name]
Goal: …
DoD: …
```

**WORKLOG.md** — append 2–3 bullets after every unit of work. This is how Claude picks up where it left off without re-reading the whole codebase.

```markdown
# WORKLOG

## 2026-05-11
- added input validation to UserService.create()
- fixed crash in AuthMiddleware when token is expired
- updated TODO: moved "session refresh" from features to in-progress
```

**TODO.md** — index only. Real entries live in `todo/` category files (capped at ~100 lines each; split when they grow). Bugs are always priority 1.

```markdown
# TODO

## Bugs (priority 1)
→ see todo/bugs.md

## Features
→ see todo/features.md

## Tech Debt
→ see todo/tech-debt.md
```

**CHANGELOG.md** — user-facing. One line per release. Claude appends here at the end of each phase.

```markdown
# CHANGELOG

## [Unreleased]
- …

## 0.1.0 — 2026-05-11
- Initial working build
```

## Project-Level .claude/settings.json

Every project should have `.claude/settings.json` for project-specific hooks. At minimum, create an empty scaffold:

```json
{
  "hooks": {}
}
```

Add hooks as you discover the need (e.g. auto-quit a running app before builds — see `setup/swift.md` for an example).

## Per-Module READMEs

Every folder under `src/` (or your equivalent) gets a `README.md` of ≤15 lines listing the public types and what they do. Claude reads this before opening any file in the folder. This prevents it from reading the whole codebase for every task.

```markdown
# auth/

Public types:
- `AuthService` — validates tokens, issues sessions
- `SessionStore` — in-memory token cache with TTL
- `AuthMiddleware` — Express middleware, reads Bearer header

Do not import from: `billing/` (circular dep risk)
```

## First Commit

```bash
git init
git add .gitignore
git commit -m "chore: add .gitignore"
git add CLAUDE.md ARCHITECTURE.md WORKLOG.md TODO.md CHANGELOG.md
git commit -m "docs: add project scaffold"
git add .claude/
git commit -m "chore: add project Claude config"
```

## Project CLAUDE.md Template

Copy `PROJECT-CLAUDE-TEMPLATE.md` (in this repo) to your project root as `CLAUDE.md` and fill it in. Keep it concise — long CLAUDE.md files get ignored; short ones get followed.

## Instincts (Advanced)

The `everything-claude-code` plugin supports *instincts* — short Markdown files in `.claude/instincts/` that encode project-specific behavioral patterns observed from git history. Unlike CLAUDE.md rules (which you write), instincts are derived from evidence: "52/52 commits follow this format" is more persuasive to Claude than "use this format."

Useful instincts to set up once a project has some history:

- Conventional commit format (scope mapping, no attribution)
- Folder-README-first navigation
- Tests co-committed with pure-logic changes
- WORKLOG update after every unit of work

Ask Claude: **"generate instincts from this repo's git history"** after you have 20+ commits. The `everything-claude-code:hookify` skill can also help derive instincts from session transcripts.
