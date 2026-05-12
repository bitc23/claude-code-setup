# AgentShield Weekly Scan

[AgentShield](https://github.com/affaan-m/everything-claude-code) (`ecc-agentshield`) is a security scanner built for AI-generated code. It grades your repo A–F and flags issues specific to LLM outputs (prompt injection vectors, over-permissioned tool calls, insecure default patterns, etc.).

**How it works:** runs every Monday at 09:00 via macOS launchd, writes a timestamped log, and sends a macOS notification **only when there's a regression** (grade drops below A, or any CRITICAL/HIGH finding appears). Healthy scans are completely silent.

## Step 1 — Create the scan script

`.claude/scripts/agentshield-weekly.sh`:

```bash
#!/bin/bash
# Weekly AgentShield regression scan.
# Triggered by launchd. Logs every run; notifies only on regression.

set -u

PROJECT_DIR="$(cd "$(dirname "$0")/../.." && pwd)"   # repo root
APP_NAME="$(basename "$PROJECT_DIR")"                # used for log folder + notification
LOG_DIR="${HOME}/Library/Logs/${APP_NAME}"
mkdir -p "$LOG_DIR"

TIMESTAMP="$(date +%Y%m%d-%H%M%S)"
LOG_FILE="$LOG_DIR/agentshield-$TIMESTAMP.log"

# launchd starts with a near-empty PATH — add common node/npm locations.
NVM_NODE_BIN=""
if [ -d "$HOME/.nvm/versions/node" ]; then
    LATEST="$(ls -1 "$HOME/.nvm/versions/node" 2>/dev/null | tail -1)"
    [ -n "$LATEST" ] && NVM_NODE_BIN="$HOME/.nvm/versions/node/$LATEST/bin"
fi
export PATH="/opt/homebrew/bin:/usr/local/bin:${NVM_NODE_BIN:+$NVM_NODE_BIN:}/usr/bin:/bin"

cd "$PROJECT_DIR" || { echo "Cannot cd to $PROJECT_DIR" > "$LOG_FILE"; exit 1; }

REPORT="$(npx --yes ecc-agentshield scan 2>&1)"
EXIT_CODE=$?
{
    echo "=== AgentShield weekly scan @ $(date) ==="
    echo "Project: $PROJECT_DIR"
    echo "Exit: $EXIT_CODE"
    echo
    echo "$REPORT"
} > "$LOG_FILE"

GRADE_LINE="$(echo "$REPORT"   | grep -E '^\s*Grade:'              | head -1 | sed 's/^[[:space:]]*//')"
SUMMARY_LINE="$(echo "$REPORT" | grep -E 'critical, .* high, .* medium' | head -1 | sed 's/^[[:space:]]*//')"

REGRESSION=0
echo "$GRADE_LINE"   | grep -qE 'Grade: A ' || REGRESSION=1
echo "$SUMMARY_LINE" | grep -qE '[1-9][0-9]* critical|[1-9][0-9]* high' && REGRESSION=1
[ $EXIT_CODE -ne 0 ] && REGRESSION=1

if [ $REGRESSION -eq 1 ]; then
    BODY="${GRADE_LINE:-scan failed}. $SUMMARY_LINE. Log: $LOG_FILE"
    osascript -e "display notification \"$BODY\" with title \"AgentShield REGRESSION\" subtitle \"${APP_NAME} weekly scan\" sound name \"Sosumi\"" || true
fi

echo "" >> "$LOG_FILE"
echo "[$(date)] Scan complete. regression=$REGRESSION" >> "$LOG_FILE"
exit 0
```

## Step 2 — Create the launchd schedule

`.claude/scripts/com.YOURPROJECT.agentshield-weekly.plist` — replace `YOURPROJECT` and the script path with your actual values:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
    "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.YOURPROJECT.agentshield-weekly</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/path/to/your/project/.claude/scripts/agentshield-weekly.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key><integer>1</integer>
        <key>Hour</key><integer>9</integer>
        <key>Minute</key><integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/YOUR_USERNAME/Library/Logs/YOURPROJECT/agentshield-launchd.out</string>
    <key>StandardErrorPath</key>
    <string>/Users/YOUR_USERNAME/Library/Logs/YOURPROJECT/agentshield-launchd.err</string>
    <key>RunAtLoad</key><false/>
    <key>ProcessType</key><string>Background</string>
</dict>
</plist>
```

## Step 3 — Install, verify, and test

```bash
# Install
cp .claude/scripts/com.YOURPROJECT.agentshield-weekly.plist \
   ~/Library/LaunchAgents/
launchctl bootstrap "gui/$(id -u)" \
   ~/Library/LaunchAgents/com.YOURPROJECT.agentshield-weekly.plist

# Verify it's registered
launchctl print "gui/$(id -u)/com.YOURPROJECT.agentshield-weekly" | head -20

# Run immediately as a sanity check
launchctl kickstart -k "gui/$(id -u)/com.YOURPROJECT.agentshield-weekly"
ls -lt ~/Library/Logs/YOURPROJECT | head

# Uninstall later
launchctl bootout "gui/$(id -u)" \
   ~/Library/LaunchAgents/com.YOURPROJECT.agentshield-weekly.plist
rm ~/Library/LaunchAgents/com.YOURPROJECT.agentshield-weekly.plist
```

> Notes: if the Mac is asleep at 09:00 Monday, launchd runs the job on next wake. The first run triggers a one-time macOS prompt to allow osascript notifications — approve it.
