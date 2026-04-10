🌍 [English](../../../best-practice/claude-anti-patterns.md) | **中文** | [日本語](../../ja/best-practice/claude-anti-patterns.md) | [Français](../../fr/best-practice/claude-anti-patterns.md) | [Русский](../../ru/best-practice/claude-anti-patterns.md) | [العربية](../../ar/best-practice/claude-anti-patterns.md)

# 反模式指南

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code 配置中的常见错误及其规避方法 — 按类别整理。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 架构反模式

### 1. 巨型 CLAUDE.md

**问题**: 单个 CLAUDE.md 文件超过 500 行，安装说明、编码规范、API 文档、部署步骤和调试技巧全部混在一起。

**症状**: Claude 会忽略文件底部的指令。规范遵守不一致。会话启动时上下文膨胀。

**修复**: 拆分为专注的文件。根 CLAUDE.md 控制在 200 行以内，只放共享规范。使用组件级 CLAUDE.md 放具体内容。使用 `.claude/rules/*.md` 配合 glob 模式来提供文件类型相关的指导。

```
# 修改前（巨型）
CLAUDE.md              # 800 行涵盖所有内容

# 修改后（拆分）
CLAUDE.md              # 150 行 — 共享规范、项目概述
frontend/CLAUDE.md     # 80 行 — React 模式、组件结构
backend/CLAUDE.md      # 80 行 — API 规范、数据库模式
.claude/rules/ts.md    # TypeScript 专用规则 (Glob: **/*.ts)
.claude/rules/test.md  # 测试规范 (Glob: **/*.test.*)
```

### 2. 什么都用 Agent

**问题**: 对仅需将指令注入当前上下文的简单任务也使用 Agent。

**症状**: 不必要的上下文隔离。结果在主对话中不可见。Agent 启动开销导致执行缓慢。重复上下文造成 token 浪费。

**修复**: 使用 Skill 来封装可复用的领域知识和流程。使用 Command 来处理用户触发的 prompt 模板。只在真正需要隔离、记忆、工具限制或后台执行时才使用 Agent。

### 3. 缺少上下文隔离

**问题**: 在主对话中直接运行长时间的探索性任务（研究、代码审查、数据分析），没有隔离。

**症状**: 对话上下文快速填满。无关的中间结果污染后续回复。需要频繁 `/compact`。

**修复**: 对产生大量中间输出的任务使用 Agent。Agent 在全新的上下文中运行，只返回摘要。

### 4. 循环 Agent 调用

**问题**: Agent A 调用 Agent B，Agent B 又调用 Agent A，形成无限循环。

**症状**: 达到 `maxTurns` 限制。token 预算耗尽。没有有用的输出。

**修复**: 设计 DAG（有向无环图）工作流。使用 `tools:` 字段限制子 Agent 可以调用的 Agent。检查 Agent 定义中是否存在循环的 `Agent(...)` 引用。

```yaml
# 限制可调用的 Agent
---
name: planner
tools: Read, Glob, Grep, Agent(implementer)
# planner 只能调用 implementer，不能调用自身
---
```

### 5. 硬编码模型名称

**问题**: 在 Agent 定义中使用完整的模型 ID，如 `claude-sonnet-4-20250514`。

**症状**: 模型 ID 被弃用时 Agent 会出错。不同环境下行为不一致。

**修复**: 使用别名：`haiku`、`sonnet`、`opus` 或 `inherit`。它们总是解析为最新版本。

---

## 配置反模式

### 6. 权限过于宽松

**问题**: 对所有 Agent 设置 `"permissionMode": "bypassPermissions"`，并将 `--dangerously-skip-permissions` 作为默认选项。

**症状**: 未经审查的文件删除。意外的 `git push --force`。来自不可信 MCP 工具的安全风险。没有工具审批的审计记录。

**修复**: 对需要写文件但不应运行任意命令的 Agent 使用 `acceptEdits`。对仅用于研究的 Agent 使用 `plan`。`bypassPermissions` 仅限在有沙箱的 CI/CD 环境中使用。

| 模式 | 编辑 | Bash | 最适用于 |
|------|------|------|----------|
| `default` | 询问 | 询问 | 交互式开发 |
| `acceptEdits` | 自动 | 询问 | 文件写入 Agent |
| `plan` | 拒绝 | 拒绝 | 研究和规划 Agent |
| `bypassPermissions` | 自动 | 自动 | 仅限有沙箱的 CI/CD |

### 7. 忽视设置层级

**问题**: 在错误的文件中定义设置，然后疑惑为什么被覆盖或忽略。

**症状**: Hook 不触发。权限不符合预期。团队成员行为不一致。

**修复**: 理解优先级顺序（从高到低）：

