🌍 [English](../../../best-practice/claude-memory-guide.md) | **中文** | [日本語](../../ja/best-practice/claude-memory-guide.md) | [Français](../../fr/best-practice/claude-memory-guide.md) | [Русский](../../ru/best-practice/claude-memory-guide.md) | [العربية](../../ar/best-practice/claude-memory-guide.md)

# 记忆与上下文管理指南

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code 记忆系统全面指南 — 涵盖 CLAUDE.md 文件、自动记忆、Agent 记忆和 Rules。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 记忆类型概览

Claude Code 有四种不同的记忆系统，各自服务于不同用途：

| 记忆类型 | 位置 | 作用域 | 持久性 | 加载时机 |
|----------|------|--------|--------|----------|
| **CLAUDE.md** | 项目目录 | 按项目、分层级 | 永久（基于文件） | 会话启动时（祖先目录）或访问文件时（后代目录） |
| **自动记忆** | `~/.claude/memory/` 和 `~/.claude/projects/<hash>/memory/` | 用户全局或按项目 | 永久（基于文件） | 会话启动时 |
| **Agent 记忆** | `~/.claude/agent-memory/<agent>/` 或 `.claude/agent-memory/<agent>/` | 按 Agent，由 `memory:` 字段限定 | 永久（基于文件） | Agent 启动时 |
| **Rules** | `.claude/rules/*.md` 和 `~/.claude/rules/*.md` | 按文件模式限定 | 永久（基于文件） | 访问匹配 glob 模式的文件时 |

---

## CLAUDE.md 层级结构

### 文件位置

| 层级 | 路径 | 共享 | 说明 |
|------|------|------|------|
| **全局** | `~/.claude/CLAUDE.md` | 否（个人） | 适用于本机所有 Claude Code 会话 |
| **项目根目录** | `./CLAUDE.md` | 是（git 跟踪） | 团队共享的项目级指令 |
| **项目本地** | `./CLAUDE.local.md` | 否（git 忽略） | 个人的项目级覆盖 |
| **组件** | `./frontend/CLAUDE.md` | 是（git 跟踪） | 组件级指令 |
| **嵌套** | `./src/api/v2/CLAUDE.md` | 是（git 跟踪） | 深层目录级指令 |

### 加载机制

Claude Code 使用两种不同的机制来发现 CLAUDE.md 文件：

#### 祖先加载（向上遍历）— 即时

会话启动时，Claude 从当前工作目录**向上**遍历到文件系统根目录，加载沿途找到的每个 CLAUDE.md。这些文件在**启动时立即加载**。

```
/home/user/projects/myapp/src/   ← cwd
/home/user/projects/myapp/       ← CLAUDE.md 已加载（祖先）
/home/user/projects/             ← CLAUDE.md 已加载（祖先）
/home/user/                      ← CLAUDE.md 如存在则加载（祖先）
~/.claude/                       ← CLAUDE.md 已加载（全局）
```

#### 后代加载（向下遍历）— 懒加载

cwd 以下子目录中的 CLAUDE.md 文件**不会**在启动时加载。只有当 Claude 在会话中读取或编辑这些子目录中的文件时才会被包含。

```
/myapp/                 ← cwd，CLAUDE.md 在启动时加载
/myapp/frontend/        ← CLAUDE.md 在 Claude 访问 frontend/ 文件时加载
/myapp/backend/         ← CLAUDE.md 在 Claude 访问 backend/ 文件时加载
/myapp/backend/api/     ← CLAUDE.md 在 Claude 访问 backend/api/ 文件时加载
```

#### 核心规则

- **祖先目录始终在启动时加载** — 从 cwd 向上即时加载
- **后代目录懒加载** — 仅在访问该目录中的文件时加载
- **兄弟目录永远不加载** — 在 `frontend/` 中工作不会加载 `backend/CLAUDE.md`
- **全局始终加载** — `~/.claude/CLAUDE.md` 适用于每个会话

---

## Monorepo 策略

### 根 CLAUDE.md — 共享规范

根文件聚焦于适用于所有地方的团队级标准：

