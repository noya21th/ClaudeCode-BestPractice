🌍 [English](i18n/en/README.md) | **中文** | [日本語](i18n/ja/README.md) | [Français](i18n/fr/README.md) | [Русский](i18n/ru/README.md) | [العربية](i18n/ar/README.md)

# ClaudeCode-BestPractice
Claude Code 最佳实践增强版 — 从「能用」到「用好」的完整指南

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-v2.1.96-white?style=flat&labelColor=555) ![Enhancement](https://img.shields.io/badge/增强版-新增_90%2B_文件-2ea44f?style=flat&labelColor=555) ![i18n](https://img.shields.io/badge/多语-中%2F英%2F日%2F法%2F俄%2F阿-0075ca?style=flat&labelColor=555)

### 本库做了什么

原社区仓库 [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)（⭐33k+）是优秀的概念参考，但存在**策略层缺失、实战示例不足、无多语支持**等问题。本增强版针对性补全：

| 优化方向 | 具体内容 |
|----------|---------|
| **填补策略空白** | 新增 [决策树](best-practice/claude-decision-tree.md)（Agent/Skill/Command 何时用哪个）、[反模式](best-practice/claude-anti-patterns.md)（16 个常见错误）、[故障排查](best-practice/claude-troubleshooting.md)（按症状定位问题） |
| **完善核心文档** | 新增 [Hooks 专题](best-practice/claude-hooks.md)（27 事件全解析）、[Memory 统一指南](best-practice/claude-memory-guide.md)（4 种记忆体系对比）、[FAQ](best-practice/claude-faq.md)（13 个高频问题） |
| **补全实战示例** | 新增 [Hooks 实现](implementation/claude-hooks-implementation.md)、[MCP 实现](implementation/claude-mcp-implementation.md) 完整可运行配置 |
| **增强现有文档** | Skills（56→219 行：9 种类型分类 + 创建流程）、Subagents（57→220 行：多 Agent 模式 + 错误处理） |
| **6 语国际化** | 中/英/日/法/俄/阿拉伯语 共 69 个翻译文件，阿拉伯语含 RTL 适配 |
| **导航与体验** | [学习路径](NAVIGATION.md)（初/中/高三轨）、12 篇文档交叉引用、6 个视频 TL;DR 摘要、[Changelog 索引](changelog/README.md) |

> 定位：原库是「知识库」，本库是「实操手册」。

[![Best Practice](!/tags/best-practice.svg)](best-practice/) [![Implemented](!/tags/implemented.svg)](implementation/) [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) [![Claude](!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](!/tags/boris-cherny.svg)](#-tips-and-tricks) [![Community](!/tags/community.svg)](#-subscribe) ![Click on these badges below to see the actual sources](!/tags/click-badges.svg)<br>
<img src="!/tags/a.svg" height="14"> = Agents · <img src="!/tags/c.svg" height="14"> = Commands · <img src="!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="!/claude-jumping.svg" alt="Claude Code mascot jumping" width="120" height="100">
</p>


## 🧠 核心概念

| 功能 | 位置 | 说明 |
|------|------|------|
| <img src="!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-subagents.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-subagents-implementation.md) 在隔离上下文中运行的自主执行者 — 拥有自定义工具、权限、模型、记忆和持久身份 |
| <img src="!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/slash-commands) | `.claude/commands/<name>.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-commands.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-commands-implementation.md) 注入到当前上下文的知识 — 用户触发的简单 prompt 模板，用于工作流编排 |
| <img src="!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-skills.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-skills-implementation.md) 注入到当前上下文的知识 — 可配置、可预加载、可自动发现，支持上下文分叉和渐进式展开 · [官方 Skills](https://github.com/anthropics/skills/tree/main/skills) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) 用户自定义的处理器（脚本、HTTP、prompt、agent），在 agentic loop 之外的特定事件上运行 · [指南](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-mcp.md) [![Implemented](!/tags/implemented.svg)](.mcp.json) Model Context Protocol 连接外部工具、数据库和 API |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | 可分发包 | Skills、subagents、hooks、MCP servers 和 LSP servers 的捆绑包 · [市场](https://code.claude.com/docs/en/discover-plugins) · [创建市场](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-settings.md) [![Implemented](!/tags/implemented.svg)](.claude/settings.json) 层级化配置系统 · [权限](https://code.claude.com/docs/en/permissions) · [模型配置](https://code.claude.com/docs/en/model-config) · [输出样式](https://code.claude.com/docs/en/output-styles) · [沙盒](https://code.claude.com/docs/en/sandboxing) · [快捷键](https://code.claude.com/docs/en/keybindings) · [快速模式](https://code.claude.com/docs/en/fast-mode) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](!/tags/implemented.svg)](.claude/settings.json) 可自定义的状态栏，显示上下文用量、模型、费用和会话信息 |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-memory.md) [![Implemented](!/tags/implemented.svg)](CLAUDE.md) 通过 CLAUDE.md 文件和 `@path` 导入实现持久化上下文 · [自动记忆](https://code.claude.com/docs/en/memory) · [规则](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | 自动（基于 git） | 自动追踪文件编辑，支持回退（`Esc Esc` 或 `/rewind`）和定向摘要 |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-cli-startup-flags.md) 启动 Claude Code 时的命令行参数、子命令和环境变量 · [交互模式](https://code.claude.com/docs/en/interactive-mode) |
| **AI Terms** | | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) Agentic Engineering · Context Engineering · Vibe Coding |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | 官方最佳实践 · [Prompt Engineering](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [扩展 Claude Code](https://code.claude.com/docs/en/features-overview) |

### 🔥 热门

| 功能 | 位置 | 说明 |
|------|------|------|
| [**Power-ups**](best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-power-ups.md) 带动画演示的交互式课程，教你 Claude Code 功能 (v2.1.90) |
| [**Ultraplan**](https://code.claude.com/docs/en/ultraplan) ![beta](!/tags/beta.svg) | `/ultraplan` | 在云端起草计划，通过浏览器审查、行内评论和灵活执行 — 可远程执行或传送回终端 |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](!/tags/beta.svg) | `claude.ai/code` | 在云端基础设施上运行任务 — 长时间运行的任务、PR 自动修复、无需本地配置的并行会话 · [定时任务](https://code.claude.com/docs/en/web-scheduled-tasks) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](!/tags/beta.svg) | `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) 无闪烁的替代屏幕渲染，支持鼠标、稳定内存和应用内滚动 — 选择性参与的研究预览 |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](!/tags/beta.svg) | `computer-use` MCP server | 让 Claude 控制你的屏幕 — 在 macOS 上打开应用、点击、输入和截屏 · [桌面端](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) ![beta](!/tags/beta.svg) | `claude --enable-auto-mode` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) 后台安全分类器替代手动权限提示 — Claude 自行判断安全性，同时阻止 prompt 注入和危险操作升级 · 使用 `claude --enable-auto-mode`（或 `--permission-mode auto`）启动，或在会话中通过 `Shift+Tab` 切换 · [博客](https://claude.com/blog/auto-mode) |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](!/tags/beta.svg) | `--channels`, 基于 plugin | 从 Telegram、Discord 或 webhooks 推送事件到运行中的会话 — Claude 在你离开时自动响应 · [参考](https://code.claude.com/docs/en/channels-reference) |
| [**Slack**](https://code.claude.com/docs/en/slack) | Slack 中 `@Claude` | 在团队聊天中 @Claude 提交编码任务 — 路由到 Claude Code 网页会话进行 bug 修复、代码审查和并行任务执行 |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](!/tags/beta.svg) | GitHub App（托管） | [![Best Practice](!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) 多 Agent PR 分析，捕获 bug、安全漏洞和回归问题 · [博客](https://claude.com/blog/code-review) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | 在 CI/CD 流水线中自动化 PR 审查、Issue 分类和代码生成 · [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) ![beta](!/tags/beta.svg) | `--chrome`, 扩展 | [![Best Practice](!/tags/best-practice.svg)](reports/claude-in-chrome-v-chrome-devtools-mcp.md) 通过 Claude in Chrome 实现浏览器自动化 — 测试 Web 应用、调试控制台、自动化表单、从页面提取数据 |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, cron 工具 | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](!/tags/implemented.svg)](implementation/claude-scheduled-tasks-implementation.md) `/loop` 在本地按循环调度运行 prompt（最长 3 天）· [`/schedule`](https://code.claude.com/docs/en/web-scheduled-tasks) 在 Anthropic 云端基础设施上运行 prompt — 即使你的电脑关机也能执行 · [公告](https://x.com/noahzweben/status/2036129220959805859) |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](!/tags/beta.svg) | `/voice` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) 按键说话的语音输入，支持 20 种语言和可自定义的激活键 |
| [**Simplify & Batch**](https://code.claude.com/docs/en/skills#bundled-skills) | `/simplify`, `/batch` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) 内置 skills：代码质量和批量操作 — simplify 重构以提升复用性和效率，batch 跨文件批量运行命令 |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](!/tags/beta.svg) | 内置（环境变量） | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](!/tags/implemented.svg)](implementation/claude-agent-teams-implementation.md) 多个 agent 在同一代码库上并行工作，支持共享任务协调 |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) 从任何设备继续本地会话 — 手机、平板或浏览器 · [Headless 模式](https://code.claude.com/docs/en/headless) |
| [**Git Worktrees**](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) | 内置 | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) 隔离的 git 分支用于并行开发 — 每个 agent 拥有独立的工作副本 |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26) 用于长时间运行任务的自主开发循环 — 持续迭代直到完成 |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="orchestration-workflow/orchestration-workflow.md"><img src="!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

参见 [orchestration-workflow](orchestration-workflow/orchestration-workflow.md) 了解 <img src="!/tags/c.svg" height="14"> **Command** → <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** 模式的实现细节。


<p align="center">
  <img src="orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

<p align="center">
  <img src="orchestration-workflow/orchestration-workflow.gif" alt="Orchestration Workflow Demo" width="600">
</p>

![How to Use](!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ⚙️ 开发工作流

所有主要工作流都汇聚于相同的架构模式：**调研 → 计划 → 执行 → 审查 → 交付**

| 名称 | ★ | 独特之处 | 计划 | <img src="!/tags/a.svg" height="14"> | <img src="!/tags/c.svg" height="14"> | <img src="!/tags/s.svg" height="14"> |
|------|---|----------|------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 146k | ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![AgentShield](https://img.shields.io/badge/AgentShield-ddf4ff) ![multi-lang rules](https://img.shields.io/badge/multi--lang_rules-ddf4ff) | <img src="!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 47 | 82 | 182 |
| [Superpowers](https://github.com/obra/superpowers) | 141k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![whole-plan review](https://img.shields.io/badge/whole--plan_review-ddf4ff) | <img src="!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 86k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![22+ tools](https://img.shields.io/badge/22%2B_tools-ddf4ff) | <img src="!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 67k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 49k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![XML plans](https://img.shields.io/badge/XML_plans-ddf4ff) | <img src="!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 24 | 68 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 44k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![22+ platforms](https://img.shields.io/badge/22%2B_platforms-ddf4ff) | <img src="!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 38k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) ![artifact DAG](https://img.shields.io/badge/artifact_DAG-ddf4ff) | <img src="!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 26k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 14k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![Plugin Marketplace](https://img.shields.io/badge/Plugin_Marketplace-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 51 | 4 | 44 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 10k | ![RPI](https://img.shields.io/badge/RPI-ddf4ff) ![context engineering](https://img.shields.io/badge/context_engineering-ddf4ff) ![300k+ LOC](https://img.shields.io/badge/300k%2B_LOC-ddf4ff) | <img src="!/tags/c.svg" height="14"> [create_plan](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) | 6 | 27 | 0 |

### 其他
- [跨模型（Claude Code + Codex）工作流](development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](!/tags/implemented.svg)](development-workflows/cross-model-workflow/cross-model-workflow.md)
- [RPI](development-workflows/rpi/rpi-workflow.md) [![Implemented](!/tags/implemented.svg)](development-workflows/rpi/rpi-workflow.md)
- [Ralph Wiggum Loop](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26)
- [Andrej Karpathy（OpenAI 创始成员）的工作流](https://x.com/karpathy/status/2015883857489522876)
- [Peter Steinberger（OpenClaw 创建者）的工作流](https://youtu.be/8lF7HmQ_RgY?t=2582)
- Boris Cherny（Claude Code 创建者）的工作流 — [13 条建议](tips/claude-boris-13-tips-03-jan-26.md) · [10 条建议](tips/claude-boris-10-tips-01-feb-26.md) · [12 条建议](tips/claude-boris-12-tips-12-feb-26.md) · [2 条建议](tips/claude-boris-2-tips-25-mar-26.md) · [15 条建议](tips/claude-boris-15-tips-30-mar-26.md) [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny)

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 💡 技巧与窍门 (69)

🚫👶 = 不要过度干预

[Prompt 技巧](#tips-prompting) · [规划](#tips-planning) · [CLAUDE.md](#tips-claudemd) · [Agents](#tips-agents) · [Commands](#tips-commands) · [Skills](#tips-skills) · [Hooks](#tips-hooks) · [工作流](#tips-workflows) · [进阶](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [调试](#tips-debugging) · [实用工具](#tips-utilities) · [日常](#tips-daily)

![Community](!/tags/community.svg)

<a id="tips-prompting"></a>■ **Prompt 技巧 (3)**

| 技巧 | 来源 |
|------|------|
| 挑战 Claude — "审查这些更改，在我通过你的测试之前不要创建 PR" 或 "向我证明这能工作"，让 Claude 对比 main 和你的分支 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| 在一次平庸的修复之后 — "根据你现在了解的一切，丢掉这些重新实现一个优雅的方案" 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude 自己能修大多数 bug — 粘贴 bug，说 "fix"，不要微管理 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **规划/Specs (6)**

| 技巧 | 来源 |
|------|------|
| 始终从 [plan mode](https://code.claude.com/docs/en/common-workflows) 开始 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| 从最简 spec 或 prompt 开始，让 Claude 用 [AskUserQuestion](https://code.claude.com/docs/en/cli-reference) 工具采访你，然后新开一个会话来执行 spec | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| 始终制定分阶段的把关计划，每个阶段包含多种测试（单元测试、自动化测试、集成测试） | |
| 启动第二个 Claude 作为 Staff Engineer 审查你的计划，或使用[跨模型](development-workflows/cross-model-workflow/cross-model-workflow.md)进行审查 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| 在交接工作前写好详细的 spec 并减少歧义 — 你越具体，输出越好 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| 原型 > PRD — 构建 20-30 个版本而不是写 spec，构建成本很低所以多试几次 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-claudemd"></a>■ **CLAUDE.md (7)**

| 技巧 | 来源 |
|------|------|
| [CLAUDE.md](https://code.claude.com/docs/en/memory) 每个文件应控制在 [200 行](https://code.claude.com/docs/en/memory#write-effective-instructions)以内。[HumanLayer 中 60 行](https://www.humanlayer.dev/blog/writing-a-good-claude-md)（[仍不能 100% 保证](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)） | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| 用 [\<important if="..."\> 标签](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md)包裹领域特定的 CLAUDE.md 规则，防止文件变长后 Claude 忽略它们 | [![Dex](!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| 在 monorepo 中使用[多个 CLAUDE.md](best-practice/claude-memory.md) — 祖先 + 后代加载机制 | |
| 使用 [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) 拆分大型指令 | |
| [memory.md](https://code.claude.com/docs/en/memory)、constitution.md 不能保证任何事 | |
| 任何开发者都应该能启动 Claude，说 "run the tests" 然后首次就成功 — 如果不行，你的 CLAUDE.md 缺少必要的安装/构建/测试命令 | [![Dex](!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| 保持代码库整洁并完成迁移 — 部分迁移的框架会让模型困惑，可能选错模式 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| 使用 [settings.json](best-practice/claude-settings.md) 实现 harness 强制行为（署名、权限、模型）— 不要在 CLAUDE.md 里写 "NEVER add Co-Authored-By"，当 `attribution.commit: ""` 是确定性行为时 | [![davila7](!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="!/tags/a.svg" height="14"> **Agents (4)**

| 技巧 | 来源 |
|------|------|
| 创建功能特定的 [sub-agents](https://code.claude.com/docs/en/sub-agents)（附加上下文）配合 [skills](https://code.claude.com/docs/en/skills)（渐进式展开），而非通用的 qa、backend engineer | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| 说 "use subagents" 来投入更多算力 — 将任务分流以保持主上下文的干净和专注 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| 使用 [agent teams 配合 tmux](https://code.claude.com/docs/en/agent-teams) 和 [git worktrees](https://x.com/bcherny/status/2025007393290272904) 进行并行开发 | |
| 使用 [test time compute](https://code.claude.com/docs/en/sub-agents) — 独立的上下文窗口让结果更好；一个 agent 可能引入 bug 而另一个（同模型）能发现它 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="!/tags/c.svg" height="14"> **Commands (3)**

| 技巧 | 来源 |
|------|------|
| 使用 [commands](https://code.claude.com/docs/en/slash-commands) 而不是 [sub-agents](https://code.claude.com/docs/en/sub-agents) 来构建你的工作流 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 为每天多次执行的 "内循环" 工作流使用 [slash commands](https://code.claude.com/docs/en/slash-commands) — 省去重复的 prompt，commands 保存在 `.claude/commands/` 中并纳入 git | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 如果你一天做某件事超过一次，就把它变成 [skill](https://code.claude.com/docs/en/skills) 或 [command](https://code.claude.com/docs/en/slash-commands) — 构建 `/techdebt`、context-dump 或 analytics commands | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="!/tags/s.svg" height="14"> **Skills (9)**

| 技巧 | 来源 |
|------|------|
| 使用 [context: fork](https://code.claude.com/docs/en/skills) 在隔离的 subagent 中运行 skill — 主上下文只看到最终结果，看不到中间工具调用。agent 字段让你设置 subagent 类型 | [![Lydia](!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| 在 monorepo 中使用[子文件夹中的 skills](reports/claude-skills-for-larger-mono-repos.md) | |
| skills 是文件夹而非文件 — 使用 references/、scripts/、examples/ 子目录实现[渐进式展开](https://code.claude.com/docs/en/skills) | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 在每个 skill 中构建 Gotchas 部分 — 最高价值的内容，随时间积累 Claude 的失败点 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| skill 的 description 字段是触发器，不是摘要 — 为模型而写（"什么时候该触发？"） | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 不要在 skills 中陈述显而易见的事 — 聚焦于推动 Claude 偏离默认行为的内容 🚫👶 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 不要在 skills 中限死 Claude — 给出目标和约束，而非规定性的逐步指令 🚫👶 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 在 skills 中包含脚本和库，让 Claude 组合而非重建样板代码 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 在 SKILL.md 中嵌入 `` !`command` `` 来将动态 shell 输出注入 prompt — Claude 在调用时运行它，模型只看到结果 | [![Lydia](!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **Hooks (5)**

| 技巧 | 来源 |
|------|------|
| 在 skills 中使用[按需 hooks](https://code.claude.com/docs/en/skills) — /careful 阻止破坏性命令，/freeze 阻止在目录外编辑 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 使用 PreToolUse hook [衡量 skill 使用率](https://code.claude.com/docs/en/skills)，发现热门或触发不足的 skills | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 使用 [PostToolUse hook](https://code.claude.com/docs/en/hooks) 自动格式化代码 — Claude 生成格式良好的代码，hook 处理最后 10% 避免 CI 失败 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| 通过 hook 将[权限请求](https://code.claude.com/docs/en/hooks)路由到 Opus — 让它扫描攻击并自动批准安全请求 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| 使用 [Stop hook](https://code.claude.com/docs/en/hooks) 提示 Claude 继续工作或在回合结束时验证其工作成果 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **工作流 (7)**

| 技巧 | 来源 |
|------|------|
| 避免 agent 蠢化区，在最多 50% 时手动执行 [/compact](https://code.claude.com/docs/en/interactive-mode)。使用 [/clear](https://code.claude.com/docs/en/cli-reference) 在切换到新任务时重置上下文 | |
| 对于较小的任务，原生 Claude Code 比任何工作流都好 | |
| 使用 [/model](https://code.claude.com/docs/en/model-config) 选择模型和推理能力，[/context](https://code.claude.com/docs/en/interactive-mode) 查看上下文用量，[/usage](https://code.claude.com/docs/en/costs) 检查计划限制，[/extra-usage](https://code.claude.com/docs/en/interactive-mode) 配置溢出计费，[/config](https://code.claude.com/docs/en/settings) 配置设置 — 规划用 Opus，编码用 Sonnet 以获得两者最佳效果 | [![Cat](!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| 始终在 [/config](https://code.claude.com/docs/en/settings) 中启用 [thinking mode](https://code.claude.com/docs/en/model-config) true（查看推理过程）和 [Output Style](https://code.claude.com/docs/en/output-styles) Explanatory（查看详细输出及 ★ Insight 方框） | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| 在 prompt 中使用 ultrathink 关键词触发[高强度推理](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) | |
| [/rename](https://code.claude.com/docs/en/cli-reference) 重要会话（如 [TODO - refactor task]），之后用 [/resume](https://code.claude.com/docs/en/cli-reference) 恢复 — 同时运行多个 Claude 时给每个实例打标签 | [![Cat](!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |
| Claude 跑偏时使用 [Esc Esc 或 /rewind](https://code.claude.com/docs/en/checkpointing) 撤销，而不是在同一上下文中尝试修复 | |

<a id="tips-workflows-advanced"></a>■ **进阶工作流 (6)**

| 技巧 | 来源 |
|------|------|
| 大量使用 ASCII 图表来理解你的架构 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| 使用 [/loop](https://code.claude.com/docs/en/scheduled-tasks) 进行本地循环监控（最长 3 天）· 使用 [/schedule](https://code.claude.com/docs/en/web-scheduled-tasks) 进行云端定时任务，即使你的电脑关机也能运行 | |
| 使用 [Ralph Wiggum plugin](https://github.com/shanraisshan/novel-llm-26) 来执行长时间运行的自主任务 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| [/permissions](https://code.claude.com/docs/en/permissions) 使用通配符语法（Bash(npm run *)、Edit(/docs/**)）代替 dangerously-skip-permissions | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing) 通过文件和网络隔离减少权限提示 — 内部测试减少 84% | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| 投入精力打造[产品验证](https://code.claude.com/docs/en/skills) skills（注册流程驱动器、结账验证器）— 值得花一周去完善 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**

| 技巧 | 来源 |
|------|------|
| 保持 PR 小而专注 — [中位数 118 行](tips/claude-boris-2-tips-25-mar-26.md)（一天 141 个 PR，修改 45K 行），每个 PR 一个功能，方便审查和回退 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| 始终 [squash merge](tips/claude-boris-2-tips-25-mar-26.md) PR — 干净的线性历史，每个功能一个 commit，便于 git revert 和 git bisect | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| 频繁 commit — 至少每小时一次，任务完成立即 commit | ![Shayan](!/tags/community-shayan.svg) |
| 在同事的 PR 上 @[@claude](https://github.com/apps/claude) 自动生成 lint 规则以处理反复出现的审查反馈 — 将自己从代码审查中自动化出来 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| 使用 [/code-review](https://code.claude.com/docs/en/code-review) 进行多 agent PR 分析 — 在合并前捕获 bug、安全漏洞和回归问题 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **调试 (7)**

| 技巧 | 来源 |
|------|------|
| 养成截图并分享给 Claude 的习惯，在遇到任何问题时 | ![Shayan](!/tags/community-shayan.svg) |
| 使用 MCP（[Claude in Chrome](https://code.claude.com/docs/en/chrome)、[Playwright](https://github.com/microsoft/playwright-mcp)、[Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp)）让 Claude 自己查看 Chrome 控制台日志 | |
| 始终要求 Claude 将需要查看日志的终端作为后台任务运行，以便更好地调试 | |
| [/doctor](https://code.claude.com/docs/en/cli-reference) 诊断安装、认证和配置问题 | |
| 压缩时的错误可以通过 [/model](https://code.claude.com/docs/en/model-config) 选择 1M token 模型后再运行 [/compact](https://code.claude.com/docs/en/interactive-mode) 来解决 | |
| 使用[跨模型](development-workflows/cross-model-workflow/cross-model-workflow.md)进行 QA — 例如 [Codex](https://github.com/shanraisshan/codex-cli-best-practice) 用于计划和实现审查 | |
| agentic search（glob + grep）优于 RAG — Claude Code 尝试过向量数据库但放弃了，因为代码会漂移且权限复杂 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **实用工具 (5)**

| 技巧 | 来源 |
|------|------|
| 使用 [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) 终端而非 IDE（[VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)） | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| [/voice](https://code.claude.com/docs/en/voice-dictation) 或 [Wispr Flow](https://wisprflow.ai) 进行语音 prompting（10x 生产力） | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) 用于 Claude 反馈 | ![Shayan](!/tags/community-shayan.svg) |
| [status line](https://github.com/shanraisshan/claude-code-status-line) 用于上下文感知和快速压缩 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](!/tags/community-shayan.svg) |
| 探索 [settings.json](best-practice/claude-settings.md) 功能如 [Plans Directory](best-practice/claude-settings.md#plans-directory)、[Spinner Verbs](best-practice/claude-settings.md#display--ux) 获得个性化体验 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **日常 (2)**

| 技巧 | 来源 |
|------|------|
| 每天[更新](https://code.claude.com/docs/en/setup) Claude Code | ![Shayan](!/tags/community-shayan.svg) |
| 每天从阅读 [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) 开始 | ![Shayan](!/tags/community-shayan.svg) |

![Boris Cherny + Team](!/tags/claude.svg)

| 文章 / 推文 | 来源 |
|-------------|------|
| [15 个 Claude Code 中隐藏和未被充分利用的功能 (Boris) \| 2026/03/30](tips/claude-boris-15-tips-30-mar-26.md) | [推文](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merge 与 PR 大小分布 (Boris) \| 2026/03/25](tips/claude-boris-2-tips-25-mar-26.md) | [推文](https://x.com/bcherny/status/2038552880018538749) |
| [构建 Claude Code 的经验：我们如何使用 Skills (Thariq) \| 2026/03/17](tips/claude-thariq-tips-17-mar-26.md) | [文章](https://x.com/trq212/status/2033949937936085378) |
| [Code Review 与 Test Time Compute (Boris) \| 2026/03/10](tips/claude-boris-2-tips-10-mar-26.md) | [推文](https://x.com/bcherny/status/2031089411820228645) |
| /loop — 安排最长 3 天的循环任务 (Boris) \| 2026/03/07 | [推文](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Thariq) \| 2026/02/28 | [推文](https://x.com/trq212/status/2027543858289250472) |
| Seeing like an Agent - 构建 Claude Code 的经验 (Thariq) \| 2026/02/28 | [文章](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees - Boris 使用的 5 种方式 \| 2026/02/21 | [推文](https://x.com/bcherny/status/2025007393290272904) |
| 构建 Claude Code 的经验：Prompt Caching 就是一切 (Thariq) \| 2026/02/20 | [文章](https://x.com/trq212/status/2024574133011673516) |
| [12 种人们自定义 Claude 的方式 (Boris) \| 2026/02/12](tips/claude-boris-12-tips-12-feb-26.md) | [推文](https://x.com/bcherny/status/2021699851499798911) |
| [团队使用 Claude Code 的 10 条建议 (Boris) \| 2026/02/01](tips/claude-boris-10-tips-01-feb-26.md) | [推文](https://x.com/bcherny/status/2017742741636321619) |
| [我如何使用 Claude Code — 来自我极简配置的 13 条建议 (Boris) \| 2026/01/03](tips/claude-boris-13-tips-03-jan-26.md) | [推文](https://x.com/bcherny/status/2007179832300581177) |
| 让 Claude 用 AskUserQuestion 工具采访你 (Thariq) \| 2025/12/28 | [推文](https://x.com/trq212/status/2005315275026260309) |
| 始终使用 plan mode，给 Claude 验证方式，使用 /code-review (Boris) \| 2025/12/27 | [推文](https://x.com/bcherny/status/2004711722926616680) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🎬 视频 / 播客

| 视频 / 播客 | 来源 | YouTube |
|-------------|------|---------|
| 我们在 Research-Plan-Implement 上犯了哪些错 (Dex) \| 2026/03/24 \| MLOps Community | [![Dex](!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| 与 Boris Cherny 一起构建 Claude Code (Boris) \| 2026/03/04 \| The Pragmatic Engineer | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| Claude Code 负责人：编码被解决之后会发生什么 (Boris) \| 2026/02/19 \| Lenny's Podcast | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| 与创建者 Boris Cherny 深入 Claude Code (Boris) \| 2026/02/17 \| Y Combinator | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Boris Cherny（Claude Code 创建者）谈什么推动了他的职业发展 (Boris) \| 2025/12/15 \| Ryan Peterman | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| 来自构建它的工程师们的 Claude Code 秘密 (Cat) \| 2025/10/29 \| Every | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🔔 订阅

| 平台 | 名称 | 标签 |
|------|------|------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Jesse Kriss](https://x.com/obra) ([Superpowers](https://github.com/obra/superpowers)), [Affaan Mustafa](https://x.com/affaanmustafa) ([ECC](https://github.com/affaan-m/everything-claude-code)), [Garry Tan](https://x.com/garrytan) ([gstack](https://github.com/garrytan/gstack)), [Dex Horthy](https://x.com/dexhorthy) ([HumanLayer](https://github.com/humanlayer/humanlayer)), [Kieran Klaassen](https://x.com/kieranklaassen) ([Compound Eng](https://github.com/EveryInc/compound-engineering-plugin)), [Tabish Gilani](https://x.com/0xTab) ([OpenSpec](https://github.com/Fission-AI/OpenSpec)), [Brian McAdams](https://x.com/BMadCode) ([BMAD](https://github.com/bmad-code-org/BMAD-METHOD)), [Lex Christopherson](https://x.com/official_taches) ([GSD](https://github.com/gsd-build/get-shit-done)), [Dani Avila](https://x.com/dani_avila7) ([CC Templates](https://github.com/davila7/claude-code-templates)), [Dan Shipper](https://x.com/danshipper) ([Every](https://every.to/)), [Andrej Karpathy](https://x.com/karpathy) ([AutoResearch](https://x.com/karpathy/status/2015883857489522876)), [Peter Steinberger](https://x.com/steipete) ([OpenClaw](https://x.com/openclaw)), [Sigrid Jin](https://x.com/realsigridjin) ([claw-code](https://github.com/ultraworkers/claw-code)), [Yeachan Heo](https://x.com/bellman_ych) ([oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)) | ![Community](!/tags/community.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Lenny's Podcast](https://www.youtube.com/@LennysPodcast), [Y Combinator](https://www.youtube.com/@ycombinator), [The Pragmatic Engineer](https://www.youtube.com/@pragmaticengineer), [Ryan Peterman](https://www.youtube.com/@ryanlpeterman), [Every](https://www.youtube.com/@every_media), [MLOps Community](https://www.youtube.com/@MLOps) | ![Community](!/tags/community.svg) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ☠️ 初创公司 / 商业

| Claude | 被替代的 |
|--------|----------|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|YC AI wrapper 初创公司 ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](!/tags/billion-dollar-questions.svg)

*欢迎在 Issues 中讨论这些问题*

**记忆与指令 (4)**

1. CLAUDE.md 里到底应该放什么 — 什么应该留在外面？
2. 如果已经有了 CLAUDE.md，还需要单独的 constitution.md 或 rules.md 吗？
3. 应该多久更新一次 CLAUDE.md，怎么知道它是否已经过时？
4. 为什么 Claude 仍然忽视 CLAUDE.md 指令 — 即使写了全大写的 MUST？（[reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)）

**Agents、Skills 与工作流 (6)**

1. 什么时候该用 command vs agent vs skill — 什么时候原生 Claude Code 就够了？
2. 随着模型提升，应该多久更新一次你的 agents、commands 和工作流？
3. 给 subagent 一个详细的人设是否能提升质量？"完美的" 研究/QA subagent 人设是什么样的？
4. 应该依赖 Claude Code 内置的 plan mode — 还是构建你自己的 planning command/agent 来强制执行团队工作流？
5. 如果你有一个个人 skill（如 /implement 带有你的编码风格），如何整合社区 skills（如 /simplify）而不冲突 — 冲突时谁的优先级更高？
6. 我们到了吗？能否将现有代码库转换为 specs，删掉代码，然后让 AI 仅从 specs 重新生成完全相同的代码？

**Specs 与文档 (3)**

1. 仓库中的每个功能是否都应有一个 spec markdown 文件？
2. 需要多久更新一次 specs 以防新功能实现后 specs 变得过时？
3. 实现新功能时，如何处理对其他功能 specs 的连锁影响？

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 报告

<p align="center">
  <a href="reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
</p>

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

![How to Use](!/tags/how-to-use.svg)

```
1. 像读课程一样阅读本仓库，先了解 commands、agents、skills 和 hooks 是什么，再尝试使用。
2. 克隆本仓库并体验示例，试试 /weather-orchestrator，听 hook 音效，运行 agent teams，看看实际效果。
3. 回到你自己的项目，让 Claude 建议应该从本仓库添加哪些最佳实践，把本仓库作为参考让它知道有哪些可能。
```

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="!/thumbnail/video-1.png" alt="Watch on YouTube" width="300"></a>

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 致谢

概念框架与社区内容来自 [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)，感谢原作者 Shayan Rais 的开源贡献。
