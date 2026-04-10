# Troubleshooting Guide

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Diagnostic guide for common Claude Code issues — organized by symptom with step-by-step solutions.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Quick Diagnostics

Run these commands first when something is not working:

| Command | What It Checks |
|---------|---------------|
| `claude --version` | Installed version — compare against [latest release](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) |
| `/doctor` | Installation health, Node.js version, authentication, settings validity |
| `/status` | Current model, account, API connectivity, subscription tier |
| `/usage` | Plan limits, rate limit status, remaining quota |
| `/context` | Context window usage, loaded memory files, active skills |
| `/hooks` | Active hook configurations and their sources |
| `/mcp` | MCP server connection status |

---

## Installation Issues

### Claude Code Fails to Install

**Symptoms**: `npm install -g @anthropic-ai/claude-code` fails with permission errors or version conflicts.

**Cause**: Node.js version too old, npm permission issues, or conflicting global packages.

**Solution**:

```bash
# Check Node.js version (requires 18+)
node --version

# If version is too old, update via nvm
nvm install 20
nvm use 20

# Fix npm permissions (macOS/Linux)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH  # Add to ~/.zshrc or ~/.bashrc

# Clean install
npm install -g @anthropic-ai/claude-code
```

### Claude Command Not Found

**Symptoms**: Terminal says `claude: command not found` after successful install.

**Cause**: npm global bin directory is not in your PATH.

**Solution**:

```bash
# Find where npm installs global binaries
npm config get prefix

# Add to your shell profile (~/.zshrc, ~/.bashrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.zshrc
```

---

## Authentication Issues

### API Key Not Recognized

**Symptoms**: Claude Code starts but cannot make API calls. Error messages mention authentication or invalid key.

**Cause**: Missing or expired API key. Wrong environment variable.

**Solution**:

```bash
# Re-authenticate
claude /login

# Or set the API key directly
export ANTHROPIC_API_KEY="sk-ant-..."

# For Bedrock users
export CLAUDE_CODE_USE_BEDROCK=1
claude /setup-bedrock
```

### Rate Limit Errors

**Symptoms**: `429 Too Many Requests` errors. Claude stops responding mid-task.

**Cause**: Exceeded plan rate limits. Too many concurrent sessions or agents.

**Solution**:

```bash
# Check current usage
/usage

# Options:
# 1. Wait for rate limit to reset (check Retry-After header)
# 2. Switch to a lighter model for high-volume tasks
/model haiku

# 3. Reduce concurrent agents
# 4. Enable extra usage
/extra-usage
```

---

## Performance Issues

### Slow Responses

**Symptoms**: Claude takes 30+ seconds to respond. Long pauses between tool calls.

**Cause**: Context window too large. Too many skills loaded. Synchronous hooks blocking responses.

**Solution**:

```bash
# Check context usage
/context

# Compact if above 50%
/compact

# Check for slow synchronous hooks
/hooks
# Make non-critical hooks async: "async": true

# Reduce loaded skills — check if unnecessary skills are auto-activating
/skills
```

### High Token Usage

**Symptoms**: Burning through quota faster than expected. `/usage` shows high consumption.

**Cause**: Large CLAUDE.md files. Many skills preloaded. Agent spawning with full context inheritance. No compaction.

**Solution**:

1. Keep CLAUDE.md under 200 lines
2. Use `user-invocable: false` on skills that should only be preloaded by specific agents
3. Set `model: haiku` on agents that do not need full capabilities
4. Compact regularly with `/compact`
5. Use `effort: low` or `effort: medium` for routine tasks

---

## Hook Issues

### Hook Not Firing

**Symptoms**: You configured a hook in settings but it never executes.

**Cause**: Wrong settings file. Missing or mismatched matcher. Hook disabled. Settings hierarchy override.

**Solution**:

```bash
# 1. Check which hooks are active and their sources
/hooks

# 2. Verify the hook is in the correct settings file
# Team hooks: .claude/settings.json
# Personal hooks: .claude/settings.local.json
# Global hooks: ~/.claude/settings.json

# 3. Check if hooks are globally disabled
# Look for "disableAllHooks": true in any settings file

# 4. Verify the matcher matches the tool name exactly
# "Bash" not "bash", "Write" not "write"

# 5. Check settings hierarchy — a higher-priority file may override
```

### Hook Blocking Responses

**Symptoms**: Long pauses after tool calls. Claude appears frozen for several seconds.

**Cause**: Synchronous hook running a slow command. Network call in a non-async hook.

**Solution**:

```json
// Add "async": true to non-blocking hooks
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

### Hook Timeout

**Symptoms**: Hook starts but the result is never used. Warning about hook timeout in logs.

**Cause**: Hook command takes too long. External service unavailable. Script hangs.

**Solution**: Test the hook command manually. Add timeout to the script itself. Ensure external services are reachable. Consider making the hook async if the result is not critical.

---

## MCP Issues

### MCP Server Not Connecting

**Symptoms**: MCP tools are not available. `/mcp` shows server as disconnected.

**Cause**: Server binary not found. Wrong configuration. Port conflict. Missing environment variables.

**Solution**:

```bash
# 1. Check MCP status
/mcp

# 2. Verify server configuration in .mcp.json or settings.json
# Ensure the command path is correct and executable

# 3. Check if required environment variables are set
# MCP servers often need API keys in .env

# 4. Try restarting the MCP connection
# Exit and restart Claude Code

