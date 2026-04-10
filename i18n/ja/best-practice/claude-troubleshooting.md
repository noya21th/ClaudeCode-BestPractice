🌍 [English](../../../best-practice/claude-troubleshooting.md) | [中文](../../zh/best-practice/claude-troubleshooting.md) | **日本語** | [Français](../../fr/best-practice/claude-troubleshooting.md) | [Русский](../../ru/best-practice/claude-troubleshooting.md) | [العربية](../../ar/best-practice/claude-troubleshooting.md)

# トラブルシューティングガイド

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2009%2C%202026-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.96-blue?style=flat&labelColor=555)

Claude Code でよくある問題の診断ガイド — 症状別にステップバイステップの解決策を整理。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## クイック診断

何かうまくいかない場合、まず以下のコマンドを実行してください：

| コマンド | チェック内容 |
|---------|------------|
| `claude --version` | インストール済みバージョン — [最新リリース](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)と比較 |
| `/doctor` | インストール状態、Node.js バージョン、認証、設定の妥当性 |
| `/status` | 現在のモデル、アカウント、API 接続性、サブスクリプションプラン |
| `/usage` | プラン上限、レート制限状況、残りクォータ |
| `/context` | コンテキストウィンドウ使用量、読み込み済みメモリファイル、アクティブな Skill |
| `/hooks` | アクティブな Hook 設定とそのソース |
| `/mcp` | MCP サーバーの接続状況 |

---

## インストールの問題

### Claude Code のインストールに失敗する

**症状**: `npm install -g @anthropic-ai/claude-code` が権限エラーまたはバージョンの競合で失敗する。

**原因**: Node.js のバージョンが古い、npm の権限に問題がある、またはグローバルパッケージが競合している。

**解決策**:

```bash
# Check Node.js version (requires 18+)
node --version

# If version is too old, update via nvm
nvm install 20
nvm use 20

# Fix npm permissions (macOS/Linux)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH  # Add to ~/.zshrc or ~/.bashrc

# Clean install
npm install -g @anthropic-ai/claude-code
```

### claude コマンドが見つからない

**症状**: インストール成功後にターミナルが `claude: command not found` と表示する。

**原因**: npm のグローバル bin ディレクトリが PATH に含まれていない。

**解決策**:

```bash
# Find where npm installs global binaries
npm config get prefix

# Add to your shell profile (~/.zshrc, ~/.bashrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.zshrc
```

---

## 認証の問題

### API Key が認識されない

**症状**: Claude Code は起動するが API 呼び出しができない。認証または無効なキーに関するエラーメッセージが表示される。

**原因**: API Key が見つからないか期限切れ。環境変数が間違っている。

**解決策**:

```bash
# Re-authenticate
claude /login

# Or set the API key directly
export ANTHROPIC_API_KEY="sk-ant-..."

# For Bedrock users
export CLAUDE_CODE_USE_BEDROCK=1
claude /setup-bedrock
```

### レート制限エラー

**症状**: `429 Too Many Requests` エラー。Claude がタスク途中で応答を停止する。

**原因**: プランのレート制限を超過した。同時実行セッションまたは Agent が多すぎる。

**解決策**:

```bash
# Check current usage
/usage

# Options:
# 1. Wait for rate limit to reset (check Retry-After header)
# 2. Switch to a lighter model for high-volume tasks
/model haiku

# 3. Reduce concurrent agents
# 4. Enable extra usage
/extra-usage
```

---

## パフォーマンスの問題

### 応答が遅い

**症状**: Claude の応答に 30 秒以上かかる。ツール呼び出し間に長い停止がある。

**原因**: コンテキストウィンドウが大きすぎる。読み込まれた Skill が多すぎる。同期 Hook が応答をブロックしている。

**解決策**:

```bash
# Check context usage
/context

# Compact if above 50%
/compact

# Check for slow synchronous hooks
/hooks
# Make non-critical hooks async: "async": true

# Reduce loaded skills — check if unnecessary skills are auto-activating
/skills
```

### トークン使用量が多い

**症状**: クォータの消費が予想より速い。`/usage` で高い消費量が表示される。

**原因**: 大きな CLAUDE.md ファイル。プリロードされた Skill が多い。完全なコンテキスト継承での Agent 生成。コンパクトしていない。

