# Nightly Memory Digest

Once a project has been running for a few weeks, sessions can start cold without context. The nightly digest solves this by asking Claude (Haiku, no tools — cheap and fast) to summarise the last 7 days of activity into four sections:

- `## Recent decisions` — architectural choices made this week
- `## Active work` — what's in progress
- `## Known gotchas` — traps, workarounds, non-obvious constraints
- `## Next priorities` — what to tackle next

The digest is a **reviewable artefact**, not an autonomous rewriter. You read it, cherry-pick the bullets worth keeping, and manually promote them to `CLAUDE.md`. Claude never edits `CLAUDE.md` unattended — that file is hand-curated.

## How It Works

1. `git pull` to get latest
2. Gather: last 100 git log lines (7 days), last 120 lines of `WORKLOG.md`, all `TODO.md` + `todo/*.md` content, first 60 lines of `CLAUDE.md`
3. Feed to `claude -p --model claude-haiku-4-5 --tools ""` with a structured prompt
4. Validate output: must contain all four section headers, must be ≤220 lines
5. Write to `docs/MEMORY-DIGEST.md`, write a freshness timestamp to `.claude/.memory-curate-last-run`
6. Send a macOS notification only on failure

A `SessionStart` hook reads the freshness marker and warns you at the start of a session if the digest is >48h stale (so you know the context may be outdated).

## To Implement This for Your Project

The full scripts (`memory-curate.sh`, `memory-curate-prompt.md`, `com.YOURPROJECT.memory-curate.plist`, and `memory-staleness.py`) follow the same launchd + bash pattern as `setup/agentshield-weekly.md`. Ask Claude to scaffold them once you have a running project with a few weeks of WORKLOG history — that's when the digest becomes genuinely useful.
