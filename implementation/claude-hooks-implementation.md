# Hooks Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Best Practice](https://img.shields.io/badge/Best_Practice-blue?style=flat)](../best-practice/claude-settings.md#hooks)

Practical hook implementations — sound notifications, security auditing, logging, auto-formatting, and agent-scoped hooks.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#sound-notification-hooks"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

Hooks are shell commands that run at specific points in the Claude Code lifecycle. They receive context via stdin as JSON and can modify Claude's behavior by returning JSON on stdout. Configure them in `.claude/settings.json` under the `"hooks"` key.

---

## Sound Notification Hooks

Play audio notifications when Claude performs actions — the most common hook use case. This repo implements sound notifications for all 27 hook events using a Python handler script.

### Configuration in settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "PreToolUse"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "Stop"
          }
        ]
      }
    ]
  }
}
```

### Handler script (simplified)

The hook handler reads the event from `CLAUDE_HOOK_EVENT`, selects the appropriate sound file, and plays it asynchronously:

```python
#!/usr/bin/env python3
"""Claude Code Hook Handler — plays sounds for hook events."""

import sys
import json
import subprocess
import platform
from pathlib import Path

HOOK_SOUND_MAP = {
    "PreToolUse": "pretooluse",
    "PostToolUse": "posttooluse",
    "Stop": "stop",
    "Notification": "notification",
    "SessionStart": "sessionstart",
    # ... all 27 events mapped
}

def play_sound(sound_file: Path):
    """Play audio file cross-platform."""
    system = platform.system()
    if system == "Darwin":
        subprocess.Popen(["afplay", str(sound_file)],
                         stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    elif system == "Linux":
        subprocess.Popen(["aplay", str(sound_file)],
                         stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

def main():
    import os
    event = os.environ.get("CLAUDE_HOOK_EVENT", "")
    folder = HOOK_SOUND_MAP.get(event)
    if not folder:
        return

    sounds_dir = Path(__file__).parent.parent / "sounds" / folder
    if sounds_dir.exists():
        mp3_files = list(sounds_dir.glob("*.mp3"))
        if mp3_files:
            play_sound(mp3_files[0])

if __name__ == "__main__":
    main()
```

Key fields:
- `async: true` — the hook runs without blocking Claude's response
- `timeout: 5000` — kills the process after 5 seconds if it hangs
- `${CLAUDE_PROJECT_DIR}` — expands to the project root at runtime

---

## Security Audit Hook

A `PreToolUse` hook that blocks dangerous commands before they execute. The hook reads the tool input from stdin and returns a JSON response to block or allow the action.

### Block dangerous rm -rf commands

```bash
#!/bin/bash
# .claude/hooks/scripts/security-audit.sh
# PreToolUse hook — blocks destructive file operations

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name // empty')
tool_input=$(echo "$input" | jq -r '.tool_input // empty')

if [ "$tool_name" = "Bash" ]; then
  command=$(echo "$tool_input" | jq -r '.command // empty')

  # Block rm -rf on root or home
  if echo "$command" | grep -qE 'rm\s+-rf\s+(/|~|/home|/Users)'; then
    echo '{"decision": "block", "reason": "Blocked: rm -rf on protected directory"}'
    exit 0
  fi

  # Block force push to main
  if echo "$command" | grep -qE 'git\s+push\s+.*--force.*\s+(main|master)'; then
    echo '{"decision": "block", "reason": "Blocked: force push to main/master"}'
    exit 0
  fi
fi

# Allow everything else
echo '{"decision": "allow"}'
```

### Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/security-audit.sh",
            "timeout": 3000
          }
        ]
      }
    ]
  }
}
```

The `matcher` field restricts this hook to only fire for `Bash` tool invocations. The hook must be synchronous (no `async: true`) so it can block the tool call before execution.

---

## Logging Hook

A `PostToolUse` hook that writes every tool invocation to a JSONL log file for auditing and debugging.

```bash
#!/bin/bash
# .claude/hooks/scripts/log-tool-use.sh
# PostToolUse hook — logs tool invocations to JSONL

input=$(cat)
log_file="${CLAUDE_PROJECT_DIR}/.claude/hooks/logs/tool-use.jsonl"

mkdir -p "$(dirname "$log_file")"

timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
tool_name=$(echo "$input" | jq -r '.tool_name // "unknown"')
session_id=$(echo "$input" | jq -r '.session_id // "unknown"')

jq -n \
  --arg ts "$timestamp" \
  --arg tool "$tool_name" \
  --arg session "$session_id" \
  --argjson input "$input" \
  '{timestamp: $ts, tool: $tool, session: $session, raw: $input}' \
  >> "$log_file"
```

### Configuration

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/log-tool-use.sh",
            "timeout": 3000,
            "async": true
          }
        ]
      }
    ]
  }
}
```

Add `.claude/hooks/logs/` to `.gitignore` to keep logs local.

---

## Auto-format Hook

A `PostToolUse` hook that runs Prettier after every file write, keeping code formatted without manual intervention.

```bash
#!/bin/bash
# .claude/hooks/scripts/auto-format.sh
# PostToolUse hook — runs Prettier after Write/Edit tool calls

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name // empty')

