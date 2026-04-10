# Anti-Patterns Guide

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Common mistakes in Claude Code configuration and how to avoid them — organized by category.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Architecture Anti-Patterns

### 1. Monolithic CLAUDE.md

**Problem**: A single CLAUDE.md file exceeding 500 lines with installation instructions, coding conventions, API docs, deployment steps, and debugging tips all mixed together.

**Symptoms**: Claude ignores instructions near the bottom of the file. Inconsistent adherence to conventions. Context bloat at session start.

**Fix**: Split into focused files. Keep root CLAUDE.md under 200 lines for shared conventions. Use component-level CLAUDE.md files for specifics. Use `.claude/rules/*.md` with glob patterns for file-type-specific guidance.

```
# Before (monolithic)
CLAUDE.md              # 800 lines covering everything

# After (split)
CLAUDE.md              # 150 lines — shared conventions, project overview
frontend/CLAUDE.md     # 80 lines — React patterns, component structure
backend/CLAUDE.md      # 80 lines — API conventions, database patterns
.claude/rules/ts.md    # TypeScript-specific rules (Glob: **/*.ts)
.claude/rules/test.md  # Testing conventions (Glob: **/*.test.*)
```

### 2. Agent for Everything

**Problem**: Using agents for simple tasks that only need instructions injected into the current context.

**Symptoms**: Unnecessary context isolation. Results not visible in the main conversation. Slow execution due to agent startup overhead. Token waste from duplicated context.

**Fix**: Use a Skill for reusable domain knowledge and procedures. Use a Command for user-triggered prompt templates. Reserve Agents for tasks that genuinely need isolation, memory, tool restrictions, or background execution.

### 3. Missing Context Isolation

**Problem**: Running long exploratory tasks (research, code review, data analysis) directly in the main conversation without isolation.

**Symptoms**: Conversation context fills up quickly. Irrelevant intermediate results pollute future responses. Need to `/compact` frequently.

**Fix**: Use an Agent for tasks that generate large intermediate output. The agent runs in fresh context and returns only the summary.

### 4. Circular Agent Spawning

**Problem**: Agent A spawns Agent B, which spawns Agent A, creating an infinite loop.

**Symptoms**: `maxTurns` limit hit. Token budget exhausted. No useful output.

**Fix**: Design DAG (directed acyclic graph) workflows. Use the `tools:` field to restrict which agents a subagent can spawn. Review agent definitions for circular `Agent(...)` references.

```yaml
# Restrict spawnable agents
---
name: planner
tools: Read, Glob, Grep, Agent(implementer)
# planner can only spawn implementer, not itself
---
```

### 5. Hardcoded Model Names

**Problem**: Using full model IDs like `claude-sonnet-4-20250514` in agent definitions.

**Symptoms**: Agents break when model IDs are deprecated. Inconsistent behavior across environments.

**Fix**: Use aliases: `haiku`, `sonnet`, `opus`, or `inherit`. These always resolve to the latest version.

---

## Configuration Anti-Patterns

### 6. Overly Permissive Permissions

**Problem**: Setting `"permissionMode": "bypassPermissions"` on all agents and using `--dangerously-skip-permissions` as default.

**Symptoms**: Unreviewed file deletions. Unintended `git push --force`. Security risk from untrusted MCP tools. No audit trail of tool approvals.

**Fix**: Use `acceptEdits` for agents that need to write files but should not run arbitrary commands. Use `plan` for research-only agents. Reserve `bypassPermissions` only for CI/CD environments with sandboxing.

| Mode | Edits | Bash | Best For |
|------|-------|------|----------|
| `default` | Ask | Ask | Interactive development |
| `acceptEdits` | Auto | Ask | File-writing agents |
| `plan` | Deny | Deny | Research and planning agents |
| `bypassPermissions` | Auto | Auto | CI/CD with sandbox only |

### 7. Ignoring Settings Hierarchy

**Problem**: Defining settings in the wrong file and wondering why they are overridden or ignored.

**Symptoms**: Hooks do not fire. Permissions are not what you expect. Team members have different behavior.

**Fix**: Understand the precedence order (highest wins):

1. **Managed** (`managed-settings.json`) — Organization-enforced
2. **CLI flags** — Single-session overrides
3. **Local** (`.claude/settings.local.json`) — Personal project settings
4. **Shared** (`.claude/settings.json`) — Team-shared project settings
5. **Global** (`~/.claude/settings.json`) — Personal defaults

### 8. All Hooks Synchronous

**Problem**: Every hook runs synchronously, blocking Claude's response while scripts execute.

