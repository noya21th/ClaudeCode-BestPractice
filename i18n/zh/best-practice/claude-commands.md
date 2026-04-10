🌍 [English](../../../best-practice/claude-commands.md) | **中文** | [日本語](../../ja/best-practice/claude-commands.md) | [Français](../../fr/best-practice/claude-commands.md) | [Русский](../../ru/best-practice/claude-commands.md) | [العربية](../../ar/best-practice/claude-commands.md)

# Commands 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A35%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-commands-implementation.md)

Claude Code commands — frontmatter 字段和官方内置 slash commands。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter 字段 (13)

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 否 | 显示名称和 `/slash-command` 标识符。省略时默认为目录名 |
| `description` | string | 推荐 | command 的功能。在自动补全中显示并被 Claude 用于自动发现 |
| `argument-hint` | string | 否 | 自动补全时显示的提示（如 `[issue-number]`、`[filename]`） |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 防止 Claude 自动调用此 command |
| `user-invocable` | boolean | 否 | 设为 `false` 从 `/` 菜单中隐藏 — command 变为仅后台知识 |
| `paths` | string/list | 否 | 限制此 skill 激活条件的 Glob 模式。接受逗号分隔字符串或 YAML 列表。设置后，仅当处理匹配文件时 Claude 才自动加载此 skill |
| `allowed-tools` | string | 否 | 此 command 激活时无需权限提示即可使用的工具 |
| `model` | string | 否 | 此 command 运行时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型的 effort level（`low`、`medium`、`high`、`max`） |
| `context` | string | 否 | 设为 `fork` 在隔离的 subagent 上下文中运行 command |
| `agent` | string | 否 | 设置 `context: fork` 时的 subagent 类型（默认：`general-purpose`） |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell — 接受 `bash`（默认）或 `powershell`。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |
| `hooks` | object | 否 | 范围限定于此 command 的生命周期 hooks |

---

## ![Official](../../../!/tags/official.svg) **(65)**

