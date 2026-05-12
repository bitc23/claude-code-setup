# Multi-Agent Orchestration

The pattern: a slash command dispatches a planner agent, which dispatches one or more worker agents, which call skills. The original user request flows down the chain; results flow back up as summaries.

## When It's Worth It

- A codebase-wide task with naturally parallel sub-tasks (e.g. "audit auth, billing, and notifications for the same vulnerability class")
- A workflow you run >5×/week that always follows the same shape
- A scenario where the intermediate agent's context would otherwise pollute yours irretrievably

## When It's Not

For personal projects, **almost always not.** The overhead of authoring + maintaining a Command → Agent → Skill graph is high, and a single sub-agent (`rules/sub-agents.md`) with a sharp prompt covers 90% of the value. Reach for orchestration only when you've already tried a single agent and it isn't enough.

## Cost Reality

Each level in the chain spawns its own context window. A three-level orchestration that does meaningful work easily 3–5× the token cost of doing the same thing inline. The status line shows your weekly limit — keep one eye on it.

## If You Build One

Standard layout:

```
.claude/
├── commands/
│   └── audit.md          # entry point: slash command
├── agents/
│   ├── audit-planner.md  # decomposes the task
│   └── audit-worker.md   # one per subdomain
└── skills/
    └── threat-classify/  # called by the worker
        └── SKILL.md
```

The command's body briefs the planner. The planner spawns workers in parallel (one tool call per worker, batched in a single message). Each worker calls skills as needed and returns a structured result. The planner aggregates and returns to the command.

## Reference

The cloned `claude-code-best-practice` repo has a worked example in `agent-teams/` and `orchestration-workflow/`. Read it before building one — most of the value is in seeing how data contracts between layers are written.