# Only run after Write or Edit
if [ "$tool_name" != "Write" ] && [ "$tool_name" != "Edit" ]; then
  exit 0
fi

file_path=$(echo "$input" | jq -r '.tool_input.file_path // .tool_input.filePath // empty')

if [ -z "$file_path" ]; then
  exit 0
fi

# Only format supported file types
case "$file_path" in
  *.js|*.ts|*.jsx|*.tsx|*.json|*.css|*.scss|*.md|*.html|*.yaml|*.yml)
    npx prettier --write "$file_path" 2>/dev/null
    ;;
esac
```

### Configuration

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/auto-format.sh",
            "timeout": 10000,
            "async": true
          }
        ]
      }
    ]
  }
}
```

---

## Agent-scoped Hooks

Hooks can be defined inside a subagent's frontmatter, scoping them to only fire when that specific agent is running. Agent frontmatter supports these hook events: `PreToolUse`, `PostToolUse`, `PermissionRequest`, `PostToolUseFailure`, `Stop`, `SubagentStop`.

### Example: weather-agent with a Stop hook

```yaml
---
name: weather-agent
description: Fetches weather data for Dubai, UAE
tools: WebFetch, Read, Write
model: sonnet
hooks:
  Stop:
    - hooks:
        - type: command
          command: "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py --agent=weather-agent"
          timeout: 5000
          async: true
  PreToolUse:
    - matcher: Bash
      hooks:
        - type: command
          command: "bash ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/security-audit.sh"
          timeout: 3000
---
```

Agent-scoped hooks follow the same structure as session-level hooks. The `--agent=weather-agent` flag lets the handler script select agent-specific sounds or behaviors.

---

## Complete settings.json Hooks Configuration

This shows all hook events wired up in a single `settings.json` — the pattern used in this repository. Each event delegates to the same Python handler script, which reads `CLAUDE_HOOK_EVENT` to determine which sound to play.

```json
{
  "disableAllHooks": false,
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "PreToolUse"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "PostToolUse"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "UserPromptSubmit"
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "Notification"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "Stop"
          }
        ]
      }
    ],
    "SubagentStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "SubagentStart"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "statusMessage": "SubagentStop"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "once": true,
            "statusMessage": "SessionStart"
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PROJECT_DIR}/.claude/hooks/scripts/hooks.py",
            "timeout": 5000,
            "async": true,
            "once": true,
            "statusMessage": "SessionEnd"
          }
        ]
      }
    ]
  }
}
```

### Hook field reference

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Always `"command"` |
| `command` | string | Shell command to execute. Receives JSON context on stdin |
| `timeout` | integer | Max execution time in milliseconds before the process is killed |
| `async` | boolean | `true` = non-blocking (fire-and-forget). `false` = blocking (can modify behavior via stdout) |
| `once` | boolean | `true` = run only once per session (useful for `SessionStart`, `PreCompact`) |
| `statusMessage` | string | Text shown in the Claude Code status bar while the hook runs |
| `matcher` | string | On the parent object — restricts when the hook fires (e.g., `"Bash"` for PreToolUse, `"Write|Edit"` for PostToolUse) |

---

## Sources

- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Settings Best Practice — Hooks Section](../best-practice/claude-settings.md#hooks)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
