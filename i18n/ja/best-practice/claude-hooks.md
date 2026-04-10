🌍 [English](../../../best-practice/claude-hooks.md) | [中文](../../zh/best-practice/claude-hooks.md) | **日本語** | [Français](../../fr/best-practice/claude-hooks.md) | [Русский](../../ru/best-practice/claude-hooks.md) | [العربية](../../ar/best-practice/claude-hooks.md)

# Hooks ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code の Hooks — 特定のライフサイクルイベントでエージェントループの外部で実行されるイベント駆動型ハンドラ。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 概要

Hooks はエージェントループの**外部**で実行されるユーザー定義ハンドラです — Claude の推論やツール呼び出しチェーンの一部ではありません。特定のライフサイクルイベントで発火し、スクリプトの実行、プロンプトの注入、Agent の起動、HTTP エンドポイントの呼び出しが可能です。会話コンテキストを汚染せずに、可観測性の追加、ポリシーの強制、副作用の自動化、意思決定のゲーティングに Hooks を使用します。

---

## Hook イベント (27)

### セッションライフサイクル

| イベント | 発火タイミング |
|---------|--------------|
| `SessionStart` | 新しいインタラクティブセッションが開始された時 |
| `SessionEnd` | セッションが終了しようとしている時 |
| `Setup` | Claude Code がプロジェクトで初めて実行された時（または `/init` の後） |

### ツールライフサイクル

| イベント | 発火タイミング |
|---------|--------------|
| `PreToolUse` | ツール呼び出しの実行前。呼び出しのブロック、許可、変更が可能 |
| `PostToolUse` | ツール呼び出しが正常に完了した後 |
| `PostToolUseFailure` | ツール呼び出しがエラーで失敗した後 |
| `PermissionRequest` | ユーザーが事前承認していないツールの権限を Claude がリクエストした時 |
| `PermissionDenied` | ツールの権限リクエストがユーザーまたはポリシーによって拒否された時 |

### Agent ライフサイクル

| イベント | 発火タイミング |
|---------|--------------|
| `SubagentStart` | Subagent（`Agent(...)` ツール経由）が実行を開始した時 |
| `SubagentStop` | Subagent が実行を完了した時 |
| `Stop` | **メイン** Agent がレスポンスターンを完了した時 |
| `StopFailure` | メイン Agent のレスポンスターンがエラーで失敗した時 |

### タスク / Teams

| イベント | 発火タイミング |
|---------|--------------|
| `TeammateIdle` | Agent Teams セッションのチームメイトがアイドル状態になり利用可能になった時 |
| `TaskCreated` | バックグラウンドタスクが作成された時 |
| `TaskCompleted` | バックグラウンドタスクが完了した時 |

### コンテキスト

| イベント | 発火タイミング |
|---------|--------------|
| `PreCompact` | コンテキストコンパクションの実行前 |
| `PostCompact` | コンテキストコンパクションの完了後 |
| `UserPromptSubmit` | ユーザーがプロンプトを送信した時（Claude が処理する前） |
| `Notification` | Claude が通知を送信した時（例：タスク完了、権限必要） |

### 設定 / 環境

| イベント | 発火タイミング |
|---------|--------------|
| `ConfigChange` | 設定ファイルが変更された時 |
| `CwdChanged` | 作業ディレクトリが変更された時（例：`/add-dir` 経由） |
| `FileChanged` | 監視対象ファイルがディスク上で変更された時 |
| `InstructionsLoaded` | CLAUDE.md またはルールファイルがコンテキストにロードされた時 |

### Worktree

| イベント | 発火タイミング |
|---------|--------------|
| `WorktreeCreate` | 分離された Agent 用の git worktree が作成された時 |
| `WorktreeRemove` | git worktree がクリーンアップされた時 |

### MCP / Elicitation

| イベント | 発火タイミング |
|---------|--------------|
| `Elicitation` | Claude が MCP elicitation 経由でユーザーから構造化入力を必要とする時 |
| `ElicitationResult` | ユーザーが MCP elicitation に応答した時 |

---

## Hook タイプ (4)

