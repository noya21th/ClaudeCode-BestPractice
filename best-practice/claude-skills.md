# Skills Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter fields, official bundled skills, skill types taxonomy, creation guide, invocation patterns, and monorepo discovery.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (13)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Display name and `/slash-command` identifier. Defaults to the directory name if omitted |
| `description` | string | Recommended | What the skill does. Shown in autocomplete and used by Claude for auto-discovery |
| `argument-hint` | string | No | Hint shown during autocomplete (e.g., `[issue-number]`, `[filename]`) |
| `disable-model-invocation` | boolean | No | Set `true` to prevent Claude from automatically invoking this skill |
| `user-invocable` | boolean | No | Set `false` to hide from the `/` menu — skill becomes background knowledge only, intended for agent preloading |
| `allowed-tools` | string | No | Tools allowed without permission prompts when this skill is active |
| `model` | string | No | Model to use when this skill runs (e.g., `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | Override the model effort level when invoked (`low`, `medium`, `high`, `max`) |
| `context` | string | No | Set to `fork` to run the skill in an isolated subagent context |
| `agent` | string | No | Subagent type when `context: fork` is set (default: `general-purpose`) |
| `hooks` | object | No | Lifecycle hooks scoped to this skill |
| `paths` | string/list | No | Glob patterns that limit when the skill auto-activates. Accepts a comma-separated string or YAML list — Claude loads the skill only when working with matching files |
| `shell` | string | No | Shell for `` !`command` `` blocks — `bash` (default) or `powershell`. Requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../!/tags/official.svg) **(5)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `simplify` | Review changed code for reuse, quality, and efficiency — refactors to eliminate duplication |
| 2 | `batch` | Run commands across multiple files in bulk |
| 3 | `debug` | Debug failing commands or code issues |
| 4 | `loop` | Run a prompt or slash command on a recurring interval (up to 3 days) |
| 5 | `claude-api` | Build apps with the Claude API or Anthropic SDK — triggers on `anthropic` / `@anthropic-ai/sdk` imports |

See also: [Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) for community-maintained installable skills.

---

## Skill Types Taxonomy (9)

Based on [Thariq's classification](https://www.thariq.io/blog/claude-code-best-practices/) of what skills are best suited for. Each type represents a distinct category of domain knowledge or automation that a skill can encapsulate.

| # | Type | Purpose | Example |
|---|------|---------|---------|
| 1 | **Library/API Reference** | Domain knowledge for APIs, frameworks, or SDKs — prevents hallucinated method signatures | Skill that embeds the latest Next.js App Router API surface |
| 2 | **Product Verification** | Testing and validation procedures — defines how to verify a feature works correctly | Skill that runs a full-story verification flow across UI, API, and data layers |
| 3 | **Data Fetching** | Retrieving external data from APIs, scraping sources, or file systems | Skill that fetches weather data from Open-Meteo API |
| 4 | **Business Process** | Organizational workflows and procedures — encodes team-specific runbooks | Skill that walks through a release checklist before deploying |
| 5 | **Code Scaffolding** | Project or file generation templates — standardizes boilerplate | Skill that generates a Next.js route with tests, types, and validation |
| 6 | **Code Quality** | Linting, formatting, and review standards — enforces team conventions | Skill that runs ESLint + Prettier + custom rule checks on changed files |
| 7 | **CI/CD** | Build, deploy, and release automation — codifies pipeline logic | Skill that triggers a staging deployment and waits for health checks |
| 8 | **Documentation** | Doc generation, formatting, and standards — keeps docs in sync with code | Skill that generates API reference from OpenAPI spec |
| 9 | **Security** | Vulnerability scanning, audit procedures, and compliance checks | Skill that scans dependencies for known CVEs before merging |

---

## Creating Custom Skills

### Step 1: Create the directory

```
.claude/skills/my-skill/
```

### Step 2: Create SKILL.md with frontmatter

```yaml
---
name: my-skill
description: What this skill does — used for auto-discovery and autocomplete
argument-hint: "[optional-argument]"
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# My Skill

## Task
Describe what the skill accomplishes.

## Instructions
1. First step
2. Second step
3. Third step

## Output
What the skill produces or where it writes results.
```

### Step 3: Add supporting files (optional)

```
.claude/skills/my-skill/
  SKILL.md           # Required — frontmatter + instructions
  reference.md       # Optional — API surface, type definitions
  examples.md        # Optional — usage examples, edge cases
```

Supporting files are referenced from SKILL.md body using relative links. Claude reads them when the skill is invoked, giving the skill access to detailed domain knowledge without bloating the main instruction file.

### Step 4: Test with the slash command

```bash
$ claude
> /my-skill
```

### Step 5: Verify auto-discovery

If your skill has a `description` field and `disable-model-invocation` is not `true`, Claude will automatically discover and invoke the skill when the conversation context matches. Test by describing the task without explicitly invoking the slash command — Claude should pick up the skill on its own.

---

## Skill Patterns

Four patterns for how skills are invoked and loaded:

| Pattern | Key Config | How It Works |
|---------|-----------|--------------|
| **User Skill** | `user-invocable: true` (default) | Appears in the `/` autocomplete menu. User invokes it explicitly via `/skill-name` or Claude discovers it from `description` |
| **Agent Skill** | `user-invocable: false` | Hidden from `/` menu. Preloaded into a subagent via the `skills:` frontmatter field. Full content is injected into the agent's context at startup — acts as domain knowledge, not a standalone command |
| **Auto-invoke Skill** | `description` triggers discovery | Claude reads the `description` of all skills and automatically invokes the skill when the conversation matches. No slash command needed. Disable with `disable-model-invocation: true` |
| **Forked Skill** | `context: fork` | Runs in an isolated subagent context. The skill gets its own conversation thread and does not pollute the main conversation. Use `agent:` to specify which subagent type runs it |

### User Skill example

```yaml
---
name: weather-svg-creator
description: Creates an SVG weather card for Dubai
---
```

Invoked via `/weather-svg-creator` or auto-discovered when the user asks about weather visualization.

### Agent Skill example

```yaml
---
name: weather-fetcher
description: Instructions for fetching weather data from Open-Meteo API
user-invocable: false
---
```

Preloaded into the weather-agent via:

```yaml
# In .claude/agents/weather-agent.md frontmatter
skills:
  - weather-fetcher
```

### Forked Skill example

```yaml
---
name: security-audit
description: Run a security audit on the current project
context: fork
agent: general-purpose
allowed-tools: Read, Bash, Grep, Glob
---
```

Runs in isolation — the audit results are returned to the main conversation when the forked context completes, but intermediate steps do not appear in the main thread.

---

## Monorepo Skill Discovery

Claude discovers skills by walking the directory tree from the project root. In monorepos with nested projects, this means skills are contextual to where Claude is working.

| Rule | Behavior |
|------|----------|
| **Root discovery** | Skills in `.claude/skills/` at the project root are always available |
| **Nested discovery** | When Claude visits a subdirectory that has its own `.claude/skills/`, those skills are discovered and become available |
| **Description-first loading** | Only the `description` frontmatter field is loaded for discovery. The full SKILL.md content is loaded on invocation — this keeps discovery lightweight |
| **Priority: nearest first** | If two skills have the same name, the one in the nearest `.claude/skills/` directory wins |
| **Nested project isolation** | A subdirectory with its own `.claude/` is treated as a nested project. Its skills are available when Claude works in that subtree |

### Example: monorepo layout

```
my-monorepo/
  .claude/skills/
    shared-lint/SKILL.md         # Available everywhere
  packages/
    frontend/
      .claude/skills/
        component-gen/SKILL.md   # Available when working in packages/frontend/
    backend/
      .claude/skills/
        api-scaffold/SKILL.md    # Available when working in packages/backend/
```

When Claude works on a file in `packages/frontend/`, it sees both `shared-lint` (from root) and `component-gen` (from the nearest `.claude/skills/`). It does not see `api-scaffold` unless it navigates into `packages/backend/`.

---

## See Also

- [Agent vs Skill vs Command — Decision Tree](./claude-decision-tree.md) — When to use a skill vs an agent vs a command
- [Sub-agents Best Practice](./claude-subagents.md) — How agents preload and invoke skills
- [Skills Implementation](../implementation/claude-skills-implementation.md) — Step-by-step skill creation examples
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md) — How skill resolution works in nested project structures

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Thariq's Claude Code Best Practices — Skill Types](https://www.thariq.io/blog/claude-code-best-practices/)
- [Agent, Command, Skill Relationship](../reports/claude-agent-command-skill.md)
