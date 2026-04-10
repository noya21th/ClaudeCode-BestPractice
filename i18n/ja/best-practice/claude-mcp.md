🌍 [English](../../../best-practice/claude-mcp.md) | [中文](../../zh/best-practice/claude-mcp.md) | **日本語** | [Français](../../fr/best-practice/claude-mcp.md) | [Русский](../../ru/best-practice/claude-mcp.md) | [العربية](../../ar/best-practice/claude-mcp.md)

# MCP Servers ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../.mcp.json)

MCP（Model Context Protocol）Servers は、外部ツール、データベース、API への接続で Claude Code を拡張します。このガイドでは、日常使用に推奨されるサーバーと設定のベストプラクティスを説明します。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 日常使用の MCP Servers

> *「15個の MCP Servers を入れて多いほど良いと思ったけど、結局毎日使うのは4つだけだった。」* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)（682 upvotes）

| MCP Server | 機能 | リソース |
|------------|------|----------|
| [**Context7**](https://github.com/upstash/context7) | 最新のライブラリドキュメントをコンテキストに取り込み。古い学習データによる API のハルシネーションを防止 | [Reddit: 「コーディングに最適な MCP」](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | ブラウザ自動化 — UI 機能の実装、テスト、検証を自律的に実行。スクリーンショット、ナビゲーション、フォームテスト | [Reddit: フロントエンドに必須](https://reddit.com/r/mcp/comments/1m59pk0/) · [ドキュメント](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | Claude を実際の Chrome ブラウザに接続 — コンソール、ネットワーク、DOM を検査。ユーザーが実際に見ているものをデバッグ | [Reddit: デバッグの「ゲームチェンジャー」](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [比較レポート](../../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | 任意の GitHub リポジトリの構造化されたウィキスタイルのドキュメントを取得 — アーキテクチャ、API サーフェス、関係性 | [Reddit: 「Context7 の背後にゲートウェイとして配置」](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | プロンプトからアーキテクチャ図、フローチャート、システム設計を手書き風の Excalidraw スケッチとして生成 | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

調査（Context7/DeepWiki）→ デバッグ（Playwright/Chrome）→ ドキュメント化（Excalidraw）

---

## 設定

MCP Servers はプロジェクトルートの `.mcp.json`（プロジェクトスコープ）または `~/.claude.json`（ユーザースコープ）で設定します。

### サーバータイプ

| タイプ | トランスポート | 例 |
|--------|---------------|-----|
| **stdio** | ローカルプロセスを起動 | `npx`, `python`, バイナリ |
| **http** | リモート URL に接続 | HTTP/SSE エンドポイント |

### `.mcp.json` の例

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

API キーを `.mcp.json` にコミットする代わりに、シークレットには環境変数展開を使用:

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

### MCP Servers 用の設定

`.claude/settings.json` のこれらの設定で MCP Server の承認を制御:

| キー | 型 | 説明 |
|------|-----|------|
| `enableAllProjectMcpServers` | boolean | プロンプトなしですべての `.mcp.json` サーバーを自動承認 |
| `enabledMcpjsonServers` | array | 自動承認する特定のサーバー名の許可リスト |
| `disabledMcpjsonServers` | array | 拒否する特定のサーバー名のブロックリスト |

### MCP ツールの権限ルール

MCP ツールは権限ルールで `mcp__<server>__<tool>` の命名規則に従います:

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

## MCP スコープ

MCP Servers は3つのレベルで定義可能:

| スコープ | 場所 | 目的 |
|---------|------|------|
| **プロジェクト** | `.mcp.json`（リポジトリルート） | チーム共有サーバー、git にコミット |
| **ユーザー** | `~/.claude.json`（`mcpServers` キー） | 全プロジェクト横断の個人サーバー |
| **Subagent** | Agent frontmatter（`mcpServers` フィールド） | 特定の Subagent にスコープされたサーバー |

優先順位: Subagent > プロジェクト > ユーザー

---

## ソース

- [MCP Servers — Claude Code ドキュメント](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol 仕様](https://modelcontextprotocol.io/)
- [本当に10倍速くなった5つの MCP — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [MCP Server 過積載の議論 — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
