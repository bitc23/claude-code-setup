# First-Time Setup

Run this once after copying `CLAUDE.md` and the `rules/` directory to `~/.claude/`. The user does Step A by hand; Claude executes Step B on request.

---

## Step A — You do once (manually)

**1. Install the superpowers plugin**

Open Claude Code and run:

```
/plugins install superpowers
```

**2. Install the everything-claude-code plugin (optional, disabled by default)**

This plugin adds ~200 skills but also injects ~4,000 tokens into every session's context window. Only enable it if you actively use more than 20% of its skills. If you only need a handful, copy those skill files to `~/.claude/skills/` instead.

To install: register the marketplace in `~/.claude/settings.json`:

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

Leave it disabled in `enabledPlugins` (see Step B) until you've confirmed you need it.

**3. Install the global hook scripts** (only needed if you use Xcode):

```bash
mkdir -p ~/.claude/hooks
# copy block-xcodebuild-test.sh from setup/swift.md into ~/.claude/hooks/
chmod +x ~/.claude/hooks/block-xcodebuild-test.sh
```

**4. Restart Claude Code** — plugins are loaded at startup.

---

## Step B — Claude executes (run once, then never again)

Read `~/.claude/settings.json`, then **merge** the block below into it (preserve existing keys, don't overwrite). Confirm when done.

```json
{
  "env": {
    "ECC_GATEGUARD": "off"
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": false,
    "superpowers@claude-plugins-official": true
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 $HOME/.claude/hooks/security-check.py",
            "timeout": 5
          },
          {
            "type": "command",
            "command": "$HOME/.claude/hooks/block-xcodebuild-test.sh",
            "timeout": 5
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

Drop the `block-xcodebuild-test.sh` hook entry from `PreToolUse` if you don't use Xcode.

---

## Token & Context Budget

**Baseline pre-fill: ~5–7% of context window** before your first message. This is the cost of the superpowers plugin, rule files, and built-in tool schemas. It was previously 10–20% when `everything-claude-code` was enabled and a redundant `SessionStart` hook was active — both removed.

**Plugin discipline is the biggest lever.** Every enabled plugin injects all its skill names into every session. `everything-claude-code` alone adds ~200 skills (~4,000 tokens). Only enable a plugin if you actively use >20% of its skills; otherwise copy the specific skills you need to `~/.claude/skills/`.

**Keep language-specific rules project-scoped.** Global rules in `~/.claude/rules/` load for every project regardless of language. Move Swift, Kotlin, Python, etc. rules into the specific project's `.claude/rules/<language>/` directory so they only load when relevant.

**What burns context fastest:**

- Reading large source files in full when you only need one function
- Long sessions that touch many files across many topics
- Agents returning large results inline instead of summarising
- Loading the entire codebase instead of navigating by folder READMEs first
- Enabled plugins with large skill lists you don't actively use

**How to stay in budget:**

| Tip | Why it helps |
|-----|-------------|
| Run `/compact` when context hits 50% | Compresses history; recovers ~40% of your window |
| Keep sessions focused on one task | One feature or bug per session; start fresh for unrelated work |
| Read READMEs before source files | Claude reads the 15-line summary, not 500-line implementations |
| Use sub-agents for heavy analysis | Sub-agents run in their own isolated context window — summaries come back, raw data doesn't |
| Switch to Haiku for lightweight tasks | 90% of Sonnet capability, 3× cheaper — good for search, summarisation, quick edits |
| Write a WORKLOG handoff note and continue in a new session | Cheaper than /compact when you're already at 70%+ |

The status line at the bottom of your terminal shows context %, 5-hour session limit, and weekly limit — color-coded green / yellow (watch it) / red (act now). When it shows `(try /compact)` next to context %, do it.
