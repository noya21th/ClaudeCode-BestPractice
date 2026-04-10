🌍 [English](../../../best-practice/claude-settings.md) | **中文** | [日本語](../../ja/best-practice/claude-settings.md) | [Français](../../fr/best-practice/claude-settings.md) | [Русский](../../ru/best-practice/claude-settings.md) | [العربية](../../ar/best-practice/claude-settings.md)

# Settings 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%2010%3A16%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.claude/settings.json)

Claude Code `settings.json` 文件中所有可用配置选项的完整指南。截至 v2.1.96，Claude Code 提供了 **60+ 个设置项**和 **170+ 个环境变量**（使用 `settings.json` 中的 `"env"` 字段即可避免编写包装脚本）。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## 目录

1. [设置层级](#设置层级)
2. [核心配置](#核心配置)
3. [权限](#权限)
4. [Hooks](#hooks)
5. [MCP Servers](#mcp-servers)
6. [Sandbox](#sandbox)
7. [Plugins](#plugins)
8. [模型配置](#模型配置)
9. [显示与用户体验](#显示与用户体验)
10. [AWS 与云凭证](#aws-与云凭证)
11. [环境变量（通过 env）](#环境变量通过-env)
12. [常用命令](#常用命令)

---

## 设置层级

设置按优先级从高到低依次生效：

| 优先级 | 位置 | 作用域 | 是否共享 | 用途 |
|--------|------|--------|----------|------|
| 1 | Managed settings | 组织级 | 是（由 IT 部署） | 不可被覆盖的安全策略 |
| 2 | 命令行参数 | 会话级 | 不适用 | 单次会话的临时覆盖 |
| 3 | `.claude/settings.local.json` | 项目级 | 否（已加入 git-ignored） | 个人的项目特定设置 |
| 4 | `.claude/settings.json` | 项目级 | 是（已提交） | 团队共享设置 |
| 5 | `~/.claude/settings.json` | 用户级 | 不适用 | 全局个人默认设置 |

**Managed settings** 由组织强制执行，无法被其他任何层级覆盖，包括命令行参数。分发方式：
- **Server-managed** settings（远程分发）
- **MDM profiles** — macOS plist 位于 `com.anthropic.claudecode`
- **Registry policies** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`（管理员级）和 `HKCU\SOFTWARE\Policies\ClaudeCode`（用户级，策略优先级最低）
- **文件** — `managed-settings.json` 和 `managed-mcp.json`（macOS：`/Library/Application Support/ClaudeCode/`，Linux/WSL：`/etc/claude-code/`，Windows：`C:\Program Files\ClaudeCode\`）
- **Drop-in 目录** — 与 `managed-settings.json` 同级的 `managed-settings.d/` 目录，用于独立策略片段（v2.1.83）。遵循 systemd 约定，先合并 `managed-settings.json` 作为基础，再按字母顺序合并 drop-in 目录中的所有 `*.json` 文件。标量值会被后面的文件覆盖；数组会拼接并去重；对象会深度合并。以 `.` 开头的隐藏文件将被忽略。使用数字前缀来控制合并顺序（例如 `10-telemetry.json`、`20-security.json`）

在 managed 层级内，优先级为：server-managed > MDM/OS 级策略 > 文件级（`managed-settings.d/*.json` + `managed-settings.json`）> HKCU registry（仅 Windows）。只会使用一个 managed 来源，不同来源之间不会互相合并。在文件级内部，drop-in 文件和基础文件会合并在一起。

> **注意：** 自 v2.1.75 起，已废弃的 Windows 回退路径 `C:\ProgramData\ClaudeCode\managed-settings.json` 已移除。请改用 `C:\Program Files\ClaudeCode\managed-settings.json`。

**重要提示**：
- `deny` 规则拥有最高的安全优先级，无法被低优先级的 allow/ask 规则覆盖。
- Managed settings 可能会锁定或覆盖本地行为，即使本地文件指定了不同的值。
- 数组设置（如 `permissions.allow`）在各作用域之间**拼接并去重** — 所有层级的条目会合并，而非替换。

---

## 核心配置

### 通用设置

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `$schema` | string | - | 用于 IDE 校验和自动补全的 JSON Schema URL（例如 `"https://json.schemastore.org/claude-code-settings.json"`） |
| `model` | string | `"default"` | 覆盖默认模型。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID |
| `agent` | string | - | 设置主会话的默认 agent。值为 `.claude/agents/` 中的 agent 名称。也可通过 `--agent` CLI 参数设置 |
| `language` | string | `"english"` | Claude 的首选响应语言。同时设定语音听写语言 |
| `cleanupPeriodDays` | number | `30` | 超过此天数的不活跃会话会在启动时被删除（最小值 1）。同时控制启动时自动清理孤立 subagent worktree 的时间阈值。设为 `0` 会触发校验错误。要在非交互模式（`-p`）中禁用会话持久化，请使用 `--no-session-persistence` 或 SDK 的 `persistSession: false` 选项 |
| `autoUpdatesChannel` | string | `"latest"` | 发布渠道：`"stable"` 或 `"latest"` |
| `alwaysThinkingEnabled` | boolean | `false` | 为所有会话默认启用 extended thinking |
| `skipWebFetchPreflight` | boolean | `false` | 跳过 WebFetch 抓取 URL 前的黑名单检查 *（存在于 JSON schema 中，但不在官方设置页面上）* |
| `availableModels` | array | - | 限制用户可通过 `/model`、`--model`、Config 工具或 `ANTHROPIC_MODEL` 选择的模型。不影响 Default 选项。示例：`["sonnet", "haiku"]` |
| `fastModePerSessionOptIn` | boolean | `false` | 要求用户在每次会话中手动选择启用 fast mode |
| `defaultShell` | string | `"bash"` | 输入框 `!` 命令的默认 shell。接受 `"bash"`（默认）或 `"powershell"`。设为 `"powershell"` 后，Windows 上的交互式 `!` 命令将通过 PowerShell 路由。需要设置 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`（v2.1.84） |
| `includeGitInstructions` | boolean | `true` | 在 Claude 的 system prompt 中包含内置的 commit 和 PR 工作流指令以及 git status 快照。当设置了 `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 环境变量时，该变量优先于此设置 |
| `voiceEnabled` | boolean | - | 启用按键说话的语音听写功能。运行 `/voice` 时会自动写入。需要 Claude.ai 账号 |
| `showClearContextOnPlanAccept` | boolean | `false` | 在 plan 确认界面显示"清除上下文"选项。设为 `true` 可恢复该选项（自 v2.1.81 起默认隐藏） |
| `disableDeepLinkRegistration` | string | - | 设为 `"disable"` 可阻止 Claude Code 在启动时向操作系统注册 `claude-cli://` 协议处理程序。Deep links 允许外部工具通过 `claude-cli://open?q=...` 打开预填提示的 Claude Code 会话。`q` 参数支持使用 URL 编码换行符（`%0A`）的多行提示。适用于协议处理程序注册受限或由其他方式管理的环境 |
| `showThinkingSummaries` | boolean | `false` | 在交互式会话中显示 extended thinking 摘要。未设置或为 `false` 时（交互模式默认），thinking blocks 会被 API 编辑并以折叠形式显示。编辑仅改变展示内容，不影响模型实际生成 — 要减少 thinking 消耗，请降低预算或禁用 thinking。非交互模式（`-p`）和 SDK 调用者无论此设置如何始终接收摘要 |
| `disableSkillShellExecution` | boolean | `false` | 禁用 skills 和自定义命令中 `` !`...` `` 块的内联 shell 执行。命令将被替换为 `[shell command execution disabled by policy]`。Bundled 和 managed skills 不受影响（v2.1.91） |
| `forceRemoteSettingsRefresh` | boolean | `false` | **（仅 Managed）** 在 CLI 启动前阻塞，直到远程 managed settings 刷新完成。如果获取失败，CLI 退出（fail-closed）。用于需要在会话开始前确保策略为最新状态的企业环境（v2.1.92） |
| `feedbackSurveyRate` | number | - | 符合条件时显示会话质量调查的概率（0–1）。企业管理员可控制调查显示频率。示例：`0.05` = 5% 的符合条件的会话 |

**示例：**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### Plans 与 Memory 目录

将 plan 和 auto-memory 文件存储在自定义位置。

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 输出存储的目录 |
| `autoMemoryDirectory` | string | - | auto-memory 的自定义存储目录。支持 `~/` 路径展开。不允许在项目设置（`.claude/settings.json`）中使用，以防止将 memory 写入重定向到敏感位置；可在 policy、local 和 user settings 中使用 |

**示例：**
```json
{
  "plansDirectory": "./my-plans"
}
```

**使用场景：** 适用于将规划产物与 Claude 内部文件分开整理，或将 plans 保存在团队共享位置。

### Worktree 设置

配置 `--worktree` 如何创建和管理 git worktrees。适用于减少大型 monorepo 中的磁盘占用和启动时间。

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `worktree.symlinkDirectories` | array | `[]` | 从主仓库创建符号链接到每个 worktree 的目录，避免在磁盘上重复大型目录 |
| `worktree.sparsePaths` | array | `[]` | 通过 git sparse-checkout（cone 模式）在每个 worktree 中检出的目录。只有列出的路径会写入磁盘 |

**示例：**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### 归属设置

自定义 git commits 和 pull requests 的归属信息。

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `attribution.commit` | string | Co-authored-by | Git commit 归属（支持 trailers） |
| `attribution.pr` | string | 自动生成的消息 | Pull request 描述归属 |
| `includeCoAuthoredBy` | boolean | `true` | **已废弃** - 请改用 `attribution` |

**示例：**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**注意：** 设为空字符串（`""`）可完全隐藏归属信息。

### 身份认证辅助

用于动态身份认证 token 生成的脚本。

| 键 | 类型 | 说明 |
|----|------|------|
| `apiKeyHelper` | string | 输出 auth token 的 shell 脚本路径（作为 `X-Api-Key` header 发送） |
| `forceLoginMethod` | string | 限制登录方式为 `"claudeai"` 或 `"console"` 账号 |
| `forceLoginOrgUUID` | string \| array | 要求登录账号属于指定组织。接受单个 UUID 字符串（同时会在登录时预选该组织）或 UUID 数组（列出的任意组织均可接受，无预选）。在 managed settings 中设置时，如果已认证账号不属于列出的组织，登录将失败；空数组将采取 fail-closed 策略并以配置错误消息阻止登录 |

**示例：**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### 公司公告

在启动时向用户显示自定义公告（随机轮播）。

| 键 | 类型 | 说明 |
|----|------|------|
| `companyAnnouncements` | array | 启动时显示的字符串数组 |

**示例：**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## 权限

控制 Claude 可执行的工具和操作。

### 权限结构

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### 权限键

| 键 | 类型 | 说明 |
|----|------|------|
| `permissions.allow` | array | 允许工具使用且无需提示的规则 |
| `permissions.ask` | array | 需要用户确认的规则 |
| `permissions.deny` | array | 阻止工具使用的规则（最高优先级） |
| `permissions.additionalDirectories` | array | Claude 可访问的额外目录 |
| `permissions.defaultMode` | string | 默认权限模式。在远程环境中，仅 `acceptEdits` 和 `plan` 生效（v2.1.70+） |
| `permissions.disableBypassPermissionsMode` | string | 阻止 bypass 模式激活 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | 跳过通过 `--dangerously-skip-permissions` 或 `defaultMode: "bypassPermissions"` 进入 bypass 权限模式前显示的确认提示。在项目设置（`.claude/settings.json`）中设置时会被忽略，以防止不受信任的仓库自动绕过该提示 |
| `allowManagedPermissionRulesOnly` | boolean | **（仅 Managed）** 仅应用 managed 权限规则；用户/项目的 `allow`、`ask`、`deny` 规则将被忽略 |
| `allow_remote_sessions` | boolean | **（仅 Managed）** 允许用户启动 Remote Control 和 web 会话。默认为 `true`。设为 `false` 可阻止远程会话访问 *（不在官方文档中 — 官方权限页面指出"Remote Control 和 web 会话的访问不受 managed settings 键控制。"在 Team 和 Enterprise 计划中，管理员通过 [Claude Code admin settings](https://claude.ai/admin-settings/claude-code) 启用/禁用）* |
| `autoMode` | object | 自定义 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) 分类器的阻止和允许行为。包含 `environment`（受信基础设施描述）、`allow`（阻止规则的例外）和 `soft_deny`（阻止规则）— 均为字符串数组。**不从共享项目设置**（`.claude/settings.json`）读取，以防止仓库注入。可在 user、local 和 managed settings 中使用。设置 `allow` 或 `soft_deny` 会**替换**该部分的整个默认列表。运行 `claude auto-mode defaults` 查看内置规则后再自定义 |
| `disableAutoMode` | string | 设为 `"disable"` 可阻止 [auto mode](/en/permission-modes#eliminate-prompts-with-auto-mode) 激活。从 `Shift+Tab` 循环中移除 `auto`，并在启动时拒绝 `--permission-mode auto`。可在任何设置层级设置；在 managed settings 中最为有用，因为用户无法覆盖 |
| `useAutoModeDuringPlan` | boolean | 当 auto mode 可用时，plan 模式是否使用 auto mode 语义。默认：`true`。不从共享项目设置（`.claude/settings.json`）读取。在 `/config` 中显示为"Use auto mode during plan" |

### 权限模式

| 模式 | 行为 |
|------|------|
| `"default"` | 标准权限检查并提示确认 |
| `"acceptEdits"` | 自动接受文件编辑，无需询问 |
| `"askEdits"` | 每次操作前都询问 *（不在官方文档中 — 未验证）* |
| `"dontAsk"` | 自动拒绝工具调用，除非已通过 `/permissions` 或 `permissions.allow` 规则预先批准 |
| `"viewOnly"` | 只读模式，不进行修改 *（不在官方文档中 — 未验证）* |
| `"bypassPermissions"` | 跳过所有权限检查（危险） |
| `"auto"` | 后台分类器替代手动提示（`--enable-auto-mode`）。研究预览 — 需要 Team 计划 + Sonnet/Opus 4.6。分类器自动批准只读和文件编辑操作；其他操作进入安全检查。连续 3 次或累计 20 次阻止后回退到手动提示。通过 `autoMode` 设置配置 |
| `"plan"` | 只读探索模式 |

### 工具权限语法

| 工具 | 语法 | 示例 |
|------|------|------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`、`Bash(* install)`、`Bash(git * main)` |
| `Read` | `Read(path pattern)` | `Read(.env)`、`Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`、`Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`、`Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 全局网页搜索 |
| `Task` | `Task(agent-name)` | `Task(Explore)`、`Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`、`Agent(*)` — 权限范围限定于 subagent 创建 |
| `Skill` | `Skill(skill-name)` | `Skill(weather-fetcher)` |
| `MCP` | `mcp__server__tool` 或 `MCP(server:tool)` | `mcp__memory__*`、`MCP(github:*)` |

**评估顺序：** 规则按顺序评估：先 deny，再 ask，最后 allow。第一个匹配的规则生效。

**Read/Edit 路径模式：** `Read`、`Edit` 和 `Write` 的权限规则支持 gitignore 风格的模式匹配，有四种前缀类型：

| 前缀 | 含义 | 示例 |
|------|------|------|
| `//` | 从文件系统根目录开始的绝对路径 | `Read(//Users/alice/file)` |
| `~/` | 相对于 home 目录 | `Read(~/.zshrc)` |
| `/` | 相对于项目根目录 | `Edit(/src/**)` |
| `./` 或无前缀 | 相对路径（当前目录） | `Read(.env)`、`Read(*.ts)` |

**Bash 通配符说明：**
- `*` 可以出现在**任意位置**：前缀（`Bash(* install)`）、后缀（`Bash(npm *)`）或中间（`Bash(git * main)`）
- **词边界：** `Bash(ls *)`（`*` 前有空格）匹配 `ls -la` 但不匹配 `lsof`；`Bash(ls*)`（无空格）两者都匹配
- `Bash(*)` 等同于 `Bash`（匹配所有 bash 命令）
- 权限规则支持输出重定向：`Bash(python:*)` 匹配 `python script.py > output.txt`
- 旧式 `:*` 后缀语法（如 `Bash(npm:*)`）等同于 ` *`，但已废弃

**示例：**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

Hook 配置（事件、属性、匹配器、退出码、环境变量和 HTTP hooks）维护在专用仓库中：

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 完整的 hook 参考，包含声音通知系统、全部 25 个 hook 事件、HTTP hooks、匹配器模式、退出码和环境变量。

Hook 相关设置键（`hooks`、`disableAllHooks`、`allowManagedHooksOnly`、`allowedHttpHookUrls`、`httpHookAllowedEnvVars`）在该仓库中有文档说明。

官方 hooks 参考请参阅 [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks)。

---

## MCP Servers

配置 Model Context Protocol 服务器以获取扩展能力。

### MCP 设置

| 键 | 类型 | 作用域 | 说明 |
|----|------|--------|------|
| `enableAllProjectMcpServers` | boolean | 任意 | 自动批准 `.mcp.json` 中的所有服务器 |
| `enabledMcpjsonServers` | array | 任意 | 特定服务器名称的白名单 |
| `disabledMcpjsonServers` | array | 任意 | 特定服务器名称的黑名单 |
| `allowedMcpServers` | array | 仅 Managed | 使用名称/命令/URL 匹配的白名单 |
| `deniedMcpServers` | array | 仅 Managed | 使用匹配的黑名单 |
| `allowManagedMcpServersOnly` | boolean | 仅 Managed | 仅允许 managed 白名单中明确列出的 MCP servers |
| `channelsEnabled` | boolean | 仅 Managed | 允许 Team 和 Enterprise 用户使用 [channels](https://code.claude.com/docs/en/channels)。未设置或为 `false` 时，无论 `--channels` 参数如何，channel 消息传递均被阻止 |
| `allowedChannelPlugins` | array | 仅 Managed | 可推送消息的 channel plugins 白名单。设置后将替换默认的 Anthropic 白名单。未定义 = 回退到默认值，空数组 = 阻止所有 channel plugins。需要 `channelsEnabled: true`。每个条目是包含 `marketplace` 和 `plugin` 字段的对象（v2.1.84） |

### MCP Server 匹配（Managed Settings）

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @modelcontextprotocol/*" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

**示例：**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

配置 bash 命令沙箱以增强安全性。

### Sandbox 设置

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `sandbox.enabled` | boolean | `false` | 启用 bash 沙箱 |
| `sandbox.failIfUnavailable` | boolean | `false` | 沙箱启用但无法启动时以错误退出，而非在非沙箱状态下运行。适用于需要严格沙箱的企业策略（v2.1.83） |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 沙箱启用时自动批准 bash 命令 |
| `sandbox.excludedCommands` | array | `[]` | 在沙箱外运行的命令 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | 允许使用 `dangerouslyDisableSandbox`。设为 `false` 时，逃逸接口完全禁用，所有命令必须在沙箱中运行（或在 `excludedCommands` 中）。适用于需要严格沙箱的企业策略 |
| `sandbox.ignoreViolations` | object | `{}` | 命令模式到路径数组的映射 — 抑制违规警告 *（存在于 JSON schema 中，但不在官方设置页面上）* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | 在 Docker 中使用较弱的沙箱（降低安全性） |
| `sandbox.network.allowUnixSockets` | array | `[]` | 沙箱中可访问的特定 Unix socket 路径 |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 允许所有 Unix sockets（覆盖 allowUnixSockets） |
| `sandbox.network.allowLocalBinding` | boolean | `false` | 允许绑定到 localhost 端口（macOS） |
| `sandbox.network.allowedDomains` | array | `[]` | 沙箱的网络域名白名单 |
| `sandbox.network.deniedDomains` | array | `[]` | 沙箱的网络域名黑名单 *（不在官方文档中 — 未验证）* |
| `sandbox.network.httpProxyPort` | number | - | HTTP 代理端口 1-65535（自定义代理） |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 代理端口 1-65535（自定义代理） |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 仅允许 managed 白名单中的域名（managed settings） |
| `sandbox.filesystem.allowWrite` | array | `[]` | 沙箱命令可写入的额外路径。数组在所有设置作用域中合并。也与 `Edit(...)` allow 权限规则中的路径合并。前缀：`/`（绝对路径）、`~/`（home 目录）、`./` 或无前缀（项目设置中为项目相对路径，用户设置中为 `~/.claude` 相对路径）。旧式 `//` 前缀用于绝对路径仍然有效。**注意：** 这与 [Read/Edit 权限规则](#工具权限语法)不同，后者使用 `//` 表示绝对路径，`/` 表示项目相对路径 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 沙箱命令不可写入的路径。数组在所有设置作用域中合并。也与 `Edit(...)` deny 权限规则中的路径合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.denyRead` | array | `[]` | 沙箱命令不可读取的路径。数组在所有设置作用域中合并。也与 `Read(...)` deny 权限规则中的路径合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.allowRead` | array | `[]` | 在 `denyRead` 区域内重新允许读取访问的路径。优先于 `denyRead`。数组在所有设置作用域中合并。路径前缀约定与 `allowWrite` 相同 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **（仅 Managed）** 仅使用 managed settings 中的 `allowRead` 路径。来自 user、project 和 local settings 的 `allowRead` 条目将被忽略 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | （仅 macOS）允许访问系统 TLS 信任（`com.apple.trustd.agent`）；降低安全性 |

**示例：**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## Plugins

配置 Claude Code plugins 和 marketplaces。

### Plugin 设置

| 键 | 类型 | 作用域 | 说明 |
|----|------|--------|------|
| `enabledPlugins` | object | 任意 | 启用/禁用特定 plugins |
| `extraKnownMarketplaces` | object | 项目级 | 添加自定义 plugin marketplaces（通过 `.claude/settings.json` 进行团队共享） |
| `strictKnownMarketplaces` | array | 仅 Managed | 允许使用的 marketplaces 白名单 |
| `skippedMarketplaces` | array | 任意 | 用户拒绝安装的 marketplaces *（存在于 JSON schema 中，但不在官方设置页面上）* |
| `skippedPlugins` | array | 任意 | 用户拒绝安装的 plugins *（存在于 JSON schema 中，但不在官方设置页面上）* |
| `pluginConfigs` | object | 任意 | 每个 plugin 的 MCP server 配置（以 `plugin@marketplace` 为键） *（存在于 JSON schema 中，但不在官方设置页面上）* |
| `blockedMarketplaces` | array | 仅 Managed | 阻止特定 plugin marketplaces |
| `pluginTrustMessage` | string | 仅 Managed | 提示用户信任 plugins 时显示的自定义消息 |

**Marketplace 来源类型：** `github`、`git`、`directory`、`hostPattern`、`settings`、`url`、`npm`、`file`。使用 `source: 'settings'` 可在不搭建托管 marketplace 仓库的情况下内联声明少量 plugins。

**示例：**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## 模型配置

### 模型别名

| 别名 | 说明 |
|------|------|
| `"default"` | 推荐用于你的账户类型 |
| `"sonnet"` | 最新 Sonnet 模型（Claude Sonnet 4.6） |
| `"opus"` | 最新 Opus 模型（Claude Opus 4.6） |
| `"haiku"` | 快速 Haiku 模型 |
| `"sonnet[1m]"` | 1M token 上下文的 Sonnet |
| `"opus[1m]"` | 1M token 上下文的 Opus（自 v2.1.75 起在 Max、Team 和 Enterprise 上默认启用） |
| `"opusplan"` | 规划用 Opus，执行用 Sonnet |

**示例：**
```json
{
  "model": "opus"
}
```

### 模型覆盖

将 Anthropic 模型 ID 映射到 Bedrock、Vertex 或 Foundry 部署的提供商特定模型 ID。

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `effortLevel` | string | - | 跨会话持久化 effort level。接受 `"low"`、`"medium"` 或 `"high"`。运行 `/effort low`、`/effort medium` 或 `/effort high` 时会自动写入。支持 Opus 4.6 和 Sonnet 4.6 |
| `modelOverrides` | object | - | 将模型选择器条目映射到提供商特定 ID（例如 Bedrock inference profile ARN）。每个键是模型选择器条目名称，每个值是提供商模型 ID |

**示例：**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Effort Level

`/model` 命令提供了 **effort level** 控制，可调节模型在每次响应中投入的推理程度。在 `/model` UI 中使用 ← → 方向键切换 effort level。

| Effort Level | 说明 |
|-------------|------|
| High（默认） | 完整推理深度，最适合复杂任务 |
| Medium | 平衡推理，适合日常任务 |
| Low | 最少推理，最快响应 |

**使用方法：**
1. 运行 `/effort low`、`/effort medium` 或 `/effort high` 直接设置（v2.1.76+）
2. 或运行 `/model` → 选择模型 → 使用 **← →** 方向键调整
3. 该设置通过 `settings.json` 中的 `effortLevel` 键持久化

**注意：** Effort level 可用于 Max 和 Team 计划中的 Opus 4.6 和 Sonnet 4.6。默认值在 v2.1.68 中从 High 改为 Medium，随后在 v2.1.94 中对 API-key、Bedrock/Vertex/Foundry、Team 和 Enterprise 用户改回 **High**。自 v2.1.75 起，Opus 4.6 的 1M 上下文窗口在 Max、Team 和 Enterprise 计划上默认可用。

### 模型环境变量

通过 `env` 键配置：

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## 显示与用户体验

### 显示设置

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `statusLine` | object | - | 自定义状态栏配置 |
| `outputStyle` | string | `"default"` | 输出样式（例如 `"Explanatory"`） |
| `spinnerTipsEnabled` | boolean | `true` | 等待时显示提示 |
| `spinnerVerbs` | object | - | 自定义加载动画文字，包含 `mode`（"append" 或 "replace"）和 `verbs` 数组 |
| `spinnerTipsOverride` | object | - | 自定义加载提示，包含 `tips`（字符串数组）和可选的 `excludeDefault`（boolean） |
| `respectGitignore` | boolean | `true` | 在文件选择器中遵循 .gitignore |
| `prefersReducedMotion` | boolean | `false` | 减少 UI 中的动画和动效 |
| `fileSuggestion` | object | - | 自定义文件建议命令（参见下方文件建议配置） |

### 全局配置设置（`~/.claude.json`）

这些显示偏好存储在 `~/.claude.json` 中，**而非** `settings.json`。将它们添加到 `settings.json` 会触发 schema 校验错误。

| 键 | 类型 | 默认值 | 说明 |
|----|------|--------|------|
| `autoConnectIde` | boolean | `false` | Claude Code 从外部终端启动时自动连接到运行中的 IDE。在 VS Code 或 JetBrains 终端外运行时，在 `/config` 中显示为 **Auto-connect to IDE (external terminal)** |
| `autoInstallIdeExtension` | boolean | `true` | 从 VS Code 终端运行时自动安装 Claude Code IDE 扩展。在 `/config` 中显示为 **Auto-install IDE extension**。也可通过 `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` 环境变量禁用 |
| `editorMode` | string | `"normal"` | 输入框的按键绑定模式：`"normal"` 或 `"vim"`。在 `/config` 中显示为 **Editor mode** |
| `showTurnDuration` | boolean | `true` | 在响应后显示回合持续时间（例如"Cooked for 1m 6s"）。直接编辑 `~/.claude.json` 修改 |
| `terminalProgressBarEnabled` | boolean | `true` | 在支持的终端中显示终端进度条（ConEmu、Ghostty 1.2.0+ 和 iTerm2 3.6.6+）。在 `/config` 中显示为 **Terminal progress bar** |
| `teammateMode` | string | `"in-process"` | [agent team](https://code.claude.com/docs/en/agent-teams) 队友的显示方式：`"auto"`（在 tmux 或 iTerm2 中选择分屏，否则使用进程内）、`"in-process"` 或 `"tmux"`。参见[选择显示模式](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) |

### 状态栏配置

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 0
  }
}
```

**状态栏输入字段：**

状态栏命令通过 stdin 接收 JSON 对象，包含以下重要字段：

| 字段 | 说明 |
|------|------|
| `workspace.added_dirs` | 通过 `/add-dir` 添加的目录 |
| `context_window.used_percentage` | 上下文窗口使用百分比 |
| `context_window.remaining_percentage` | 上下文窗口剩余百分比 |
| `current_usage` | 当前上下文窗口 token 数量 |
| `exceeds_200k_tokens` | 上下文是否超过 200k tokens |
| `rate_limits.five_hour.used_percentage` | 5 小时速率限制使用百分比（v2.1.80+） |
| `rate_limits.five_hour.resets_at` | 5 小时速率限制重置时间戳 |
| `rate_limits.seven_day.used_percentage` | 7 天速率限制使用百分比 |
| `rate_limits.seven_day.resets_at` | 7 天速率限制重置时间戳 |

### 文件建议配置

文件建议脚本通过 stdin 接收 JSON 对象（例如 `{"query": "src/comp"}`），必须输出最多 15 个文件路径（每行一个）。

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**示例：**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS 与云凭证

### AWS 设置

| 键 | 类型 | 说明 |
|----|------|------|
| `awsAuthRefresh` | string | 刷新 AWS 认证的脚本（修改 `.aws` 目录） |
| `awsCredentialExport` | string | 输出包含 AWS 凭证的 JSON 的脚本 |

**示例：**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| 键 | 类型 | 说明 |
|----|------|------|
| `otelHeadersHelper` | string | 生成动态 OpenTelemetry headers 的脚本 |

**示例：**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## 环境变量（通过 `env`）

为所有 Claude Code 会话设置环境变量。

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### 常用环境变量

| 变量 | 说明 |
|------|------|
| `ANTHROPIC_API_KEY` | 认证用 API key |
| `ANTHROPIC_AUTH_TOKEN` | OAuth token |
| `CLAUDE_CODE_OAUTH_TOKEN` | 用于 Claude.ai 认证的 OAuth access token。在 SDK 和自动化环境中可替代 `/login`。优先于 keychain 存储的凭证 |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | 用于 Claude.ai 认证的 OAuth refresh token。设置后，`claude auth login` 会直接交换此 token 而非打开浏览器。需要 `CLAUDE_CODE_OAUTH_SCOPES` |
| `CLAUDE_CODE_OAUTH_SCOPES` | 以空格分隔的 OAuth scopes，即 refresh token 签发时的作用域（例如 `"user:profile user:inference user:sessions:claude_code"`）。在设置了 `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` 时必填 |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_BEDROCK_BASE_URL` | 覆盖 Bedrock 端点 URL |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | 覆盖 Bedrock Mantle 端点 URL。参见 [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) |
| `ANTHROPIC_VERTEX_BASE_URL` | 覆盖 Vertex AI 端点 URL |
| `ANTHROPIC_BETAS` | 以逗号分隔的 Anthropic beta header 值 |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Vertex AI 的 GCP 项目 ID |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | 在 `/model` 选择器中作为自定义条目添加的模型 ID。用于使非标准或网关特定模型可选，而不替换内置别名 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | `/model` 选择器中自定义模型条目的显示名称。未设置时默认为模型 ID |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | `/model` 选择器中自定义模型条目的显示描述。未设置时默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_MODEL` | 要使用的模型名称。接受别名（`sonnet`、`opus`、`haiku`）或完整模型 ID。覆盖 `model` 设置 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | 用自定义模型 ID 覆盖 Haiku 模型别名（例如用于第三方部署） |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Haiku 条目的标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Haiku 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Haiku 模型的能力检测。以逗号分隔的值（例如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时必填 |
| `CLAUDECODE` | 在 Claude Code 创建的 shell 环境中设为 `1`（Bash tool、tmux 会话）。不在 hooks 或状态栏命令中设置。用于检测脚本是否在 Claude Code shell 内运行 |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | 设为 `1` 允许在组织状态检查因网络错误失败时仍启用 fast mode。适用于企业代理阻止状态端点的情况 |
| `CLAUDE_CODE_USE_BEDROCK` | 使用 AWS Bedrock（`1` 启用） |
| `CLAUDE_CODE_USE_VERTEX` | 使用 Google Vertex AI（`1` 启用） |
| `CLAUDE_CODE_USE_FOUNDRY` | 使用 Microsoft Foundry（`1` 启用） |
| `CLAUDE_CODE_USE_MANTLE` | 使用 Bedrock [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint)（`1` 启用） |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | 设为 `1` 在 Windows 上启用 PowerShell 工具（试用预览）。启用后，Claude 可原生运行 PowerShell 命令而非通过 Git Bash 路由。仅支持原生 Windows，不支持 WSL（v2.1.84） |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 自动生成的 Remote Control 会话名称前缀。默认为机器主机名 |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用/禁用遥测（`0` 或 `1`） |
| `DISABLE_ERROR_REPORTING` | 禁用错误报告（`1` 禁用） |
| `DISABLE_TELEMETRY` | 禁用遥测（`1` 禁用） |
| `MCP_TIMEOUT` | MCP 启动超时时间（毫秒） |
| `MAX_MCP_OUTPUT_TOKENS` | MCP 最大输出 tokens（默认：25000）。输出超过 10,000 tokens 时显示警告 |
| `API_TIMEOUT_MS` | API 请求超时时间（毫秒，默认：600000） |
| `BASH_MAX_TIMEOUT_MS` | Bash 命令超时时间 |
| `BASH_MAX_OUTPUT_LENGTH` | Bash 最大输出长度 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 自动压缩阈值百分比（1-100）。默认约 95%。设较低值（如 `50`）可提前触发压缩。超过 95% 的值无效。使用 `/context` 监控当前使用量。示例：`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 在 bash 调用之间保持工作目录（`1` 启用） |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 禁用后台任务（`1` 禁用） |
| `ENABLE_TOOL_SEARCH` | MCP 工具搜索阈值（例如 `auto:5`） |
| `DISABLE_PROMPT_CACHING` | 禁用所有 prompt caching（`1` 禁用） |
| `DISABLE_PROMPT_CACHING_HAIKU` | 禁用 Haiku prompt caching |
| `DISABLE_PROMPT_CACHING_SONNET` | 禁用 Sonnet prompt caching |
| `DISABLE_PROMPT_CACHING_OPUS` | 禁用 Opus prompt caching |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | 在 Bedrock 上请求 1 小时缓存 TTL（`1` 启用） |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 禁用实验性 beta 功能（`1` 禁用） |
| `CLAUDE_CODE_SHELL` | 覆盖自动 shell 检测 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 覆盖默认文件读取 token 限制 |
| `CLAUDE_CODE_GLOB_HIDDEN` | 设为 `false` 可在 Claude 调用 Glob 工具时排除隐藏文件。默认包含。不影响 `@` 文件自动补全、`ls`、Grep 或 Read |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | 设为 `false` 可让 Glob 工具遵循 `.gitignore` 模式。默认情况下，Glob 返回所有匹配文件（包括 gitignored 的）。不影响有自己 `respectGitignore` 设置的 `@` 文件自动补全 |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 文件发现的超时秒数 |
| `CLAUDE_CODE_ENABLE_TASKS` | 设为 `true` 在非交互模式（`-p` 参数）中启用任务跟踪。在交互模式中默认启用 |
| `CLAUDE_CODE_SIMPLE` | 设为 `1` 使用最小 system prompt 运行，仅提供 Bash、文件读取和文件编辑工具 |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | SDK 模式空闲后自动退出的延迟（毫秒） |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 禁用自适应 thinking（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_THINKING` | 强制禁用 extended thinking（`1` 禁用） |
| `DISABLE_INTERLEAVED_THINKING` | 阻止发送 interleaved-thinking beta header（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 禁用 1M token 上下文窗口（`1` 禁用） |
| `CLAUDE_CODE_ACCOUNT_UUID` | 覆盖认证用的账户 UUID |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | 禁用 git 相关的 system prompt 指令 |
| `CLAUDE_CODE_NEW_INIT` | 设为 `true` 让 `/init` 运行交互式设置流程。在探索代码库前先询问要生成哪些文件（CLAUDE.md、skills、hooks）。不设此项时，`/init` 自动生成 CLAUDE.md |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 一个或多个只读 plugin seed 目录的路径，Unix 上用 `:` 分隔，Windows 上用 `;` 分隔。将预配置的 plugins 打包到容器镜像中。Claude Code 在启动时从这些目录注册 marketplaces，并使用预缓存的 plugins 而无需重新克隆 |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | 启用 Claude.ai MCP servers |
| `CLAUDE_CODE_EFFORT_LEVEL` | 设置 effort level：`low`、`medium`、`high`、`max`（仅 Opus 4.6）或 `auto`（使用模型默认值）。优先于 `/effort` 和 `effortLevel` 设置 |
| `CLAUDE_CODE_MAX_TURNS` | 停止前的最大 agentic 回合数 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 等同于同时设置 `DISABLE_AUTOUPDATER`、`DISABLE_FEEDBACK_COMMAND`、`DISABLE_ERROR_REPORTING` 和 `DISABLE_TELEMETRY` |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | 跳过首次运行的设置引导流程 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | 覆盖 prompt caching 行为 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_DISABLE_TOOLS` | 以逗号分隔的要禁用的工具列表 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_DISABLE_MCP` | 禁用所有 MCP servers（`1` 禁用） *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 每次响应的最大输出 tokens。默认：32,000（Opus 4.6 自 v2.1.77 起为 64,000）。上限：64,000（Opus 4.6 和 Sonnet 4.6 自 v2.1.77 起为 128,000） |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | 完全禁用 fast mode（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 设为 `1` 禁用流式请求中途失败时的非流式回退。流式错误将传播到重试层。适用于代理或网关导致回退产生重复工具执行的情况（v2.1.83） |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | 中止停滞的流（`1` 启用） |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | 启用细粒度工具流（`1` 启用） |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 禁用自动记忆（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | 禁用 `/rewind` 的文件检查点（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 禁用附件处理（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 阻止加载 CLAUDE.md 文件（`1` 禁用） |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | 如果上次会话在回合中途结束则自动恢复（`1` 启用） |
| `CLAUDE_CODE_USER_EMAIL` | 同步提供用于认证的用户邮箱 |
| `CLAUDE_CODE_ORGANIZATION_UUID` | 同步提供用于认证的组织 UUID |
| `CLAUDE_CONFIG_DIR` | 自定义配置目录（覆盖默认的 `~/.claude`） |
| `CLAUDE_CODE_TMPDIR` | 覆盖内部临时文件使用的临时目录。Claude Code 会在此路径后追加 `/claude/`。默认：Unix/macOS 为 `/tmp`，Windows 为 `os.tmpdir()` |
| `ANTHROPIC_CUSTOM_HEADERS` | API 请求的自定义 headers（`Name: Value` 格式，多个 headers 用换行分隔） |
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry 认证用 API key |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 资源的基础 URL |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry 资源名称 |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock 认证用 API key |
| `ANTHROPIC_SMALL_FAST_MODEL` | **已废弃** — 请改用 `ANTHROPIC_DEFAULT_HAIKU_MODEL` |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | 已废弃的 Haiku 级模型覆盖的 AWS 区域 |
| `CLAUDE_CODE_SHELL_PREFIX` | 添加到 bash 命令前的命令前缀 |
| `BASH_DEFAULT_TIMEOUT_MS` | Bash 命令默认超时时间（毫秒） |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | 跳过 Bedrock 的 AWS 认证（`1` 跳过） |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | 跳过 Foundry 的 Azure 认证（`1` 跳过） |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | 跳过 Bedrock Mantle 的 AWS 认证（例如使用 LLM 网关时） |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | 跳过 Vertex 的 Google 认证（`1` 跳过） |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 允许代理执行 DNS 解析 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` 的凭证刷新间隔（毫秒） |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS 客户端证书路径 |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS 客户端私钥路径 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 加密的 mTLS 密钥的密码短语 |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Plugin marketplace git clone 超时时间（毫秒，默认：120000） |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 覆盖 plugins 根目录 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 跳过自动添加官方 marketplace（`1` 禁用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | 在首次查询前等待 plugin 安装完成（`1` 启用） |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 同步 plugin 安装的超时时间（毫秒） |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | 设为 `1` 在 `git pull` 失败时保留现有 marketplace 缓存而非清除并重新克隆。适用于离线或隔离环境，因为重新克隆也会以相同方式失败 |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | 从 UI 隐藏邮箱/组织信息 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_DISABLE_CRON` | 禁用计划/cron 任务（`1` 禁用） |
| `DISABLE_INSTALLATION_CHECKS` | 禁用安装警告 |
| `DISABLE_FEEDBACK_COMMAND` | 禁用 `/feedback` 命令。旧名称 `DISABLE_BUG_COMMAND` 也可使用 |
| `DISABLE_DOCTOR_COMMAND` | 隐藏 `/doctor` 命令（`1` 禁用） |
| `DISABLE_LOGIN_COMMAND` | 隐藏 `/login` 命令（`1` 禁用） |
| `DISABLE_LOGOUT_COMMAND` | 隐藏 `/logout` 命令（`1` 禁用） |
| `DISABLE_UPGRADE_COMMAND` | 隐藏 `/upgrade` 命令（`1` 禁用） |
| `DISABLE_EXTRA_USAGE_COMMAND` | 隐藏 `/extra-usage` 命令（`1` 禁用） |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | 隐藏 `/install-github-app` 命令（`1` 禁用） |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | 禁用修饰性文字和非必要模型调用 *（不在官方文档中 — 未验证）* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 覆盖调试日志文件目录路径 |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | 最低调试日志级别 |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 强制自动将长任务转为后台（`1` 启用） |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | 阻止将 Opus 4.0/4.1 重映射到新版模型（`1` 禁用） |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 对所有主模型触发回退模型，而非仅对默认模型（`1` 启用） |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows Git Bash 可执行文件路径（仅启动时） |
| `DISABLE_COST_WARNINGS` | 禁用费用警告消息 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 覆盖 subagents 使用的模型（例如 `haiku`、`sonnet`） |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 设为 `1` 从子进程环境中清除 Anthropic 和云提供商凭证（Bash tool、hooks、MCP stdio servers）。用于纵深防御，当子进程不应继承 API keys 时使用（v2.1.83） |
| `CLAUDE_CODE_MAX_RETRIES` | 覆盖 API 请求重试次数（默认：10） |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 最大并行只读工具数（默认：10） |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | 在 SDK 模式中禁用内置 subagent 类型（`1` 禁用） |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | 在 SDK 模式中跳过 MCP 工具的 `mcp__<server>__` 前缀（`1` 启用） |
| `MCP_CONNECTION_NONBLOCKING` | 在 `-p` 模式中设为 `true` 可完全跳过 MCP 连接等待。将 `--mcp-config` 服务器连接限制在 5 秒而非等待最慢的服务器 *（在 v2.1.89 changelog 中，尚未出现在官方 env-vars 页面）* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook 超时时间（毫秒，替代硬编码的 1.5 秒限制） |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 禁用反馈调查提示（`1` 禁用） |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 禁用终端标题更新（`1` 禁用） |
| `CLAUDE_CODE_NO_FLICKER` | 设为 `1` 启用无闪烁的 alt-screen 渲染。消除全屏重绘时的视觉闪烁（v2.1.88） |
| `CLAUDE_CODE_SCROLL_SPEED` | 全屏渲染的鼠标滚轮速度倍数。增大可加快滚动，减小可更精细控制 |
| `CLAUDE_CODE_DISABLE_MOUSE` | 设为 `1` 禁用全屏渲染中的鼠标跟踪。适用于鼠标事件干扰终端复用器或辅助功能工具的情况 |
| `CLAUDE_CODE_ACCESSIBILITY` | 设为 `1` 保持原生终端光标可见，用于屏幕阅读器和辅助功能工具 |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | 设为 `0` 禁用 diff 输出中的语法高亮 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 跳过自动 IDE 扩展安装（`1` 跳过） |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 覆盖 IDE 自动连接行为 |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 覆盖 IDE 连接主机地址 |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | 跳过 IDE lockfile 验证（`1` 跳过） |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | OTel headers helper 脚本的防抖间隔（毫秒） |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | OpenTelemetry flush 超时时间（毫秒） |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | OpenTelemetry 关闭超时时间（毫秒） |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | 流式空闲看门狗关闭停滞连接前的超时时间（毫秒）。默认：`90000`（90 秒）。如果长时间运行的工具或慢速网络导致过早超时错误，请增大此值 |
| `OTEL_LOG_TOOL_DETAILS` | 设为 `1` 在 OpenTelemetry 事件中包含 `tool_parameters`。默认出于隐私原因省略 *（在 v2.1.85 changelog 中，尚未出现在官方 env-vars 页面）* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP server 名称，作为环境变量传递给 `headersHelper` 脚本，以便生成服务器特定的认证 headers *（在 v2.1.85 changelog 中，尚未出现在官方 env-vars 页面）* |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP server URL，与 `CLAUDE_CODE_MCP_SERVER_NAME` 一起作为环境变量传递给 `headersHelper` 脚本 *（在 v2.1.85 changelog 中，尚未出现在官方 env-vars 页面）* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | 覆盖 Opus 模型别名（例如 `claude-opus-4-6[1m]`） |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Opus 条目的标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Opus 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Opus 模型的能力检测。以逗号分隔的值（例如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时必填 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | 覆盖 Sonnet 模型别名（例如 `claude-sonnet-4-6`） |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | 在 Bedrock/Vertex/Foundry 上使用固定模型时，自定义 `/model` 选择器中 Sonnet 条目的标签。默认为模型 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | 自定义 `/model` 选择器中 Sonnet 条目的描述。默认为 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | 覆盖固定 Sonnet 模型的能力检测。以逗号分隔的值（例如 `effort,thinking`）。当固定模型支持自动检测无法确认的功能时必填 |
| `MAX_THINKING_TOKENS` | 每次响应的最大 extended thinking tokens |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 设置用于自动压缩计算的上下文容量（tokens）。默认为模型的上下文窗口（标准 200K，扩展上下文模型为 1M）。在 1M 模型上使用较低值（如 `500000`）将其视为 500K 进行压缩。上限为实际上下文窗口大小。`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 作为此值的百分比应用。设置此项可将压缩阈值与状态栏的 `used_percentage` 解耦 |
| `DISABLE_AUTO_COMPACT` | 禁用自动上下文压缩（`1` 禁用）。手动 `/compact` 仍可使用 |
| `DISABLE_COMPACT` | 禁用所有压缩 — 自动和手动均禁用（`1` 禁用） |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 启用提示建议 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | 要求会话使用 plan 模式 |
| `CLAUDE_CODE_TEAM_NAME` | Agent teams 的团队名称 |
| `CLAUDE_CODE_TASK_LIST_ID` | 任务集成的任务列表 ID |
| `CLAUDE_ENV_FILE` | 自定义环境文件路径 |
| `FORCE_AUTOUPDATE_PLUGINS` | 强制 plugin 自动更新（`1` 启用） |
| `HTTP_PROXY` | 网络请求的 HTTP 代理 URL |
| `HTTPS_PROXY` | 网络请求的 HTTPS 代理 URL |
| `NO_PROXY` | 以逗号分隔的绕过代理的主机列表 |
| `MCP_TOOL_TIMEOUT` | MCP 工具执行超时时间（毫秒） |
| `MCP_CLIENT_SECRET` | MCP OAuth client secret |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth 回调端口 |
| `IS_DEMO` | 启用演示模式 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Slash command 工具输出的字符预算 |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet 的 Vertex AI 区域覆盖 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus 的 Vertex AI 区域覆盖 |

---

## 常用命令

| 命令 | 说明 |
|------|------|
| `/model` | 切换模型并调整 Opus 4.6 effort level |
| `/effort` | 直接设置 effort level：`low`、`medium`、`high`（v2.1.76+） |
| `/config` | 交互式配置 UI |
| `/memory` | 查看/编辑所有 memory 文件 |
| `/agents` | 管理 subagents |
| `/mcp` | 管理 MCP servers |
| `/hooks` | 查看已配置的 hooks |
| `/plugin` | 管理 plugins |
| `/keybindings` | 配置自定义键盘快捷键 |
| `/skills` | 查看和管理 skills |
| `/permissions` | 查看和管理权限规则 |
| `--doctor` | 诊断配置问题 |
| `--debug` | 调试模式，显示 hook 执行详情 |

---

## 快速参考：完整示例

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "effortLevel": "medium",

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  }
}
```

---

## 参考来源

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Shipyard - Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Claude Code Environment Variables Reference](https://code.claude.com/docs/en/env-vars)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)