```markdown
# CLAUDE.md (root)

## Coding Standards
- TypeScript strict mode everywhere
- No `any` types without justification comments
- Prefer functional components with hooks

## Git Conventions
- Conventional commits: feat/fix/chore/docs/refactor
- Squash merge to main

## Testing
- Every PR must include tests
- Minimum 80% coverage for new code
```

### 组件 CLAUDE.md — 具体内容

每个组件有自己专注的指令：

```markdown
# frontend/CLAUDE.md

## Stack
- Next.js 15 App Router
- Tailwind CSS + shadcn/ui
- React Query for server state

## Patterns
- Server Components by default
- Client Components only for interactivity
- Use app/api/ for API routes
```

### CLAUDE.local.md — 个人偏好

不应被提交的个人覆盖：

```markdown
# CLAUDE.local.md (git-ignored)

- Use verbose comments — I am learning this codebase
- Run prettier after every file edit
- Always explain your reasoning before making changes
```

---

## 自动记忆系统

自动记忆是 Claude Code 跨对话记忆事物的系统。它存储在会话间持久化的观察和学习成果。

### 类型

| 类型 | 存储位置 | 说明 |
|------|----------|------|
| `user` | `~/.claude/memory/` | 用户偏好和全局模式 — 适用于所有项目 |
| `feedback` | `~/.claude/memory/` | 来自用户反馈的纠正和行为调整 |
| `project` | `~/.claude/projects/<hash>/memory/` | 项目特定的学习成果和上下文 |
| `reference` | `~/.claude/projects/<hash>/memory/` | Claude 认为对该项目有用的参考材料 |

### 工作原理

1. Claude 在会话中观察到某种模式、偏好或纠正
2. 如果该观察可能在未来的会话中有用，Claude 将其存储为自动记忆
3. 在下次会话启动时，相关的自动记忆条目被加载到上下文中
4. 用户可以通过 `/memory` 查看和管理条目

### 何时使用自动记忆 vs CLAUDE.md

| 使用自动记忆的场景 | 使用 CLAUDE.md 的场景 |
|-------------------|---------------------|
| 指令是在会话中从反馈中产生的 | 指令是预先规划好的 |
| 这是个人偏好（语气、详细程度、工作流） | 这是团队规范（编码标准、架构） |
| 它随着使用不断演变 | 它是稳定的，经过代码评审 |
| 它适用于该用户的所有项目 | 它适用于该项目，不论是谁 |

---

## Agent 记忆

frontmatter 中带有 `memory:` 字段的 Agent 维护跨会话持久化的记忆。这使得 Agent 能够随时间学习和改进。

### 作用域

| 作用域 | 存储位置 | 共享 | 说明 |
|--------|----------|------|------|
| `user` | `~/.claude/agent-memory/<agent>/` | 否 | 该 Agent 的跨项目记忆 — 跟随用户 |
| `project` | `.claude/agent-memory/<agent>/` | 是 | 项目特定记忆 — 通过 git 与团队共享 |
| `local` | `.claude/agent-memory/<agent>/`（git 忽略） | 否 | 项目特定的个人记忆 |

### 配置

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes"
memory: project
---
Review code changes. After each review, note patterns you see frequently
so you can flag them earlier in future reviews.
```

### 使用场景

- **代码审查器**配合 `memory: project` — 学习团队特定模式、常见 bug、风格偏好
- **研究助手**配合 `memory: user` — 记住已探索的主题、偏好的来源、写作风格
- **部署 Agent** 配合 `memory: project` — 追踪部署历史、已知的环境问题

---

## Rules (.claude/rules/)

Rules 是基于 glob 模式的 Markdown 文件，当 Claude 访问匹配 glob 模式的文件时自动加载。

### 位置

| 路径 | 作用域 |
|------|--------|
| `.claude/rules/*.md` | 项目级 Rules（git 跟踪，团队共享） |
| `~/.claude/rules/*.md` | 全局 Rules（个人，适用于所有项目） |

### 格式

每个 Rule 文件以 `# Glob:` 行开头，定义何时激活：

```markdown
# Glob: **/*.tsx

