🌍 [English](../../../best-practice/claude-decision-tree.md) | **中文** | [日本語](../../ja/best-practice/claude-decision-tree.md) | [Français](../../fr/best-practice/claude-decision-tree.md) | [Русский](../../ru/best-practice/claude-decision-tree.md) | [العربية](../../ar/best-practice/claude-decision-tree.md)

# Agent vs Skill vs Command — 决策树

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

何时使用 Agent、Skill 或 Command — 一套包含真实场景的决策框架。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 快速对比

| 维度 | Agent | Command | Skill |
|------|-------|---------|-------|
| **位置** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **上下文** | 全新隔离上下文 | 注入当前上下文 | 注入当前上下文（或通过 `context: fork` 分叉） |
| **调用方式** | `Agent(...)` 工具、通过 description 自动发现 | 用户输入 `/slash-command` | `/slash-command`、自动发现、或 Agent 预加载 |
| **记忆** | 持久化（`user`/`project`/`local` 作用域） | 无 | 无 |
| **工具** | 通过 `tools:` 字段自定义允许列表 | 继承会话工具 | 继承会话工具（或通过 `allowed-tools:` 限制） |
| **模型** | 可配置（`haiku`/`sonnet`/`opus`/`inherit`） | 可配置 | 可配置 |
| **权限** | 可配置（`acceptEdits`/`plan`/`bypassPermissions`） | 继承会话权限 | 继承会话权限 |
| **自动发现** | 是（通过 `description`，使用 `PROACTIVELY`） | 否（仅用户触发） | 是（通过 `description`） |
| **可预加载** | 否 | 否 | 是（通过 Agent 的 `skills:` 字段） |
| **后台运行** | 是（`background: true`） | 否 | 否 |
| **Worktree 隔离** | 是（`isolation: worktree`） | 否 | 否 |
| **最适用于** | 需要隔离的复杂多步骤任务 | 用户触发的工作流和编排 | 可复用的领域知识和流程 |

---

## 决策流程图

```
起点: "我需要给 Claude Code 添加一项能力"
│
├─ 是否需要独立的隔离上下文？
│  ├─ 是 ──→ 是否需要跨会话持久记忆？
│  │          ├─ 是 ──→ AGENT (配合 memory: user/project/local)
│  │          └─ 否 ──→ AGENT (或使用 context: fork 的 Skill)
│  │
│  └─ 否 ──→ 是否只能由用户显式触发？
│             ├─ 是 ──→ COMMAND (用户输入 /slash-command)
│             └─ 否 ──→ 是否应该让 Claude 自动发现并调用？
│                        ├─ 是 ──→ SKILL (带 description 用于发现)
│                        └─ 否 ──→ 是否为可复用的领域知识？
│                                   ├─ 是 ──→ SKILL (设置 user-invocable: false 仅预加载)
│                                   └─ 否 ──→ COMMAND (简单的 prompt 模板)
```

### 快速决策规则

- **需要上下文隔离？** → Agent
- **需要持久记忆？** → Agent
- **需要后台执行？** → Agent
- **需要 worktree 隔离？** → Agent
- **需要自定义工具限制？** → Agent
- **用户触发的工作流？** → Command
- **可复用的流程或领域知识？** → Skill
- **Claude 可自动发现？** → Skill（或在 description 中使用 `PROACTIVELY` 的 Agent）
- **可预加载到 Agent？** → Skill

---

## 真实场景（12 个）

### 1. 代码审查

**推荐**: Agent

**原因**: 代码审查需要隔离上下文，以免审查结果污染主对话。Agent 可以拥有自己的工具（只读）、模型（使用 haiku 提速），以及持久记忆来学习团队特定的审查模式。

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. 天气查询 (Command → Agent → Skill 链)

**推荐**: Command 编排 Agent，Agent 预加载 Skill

**原因**: 用户显式触发（Command），获取逻辑在隔离环境中运行（Agent），数据获取指令是可复用的领域知识（Skill 预加载到 Agent 中）。

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. 文件脚手架

**推荐**: Skill

