🌍 [English](../../../best-practice/claude-troubleshooting.md) | **中文** | [日本語](../../ja/best-practice/claude-troubleshooting.md) | [Français](../../fr/best-practice/claude-troubleshooting.md) | [Русский](../../ru/best-practice/claude-troubleshooting.md) | [العربية](../../ar/best-practice/claude-troubleshooting.md)

# 故障排查指南

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code 常见问题的诊断指南 — 按症状分类，附逐步解决方案。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 快速诊断

遇到问题时先运行以下命令：

| 命令 | 检查内容 |
|------|---------|
| `claude --version` | 已安装版本 — 与[最新版本](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)对比 |
| `/doctor` | 安装健康状况、Node.js 版本、认证、设置有效性 |
| `/status` | 当前模型、账户、API 连接、订阅等级 |
| `/usage` | 套餐限额、速率限制状态、剩余配额 |
| `/context` | 上下文窗口使用量、已加载的记忆文件、活跃的 Skill |
| `/hooks` | 活跃的 Hook 配置及其来源 |
| `/mcp` | MCP Server 连接状态 |

---

## 安装问题

### Claude Code 安装失败

**症状**: `npm install -g @anthropic-ai/claude-code` 因权限错误或版本冲突而失败。

**原因**: Node.js 版本过旧、npm 权限问题或全局包冲突。

**解决方案**:

```bash
# 检查 Node.js 版本（需要 18+）
node --version

# 如果版本过旧，通过 nvm 更新
nvm install 20
nvm use 20

# 修复 npm 权限 (macOS/Linux)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH  # 添加到 ~/.zshrc 或 ~/.bashrc

# 全新安装
npm install -g @anthropic-ai/claude-code
```

### 找不到 claude 命令

**症状**: 安装成功后终端提示 `claude: command not found`。

**原因**: npm 全局 bin 目录不在 PATH 中。

**解决方案**:

```bash
# 查找 npm 安装全局二进制文件的位置
npm config get prefix

# 添加到 shell 配置文件 (~/.zshrc, ~/.bashrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# 重新加载 shell
source ~/.zshrc
```

---

## 认证问题

### API Key 无法识别

**症状**: Claude Code 启动但无法调用 API。错误消息提示认证或 key 无效。

**原因**: API key 缺失或已过期。环境变量不正确。

**解决方案**:

```bash
# 重新认证
claude /login

# 或直接设置 API key
export ANTHROPIC_API_KEY="sk-ant-..."

# Bedrock 用户
export CLAUDE_CODE_USE_BEDROCK=1
claude /setup-bedrock
```

### 速率限制错误

**症状**: `429 Too Many Requests` 错误。Claude 在任务中途停止响应。

**原因**: 超出套餐速率限制。并发会话或 Agent 过多。

**解决方案**:

```bash
# 检查当前使用量
/usage

# 选项：
# 1. 等待速率限制重置（检查 Retry-After 头）
# 2. 对高频任务切换到更轻量的模型
/model haiku

# 3. 减少并发 Agent
# 4. 启用额外用量
/extra-usage
```

---

## 性能问题

### 响应缓慢

**症状**: Claude 回复需要 30 秒以上。工具调用之间有长时间停顿。

**原因**: 上下文窗口过大。加载了太多 Skill。同步 Hook 阻塞了响应。

**解决方案**:

```bash
# 检查上下文使用量
/context

# 超过 50% 时进行压缩
/compact

# 检查是否有缓慢的同步 Hook
/hooks
# 将非关键 Hook 设为异步: "async": true

# 减少已加载的 Skill — 检查是否有不必要的 Skill 被自动激活
/skills
```

### Token 消耗过高

**症状**: 配额消耗速度超出预期。`/usage` 显示高消耗量。

**原因**: CLAUDE.md 文件过大。预加载了很多 Skill。Agent 调用时继承了完整上下文。没有进行压缩。

**解决方案**:

