# Hooks — Event Reference

Hooks are shell commands the harness runs in response to events. Wire them in `~/.claude/settings.json` (global) or `<project>/.claude/settings.json` (project).

This is a curated list of the events I actually use. Claude Code ships more — check `/hooks` in-session or the official docs at https://docs.claude.com/en/docs/claude-code/hooks for the full set, since Anthropic adds and renames them periodically.

## Most-Used Events

| Event | When it fires | Typical use |
|-------|---------------|-------------|
| `PreToolUse` | Right before any tool runs (Bash, Edit, Write, etc.) — can return non-zero to block | Security gate (block `curl \| sh`, deny `git push` with secrets staged), guardrails on dangerous commands |
| `PostToolUse` | After a tool runs successfully | Auto-format on Edit, run a linter on Write, log destructive ops |
| `UserPromptSubmit` | When the user submits a prompt | Inject session context (e.g. WORKLOG entries), block on banned phrases |
| `SessionStart` | At session creation | Print a welcome banner, warn on stale memory digests, refresh shared state |
| `SessionEnd` | When a session terminates | Summary log, freshness markers |
| `Stop` | When Claude finishes its turn | Session-end secret scan, append to a metrics log |
| `SubagentStop` | When a sub-agent finishes | Log sub-agent activity, audit token spend |
| `PreCompact` | Before `/compact` runs | Snapshot the conversation before it's compressed |
| `Notification` | When the harness raises a notification | Pipe to macOS `osascript` for desktop alerts |

## Matchers

`PreToolUse` and `PostToolUse` accept a `matcher` field that filters by tool name:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [{ "type": "command", "command": "$HOME/.claude/hooks/security-check.py", "timeout": 5 }]
    },
    {
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command", "command": "prettier --write $CLAUDE_FILE_PATH" }]
    }
  ]
}
```

Other events do not use matchers — the array contains plain hook entries.

## Block Behaviour

A `PreToolUse` hook that exits **non-zero** cancels the tool call. Stderr from the hook is shown to Claude as the reason. Other hooks (Post*, Stop, Notification) cannot cancel anything — only log or react.

## Input

Each hook receives JSON on stdin describing the event. For `PreToolUse`:

```json
{
  "tool_name": "Bash",
  "tool_input": { "command": "git push" },
  "session_id": "...",
  "cwd": "/Users/.../my-project"
}
```

Parse with `jq`:

```bash
cmd="$(jq -r '.tool_input.command // empty')"
```

## Where to Wire Them

- **Global (every session):** `~/.claude/settings.json` — for protections that apply everywhere (e.g. `block-xcodebuild-test.sh`, your security gate)
- **Project (this repo only):** `<project>/.claude/settings.json` — for project-specific behaviour (e.g. auto-quit a running app before `xcodebuild`)
- **Local (don't commit):** `<project>/.claude/settings.local.json` — for personal-machine overrides

## Related

- `setup/security-hooks.md` — the two security hooks I run on every project
- `setup/swift.md` — the `block-xcodebuild-test.sh` global hook
