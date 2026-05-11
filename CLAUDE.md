# Claude Code Setup

Drop this file at `~/.claude/CLAUDE.md`. On first use, tell Claude **"run first-time setup"** and it will execute the steps below. After that, these rules apply automatically to every session.

---

## First-Time Setup

### Step A — You do once (manually)

**1. Install the superpowers plugin**

Open Claude Code and run:
```
/plugins install superpowers
```

**2. Install the everything-claude-code plugin**

First, register the marketplace. Add this to `~/.claude/settings.json` under the top-level object:

```json
"extraKnownMarketplaces": {
  "everything-claude-code": {
    "source": {
      "source": "git",
      "url": "https://github.com/affaan-m/everything-claude-code.git"
    }
  }
}
```

Then in Claude Code run:
```
/plugins install everything-claude-code
```

**3. Restart Claude Code** — plugins are loaded at startup.

---

### Step B — Claude executes (run once, then never again)

Read `~/.claude/settings.json`, then **merge** the block below into it (preserve existing keys, don't overwrite). Confirm when done.

```json
{
  "env": {
    "ECC_GATEGUARD": "off"
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true,
    "superpowers@claude-plugins-official": true
  },
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\":{\"hookEventName\":\"SessionStart\",\"additionalContext\":\"At the start of this session, invoke the `superpowers:using-superpowers` skill via the Skill tool before any other work. It establishes how to discover and use the rest of the superpowers skill set.\"}}'"
          }
        ]
      }
    ]
  },
  "statusLine": {
    "type": "command",
    "command": "input=$(cat); ctx=$(echo \"$input\" | jq -r '.context_window.used_percentage // empty'); five_pct=$(echo \"$input\" | jq -r '.rate_limits.five_hour.used_percentage // empty'); five_reset=$(echo \"$input\" | jq -r '.rate_limits.five_hour.resets_at // empty'); week_pct=$(echo \"$input\" | jq -r '.rate_limits.seven_day.used_percentage // empty'); week_reset=$(echo \"$input\" | jq -r '.rate_limits.seven_day.resets_at // empty'); now=$(date +%s); RED=$(printf '\\033[1;31m'); YEL=$(printf '\\033[33m'); GRN=$(printf '\\033[32m'); RST=$(printf '\\033[0m'); ci=0; fi5=0; wki=0; [ -n \"$ctx\" ] && ci=$(printf '%.0f' \"$ctx\"); [ -n \"$five_pct\" ] && fi5=$(printf '%.0f' \"$five_pct\"); [ -n \"$week_pct\" ] && wki=$(printf '%.0f' \"$week_pct\"); breach=0; [ \"$ci\" -ge 75 ] && breach=1; [ \"$fi5\" -ge 75 ] && breach=1; [ \"$wki\" -ge 75 ] && breach=1; pick(){ if [ \"$1\" -ge 90 ]; then printf '%s' \"$RED\"; elif [ \"$1\" -ge 75 ]; then printf '%s' \"$YEL\"; elif [ \"$breach\" -eq 0 ]; then printf '%s' \"$GRN\"; fi; }; out=\"\"; if [ -n \"$ctx\" ]; then c=$(pick \"$ci\"); hint=\"\"; [ \"$ci\" -ge 50 ] && hint=\" (try /compact)\"; out=\"${c}ctx:${ci}%${hint}${RST}\"; fi; if [ -n \"$five_pct\" ]; then c=$(pick \"$fi5\"); five_str=\"${c}5h session limit: ${fi5}%\"; if [ -n \"$five_reset\" ]; then diff=$(( five_reset - now )); hrs=$(( diff / 3600 )); mins=$(( (diff % 3600) / 60 )); if [ \"$hrs\" -gt 0 ]; then t=\"${hrs}h\"; else t=\"${mins}m\"; fi; five_str=\"$five_str (resets in $t)\"; fi; five_str=\"${five_str}${RST}\"; out=\"$out $five_str\"; fi; if [ -n \"$week_pct\" ]; then c=$(pick \"$wki\"); week_str=\"${c}weekly limit: ${wki}%\"; if [ -n \"$week_reset\" ]; then diff=$(( week_reset - now )); days=$(( diff / 86400 )); hrs=$(( (diff % 86400) / 3600 )); if [ \"$days\" -gt 0 ]; then t=\"${days}d\"; else t=\"${hrs}h\"; fi; week_str=\"$week_str (resets in $t)\"; fi; week_str=\"${week_str}${RST}\"; out=\"$out $week_str\"; fi; printf '%s' \"$out\""
  }
}
```

The status line shows context window %, 5-hour session limit, and weekly limit — color-coded green/yellow/red.

---

## Starting a New Project

When you start a new project, tell Claude **"set up a new project called X"** and it will create the structure below. Here is what gets created and why.

### File Checklist

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

### .gitignore

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

### Standard Project Docs

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

### Project-Level .claude/settings.json

Every project should have `.claude/settings.json` for project-specific hooks. At minimum, create an empty scaffold:

```json
{
  "hooks": {}
}
```

Add hooks as you discover the need (e.g. auto-quit a running app before builds — see the Xcode section for an example).

### Per-Module READMEs

Every folder under `src/` (or your equivalent) gets a `README.md` of ≤15 lines listing the public types and what they do. Claude reads this before opening any file in the folder. This prevents it from reading the whole codebase for every task.

```markdown
# auth/

Public types:
- `AuthService` — validates tokens, issues sessions
- `SessionStore` — in-memory token cache with TTL
- `AuthMiddleware` — Express middleware, reads Bearer header

Do not import from: `billing/` (circular dep risk)
```

### First Commit

```bash
git init
git add .gitignore
git commit -m "chore: add .gitignore"
git add CLAUDE.md ARCHITECTURE.md WORKLOG.md TODO.md CHANGELOG.md
git commit -m "docs: add project scaffold"
git add .claude/
git commit -m "chore: add project Claude config"
```

---

## Code Quality

### Immutability — Critical

Always create new objects; never mutate in place. Immutable data prevents hidden side effects and makes debugging easier.

### Core Principles

- **KISS**: simplest solution that works; optimize for clarity over cleverness
- **DRY**: extract when repetition is real, not speculative
- **YAGNI**: build what's needed now; refactor only when the pressure is real
- No half-finished abstractions, feature flags, or backwards-compat shims when you can just change the code

### File Organization

Many small files over few large ones. Target 200–400 lines per file, 800 max. Organize by feature/domain, not type.

### Error Handling

Handle errors explicitly at every level. Never silently swallow. UI-facing code shows friendly messages; server-side logs full context with detail.

### Naming

- Variables/functions: `camelCase`
- Booleans: `is`, `has`, `should`, `can` prefix
- Types/interfaces/components: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Custom hooks: `use` prefix + `camelCase`

### Code Smells to Avoid

- Deep nesting (>4 levels) — prefer early returns
- Magic numbers — use named constants
- Long functions (>50 lines) — split into focused pieces

---

## Testing

**TDD is mandatory:**
1. Write test first (RED — it must fail before you write anything)
2. Write minimal implementation (GREEN — make the test pass)
3. Refactor (IMPROVE)
4. Verify 80%+ coverage

**Required test types:** unit, integration, E2E for critical user flows.

**Test structure — AAA:**
```
// Arrange — set up state
// Act — call the thing
// Assert — check the result
```

**Naming:** describe the behavior, not the implementation.
- `"returns empty array when no results match query"`
- `"throws error when API key is missing"`

---

## Git Workflow

```
<type>: <description>

<optional body>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

**Never** add `Co-Authored-By: Claude…` or "Generated with Claude Code" footers — not in commits, not in PRs, not anywhere.

**One change, one commit.** A bug fix doesn't need surrounding cleanup; a one-shot operation doesn't need a helper.

**PRs:** run `git diff [base]...HEAD` to see all changes across all commits before writing the PR summary. Include a test plan checklist.

---

## Development Workflow

For every non-trivial task:

1. **Research first** — search GitHub and package registries before writing new code; prefer proven libraries over hand-rolled solutions
2. **Plan** — use `planner` agent for complex features; break into phases with clear milestones
3. **TDD** — write tests before implementation
4. **Review** — use `code-reviewer` agent immediately after writing code
5. **Commit** — conventional message, one logical change per commit

Do not add features, refactor, or introduce abstractions beyond what the task requires. Three similar lines is better than a premature abstraction.

---

## Code Review

Use the `code-reviewer` agent **after every code change**, before committing.

| Level | Meaning | Action |
|-------|---------|--------|
| CRITICAL | Security vulnerability or data loss | Block — fix before merge |
| HIGH | Bug or significant quality issue | Warn — should fix |
| MEDIUM | Maintainability concern | Consider fixing |
| LOW | Style/minor | Optional |

Use `security-reviewer` when touching: auth, user input, database queries, file ops, external APIs, crypto, or payments.

---

## Security

Before any commit, verify:
- No hardcoded secrets (API keys, tokens, passwords)
- All user inputs validated at system boundaries
- Parameterized queries (no string concatenation in SQL)
- XSS prevention (sanitized HTML output)
- Errors don't leak sensitive data

If a security issue is found: stop, use `security-reviewer`, fix before continuing, rotate any exposed secrets.

---

## Performance & Model Selection

| Model | Use for |
|-------|---------|
| Haiku 4.5 | Lightweight/frequent agents, worker agents in multi-agent systems |
| Sonnet 4.6 | Main development work, complex coding, orchestration |
| Opus 4.7 | Architectural decisions, deep research, maximum reasoning |

Avoid the last 20% of context window for large refactors or multi-file features — use `/compact` when context climbs above ~50%.

---

## Agents & Skills

**Use proactively — no need to wait to be asked:**
- Complex feature → `planner` agent
- Code written or modified → `code-reviewer` agent
- Bug fix or new feature → `tdd-guide` agent
- Architectural decision → `architect` agent
- Build fails → `build-error-resolver` agent
- Security-sensitive change → `security-reviewer` agent

**Run independent agents in parallel.** Batch unrelated tasks in the same message instead of sequential calls.

**Invoke relevant skills before any response.** Even a 1% chance a skill applies means invoke it. The `superpowers:using-superpowers` skill (loaded via the SessionStart hook) explains how to discover skills at runtime.

---

## Memory System

Claude Code maintains persistent memory at `~/.claude/projects/<encoded-path>/memory/`. Build it up over time so future sessions don't start cold.

**Four memory types:**
- `user/` — who you're working with, their role, preferences, expertise level
- `feedback/` — corrections and confirmed approaches; lead with the rule, add **Why:** and **How to apply:** lines
- `project/` — ongoing goals, decisions, deadlines (convert relative dates to absolute)
- `reference/` — pointers to external systems (Linear boards, Grafana dashboards, Slack channels)

`MEMORY.md` is the index — one line per entry, under 200 lines total. Each memory lives in its own file with YAML frontmatter (`name`, `description`, `type`).

**Save immediately** when the user corrects your approach, confirms something worked, or shares context about the project or themselves.

---

## Session & Project Discipline

Good session discipline is what makes Claude useful across many sessions on the same codebase — without it, every session starts cold and wastes time re-reading files.

### At Session Start

1. Run `git pull` — never report status or TODOs from a stale checkout
2. Read `WORKLOG.md` — understand what was last done
3. Read `TODO.md` — know current outstanding work; bugs are priority 1
4. Read the memory index (`~/.claude/projects/…/memory/MEMORY.md`) for any relevant context

### During Work

**Stay in scope.** Work on the one thing you were asked. If you discover something that needs fixing elsewhere, add it to `todo/tech-debt.md` and continue. Don't fix unrelated things silently.

**List cross-cutting changes upfront.** If a task genuinely requires touching multiple modules, name every file you intend to change before touching any of them. Give the user a chance to redirect.

**Read only what you need.** For any folder you haven't visited this session, read its `README.md` first. Only open source files whose README tells you you need them. This prevents re-reading the entire codebase every session.

**Phased development.** Work through the phases defined in `ARCHITECTURE.md` in order. Each phase ends with a runnable, testable build. Don't start phase N+1 until phase N's Definition of Done is met.

**Library/platform first.** Before writing any helper or utility, check if the language standard library or platform already provides it. Hand-rolled solutions are maintenance debt.

### After Each Unit of Work

Append 2–3 bullets to `WORKLOG.md`:
```
## 2026-05-11
- fixed null-deref in TokenValidator when header is missing
- added unit test covering the empty-header case
```

Update `TODO.md` / `todo/*.md` if you completed a carry-over item or found a new one.

### When Context Gets Low (~50%+)

Use `/compact` to compress conversation history. When approaching the end of context, write a handoff note:

```
## Handoff — 2026-05-11 16:30
Done: implemented email validation in UserService; tests passing
Blocked: integration test needs a running DB — deferred
Next: wire up the validation to the API route in routes/users.ts
Gotcha: the `validateEmail` helper in utils/ has a known false-positive on .museum TLDs — do not remove the regex workaround
```

This note goes in `WORKLOG.md`. The next session resumes from it without re-reading everything.

### Before Reporting Work Complete

Invoke `superpowers:verification-before-completion` to catch gaps — missed edge cases, incomplete tests, forgotten WORKLOG updates.

---

## Project CLAUDE.md Template

Copy `PROJECT-CLAUDE-TEMPLATE.md` (in this repo) to your project root as `CLAUDE.md` and fill it in. Keep it concise — long CLAUDE.md files get ignored; short ones get followed.

---

## Xcode / Swift (Optional)

Skip this section if you're not working on Apple platform projects.

### Swift Coding Style

- Prefer `let` over `var` — only use `var` when the compiler requires it
- Use `struct` with value semantics by default; `class` only when identity or reference semantics are needed
- Follow [Apple API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/): clarity at point of use, omit needless words
- Typed throws (Swift 6+): `throws(MyError)` not bare `throws`
- Swift 6 strict concurrency: `Sendable` types crossing isolation boundaries, `actor` for shared mutable state, structured concurrency (`async let`, `TaskGroup`) over GCD

### Swift Testing

Use **Swift Testing** (`import Testing`) for all new tests — not XCTest (XCTest only for UI tests):

```swift
@Test("validates email on creation")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}

@Test("handles multiple formats", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

Each test gets fresh state — set up in `init`, tear down in `deinit`. No shared mutable state.

### Swift Security

- Use **Keychain Services** for sensitive data (tokens, passwords) — never `UserDefaults`
- Use environment variables or `.xcconfig` files for build-time secrets
- Never hardcode secrets — decompilation extracts them trivially
- App Transport Security (ATS) is on by default — do not disable it
- Validate all data from external sources (APIs, deep links, pasteboard) before use

### macOS Native-First

Before writing any helper or utility type, check if macOS already provides it:

| Need | Use this | Don't write |
|------|----------|-------------|
| Render loop | `CADisplayLink` | Custom `Timer` |
| Short audio | `AVAudioPlayer` | Custom mixing |
| Persistence | `@AppStorage` / `UserDefaults` | Custom INI/JSON |
| JSON | `Codable` + `JSONDecoder` | Hand-rolled parser |
| Caching | `NSCache` | Custom LRU |
| State observation | `@Observable` macro | Notification-center plumbing |
| Unit tests | Swift Testing (`@Test`, `#expect`) | XCTest for new code |

When in doubt: macOS probably already provides it. Check Apple developer docs first.

### xcodebuild Hook — Critical

**Problem:** Running `xcodebuild test` with UI tests from Terminal.app crashes it on macOS 26.x — a null-deref in `XCTAutomationSupport` triggered when the test harness injects into the foreground app (Terminal).

**Solution:** Create `~/.claude/hooks/block-xcodebuild-test.sh`:

```bash
#!/usr/bin/env bash
# Blocks `xcodebuild test` without scoping — crashes Terminal on macOS 26.x.
# Safe: build, build-for-testing, test-without-building, scoped test runs.

set -u

input="$(cat)"
cmd="$(printf '%s' "$input" | jq -r '.tool_input.command // empty')"

case "$cmd" in
  *xcodebuild*) ;;
  *) exit 0 ;;
esac

stripped="$(printf '%s' "$cmd" | sed -E 's/build-for-testing|test-without-building//g')"

printf '%s' "$stripped" | grep -qE '\bxcodebuild\b' || exit 0
printf '%s' "$stripped" | grep -qE '\btest\b'       || exit 0

if printf '%s' "$cmd" | grep -qE -- '-(only|skip)-testing:'; then
  exit 0
fi

cat >&2 <<'MSG'
Blocked: `xcodebuild test` without scoping crashes Terminal.app on macOS 26.x
(XCTAutomationSupport bug when UI tests run from a Terminal-hosted shell).

Use one of:
  xcodebuild build                           # build-only verification
  xcodebuild build-for-testing               # compile tests without running
  xcodebuild test -only-testing:<Bundle>     # restrict to unit tests
  xcodebuild test -skip-testing:<UITests>    # exclude UI tests
  xcodebuild test-without-building           # run a pre-built bundle

For UI tests: ask the user to run them via Xcode's play button.
MSG
exit 2
```

Make it executable and wire it up:
```bash
chmod +x ~/.claude/hooks/block-xcodebuild-test.sh
```

Add to `~/.claude/settings.json` hooks (merge into existing `PreToolUse` array):
```json
"PreToolUse": [
  {
    "matcher": "Bash",
    "hooks": [
      {
        "type": "command",
        "command": "$HOME/.claude/hooks/block-xcodebuild-test.sh",
        "timeout": 5
      }
    ]
  }
]
```

> Note: the hook matches on the raw command string — any Bash command containing both "xcodebuild" and "test" as words is caught, including file paths. This is conservative by design.

### Project-Level Build Hook

Add to your project's `.claude/settings.json` to auto-quit the running app before builds — prevents codesign collisions when the app is already running:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command // \"\"' | grep -qE 'xcodebuild|swift test' && pkill -x YourApp 2>/dev/null; exit 0"
          }
        ]
      }
    ]
  }
}
```

Replace `YourApp` with your app's process name (as shown in Activity Monitor).
