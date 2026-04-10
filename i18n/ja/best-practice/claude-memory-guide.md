🌍 [English](../../../best-practice/claude-memory-guide.md) | [中文](../../zh/best-practice/claude-memory-guide.md) | **日本語** | [Français](../../fr/best-practice/claude-memory-guide.md) | [Русский](../../ru/best-practice/claude-memory-guide.md) | [العربية](../../ar/best-practice/claude-memory-guide.md)

# メモリ & コンテキスト管理ガイド

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code のメモリシステムに関する包括的ガイド — CLAUDE.md ファイル、自動メモリ、Agent メモリ、Rules。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## メモリの種類の概要

Claude Code には 4 つの異なるメモリシステムがあり、それぞれ異なる目的を果たします：

| メモリの種類 | 保存場所 | スコープ | 永続性 | 読み込みタイミング |
|-------------|---------|--------|--------|-----------------|
| **CLAUDE.md** | プロジェクトディレクトリ | プロジェクト単位、階層的 | 永続（ファイルベース） | セッション開始時（祖先）またはファイルアクセス時（子孫） |
| **自動メモリ** | `~/.claude/memory/` および `~/.claude/projects/<hash>/memory/` | ユーザーグローバルまたはプロジェクト単位 | 永続（ファイルベース） | セッション開始時 |
| **Agent メモリ** | `~/.claude/agent-memory/<agent>/` または `.claude/agent-memory/<agent>/` | Agent 単位、`memory:` フィールドでスコープ指定 | 永続（ファイルベース） | Agent 起動時 |
| **Rules** | `.claude/rules/*.md` および `~/.claude/rules/*.md` | ファイルパターン単位 | 永続（ファイルベース） | glob パターンにマッチするファイルがアクセスされた時 |

---

## CLAUDE.md の階層構造

### ファイルの配置

| レベル | パス | 共有 | 説明 |
|-------|------|------|------|
| **グローバル** | `~/.claude/CLAUDE.md` | いいえ（個人用） | このマシンのすべての Claude Code セッションに適用 |
| **プロジェクトルート** | `./CLAUDE.md` | はい（git 追跡） | チーム共有のプロジェクト全体の指示 |
| **プロジェクトローカル** | `./CLAUDE.local.md` | いいえ（git 除外） | 個人用のプロジェクト固有オーバーライド |
| **コンポーネント** | `./frontend/CLAUDE.md` | はい（git 追跡） | コンポーネント固有の指示 |
| **ネスト** | `./src/api/v2/CLAUDE.md` | はい（git 追跡） | 深いディレクトリ固有の指示 |

### 読み込みメカニズム

Claude Code は CLAUDE.md ファイルの検出に 2 つの異なるメカニズムを使用します：

#### 祖先ロード（ツリーを上方へ） — 即時

セッション開始時に、Claude は現在の作業ディレクトリからファイルシステムルートまで**上方に**走査し、見つかったすべての CLAUDE.md を読み込みます。これらは**起動時に即座に**読み込まれます。

```
/home/user/projects/myapp/src/   ← cwd
/home/user/projects/myapp/       ← CLAUDE.md 読み込み（祖先）
/home/user/projects/             ← CLAUDE.md 読み込み（祖先）
/home/user/                      ← CLAUDE.md があれば読み込み（祖先）
~/.claude/                       ← CLAUDE.md 読み込み（グローバル）
```

#### 子孫ロード（ツリーを下方へ） — 遅延

cwd 配下のサブディレクトリにある CLAUDE.md ファイルは起動時には**読み込まれません**。セッション中に Claude がそのサブディレクトリのファイルを読み取りまたは編集した際にのみ読み込まれます。

```
/myapp/                 ← cwd、CLAUDE.md は起動時に読み込み
/myapp/frontend/        ← CLAUDE.md は Claude が frontend/ のファイルにアクセスした時に読み込み
/myapp/backend/         ← CLAUDE.md は Claude が backend/ のファイルにアクセスした時に読み込み
/myapp/backend/api/     ← CLAUDE.md は Claude が backend/api/ のファイルにアクセスした時に読み込み
```

#### 主要ルール

