🌍 [English](../../../best-practice/claude-subagents.md) | [中文](../../zh/best-practice/claude-subagents.md) | **日本語** | [Français](../../fr/best-practice/claude-subagents.md) | [Русский](../../ru/best-practice/claude-subagents.md) | [العربية](../../ar/best-practice/claude-subagents.md)

# Sub-agents ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A34%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-subagents-implementation.md)

Claude Code の Subagents — frontmatter フィールドと公式組み込み Agent タイプ。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter フィールド (16)

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| `name` | string | はい | 小文字とハイフンを使用した一意の識別子 |
| `description` | string | はい | 呼び出しタイミング。Claude による自動呼び出しには `"PROACTIVELY"` を使用 |
| `tools` | string/list | いいえ | ツールのカンマ区切り許可リスト（例：`Read, Write, Edit, Bash`）。省略時はすべてのツールを継承。`Agent(agent_type)` 構文で生成可能な Subagent を制限可能。旧 `Task(agent_type)` エイリアスも動作する |
| `disallowedTools` | string/list | いいえ | 拒否するツール。継承または指定されたリストから除外される |
| `model` | string | いいえ | 使用モデル: `sonnet`, `opus`, `haiku`, 完全なモデル ID（例：`claude-opus-4-6`）、または `inherit`（デフォルト: `inherit`） |
| `permissionMode` | string | いいえ | 権限モード: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns` | integer | いいえ | Subagent が停止するまでのエージェントターンの最大数 |
| `skills` | list | いいえ | Agent 起動時にコンテキストにプリロードする Skill 名（利用可能にするだけでなく、全内容が注入される） |
| `mcpServers` | list | いいえ | この Subagent 用の MCP Servers — サーバー名の文字列またはインラインの `{name: config}` オブジェクト |
| `hooks` | object | いいえ | この Subagent にスコープされたライフサイクル Hooks。すべての Hook イベントがサポートされ、`PreToolUse`、`PostToolUse`、`Stop` が最も一般的 |
| `memory` | string | いいえ | 永続メモリスコープ: `user`, `project`, `local` |
| `background` | boolean | いいえ | `true` に設定するとバックグラウンドタスクとして常に実行（デフォルト: `false`） |
| `effort` | string | いいえ | この Subagent がアクティブな時の努力レベルオーバーライド: `low`, `medium`, `high`, `max`（Opus 4.6 のみ）。デフォルト: セッションから継承 |
| `isolation` | string | いいえ | `"worktree"` に設定すると一時的な git worktree で実行（変更がなければ自動クリーンアップ） |
| `initialPrompt` | string | いいえ | この Agent がメインセッション Agent として実行される時（`--agent` または `agent` 設定経由）、最初のユーザーターンとして自動送信。Commands と Skills が処理される。ユーザー提供のプロンプトの前に追加 |
| `color` | string | いいえ | タスクリストとトランスクリプトでの Subagent の表示色: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Agent | モデル | ツール | 説明 |
|---|-------|--------|--------|------|
| 1 | `general-purpose` | inherit | すべて | 複雑なマルチステップタスク — 調査、コード検索、自律作業のデフォルト Agent タイプ |
| 2 | `Explore` | haiku | 読み取り専用（Write, Edit なし） | 高速なコードベース検索と探索 — ファイル検索、コード検索、コードベースに関する質問への回答に最適化 |
| 3 | `Plan` | inherit | 読み取り専用（Write, Edit なし） | Plan mode での事前計画調査 — コード記述前にコードベースを探索し実装方針を設計 |
| 4 | `statusline-setup` | sonnet | Read, Edit | ユーザーの Claude Code Status Line 設定を構成 |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Claude Code の機能、Agent SDK、Claude API に関する質問に回答 |

---

## ソース

- [カスタム Subagents の作成 — Claude Code ドキュメント](https://code.claude.com/docs/en/sub-agents)
- [CLI リファレンス — Claude Code ドキュメント](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
