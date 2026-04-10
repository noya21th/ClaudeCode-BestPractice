🌍 [English](../../../best-practice/claude-cli-startup-flags.md) | **中文** | [日本語](../../ja/best-practice/claude-cli-startup-flags.md) | [Français](../../fr/best-practice/claude-cli-startup-flags.md) | [Русский](../../ru/best-practice/claude-cli-startup-flags.md) | [العربية](../../ar/best-practice/claude-cli-startup-flags.md)

# CLI 启动参数最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

从终端启动 Claude Code 时的启动参数、顶级子命令和启动环境变量参考。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 目录

1. [会话管理](#会话管理)
2. [模型与配置](#模型与配置)
3. [权限与安全](#权限与安全)
4. [输出与格式](#输出与格式)
5. [System Prompt](#system-prompt)
6. [Agent 与 Subagent](#agent-与-subagent)
7. [MCP 与 Plugins](#mcp-与-plugins)
8. [目录与工作区](#目录与工作区)
9. [预算与限制](#预算与限制)
10. [集成](#集成)
11. [初始化与维护](#初始化与维护)
12. [调试与诊断](#调试与诊断)
13. [Settings 覆盖](#settings-覆盖)
14. [版本与帮助](#版本与帮助)
15. [子命令](#子命令)
16. [环境变量](#环境变量)

---

## 会话管理

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--continue` | `-c` | 继续当前目录中最近的对话 |
| `--resume` | `-r` | 按 ID 或名称恢复特定会话，或显示交互式选择器 |
| `--from-pr <NUMBER\|URL>` | | 恢复链接到特定 GitHub PR 的会话 |
| `--fork-session` | | 恢复时创建新的会话 ID（与 `--resume` 或 `--continue` 一起使用） |
| `--session-id <UUID>` | | 使用特定会话 ID（必须是有效 UUID） |
| `--no-session-persistence` | | 禁用会话持久化（仅 print 模式） |
| `--remote` | | 在 claude.ai 上创建新的网页会话 |
| `--teleport` | | 在本地终端恢复网页会话 |

---

## 模型与配置

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--model <NAME>` | | 使用别名（`sonnet`、`opus`、`haiku`）或完整模型 ID 设置模型 |
| `--fallback-model <NAME>` | | 默认模型过载时的自动回退模型（仅 print 模式） |
| `--betas <LIST>` | | API 请求中包含的 beta headers（仅 API key 用户） |

---

## 权限与安全

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--dangerously-skip-permissions` | | 跳过所有权限提示。请极其谨慎使用 |
| `--allow-dangerously-skip-permissions` | | 启用权限绕过选项但不激活 |
| `--permission-mode <MODE>` | | 以指定权限模式开始：`default`、`plan`、`acceptEdits`、`bypassPermissions` |
| `--allowedTools <TOOLS>` | | 无需提示即可执行的工具（权限规则语法） |
| `--disallowedTools <TOOLS>` | | 从模型上下文中完全移除的工具 |
| `--tools <TOOLS>` | | 限制 Claude 可使用的内置工具（使用 `""` 禁用全部） |
| `--permission-prompt-tool <TOOL>` | | 指定在非交互模式下处理权限提示的 MCP 工具 |

---

## 输出与格式

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--print` | `-p` | 不进入交互模式直接打印响应（headless/SDK 模式） |
| `--output-format <FORMAT>` | | 输出格式：`text`、`json`、`stream-json` |
| `--input-format <FORMAT>` | | 输入格式：`text`、`stream-json` |
| `--json-schema <SCHEMA>` | | 获取匹配 schema 的经过验证的 JSON（仅 print 模式） |
| `--include-partial-messages` | | 包含部分流式事件（需要 `--print` 和 `--output-format=stream-json`） |
| `--verbose` | | 启用详细日志，显示完整的逐轮输出 |

---

## System Prompt

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--system-prompt <TEXT>` | | 用自定义文本替换整个 system prompt |
| `--system-prompt-file <PATH>` | | 从文件加载 system prompt 替换默认值（仅 print 模式） |
| `--append-system-prompt <TEXT>` | | 将自定义文本追加到默认 system prompt |
| `--append-system-prompt-file <PATH>` | | 将文件内容追加到默认 prompt（仅 print 模式） |

---

## Agent 与 Subagent

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--agent <NAME>` | | 为当前会话指定 agent |
| `--agents <JSON>` | | 通过 JSON 动态定义自定义 subagents |
| `--teammate-mode <MODE>` | | 设置 agent team 显示模式：`auto`、`in-process`、`tmux` |

---

## MCP 与 Plugins

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--mcp-config <PATH\|JSON>` | | 从 JSON 文件或字符串加载 MCP servers |
| `--strict-mcp-config` | | 仅使用 `--mcp-config` 中的 MCP servers，忽略其他所有 |
| `--plugin-dir <PATH>` | | 仅在本次会话中从目录加载 plugins（可重复使用） |

---

## 目录与工作区

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--add-dir <PATH>` | | 添加 Claude 可访问的额外工作目录 |
| `--worktree` | `-w` | 在隔离的 git worktree 中启动 Claude（从 HEAD 分支） |

---

## 预算与限制

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--max-budget-usd <AMOUNT>` | | 停止前 API 调用的最大美元金额（仅 print 模式） |
| `--max-turns <NUMBER>` | | 限制 agentic 回合数（仅 print 模式） |

---

## 集成

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--chrome` | | 启用 Chrome 浏览器集成进行 Web 自动化 |
| `--no-chrome` | | 在本次会话中禁用 Chrome 浏览器集成 |
| `--ide` | | 如果恰好有一个有效 IDE 可用则在启动时自动连接 |

---

## 初始化与维护

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--init` | | 运行初始化 hooks 并启动交互模式 |
| `--init-only` | | 运行初始化 hooks 并退出（不启动交互会话） |
| `--maintenance` | | 运行维护 hooks 并退出 |

---

## 调试与诊断

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--debug <CATEGORIES>` | | 启用调试模式，可选类别过滤（如 `"api,hooks"`） |

---

## Settings 覆盖

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--settings <PATH\|JSON>` | | 要加载的 settings JSON 文件路径或 JSON 字符串 |
| `--setting-sources <LIST>` | | 逗号分隔的要加载的来源列表：`user`、`project`、`local` |
| `--disable-slash-commands` | | 在本次会话中禁用所有 skills 和 slash commands |

---

## 版本与帮助

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--version` | `-v` | 输出版本号 |
| `--help` | `-h` | 显示帮助信息 |

---

## 子命令

这些是作为 `claude <subcommand>` 运行的顶级命令：

| 子命令 | 说明 |
|--------|------|
| `claude` | 启动交互式 REPL |
| `claude "query"` | 带初始 prompt 启动 REPL |
| `claude agents` | 列出已配置的 agents |
| `claude auth` | 管理 Claude Code 认证 |
| `claude doctor` | 从命令行运行诊断 |
| `claude install` | 安装或切换 Claude Code 原生构建 |
| `claude mcp` | 配置 MCP servers（`add`、`remove`、`list`、`get`、`enable`） |
| `claude plugin` | 管理 Claude Code plugins |
| `claude remote-control` | 管理远程控制会话 |
| `claude setup-token` | 创建用于订阅使用的长期 token |
| `claude update` / `claude upgrade` | 更新到最新版本 |

---

## 环境变量

这些仅在启动时使用的环境变量在启动 Claude Code 前设置在 shell 中（不能通过 `settings.json` 配置）：

| 变量 | 说明 |
|------|------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 启用实验性 agent teams |
| `CLAUDE_CODE_TMPDIR` | 覆盖内部文件的临时目录。也可通过 `env` 键配置 — 参见 [Settings 参考](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 启用额外目录的 CLAUDE.md 加载 |
| `DISABLE_AUTOUPDATER=1` | 禁用自动更新 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 控制思考深度 — 参见 [Settings 参考](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `USE_BUILTIN_RIPGREP=0` | 使用系统 ripgrep 代替内置版本（Alpine Linux） |
| `CLAUDE_CODE_SIMPLE` | 启用简单模式（仅 Bash + Edit 工具）。也可通过 `env` 键配置 — 参见 [Settings 参考](../../../best-practice/claude-settings.md#environment-variables-via-env) |
| `CLAUDE_BASH_NO_LOGIN=1` | 跳过 BashTool 的 login shell |

关于可通过 `settings.json` 中 `"env"` 键配置的环境变量（包括 `MAX_THINKING_TOKENS`、`CLAUDE_CODE_SHELL`、`CLAUDE_CODE_ENABLE_TASKS`、`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`、`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 等），参见 [Claude Settings 参考](../../../best-practice/claude-settings.md#environment-variables-via-env)。

---

## 来源

- [Claude Code CLI 参考](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Headless 模式](https://code.claude.com/docs/en/headless)
- [Claude Code 安装](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code 常用工作流](https://code.claude.com/docs/en/common-workflows)
