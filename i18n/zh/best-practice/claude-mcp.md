🌍 [English](../../../best-practice/claude-mcp.md) | **中文** | [日本語](../../ja/best-practice/claude-mcp.md) | [Français](../../fr/best-practice/claude-mcp.md) | [Русский](../../ru/best-practice/claude-mcp.md) | [العربية](../../ar/best-practice/claude-mcp.md)

# MCP Servers 最佳实践

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.mcp.json)

MCP（Model Context Protocol）servers 通过连接外部工具、数据库和 API 来扩展 Claude Code。本指南涵盖日常使用的推荐 servers 和配置最佳实践。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 日常使用的 MCP Servers

> *"装了 15 个 MCP servers 以为越多越好。结果每天只用 4 个。"* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)（682 赞）

| MCP Server | 功能 | 资源 |
|------------|------|------|
| [**Context7**](https://github.com/upstash/context7) | 将最新的库文档拉入上下文。防止因过时训练数据导致的 API 幻觉 | [Reddit: "迄今为止最好的编码 MCP"](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | 浏览器自动化 — 自主实现、测试和验证 UI 功能。截图、导航、表单测试 | [Reddit: 前端必备](https://reddit.com/r/mcp/comments/1m59pk0/) · [文档](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | 将 Claude 连接到你的真实 Chrome 浏览器 — 检查控制台、网络、DOM。调试用户实际看到的内容 | [Reddit: 调试的 "game changer"](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [对比报告](../../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | 获取任何 GitHub 仓库的结构化 wiki 风格文档 — 架构、API 接口、关系 | [Reddit: "配合 Context7 放在网关后面"](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | 从 prompt 生成架构图、流程图和系统设计，以手绘风格的 Excalidraw 草图呈现 | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

调研（Context7/DeepWiki）-> 调试（Playwright/Chrome）-> 文档化（Excalidraw）

---

## 配置

MCP servers 在项目根目录的 `.mcp.json`（项目范围）或 `~/.claude.json`（用户范围）中配置。

### Server 类型

| 类型 | 传输方式 | 示例 |
|------|----------|------|
| **stdio** | 启动一个本地进程 | `npx`、`python`、二进制文件 |
| **http** | 连接到远程 URL | HTTP/SSE 端点 |

### `.mcp.json` 示例

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

使用环境变量展开来处理密钥，而非在 `.mcp.json` 中提交 API keys：

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### MCP Servers 的设置

以下 `.claude/settings.json` 中的设置控制 MCP server 的审批：

| 键 | 类型 | 说明 |
|----|------|------|
| `enableAllProjectMcpServers` | boolean | 自动批准所有 `.mcp.json` servers 而无需提示 |
| `enabledMcpjsonServers` | array | 特定 server 名称的白名单，自动批准 |
| `disabledMcpjsonServers` | array | 特定 server 名称的黑名单，自动拒绝 |

### MCP 工具的权限规则

MCP 工具在权限规则中遵循 `mcp__<server>__<tool>` 命名约定：

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## MCP 作用域

MCP servers 可以在三个层级定义：

| 作用域 | 位置 | 用途 |
|--------|------|------|
| **项目** | `.mcp.json`（仓库根目录） | 团队共享的 servers，提交到 git |
| **用户** | `~/.claude.json`（`mcpServers` 键） | 跨所有项目的个人 servers |
| **Subagent** | Agent frontmatter（`mcpServers` 字段） | 范围限定于特定 subagent 的 servers |

优先级：Subagent > 项目 > 用户

---

## 来源

- [MCP Servers — Claude Code 文档](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol 规范](https://modelcontextprotocol.io/)
- [5 个真正让我效率提升 10 倍的 MCP — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [MCP Server 过载讨论 — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