**解決策**:

1. CLAUDE.md を 200 行以内に収める
2. 特定の Agent のみがプリロードすべき Skill には `user-invocable: false` を使用する
3. フル機能が不要な Agent には `model: haiku` を設定する
4. `/compact` で定期的にコンパクトする
5. 定型タスクには `effort: low` または `effort: medium` を使用する

---

## Hook の問題

### Hook が発火しない

**症状**: 設定で Hook を構成したが実行されない。

**原因**: 設定ファイルが間違っている。マッチャーが見つからないか不一致。Hook が無効化されている。設定の優先順位によるオーバーライド。

**解決策**:

```bash
# 1. Check which hooks are active and their sources
/hooks

# 2. Verify the hook is in the correct settings file
# Team hooks: .claude/settings.json
# Personal hooks: .claude/settings.local.json
# Global hooks: ~/.claude/settings.json

# 3. Check if hooks are globally disabled
# Look for "disableAllHooks": true in any settings file

# 4. Verify the matcher matches the tool name exactly
# "Bash" not "bash", "Write" not "write"

# 5. Check settings hierarchy — a higher-priority file may override
```

### Hook が応答をブロックしている

**症状**: ツール呼び出し後に長い停止。Claude が数秒間フリーズしたように見える。

**原因**: 同期 Hook が遅いコマンドを実行している。非同期でない Hook 内でのネットワーク呼び出し。

**解決策**:

```json
// Add "async": true to non-blocking hooks
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "*",
        "type": "command",
        "command": "python3 log.py",
        "async": true
      }
    ]
  }
}
```

### Hook のタイムアウト

**症状**: Hook は開始されるが結果が使用されない。ログに Hook タイムアウトの警告がある。

**原因**: Hook コマンドに時間がかかりすぎる。外部サービスが利用できない。スクリプトがハングしている。

**解決策**: Hook コマンドを手動でテストする。スクリプト自体にタイムアウトを追加する。外部サービスに到達可能であることを確認する。結果が重要でない場合は Hook を非同期にすることを検討する。

---

## MCP の問題

### MCP サーバーに接続できない

**症状**: MCP ツールが利用できない。`/mcp` でサーバーが切断と表示される。

**原因**: サーバーバイナリが見つからない。設定が間違っている。ポートの競合。環境変数が不足している。

**解決策**:

```bash
# 1. Check MCP status
/mcp

# 2. Verify server configuration in .mcp.json or settings.json
# Ensure the command path is correct and executable

# 3. Check if required environment variables are set
# MCP servers often need API keys in .env

# 4. Try restarting the MCP connection
# Exit and restart Claude Code

# 5. Test the MCP server manually
npx @modelcontextprotocol/server-name
```

### MCP の権限拒否

**症状**: MCP ツールは表示されるが、呼び出し時に権限エラーで失敗する。

**原因**: ツールが許可リストにない。権限モードが制限的すぎる。

**解決策**: MCP ツールを設定の `allowedTools` リストに追加するか、プロンプト時に承認する。Agent の場合は `tools:` フィールドに `mcp__<server>__<tool>` を含める。

---

## Agent の問題

### Agent が自動呼び出しされない

**症状**: `PROACTIVELY` を含む description で Agent を作成したが、Claude が自動的に生成しない。

**原因**: description に呼び出し条件が明確に記述されていない。現在のコンテキストで Agent ツールが利用できない。別のメカニズムがタスクを処理している。

**解決策**:

1. `description` フィールドに `PROACTIVELY` を使用し、トリガー条件を明確に記述する
2. Agent ファイルが `.claude/agents/` に `.md` 拡張子で配置されていることを確認する
3. Claude が `Agent` ツールにアクセスできることを確認する（親 Agent の `tools:` フィールドで制限されていないこと）

### Agent が Agent ツールではなく Bash を生成する

**症状**: Claude が `Agent(...)` ツールを使用する代わりに `claude --agent my-agent` を Bash で実行する。

**原因**: Agent 定義に「起動」や「実行」のような曖昧な言葉が使われており、Claude がシェルコマンドとして解釈する。

