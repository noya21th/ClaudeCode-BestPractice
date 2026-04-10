🌍 [English](../../../best-practice/claude-skills.md) | **中文** | [日本語](../../ja/best-practice/claude-skills.md) | [Français](../../fr/best-practice/claude-skills.md) | [Русский](../../ru/best-practice/claude-skills.md) | [العربية](../../ar/best-practice/claude-skills.md)

# Skills 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A33%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-skills-implementation.md)

Claude Code skills — frontmatter 字段和官方内置 skills。

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
| `description` | string | 推荐 | skill 的功能。在自动补全中显示并被 Claude 用于自动发现 |
| `argument-hint` | string | 否 | 自动补全时显示的提示（如 `[issue-number]`、`[filename]`） |
| `disable-model-invocation` | boolean | 否 | 设为 `true` 防止 Claude 自动调用此 skill |
| `user-invocable` | boolean | 否 | 设为 `false` 从 `/` 菜单中隐藏 — skill 变为仅后台知识，供 agent 预加载使用 |
| `allowed-tools` | string | 否 | 此 skill 激活时无需权限提示即可使用的工具 |
| `model` | string | 否 | 此 skill 运行时使用的模型（如 `haiku`、`sonnet`、`opus`） |
| `effort` | string | 否 | 调用时覆盖模型的 effort level（`low`、`medium`、`high`、`max`） |
| `context` | string | 否 | 设为 `fork` 在隔离的 subagent 上下文中运行 skill |
| `agent` | string | 否 | 设置 `context: fork` 时的 subagent 类型（默认：`general-purpose`） |
| `hooks` | object | 否 | 范围限定于此 skill 的生命周期 hooks |
| `paths` | string/list | 否 | 限制 skill 自动激活条件的 Glob 模式。接受逗号分隔字符串或 YAML 列表 — 仅当处理匹配文件时 Claude 才加载此 skill |
| `shell` | string | 否 | `` !`command` `` 块使用的 shell — `bash`（默认）或 `powershell`。需要 `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Skill | 说明 |
|---|-------|------|
| 1 | `simplify` | 审查修改的代码以提升复用性、质量和效率 — 重构以消除重复 |
| 2 | `batch` | 跨多个文件批量运行命令 |
| 3 | `debug` | 调试失败的命令或代码问题 |
| 4 | `loop` | 按循环间隔运行 prompt 或 slash command（最长 3 天） |
| 5 | `claude-api` | 使用 Claude API 或 Anthropic SDK 构建应用 — 在 `anthropic` / `@anthropic-ai/sdk` 导入时触发 |

另请参阅：[官方 Skills 仓库](https://github.com/anthropics/skills/tree/main/skills)，获取社区维护的可安装 skills。

---

## 来源

- [Claude Code Skills — 文档](https://code.claude.com/docs/en/skills)
- [Monorepo 中的 Skills 发现](../../../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
