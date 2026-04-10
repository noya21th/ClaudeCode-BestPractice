🌍 [English](../../../best-practice/claude-hooks.md) | **中文** | [日本語](../../ja/best-practice/claude-hooks.md) | [Français](../../fr/best-practice/claude-hooks.md) | [Русский](../../ru/best-practice/claude-hooks.md) | [العربية](../../ar/best-practice/claude-hooks.md)

# Hooks 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code hooks — 在特定生命周期事件上于 agentic loop 之外运行的事件驱动处理器。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 概述

Hooks 是用户定义的处理器，在 agentic loop **之外**执行 — 它们不是 Claude 推理或工具调用链的一部分。它们在特定生命周期事件上触发，可以运行脚本、注入 prompts、派生 agents 或调用 HTTP 端点。使用 hooks 来添加可观测性、执行策略、自动化副作用和把关决策，而不会污染对话上下文。

---

## Hook 事件 (27)

### 会话生命周期

| 事件 | 触发时机 |
|------|----------|
| `SessionStart` | 新的交互会话开始 |
| `SessionEnd` | 会话即将关闭 |
| `Setup` | Claude Code 首次在项目中运行（或执行 `/init` 之后） |

### 工具生命周期

| 事件 | 触发时机 |
|------|----------|
| `PreToolUse` | 工具调用执行前。可以阻止、允许或修改调用 |
| `PostToolUse` | 工具调用成功完成后 |
| `PostToolUseFailure` | 工具调用失败并出错后 |
| `PermissionRequest` | 当 Claude 请求用户未预批准的工具权限时 |
| `PermissionDenied` | 当工具权限请求被用户或策略拒绝时 |

### Agent 生命周期

| 事件 | 触发时机 |
|------|----------|
| `SubagentStart` | subagent（通过 `Agent(...)` 工具）开始执行 |
| `SubagentStop` | subagent 完成执行 |
| `Stop` | **主** agent 完成其响应回合 |
| `StopFailure` | 主 agent 的响应回合出错失败 |

### 任务 / Teams

| 事件 | 触发时机 |
|------|----------|
| `TeammateIdle` | Agent Teams 会话中的一个队友变为空闲可用 |
| `TaskCreated` | 创建了一个后台任务 |
| `TaskCompleted` | 后台任务完成 |

### 上下文

| 事件 | 触发时机 |
|------|----------|
| `PreCompact` | 上下文压缩运行前 |
| `PostCompact` | 上下文压缩完成后 |
| `UserPromptSubmit` | 用户提交 prompt（在 Claude 处理之前） |
| `Notification` | Claude 发送通知（如任务完成、需要权限） |

### 配置 / 环境

| 事件 | 触发时机 |
|------|----------|
| `ConfigChange` | settings 文件被修改 |
| `CwdChanged` | 工作目录变更（如通过 `/add-dir`） |
| `FileChanged` | 被监视的文件在磁盘上被修改 |
| `InstructionsLoaded` | CLAUDE.md 或 rules 文件被加载到上下文中 |

### Worktree

| 事件 | 触发时机 |
|------|----------|
| `WorktreeCreate` | 为隔离 agent 创建了 git worktree |
| `WorktreeRemove` | git worktree 被清理 |

### MCP / Elicitation

| 事件 | 触发时机 |
|------|----------|
| `Elicitation` | Claude 需要通过 MCP elicitation 从用户获取结构化输入 |
| `ElicitationResult` | 用户响应了 MCP elicitation |

---

## Hook 类型 (4)

| 类型 | 语法 | 说明 |
|------|------|------|
| `command` | `{"type": "command", "command": "python3 script.py"}` | 运行 shell 命令。通过环境变量接收事件数据。Stdout 被注入回上下文（用于 `PreToolUse`/`Stop`）。非零退出码阻止操作（用于 `PreToolUse`） |
| `prompt` | `{"type": "prompt", "prompt": "审查此处的安全问题"}` | 在 hook 点注入 prompt 到 Claude 的上下文中。用于添加指令或约束 |
| `agent` | `{"type": "agent", "agent": "security-reviewer"}` | 派生 subagent 处理事件。agent 在自己的上下文中运行，可使用工具 |
| `http` | `{"type": "http", "url": "https://example.com/webhook"}` | 通过 POST 将事件数据发送到 HTTP 端点。响应体被注入回上下文 |

