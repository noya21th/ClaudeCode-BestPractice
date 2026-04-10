🌍 [English](../../../best-practice/claude-cli-startup-flags.md) | [中文](../../zh/best-practice/claude-cli-startup-flags.md) | **日本語** | [Français](../../fr/best-practice/claude-cli-startup-flags.md) | [Русский](../../ru/best-practice/claude-cli-startup-flags.md) | [العربية](../../ar/best-practice/claude-cli-startup-flags.md)

# CLI Startup Flags ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

ターミナルから Claude Code を起動する際の起動フラグ、トップレベルサブコマンド、起動環境変数のリファレンス。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 目次

1. [セッション管理](#セッション管理)
2. [モデルと設定](#モデルと設定)
3. [権限とセキュリティ](#権限とセキュリティ)
4. [出力とフォーマット](#出力とフォーマット)
5. [システムプロンプト](#システムプロンプト)
6. [Agent と Subagent](#agent-と-subagent)
7. [MCP と Plugins](#mcp-と-plugins)
8. [ディレクトリとワークスペース](#ディレクトリとワークスペース)
9. [バジェットと制限](#バジェットと制限)
10. [インテグレーション](#インテグレーション)
11. [初期化とメンテナンス](#初期化とメンテナンス)
12. [デバッグと診断](#デバッグと診断)
13. [設定オーバーライド](#設定オーバーライド)
14. [バージョンとヘルプ](#バージョンとヘルプ)
15. [サブコマンド](#サブコマンド)
16. [環境変数](#環境変数)

---

## セッション管理

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--continue` | `-c` | 現在のディレクトリで最新の会話を継続 |
| `--resume` | `-r` | ID または名前で特定のセッションを再開、またはインタラクティブピッカーを表示 |
| `--from-pr <NUMBER\|URL>` | | 特定の GitHub PR にリンクされたセッションを再開 |
| `--fork-session` | | 再開時に新しいセッション ID を作成（`--resume` または `--continue` と併用） |
| `--session-id <UUID>` | | 特定のセッション ID を使用（有効な UUID が必要） |
| `--no-session-persistence` | | セッション永続化を無効化（print mode のみ） |
| `--remote` | | claude.ai で新しい Web セッションを作成 |
| `--teleport` | | Web セッションをローカルターミナルで再開 |

---

## モデルと設定

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--model <NAME>` | | エイリアス（`sonnet`, `opus`, `haiku`）または完全なモデル ID でモデルを設定 |
| `--fallback-model <NAME>` | | デフォルトが過負荷時の自動フォールバックモデル（print mode のみ） |
| `--betas <LIST>` | | API リクエストに含めるベータヘッダー（API キーユーザーのみ） |

---

## 権限とセキュリティ

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--dangerously-skip-permissions` | | すべての権限プロンプトをスキップ。極めて慎重に使用すること |
| `--allow-dangerously-skip-permissions` | | 権限バイパスをオプションとして有効化（実際には有効化しない） |
| `--permission-mode <MODE>` | | 指定された権限モードで開始: `default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <TOOLS>` | | プロンプトなしで実行されるツール（権限ルール構文） |
| `--disallowedTools <TOOLS>` | | モデルコンテキストから完全に除外されるツール |
| `--tools <TOOLS>` | | Claude が使用できる組み込みツールを制限（`""` ですべて無効化） |
| `--permission-prompt-tool <TOOL>` | | 非インタラクティブモードで権限プロンプトを処理する MCP ツールを指定 |

---

## 出力とフォーマット

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--print` | `-p` | インタラクティブモードなしでレスポンスを出力（ヘッドレス/SDK モード） |
| `--output-format <FORMAT>` | | 出力形式: `text`, `json`, `stream-json` |
| `--input-format <FORMAT>` | | 入力形式: `text`, `stream-json` |
| `--json-schema <SCHEMA>` | | スキーマに一致する検証済み JSON を取得（print mode のみ） |
| `--include-partial-messages` | | 部分的なストリーミングイベントを含める（`--print` と `--output-format=stream-json` が必要） |
| `--verbose` | | 完全なターンバイターン出力による詳細ログを有効化 |

---

## システムプロンプト

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--system-prompt <TEXT>` | | システムプロンプト全体をカスタムテキストで置換 |
| `--system-prompt-file <PATH>` | | ファイルからシステムプロンプトをロードしデフォルトを置換（print mode のみ） |
| `--append-system-prompt <TEXT>` | | デフォルトのシステムプロンプトにカスタムテキストを追加 |
| `--append-system-prompt-file <PATH>` | | デフォルトプロンプトにファイル内容を追加（print mode のみ） |

---

## Agent と Subagent

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--agent <NAME>` | | 現在のセッションの Agent を指定 |
| `--agents <JSON>` | | JSON 経由でカスタム Subagents を動的に定義 |
| `--teammate-mode <MODE>` | | Agent Teams の表示を設定: `auto`, `in-process`, `tmux` |

---

## MCP と Plugins

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--mcp-config <PATH\|JSON>` | | JSON ファイルまたは文字列から MCP Servers をロード |
| `--strict-mcp-config` | | `--mcp-config` の MCP Servers のみ使用、他はすべて無視 |
| `--plugin-dir <PATH>` | | このセッションのみディレクトリから Plugins をロード（繰り返し可能） |

---

## ディレクトリとワークスペース

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--add-dir <PATH>` | | Claude がアクセスできる追加の作業ディレクトリを追加 |
| `--worktree` | `-w` | 分離された git worktree で Claude を開始（HEAD からブランチ） |

---

## バジェットと制限

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--max-budget-usd <AMOUNT>` | | 停止前の API コールの最大ドル金額（print mode のみ） |
| `--max-turns <NUMBER>` | | エージェントターン数の制限（print mode のみ） |

---

## インテグレーション

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--chrome` | | Web 自動化のための Chrome ブラウザ連携を有効化 |
| `--no-chrome` | | このセッションの Chrome ブラウザ連携を無効化 |
| `--ide` | | 起動時に有効な IDE が1つだけの場合、自動的に IDE に接続 |

---

## 初期化とメンテナンス

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--init` | | 初期化 Hooks を実行しインタラクティブモードを開始 |
| `--init-only` | | 初期化 Hooks を実行して終了（インタラクティブセッションなし） |
| `--maintenance` | | メンテナンス Hooks を実行して終了 |

---

## デバッグと診断

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--debug <CATEGORIES>` | | オプションのカテゴリフィルタリング付きデバッグモードを有効化（例：`"api,hooks"`） |

---

## 設定オーバーライド

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--settings <PATH\|JSON>` | | 設定 JSON ファイルまたは JSON 文字列のパス |
| `--setting-sources <LIST>` | | ロードするソースのカンマ区切りリスト: `user`, `project`, `local` |
| `--disable-slash-commands` | | このセッションのすべての Skills とスラッシュコマンドを無効化 |

---

## バージョンとヘルプ

| フラグ | 短縮形 | 説明 |
|--------|--------|------|
| `--version` | `-v` | バージョン番号を出力 |
| `--help` | `-h` | ヘルプ情報を表示 |

---

## サブコマンド

`claude <subcommand>` として実行するトップレベルコマンド:

| サブコマンド | 説明 |
|-------------|------|
| `claude` | インタラクティブ REPL を開始 |
| `claude "query"` | 初期プロンプト付きで REPL を開始 |
| `claude agents` | 設定済み Agents の一覧 |
| `claude auth` | Claude Code の認証管理 |
| `claude doctor` | コマンドラインから診断を実行 |
| `claude install` | Claude Code のネイティブビルドをインストールまたは切り替え |
| `claude mcp` | MCP Servers の設定（`add`, `remove`, `list`, `get`, `enable`） |
| `claude plugin` | Claude Code Plugins の管理 |
| `claude remote-control` | リモートコントロールセッションの管理 |
| `claude setup-token` | サブスクリプション使用のための長期トークンを作成 |
| `claude update` / `claude upgrade` | 最新バージョンにアップデート |

---

## 環境変数

これらの起動時専用環境変数は、Claude Code を起動する前にシェルで設定します（`settings.json` では設定できません）:

| 変数 | 説明 |
|------|------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 実験的な Agent Teams を有効化 |
| `CLAUDE_CODE_TMPDIR` | 内部ファイル用の一時ディレクトリをオーバーライド。`env` キーでも設定可能 — [Settings リファレンス](../../../best-practice/claude-settings.md#environment-variables-via-env)を参照 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 追加ディレクトリの CLAUDE.md ローディングを有効化 |
| `DISABLE_AUTOUPDATER=1` | 自動アップデートを無効化 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 思考の深さを制御 — [Settings リファレンス](../../../best-practice/claude-settings.md#environment-variables-via-env)を参照 |
| `USE_BUILTIN_RIPGREP=0` | 組み込みの代わりにシステムの ripgrep を使用（Alpine Linux） |
| `CLAUDE_CODE_SIMPLE` | シンプルモードを有効化（Bash + Edit ツールのみ）。`env` キーでも設定可能 — [Settings リファレンス](../../../best-practice/claude-settings.md#environment-variables-via-env)を参照 |
| `CLAUDE_BASH_NO_LOGIN=1` | BashTool でログインシェルをスキップ |

`settings.json` の `"env"` キーで設定可能な環境変数（`MAX_THINKING_TOKENS`、`CLAUDE_CODE_SHELL`、`CLAUDE_CODE_ENABLE_TASKS`、`CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`、`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` など）については、[Claude Settings リファレンス](../../../best-practice/claude-settings.md#environment-variables-via-env)を参照してください。

---

## ソース

- [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
- [Claude Code ヘッドレスモード](https://code.claude.com/docs/en/headless)
- [Claude Code セットアップ](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code 一般的なワークフロー](https://code.claude.com/docs/en/common-workflows)