- **祖先は常に起動時に読み込まれる** — cwd から上方に即時ロード
- **子孫は遅延ロード** — そのディレクトリのファイルがアクセスされた時のみ
- **兄弟は読み込まれない** — `frontend/` で作業中に `backend/CLAUDE.md` は読み込まれない
- **グローバルは常に読み込まれる** — `~/.claude/CLAUDE.md` はすべてのセッションに適用

---

## モノレポ戦略

### ルート CLAUDE.md — 共有規約

ルートファイルはどこでも適用されるチーム全体の標準に絞る：

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

### コンポーネント CLAUDE.md — 詳細

各コンポーネントが独自の焦点を絞った指示を持つ：

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

### CLAUDE.local.md — 個人設定

コミットすべきでない個人的なオーバーライド：

```markdown
# CLAUDE.local.md (git-ignored)

- Use verbose comments — I am learning this codebase
- Run prettier after every file edit
- Always explain your reasoning before making changes
```

---

## 自動メモリシステム

自動メモリは、Claude Code が会話を跨いで情報を記憶するためのシステムです。セッション間で永続する観察と学習を保存します。

### 種類

| 種類 | 保存場所 | 説明 |
|------|---------|------|
| `user` | `~/.claude/memory/` | ユーザーの好みとグローバルなパターン — すべてのプロジェクトに適用 |
| `feedback` | `~/.claude/memory/` | ユーザーのフィードバックからの修正と行動調整 |
| `project` | `~/.claude/projects/<hash>/memory/` | プロジェクト固有の学習とコンテキスト |
| `reference` | `~/.claude/projects/<hash>/memory/` | このプロジェクトで Claude が有用と判断した参考資料 |

### 仕組み

1. Claude がセッション中にパターン、好み、または修正を観察する
2. その観察が将来のセッションで有用になりそうであれば、自動メモリとして保存する
3. 次のセッション開始時に、関連する自動メモリエントリがコンテキストに読み込まれる
4. ユーザーは `/memory` でエントリを確認・管理できる

### 自動メモリと CLAUDE.md の使い分け

| 自動メモリを使うとき | CLAUDE.md を使うとき |
|-------------------|-------------------|
| セッション中のフィードバックから生まれた指示 | 事前に計画された指示 |
| 個人の好み（トーン、詳細度、ワークフロー） | チームの規約（コーディング標準、アーキテクチャ） |
| 使用を通じて時間とともに進化する | 安定しており、コードレビューで検証される |
| プロジェクトを跨いでユーザーに適用する | ユーザーに関係なくプロジェクトに適用する |

---

## Agent メモリ

frontmatter に `memory:` フィールドを持つ Agent は、セッションを跨いで永続するメモリを維持します。これにより Agent が時間とともに学習し改善できます。

### スコープ

| スコープ | 保存場所 | 共有 | 説明 |
|---------|---------|------|------|
| `user` | `~/.claude/agent-memory/<agent>/` | いいえ | この Agent のプロジェクト横断メモリ — ユーザーに従う |
| `project` | `.claude/agent-memory/<agent>/` | はい | プロジェクト固有のメモリ — git 経由でチームと共有 |
| `local` | `.claude/agent-memory/<agent>/`（git 除外） | いいえ | プロジェクト固有の個人メモリ |

### 設定

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

### ユースケース

- **コードレビューア**（`memory: project`） — チーム固有のパターン、よくあるバグ、スタイルの好みを学習
- **リサーチアシスタント**（`memory: user`） — 調査したトピック、好みのソース、ライティングスタイルを記憶
- **デプロイ Agent**（`memory: project`） — デプロイ履歴、既知の環境固有の問題を追跡

---

## Rules（.claude/rules/）

Rules は glob スコープのマークダウンファイルで、Claude が glob パターンにマッチするファイルにアクセスした際に自動的に読み込まれます。

### 配置場所

| パス | スコープ |
|------|--------|
| `.claude/rules/*.md` | プロジェクトレベルの Rules（git 追跡、チーム共有） |
| `~/.claude/rules/*.md` | グローバル Rules（個人用、すべてのプロジェクトに適用） |

### 形式

