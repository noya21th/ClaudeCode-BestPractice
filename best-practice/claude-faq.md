# Claude Code FAQ — Billion-Dollar Questions

Answers to the open questions from the [README](../README.md#billion-dollar-questions), informed by patterns documented across this repository.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Memory & Instructions

### 1. What exactly should you put inside your CLAUDE.md — and what should you leave out?

Keep it short. Boris Cherny's personal CLAUDE.md is two lines. The [Memory Guide](claude-memory-guide.md) recommends a layered approach:

- **Project-level CLAUDE.md** (shared, checked in): repo conventions, tech stack, test commands, PR workflow, architectural constraints. Contributed to by the whole team.
- **Personal `~/.claude/CLAUDE.md`** (global): your individual preferences (automerge, notification channels, editor style).
- **Rules files** (`.claude/rules/*.md`): glob-scoped instructions for specific file types or directories.

**Leave out**: anything the model can infer from the codebase itself, transient context that changes weekly, and instructions longer than 200 lines per file (adherence drops beyond that). See [Anti-Patterns](claude-anti-patterns.md) for common CLAUDE.md mistakes.

### 2. If you already have a CLAUDE.md, is a separate constitution.md or rules.md actually needed?

Yes, but for separation of concerns — not duplication. CLAUDE.md loads on every session start. Rules files (`.claude/rules/`) are glob-scoped, meaning they only load when the model touches matching files. This keeps context windows lean.

Use CLAUDE.md for universal project conventions. Use rules for file-type-specific constraints (e.g., "all `*.md` files must follow these formatting standards" or "never edit `presentation/` directly — delegate to the curator agent"). See the [Settings doc](claude-settings.md) for the full configuration hierarchy.

### 3. How often should you update your CLAUDE.md, and how do you know when it's become stale?

Update it reactively, not on a schedule. Boris's approach: when a PR introduces a preventable mistake, tag Claude on the PR with "add this to the CLAUDE.md." The team contributes to the shared CLAUDE.md multiple times per week.

Signs of staleness:
- The model keeps ignoring an instruction (the rule may conflict with newer model behavior).
- You have instructions for scaffolding the model no longer needs (each new model needs fewer guardrails).
- The file exceeds 200 lines.

When in doubt, delete the CLAUDE.md and rebuild from scratch. Add rules back one at a time only when the model goes off track. See the [Y Combinator interview](../videos/claude-boris-y-combinator-17-feb-26.md) where Boris explains this philosophy.

### 4. Why does Claude still ignore CLAUDE.md instructions — even when they say MUST in all caps?

Three likely causes:

1. **Instruction budget overflow.** Frontier LLMs follow roughly 150-200 instructions with good consistency ([Dex's MLOps talk](../videos/claude-dex-mlops-community-24-mar-26.md)). If your CLAUDE.md, system prompt, tools, and MCP servers collectively exceed that, adherence degrades. Audit total instruction count.
2. **Conflicting signals.** A rule file, a skill, or the codebase itself may contain patterns that contradict your CLAUDE.md. The model resolves conflicts unpredictably. Use the [Troubleshooting guide](claude-troubleshooting.md) to diagnose.
3. **Context window position.** Instructions at the edges of a large context window receive less attention. Keep CLAUDE.md concise and front-loaded with the most critical rules.

---

## Agents, Skills & Workflows

### 1. When should you use a command vs an agent vs a skill — and when is vanilla Claude Code just better?

The [Decision Tree](claude-decision-tree.md) provides a complete flowchart. The short version:

| Mechanism | Use when... |
|-----------|------------|
| **Vanilla prompt** | One-off task, no reuse needed |
| **Command** | Reusable workflow template injected into current context (e.g., `/commit`, `/feature-dev`) |
| **Skill** | Reusable knowledge that may be auto-discovered, preloaded into agents, or context-forked |
| **Subagent** | Task needs isolation (fresh context window), different model, restricted tools, or parallel execution |

When in doubt, start with a command. Promote to a skill when you need auto-discovery or agent preloading. Promote to a subagent when you need context isolation.

### 2. How often should you update your agents, commands, and workflows as models improve?

Every model release. Boris's team deletes 2,000 tokens from the system prompt when a new model ships if the model no longer needs the guidance. The principle from the [Every interview](../videos/claude-cat-every-29-oct-25.md): "We build features expecting to remove them in 3 months."

Practically: after each model upgrade, test your existing commands and skills. If the model handles a task without the scaffolding, remove it. Simpler prompts with fewer instructions yield better adherence.

### 3. Does giving your subagent a detailed persona improve quality? What does a "perfect persona/prompt" for a research/QA subagent look like?

Yes, but keep it focused. The [Subagents best practice](claude-subagents.md) documents the frontmatter fields. A good subagent definition includes:

- A clear `description` stating when to invoke (use "PROACTIVELY" for auto-invocation).
- A restricted `tools` allowlist — fewer tools means less decision-making overhead for the model.
- Preloaded `skills` that give the agent domain-specific knowledge.
- An appropriate `model` — use `haiku` for fast, simple tasks; `opus` for complex reasoning.

Avoid vague persona descriptions. Instead, provide concrete behavioral rules ("always check test output before approving," "never modify files outside `src/`").

### 4. Should you rely on Claude Code's built-in plan mode — or build your own planning command/agent?

Both, depending on task complexity. Boris uses built-in plan mode (shift-tab twice) for most tasks and expects it to be un-shipped eventually as models improve.

For team-standardized workflows, build your own. The [RPI-to-CRISPY evolution](../videos/claude-dex-mlops-community-24-mar-26.md) shows why: a custom planning pipeline with separated stages (questions, research, design, structure, plan, implement) gives better instruction adherence than a single monolithic prompt. See also the [Orchestration Workflow](../orchestration-workflow/orchestration-workflow.md) for a command-to-agent-to-skill pattern.

### 5. If you have a personal skill and a community skill that conflict, who wins?

The [Settings doc](claude-settings.md) documents the configuration hierarchy. Skills loaded later in the context override earlier ones when instructions conflict. Practically:

- Personal skills in your global `~/.claude/skills/` load at session start.
- Project skills in `.claude/skills/` load based on auto-discovery or explicit invocation.
- Agent-preloaded skills (via the `skills:` frontmatter field) load into the agent's context.

When skills disagree, the model sees both instructions and resolves the conflict itself — often unpredictably. Avoid overlap. If you use a community skill like `/simplify`, scope your personal skill to a different phase of the workflow.

### 6. Can we convert a codebase into specs, delete the code, and regenerate the exact same code from specs alone?

Not yet reliably. The current state of the art is demonstrated by projects like Beads (300,000+ lines, AI-generated) and OpenClaw (community-built skills). But as Dex notes in the [MLOps talk](../videos/claude-dex-mlops-community-24-mar-26.md): "These are OSS projects. They don't charge money." For production SaaS code in regulated industries, spec-to-code roundtripping still produces drift that requires human review.

The practical approach today: maintain specs as living documents alongside the code, use them to guide AI implementation, and verify the output. The gap closes with every model release.

---

## Specs & Documentation

### 1. Should every feature in your repo have a spec as a markdown file?

For features that involve multiple systems or non-obvious design decisions, yes. Specs serve dual purpose: they align humans on what to build and they provide compressed context for the model. The [CRISPY workflow](../videos/claude-dex-mlops-community-24-mar-26.md) places design discussions and structure outlines in markdown artifacts for exactly this reason.

For simple, self-evident features, the code and its tests are sufficient documentation.

### 2. How often do you need to update specs so they don't become obsolete?

Treat specs like tests: update them in the same PR as the feature change. The [changelog system](../changelog/) in this repo demonstrates the pattern — every official Claude Code release triggers an audit of documentation against the new version. Apply the same discipline to feature specs.

### 3. When implementing a new feature, how do you handle the ripple effect on specs for other features?

Use the subagent pattern. Spawn a research subagent to identify which existing specs are affected by the new feature, then update them as part of the implementation PR. The [Subagents best practice](claude-subagents.md) covers how to configure a research agent with read-only tools for this exact use case.

The key insight from Dex's talk: review the 200-line design discussion (where you catch ripple effects early) rather than the 1,000-line plan (where ripple effects are buried in implementation details).
