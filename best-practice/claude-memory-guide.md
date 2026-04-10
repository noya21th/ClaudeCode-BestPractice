# Memory & Context Management Guide

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Comprehensive guide to Claude Code's memory systems — CLAUDE.md files, auto-memory, agent memory, and rules.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Memory Types Overview

Claude Code has four distinct memory systems, each serving a different purpose:

| Memory Type | Location | Scope | Persistence | Loaded When |
|-------------|----------|-------|-------------|-------------|
| **CLAUDE.md** | Project directories | Per-project, hierarchical | Permanent (file-based) | Session start (ancestors) or on file access (descendants) |
| **Auto-memory** | `~/.claude/memory/` and `~/.claude/projects/<hash>/memory/` | User-global or per-project | Permanent (file-based) | Session start |
| **Agent memory** | `~/.claude/agent-memory/<agent>/` or `.claude/agent-memory/<agent>/` | Per-agent, scoped by `memory:` field | Permanent (file-based) | Agent startup |
| **Rules** | `.claude/rules/*.md` and `~/.claude/rules/*.md` | File-pattern-scoped | Permanent (file-based) | When files matching glob patterns are accessed |

---

## CLAUDE.md Hierarchy

### File Locations

| Level | Path | Shared | Description |
|-------|------|--------|-------------|
| **Global** | `~/.claude/CLAUDE.md` | No (personal) | Applies to all Claude Code sessions on this machine |
| **Project root** | `./CLAUDE.md` | Yes (git-tracked) | Team-shared project-wide instructions |
| **Project local** | `./CLAUDE.local.md` | No (git-ignored) | Personal project-specific overrides |
| **Component** | `./frontend/CLAUDE.md` | Yes (git-tracked) | Component-specific instructions |
| **Nested** | `./src/api/v2/CLAUDE.md` | Yes (git-tracked) | Deep directory-specific instructions |

### Loading Mechanisms

Claude Code uses two distinct mechanisms for discovering CLAUDE.md files:

#### Ancestor Loading (Up the Tree) — Eager

When a session starts, Claude walks **upward** from the current working directory to the filesystem root and loads every CLAUDE.md it finds. These are loaded **immediately at startup**.

```
/home/user/projects/myapp/src/   ← cwd
/home/user/projects/myapp/       ← CLAUDE.md loaded (ancestor)
/home/user/projects/             ← CLAUDE.md loaded (ancestor)
/home/user/                      ← CLAUDE.md loaded if exists (ancestor)
~/.claude/                       ← CLAUDE.md loaded (global)
```

#### Descendant Loading (Down the Tree) — Lazy

CLAUDE.md files in subdirectories below the cwd are **not** loaded at startup. They are included only when Claude reads or edits files in those subdirectories during the session.

```
/myapp/                 ← cwd, CLAUDE.md loaded at startup
/myapp/frontend/        ← CLAUDE.md loaded when Claude accesses frontend/ files
/myapp/backend/         ← CLAUDE.md loaded when Claude accesses backend/ files
/myapp/backend/api/     ← CLAUDE.md loaded when Claude accesses backend/api/ files
```

#### Key Rules

- **Ancestors always load at startup** — eagerly, upward from cwd
- **Descendants load lazily** — only when files in that directory are accessed
- **Siblings never load** — working in `frontend/` does not load `backend/CLAUDE.md`
- **Global always loads** — `~/.claude/CLAUDE.md` applies to every session

---

## Monorepo Strategies

### Root CLAUDE.md — Shared Conventions

Keep the root file focused on team-wide standards that apply everywhere:

```markdown
# CLAUDE.md (root)

## Coding Standards
- TypeScript strict mode everywhere
- No `any` types without justification comments
- Prefer functional components with hooks

## Git Conventions
- Conventional commits: feat/fix/chore/docs/refactor
- Squash merge to main

## Testing
- Every PR must include tests
- Minimum 80% coverage for new code
```

### Component CLAUDE.md — Specifics

Each component gets its own focused instructions:

```markdown
# frontend/CLAUDE.md

## Stack
- Next.js 15 App Router
- Tailwind CSS + shadcn/ui
- React Query for server state

## Patterns
- Server Components by default
- Client Components only for interactivity
- Use app/api/ for API routes
```

### CLAUDE.local.md — Personal Preferences

Personal overrides that should never be committed:

```markdown
# CLAUDE.local.md (git-ignored)

- Use verbose comments — I am learning this codebase
- Run prettier after every file edit
- Always explain your reasoning before making changes
```

---

## Auto-Memory System

Auto-memory is Claude Code's system for remembering things across conversations. It stores observations and learnings that persist between sessions.

### Types

| Type | Stored In | Description |
|------|-----------|-------------|
| `user` | `~/.claude/memory/` | User preferences and global patterns — applies to all projects |
| `feedback` | `~/.claude/memory/` | Corrections and behavioral adjustments from user feedback |
| `project` | `~/.claude/projects/<hash>/memory/` | Project-specific learnings and context |
| `reference` | `~/.claude/projects/<hash>/memory/` | Reference material Claude found useful for this project |

### How It Works

1. Claude observes a pattern, preference, or correction during a session
2. If the observation is likely useful in future sessions, Claude stores it as auto-memory
3. On the next session start, relevant auto-memory entries are loaded into context
4. Users can view and manage entries via `/memory`

### When to Use Auto-Memory vs CLAUDE.md

| Use Auto-Memory When | Use CLAUDE.md When |
|----------------------|-------------------|
| The instruction emerged from feedback during a session | The instruction is planned upfront |
| It is a personal preference (tone, verbosity, workflow) | It is a team convention (coding standards, architecture) |
| It evolves over time through usage | It is stable and reviewed in code review |
| It applies to the user across projects | It applies to the project regardless of user |

