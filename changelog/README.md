# Changelog Tracking

Tracks drift between this repository's best-practice documentation and official Claude Code releases. Each changelog file records what changed in the official docs, what actions were taken in the repo, and whether each action was completed, invalidated, or deferred.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Changelog Files

| File | Scope | Last Verified Version |
|------|-------|-----------------------|
| [best-practice/claude-commands/changelog.md](best-practice/claude-commands/changelog.md) | Commands frontmatter, slash command patterns, built-in command reference | v2.1.74 |
| [best-practice/claude-skills/changelog.md](best-practice/claude-skills/changelog.md) | Skills frontmatter, SKILL.md structure, auto-discovery, context forking | v2.1.74 |
| [best-practice/claude-subagents/changelog.md](best-practice/claude-subagents/changelog.md) | Subagent frontmatter, hooks, memory, invocation methods, repository agents | v2.1.63 |
| [best-practice/claude-settings/changelog.md](best-practice/claude-settings/changelog.md) | Settings hierarchy, permissions, model config, sandboxing, fast mode | v2.1.69 |
| [best-practice/concepts/changelog.md](best-practice/concepts/changelog.md) | README CONCEPTS table — feature list, descriptions, badge links | v2.1.63 |
| [development-workflows/changelog.md](development-workflows/changelog.md) | Community development workflow repos — agent/skill/command counts, repo links | v2.1.74 |

## Verification Checklists

Some changelog categories include a companion verification checklist used during audits:

- [best-practice/concepts/verification-checklist.md](best-practice/concepts/verification-checklist.md)
- [best-practice/claude-subagents/verification-checklist.md](best-practice/claude-subagents/verification-checklist.md)
- [best-practice/claude-settings/verification-checklist.md](best-practice/claude-settings/verification-checklist.md)

## How It Works

1. When a new Claude Code version ships, an audit agent compares official docs against the repo content.
2. Findings are logged as rows in the relevant changelog with priority, type, and recommended action.
3. Each action is resolved with a status: `COMPLETE`, `INVALID`, or `ON HOLD`.
4. The cycle repeats on every significant release.
