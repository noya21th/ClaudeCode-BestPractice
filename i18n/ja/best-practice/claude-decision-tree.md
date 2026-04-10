🌍 [English](../../../best-practice/claude-decision-tree.md) | [中文](../../zh/best-practice/claude-decision-tree.md) | **日本語** | [Français](../../fr/best-practice/claude-decision-tree.md) | [Русский](../../ru/best-practice/claude-decision-tree.md) | [العربية](../../ar/best-practice/claude-decision-tree.md)

# Agent vs Skill vs Command — 判断ツリー

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Agent、Skill、Command のどれを使うべきか — 実践シナリオに基づく判断フレームワーク。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## クイック比較

| 観点 | Agent | Command | Skill |
|------|-------|---------|-------|
| **配置場所** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **コンテキスト** | 独立した新しいコンテキスト | 現在のコンテキストに注入 | 現在のコンテキストに注入（`context: fork` でフォーク可能） |
| **呼び出し方法** | `Agent(...)` ツール、description による自動検出 | ユーザーが `/slash-command` を入力 | `/slash-command`、自動検出、または Agent へのプリロード |
| **メモリ** | 永続的（`user`/`project`/`local` スコープ） | なし | なし |
| **ツール** | `tools:` フィールドによるカスタム許可リスト | セッションのツールを継承 | セッションのツールを継承（`allowed-tools:` で制限可能） |
| **モデル** | 設定可能（`haiku`/`sonnet`/`opus`/`inherit`） | 設定可能 | 設定可能 |
| **権限** | 設定可能（`acceptEdits`/`plan`/`bypassPermissions`） | セッション権限を継承 | セッション権限を継承 |
| **自動検出** | あり（`description` 経由、`PROACTIVELY` を使用） | なし（ユーザー起動のみ） | あり（`description` 経由） |
| **プリロード可能** | 不可 | 不可 | 可能（Agent の `skills:` フィールド経由） |
| **バックグラウンド** | 可能（`background: true`） | 不可 | 不可 |
| **Worktree 分離** | 可能（`isolation: worktree`） | 不可 | 不可 |
| **最適な用途** | 分離が必要な複雑なマルチステップタスク | ユーザー起動のワークフローとオーケストレーション | 再利用可能なドメイン知識と手順 |

---

## 判断フローチャート

```
開始: 「Claude Code に機能を追加したい」
│
├─ 独立したコンテキストが必要か？
│  ├─ はい ──→ セッション間で永続的なメモリが必要か？
│  │          ├─ はい ──→ AGENT（memory: user/project/local を指定）
│  │          └─ いいえ ──→ AGENT（または context: fork の Skill）
│  │
│  └─ いいえ ──→ ユーザーが明示的にトリガーすべきか？
│             ├─ はい ──→ COMMAND（ユーザーが /slash-command を入力）
│             └─ いいえ ──→ Claude が自動検出して呼び出すべきか？
│                        ├─ はい ──→ SKILL（検出用の description を記述）
│                        └─ いいえ ──→ 再利用可能なドメイン知識か？
│                                   ├─ はい ──→ SKILL（user-invocable: false でプリロード専用）
│                                   └─ いいえ ──→ COMMAND（シンプルなプロンプトテンプレート）
```

### クイック判断ルール

- **コンテキスト分離が必要？** → Agent
- **永続メモリが必要？** → Agent
- **バックグラウンド実行が必要？** → Agent
- **Worktree 分離が必要？** → Agent
- **カスタムツール制限が必要？** → Agent
- **ユーザー起動のワークフロー？** → Command
- **再利用可能な手順やドメイン知識？** → Skill
- **Claude による自動検出？** → Skill（または description に `PROACTIVELY` を含む Agent）
- **Agent へのプリロード？** → Skill

---

## 実践シナリオ (12)

### 1. コードレビュー

**推奨**: Agent

**理由**: コードレビューにはコンテキスト分離が必要です。レビュー結果がメイン会話を汚染しないようにするためです。Agent は独自のツール（読み取り専用）、モデル（高速な haiku）、永続メモリ（チーム固有のレビューパターンを学習）を持つことができます。

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: "PROACTIVELY review code changes for bugs, security, and style"
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
permissionMode: plan
---
```

### 2. 天気取得（Command → Agent → Skill チェーン）

**推奨**: Command が Agent をオーケストレーションし、Agent がプリロードされた Skill を使用

**理由**: ユーザーが明示的にトリガー（Command）し、取得ロジックは分離して実行（Agent）し、データ取得の手順は再利用可能なドメイン知識（Agent にプリロードされた Skill）です。

```
/weather-orchestrator  →  Agent(weather-agent)  →  Skill(weather-svg-creator)
     Command                   Agent                      Skill
```

### 3. ファイルスキャフォールディング

**推奨**: Skill

**理由**: スキャフォールディングテンプレートは再利用可能なドメイン知識です。ユーザーが新しいコンポーネント、ルート、モジュールの作成を依頼した際に、Claude が自動検出できます。分離は不要で、ファイルは現在のプロジェクトに直接書き込まれます。

```yaml
# .claude/skills/react-scaffold/SKILL.md
---
name: react-scaffold
description: "Scaffold React components with TypeScript, tests, and Storybook stories"
paths: "src/components/**"
---
```

### 4. CI/CD オーケストレーション

**推奨**: Command

**理由**: CI/CD ワークフローは常にユーザー起動（`/deploy`、`/release`）です。固定のシーケンスに従い、現在のセッションコンテキスト（ブランチ、最近の変更）へのアクセスが必要です。

```yaml
# .claude/commands/deploy.md
---
name: deploy
description: "Build, test, and deploy to staging or production"
argument-hint: "[staging|production]"
---
```

### 5. ブラウザテスト

**推奨**: Agent

**理由**: ブラウザテストは長時間実行され、バックグラウンド実行が必要な場合があり、メイン会話をブロックすべきではありません。Agent は特定の MCP サーバー（Playwright、Chrome DevTools）を持ち、Worktree で分離して実行できます。

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: "Run end-to-end browser tests against the dev server"
tools: Bash, Read, WebFetch, mcp__playwright__*
background: true
model: sonnet
---
```

