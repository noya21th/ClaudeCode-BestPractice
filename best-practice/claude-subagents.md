# Sub-agents Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-subagents-implementation.md)

Claude Code subagents — frontmatter fields, official built-in agent types, when to use subagents, configuration best practices, multi-agent patterns, and error handling.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique identifier using lowercase letters and hyphens |
| `description` | string | Yes | When to invoke. Use `"PROACTIVELY"` for auto-invocation by Claude |
| `tools` | string/list | No | Comma-separated allowlist of tools (e.g., `Read, Write, Edit, Bash`). Inherits all tools if omitted. Supports `Agent(agent_type)` syntax to restrict spawnable subagents; the older `Task(agent_type)` alias still works |
| `disallowedTools` | string/list | No | Tools to deny, removed from inherited or specified list |
| `model` | string | No | Model to use: `sonnet`, `opus`, `haiku`, a full model ID (e.g., `claude-opus-4-6`), or `inherit` (default: `inherit`) |
| `permissionMode` | string | No | Permission mode: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, or `plan` |
| `maxTurns` | integer | No | Maximum number of agentic turns before the subagent stops |
| `skills` | list | No | Skill names to preload into agent context at startup (full content injected, not just made available) |
| `mcpServers` | list | No | MCP servers for this subagent — server name strings or inline `{name: config}` objects |
| `hooks` | object | No | Lifecycle hooks scoped to this subagent. All hook events are supported; `PreToolUse`, `PostToolUse`, and `Stop` are the most common |
| `memory` | string | No | Persistent memory scope: `user`, `project`, or `local` |
| `background` | boolean | No | Set to `true` to always run as a background task (default: `false`) |
| `effort` | string | No | Effort level override when this subagent is active: `low`, `medium`, `high`, `max` (Opus 4.6 only). Default: inherits from session |
| `isolation` | string | No | Set to `"worktree"` to run in a temporary git worktree (auto-cleaned if no changes) |
| `initialPrompt` | string | No | Auto-submitted as the first user turn when this agent runs as the main session agent (via `--agent` or the `agent` setting). Commands and skills are processed. Prepended to any user-provided prompt |
| `color` | string | No | Display color for the subagent in the task list and transcript: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, or `cyan` |

---

## ![Official](../!/tags/official.svg) **(5)**

| # | Agent | Model | Tools | Description |
|---|-------|-------|-------|-------------|
| 1 | `general-purpose` | inherit | All | Complex multi-step tasks — the default agent type for research, code search, and autonomous work |
| 2 | `Explore` | haiku | Read-only (no Write, Edit) | Fast codebase search and exploration — optimized for finding files, searching code, and answering codebase questions |
| 3 | `Plan` | inherit | Read-only (no Write, Edit) | Pre-planning research in plan mode — explores the codebase and designs implementation approaches before writing code |
| 4 | `statusline-setup` | sonnet | Read, Edit | Configures the user's Claude Code status line setting |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Answers questions about Claude Code features, Agent SDK, and Claude API |

---

## When to Use Subagents

Use a subagent when you need one or more of the following:

| Criteria | Why a Subagent Helps |
|----------|---------------------|
| **Multi-step autonomous execution** | The task requires many tool calls in sequence — a subagent can run through them without polluting the main conversation |
| **Context isolation** | The task's intermediate steps (file reads, searches, trial-and-error) should not fill the parent's context window |
| **Persistent memory across sessions** | Set `memory: project` or `memory: local` to let the agent accumulate knowledge that survives conversation restarts |
| **Background execution** | Set `background: true` to run the agent as a task while the main conversation continues |
| **Different permission mode** | The task needs a stricter or more permissive mode (e.g., `plan` for research-only, `bypassPermissions` for fully autonomous CI) |
| **Tool restrictions** | The task should only have access to a subset of tools (e.g., read-only exploration with no Write or Edit) |
| **Git isolation** | Set `isolation: "worktree"` to let the agent work in a temporary git worktree — changes are discarded if the agent produces nothing useful |

---

## When NOT to Use Subagents

| Scenario | Better Alternative |
|----------|-------------------|
| **Simple one-shot tasks** | Use a Skill instead — skills run in the current context and return immediately |
| **User needs to see intermediate results** | Run the task inline in the main conversation so the user can observe and redirect |
| **Task requires main conversation context** | Subagents start with a clean context. If the task depends on prior conversation history, keep it in the main thread or pass the relevant context explicitly in the prompt |
| **Quick file lookups or searches** | Use the built-in `Explore` agent type (`model: haiku`) — it is optimized for fast read-only operations |

---

## Configuration Best Practices

### maxTurns

Set `maxTurns` based on task complexity to prevent runaway agents:

| Complexity | Recommended maxTurns | Example |
|-----------|---------------------|---------|
| Simple (lookup, single edit) | 3-5 | Fetching data from an API |
| Medium (multi-file edit, search + modify) | 8-12 | Refactoring a module |
| Complex (research + implement + test) | 15-25 | Building a feature end-to-end |
| Autonomous (long-running CI/CD) | 30+ | Running a full test suite and fixing failures |

