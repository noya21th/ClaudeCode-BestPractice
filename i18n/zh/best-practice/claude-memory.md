🌍 [English](../../../best-practice/claude-memory.md) | **中文** | [日本語](../../ja/best-practice/claude-memory.md) | [Français](../../fr/best-practice/claude-memory.md) | [Русский](../../ru/best-practice/claude-memory.md) | [العربية](../../ar/best-practice/claude-memory.md)

# Claude Memory

通过 CLAUDE.md 文件实现持久化上下文 — 如何编写以及在 monorepo 中如何加载。

<table width="100%">
<tr>
<td><a href="../../zh/">← 返回 Claude Code 最佳实践</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. 编写好的 CLAUDE.md

结构良好的 CLAUDE.md 是提升 Claude Code 输出质量的最有效方式。HumanLayer 有一份优秀的指南，涵盖了应该包含什么、如何组织以及常见陷阱。

- [HumanLayer - 编写好的 Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. 大型 Monorepo 中的 CLAUDE.md

在 monorepo 中使用 Claude Code 时，理解 CLAUDE.md 文件如何加载到上下文中对于有效组织项目指令至关重要。

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="../../../best-practice/assets/claude-memory/claude-memory-monorepo.jpg" alt="monorepo 中的 CLAUDE.md 加载" width="600"></a>
</p>

### 两种加载机制

Claude Code 使用两种不同的机制加载 CLAUDE.md 文件：

#### 祖先加载（向上遍历目录树）

启动 Claude Code 时，它从你的当前工作目录**向上**遍历到文件系统根目录，加载沿途找到的每一个 CLAUDE.md。这些文件在**启动时立即加载**。

#### 后代加载（向下遍历目录树）

当前工作目录下方子目录中的 CLAUDE.md 文件**不会在启动时加载**。它们仅在 Claude 在会话期间读取那些子目录中的文件时才被包含。这就是所谓的**延迟加载**。

### Monorepo 结构示例

考虑一个典型的 monorepo，具有不同组件的独立目录：

```
/mymonorepo/
├── CLAUDE.md          # 根级指令（跨所有组件共享）
├── frontend/
│   └── CLAUDE.md      # 前端特定指令
├── backend/
│   └── CLAUDE.md      # 后端特定指令
└── api/
    └── CLAUDE.md      # API 特定指令
```

### 场景 1：从根目录运行 Claude Code

从 `/mymonorepo/` 运行 Claude Code 时：

```bash
cd /mymonorepo
claude
```

| 文件 | 启动时加载？ | 原因 |
|------|-------------|------|
| `/mymonorepo/CLAUDE.md` | 是 | 这是你的当前工作目录 |
| `/mymonorepo/frontend/CLAUDE.md` | 否 | 仅在你读取/编辑 `frontend/` 中的文件时加载 |
| `/mymonorepo/backend/CLAUDE.md` | 否 | 仅在你读取/编辑 `backend/` 中的文件时加载 |
| `/mymonorepo/api/CLAUDE.md` | 否 | 仅在你读取/编辑 `api/` 中的文件时加载 |

### 场景 2：从组件目录运行 Claude Code

从 `/mymonorepo/frontend/` 运行 Claude Code 时：

```bash
cd /mymonorepo/frontend
claude
```

| 文件 | 启动时加载？ | 原因 |
|------|-------------|------|
| `/mymonorepo/CLAUDE.md` | 是 | 这是祖先目录 |
| `/mymonorepo/frontend/CLAUDE.md` | 是 | 这是你的当前工作目录 |
| `/mymonorepo/backend/CLAUDE.md` | 否 | 目录树的不同分支 |
| `/mymonorepo/api/CLAUDE.md` | 否 | 目录树的不同分支 |

### 关键要点

1. **祖先始终在启动时加载** — Claude 向上遍历目录树并加载找到的所有 CLAUDE.md 文件。这确保你始终能访问到根级的、仓库范围的指令。

2. **后代延迟加载** — 子目录的 CLAUDE.md 文件仅在你与那些子目录中的文件交互时才加载。这防止了无关上下文膨胀你的会话。

3. **同级目录永远不加载** — 如果你在 `frontend/` 中工作，`backend/CLAUDE.md` 或 `api/CLAUDE.md` 不会被加载到上下文中。

4. **全局 CLAUDE.md** — 你还可以在 `~/.claude/CLAUDE.md` 放置一个全局 CLAUDE.md，它适用于所有 Claude Code 会话，不限项目。

### 为什么这种设计适合 Monorepo

- **共享指令向下传播** — 根级 CLAUDE.md 包含适用于所有地方的仓库范围约定、编码标准和通用模式。

- **组件特定指令保持隔离** — 前端开发者不需要后端特定指令干扰他们的上下文，反之亦然。

- **上下文得到优化** — 通过延迟加载后代 CLAUDE.md 文件，Claude Code 避免在启动时加载可能数百 KB 的无关指令。

### 最佳实践

1. **将共享约定放在根 CLAUDE.md** — 编码标准、commit message 格式、PR 模板和其他仓库范围的准则。

2. **将组件特定指令放在组件 CLAUDE.md** — 框架特定模式、组件架构、该组件独有的测试约定。

3. **使用 CLAUDE.local.md 存放个人偏好** — 加入 `.gitignore`，用于不应与团队共享的指令。

---

## 来源

- [Claude Code 文档 - Claude 如何查找记忆](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny 在 X 上 - CLAUDE.md 加载机制说明](https://x.com/bcherny/status/2016339448863355206)
- [HumanLayer - 编写好的 Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
