# Hooks Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code hooks — event-driven handlers that run outside the agentic loop on specific lifecycle events.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Overview

Hooks are user-defined handlers that execute **outside** the agentic loop — they are not part of Claude's reasoning or tool-calling chain. They fire on specific lifecycle events and can run scripts, inject prompts, spawn agents, or call HTTP endpoints. Use hooks to add observability, enforce policies, automate side-effects, and gate decisions without polluting the conversation context.

---

## Hook Events (27)

### Session Lifecycle

| Event | Fires When |
|-------|-----------|
| `SessionStart` | A new interactive session begins |
| `SessionEnd` | The session is about to close |
| `Setup` | First time Claude Code runs in a project (or after `/init`) |

### Tool Lifecycle

| Event | Fires When |
|-------|-----------|
| `PreToolUse` | Before a tool call is executed. Can block, allow, or modify the call |
| `PostToolUse` | After a tool call completes successfully |
| `PostToolUseFailure` | After a tool call fails with an error |
| `PermissionRequest` | When Claude requests permission for a tool the user hasn't pre-approved |
| `PermissionDenied` | When a tool permission request is denied by the user or a policy |

### Agent Lifecycle

| Event | Fires When |
|-------|-----------|
| `SubagentStart` | A subagent (via `Agent(...)` tool) begins execution |
| `SubagentStop` | A subagent finishes execution |
| `Stop` | The **main** agent finishes its response turn |
| `StopFailure` | The main agent's response turn fails with an error |

### Task / Teams

| Event | Fires When |
|-------|-----------|
| `TeammateIdle` | A teammate in an Agent Teams session becomes idle and available |
| `TaskCreated` | A background task is created |
| `TaskCompleted` | A background task finishes |

### Context

| Event | Fires When |
|-------|-----------|
| `PreCompact` | Before context compaction runs |
| `PostCompact` | After context compaction completes |
| `UserPromptSubmit` | The user submits a prompt (before Claude processes it) |
| `Notification` | Claude sends a notification (e.g., task done, permission needed) |

### Config / Environment

| Event | Fires When |
|-------|-----------|
| `ConfigChange` | A settings file is modified |
| `CwdChanged` | The working directory changes (e.g., via `/add-dir`) |
| `FileChanged` | A watched file is modified on disk |
| `InstructionsLoaded` | CLAUDE.md or rules files are loaded into context |

### Worktree

| Event | Fires When |
|-------|-----------|
| `WorktreeCreate` | A git worktree is created for an isolated agent |
| `WorktreeRemove` | A git worktree is cleaned up |

### MCP / Elicitation

| Event | Fires When |
|-------|-----------|
| `Elicitation` | Claude needs structured input from the user via an MCP elicitation |
| `ElicitationResult` | The user responds to an MCP elicitation |

---

## Hook Types (4)

| Type | Syntax | Description |
|------|--------|-------------|
| `command` | `{"type": "command", "command": "python3 script.py"}` | Run a shell command. Receives event data via environment variables. Stdout is injected back into context (for `PreToolUse`/`Stop`). Non-zero exit code blocks the action (for `PreToolUse`) |
| `prompt` | `{"type": "prompt", "prompt": "Review this for security issues"}` | Inject a prompt into Claude's context at the hook point. Useful for adding instructions or constraints |
| `agent` | `{"type": "agent", "agent": "security-reviewer"}` | Spawn a subagent to handle the event. The agent runs in its own context and can use tools |
| `http` | `{"type": "http", "url": "https://example.com/webhook"}` | Send event data to an HTTP endpoint via POST. Response body is injected back into context |

---

## Configuration Hierarchy

Hooks can be defined at multiple levels. Higher levels override lower levels for the same event:

| Level | Location | Scope |
|-------|----------|-------|
| **Managed** | `managed-settings.json` (MDM / Registry) | Organization-enforced, cannot be overridden |
| **Local settings** | `.claude/settings.local.json` | Personal project overrides (git-ignored) |
| **Shared settings** | `.claude/settings.json` | Team-shared project hooks |
| **Global settings** | `~/.claude/settings.json` | Personal defaults across all projects |
| **Agent frontmatter** | `.claude/agents/<name>.md` `hooks:` field | Scoped to a specific subagent |
| **Skill frontmatter** | `.claude/skills/<name>/SKILL.md` `hooks:` field | Scoped to a specific skill |

To disable all hooks, set `"disableAllHooks": true` in `.claude/settings.local.json`.

---

## Matcher Syntax

Matchers filter which tool invocations trigger a hook. They apply to `PreToolUse`, `PostToolUse`, and `PostToolUseFailure` events.

| Matcher | Matches |
|---------|---------|
| `"Bash"` | All Bash tool calls |
| `"Read"` | All Read tool calls |
| `"Write"` | All Write tool calls |
| `"Edit"` | All Edit tool calls |
| `"Glob"` | All Glob tool calls |
| `"Grep"` | All Grep tool calls |
| `"Agent"` | All Agent (subagent) tool calls |
| `"Skill"` | All Skill tool calls |
| `"WebFetch"` | All WebFetch tool calls |
| `"WebSearch"` | All WebSearch tool calls |
| `"mcp__<server>__<tool>"` | A specific MCP tool on a specific server |
| `"*"` | All tool calls (wildcard) |

Multiple matchers can be combined in an array: `["Bash", "Write", "Edit"]`.

