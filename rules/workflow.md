# Development Workflow

For every non-trivial task:

1. **Plan Mode first** — press **Shift+Tab** at the start of any non-trivial task. Claude outlines its full plan and waits for your approval before executing anything. This is the easiest way to catch misunderstandings before 20 files are edited the wrong way.
2. **Research** — search GitHub and package registries before writing new code; prefer proven libraries over hand-rolled solutions
3. **Plan in detail** — use `planner` agent for complex features; break into phases with clear milestones (see `rules/sub-agents.md` for how custom agents are authored — most of the agent names referenced here are aspirational; install or write them as needed)
4. **TDD** — write tests before implementation (see `rules/testing.md`)
5. **Review** — use `code-reviewer` agent immediately after writing code
6. **Commit** — conventional message, one logical change per commit (see `rules/git.md`)

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

## Performance & Model Selection

| Model | Use for |
|-------|---------|
| Haiku 4.5 | Lightweight/frequent agents, worker agents in multi-agent systems, search/summarisation, headless `-p` calls |
| Sonnet 4.6 | Main development work, complex coding, orchestration |
| Opus 4.7 | Architectural decisions, deep research, maximum reasoning |

Avoid the last 20% of context window for large refactors or multi-file features — use `/compact` when context climbs above ~50%.

---

## Cross-Model Workflow

Switching models mid-session is a real cost lever. Use `/model` to swap between Haiku, Sonnet, and Opus without losing context.

Typical pattern for a multi-step task:

1. **Exploration / search / "what's in this repo"** → Haiku
2. **Implementation, refactor, debugging** → Sonnet (default)
3. **Architectural decision, gnarly bug, cross-cutting design** → Opus, then switch back to Sonnet to execute

Do not run Opus continuously — it eats both rate limits and money. Treat it as the model you call in for hard reasoning, not the default driver.

---

## Agents & Skills

**Use proactively — no need to wait to be asked:**

- Complex feature → `planner` agent
- Code written or modified → `code-reviewer` agent
- Bug fix or new feature → `tdd-guide` agent
- Architectural decision → `architect` agent
- Build fails → `build-error-resolver` agent
- Security-sensitive change → `security-reviewer` agent

> Note: these agent names are conventions, not built-ins. None ship with Claude Code by default. Either install a marketplace plugin that provides them, or author them yourself in `~/.claude/agents/<name>.md` (see `rules/sub-agents.md`).

**Run independent agents in parallel.** Batch unrelated tasks in the same message instead of sequential calls.

**Invoke relevant skills before any response.** Even a 1% chance a skill applies means invoke it. The `superpowers:using-superpowers` skill is embedded inline by the superpowers plugin at startup — no SessionStart hook needed.