**解決策**: CLAUDE.md または Agent の指示に「サブエージェントの生成には bash コマンドではなく Agent ツールを使用すること」と明記する。Agent の description で「launch」や「execute」といった用語を避ける。

### Agent が maxTurns を超過する

**症状**: Agent がタスク途中で不完全な結果で停止する。

**原因**: タスクの複雑さに対して `maxTurns` が低すぎる。Agent がループまたはリトライしている。

**解決策**: Agent の frontmatter で `maxTurns` を増やす。不要なループがないか Agent の動作を確認する。複雑なタスクをより小さなサブタスクに分割する。

---

## Skill の問題

### Skill が /menu に表示されない

**症状**: Skill を作成したが `/` 入力時に表示されない。

**原因**: `user-invocable: false` が設定されている。Skill ファイルが正しい場所にない。YAML frontmatter の構文エラー。

**解決策**:

```bash
# 1. Check skill listing
/skills

# 2. Verify file location: .claude/skills/<name>/SKILL.md

# 3. Check frontmatter — remove user-invocable: false
# or set user-invocable: true

# 4. Validate YAML — common errors:
# - Missing --- delimiters
# - Incorrect indentation
# - Unquoted special characters
```

### Skill が自動検出されない

**症状**: Skill が存在し description もあるが、Claude が自動的に呼び出さない。

**原因**: `disable-model-invocation: true` が設定されている。description が Claude の遭遇するコンテキストにマッチしない。同じトリガーを競合する Skill が多すぎる。

**解決策**:

1. 自動検出が望ましい場合は `disable-model-invocation: true` を削除する
2. Claude が自然に遭遇するキーワードを使った明確な `description` を記述する
3. Skill の数を減らす — 自動検出可能な Skill が多すぎると曖昧さが生じる

---

## メモリの問題

### CLAUDE.md が読み込まれない

**症状**: CLAUDE.md の指示が無視される。Claude がプロジェクトの規約に従わない。

**原因**: ファイルが子孫ディレクトリにある（遅延ロード、即時ロードではない）。ファイル名にタイプミスがある。managed ポリシーによる設定のオーバーライド。

**解決策**:

```bash
# 1. Check loaded memory files
/context

# 2. Verify file name is exactly "CLAUDE.md" (case-sensitive)

# 3. Understand loading rules:
# Ancestors (upward from cwd): loaded eagerly at startup
# Descendants (downward from cwd): loaded lazily when files in that dir are accessed
# Siblings: never loaded

# 4. If you need it always loaded, move it to an ancestor directory
# or reference it from root CLAUDE.md
```

### CLAUDE.local.md がコミットされている

**症状**: 個人的な設定がチームメイトに表示される。`.claude/settings.local.json` が git 履歴にある。

**原因**: ローカルファイルが `.gitignore` に含まれていない。

**解決策**:

```bash
# Add to .gitignore
echo "CLAUDE.local.md" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore

# Remove from tracking if already committed
git rm --cached CLAUDE.local.md
git rm --cached .claude/settings.local.json
```

---

## Git の問題

### Worktree のクリーンアップ

**症状**: Agent 実行後に古い git Worktree が残っている。ディスク容量が増加する。

**原因**: `isolation: worktree` を持つ Agent がクリーンアップ前にクラッシュまたは中断された。

**解決策**:

```bash
# List worktrees
git worktree list

# Remove stale worktrees
git worktree prune

# Force remove a specific worktree
git worktree remove /path/to/worktree --force
```

### コミットの Author

**症状**: Claude Code がコミットを作成する際に、間違った author が設定される。

**原因**: Claude Code セッション用に git の author が設定されていない。

**解決策**:

```bash
# Set git author for the project
git config user.name "Your Name"
git config user.email "your@email.com"

# Or use --author flag in hooks/scripts
git commit --author="Your Name <your@email.com>" -m "message"
```

---

## ソース

- [Claude Code Troubleshooting — Docs](https://code.claude.com/docs/en/troubleshooting)
- [Claude Code Settings — Docs](https://code.claude.com/docs/en/settings)
- [Claude Code Hooks — Docs](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP — Docs](https://code.claude.com/docs/en/mcp)
- [Claude Code Memory — Docs](https://code.claude.com/docs/en/memory)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