# 5. Test the MCP server manually
npx @modelcontextprotocol/server-name
```

### MCP Permission Denied

**Symptoms**: MCP tools appear but fail with permission errors when invoked.

**Cause**: Tool not in the allowed list. Permission mode too restrictive.

**Solution**: Add the MCP tool to the `allowedTools` list in settings, or approve it when prompted. For agents, include `mcp__<server>__<tool>` in the `tools:` field.

---

## Agent Issues

### Agent Not Auto-Invoking

**Symptoms**: You created an agent with a description containing `PROACTIVELY` but Claude never spawns it automatically.

**Cause**: Description does not clearly describe when to invoke. Agent tool not available in current context. Another mechanism is handling the task.

**Solution**:

1. Ensure the `description` field uses `PROACTIVELY` and clearly states the trigger condition
2. Check that the agent file is in `.claude/agents/` with a `.md` extension
3. Verify Claude has access to the `Agent` tool (not restricted by parent agent's `tools:` field)

### Agent Spawning Bash Instead of Agent Tool

**Symptoms**: Claude runs `claude --agent my-agent` via Bash instead of using the `Agent(...)` tool.

**Cause**: Agent definition uses vague language like "launch" or "run" that Claude interprets as a shell command.

**Solution**: In your CLAUDE.md or agent instructions, explicitly state: "Use the Agent tool, not bash commands, to spawn subagents." Avoid terms like "launch" or "execute" in agent descriptions.

### Agent Exceeding maxTurns

**Symptoms**: Agent stops mid-task with an incomplete result.

**Cause**: `maxTurns` set too low for the task complexity. Agent is looping or retrying.

**Solution**: Increase `maxTurns` in the agent frontmatter. Review the agent's behavior for unnecessary loops. Break complex tasks into smaller subtasks.

---

## Skill Issues

### Skill Not Appearing in /menu

**Symptoms**: You created a skill but it does not show up when you type `/`.

**Cause**: `user-invocable: false` is set. Skill file not in the correct location. YAML frontmatter syntax error.

**Solution**:

```bash
# 1. Check skill listing
/skills

# 2. Verify file location: .claude/skills/<name>/SKILL.md

# 3. Check frontmatter — remove user-invocable: false
# or set user-invocable: true

# 4. Validate YAML — common errors:
# - Missing --- delimiters
# - Incorrect indentation
# - Unquoted special characters
```

### Skill Not Auto-Discovered

**Symptoms**: Skill exists and has a description but Claude never invokes it automatically.

**Cause**: `disable-model-invocation: true` is set. Description does not match the context Claude encounters. Too many skills competing for the same trigger.

**Solution**:

1. Remove `disable-model-invocation: true` if auto-discovery is desired
2. Write a clear `description` using keywords Claude would encounter naturally
3. Reduce skill count — too many auto-discoverable skills creates ambiguity

---

## Memory Issues

### CLAUDE.md Not Loading

**Symptoms**: Instructions in CLAUDE.md are ignored. Claude does not follow project conventions.

**Cause**: File is in a descendant directory (lazy-loaded, not eagerly loaded). File has a typo in the name. Settings override via managed policy.

**Solution**:

```bash
# 1. Check loaded memory files
/context

# 2. Verify file name is exactly "CLAUDE.md" (case-sensitive)

# 3. Understand loading rules:
# Ancestors (upward from cwd): loaded eagerly at startup
# Descendants (downward from cwd): loaded lazily when files in that dir are accessed
# Siblings: never loaded

# 4. If you need it always loaded, move it to an ancestor directory
# or reference it from root CLAUDE.md
```

### CLAUDE.local.md Being Committed

**Symptoms**: Personal preferences appearing for teammates. `.claude/settings.local.json` in git history.

**Cause**: Local files not in `.gitignore`.

**Solution**:

```bash
# Add to .gitignore
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

# Remove from tracking if already committed
git rm --cached CLAUDE.local.md
git rm --cached .claude/settings.local.json
```

---

## Git Issues

### Worktree Cleanup

**Symptoms**: Stale git worktrees left behind after agent execution. Disk space growing.

**Cause**: Agent with `isolation: worktree` crashed or was interrupted before cleanup.

**Solution**:

```bash
# List worktrees
git worktree list

# Remove stale worktrees
git worktree prune

# Force remove a specific worktree
git worktree remove /path/to/worktree --force
```

### Commit Author

**Symptoms**: Git commits attributed to the wrong author when Claude Code creates them.

**Cause**: Git author not configured for Claude Code sessions.

**Solution**:

```bash
# Set git author for the project
git config user.name "Your Name"
git config user.email "your@email.com"

# Or use --author flag in hooks/scripts
git commit --author="Your Name <your@email.com>" -m "message"
```

---

## See Also

- [Settings Best Practice](./claude-settings.md) — Configuration reference for resolving settings-related issues
- [Hooks Best Practice](./claude-hooks.md) — Hook lifecycle and common pitfalls
- [MCP Servers Best Practice](./claude-mcp.md) — MCP server setup and connection debugging
- [Anti-Patterns Guide](./claude-anti-patterns.md) — Patterns that commonly cause the issues described here
- [Memory & Context Management Guide](./claude-memory-guide.md) — Resolving memory and context overflow problems

---

## Sources

- [Claude Code Troubleshooting — Docs](https://code.claude.com/docs/en/troubleshooting)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP — Docs](https://code.claude.com/docs/en/mcp)
- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
