# Agent vs Skill vs Command — Decision Tree

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

When to use an Agent, Skill, or Command — a decision framework with real-world scenarios.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Quick Comparison

| Dimension | Agent | Command | Skill |
|-----------|-------|---------|-------|
| **Location** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **Context** | Fresh isolated context | Injected into current context | Injected into current context (or forked with `context: fork`) |
| **Invocation** | `Agent(...)` tool, auto-discovery via description | `/slash-command` by user | `/slash-command`, auto-discovery, or agent preloading |
| **Memory** | Persistent (`user`/`project`/`local` scopes) | None | None |
| **Tools** | Custom allowlist via `tools:` field | Inherits session tools | Inherits session tools (or restricted via `allowed-tools:`) |
| **Model** | Configurable (`haiku`/`sonnet`/`opus`/`inherit`) | Configurable | Configurable |
| **Permissions** | Configurable (`acceptEdits`/`plan`/`bypassPermissions`) | Inherits session permissions | Inherits session permissions |
| **Auto-discovery** | Yes (via `description`, use `PROACTIVELY`) | No (user-triggered only) | Yes (via `description`) |
| **Preloadable** | No | No | Yes (via agent `skills:` field) |
| **Background** | Yes (`background: true`) | No | No |
| **Worktree isolation** | Yes (`isolation: worktree`) | No | No |
| **Best for** | Complex multi-step tasks needing isolation | User-triggered workflows and orchestration | Reusable domain knowledge and procedures |

---

## Decision Flowchart

```
Start: "I need to add a capability to Claude Code"
│
├─ Does it need its own isolated context?
│  ├─ YES ──→ Does it need persistent memory across sessions?
│  │          ├─ YES ──→ AGENT (with memory: user/project/local)
│  │          └─ NO  ──→ AGENT (or Skill with context: fork)
│  │
│  └─ NO ──→ Should only the user trigger it explicitly?
│             ├─ YES ──→ COMMAND (user types /slash-command)
│             └─ NO  ──→ Should Claude auto-discover and invoke it?
│                        ├─ YES ──→ SKILL (with description for discovery)
│                        └─ NO  ──→ Is it reusable domain knowledge?
│                                   ├─ YES ──→ SKILL (with user-invocable: false for preload-only)
│                                   └─ NO  ──→ COMMAND (simple prompt template)
```

### Quick Decision Rules

- **Need context isolation?** → Agent
- **Need persistent memory?** → Agent
- **Need background execution?** → Agent
- **Need worktree isolation?** → Agent
- **Need custom tool restrictions?** → Agent
- **User-triggered workflow?** → Command
- **Reusable procedure or domain knowledge?** → Skill
- **Auto-discoverable by Claude?** → Skill (or Agent with `PROACTIVELY` in description)
- **Preloadable into an agent?** → Skill

---

## Real-World Scenarios (12)

### 1. Code Review

**Recommendation**: Agent

**Why**: Code review needs isolated context so review findings do not pollute the main conversation. The agent can have its own tools (Read-only), model (haiku for speed), and persistent memory to learn team-specific review patterns over time.

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. Weather Fetch (Command → Agent → Skill Chain)

**Recommendation**: Command orchestrating an Agent with preloaded Skill

**Why**: The user triggers it explicitly (Command), the fetch logic runs in isolation (Agent), and the data-fetching instructions are reusable domain knowledge (Skill preloaded into the agent).

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. File Scaffolding

**Recommendation**: Skill

**Why**: Scaffolding templates are reusable domain knowledge. Claude can auto-discover the skill when the user asks to create a new component, route, or module. No isolation needed — files are written in the current project.

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. CI/CD Orchestration

**Recommendation**: Command

**Why**: CI/CD workflows are always user-triggered (`/deploy`, `/release`). They follow a fixed sequence and need access to the current session context (branch, recent changes).

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. Browser Testing

**Recommendation**: Agent

**Why**: Browser testing is long-running, may need background execution, and should not block the main conversation. The agent can have specific MCP servers (Playwright, Chrome DevTools) and run in a worktree for isolation.

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. Time Display

**Recommendation**: Skill

**Why**: Lightweight, no isolation needed, auto-discoverable. Claude invokes the skill whenever someone asks "what time is it?"

### 7. Multi-Step Deployment

**Recommendation**: Command orchestrating Agents

**Why**: The command is the user-facing entry point (`/full-deploy`). It spawns agents for each stage — build agent, test agent, deploy agent — each running in its own context with appropriate tools.

### 8. Linting / Formatting

**Recommendation**: Skill (auto-invoked)

**Why**: Linting rules are reusable domain knowledge. The skill auto-activates via `paths:` when Claude edits matching files. No user interaction needed.

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. Research Assistant

**Recommendation**: Agent

**Why**: Research tasks are long-running, benefit from memory (remembering past research), need web tools (`WebFetch`, `WebSearch`), and should run in isolated context to avoid cluttering the main conversation.

### 10. Data Migration

**Recommendation**: Agent with worktree isolation

**Why**: Database migrations modify critical files and should be tested in isolation. Running in a git worktree means the main branch is untouched until the migration is verified.

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. Documentation Generation

**Recommendation**: Skill

**Why**: Doc generation is a reusable procedure. Claude can auto-discover it when asked to "write docs" or "generate API reference." It runs in the current context and writes files directly to the project.

### 12. Project Setup

**Recommendation**: Command (with Setup hook)

**Why**: Project initialization is always user-triggered and runs once. Combine a `/setup` command with a `Setup` hook to auto-run on first clone.

---

## Composition Patterns

### Command → Agent → Skill

The canonical orchestration pattern. The command is the user entry point, the agent provides isolation and tool control, and skills provide domain knowledge.

```
User types /command
  └─ Command prompt runs in current context
       └─ Agent(...) spawned with isolated context
            └─ Preloaded skill provides instructions
                 └─ Agent completes, result surfaces to command context
                      └─ Skill tool invoked for final output
```

### Agent with Preloaded Skills

An agent loads skill content at startup via the `skills:` field. The skill content is injected into the agent's system prompt, giving it domain expertise without the user invoking anything.

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### Skill with Context Fork

A skill that needs isolation but not the full weight of an agent definition. Setting `context: fork` runs the skill in a subagent context while keeping the simple skill file format.

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## Anti-Patterns in Mechanism Choice

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|----------------|
| Agent for simple tasks | Overhead of isolated context when you just need instructions injected | Use a Skill |
| Command for auto-discovery | Commands cannot be auto-invoked by Claude | Use a Skill with a `description` |
| Skill for long-running tasks | Skills run in the main context and block the conversation | Use an Agent with `background: true` |
| Agent for shared conventions | Agent context is isolated — conventions do not carry to the main session | Use CLAUDE.md or Rules |
| Command for preloadable knowledge | Commands cannot be preloaded into agents | Use a Skill with `user-invocable: false` |
| Skill without `description` | Claude cannot auto-discover it | Always add a meaningful description |

---

## See Also

- [Skills Best Practice](./claude-skills.md) — Skill frontmatter, discovery, and monorepo patterns
- [Sub-agents Best Practice](./claude-subagents.md) — Agent frontmatter, isolation, and orchestration
- [Commands Best Practice](./claude-commands.md) — Built-in and custom slash command reference
- [Agents vs Commands vs Skills Report](../reports/claude-agent-command-skill.md) — Deep comparison of when to use each mechanism
- [Orchestration Workflow](../orchestration-workflow/orchestration-workflow.md) — End-to-end Command-Agent-Skill workflow example

---

## Sources

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
