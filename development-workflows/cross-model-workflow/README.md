# Cross-Model Workflow

<table width="100%">
<tr>
<td><a href="../../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

Use **Claude Code (Opus 4.6)** and **Codex CLI (GPT-5.4)** together in a four-step loop: Plan, QA Review, Implement, Verify. Each model covers the other's blind spots -- Claude excels at planning and implementation, Codex catches gaps during review and verification.

See the full workflow with diagrams: **[cross-model-workflow.md](cross-model-workflow.md)**

---

## Quick Start

1. **Open Terminal 1** -- launch Claude Code in plan mode.
   ```bash
   claude --plan
   ```
2. Describe your feature. Claude interviews you and produces `plans/{feature-name}.md`.
3. **Open Terminal 2** -- launch Codex CLI.
   ```bash
   codex
   ```
4. Ask Codex to review the plan against the codebase. It inserts "Codex Finding" sections without rewriting original phases.
5. **Back in Terminal 1** -- start a new Claude Code session and implement phase-by-phase, running test gates after each phase.
6. **Back in Terminal 2** -- start a new Codex session and verify the implementation against the plan.

---

## When to Use This Workflow

| Scenario | Why it helps |
|----------|-------------|
| Large features spanning multiple files | Cross-model review catches assumptions a single model misses |
| Unfamiliar codebase | Codex QA surfaces structural concerns before you write code |
| High-stakes changes (auth, payments, data migrations) | Two-model verification reduces the chance of subtle bugs |
| Team onboarding | The plan artifact doubles as a design doc for reviewers |

---

## Contents

- [cross-model-workflow.md](cross-model-workflow.md) -- Full workflow diagram, production screenshot, and step-by-step details
- [assets/](assets/) -- Workflow images

*Last Updated: 2026-04-09*