### 6. 時刻表示

**推奨**: Skill

**理由**: 軽量で、分離不要、自動検出可能です。「今何時？」と聞かれた際に Claude が Skill を呼び出します。

### 7. マルチステップデプロイ

**推奨**: Command が Agent をオーケストレーション

**理由**: Command がユーザー向けのエントリポイント（`/full-deploy`）です。ビルド Agent、テスト Agent、デプロイ Agent をそれぞれのコンテキストと適切なツールで生成します。

### 8. Lint / フォーマット

**推奨**: Skill（自動呼び出し）

**理由**: Lint ルールは再利用可能なドメイン知識です。`paths:` によって Claude がマッチするファイルを編集した際に自動的にアクティブになります。ユーザーの操作は不要です。

```yaml
# .claude/skills/python-linting/SKILL.md
---
name: python-linting
description: "Apply Black formatting and ruff linting to Python files"
paths: "**/*.py"
---
```

### 9. リサーチアシスタント

**推奨**: Agent

**理由**: リサーチタスクは長時間実行され、メモリ（過去の調査の記憶）の恩恵を受け、Web ツール（`WebFetch`、`WebSearch`）が必要で、メイン会話の煩雑化を避けるために分離されたコンテキストで実行すべきです。

### 10. データマイグレーション

**推奨**: Worktree 分離を使用した Agent

**理由**: データベースマイグレーションは重要なファイルを変更するため、分離してテストすべきです。Git Worktree で実行することで、マイグレーションが検証されるまでメインブランチは影響を受けません。

```yaml
# .claude/agents/data-migrator.md
---
name: data-migrator
description: "Create and test database migration scripts"
isolation: worktree
model: sonnet
---
```

### 11. ドキュメント生成

**推奨**: Skill

**理由**: ドキュメント生成は再利用可能な手順です。「ドキュメントを書いて」や「API リファレンスを生成して」と依頼された際に Claude が自動検出できます。現在のコンテキストで実行し、プロジェクトに直接ファイルを書き込みます。

### 12. プロジェクトセットアップ

**推奨**: Command（Setup Hook と併用）

**理由**: プロジェクト初期化は常にユーザー起動で、一度だけ実行されます。`/setup` Command と `Setup` Hook を組み合わせて、初回クローン時に自動実行させます。

---

## 構成パターン

### Command → Agent → Skill

標準的なオーケストレーションパターンです。Command はユーザーのエントリポイント、Agent は分離とツール制御を提供、Skill はドメイン知識を提供します。

```
ユーザーが /command を入力
  └─ Command プロンプトが現在のコンテキストで実行
       └─ Agent(...) が分離されたコンテキストで生成
            └─ プリロードされた Skill が手順を提供
                 └─ Agent が完了し、結果が Command コンテキストに表示
                      └─ Skill ツールが最終出力のために呼び出される
```

### Agent にプリロードされた Skill

Agent が `skills:` フィールドによって起動時に Skill のコンテンツをロードします。Skill のコンテンツは Agent のシステムプロンプトに注入され、ユーザーが何も呼び出さなくてもドメインの専門知識が提供されます。

```yaml
# .claude/agents/api-builder.md
---
name: api-builder
skills:
  - openapi-validator
  - rest-conventions
  - error-handling-patterns
---
```

### コンテキストフォーク付き Skill

分離が必要だが Agent 定義の完全な機能は不要な Skill です。`context: fork` を設定すると、シンプルな Skill ファイル形式を維持しながら、サブエージェントコンテキストで Skill が実行されます。

```yaml
# .claude/skills/security-scan/SKILL.md
---
name: security-scan
description: "Scan the codebase for security vulnerabilities"
context: fork
agent: general-purpose
---
```

---

## メカニズム選択のアンチパターン

| アンチパターン | 問題 | より良いアプローチ |
|--------------|------|------------------|
| 単純なタスクに Agent を使用 | 手順の注入だけで済む場面でコンテキスト分離のオーバーヘッドが発生 | Skill を使用する |
| 自動検出のために Command を使用 | Command は Claude から自動呼び出しできない | `description` 付きの Skill を使用する |
| 長時間タスクに Skill を使用 | Skill はメインコンテキストで実行され会話をブロックする | `background: true` の Agent を使用する |
| 共有規約に Agent を使用 | Agent のコンテキストは分離されており、規約がメインセッションに引き継がれない | CLAUDE.md または Rules を使用する |
| プリロード可能な知識に Command を使用 | Command は Agent にプリロードできない | `user-invocable: false` の Skill を使用する |
| `description` のない Skill | Claude が自動検出できない | 常に意味のある description を記述する |

---

## ソース

- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Common Workflows — Docs](https://code.claude.com/docs/en/common-workflows)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
