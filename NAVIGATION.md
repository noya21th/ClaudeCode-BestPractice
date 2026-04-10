# Learning Path & Navigation Guide

Three structured tracks through this repository, from first-time setup to advanced orchestration.

<table width="100%">
<tr>
<td><a href="./">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Learning Tracks

### Beginner: Foundations

Get Claude Code running and understand the core concepts.

| Step | Document | What You Learn |
|------|----------|----------------|
| 1 | [Tutorial](tutorial/day0/) | Guided first-day walkthrough of Claude Code |
| 2 | [Power-ups](best-practice/claude-power-ups.md) | Interactive lessons with animated demos for each feature |
| 3 | [CLAUDE.md / Memory Guide](best-practice/claude-memory-guide.md) | How to write effective CLAUDE.md files, rules, and memory hierarchy |
| 4 | [Memory](best-practice/claude-memory.md) | Persistent context via CLAUDE.md — writing and loading in monorepos |
| 5 | [Commands](best-practice/claude-commands.md) | Slash command patterns, built-in commands, and custom workflow templates |
| 6 | [FAQ](best-practice/claude-faq.md) | Answers to the most common questions about Claude Code configuration |

### Intermediate: Customization & Workflows

Build your own agents, skills, and automation pipelines.

| Step | Document | What You Learn |
|------|----------|----------------|
| 1 | [Skills](best-practice/claude-skills.md) | SKILL.md structure, auto-discovery, preloading, and context forking |
| 2 | [Subagents](best-practice/claude-subagents.md) | Agent frontmatter, tools, permissions, model selection, and memory |
| 3 | [Hooks](best-practice/claude-hooks.md) | Event-driven scripts that run outside the agentic loop |
| 4 | [Decision Tree](best-practice/claude-decision-tree.md) | When to use Agent vs Command vs Skill — flowchart and scenario mapping |
| 5 | [Orchestration Workflow](orchestration-workflow/orchestration-workflow.md) | Command-to-Agent-to-Skill architecture (weather system example) |
| 6 | [MCP Servers](best-practice/claude-mcp.md) | Model Context Protocol connections to external tools and APIs |
| 7 | [CLI Startup Flags](best-practice/claude-cli-startup-flags.md) | Command-line flags, subcommands, and environment variables |

### Advanced: Architecture & Operations

Scale Claude Code across teams, diagnose issues, and study real-world patterns.

| Step | Document | What You Learn |
|------|----------|----------------|
| 1 | [Settings](best-practice/claude-settings.md) | Full configuration hierarchy, permissions, sandboxing, fast mode |
| 2 | [Anti-Patterns](best-practice/claude-anti-patterns.md) | Common mistakes in architecture, configuration, skills, and operations |
| 3 | [Troubleshooting](best-practice/claude-troubleshooting.md) | Diagnostic guide for installation, performance, hooks, MCP, and memory |
| 4 | [Agent Teams](agent-teams/agent-teams-prompt.md) | Multi-agent topologies and team coordination patterns |
| 5 | [Cross-Model Workflow](development-workflows/cross-model-workflow/) | Using different models for different pipeline stages |
| 6 | [RPI Workflow](development-workflows/rpi/) | Research-Plan-Implement methodology and its evolution |
| 7 | [Reports](reports/) | Deep-dive analyses on specific topics (see list below) |

---

## File Directory Map