| # | Command | 标签 | 说明 |
|---|---------|------|------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登录你的 Anthropic 账号 |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 登出你的 Anthropic 账号 |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 通过交互式向导配置 Amazon Bedrock 认证、区域和模型固定。仅在设置 `CLAUDE_CODE_USE_BEDROCK=1` 时可见。首次使用 Bedrock 的用户也可从登录界面访问此向导 |
| 4 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 打开升级页面切换到更高计划等级 |
| 5 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 设置当前会话的提示栏颜色。可用颜色：`red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink`、`cyan`。使用 `default` 重置 |
| 6 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开设置界面调整主题、模型、输出样式等偏好。别名：`/settings` |
| 7 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 打开或创建快捷键配置文件 |
| 8 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 管理工具权限的 allow、ask 和 deny 规则。打开交互式对话框，可按范围查看规则、添加或移除规则、管理工作目录以及查看最近的 auto mode 拒绝记录。别名：`/allowed-tools` |
| 9 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 查看和更新你的隐私设置。仅 Pro 和 Max 计划订阅者可用 |
| 10 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换沙盒模式。仅在支持的平台上可用 |
| 11 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置 Claude Code 的 status line。描述你想要的内容，或不带参数运行以从 shell 提示符自动配置 |
| 12 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 订购 Claude Code 贴纸 |
| 13 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 配置终端快捷键（Shift+Enter 等）。仅在需要的终端中可见，如 VS Code、Alacritty 或 Warp |
| 14 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 更改颜色主题。包含浅色和深色变体、色盲友好（daltonized）主题以及使用终端颜色板的 ANSI 主题 |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 切换按键说话的语音输入。需要 Claude.ai 账号 |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 以彩色网格可视化当前上下文用量。显示高上下文工具、记忆膨胀和容量警告的优化建议 |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示 token 使用统计。订阅特定的详细信息请参阅费用追踪指南 |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 配置额外用量以在触及速率限制时继续工作 |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 生成分析 Claude Code 会话的报告，包含项目区域、交互模式和摩擦点 |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 可视化每日用量、会话历史、连续使用天数和模型偏好 |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 打开设置界面（状态标签）显示版本、模型、账号和连接状态。在 Claude 响应时也可使用，无需等待当前响应完成 |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 显示计划用量限制和速率限制状态 |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 诊断并验证你的 Claude Code 安装和设置 |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 提交关于 Claude Code 的反馈。别名：`/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 显示帮助和可用命令 |
| 26 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 通过带动画演示的快速交互式课程发现 Claude Code 功能 |
| 27 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 在交互式版本选择器中查看更新日志。选择特定版本查看其发布说明，或选择显示所有版本 |
| 28 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 列出和管理后台任务。别名：`/bashes` |
| 29 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将最后一条助手响应复制到剪贴板。传入数字 `N` 复制倒数第 N 条响应：`/copy 2` 复制倒数第二条。存在代码块时显示交互式选择器选择单个代码块或完整响应。在选择器中按 `w` 将选择写入文件而非剪贴板，适用于 SSH 连接 |
| 30 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 将当前对话导出为纯文本。带文件名时直接写入该文件。不带参数时打开对话框选择复制到剪贴板或保存到文件 |
| 31 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 agent 配置 |
| 32 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 配置 Claude in Chrome 设置 |
| 33 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 查看工具事件的 hook 配置 |
| 34 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 IDE 集成并显示状态 |
| 35 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 MCP server 连接和 OAuth 认证 |
| 36 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 管理 Claude Code plugins |
| 37 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 重新加载所有活跃 plugins 以应用待处理的更改而无需重启。报告每个重新加载组件的数量并标记任何加载错误 |
| 38 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 列出可用 skills |
| 39 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | 编辑 `CLAUDE.md` 记忆文件，启用或禁用自动记忆，查看自动记忆条目 |
| 40 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 设置模型 effort level。`low`、`medium`、`high` 跨会话持久化。`max` 仅适用于当前会话，需要 Opus 4.6。`auto` 重置为模型默认值。不带参数显示当前级别。立即生效，无需等待当前响应完成 |
| 41 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 切换快速模式的开关 |
| 42 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 选择或更换 AI 模型。对于支持的模型，使用左右方向键调整 effort level。更改立即生效，无需等待当前响应完成 |
| 43 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 与朋友分享一周免费的 Claude Code。仅在账号符合条件时可见 |
| 44 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 从提示直接进入 plan mode。传入可选描述以进入 plan mode 并立即开始该任务，例如 `/plan fix the auth bug` |
| 45 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 在 ultraplan 会话中起草计划，在浏览器中审查，然后远程执行或发送回终端 |
| 46 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 向当前会话添加新的工作目录 |
| 47 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 打开交互式 diff 查看器显示未提交的更改和每回合的 diff。使用左右方向键在当前 git diff 和各 Claude 回合之间切换，上下方向键浏览文件 |
| 48 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 使用 `CLAUDE.md` 指南初始化项目。设置 `CLAUDE_CODE_NEW_INIT=1` 可启用交互式流程，还会引导设置 skills、hooks 和个人记忆文件 |
| 49 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 已弃用。请改为安装 `code-review` plugin：`claude plugin install code-review@claude-plugins-official` |
| 50 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 分析当前分支的待处理更改中的安全漏洞。审查 git diff 并识别注入、认证问题和数据暴露等风险 |
| 51 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 在 Claude Code Desktop 应用中继续当前会话。仅 macOS 和 Windows。别名：`/app` |
| 52 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为仓库设置 Claude GitHub Actions 应用。引导你选择仓库和配置集成 |
| 53 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 安装 Claude Slack 应用。打开浏览器完成 OAuth 流程 |
| 54 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 显示二维码下载 Claude 移动应用。别名：`/ios`、`/android` |
| 55 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 使此会话可从 claude.ai 远程控制。别名：`/rc` |
| 56 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 为使用 `--remote` 启动的网页会话配置默认远程环境 |
| 57 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 创建、更新、列出或运行云端定时任务。Claude 通过对话引导你完成设置 |
| 58 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 在此处创建当前对话的分支。别名：`/fork` |
| 59 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 快速提一个不加入对话的旁问 |
| 60 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 清除对话历史并释放上下文。别名：`/reset`、`/new` |
| 61 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 压缩对话，可附带可选的聚焦指令 |
| 62 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 退出 CLI。别名：`/quit` |
| 63 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 重命名当前会话并在提示栏显示名称。不带名称时从对话历史自动生成 |
| 64 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 按 ID 或名称恢复对话，或打开会话选择器。别名：`/continue` |
| 65 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 将对话和/或代码回退到之前的节点，或从选定消息开始摘要。参见 checkpointing。别名：`/checkpoint` |

内置 skills 如 `/debug` 也可能出现在 slash-command 菜单中，但它们不是内置 commands。

---

## 来源

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
