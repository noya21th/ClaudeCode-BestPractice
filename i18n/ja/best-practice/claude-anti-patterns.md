🌍 [English](../../../best-practice/claude-anti-patterns.md) | [中文](../../zh/best-practice/claude-anti-patterns.md) | **日本語** | [Français](../../fr/best-practice/claude-anti-patterns.md) | [Русский](../../ru/best-practice/claude-anti-patterns.md) | [العربية](../../ar/best-practice/claude-anti-patterns.md)

# アンチパターンガイド

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code 設定でよくある間違いとその回避方法 — カテゴリ別に整理。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## アーキテクチャのアンチパターン

### 1. モノリシック CLAUDE.md

**問題**: インストール手順、コーディング規約、API ドキュメント、デプロイ手順、デバッグのコツがすべて混在した 500 行超の CLAUDE.md ファイル。

**症状**: Claude がファイル末尾付近の指示を無視する。規約の順守が不安定。セッション開始時のコンテキスト肥大化。

**修正**: 目的別のファイルに分割する。ルートの CLAUDE.md は共有規約のみで 200 行以内に収める。コンポーネントレベルの CLAUDE.md で詳細を記述する。`.claude/rules/*.md` に glob パターン付きでファイルタイプ固有のガイダンスを配置する。

```
# 修正前（モノリシック）
CLAUDE.md              # すべてを網羅した 800 行

# 修正後（分割）
CLAUDE.md              # 150 行 — 共有規約、プロジェクト概要
frontend/CLAUDE.md     # 80 行 — React パターン、コンポーネント構成
backend/CLAUDE.md      # 80 行 — API 規約、データベースパターン
.claude/rules/ts.md    # TypeScript 固有ルール（Glob: **/*.ts）
.claude/rules/test.md  # テスト規約（Glob: **/*.test.*）
```

### 2. 何でも Agent

**問題**: 現在のコンテキストに手順を注入するだけで済む単純なタスクに Agent を使用する。

**症状**: 不要なコンテキスト分離。メイン会話に結果が表示されない。Agent 起動オーバーヘッドによる実行速度の低下。コンテキスト重複によるトークンの浪費。

**修正**: 再利用可能なドメイン知識と手順には Skill を使用する。ユーザー起動のプロンプトテンプレートには Command を使用する。Agent は分離、メモリ、ツール制限、バックグラウンド実行が本当に必要なタスクにのみ使用する。

### 3. コンテキスト分離の欠如

**問題**: 長時間の探索タスク（リサーチ、コードレビュー、データ分析）を分離せずにメイン会話で直接実行する。

**症状**: 会話コンテキストがすぐに埋まる。無関係な中間結果が後続の応答を汚染する。頻繁に `/compact` が必要になる。

**修正**: 大量の中間出力を生成するタスクには Agent を使用する。Agent は新しいコンテキストで実行され、サマリーのみを返します。

### 4. 循環 Agent 生成

**問題**: Agent A が Agent B を生成し、Agent B が Agent A を生成する無限ループ。

**症状**: `maxTurns` の上限に到達する。トークン予算が枯渇する。有用な出力が得られない。

**修正**: DAG（有向非巡回グラフ）ワークフローを設計する。`tools:` フィールドでサブエージェントが生成できる Agent を制限する。Agent 定義に循環する `Agent(...)` 参照がないか確認する。

```yaml
# 生成可能な Agent を制限する
---
name: planner
tools: Read, Glob, Grep, Agent(implementer)
# planner は implementer のみ生成可能、自分自身は生成できない
---
```

### 5. モデル名のハードコーディング

**問題**: Agent 定義で `claude-sonnet-4-20250514` のような完全なモデル ID を使用する。

**症状**: モデル ID が非推奨になると Agent が動作しなくなる。環境間で動作が不安定になる。

**修正**: エイリアスを使用する：`haiku`、`sonnet`、`opus`、または `inherit`。これらは常に最新バージョンに解決されます。

---

## 設定のアンチパターン

### 6. 過度に緩い権限

**問題**: すべての Agent に `"permissionMode": "bypassPermissions"` を設定し、`--dangerously-skip-permissions` をデフォルトとして使用する。

**症状**: レビューなしのファイル削除。意図しない `git push --force`。信頼できない MCP ツールからのセキュリティリスク。ツール承認の監査証跡がない。

**修正**: ファイル書き込みは必要だが任意のコマンドは実行すべきでない Agent には `acceptEdits` を使用する。調査専用の Agent には `plan` を使用する。`bypassPermissions` はサンドボックスのある CI/CD 環境のみに限定する。

| モード | 編集 | Bash | 最適な用途 |
|--------|------|------|-----------|
| `default` | 確認 | 確認 | インタラクティブな開発 |
| `acceptEdits` | 自動 | 確認 | ファイル書き込み Agent |
| `plan` | 拒否 | 拒否 | 調査・計画 Agent |
| `bypassPermissions` | 自動 | 自動 | サンドボックス付き CI/CD のみ |

### 7. 設定の優先順位を無視

**問題**: 間違ったファイルで設定を定義し、なぜ上書きや無視されるのか疑問に思う。

**症状**: Hook が発火しない。権限が期待通りでない。チームメンバー間で動作が異なる。

**修正**: 優先順位を理解する（高い方が優先）：

