# MCP Servers Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Best Practice](https://img.shields.io/badge/Best_Practice-blue?style=flat)](../best-practice/claude-mcp.md)

Practical MCP server setup — recommended servers, project and agent configuration, permission rules, and complete working examples.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## What MCP Is

MCP (Model Context Protocol) is a standard for connecting AI models to external tools, data sources, and services. In Claude Code, MCP servers extend the available tool set — each server exposes one or more tools that Claude can call during a session. Tools from MCP servers follow the naming convention `mcp__<server>__<tool>`.

---

## Recommended MCP Servers

Five servers that cover the most common daily workflows:

| Server | Category | What It Does |
|--------|----------|-------------|
| [**Context7**](https://github.com/upstash/context7) | Research | Fetches up-to-date library documentation into context. Prevents hallucinated APIs from outdated training data |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | Testing | Browser automation — navigate pages, fill forms, take screenshots, verify UI. Works headless or with a visible browser |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | Debugging | Connects Claude to your real Chrome browser — inspect console, network tab, DOM. Debug what users actually see |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | Research | Fetches structured wiki-style documentation for any GitHub repo — architecture, API surface, relationships |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | Documentation | Generates architecture diagrams, flowcharts, and system designs as hand-drawn Excalidraw sketches |

---

## Configuration in .mcp.json (Project-level)

Create `.mcp.json` at the project root. This file is committed to git and shared with the team.

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
    "claude-in-chrome": {
      "command": "npx",
      "args": ["-y", "@anthropic/claude-in-chrome-mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "excalidraw": {
      "command": "npx",
      "args": ["-y", "excalidraw-mcp"]
    }
  }
}
```

### Remote (HTTP) servers

For servers running as a remote service instead of a local process:

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

### Environment variable expansion

Use `${VAR_NAME}` syntax to reference secrets without committing them:

```json
{
  "mcpServers": {
    "authenticated-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

---

## Configuration in Agent Frontmatter

Scope MCP servers to a specific subagent using the `mcpServers` field. The agent will have access to these servers in addition to any project-level servers.

### Reference existing project servers by name

```yaml
---
name: research-agent
description: Researches codebases and documentation
tools: Read, Glob, Grep, WebFetch
model: haiku
mcpServers:
  - context7
  - deepwiki
---
```

### Inline server configuration

Define a server directly in the agent frontmatter when the server is only needed by this agent:

```yaml
---
name: browser-test-agent
description: Runs browser-based UI tests
tools: Read, Write, Bash
model: sonnet
mcpServers:
  - playwright:
      command: npx
      args: ["-y", "@playwright/mcp", "--headless"]
---
```

---

## Permission Rules for MCP Tools

MCP tools follow the `mcp__<server>__<tool>` naming convention in permission rules. Configure in `.claude/settings.json`:

### Allow all tools from all MCP servers

```json
{
  "permissions": {
    "allow": ["mcp__*"]
  }
}
```

### Allow all tools from a specific server

```json
{
  "permissions": {
    "allow": ["mcp__playwright__*"]
  }
}
```

### Allow specific tools only

```json
{
  "permissions": {
    "allow": [
      "mcp__context7__resolve-library-id",
      "mcp__context7__get-library-docs",
      "mcp__playwright__browser_snapshot",
      "mcp__playwright__browser_navigate"
    ]
  }
}
```

### Deny a dangerous server

```json
{
  "permissions": {
    "deny": ["mcp__untrusted-server__*"]
  }
}
```

### Auto-approve MCP servers without prompting

Add to `.claude/settings.json`:

```json
{
  "enableAllProjectMcpServers": true
}
```

Or approve specific servers only:

```json
{
  "enabledMcpjsonServers": ["context7", "playwright"]
}
```

---

## Complete Example: Setting Up Playwright MCP for Browser Testing

### Step 1: Add to .mcp.json

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

### Step 2: Auto-approve in settings.json

```json
{
  "permissions": {
    "allow": ["mcp__playwright__*"]
  },
  "enabledMcpjsonServers": ["playwright"]
}
```

### Step 3: Use in a conversation

```
> Navigate to http://localhost:3000 and take a screenshot of the homepage
```

Claude will use the Playwright MCP tools (`browser_navigate`, `browser_snapshot`) to open the page, capture a screenshot, and report what it sees.

### Step 4: Scope to a testing agent (optional)

```yaml
---
name: ui-test-agent
description: PROACTIVELY use this agent to verify UI features in the browser
tools: Read, Write, Bash
model: sonnet
mcpServers:
  - playwright
hooks:
  Stop:
    - hooks:
        - type: command
          command: "npx playwright close 2>/dev/null || true"
          timeout: 5000
          async: true
---

# UI Test Agent

You verify UI features by navigating to pages, interacting with elements,
and taking screenshots to confirm visual correctness.
```

---

## Complete Example: Setting Up Context7 for Documentation Lookup

### Step 1: Add to .mcp.json

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### Step 2: Auto-approve in settings.json

```json
{
  "permissions": {
    "allow": [
      "mcp__context7__resolve-library-id",
      "mcp__context7__get-library-docs"
    ]
  },
  "enabledMcpjsonServers": ["context7"]
}
```

### Step 3: Use in a conversation

```
> How do I use the App Router in Next.js 15? Use context7 to get the latest docs.
```

Claude calls `mcp__context7__resolve-library-id` to find the Next.js library, then `mcp__context7__get-library-docs` to fetch the current App Router documentation. This prevents Claude from relying on potentially outdated training data.

### Step 4: Scope to a research agent (optional)

```yaml
---
name: docs-researcher
description: Researches library documentation using Context7
tools: Read, Glob, Grep, WebFetch, WebSearch
model: haiku
mcpServers:
  - context7
  - deepwiki
---

# Documentation Researcher

You look up current library documentation using Context7 and DeepWiki
before answering technical questions. Always verify API signatures against
the latest docs rather than relying on training data.
```

---

## Scoping MCP Servers to Specific Agents

MCP servers can be defined at three levels, with clear precedence:

| Scope | Location | When to Use |
|-------|----------|-------------|
| **Project** | `.mcp.json` (repo root) | Servers the whole team needs — committed to git |
| **User** | `~/.claude.json` (`mcpServers` key) | Personal servers across all projects (e.g., your Excalidraw instance) |
| **Subagent** | Agent frontmatter (`mcpServers` field) | Servers needed only by a specific agent |

**Precedence**: Subagent > Project > User. If an agent defines a server with the same name as a project-level server, the agent's configuration wins.

### Why scope matters

- **Security**: A research agent should not have access to a database MCP server
- **Performance**: Fewer MCP servers means faster tool discovery and less context overhead
- **Clarity**: Each agent's capabilities are explicit in its frontmatter

---

## Sources

- [MCP Servers — Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [MCP Servers Best Practice](../best-practice/claude-mcp.md)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Playwright MCP — GitHub](https://github.com/microsoft/playwright-mcp)
- [Context7 MCP — GitHub](https://github.com/upstash/context7)
