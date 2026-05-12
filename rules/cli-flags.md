# Claude Code CLI Flags

The flags I actually use, with the situations they fit. Run `claude --help` for the full list — Anthropic adds and renames flags often, so verify before relying on anything obscure.

## Session Management

| Flag | What it does | Use when |
|------|--------------|----------|
| `--continue` / `-c` | Resumes the most recent session in this directory | You closed a session by accident or want to pick up where you left off |
| `--resume` | Shows a picker of recent sessions to resume | Multiple sessions in flight; you want to choose |
| `--session-id <id>` | Resume a specific session by id | Scripted resumption |

## Model Selection

| Flag | What it does | Use when |
|------|--------------|----------|
| `--model <name>` | Pin a model for this session (`claude-haiku-4-5`, `claude-sonnet-4-6`, `claude-opus-4-7`) | One-off cost/latency choice; pairs with cross-model workflow |

## Headless / Scripted

| Flag | What it does | Use when |
|------|--------------|----------|
| `-p "<prompt>"` / `--print` | One-shot non-interactive run; prints result to stdout | Cron jobs, automation, piping into other tools |
| `--tools "<list>"` | Restrict tool surface (empty string = no tools at all) | Headless summarisers; cheaper + safer when no edits should happen |
| `--add-dir <path>` | Add another working directory to the session | Cross-repo work |

## Permissions

| Flag | What it does | Use when |
|------|--------------|----------|
| `--dangerously-skip-permissions` | Disables all confirmation prompts for the session | You've reviewed the plan and trust the run; mistakes may be hard to undo. Hooks still run. |

Prefer the persistent `allowedTools` setting in `~/.claude/settings.json` over this flag for everyday use — see `rules/tips.md` for the snippet.

## MCP

| Flag | What it does | Use when |
|------|--------------|----------|
| `--mcp-config <path>` | Load MCP server config from a specific file | Project-specific MCP setup; testing a new server in isolation |

See `rules/mcp.md` for what MCP is and how to set servers up.

## Common Patterns

```bash
# Resume the last session here
claude -c

# Cheap summary, no tools, no interaction
claude -p "summarise the last 10 commits" --model claude-haiku-4-5 --tools ""

# Scripted code review piped to a file
claude -p "review the diff in $(pwd)" --model claude-sonnet-4-6 > /tmp/review.md

# Open a session with another repo also visible
claude --add-dir ~/Developer/other-repo
```