## React Component Rules

- Always use functional components
- Export components as named exports, not default
- Co-locate styles in a .module.css file
```

```markdown
# Glob: **/*.test.*

## Testing Rules

- Use describe/it blocks, not test()
- Mock external dependencies, not internal modules
- Assert on behavior, not implementation
```

### 何时使用 Rules vs CLAUDE.md

| 使用 Rules 的场景 | 使用 CLAUDE.md 的场景 |
|------------------|---------------------|
| 指令适用于特定文件类型 | 指令适用于整个项目 |
| 你希望基于文件访问进行懒加载 | 你希望在会话启动时即时加载 |
| 指南限定于某种模式（`*.py`、`*.tsx`、`migrations/*.sql`） | 指南是通用的（git 规范、架构决策） |

---

## 对比表

| 特性 | CLAUDE.md | 自动记忆 | Agent 记忆 | Rules |
|------|-----------|----------|-----------|-------|
| **创建者** | 开发者（手动） | Claude（自动） | Agent（自动） | 开发者（手动） |
| **与团队共享** | 是（`.local.md` 除外） | 否 | 是（`project` 作用域时） | 是 |
| **启动时加载** | 是（祖先目录） | 是 | 是（Agent 启动时） | 否（懒加载） |
| **按文件类型限定** | 否 | 否 | 否 | 是（glob 模式） |
| **用户可编辑** | 是（文件编辑） | 是（`/memory`） | 是（文件编辑） | 是（文件编辑） |
| **跨会话持久化** | 是 | 是 | 是 | 是 |
| **最适用于** | 项目规范 | 学习到的偏好 | Agent 改进 | 文件类型相关指导 |

---

## 最佳实践

### CLAUDE.md 控制在 200 行以内

过长的 CLAUDE.md 文件会导致上下文膨胀，Claude 可能忽略底部的指令。将大文件拆分为组件级 CLAUDE.md 和 Rules。

### 使用 Rules 处理文件类型指导

不要在 CLAUDE.md 中写"编辑 Python 文件时做 X"。改为创建 `.claude/rules/python.md`，带上 `# Glob: **/*.py`。这样只在需要时才加载。

### 使用 Agent 记忆让 Agent 持续进化

如果一个 Agent 应该随时间变得更智能，给它一个 `memory:` 作用域。Agent 会存储改进后续运行的观察结果 — 审查模式、代码异味、部署怪癖。

### 在 50% 上下文使用量时压缩

定期运行 `/context` 检查使用量。达到 50% 时运行 `/compact`，可附带聚焦说明：`/compact focus on the authentication refactor`。这样在释放空间的同时保留重要上下文。

### Git 忽略个人文件

始终添加到 `.gitignore`：

```
CLAUDE.local.md
.claude/settings.local.json
```

### 将稳定指令与演进指令分开

- **稳定**（编码标准、架构）→ CLAUDE.md 和 Rules
- **演进**（偏好、模式、学习成果）→ 自动记忆和 Agent 记忆

---

## 陈旧与清理

### 记忆过时的迹象

- CLAUDE.md 引用了已废弃的 API 或已移除的功能
- 自动记忆条目与当前 CLAUDE.md 指令矛盾
- Agent 记忆包含过时的项目特定模式
- Rules 引用了代码库中已不存在的文件模式

### 清理策略

1. **每季度审计 CLAUDE.md** — 逐条阅读指令，验证是否仍然适用
2. **审查自动记忆** — 运行 `/memory` 并删除不再相关的条目
3. **清理 Agent 记忆** — 检查 `.claude/agent-memory/` 中是否有过时的观察
4. **更新 Rules** — 验证 glob 模式是否仍匹配项目中的文件
5. **用注释追踪** — 在 CLAUDE.md 各节添加 `<!-- Last reviewed: 2026-04-09 -->`

---

## 参考资料

- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code Rules — Docs](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)
- [Claude Code Sub-agents (Memory) — Docs](https://code.claude.com/docs/en/sub-agents)
- [CLAUDE.md in Monorepos](../best-practice/claude-memory.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