```
claude-code-best-practice-v2/
├── README.md                          # Main entry point — concepts table, tips, links
├── CLAUDE.md                          # Repository instructions for Claude Code
├── NAVIGATION.md                      # This file — learning tracks and directory map
│
├── best-practice/                     # Best practice guides (one topic per file)
│   ├── claude-anti-patterns.md        #   Common mistakes and how to avoid them
│   ├── claude-cli-startup-flags.md    #   CLI flags, subcommands, env vars
│   ├── claude-commands.md             #   Slash command patterns and reference
│   ├── claude-decision-tree.md        #   Agent vs Command vs Skill flowchart
│   ├── claude-faq.md                  #   Answers to Billion-Dollar Questions
│   ├── claude-hooks.md                #   Event-driven hook system
│   ├── claude-mcp.md                  #   Model Context Protocol servers
│   ├── claude-memory.md               #   CLAUDE.md writing and loading
│   ├── claude-memory-guide.md         #   Comprehensive memory management guide
│   ├── claude-power-ups.md            #   Interactive feature lessons
│   ├── claude-settings.md             #   Configuration hierarchy and permissions
│   ├── claude-skills.md               #   Skill definition and auto-discovery
│   ├── claude-subagents.md            #   Subagent architecture and frontmatter
│   └── claude-troubleshooting.md      #   Diagnostic and debugging guide
│
├── implementation/                    # Reference implementations
│   ├── claude-agent-teams-implementation.md
│   ├── claude-commands-implementation.md
│   ├── claude-hooks-implementation.md
│   ├── claude-mcp-implementation.md
│   ├── claude-scheduled-tasks-implementation.md
│   ├── claude-skills-implementation.md
│   └── claude-subagents-implementation.md
│
├── reports/                           # Deep-dive analyses
│   ├── claude-advanced-tool-use.md
│   ├── claude-agent-command-skill.md
│   ├── claude-agent-memory.md
│   ├── claude-agent-sdk-vs-cli-system-prompts.md
│   ├── claude-global-vs-project-settings.md
│   ├── claude-in-chrome-v-chrome-devtools-mcp.md
│   ├── claude-skills-for-larger-mono-repos.md
│   ├── claude-usage-and-rate-limits.md
│   └── llm-day-to-day-degradation.md
│
├── tips/                              # Boris Cherny and community tips
│   ├── claude-boris-10-tips-01-feb-26.md
│   ├── claude-boris-12-tips-12-feb-26.md
│   ├── claude-boris-13-tips-03-jan-26.md
│   ├── claude-boris-15-tips-30-mar-26.md
│   ├── claude-boris-2-tips-10-mar-26.md
│   ├── claude-boris-2-tips-25-mar-26.md
│   └── claude-thariq-tips-17-mar-26.md
│
├── videos/                            # Interview transcripts with TL;DR summaries
│   ├── claude-boris-lennys-podcast-19-feb-26.md
│   ├── claude-boris-pragmatic-engineer-04-mar-26.md
│   ├── claude-boris-ryan-peterman-15-dec-25.md
│   ├── claude-boris-y-combinator-17-feb-26.md
│   ├── claude-cat-every-29-oct-25.md
│   └── claude-dex-mlops-community-24-mar-26.md
│
├── tutorial/                          # Step-by-step tutorials
│   └── day0/                          #   Day 0 getting started guide
│
├── changelog/                         # Documentation drift tracking
│   ├── README.md                      #   Index of all changelog files
│   ├── best-practice/                 #   Per-topic changelogs and checklists
│   │   ├── claude-commands/
│   │   ├── claude-settings/
│   │   ├── claude-skills/
│   │   ├── claude-subagents/
│   │   └── concepts/
│   └── development-workflows/         #   Community workflow repo tracking
│
├── orchestration-workflow/            # Weather system demo (Command → Agent → Skill)
│   ├── orchestration-workflow.md
│   ├── orchestration-workflow.svg
│   └── output.md
│
├── development-workflows/             # External methodology references
│   ├── cross-model-workflow/
│   └── rpi/
│
├── agent-teams/                       # Multi-agent coordination patterns
│   ├── agent-teams-prompt.md
│   └── output/
│
├── presentation/                      # Slide deck (managed by presentation-curator agent)
│
├── i18n/                              # Translations (zh, ja, fr, ru, ar)
│
└── .claude/                           # Claude Code configuration
    ├── agents/                        #   Subagent definitions
    ├── commands/                      #   Slash command templates
    ├── hooks/                         #   Hook scripts and config
    ├── rules/                         #   Glob-scoped rules
    ├── skills/                        #   Skill definitions (SKILL.md)
    ├── settings.json                  #   Team-shared settings
    └── settings.local.json            #   Personal settings (git-ignored)
```

---

## Quick Links

### Concepts
- [Subagents](best-practice/claude-subagents.md) | [Commands](best-practice/claude-commands.md) | [Skills](best-practice/claude-skills.md) | [Hooks](best-practice/claude-hooks.md) | [MCP](best-practice/claude-mcp.md) | [Settings](best-practice/claude-settings.md) | [Memory](best-practice/claude-memory.md)

### Guides
- [Decision Tree](best-practice/claude-decision-tree.md) | [Anti-Patterns](best-practice/claude-anti-patterns.md) | [Troubleshooting](best-practice/claude-troubleshooting.md) | [Memory Guide](best-practice/claude-memory-guide.md) | [FAQ](best-practice/claude-faq.md) | [Power-ups](best-practice/claude-power-ups.md)

### Workflows
- [Orchestration](orchestration-workflow/orchestration-workflow.md) | [RPI](development-workflows/rpi/) | [Cross-Model](development-workflows/cross-model-workflow/) | [Agent Teams](agent-teams/agent-teams-prompt.md)

### Reports
- [Agent SDK vs CLI](reports/claude-agent-sdk-vs-cli-system-prompts.md) | [Browser Automation MCP](reports/claude-in-chrome-v-chrome-devtools-mcp.md) | [Global vs Project Settings](reports/claude-global-vs-project-settings.md) | [Skills in Monorepos](reports/claude-skills-for-larger-mono-repos.md) | [Agent Memory](reports/claude-agent-memory.md) | [Usage & Rate Limits](reports/claude-usage-and-rate-limits.md) | [Advanced Tool Use](reports/claude-advanced-tool-use.md) | [Agent vs Command vs Skill](reports/claude-agent-command-skill.md) | [LLM Degradation](reports/llm-day-to-day-degradation.md)

### Video Transcripts
- [Boris on Lenny's Podcast](videos/claude-boris-lennys-podcast-19-feb-26.md) | [Boris on Pragmatic Engineer](videos/claude-boris-pragmatic-engineer-04-mar-26.md) | [Boris on Ryan Peterman](videos/claude-boris-ryan-peterman-15-dec-25.md) | [Boris on Y Combinator](videos/claude-boris-y-combinator-17-feb-26.md) | [Cat & Boris on Every](videos/claude-cat-every-29-oct-25.md) | [Dex on MLOps](videos/claude-dex-mlops-community-24-mar-26.md)

### Tips Collections
- [13 Tips (Jan 3)](tips/claude-boris-13-tips-03-jan-26.md) | [10 Tips (Feb 1)](tips/claude-boris-10-tips-01-feb-26.md) | [12 Tips (Feb 12)](tips/claude-boris-12-tips-12-feb-26.md) | [2 Tips (Mar 10)](tips/claude-boris-2-tips-10-mar-26.md) | [Thariq Tips (Mar 17)](tips/claude-thariq-tips-17-mar-26.md) | [2 Tips (Mar 25)](tips/claude-boris-2-tips-25-mar-26.md) | [15 Tips (Mar 30)](tips/claude-boris-15-tips-30-mar-26.md)