---

## Environment Variables

Command-type hooks receive event data through environment variables:

| Variable | Available In | Description |
|----------|-------------|-------------|
| `CLAUDE_HOOK_EVENT` | All | The event name (e.g., `PreToolUse`, `Stop`) |
| `CLAUDE_HOOK_TOOL_NAME` | Tool events | The tool being called (e.g., `Bash`, `Write`) |
| `CLAUDE_HOOK_TOOL_INPUT` | `PreToolUse` | JSON string of the tool's input parameters |
| `CLAUDE_HOOK_TOOL_OUTPUT` | `PostToolUse` | JSON string of the tool's output |
| `CLAUDE_HOOK_ERROR` | Failure events | Error message from the failed operation |
| `CLAUDE_HOOK_SESSION_ID` | All | Current session identifier |
| `CLAUDE_HOOK_CWD` | All | Current working directory |
| `CLAUDE_HOOK_PROJECT_DIR` | All | Project root directory |
| `CLAUDE_HOOK_AGENT_NAME` | Agent events | Name of the subagent involved |
| `CLAUDE_HOOK_MATCHED` | Tool events | The matcher that triggered this hook |

---

## Practical Examples

### Sound Notification on Task Completion

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "afplay ~/.claude/hooks/sounds/done.mp3",
        "async": true
      }
    ]
  }
}
```

### Logging All Tool Calls

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) $CLAUDE_HOOK_TOOL_NAME\" >> ~/.claude/logs/tools.log",
        "async": true
      }
    ]
  }
}
```

### Security Audit — Block Dangerous Commands

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "python3 .claude/hooks/scripts/security-check.py"
      }
    ]
  }
}
```

The script checks `CLAUDE_HOOK_TOOL_INPUT` for dangerous patterns (`rm -rf /`, `curl | bash`, `chmod 777`). A non-zero exit code blocks the tool call.

### Auto-Format on File Write

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ["Write", "Edit"],
        "type": "command",
        "command": "prettier --write \"$CLAUDE_HOOK_TOOL_INPUT\" 2>/dev/null || true",
        "async": true
      }
    ]
  }
}
```

---

## Decision Control Patterns

`PreToolUse` hooks can return decision signals via stdout or exit code:

| Pattern | Mechanism | Effect |
|---------|-----------|--------|
| **Allow** | Exit code `0`, stdout contains `ALLOW` | Tool call proceeds without permission prompt |
| **Deny** | Non-zero exit code | Tool call is blocked, error message shown |
| **Ask** | Exit code `0`, stdout contains `ASK` | Standard permission prompt is shown to user |
| **Defer** | Exit code `0`, no output or empty stdout | Falls through to default permission handling |

For `prompt` and `agent` type hooks, the injected response guides Claude's behavior rather than directly controlling execution.

---

## Best Practices

1. **Use `async: true` for side-effects** — Sound notifications, logging, and telemetry should never block the agentic loop. Set `"async": true` on hooks that do not need to influence Claude's behavior.

2. **Use `once: true` for session hooks** — `SessionStart` hooks that perform setup (install dependencies, check environment) should run only once per session.

3. **Keep timeouts short** — Synchronous hooks block Claude's response. If your hook takes more than 2-3 seconds, make it async or optimize the script.

4. **Scope hooks with matchers** — Avoid running hooks on every tool call when you only need specific tools. Use matchers to reduce overhead.

5. **Use `command` for gating, `prompt` for guidance** — When you need hard allow/deny control, use a command hook with exit codes. When you need to influence Claude's approach, use a prompt hook.

6. **Test hooks in isolation** — Run your hook scripts manually with sample environment variables before adding them to settings.

7. **Log hook failures** — Wrap hook commands in error-handling logic that logs to a file, so silent failures are detectable.

---

## Common Pitfalls

| Pitfall | Details |
|---------|---------|
| **`Stop` vs `SubagentStop` confusion** | `Stop` fires for the **main** agent only. Use `SubagentStop` for subagent completion events. In earlier versions, `Stop` was called `AgentStop` — this was renamed |
| **Process spawning overhead** | Each `command` hook spawns a new shell process. For high-frequency events like `PostToolUse` with a wildcard matcher, this adds latency. Consider using `async: true` or consolidating logic into a single script |
| **Forgetting the matcher** | A `PreToolUse` hook without a `matcher` field fires for **all** tool calls. Always set a matcher unless you intentionally want universal coverage |
| **Synchronous hooks blocking responses** | A slow synchronous hook on `Stop` delays the user seeing Claude's response. Always make non-critical hooks async |
| **Hook output polluting context** | Stdout from synchronous command hooks is injected into Claude's context. Keep output minimal or redirect to a log file |
| **Settings hierarchy surprises** | A hook defined in `.claude/settings.json` can be overridden by `.claude/settings.local.json`. Check both files when debugging why a hook does not fire |

---

## See Also

- [Settings Best Practice](./claude-settings.md) — Configuration reference including hooks settings
- [Anti-Patterns Guide](./claude-anti-patterns.md) — Common hook pitfalls and how to avoid them
- [Troubleshooting Guide](./claude-troubleshooting.md) — Debugging hook issues and silent failures
- [Hooks Implementation](../implementation/claude-hooks-implementation.md) — Step-by-step implementation examples

---

## Sources

- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code Hooks Guide](https://code.claude.com/docs/en/hooks-guide)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