---

## 配置层级

Hooks 可以在多个层级定义。对于相同事件，高层级覆盖低层级：

| 层级 | 位置 | 范围 |
|------|------|------|
| **托管** | `managed-settings.json`（MDM / Registry） | 组织强制执行，不可覆盖 |
| **本地设置** | `.claude/settings.local.json` | 个人项目覆盖（git-ignored） |
| **共享设置** | `.claude/settings.json` | 团队共享的项目 hooks |
| **全局设置** | `~/.claude/settings.json` | 跨所有项目的个人默认值 |
| **Agent frontmatter** | `.claude/agents/<name>.md` `hooks:` 字段 | 范围限定于特定 subagent |
| **Skill frontmatter** | `.claude/skills/<name>/SKILL.md` `hooks:` 字段 | 范围限定于特定 skill |

要禁用所有 hooks，在 `.claude/settings.local.json` 中设置 `"disableAllHooks": true`。

---

## Matcher 语法

Matchers 过滤哪些工具调用触发 hook。它们适用于 `PreToolUse`、`PostToolUse` 和 `PostToolUseFailure` 事件。

| Matcher | 匹配 |
|---------|------|
| `"Bash"` | 所有 Bash 工具调用 |
| `"Read"` | 所有 Read 工具调用 |
| `"Write"` | 所有 Write 工具调用 |
| `"Edit"` | 所有 Edit 工具调用 |
| `"Glob"` | 所有 Glob 工具调用 |
| `"Grep"` | 所有 Grep 工具调用 |
| `"Agent"` | 所有 Agent（subagent）工具调用 |
| `"Skill"` | 所有 Skill 工具调用 |
| `"WebFetch"` | 所有 WebFetch 工具调用 |
| `"WebSearch"` | 所有 WebSearch 工具调用 |
| `"mcp__<server>__<tool>"` | 特定 server 上的特定 MCP 工具 |
| `"*"` | 所有工具调用（通配符） |

多个 matchers 可以组合为数组：`["Bash", "Write", "Edit"]`。

---

## 环境变量

command 类型的 hooks 通过环境变量接收事件数据：

| 变量 | 可用于 | 说明 |
|------|--------|------|
| `CLAUDE_HOOK_EVENT` | 全部 | 事件名称（如 `PreToolUse`、`Stop`） |
| `CLAUDE_HOOK_TOOL_NAME` | 工具事件 | 被调用的工具（如 `Bash`、`Write`） |
| `CLAUDE_HOOK_TOOL_INPUT` | `PreToolUse` | 工具输入参数的 JSON 字符串 |
| `CLAUDE_HOOK_TOOL_OUTPUT` | `PostToolUse` | 工具输出的 JSON 字符串 |
| `CLAUDE_HOOK_ERROR` | 失败事件 | 失败操作的错误消息 |
| `CLAUDE_HOOK_SESSION_ID` | 全部 | 当前会话标识符 |
| `CLAUDE_HOOK_CWD` | 全部 | 当前工作目录 |
| `CLAUDE_HOOK_PROJECT_DIR` | 全部 | 项目根目录 |
| `CLAUDE_HOOK_AGENT_NAME` | Agent 事件 | 涉及的 subagent 名称 |
| `CLAUDE_HOOK_MATCHED` | 工具事件 | 触发此 hook 的 matcher |

---

## 实际示例

### 任务完成时的声音通知

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "afplay ~/.claude/hooks/sounds/done.mp3",
        "async": true
      }
    ]
  }
}
```

### 记录所有工具调用

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) $CLAUDE_HOOK_TOOL_NAME\" >> ~/.claude/logs/tools.log",
        "async": true
      }
    ]
  }
}
```