1. **Managed**（`managed-settings.json`） — 組織が強制する設定
2. **CLI フラグ** — 単一セッションのオーバーライド
3. **Local**（`.claude/settings.local.json`） — 個人のプロジェクト設定
4. **Shared**（`.claude/settings.json`） — チーム共有のプロジェクト設定
5. **Global**（`~/.claude/settings.json`） — 個人のデフォルト設定

### 8. すべての Hook が同期

**問題**: すべての Hook を同期実行に設定し、スクリプト実行中に Claude の応答がブロックされる。

**症状**: 応答が遅い。ツール呼び出しのたびに顕著な停止。ユーザーが Claude を遅いと感じる。

**修正**: 副作用を実行する Hook（ログ記録、通知、テレメトリ）には `"async": true` を設定する。セキュリティチェックや権限判断など、動作をゲートまたは変更する必要がある Hook のみを同期にする。

### 9. settings.local.json を使わない

**問題**: 個人的な設定（通知音、カスタムパス、デバッグログ）を `.claude/settings.json` にコミットし、チーム全体に影響を与える。

**症状**: チームメイトにあなたの通知音が聞こえる。デバッグ Hook が本番環境で実行される。個人用の MCP サーバーが全員に表示される。

**修正**: 個人的なオーバーライドには `.claude/settings.local.json` を使用する。`.gitignore` に追加されていることを確認する。チーム全体が必要とする Hook と設定のみを共有する。

---

## Skill のアンチパターン

### 10. Agent 専用 Skill に user-invocable: true を設定

**問題**: Agent プリロード専用（`skills:` フィールド経由）の Skill がユーザーの `/` スラッシュコマンドメニューに表示される。

**症状**: ユーザーが Skill を直接呼び出し、Agent のバックグラウンド知識として設計されているため紛らわしい結果が得られる。

**修正**: Agent プリロード専用の Skill には `user-invocable: false` を設定する。

```yaml
---
name: internal-api-conventions
user-invocable: false
description: "REST API conventions for the internal platform"
---
```

### 11. description の欠如

**問題**: Skill の frontmatter に `description` フィールドがない。

**症状**: Claude が Skill を自動検出できない。ユーザーが明示的に `/slash-command` を入力した場合にのみ発火する。自動呼び出しロジックから見えない状態になる。

**修正**: Skill を **いつ** 使うべきかを説明する `description` を必ず記述する。Claude が自然に遭遇するキーワードを使用する：「React コンポーネント作成時に使用」や「Python テストファイルでトリガー」など。

### 12. 巨大な Skill ファイル

**問題**: すべてのエッジケース、例、リファレンスを含む 1000 行超の SKILL.md ファイル。

**症状**: Skill ロード時のコンテキスト肥大化。Claude が末尾付近の指示を無視する可能性。Agent にプリロードする際の読み込みが遅い。

**修正**: SKILL.md はコア手順に絞って 200 行以内に収める。参考資料は Skill ディレクトリ内の `references/` ファイルに移動する。Claude がオンデマンドで読み込む補足コンテンツには `@path` インポートを使用する。

### 13. 頻繁に必要な Skill に disable-model-invocation を設定

**問題**: Claude が自動検出して呼び出すべき Skill に `disable-model-invocation: true` を設定する。

**症状**: Claude が Skill を自動的に呼び出さない。コンテキスト上明らかに Skill が必要な場合でも、ユーザーが常に `/slash-command` を手動入力する必要がある。

**修正**: `disable-model-invocation: true` は純粋にユーザー起動の Skill（危険な操作、デプロイなど）にのみ使用する。Claude が自ら検出すべき Skill では未設定にするか `false` に設定する。

---

## 運用のアンチパターン

### 14. コンパクトしない

**問題**: コンパクトせずに長時間セッションを実行し、コンテキストが 100% に達する。

**症状**: Claude が以前の指示を忘れ始める。応答の精度が低下する。最終的にコンテキストオーバーフローでエラーが発生する。

**修正**: コンテキスト使用量が約 50% になったら手動で `/compact` を実行する。`/context` で現在の使用量を可視化する。長いタスクの場合、各サブタスクがコンテキスト容量の 50% 以内で完了するように分割する。

### 15. 検証ループなし

**問題**: Claude の出力を検証せずに受け入れる — テスト実行なし、型チェックなし、目視確認なし。

**症状**: 生成されたコードに構文エラーがある。API 連携で間違ったエンドポイントを使用している。設定ファイルにタイプミスがある。

**修正**: 常に出力を検証する。コード生成後にテストを実行する。視覚的な検証にはブラウザ自動化（Chrome、Playwright）を使用する。Command や Agent の定義に検証ステップを含める。

### 16. レート制限の無視

**問題**: トークン予算とレート制限を考慮せずに大規模なマルチ Agent タスクを計画する。

**症状**: Agent がタスク途中でレート制限に到達する。作業が失われるか不完全になる。請求の予想外の増加。

**修正**: `/usage` で現在のレート制限状況を確認する。予算内に収まるようにタスクを計画する。高頻度・低複雑度のサブタスクには `model: haiku` を使用する。大きなタスクを小さなセッションに分割する。

---

## ソース

- [Claude Code Best Practices — Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Permissions — Docs](https://code.claude.com/docs/en/permissions)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