**原因**: 脚手架模板是可复用的领域知识。当用户要求创建新组件、路由或模块时，Claude 可以自动发现该 Skill。无需隔离 — 文件直接写入当前项目。

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. CI/CD 编排

**推荐**: Command

**原因**: CI/CD 工作流始终由用户触发（`/deploy`、`/release`）。它们按固定顺序执行，且需要访问当前会话上下文（分支、最近的改动）。

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. 浏览器测试

**推荐**: Agent

**原因**: 浏览器测试是长时间运行的任务，可能需要后台执行，且不应阻塞主对话。Agent 可以拥有特定的 MCP Server（Playwright、Chrome DevTools），并在 worktree 中运行以实现隔离。

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. 时间显示

**推荐**: Skill

**原因**: 轻量级，无需隔离，可自动发现。当有人问"现在几点"时，Claude 会调用该 Skill。

### 7. 多步骤部署

**推荐**: Command 编排多个 Agent

**原因**: Command 是面向用户的入口（`/full-deploy`）。它为每个阶段生成 Agent — 构建 Agent、测试 Agent、部署 Agent — 每个都在自己的上下文中运行并使用相应的工具。

### 8. 代码检查 / 格式化

**推荐**: Skill（自动调用）

**原因**: 代码检查规则是可复用的领域知识。当 Claude 编辑匹配文件时，Skill 通过 `paths:` 自动激活。无需用户交互。

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. 研究助手

**推荐**: Agent

**原因**: 研究任务是长时间运行的，受益于记忆（记住过去的研究），需要网络工具（`WebFetch`、`WebSearch`），并且应该在隔离上下文中运行，避免使主对话变得杂乱。

### 10. 数据迁移

**推荐**: 带 worktree 隔离的 Agent

**原因**: 数据库迁移修改的是关键文件，应在隔离环境中测试。在 git worktree 中运行意味着主分支在迁移验证完成前不受影响。

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. 文档生成

**推荐**: Skill

**原因**: 文档生成是可复用的流程。当用户要求"写文档"或"生成 API 参考"时，Claude 可以自动发现。它在当前上下文中运行，直接将文件写入项目。

### 12. 项目初始化

**推荐**: Command（配合 Setup Hook）

**原因**: 项目初始化始终由用户触发，且只运行一次。将 `/setup` Command 与 `Setup` Hook 结合使用，可在首次 clone 时自动运行。

---

## 组合模式

### Command → Agent → Skill

经典的编排模式。Command 是用户入口，Agent 提供隔离和工具控制，Skill 提供领域知识。

```
用户输入 /command
  └─ Command prompt 在当前上下文中运行
       └─ Agent(...) 在隔离上下文中生成
            └─ 预加载的 Skill 提供指令
                 └─ Agent 完成，结果返回到 Command 上下文
                      └─ Skill 工具被调用以生成最终输出
```

### Agent 预加载 Skill

Agent 在启动时通过 `skills:` 字段加载 Skill 内容。Skill 内容被注入到 Agent 的系统提示中，赋予其领域专业知识，无需用户进行任何调用。

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### Skill 的上下文分叉

需要隔离但不需要完整 Agent 定义的 Skill。设置 `context: fork` 可以在子 Agent 上下文中运行 Skill，同时保留简单的 Skill 文件格式。

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## 机制选择中的反模式

| 反模式 | 问题 | 更好的做法 |
|--------|------|-----------|
| 简单任务用 Agent | 只需注入指令时，隔离上下文的开销过大 | 使用 Skill |
| Command 用于自动发现 | Command 无法被 Claude 自动调用 | 使用带 `description` 的 Skill |
| Skill 用于长时间运行的任务 | Skill 在主上下文中运行，会阻塞对话 | 使用 `background: true` 的 Agent |
| Agent 用于共享约定 | Agent 上下文是隔离的 — 约定不会传递到主会话 | 使用 CLAUDE.md 或 Rules |
| Command 用于可预加载的知识 | Command 无法预加载到 Agent 中 | 使用 `user-invocable: false` 的 Skill |
| Skill 没有 `description` | Claude 无法自动发现它 | 始终添加有意义的 description |

---

## 参考资料

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
