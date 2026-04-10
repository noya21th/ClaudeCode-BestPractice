🌍 [English](../../../best-practice/claude-commands.md) | [中文](../../zh/best-practice/claude-commands.md) | **日本語** | [Français](../../fr/best-practice/claude-commands.md) | [Русский](../../ru/best-practice/claude-commands.md) | [العربية](../../ar/best-practice/claude-commands.md)

# Commands ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2008%2C%202026%209%3A35%20PM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../../../implementation/claude-commands-implementation.md)

Claude Code の Commands — frontmatter フィールドと公式組み込みスラッシュコマンド。

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
| `description` | string | 推奨 | コマンドの機能説明。オートコンプリートに表示され、Claude の自動検出に使用 |
| `argument-hint` | string | いいえ | オートコンプリート時に表示されるヒント（例：`[issue-number]`, `[filename]`） |
| `disable-model-invocation` | boolean | いいえ | `true` に設定すると Claude がこのコマンドを自動呼び出しするのを防止 |
| `user-invocable` | boolean | いいえ | `false` に設定すると `/` メニューから非表示 — コマンドはバックグラウンドナレッジのみ |
| `paths` | string/list | いいえ | この Skill がアクティベーションされる条件を制限する Glob パターン。カンマ区切りの文字列または YAML リストを受け付け、一致するファイルを操作中のみ Claude が Skill を自動ロード |
| `allowed-tools` | string | いいえ | このコマンドがアクティブな時に権限プロンプトなしで許可されるツール |
| `model` | string | いいえ | このコマンド実行時に使用するモデル（例：`haiku`, `sonnet`, `opus`） |
| `effort` | string | いいえ | 呼び出し時のモデル努力レベルオーバーライド（`low`, `medium`, `high`, `max`） |
| `context` | string | いいえ | `fork` に設定すると分離された Subagent コンテキストでコマンドを実行 |
| `agent` | string | いいえ | `context: fork` 設定時の Subagent タイプ（デフォルト: `general-purpose`） |
| `shell` | string | いいえ | `` !`command` `` ブロック用のシェル — `bash`（デフォルト）または `powershell`。`CLAUDE_CODE_USE_POWERSHELL_TOOL=1` が必要 |
| `hooks` | object | いいえ | このコマンドにスコープされたライフサイクル Hooks |

---

## ![Official](../../../!/tags/official.svg) **(65)**