1. CLAUDE.md 控制在 200 行以内
2. 对只应被特定 Agent 预加载的 Skill 使用 `user-invocable: false`
3. 对不需要完整能力的 Agent 设置 `model: haiku`
4. 使用 `/compact` 定期压缩
5. 对常规任务使用 `effort: low` 或 `effort: medium`

---

## Hook 问题

### Hook 不触发

**症状**: 在设置中配置了 Hook 但它从不执行。

**原因**: 写在了错误的设置文件中。matcher 缺失或不匹配。Hook 被禁用。设置层级覆盖。

**解决方案**:

```bash
# 1. 检查哪些 Hook 处于活跃状态及其来源
/hooks

# 2. 确认 Hook 在正确的设置文件中
# 团队 Hook: .claude/settings.json
# 个人 Hook: .claude/settings.local.json
# 全局 Hook: ~/.claude/settings.json

# 3. 检查 Hook 是否被全局禁用
# 查看任一设置文件中是否有 "disableAllHooks": true

# 4. 确认 matcher 与工具名称完全匹配
# "Bash" 而非 "bash"，"Write" 而非 "write"

# 5. 检查设置层级 — 更高优先级的文件可能会覆盖
```

### Hook 阻塞响应

**症状**: 工具调用后出现长时间停顿。Claude 看起来像冻住了几秒。

**原因**: 同步 Hook 运行了一个缓慢的命令。非异步 Hook 中有网络调用。

**解决方案**:

```json
// 对非阻塞 Hook 添加 "async": true
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "python3 log.py",
        "async": true
      }
    ]
  }
}
```

### Hook 超时

**症状**: Hook 启动了但结果从未被使用。日志中出现 Hook 超时警告。

**原因**: Hook 命令耗时太长。外部服务不可用。脚本挂起。

**解决方案**: 手动测试 Hook 命令。在脚本本身中添加超时。确保外部服务可达。如果结果不是关键性的，考虑将 Hook 改为异步。

---

## MCP 问题

### MCP Server 无法连接

**症状**: MCP 工具不可用。`/mcp` 显示 Server 已断开。

**原因**: 找不到 Server 二进制文件。配置错误。端口冲突。缺少环境变量。

**解决方案**:

```bash
# 1. 检查 MCP 状态
/mcp

# 2. 验证 .mcp.json 或 settings.json 中的 Server 配置
# 确保命令路径正确且可执行

# 3. 检查所需的环境变量是否已设置
# MCP Server 通常需要 .env 中的 API key

# 4. 尝试重启 MCP 连接
# 退出并重新启动 Claude Code

# 5. 手动测试 MCP Server
npx @modelcontextprotocol/server-name
```

### MCP 权限被拒绝

**症状**: MCP 工具出现了但调用时报权限错误。

**原因**: 工具不在允许列表中。权限模式过于严格。

**解决方案**: 在设置中将 MCP 工具添加到 `allowedTools` 列表，或在提示时批准。对于 Agent，在 `tools:` 字段中包含 `mcp__<server>__<tool>`。

---

## Agent 问题

### Agent 不自动调用

**症状**: 你创建了一个 description 中包含 `PROACTIVELY` 的 Agent，但 Claude 从不自动启动它。

**原因**: description 没有清楚描述何时调用。Agent 工具在当前上下文中不可用。其他机制已在处理该任务。

**解决方案**:

1. 确保 `description` 字段使用了 `PROACTIVELY` 并清楚说明触发条件
2. 检查 Agent 文件是否在 `.claude/agents/` 中，扩展名为 `.md`
3. 确认 Claude 有权使用 `Agent` 工具（未被父 Agent 的 `tools:` 字段限制）

### Agent 通过 Bash 而非 Agent 工具调用

**症状**: Claude 通过 Bash 运行 `claude --agent my-agent` 而不是使用 `Agent(...)` 工具。

**原因**: Agent 定义中使用了模糊的语言，如 "launch" 或 "run"，Claude 将其解读为 shell 命令。