1. **Managed**（`managed-settings.json`）— 组织强制执行
2. **CLI 参数** — 单次会话覆盖
3. **Local**（`.claude/settings.local.json`）— 个人项目设置
4. **Shared**（`.claude/settings.json`）— 团队共享项目设置
5. **Global**（`~/.claude/settings.json`）— 个人默认值

### 8. 所有 Hook 都是同步的

**问题**: 所有 Hook 都同步运行，脚本执行期间阻塞 Claude 的响应。

**症状**: 响应缓慢。每次工具调用后都有明显停顿。用户感觉 Claude 很卡。

**修复**: 对执行副作用（日志、通知、遥测）的 Hook 设置 `"async": true`。仅在需要拦截或修改行为（安全检查、权限决策）时保持 Hook 同步。

### 9. 没有 settings.local.json

**问题**: 个人偏好（声音通知、自定义路径、调试日志）被提交到 `.claude/settings.json`，影响整个团队。

**症状**: 队友听到你的通知声音。调试 Hook 在生产环境运行。个人 MCP Server 对所有人可见。

**修复**: 使用 `.claude/settings.local.json` 存放个人覆盖。确保它在 `.gitignore` 中。只共享整个团队需要的 Hook 和设置。

---

## Skill 反模式

### 10. 仅供 Agent 使用的 Skill 设置了 user-invocable: true

**问题**: 一个仅用于 Agent 预加载（通过 `skills:` 字段）的 Skill 出现在用户的 `/` 斜杠命令菜单中。

**症状**: 用户直接调用该 Skill，得到令人困惑的结果，因为它本来是作为 Agent 的背景知识而设计的。

**修复**: 对仅用于 Agent 预加载的 Skill 设置 `user-invocable: false`。

```yaml
---
name: internal-api-conventions
user-invocable: false
description: "REST API conventions for the internal platform"
---
```

### 11. 缺少 Description

**问题**: Skill 的 frontmatter 中没有 `description` 字段。

**症状**: Claude 无法自动发现该 Skill。只有当用户显式输入 `/slash-command` 时才会触发。该 Skill 对自动调用逻辑不可见。

**修复**: 始终包含一个说明**何时**使用该 Skill 的 `description`。使用 Claude 自然会遇到的关键词："Use when creating React components" 或 "Triggers on Python test files"。

### 12. 巨型 Skill 文件

**问题**: 单个 SKILL.md 文件超过 1000 行，涵盖每一个边界情况、示例和参考资料。

**症状**: Skill 加载时上下文膨胀。Claude 可能忽略末尾的指令。预加载到 Agent 时加载缓慢。

**修复**: SKILL.md 聚焦于核心流程（控制在 200 行以内）。将参考资料移至 Skill 目录内的 `references/` 文件中。使用 `@path` 引用补充内容，Claude 按需加载。

### 13. 频繁使用的 Skill 设置了 disable-model-invocation

**问题**: 对 Claude 应该自动发现并调用的 Skill 设置了 `disable-model-invocation: true`。

**症状**: Claude 永远不会自动调用该 Skill。即使上下文明确需要，用户也必须始终手动输入 `/slash-command`。

**修复**: 仅对纯用户触发的 Skill（危险操作、部署等）使用 `disable-model-invocation: true`。对于 Claude 应自行发现的 Skill，不设置或设为 `false`。

---

## 运维反模式

### 14. 从不压缩

**问题**: 长时间运行会话却不压缩，让上下文填充到 100%。

**症状**: Claude 开始遗忘早期的指令。回复准确性下降。会话最终因上下文溢出而报错。

**修复**: 在上下文使用率约 50% 时手动运行 `/compact`。使用 `/context` 可视化当前使用量。对于长任务，将工作拆分为子任务，每个子任务在 50% 上下文容量内完成。

### 15. 没有验证循环

**问题**: 直接接受 Claude 的输出而不验证其是否有效 — 不跑测试、不做类型检查、不做视觉检查。

**症状**: 生成的代码有语法错误。API 集成使用了错误的端点。配置文件有拼写错误。

**修复**: 始终验证输出。代码生成后运行测试。使用浏览器自动化（Chrome、Playwright）进行视觉验证。在 Command 和 Agent 定义中包含验证步骤。

### 16. 忽视速率限制

**问题**: 规划大型多 Agent 任务时不考虑 token 预算和速率限制。

**症状**: Agent 在任务中途命中速率限制。工作丢失或不完整。账单超出预期。

**修复**: 使用 `/usage` 检查当前速率限制状态。规划任务以适应预算。对高频低复杂度的子任务使用 `model: haiku`。将大任务拆分为较小的会话。

---

## 参考资料

- [Claude Code Best Practices — Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Permissions — Docs](https://code.claude.com/docs/en/permissions)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