| タイプ | 構文 | 説明 |
|--------|------|------|
| `command` | `{"type": "command", "command": "python3 script.py"}` | シェルコマンドを実行。環境変数経由でイベントデータを受信。標準出力はコンテキストに注入される（`PreToolUse`/`Stop` の場合）。非ゼロの終了コードはアクションをブロック（`PreToolUse` の場合） |
| `prompt` | `{"type": "prompt", "prompt": "セキュリティ問題を確認してください"}` | Hook ポイントで Claude のコンテキストにプロンプトを注入。指示や制約の追加に便利 |
| `agent` | `{"type": "agent", "agent": "security-reviewer"}` | イベントを処理する Subagent を起動。Agent は独自のコンテキストで実行されツールを使用可能 |
| `http` | `{"type": "http", "url": "https://example.com/webhook"}` | POST 経由でイベントデータを HTTP エンドポイントに送信。レスポンスボディはコンテキストに注入される |

---

## 設定の階層

Hooks は複数のレベルで定義可能。上位レベルが同じイベントの下位レベルをオーバーライド:

| レベル | 場所 | スコープ |
|--------|------|---------|
| **マネージド** | `managed-settings.json`（MDM / Registry） | 組織が強制、オーバーライド不可 |
| **ローカル設定** | `.claude/settings.local.json` | 個人のプロジェクトオーバーライド（git 除外） |
| **共有設定** | `.claude/settings.json` | チーム共有のプロジェクト Hooks |
| **グローバル設定** | `~/.claude/settings.json` | 全プロジェクト横断の個人デフォルト |
| **Agent frontmatter** | `.claude/agents/<name>.md` `hooks:` フィールド | 特定の Subagent にスコープ |
| **Skill frontmatter** | `.claude/skills/<name>/SKILL.md` `hooks:` フィールド | 特定の Skill にスコープ |

すべての Hooks を無効にするには、`.claude/settings.local.json` で `"disableAllHooks": true` を設定。

---

## マッチャー構文

マッチャーは、どのツール呼び出しが Hook をトリガーするかをフィルタリングします。`PreToolUse`、`PostToolUse`、`PostToolUseFailure` イベントに適用されます。

| マッチャー | マッチ対象 |
|-----------|----------|
| `"Bash"` | すべての Bash ツール呼び出し |
| `"Read"` | すべての Read ツール呼び出し |
| `"Write"` | すべての Write ツール呼び出し |
| `"Edit"` | すべての Edit ツール呼び出し |
| `"Glob"` | すべての Glob ツール呼び出し |
| `"Grep"` | すべての Grep ツール呼び出し |
| `"Agent"` | すべての Agent（Subagent）ツール呼び出し |
| `"Skill"` | すべての Skill ツール呼び出し |
| `"WebFetch"` | すべての WebFetch ツール呼び出し |
| `"WebSearch"` | すべての WebSearch ツール呼び出し |
| `"mcp__<server>__<tool>"` | 特定サーバーの特定 MCP ツール |
| `"*"` | すべてのツール呼び出し（ワイルドカード） |

複数のマッチャーは配列で組み合わせ可能: `["Bash", "Write", "Edit"]`。

---

## 環境変数

command タイプの Hooks は環境変数経由でイベントデータを受信:

| 変数 | 利用可能場所 | 説明 |
|------|-------------|------|
| `CLAUDE_HOOK_EVENT` | すべて | イベント名（例：`PreToolUse`, `Stop`） |
| `CLAUDE_HOOK_TOOL_NAME` | ツールイベント | 呼び出されるツール（例：`Bash`, `Write`） |
| `CLAUDE_HOOK_TOOL_INPUT` | `PreToolUse` | ツールの入力パラメータの JSON 文字列 |
| `CLAUDE_HOOK_TOOL_OUTPUT` | `PostToolUse` | ツールの出力の JSON 文字列 |
| `CLAUDE_HOOK_ERROR` | 失敗イベント | 失敗した操作のエラーメッセージ |
| `CLAUDE_HOOK_SESSION_ID` | すべて | 現在のセッション識別子 |
| `CLAUDE_HOOK_CWD` | すべて | 現在の作業ディレクトリ |
| `CLAUDE_HOOK_PROJECT_DIR` | すべて | プロジェクトルートディレクトリ |
| `CLAUDE_HOOK_AGENT_NAME` | Agent イベント | 関与する Subagent の名前 |
| `CLAUDE_HOOK_MATCHED` | ツールイベント | この Hook をトリガーしたマッチャー |

---

## 実践例

### タスク完了時のサウンド通知

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "afplay ~/.claude/hooks/sounds/done.mp3",
        "async": true
      }
    ]
  }
}
```

### すべてのツール呼び出しのログ

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) $CLAUDE_HOOK_TOOL_NAME\" >> ~/.claude/logs/tools.log",
        "async": true
      }
    ]
  }
}
```