### 安全审计 — 阻止危险命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "python3 .claude/hooks/scripts/security-check.py"
      }
    ]
  }
}
```

脚本检查 `CLAUDE_HOOK_TOOL_INPUT` 中的危险模式（`rm -rf /`、`curl | bash`、`chmod 777`）。非零退出码阻止工具调用。

### 文件写入时自动格式化

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ["Write", "Edit"],
        "type": "command",
        "command": "prettier --write \"$CLAUDE_HOOK_TOOL_INPUT\" 2>/dev/null || true",
        "async": true
      }
    ]
  }
}
```

---

## 决策控制模式

`PreToolUse` hooks 可以通过 stdout 或退出码返回决策信号：

| 模式 | 机制 | 效果 |
|------|------|------|
| **Allow** | 退出码 `0`，stdout 包含 `ALLOW` | 工具调用无需权限提示直接执行 |
| **Deny** | 非零退出码 | 工具调用被阻止，显示错误消息 |
| **Ask** | 退出码 `0`，stdout 包含 `ASK` | 显示标准权限提示 |
| **Defer** | 退出码 `0`，无输出或 stdout 为空 | 降级到默认权限处理 |

对于 `prompt` 和 `agent` 类型的 hooks，注入的响应引导 Claude 的行为，而非直接控制执行。

---

## 最佳实践

1. **对副作用使用 `async: true`** — 声音通知、日志记录和遥测永远不应阻塞 agentic loop。对不需要影响 Claude 行为的 hooks 设置 `"async": true`。

2. **对会话 hooks 使用 `once: true`** — 执行安装（安装依赖、检查环境）的 `SessionStart` hooks 每个会话应只运行一次。

3. **保持超时时间短** — 同步 hooks 会阻塞 Claude 的响应。如果你的 hook 需要超过 2-3 秒，将其设为异步或优化脚本。

4. **用 matchers 限定 hooks 范围** — 当你只需要特定工具时，避免对每个工具调用都运行 hooks。使用 matchers 减少开销。

5. **用 `command` 做把关，用 `prompt` 做引导** — 当你需要硬性的允许/拒绝控制时，使用带退出码的 command hook。当你需要影响 Claude 的方法时，使用 prompt hook。

6. **隔离测试 hooks** — 在添加到 settings 之前，用示例环境变量手动运行你的 hook 脚本。

7. **记录 hook 失败** — 将 hook 命令包装在错误处理逻辑中并记录到文件，使静默失败可被检测。

---

## 常见陷阱

| 陷阱 | 详情 |
|------|------|
| **`Stop` 与 `SubagentStop` 混淆** | `Stop` 仅对**主** agent 触发。对 subagent 完成事件使用 `SubagentStop`。在早期版本中，`Stop` 被称为 `AgentStop` — 后来被重命名 |
| **进程启动开销** | 每个 `command` hook 都会启动一个新的 shell 进程。对于高频事件如带通配符 matcher 的 `PostToolUse`，这会增加延迟。考虑使用 `async: true` 或将逻辑合并到单个脚本中 |
| **忘记 matcher** | 没有 `matcher` 字段的 `PreToolUse` hook 会对**所有**工具调用触发。除非你有意要全局覆盖，否则始终设置 matcher |
| **同步 hooks 阻塞响应** | `Stop` 上的慢速同步 hook 会延迟用户看到 Claude 的响应。始终将非关键 hooks 设为异步 |
| **Hook 输出污染上下文** | 同步 command hooks 的 stdout 会被注入到 Claude 的上下文中。保持输出最小化或重定向到日志文件 |
| **Settings 层级意外** | 在 `.claude/settings.json` 中定义的 hook 可以被 `.claude/settings.local.json` 覆盖。调试 hook 不触发时请检查两个文件 |

---

## 来源

- [Claude Code Hooks — 文档](https://code.claude.com/docs/en/hooks)
- [Claude Code Hooks 指南](https://code.claude.com/docs/en/hooks-guide)
- [Claude Code Settings — 文档](https://code.claude.com/docs/en/settings)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
