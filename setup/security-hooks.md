# Project-Level Security Hooks

Add these two Python scripts to your project's `.claude/hooks/` and wire them in `.claude/settings.json`. They run silently — only make noise when something looks wrong.

## `security-check.py` — PreToolUse Bash gate

Blocks two specific dangerous patterns before any Bash command runs:

1. `curl … | sh` / `wget … | sh` — piping remote scripts to a shell
2. `git push` with secret-like files staged (`.env`, `.pem`, `id_rsa`, `credentials.json`, etc.)

```python
#!/usr/bin/env python3
"""PreToolUse Bash security gate."""
from __future__ import annotations
import json, re, subprocess, sys

PIPE_TO_SHELL = re.compile(r"\b(curl|wget)\b[^|]*\|\s*(sh|bash|zsh)\b")
GIT_PUSH      = re.compile(r"\bgit\s+push\b")
SECRET_FILE   = re.compile(
    r"(^|/)\.env(\.|$)|\.pem$|(^|/)id_rsa(\.|$)"
    r"|(^|/)secrets?\.(json|ya?ml)$|(^|/)credentials?\.(json|ya?ml)$"
)

def strip_noise(s: str) -> str:
    """Remove shell comments and quoted segments to reduce false positives.
    Strips # comment lines then removes double- and single-quoted regions."""
    lines = [ln for ln in s.splitlines() if not ln.lstrip().startswith('#')]
    s = '\n'.join(lines)
    s = re.sub(r'"[^"]*"', '""', s)
    s = re.sub(r"'[^']*'", "''", s)
    return s

def block(reason: str, *details: str) -> None:
    print(f"Blocked: {reason}", file=sys.stderr)
    for line in details: print(f"  {line}", file=sys.stderr)
    sys.exit(2)

def main() -> None:
    try:
        data = json.load(sys.stdin)
    except Exception:
        sys.exit(0)
    cmd = (data.get("tool_input") or {}).get("command", "") or ""
    if PIPE_TO_SHELL.search(strip_noise(cmd)):
        block("piped-to-shell pattern (curl/wget | sh)", cmd)
    if GIT_PUSH.search(cmd):
        try:
            staged = subprocess.run(
                ["git", "diff", "--cached", "--name-only"],
                capture_output=True, text=True, timeout=5, check=False,
            ).stdout
        except Exception:
            staged = ""
        leaks = [l for l in staged.splitlines() if SECRET_FILE.search(l)]
        if leaks:
            block("git push with secret-like staged files", *leaks)
    sys.exit(0)

if __name__ == "__main__":
    try: main()
    except Exception as exc:
        print(f"security-check error (allowing): {exc}", file=sys.stderr)
        sys.exit(0)
```

## `stop-check.py` — Stop hook session-end secret scan

Scans all files modified since the last commit (staged, unstaged, and untracked) for hardcoded credentials when Claude finishes a session. Informational only — never blocks, never raises.

```python
#!/usr/bin/env python3
"""Stop hook — session-end secret scan."""
from __future__ import annotations
import re, subprocess, sys
from pathlib import Path

PATTERNS = {
    "AWS access key":    re.compile(r"\bAKIA[0-9A-Z]{16}\b"),
    "Anthropic API key": re.compile(r"\bsk-ant-[a-zA-Z0-9_\-]{20,}"),
    "OpenAI API key":    re.compile(r"\bsk-[a-zA-Z0-9]{40,}"),
    "GitHub token":      re.compile(r"\bgh[psoru]_[a-zA-Z0-9]{36,}"),
    "Slack token":       re.compile(r"\bxox[bapors]-[a-zA-Z0-9\-]{10,}"),
    "Private key":       re.compile(r"-----BEGIN [A-Z ]*PRIVATE KEY-----"),
}
MAX_FILE_SIZE = 1_000_000

def candidate_files() -> list[str]:
    out: list[str] = []
    for cmd in (["git", "diff", "HEAD", "--name-only"],
                ["git", "ls-files", "--others", "--exclude-standard"]):
        try:
            out.extend(subprocess.run(cmd, capture_output=True, text=True,
                                      timeout=5, check=False).stdout.splitlines())
        except Exception: continue
    return out

def main() -> None:
    try: sys.stdin.read()
    except Exception: pass
    findings: list[str] = []
    for f in candidate_files():
        path = Path(f)
        if not path.is_file(): continue
        try:
            if path.stat().st_size > MAX_FILE_SIZE: continue
            text = path.read_text(errors="ignore")
        except Exception: continue
        for label, pat in PATTERNS.items():
            if pat.search(text):
                findings.append(f"{f}: {label}")
                break
    if findings:
        print("Stop-hook: possible secrets in working tree:", file=sys.stderr)
        for line in findings: print(f"  {line}", file=sys.stderr)
        print("Review before committing.", file=sys.stderr)

if __name__ == "__main__":
    try: main()
    except Exception as exc:
        print(f"stop-check error: {exc}", file=sys.stderr)
    sys.exit(0)
```

## Install

Make both executable:

```bash
chmod +x .claude/hooks/security-check.py .claude/hooks/stop-check.py
```

Wire them into `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/security-check.py"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/stop-check.py"
          }
        ]
      }
    ]
  }
}
```