各 Rules ファイルは `# Glob:` 行で始まり、いつアクティブになるかを定義します：

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

### Rules と CLAUDE.md の使い分け

| Rules を使うとき | CLAUDE.md を使うとき |
|-----------------|-------------------|
| 特定のファイルタイプに適用する指示 | プロジェクト全体に適用する指示 |
| ファイルアクセスに基づく遅延ロードが望ましい | セッション開始時の即時ロードが望ましい |
| パターン（`*.py`、`*.tsx`、`migrations/*.sql`）にスコープされたガイドライン | 普遍的なガイドライン（git 規約、アーキテクチャ決定） |

---

## 比較表

| 機能 | CLAUDE.md | 自動メモリ | Agent メモリ | Rules |
|------|-----------|----------|-------------|-------|
| **作成者** | 開発者（手動） | Claude（自動） | Agent（自動） | 開発者（手動） |
| **チームと共有** | はい（`.local.md` 以外） | いいえ | はい（`project` スコープの場合） | はい |
| **起動時に読み込み** | はい（祖先） | はい | はい（Agent 起動時） | いいえ（遅延） |
| **ファイルタイプ限定** | いいえ | いいえ | いいえ | はい（glob パターン） |
| **ユーザーが編集可能** | はい（ファイル編集） | はい（`/memory`） | はい（ファイル編集） | はい（ファイル編集） |
| **セッション間で永続** | はい | はい | はい | はい |
| **最適な用途** | プロジェクトの規約 | 学習された好み | Agent の改善 | ファイルタイプ固有のガイダンス |

---

## ベストプラクティス

### CLAUDE.md は 200 行以内に

長い CLAUDE.md ファイルはコンテキストの肥大化を招き、Claude がファイル末尾付近の指示を無視する可能性があります。大きなファイルはコンポーネントレベルの CLAUDE.md と Rules に分割してください。

### ファイルタイプのガイダンスには Rules を使用

CLAUDE.md に「Python ファイル編集時は X を行う」と書かないでください。代わりに `.claude/rules/python.md` を作成し `# Glob: **/*.py` を指定してください。必要な時にのみ読み込まれます。

### 進化する Agent には Agent メモリを使用

Agent が時間とともに賢くなるべきであれば、`memory:` スコープを付与してください。Agent は将来の実行を改善する観察を保存します — レビューパターン、コードの問題、デプロイの特異性など。

### コンテキスト使用量 50% でコンパクト

定期的に `/context` で使用量を確認してください。50% に達したら、オプションのフォーカス指示付きで `/compact` を実行します：`/compact focus on the authentication refactor`。重要なコンテキストを保持しながらスペースを解放します。

### 個人ファイルを git 除外

`.gitignore` に必ず追加：

```
CLAUDE.local.md
.claude/settings.local.json
```

### 安定した指示と進化する指示を分離

- **安定**（コーディング標準、アーキテクチャ） → CLAUDE.md と Rules
- **進化**（好み、パターン、学習） → 自動メモリと Agent メモリ

---

## 陳腐化とクリーンアップ

### 陳腐化メモリの兆候

- CLAUDE.md が非推奨の API や削除された機能を参照している
- 自動メモリのエントリが現在の CLAUDE.md の指示と矛盾している
- Agent メモリに古くなったプロジェクト固有のパターンが含まれている
- Rules がコードベースに存在しないファイルパターンを参照している

### クリーンアップ戦略

1. **CLAUDE.md を四半期ごとに監査** — 各指示を読み、まだ適用されるか確認する
2. **自動メモリを確認** — `/memory` を実行し、もう関連のないエントリを削除する
3. **Agent メモリを整理** — `.claude/agent-memory/` で陳腐化した観察を確認する
4. **Rules を更新** — glob パターンがまだプロジェクトのファイルにマッチするか確認する
5. **コメントで追跡** — CLAUDE.md のセクションに `<!-- Last reviewed: 2026-04-09 -->` を追加する

---

## ソース

- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code Rules — Docs](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)
- [Claude Code Sub-agents (Memory) — Docs](https://code.claude.com/docs/en/sub-agents)
- [CLAUDE.md in Monorepos](../best-practice/claude-memory.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
