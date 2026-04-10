# Cross-Model (Claude Code + Codex) Workflow

based on [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) and [codex-cli-best-practice](https://github.com/shanraisshan/codex-cli-best-practice)

## Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│              CROSS-MODEL CLAUDE CODE + CODEX WORKFLOW                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  STEP 1: PLAN                                          Claude Code      │
│  ─────────────                                         Opus 4.6         │
│  Open Claude Code in plan mode (Terminal 1).           Plan Mode        │
│  Claude interviews you via AskUserQuestion.                             │
│  Produces a phased plan with test gates.                                │
│                                                                         │
│  Output: plans/{feature-name}.md                                        │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 2: QA REVIEW                                     Codex CLI        │
│  ──────────────────                                    GPT-5.4          │
│  Open Codex CLI in another terminal (Terminal 2).                       │
│  Codex reviews plan against the actual codebase.                        │
│  Inserts intermediate phases ("Phase 2.5")                              │
│  with "Codex Finding" headings.                                         │
│  Adds to the plan — never rewrites original phases.                     │
│                                                                         │
│  Output: plans/{feature-name}.md (updated)                              │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 3: IMPLEMENT                                     Claude Code      │
│  ──────────────────                                    Opus 4.6         │
│  Start a new Claude Code session (Terminal 1).                          │
│  You implement phase-by-phase                                           │
│  with test gates at each phase.                                         │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 4: VERIFY                                        Codex CLI        │
│  ────────────────                                      GPT-5.4          │
│  Start a new Codex CLI session (Terminal 2).                            │
│  Codex verifies the implementation                                      │
│  against the plan.                                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## How cross-model workflow actually looks in production

![Cross-Model Workflow](assets/cross-model-workflow.png)

*Last Updated: 2026-03-06*

---

## Failure Recovery

When one model fails mid-workflow, use these fallback strategies.

### Plan Phase Failure (Claude Code)

If Claude Code crashes or loses context during planning:

1. Check `plans/{feature-name}.md` -- partial plans are saved incrementally.
2. Start a new Claude Code session and point it at the partial plan: "Continue this plan from where it left off."
3. If the plan file is empty or corrupt, re-describe the feature. Claude's plan-mode interview is fast enough to restart cheaply.

### QA Review Failure (Codex CLI)

If Codex CLI fails during review:

1. The original plan in `plans/{feature-name}.md` is untouched -- Codex only appends.
2. Restart Codex and ask it to review the plan again. It is stateless, so a fresh session works identically.
3. **Fallback**: skip QA and proceed to implementation. Run verification (Step 4) more thoroughly to compensate.

### Implementation Failure (Claude Code)

If Claude Code fails during implementation:

1. Check `git log` -- each phase commits separately, so completed phases are preserved.
2. Start a new Claude Code session and say: "Continue implementing from Phase N of `plans/{feature-name}.md`."
3. Run the test gate for the last completed phase to confirm your starting point is clean.

### Verification Failure (Codex CLI)

If Codex CLI fails during verification:

1. Re-run verification in a new Codex session -- point it at the plan and the current codebase.
2. **Fallback**: use Claude Code itself to verify by asking it to diff the implementation against the plan. This loses the cross-model benefit but still catches obvious gaps.

---

## Model Selection Criteria

Choose the right model for each phase based on these strengths.

| Phase | Recommended Model | Why |
|-------|-------------------|-----|
| **Plan** | Claude Code (Opus 4.6) | Strong at structured reasoning, interview-style elicitation, and producing phased plans with test gates |
| **QA Review** | Codex CLI (GPT-5.4) | Fresh perspective on the plan -- different training data catches different assumptions |
| **Implement** | Claude Code (Opus 4.6) | Superior code generation, tool use, and multi-file editing within a single session |
| **Verify** | Codex CLI (GPT-5.4) | Independent verification avoids confirmation bias from the implementing model |

### When to Swap Roles

- **Small, well-defined tasks**: skip the cross-model loop entirely -- use Claude Code alone.
- **Codex is better at the domain**: if your codebase is heavily Python/data-science, Codex may produce stronger implementation. Swap Steps 1-3 accordingly.
- **Claude is better at review**: for complex architectural concerns, use Claude Code for both planning and QA, and reserve Codex for implementation verification only.

---

## Cost Optimization

Token budgets vary by phase. Allocate accordingly.

| Phase | Typical Token Usage | Cost Weight | Optimization Tips |
|-------|--------------------:|:-----------:|-------------------|
| Plan | 5k-15k input, 3k-8k output | Low | Use plan mode (`--plan`) to keep Claude focused. Avoid open-ended exploration. |
| QA Review | 10k-30k input, 2k-5k output | Medium | Pass only the plan file and relevant source files -- do not let Codex read the entire repo. |
| Implement | 20k-80k input, 10k-40k output | High | Implement one phase per session. Compact (`/compact`) at 50% context. |
| Verify | 10k-25k input, 2k-5k output | Medium | Scope verification to changed files only -- use `git diff` output as context. |

### Budget Allocation Rule of Thumb

For a typical feature, expect this token split:

- **Plan**: ~10% of total budget
- **QA Review**: ~15% of total budget
- **Implement**: ~60% of total budget
- **Verify**: ~15% of total budget

### Reducing Costs

1. **Skip QA for low-risk changes.** Bug fixes and small refactors do not need cross-model review.
2. **Batch verification.** Implement multiple phases, then verify them together in one Codex session.
3. **Use `git diff` as input.** Instead of passing full files to the verifier, pass only the diff and the plan.
4. **Compact aggressively.** Run `/compact` between phases during implementation to keep context lean.

---

## Complete Example: Adding a Webhook System

This walkthrough shows the full cross-model workflow for a real feature.

### Step 1: Plan (Claude Code, Terminal 1)

Launch Claude Code in plan mode:

```bash
claude --plan
```

Conversation:

```
You:    "Add a webhook system that lets users register URLs and receive
         POST notifications when orders change status."

Claude: "What event types should trigger webhooks?"
You:    "order.created, order.updated, order.cancelled"

Claude: "Should retries be automatic? How many attempts?"
You:    "Yes, 3 retries with exponential backoff."

Claude: "Here's the plan..."
```

Output: `plans/webhook-system.md`

```markdown
# Webhook System

## Phase 1: Data Model & Registration API
- Create webhooks table (id, url, events, secret, created_at)
- POST /webhooks - register a webhook
- GET /webhooks - list webhooks
- DELETE /webhooks/:id - remove a webhook
- Test gate: API tests pass for all CRUD operations

## Phase 2: Event Dispatch Engine
- Create event queue (database-backed)
- Dispatch worker that POSTs to registered URLs
- HMAC-SHA256 signature in X-Webhook-Signature header
- Test gate: Mock server receives signed payloads

## Phase 3: Retry & Monitoring
- Exponential backoff: 1s, 4s, 16s (3 attempts)
- Mark webhook as failed after 3 consecutive failures
- GET /webhooks/:id/deliveries - delivery log
- Test gate: Retry logic works with simulated failures
```

### Step 2: QA Review (Codex CLI, Terminal 2)

```
You:    "Review plans/webhook-system.md against the codebase.
         Check for conflicts, missing edge cases, and integration
         gaps. Add findings as intermediate phases -- do not
         rewrite existing phases."
```

Codex adds:

```markdown
## Phase 1.5: Codex Finding — URL Validation
- Validate webhook URLs (HTTPS only, no localhost in production)
- Reject private IP ranges (10.x, 172.16-31.x, 192.168.x)
- Existing: src/middleware/auth.js already validates Bearer tokens —
  reuse the same pattern for webhook secret generation

## Phase 2.5: Codex Finding — Payload Size
- Current order serializer in src/models/order.js includes
  related line_items — payloads could exceed 1MB
- Add a webhook-specific slim serializer
- Cap payload at 256KB with a link to the full resource
```

### Step 3: Implement (Claude Code, Terminal 1)

Start a new session:

```bash
claude
```

```
You:    "Implement plans/webhook-system.md phase by phase.
         Run the test gate after each phase before proceeding."
```

Claude implements Phase 1, runs tests, confirms PASS. Proceeds to Phase 1.5 (Codex finding), then Phase 2, and so on. Each phase is committed separately.

### Step 4: Verify (Codex CLI, Terminal 2)

Start a new session:

```
You:    "Verify the webhook-system implementation against
         plans/webhook-system.md. Check that every phase
         and every Codex Finding is addressed. Review the
         git diff for issues."
```

Codex confirms:
- All 5 phases implemented
- URL validation rejects private IPs (Phase 1.5) -- verified
- Slim serializer added (Phase 2.5) -- verified
- One suggestion: "Add a `Content-Type: application/json` header to webhook POSTs -- currently missing"

You fix the header, commit, and the feature is complete.

*Last Updated: 2026-04-09*
