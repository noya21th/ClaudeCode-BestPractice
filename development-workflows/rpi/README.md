# RPI Workflow (Research - Plan - Implement)

<table width="100%">
<tr>
<td><a href="../../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

A systematic development workflow with validation gates at each phase. You describe a feature, a team of specialized agents researches feasibility, plans the implementation, and builds it -- with GO/NO-GO checkpoints preventing wasted effort.

See the full workflow with diagrams: **[rpi-workflow.md](rpi-workflow.md)**

---

## Quick Start

1. **Install**: copy the `.claude` folder (containing `agents/` and `commands/rpi/`) into your repository root.
2. **Create the plans directory**: `mkdir -p rpi/plans`
3. **Describe your feature** to Claude Code. It generates `rpi/plans/{feature-slug}.md`.
4. **Create the feature folder** and move the plan:
   ```bash
   mkdir -p rpi/{feature-slug}
   cp rpi/plans/{feature-slug}.md rpi/{feature-slug}/REQUEST.md
   ```
5. **Research**: `/rpi:research rpi/{feature-slug}/REQUEST.md`
6. **Plan**: `/rpi:plan {feature-slug}`
7. **Implement**: `/rpi:implement {feature-slug}`

---

## Agent Roster

Eight specialized agents collaborate across the three phases.

| Agent | Role | Phases |
|-------|------|--------|
| **requirement-parser** | Extracts structured requirements from the feature request | Research |
| **product-manager** | Defines user stories, acceptance criteria, and GO/NO-GO verdict | Research, Plan |
| **senior-software-engineer** | Evaluates technical feasibility, writes architecture specs, implements code | Research, Plan, Implement |
| **technical-cto-advisor** | Reviews strategic alignment and technical risk | Research |
| **ux-designer** | Designs UI flows and interaction patterns | Plan |
| **documentation-analyst-writer** | Produces research reports, plan documents, and implementation records | Research, Plan |
| **code-reviewer** | Reviews implementation for quality, correctness, and standards compliance | Implement |
| **constitutional-validator** | Validates agent outputs against project constraints and rules | All |

---

## Phase Overview

### Phase 1: Research (`/rpi:research`)

Agents analyze the feature request against the existing codebase. Output: `research/RESEARCH.md` with a **GO** or **NO-GO** verdict.

- Parses requirements into structured format
- Evaluates technical feasibility
- Checks strategic alignment
- Identifies risks and dependencies

### Phase 2: Plan (`/rpi:plan`)

Agents produce a complete implementation roadmap. Output:

| File | Contents |
|------|----------|
| `plan/pm.md` | User stories and acceptance criteria |
| `plan/ux.md` | UI/UX flows and wireframes |
| `plan/eng.md` | Technical architecture and specifications |
| `plan/PLAN.md` | Phased task breakdown with test gates |

### Phase 3: Implement (`/rpi:implement`)

Agents build the feature phase-by-phase, running test gates after each phase. Output: `implement/IMPLEMENT.md` with phase-by-phase results.

---

## When to Use This Workflow

| Scenario | Why RPI helps |
|----------|--------------|
| New features with unclear scope | Research phase forces clarity before any code is written |
| Cross-cutting changes (auth, database schema, API contracts) | Multi-agent planning catches integration issues early |
| Features requiring product and engineering alignment | PM, UX, and engineering agents produce aligned artifacts |
| Onboarding new team members | The artifact trail (REQUEST, RESEARCH, PLAN, IMPLEMENT) serves as living documentation |
| High-risk changes where rollback is expensive | GO/NO-GO gate prevents wasted effort on non-viable features |

---

## Feature Folder Structure

```
rpi/{feature-slug}/
├── REQUEST.md              # Initial feature description
├── research/
│   └── RESEARCH.md         # GO/NO-GO analysis
├── plan/
│   ├── PLAN.md             # Implementation roadmap
│   ├── pm.md               # Product requirements
│   ├── ux.md               # UX design
│   └── eng.md              # Technical specification
└── implement/
    └── IMPLEMENT.md        # Implementation record
```

---

## Contents

- [rpi-workflow.md](rpi-workflow.md) -- Full workflow diagram and example walkthrough
- [.claude/agents/](.claude/agents/) -- Agent definitions (8 agents)
- [.claude/commands/rpi/](.claude/commands/rpi/) -- Slash commands (research, plan, implement)

*Last Updated: 2026-04-09*
