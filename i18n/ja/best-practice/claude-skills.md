🌍 [English](../../../best-practice/claude-skills.md) | [中文](../../zh/best-practice/claude-skills.md) | **日本語** | [Français](../../fr/best-practice/claude-skills.md) | [Русский](../../ru/best-practice/claude-skills.md) | [العربية](../../ar/best-practice/claude-skills.md)

# Skills ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A33%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-skills-implementation.md)

Claude Code の Skills — frontmatter フィールドと公式バンドル Skill。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter フィールド (13)

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| `name` | string | いいえ | 表示名と `/slash-command` 識別子。省略時はディレクトリ名がデフォルト |
| `description` | string | 推奨 | Skill の機能説明。オートコンプリートに表示され、Claude の自動検出に使用 |
| `argument-hint` | string | いいえ | オートコンプリート時に表示されるヒント（例：`[issue-number]`, `[filename]`） |
| `disable-model-invocation` | boolean | いいえ | `true` に設定すると Claude がこの Skill を自動呼び出しするのを防止 |
| `user-invocable` | boolean | いいえ | `false` に設定すると `/` メニューから非表示 — Skill はバックグラウンドナレッジのみとなり、Agent プリロード用 |
| `allowed-tools` | string | いいえ | この Skill がアクティブな時に権限プロンプトなしで許可されるツール |
| `model` | string | いいえ | この Skill 実行時に使用するモデル（例：`haiku`, `sonnet`, `opus`） |
| `effort` | string | いいえ | 呼び出し時のモデル努力レベルオーバーライド（`low`, `medium`, `high`, `max`） |
| `context` | string | いいえ | `fork` に設定すると分離された Subagent コンテキストで Skill を実行 |
| `agent` | string | いいえ | `context: fork` 設定時の Subagent タイプ（デフォルト: `general-purpose`） |
| `hooks` | object | いいえ | この Skill にスコープされたライフサイクル Hooks |
| `paths` | string/list | いいえ | Skill が自動アクティベーションする条件を制限する Glob パターン。カンマ区切りの文字列または YAML リストを受け付け、一致するファイルを操作中のみ Claude が Skill をロード |
| `shell` | string | いいえ | `` !`command` `` ブロック用のシェル — `bash`（デフォルト）または `powershell`。`CLAUDE_CODE_USE_POWERSHELL_TOOL=1` が必要 |

---

## ![Official](../../../!/tags/official.svg) **(5)**

| # | Skill | 説明 |
|---|-------|------|
| 1 | `simplify` | 変更されたコードの再利用性、品質、効率をレビュー — 重複を排除するリファクタリング |
| 2 | `batch` | 複数ファイルにわたってコマンドを一括実行 |
| 3 | `debug` | 失敗するコマンドやコードの問題をデバッグ |
| 4 | `loop` | プロンプトやスラッシュコマンドを定期的に実行（最大3日間） |
| 5 | `claude-api` | Claude API または Anthropic SDK でアプリを構築 — `anthropic` / `@anthropic-ai/sdk` のインポートでトリガー |

参照: [公式 Skills リポジトリ](https://github.com/anthropics/skills/tree/main/skills) — コミュニティがメンテナンスするインストール可能な Skills。

---

## ソース

- [Claude Code Skills — ドキュメント](https://code.claude.com/docs/en/skills)
- [モノレポでの Skills 検出](../../../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