**解决方案**: 在 CLAUDE.md 或 Agent 指令中明确说明："Use the Agent tool, not bash commands, to spawn subagents。" 避免在 Agent description 中使用 "launch" 或 "execute" 等术语。

### Agent 超出 maxTurns

**症状**: Agent 在任务中途停止，结果不完整。

**原因**: `maxTurns` 对任务复杂度来说设置得太低。Agent 正在循环或重试。

**解决方案**: 在 Agent frontmatter 中增大 `maxTurns`。检查 Agent 行为是否存在不必要的循环。将复杂任务拆分为更小的子任务。

---

## Skill 问题

### Skill 没有出现在 / 菜单中

**症状**: 你创建了一个 Skill 但输入 `/` 时看不到它。

**原因**: 设置了 `user-invocable: false`。Skill 文件不在正确的位置。YAML frontmatter 语法错误。

**解决方案**:

```bash
# 1. 检查 Skill 列表
/skills

# 2. 确认文件位置: .claude/skills/<name>/SKILL.md

# 3. 检查 frontmatter — 移除 user-invocable: false
# 或设置 user-invocable: true

# 4. 验证 YAML — 常见错误：
# - 缺少 --- 分隔符
# - 缩进不正确
# - 未加引号的特殊字符
```

### Skill 不被自动发现

**症状**: Skill 存在且有 description，但 Claude 从不自动调用。

**原因**: 设置了 `disable-model-invocation: true`。description 与 Claude 遇到的上下文不匹配。太多 Skill 争夺同一个触发条件。

**解决方案**:

1. 如果需要自动发现，移除 `disable-model-invocation: true`
2. 编写清晰的 `description`，使用 Claude 自然会遇到的关键词
3. 减少 Skill 数量 — 太多可自动发现的 Skill 会造成歧义

---

## 记忆问题

### CLAUDE.md 未加载

**症状**: CLAUDE.md 中的指令被忽略。Claude 不遵循项目规范。

**原因**: 文件在子目录中（懒加载，非即时加载）。文件名有拼写错误。通过 Managed 策略覆盖了设置。

**解决方案**:

```bash
# 1. 检查已加载的记忆文件
/context

# 2. 确认文件名准确为 "CLAUDE.md"（区分大小写）

# 3. 理解加载规则：
# 祖先目录（从 cwd 向上）：启动时即时加载
# 后代目录（从 cwd 向下）：访问该目录中的文件时懒加载
# 兄弟目录：永远不加载

# 4. 如果需要始终加载，将其移至祖先目录
# 或从根 CLAUDE.md 中引用它
```

### CLAUDE.local.md 被提交到版本库

**症状**: 个人偏好出现在队友那里。`.claude/settings.local.json` 出现在 git 历史中。

**原因**: 本地文件未加入 `.gitignore`。

**解决方案**:

```bash
# 添加到 .gitignore
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

# 如果已被提交，从跟踪中移除
git rm --cached CLAUDE.local.md
git rm --cached .claude/settings.local.json
```

---

## Git 问题

### Worktree 清理

**症状**: Agent 执行后遗留了过期的 git worktree。磁盘空间持续增长。

**原因**: 使用 `isolation: worktree` 的 Agent 在清理前崩溃或被中断。

**解决方案**:

```bash
# 列出 worktree
git worktree list

# 移除过期的 worktree
git worktree prune

# 强制移除特定 worktree
git worktree remove /path/to/worktree --force
```

### 提交作者

**症状**: Claude Code 创建的 git 提交归属到了错误的作者。

**原因**: Claude Code 会话中未配置 git 作者。

**解决方案**:

```bash
# 为项目设置 git 作者
git config user.name "Your Name"
git config user.email "your@email.com"

# 或在 Hook/脚本中使用 --author 参数
git commit --author="Your Name <your@email.com>" -m "message"
```

---

## 参考资料

- [Claude Code Troubleshooting — Docs](https://code.claude.com/docs/en/troubleshooting)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP — Docs](https://code.claude.com/docs/en/mcp)
- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
