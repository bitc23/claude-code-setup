# CLAUDE.md — [Project Name]

[One sentence: what this does and who it's for. E.g. "A macOS menu-bar app that tracks daily water intake for health-conscious users."]

**Read `ARCHITECTURE.md` first.** It is the source of truth for project structure, module boundaries, and the phased implementation plan.

---

## How to Work in This Codebase

1. **Read `WORKLOG.md` and `TODO.md` at the start of every session.** WORKLOG is what got done; TODO is the current outstanding work. Bugs in `todo/bugs.md` are priority 1.
2. **Read a folder's `README.md` before opening any files in it.** Each module has a ≤15-line README listing public types. Only open files whose README tells you you need them.
3. **Stay in the folder you were asked to work in** unless cross-cutting changes are genuinely required. If they are, list every file you intend to change before touching any of them.
4. **Append 2–3 bullets to `WORKLOG.md` after finishing each unit of work** — what changed and where, no prose.
5. **One change, one commit.** Format: `type(scope): summary`. Never add Co-Authored-By or attribution trailers.

---

## Stack & Constraints

- **Language/runtime:** [e.g. Swift 5.10+ / Python 3.12+ / Node 22+]
- **Framework:** [e.g. AppKit / FastAPI / Next.js]
- **Minimum target:** [e.g. macOS 26.0 / iOS 18 / Node LTS]
- **Hard constraints:**
  - [e.g. No third-party dependencies in v1]
  - [e.g. All UI on the main thread]
  - [e.g. No network calls; offline only]

---

## How to Build & Run

```bash
# Build
[build command]

# Run tests
[test command]

# Run / dev server
[run command]
```

[Any gotchas before running, e.g. "Quit any running instance of the app first to avoid codesign collisions: `pkill -x AppName`"]

---

## Key Conventions

- [Domain-specific naming that deviates from the global rules, e.g. "State machine events are named in past tense: `didEnterIdle`, `didStartWalking`"]
- [Important files/folders to read first, e.g. "Always read `src/engine/README.md` before touching engine code"]
- [Any patterns locked in for this project, e.g. "Animation data lives in `manifest.json`, never hardcoded in source"]

---

## What Not to Do

- [Hard limits that would break the project or violate product decisions]
- [Things that look tempting but are explicitly off the table, e.g. "No polling faster than 5 Hz", "No SwiftUI in the main window — AppKit only"]
- [Known footguns, e.g. "Do not remove the retry logic in NetworkClient — it handles a known flakiness in the upstream API"]

---

## Source-of-Truth Ordering

When `ARCHITECTURE.md`, this file, and any other docs disagree:

1. `ARCHITECTURE.md` wins on design and structure
2. `CLAUDE.md` (this file) wins on conventions and tooling
3. Ask for anything else — don't invent

---

## Implementation Phases

Work through phases in order. Each phase ends with a **runnable, testable build**. Do not start phase N+1 until phase N's Definition of Done is met.

See `ARCHITECTURE.md` for the full phase definitions.

**Current phase:** [Phase name and number, e.g. "Phase 2: Core Engine"]

---

## Project State / Status Checks

Always run `git pull` before reporting on TODOs, project status, or any state-dependent information. Stale answers cause rework.