| # | コマンド | タグ | 説明 |
|---|---------|------|------|
| 1 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic アカウントにサインイン |
| 2 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic アカウントからサインアウト |
| 3 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Amazon Bedrock の認証、リージョン、モデルピンをインタラクティブウィザードで設定。`CLAUDE_CODE_USE_BEDROCK=1` 設定時のみ表示。初回 Bedrock ユーザーはログイン画面からもアクセス可能 |
| 4 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | アップグレードページを開き、上位プランに切り替え |
| 5 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 現在のセッションのプロンプトバーの色を設定。使用可能な色: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`。`default` でリセット |
| 6 | `/config` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Settings インターフェースを開き、テーマ、モデル、出力スタイルなどを調整。エイリアス: `/settings` |
| 7 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | キーバインド設定ファイルを開くまたは作成 |
| 8 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | ツール権限の許可、確認、拒否ルールを管理。スコープ別のルール表示、ルールの追加・削除、作業ディレクトリの管理、最近の auto mode 拒否の確認ができるインタラクティブダイアログ。エイリアス: `/allowed-tools` |
| 9 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | プライバシー設定の表示と更新。Pro および Max プラン加入者のみ利用可能 |
| 10 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | サンドボックスモードの切り替え。対応プラットフォームのみ利用可能 |
| 11 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code の Status Line を設定。要望を記述するか、引数なしで実行してシェルプロンプトから自動設定 |
| 12 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code ステッカーを注文 |
| 13 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Shift+Enter などのターミナルキーバインドを設定。VS Code、Alacritty、Warp など必要なターミナルでのみ表示 |
| 14 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | カラーテーマを変更。ライト・ダークバリアント、色覚対応（ダルトン化）テーマ、ターミナルのカラーパレットを使用する ANSI テーマを含む |
| 15 | `/voice` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | プッシュトゥトーク音声入力の切り替え。Claude.ai アカウントが必要 |
| 16 | `/context` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 現在のコンテキスト使用量をカラーグリッドで可視化。コンテキストを多く消費するツール、メモリ肥大化、容量警告に対する最適化提案を表示 |
| 17 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | トークン使用統計を表示。サブスクリプション固有の詳細はコスト追跡ガイドを参照 |
| 18 | `/extra-usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | レート制限に達した時も作業を続けるためのエクストラ使用量を設定 |
| 19 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Claude Code セッションの分析レポートを生成。プロジェクト領域、インタラクションパターン、フリクションポイントを含む |
| 20 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 日次使用量、セッション履歴、連続記録、モデル設定を可視化 |
| 21 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | Settings インターフェース（Status タブ）を開き、バージョン、モデル、アカウント、接続状態を表示。現在のレスポンスを待たずに動作 |
| 22 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | プラン使用制限とレート制限状態を表示 |
| 23 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Claude Code のインストールと設定を診断・検証 |
| 24 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Claude Code についてフィードバックを送信。エイリアス: `/bug` |
| 25 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | ヘルプと利用可能なコマンドを表示 |
| 26 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | アニメーションデモ付きのクイックインタラクティブレッスンで Claude Code の機能を発見 |
| 27 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | インタラクティブなバージョンピッカーで変更履歴を表示。特定バージョンのリリースノートを選択するか、全バージョンを表示 |
| 28 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | バックグラウンドタスクの一覧と管理。エイリアス: `/bashes` |
| 29 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 最後のアシスタントレスポンスをクリップボードにコピー。数字 `N` を渡すと N 番目に新しいレスポンスをコピー: `/copy 2` で2番目に新しいものをコピー。コードブロックがある場合、個別ブロックまたは全レスポンスを選択するインタラクティブピッカーを表示。ピッカーで `w` を押すとクリップボードの代わりにファイルに書き出し（SSH 時に便利） |
| 30 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 現在の会話をプレーンテキストとしてエクスポート。ファイル名を指定すると直接そのファイルに書き込み。指定なしではクリップボードにコピーまたはファイル保存のダイアログを表示 |
| 31 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Agent 設定の管理 |
| 32 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude in Chrome の設定を構成 |
| 33 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | ツールイベントの Hook 設定を表示 |
| 34 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | IDE 連携の管理とステータス表示 |
| 35 | `/mcp` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | MCP Server 接続と OAuth 認証の管理 |
| 36 | `/plugin` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude Code Plugins の管理 |
| 37 | `/reload-plugins` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | すべてのアクティブ Plugins をリロードして保留中の変更を再起動なしで適用。リロードされた各コンポーネントの数を報告し、ロードエラーをフラグ |
| 38 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 利用可能な Skills の一覧 |
| 39 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | `CLAUDE.md` メモリファイルの編集、自動メモリの有効化/無効化、自動メモリエントリの表示 |
| 40 | `/effort [low\|medium\|high\|max\|auto]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | モデルの努力レベルを設定。`low`, `medium`, `high` はセッション間で永続。`max` は現在のセッションのみで Opus 4.6 が必要。`auto` でモデルデフォルトにリセット。引数なしで現在のレベルを表示。現在のレスポンスを待たずに即座に有効 |
| 41 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 高速モードのオン/オフ切り替え |
| 42 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | AI モデルの選択または変更。対応モデルでは左右矢印で努力レベルを調整。現在のレスポンスを待たずに即座に有効 |
| 43 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | Claude Code の無料1週間を友人とシェア。アカウントが対象の場合のみ表示 |
| 44 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | プロンプトから直接 plan mode に入る。オプションの説明を渡すと plan mode に入って即座にそのタスクを開始（例：`/plan 認証バグを修正`） |
| 45 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | ultraplan セッションでプランを作成し、ブラウザでレビュー、リモートで実行またはターミナルに送り返す |
| 46 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 現在のセッションに新しい作業ディレクトリを追加 |
| 47 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | コミットされていない変更とターンごとの差分を表示するインタラクティブ diff ビューアを開く。左右矢印で現在の git diff と個別の Claude ターンを切り替え、上下でファイルを閲覧 |
| 48 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | `CLAUDE.md` ガイドでプロジェクトを初期化。`CLAUDE_CODE_NEW_INIT=1` を設定すると Skills、Hooks、個人メモリファイルのウォークスルーも含むインタラクティブフローを実行 |
| 49 | `/review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 非推奨。代わりに `code-review` Plugin をインストール: `claude plugin install code-review@claude-plugins-official` |
| 50 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 現在のブランチの保留中の変更をセキュリティ脆弱性について分析。git diff をレビューし、インジェクション、認証問題、データ漏洩などのリスクを特定 |
| 51 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Code Desktop アプリで現在のセッションを継続。macOS と Windows のみ。エイリアス: `/app` |
| 52 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | リポジトリ用の Claude GitHub Actions アプリをセットアップ。リポジトリの選択と連携の設定をガイド |
| 53 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Slack アプリをインストール。ブラウザを開いて OAuth フローを完了 |
| 54 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude モバイルアプリのダウンロード用 QR コードを表示。エイリアス: `/ios`, `/android` |
| 55 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | このセッションを claude.ai からのリモートコントロールに対応させる。エイリアス: `/rc` |
| 56 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | `--remote` で開始される Web セッションのデフォルトリモート環境を設定 |
| 57 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | クラウドのスケジュールタスクを作成、更新、一覧表示、実行。Claude が会話形式でセットアップをガイド |
| 58 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | この時点で現在の会話のブランチを作成。エイリアス: `/fork` |
| 59 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 会話に追加せずにサイドクエスチョンを素早く質問 |
| 60 | `/clear` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 会話履歴をクリアしてコンテキストを解放。エイリアス: `/reset`, `/new` |
| 61 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | オプションのフォーカス指示付きで会話をコンパクト化 |
| 62 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | CLI を終了。エイリアス: `/quit` |
| 63 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 現在のセッションの名前を変更してプロンプトバーに表示。名前なしの場合は会話履歴から自動生成 |
| 64 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | ID または名前で会話を再開、またはセッションピッカーを開く。エイリアス: `/continue` |
| 65 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 会話やコードを以前の時点に巻き戻し、または選択したメッセージから要約。Checkpointing を参照。エイリアス: `/checkpoint` |

`/debug` などのバンドル Skills もスラッシュコマンドメニューに表示されることがありますが、組み込みコマンドではありません。

---

## ソース

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code インタラクティブモード](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