### model

| Model | When to Use |
|-------|------------|
| `haiku` | Fast, cheap tasks — file search, exploration, simple lookups |
| `sonnet` | Balanced — code generation, moderate reasoning, most everyday tasks |
| `opus` | Complex reasoning — architecture decisions, nuanced code review, multi-step planning |
| `inherit` | Use the parent session's model (default) |

### memory

| Scope | Stored At | Shared With |
|-------|----------|-------------|
| `user` | `~/.claude/agent-memory/<name>/MEMORY.md` | All projects for this user |
| `project` | `.claude/agent-memory/<name>/MEMORY.md` | All team members (committed to git) |
| `local` | `.claude/agent-memory-local/<name>/MEMORY.md` | Only this user on this machine (git-ignored) |

### permissionMode

| Mode | Behavior |
|------|----------|
| `default` | Prompts for every tool use that requires permission |
| `acceptEdits` | Auto-approves file edits (Write, Edit), prompts for everything else |
| `auto` | Auto-approves all tools that match the session's permission rules |
| `plan` | Read-only — the agent can explore but cannot write or execute |
| `bypassPermissions` | No permission prompts at all (use only in trusted CI/CD environments) |

### tools

Follow the principle of least privilege — restrict tools to the minimum needed:

```yaml
# Read-only research agent
tools: Read, Glob, Grep, WebFetch, WebSearch

# Code modification agent
tools: Read, Write, Edit, Bash, Glob, Grep

# Agent that can spawn only specific subagents
tools: Read, Write, Edit, Agent(code-reviewer), Agent(test-runner)
```

---

## Multi-Agent Patterns

### Sequential Chain

Agent A completes, then Agent B starts with Agent A's output:

```
Command → Agent(researcher) → Agent(implementer) → Agent(reviewer)
```

The command orchestrates the chain, passing each agent's result to the next as the prompt input.

### Fan-out (Parallel)

Parent spawns multiple agents in parallel for independent tasks:

```
Command → Agent(frontend-checker)  (background: true)
        → Agent(backend-checker)   (background: true)
        → Agent(security-scanner)  (background: true)
```

Use `background: true` so all three run concurrently. The parent collects results when each completes.

### Orchestrator

A command or agent coordinates multiple agents based on dynamic conditions:

```yaml
# In a command
1. Agent(planner) → produces a task list
2. For each task: Agent(implementer) with task-specific prompt
3. Agent(reviewer) → reviews all changes
```

### Specialist Team

Each agent has domain expertise via preloaded skills:

```yaml
# weather-agent.md
skills:
  - weather-fetcher      # Domain knowledge for API
model: sonnet
tools: WebFetch, Read, Write

# svg-agent.md
skills:
  - svg-design-system    # Domain knowledge for SVG
model: haiku
tools: Read, Write, Edit
```

The command invokes each specialist for the part of the workflow that matches their expertise.

---

## Error Handling and Degradation

| Mechanism | How It Protects You |
|-----------|-------------------|
| **maxTurns limit** | Prevents runaway agents — the agent stops after N turns regardless of completion state |
| **Agent tool returns result** | The parent always receives the agent's final output, allowing verification before acting on it |
| **Stop hooks** | Use `Stop` hooks in agent frontmatter to run cleanup scripts when the agent finishes (e.g., close connections, delete temp files) |
| **SubagentStart/SubagentStop hooks** | Session-level hooks that fire when any subagent starts or stops — use for logging, metrics, or sound notifications |
| **Worktree isolation** | `isolation: "worktree"` runs the agent in a temporary git worktree. If the agent fails or produces bad output, the worktree is auto-cleaned with no impact on the main branch |
| **Permission escalation prevention** | A subagent cannot grant itself more permissions than its `permissionMode` allows, even if instructed to in the prompt |

### Degradation strategy

1. Set `maxTurns` conservatively — you can always re-invoke with a higher limit
2. Use `plan` mode for research phases, then `acceptEdits` for implementation
3. Scope `tools` narrowly — an agent that cannot Write cannot corrupt your codebase
4. Use `memory: project` to let agents learn from failures across sessions
5. Monitor via `SubagentStart` and `SubagentStop` hooks to track agent behavior over time

---

## See Also

- [Agent vs Skill vs Command — Decision Tree](./claude-decision-tree.md) — When to use an agent vs a skill vs a command
- [Skills Best Practice](./claude-skills.md) — Skills that agents can preload or invoke
- [Sub-agents Implementation](../implementation/claude-subagents-implementation.md) — Step-by-step agent creation examples
- [Agent Memory Report](../reports/claude-agent-memory.md) — Memory frontmatter and persistence scopes for agents

---

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [CLI reference — Claude Code Docs](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Agent Memory Report](../reports/claude-agent-memory.md)
- [Agent, Command, Skill Relationship](../reports/claude-agent-command-skill.md)