### セキュリティ監査 — 危険なコマンドのブロック

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "python3 .claude/hooks/scripts/security-check.py"
      }
    ]
  }
}
```

スクリプトは `CLAUDE_HOOK_TOOL_INPUT` に危険なパターン（`rm -rf /`、`curl | bash`、`chmod 777`）がないかチェックします。非ゼロの終了コードでツール呼び出しをブロックします。

### ファイル書き込み時の自動整形

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ["Write", "Edit"],
        "type": "command",
        "command": "prettier --write \"$CLAUDE_HOOK_TOOL_INPUT\" 2>/dev/null || true",
        "async": true
      }
    ]
  }
}
```

---

## 決定制御パターン

`PreToolUse` Hooks は標準出力または終了コード経由で決定シグナルを返せます:

| パターン | メカニズム | 効果 |
|---------|----------|------|
| **許可** | 終了コード `0`、標準出力に `ALLOW` を含む | 権限プロンプトなしでツール呼び出しが進行 |
| **拒否** | 非ゼロの終了コード | ツール呼び出しがブロックされ、エラーメッセージを表示 |
| **確認** | 終了コード `0`、標準出力に `ASK` を含む | 標準の権限プロンプトをユーザーに表示 |
| **委任** | 終了コード `0`、出力なしまたは空の標準出力 | デフォルトの権限処理にフォールスルー |

`prompt` と `agent` タイプの Hooks では、注入されたレスポンスが実行を直接制御するのではなく Claude の動作を誘導します。

---

## ベストプラクティス

1. **副作用には `async: true` を使用** — サウンド通知、ログ、テレメトリはエージェントループをブロックすべきではありません。Claude の動作に影響を与える必要のない Hooks には `"async": true` を設定します。

2. **セッション Hooks には `once: true` を使用** — セットアップ（依存関係のインストール、環境チェック）を行う `SessionStart` Hooks はセッションごとに1回だけ実行すべきです。

3. **タイムアウトを短く保つ** — 同期 Hooks は Claude のレスポンスをブロックします。Hook が2-3秒以上かかる場合、非同期にするかスクリプトを最適化します。

4. **マッチャーで Hooks をスコープする** — 特定のツールのみ必要な時にすべてのツール呼び出しで Hooks を実行するのを避けます。マッチャーを使用してオーバーヘッドを削減します。

5. **ゲーティングには `command`、ガイダンスには `prompt` を使用** — 許可/拒否のハードコントロールが必要な場合は、終了コード付きの command Hook を使用します。Claude のアプローチに影響を与えたい場合は prompt Hook を使用します。

6. **Hooks を分離してテスト** — サンプルの環境変数で Hook スクリプトを手動実行してから設定に追加します。

7. **Hook の失敗をログに記録** — Hook コマンドをファイルにログ出力するエラーハンドリングロジックでラップし、サイレントな失敗を検出可能にします。

---

## よくある落とし穴

| 落とし穴 | 詳細 |
|---------|------|
| **`Stop` と `SubagentStop` の混同** | `Stop` は**メイン** Agent のみで発火。Subagent の完了イベントには `SubagentStop` を使用。以前のバージョンでは `Stop` は `AgentStop` と呼ばれていた — これはリネームされた |
| **プロセス起動のオーバーヘッド** | 各 `command` Hook は新しいシェルプロセスを起動。ワイルドカードマッチャー付き `PostToolUse` のような高頻度イベントではレイテンシが増加。`async: true` の使用またはロジックの単一スクリプトへの統合を検討 |
| **マッチャーの設定忘れ** | `matcher` フィールドのない `PreToolUse` Hook は**すべて**のツール呼び出しで発火。意図的にユニバーサルカバレッジが必要でない限り、常にマッチャーを設定 |
| **同期 Hooks によるレスポンスブロック** | `Stop` 上の遅い同期 Hook はユーザーが Claude のレスポンスを見るのを遅延させる。重要でない Hooks は常に非同期に |
| **Hook 出力によるコンテキスト汚染** | 同期 command Hooks の標準出力は Claude のコンテキストに注入される。出力を最小限に抑えるかログファイルにリダイレクト |
| **設定階層の意外な挙動** | `.claude/settings.json` で定義された Hook は `.claude/settings.local.json` でオーバーライドされることがある。Hook が発火しない理由をデバッグする際は両方のファイルを確認 |

---

## ソース

- [Claude Code Hooks — ドキュメント](https://code.claude.com/docs/en/hooks)
- [Claude Code Hooks ガイド](https://code.claude.com/docs/en/hooks-guide)
- [Claude Code Settings — ドキュメント](https://code.claude.com/docs/en/settings)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
