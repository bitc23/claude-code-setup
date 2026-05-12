# Claude Code Tips & Tricks

Quick reference for commands and behaviors that aren't obvious to new users.

## Slash Commands

| Command | What it does |
|---------|-------------|
| `/compact` | Compresses conversation history to free context. Use at ~50% context usage. |
| `/clear` | Wipes the conversation and starts completely fresh. Use when switching to an unrelated task. |
| `/insights` | Shows Claude's analysis of your session behavior — patterns being corrected repeatedly, what's working. Run every few sessions to catch drift. |
| `/help` | Lists all available commands and keyboard shortcuts. |
| `/memory` | Opens your persistent memory for this project — review what Claude knows about your work. |
| `/model` | Switch between Claude models mid-session (Haiku/Sonnet/Opus). Pairs with the cross-model workflow in `rules/workflow.md`. |
| `/agents` | List installed sub-agents and invoke one directly. |
| `/mcp` | List connected MCP servers and the tools they expose. Essential when an MCP is misbehaving. |
| `/cost` | Show session token usage and dollar cost. Pairs well with the status line. |
| `/login` `/logout` | Switch between API keys (e.g. personal vs work). |
| `/plugins` | Manage plugins — install, list, enable/disable. |
| `/hooks` | Review which hooks are active in this session. |
| `/settings` | Open settings for review or editing. |
| `/bug` | Report a Claude Code bug directly to Anthropic with session context attached. |

## Keyboard Shortcuts

| Shortcut | What it does |
|----------|-------------|
| **Shift+Tab** | Enters Plan Mode — Claude outlines its full plan and waits for approval before executing anything. Use for every non-trivial task. |
| **Option+T** (macOS) / **Alt+T** | Toggles extended thinking — Claude reasons longer before answering. On by default with this setup. |
| **Ctrl+O** | Shows Claude's thinking output (verbose mode). Useful when Claude's answer seems wrong and you want to see its reasoning. |
| **Ctrl+C** | Interrupts the current response or tool call. Use to redirect Claude mid-flight when it goes off track. |
| **↑ arrow** | In the input field, cycles through previous messages you've sent. |

## Input Prefixes

- **`!`** — run a shell command in the current session and have output land in the conversation:
  ```
  ! git log --oneline -10
  ! npm run test
  ```
  Useful after interactive logins (`gcloud auth login`, `aws sso login`) so Claude sees the result.

- **`@`** — attach a file path to the next message without a Read tool round-trip:
  ```
  what does @src/auth/AuthService.ts do?
  ```
  Cuts context for short questions about a single file.

- **`#`** — drop a quick fact into memory mid-conversation:
  ```
  # remember: this project uses PKCE for the auth flow
  ```

## CLAUDE.md `@` Imports

Inside any CLAUDE.md, write `@./other.md` to pull another file's content into the parent's context. Use this to split a large CLAUDE.md into composable pieces.

## Plan Mode (Shift+Tab)

Use Plan Mode before any non-trivial task. Claude will outline its intended approach — files to read, changes to make, order of operations — and wait for your approval. This is the easiest way to catch misunderstandings before 20 files are edited in the wrong direction.

Press **Shift+Tab** at the start of a message, or type your request and then press Shift+Tab to switch into plan mode.

## Skipping Permission Prompts

By default, Claude asks for confirmation before running shell commands, editing files, and so on. Two ways to change this:

**For one session — start with the `--dangerously-skip-permissions` flag:**

```bash
claude --dangerously-skip-permissions
```

This disables all confirmation prompts for the session. Only use when you've reviewed the plan and trust what Claude is about to do — mistakes may be hard to undo.

**For specific tools permanently — use `allowedTools` in `~/.claude/settings.json`:**

```json
{
  "allowedTools": ["Bash", "Read", "Edit", "Write", "Glob", "Grep"]
}
```

This auto-approves the listed tools without prompting but still lets hooks run. Safer than `--dangerously-skip-permissions` because your security hooks remain active.

## Sub-Agents and Context Isolation

The `Agent` tool spawns a sub-agent in its own isolated context window. Heavy tasks — security review, codebase exploration, large refactor analysis — run in the sub-agent's window, not yours. The sub-agent returns a summary, keeping your main window clean.

If your context is high and you need something expensive: ask Claude to use a sub-agent rather than doing it inline.

See `rules/sub-agents.md` for how to author one.

## /compact vs /clear

- **`/compact`** — keeps the conversation; compresses old messages. Claude retains memory of the session and can continue where it left off.
- **`/clear`** — wipes everything. Claude starts with no memory of the session. Use when switching to a completely different task, or when the context is deeply polluted by a failed direction.

## Recovering from a Stuck or Looping Session

If Claude is going in circles or committed to a wrong approach:

1. **Ctrl+C** to interrupt
2. Give a clear correction: "Stop. That approach is wrong because X. Instead, let's try Y."
3. If badly stuck: `/clear` and restate the problem from scratch — a clean context often unblocks immediately.

## Headless / Scripted Usage

Claude Code can run non-interactively for automation:

```bash
# One-shot task, no interaction
claude -p "summarise what changed in the last 7 git commits" --model claude-haiku-4-5

# Read stdin
echo "what does this function do?" | claude -p --model claude-haiku-4-5

# Pipe output to a file
claude -p "write a CHANGELOG entry for this week's commits" > /tmp/changelog-draft.md
```

This is how the nightly memory digest (see `setup/memory-digest.md`) works — Haiku, no tools, cheap and fast.

See `rules/cli-flags.md` for the full flag reference.