---

## Agent Memory

Agents with the `memory:` field in their frontmatter maintain persistent memory that survives across sessions. This allows agents to learn and improve over time.

### Scopes

| Scope | Stored In | Shared | Description |
|-------|-----------|--------|-------------|
| `user` | `~/.claude/agent-memory/<agent>/` | No | Cross-project memory for this agent — follows the user |
| `project` | `.claude/agent-memory/<agent>/` | Yes | Project-specific memory — shared with the team via git |
| `local` | `.claude/agent-memory/<agent>/` (git-ignored) | No | Project-specific, personal memory |

### Configuration

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes"
memory: project
---
Review code changes. After each review, note patterns you see frequently
so you can flag them earlier in future reviews.
```

### Use Cases

- **Code reviewer** with `memory: project` — learns team-specific patterns, common bugs, style preferences
- **Research assistant** with `memory: user` — remembers topics explored, preferred sources, writing style
- **Deployment agent** with `memory: project` — tracks deployment history, known environment quirks

---

## Rules (.claude/rules/)

Rules are glob-scoped markdown files that load automatically when Claude accesses files matching the glob pattern.

### Location

| Path | Scope |
|------|-------|
| `.claude/rules/*.md` | Project-level rules (git-tracked, team-shared) |
| `~/.claude/rules/*.md` | Global rules (personal, applies to all projects) |

### Format

Each rule file begins with a `# Glob:` line that defines when the rule activates:

```markdown
# Glob: **/*.tsx

## React Component Rules

- Always use functional components
- Export components as named exports, not default
- Co-locate styles in a .module.css file
```

```markdown
# Glob: **/*.test.*

## Testing Rules

- Use describe/it blocks, not test()
- Mock external dependencies, not internal modules
- Assert on behavior, not implementation
```

### When to Use Rules vs CLAUDE.md

| Use Rules When | Use CLAUDE.md When |
|----------------|-------------------|
| Instructions apply to specific file types | Instructions apply to the whole project |
| You want lazy loading based on file access | You want eager loading at session start |
| Guidelines are scoped to a pattern (`*.py`, `*.tsx`, `migrations/*.sql`) | Guidelines are universal (git conventions, architecture decisions) |

---

## Comparison Table

| Feature | CLAUDE.md | Auto-Memory | Agent Memory | Rules |
|---------|-----------|-------------|--------------|-------|
| **Created by** | Developer (manually) | Claude (automatically) | Agent (automatically) | Developer (manually) |
| **Shared with team** | Yes (except `.local.md`) | No | Yes (if `project` scope) | Yes |
| **Loads at startup** | Yes (ancestors) | Yes | Yes (agent start) | No (lazy) |
| **Scoped to file types** | No | No | No | Yes (glob patterns) |
| **Editable by user** | Yes (file edit) | Yes (`/memory`) | Yes (file edit) | Yes (file edit) |
| **Survives across sessions** | Yes | Yes | Yes | Yes |
| **Best for** | Project conventions | Learned preferences | Agent improvement | File-type-specific guidance |

---

## Best Practices

### Keep CLAUDE.md Under 200 Lines

Long CLAUDE.md files cause context bloat and Claude may ignore instructions near the bottom. Split large files into component-level CLAUDE.md files and rules.

### Use Rules for File-Type Guidance

Do not put "when editing Python files, do X" in CLAUDE.md. Create `.claude/rules/python.md` with `# Glob: **/*.py` instead. This loads only when needed.

### Use Agent Memory for Evolving Agents

If an agent should get smarter over time, give it a `memory:` scope. The agent stores observations that improve future runs — review patterns, code smells, deployment quirks.

### Compact at 50% Context Usage

Run `/context` periodically to check usage. When it reaches 50%, run `/compact` with optional focus instructions: `/compact focus on the authentication refactor`. This preserves important context while freeing space.

### Git-Ignore Personal Files

Always add to `.gitignore`:

```
CLAUDE.local.md
.claude/settings.local.json
```

### Separate Stable from Evolving Instructions

- **Stable** (coding standards, architecture) → CLAUDE.md and rules
- **Evolving** (preferences, patterns, learnings) → auto-memory and agent memory

---

## Staleness and Cleanup

### Signs of Stale Memory

- CLAUDE.md references deprecated APIs or removed features
- Auto-memory entries contradict current CLAUDE.md instructions
- Agent memory contains outdated project-specific patterns
- Rules reference file patterns that no longer exist in the codebase

### Cleanup Strategies

1. **Audit CLAUDE.md quarterly** — Read each instruction and verify it still applies
2. **Review auto-memory** — Run `/memory` and delete entries that are no longer relevant
3. **Prune agent memory** — Check `.claude/agent-memory/` for stale observations
4. **Update rules** — Verify glob patterns still match files in the project
5. **Track with comments** — Add `<!-- Last reviewed: 2026-04-09 -->` to CLAUDE.md sections

---

## See Also

- [Claude Memory](./claude-memory.md) — CLAUDE.md writing patterns and monorepo loading behavior
- [Agent Memory Report](../reports/claude-agent-memory.md) — Agent memory frontmatter and persistence scopes
- [Settings Best Practice](./claude-settings.md) — Configuration that affects memory and context behavior
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md) — How skill discovery interacts with monorepo memory layout

---

## Sources

- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code Rules — Docs](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)
- [Claude Code Sub-agents (Memory) — Docs](https://code.claude.com/docs/en/sub-agents)
- [CLAUDE.md in Monorepos](../best-practice/claude-memory.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