**Symptoms**: Slow responses. Noticeable pause after every tool call. User perceives Claude as laggy.

**Fix**: Set `"async": true` on hooks that perform side-effects (logging, notifications, telemetry). Only keep hooks synchronous when they need to gate or modify behavior (security checks, permission decisions).

### 9. No settings.local.json

**Problem**: Personal preferences (sound notifications, custom paths, debug logging) committed to `.claude/settings.json` where they affect the entire team.

**Symptoms**: Teammates hear your notification sounds. Debug hooks run in production. Personal MCP servers appear for everyone.

**Fix**: Use `.claude/settings.local.json` for personal overrides. Ensure it is in `.gitignore`. Only share hooks and settings that the entire team needs.

---

## Skill Anti-Patterns

### 10. user-invocable: true on Agent-Only Skills

**Problem**: A skill intended only for agent preloading (via `skills:` field) appears in the user's `/` slash-command menu.

**Symptoms**: Users invoke the skill directly and get confusing results because it was designed as background knowledge for an agent.

**Fix**: Set `user-invocable: false` on skills that are only meant for agent preloading.

```yaml
---
name: internal-api-conventions
user-invocable: false
description: "REST API conventions for the internal platform"
---
```

### 11. Missing Description

**Problem**: A skill has no `description` field in its frontmatter.

**Symptoms**: Claude cannot auto-discover the skill. It only fires when the user explicitly types the `/slash-command`. The skill is invisible to auto-invocation logic.

**Fix**: Always include a `description` that explains **when** to use the skill. Use keywords Claude would encounter naturally: "Use when creating React components" or "Triggers on Python test files."

### 12. Giant Skill Files

**Problem**: A single SKILL.md file with 1000+ lines covering every edge case, example, and reference.

**Symptoms**: Context bloat when the skill is loaded. Claude may ignore instructions near the end. Slow to load when preloaded into agents.

**Fix**: Keep SKILL.md focused on the core procedure (under 200 lines). Move reference material to `references/` files within the skill directory. Use `@path` imports for supplementary content that Claude loads on demand.

### 13. disable-model-invocation on Frequently Needed Skills

**Problem**: Setting `disable-model-invocation: true` on a skill that Claude should auto-discover and invoke.

**Symptoms**: Claude never invokes the skill automatically. Users must always type the `/slash-command` manually even when the context clearly calls for the skill.

**Fix**: Only use `disable-model-invocation: true` on skills that are purely user-triggered (dangerous operations, deployment, etc.). For skills Claude should discover on its own, leave it unset or set to `false`.

---

## Operational Anti-Patterns

### 14. Never Compacting

**Problem**: Running a long session without compacting, letting context fill to 100%.

**Symptoms**: Claude starts forgetting earlier instructions. Responses become less accurate. Session eventually errors with context overflow.

**Fix**: Run `/compact` manually at approximately 50% context usage. Use `/context` to visualize current usage. For long tasks, break work into subtasks that each complete within 50% of context capacity.

### 15. No Verification Loops

**Problem**: Accepting Claude's output without verifying it works — no test runs, no type checks, no visual inspection.

**Symptoms**: Generated code has syntax errors. API integrations use wrong endpoints. Configuration files have typos.

**Fix**: Always verify output. Run tests after code generation. Use browser automation (Chrome, Playwright) for visual verification. Include verification steps in your commands and agent definitions.

### 16. Ignoring Rate Limits

**Problem**: Planning large multi-agent tasks without considering token budget and rate limits.

**Symptoms**: Agents hit rate limits mid-task. Work is lost or incomplete. Billing surprises.

**Fix**: Use `/usage` to check your current rate limit status. Plan tasks to fit within your budget. Use `model: haiku` for high-volume, low-complexity subtasks. Break large tasks into smaller sessions.

---

## See Also

- [Settings Best Practice](./claude-settings.md) — Correct configuration patterns to avoid misconfiguration anti-patterns
- [Skills Best Practice](./claude-skills.md) — Proper skill design to prevent discovery and context issues
- [Sub-agents Best Practice](./claude-subagents.md) — Agent patterns that avoid isolation and orchestration mistakes
- [Memory & Context Management Guide](./claude-memory-guide.md) — Managing context to prevent overflow and staleness
- [Troubleshooting Guide](./claude-troubleshooting.md) — Diagnosing issues that stem from anti-patterns

---

## Sources

- [Claude Code Best Practices — Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Permissions — Docs](https://code.claude.com/docs/en/permissions)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
