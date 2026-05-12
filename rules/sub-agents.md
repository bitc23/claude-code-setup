# Custom Sub-Agents

A sub-agent is a Markdown file in `.claude/agents/` that defines a specialised assistant with its own tool surface and isolated context window. Spawning one keeps heavy work (security review, codebase exploration, refactor planning) out of your main session.

## Where to Put Them

- `~/.claude/agents/<name>.md` — global, available in every project
- `<project>/.claude/agents/<name>.md` — project-only

## Reality Check

The agent names referenced throughout these rules — `planner`, `code-reviewer`, `tdd-guide`, `architect`, `build-error-resolver`, `security-reviewer` — **do not ship with Claude Code by default**. Either install a marketplace plugin that provides them, or author them yourself. The references are aspirational shorthand for "use the right specialised agent for this task."

Built-in agent types that *do* always exist: `general-purpose`, `Explore`, `Plan`, `statusline-setup`, `claude-code-guide`. List them with `/agents`.

## Minimal Example

`~/.claude/agents/code-reviewer.md`:

```markdown
---
name: code-reviewer
description: Use after every code change, before committing. Reviews for correctness, clarity, security, and adherence to the rules in ~/.claude/rules/.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a code reviewer. Read the diff (`git diff` against the base branch), then review the changed files in full context.

For each finding, label severity: CRITICAL / HIGH / MEDIUM / LOW.

- CRITICAL: security vulnerability, data loss, broken contract
- HIGH: bug or significant quality issue
- MEDIUM: maintainability concern
- LOW: style or minor

Return a punch list. Be terse. No emojis.
```

Invoke from the main agent: "use the code-reviewer agent on the current diff." The harness routes to your file.

## Frontmatter Fields

| Field | Purpose |
|-------|---------|
| `name` | Identifier — must match the filename |
| `description` | When to use this agent. The main agent reads this to decide whether to dispatch. Be specific. |
| `tools` | Comma-separated tool list. Omit to inherit all tools. Restrict for safety (e.g. read-only reviewer = no Write/Edit). |
| `model` | Pin to `haiku`, `sonnet`, or `opus`. Default inherits from the parent. |

## When to Author One

- You repeatedly hand the main session the same kind of complex task (review, planning, exploration)
- The task generates a lot of intermediate context you don't want to keep
- You want strict tool restrictions (e.g. an agent that can read but never edit)

## When *Not* To

- For one-off prompts → use a slash command (`rules/slash-commands.md`)
- For instructions that should always apply → put them in a rule file
- For complex multi-step processes → use a Skill (the superpowers plugin is a good example)

## Parallel Execution

Spawn multiple independent sub-agents in a single message — the harness runs them concurrently. Useful for "explore A and B and C" or "lint, type-check, and test" patterns.

## Related

- `rules/slash-commands.md` — for one-off prompt templates
- `rules/orchestration.md` — for chaining agents together (usually overkill)
