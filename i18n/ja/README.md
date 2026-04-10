🌍 [English](../../README.md) | [中文](../zh/README.md) | **日本語** | [Français](../fr/README.md) | [Русский](../ru/README.md) | [العربية](../ar/README.md)

# claude-code-best-practice
vibe coding からアジェンティック・エンジニアリングへ — 実践が Claude を完璧にする

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-v2.1.96%20(Apr%2008%2C%202026%209%3A38%20PM%20PKT)-white?style=flat&labelColor=555)<br>

[![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/) [![Implemented](../../!/tags/implemented.svg)](../../implementation/) [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) [![Claude](../../!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](../../!/tags/boris-cherny.svg)](#-ヒントとコツ) [![Community](../../!/tags/community.svg)](#-購読) ![Click on these badges below to see the actual sources](../../!/tags/click-badges.svg)<br>
<img src="../../!/tags/a.svg" height="14"> = Agents · <img src="../../!/tags/c.svg" height="14"> = Commands · <img src="../../!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="Claude Code mascot jumping" width="120" height="100"><br>

</p>


## 🧠 コンセプト

| 機能 | 場所 | 説明 |
|------|------|------|
| <img src="../../!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-subagents.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-subagents-implementation.md) 独立したコンテキストで動作する自律アクター — カスタムツール、権限、モデル、メモリ、永続的なアイデンティティ |
| <img src="../../!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/slash-commands) | `.claude/commands/<name>.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-commands.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-commands-implementation.md) 既存のコンテキストに注入されるナレッジ — ワークフローオーケストレーション用のシンプルなプロンプトテンプレート |
| <img src="../../!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-skills.md) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-skills-implementation.md) 既存のコンテキストに注入されるナレッジ — 設定可能、プリロード可能、自動検出可能で、コンテキストフォークとプログレッシブ・ディスクロージャーに対応 · [公式 Skills](https://github.com/anthropics/skills/tree/main/skills) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](../../.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](../../!/tags/orchestration-workflow.svg)](../../orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) エージェントループの外部で実行されるユーザー定義ハンドラ（スクリプト、HTTP、プロンプト、Agent） · [ガイド](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-mcp.md) [![Implemented](../../!/tags/implemented.svg)](../../.mcp.json) 外部ツール、データベース、API への Model Context Protocol 接続 |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | 配布可能パッケージ | Skills、Subagents、Hooks、MCP Servers、LSP Servers のバンドル · [マーケットプレイス](https://code.claude.com/docs/en/discover-plugins) · [マーケットプレイス作成](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-settings.md) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) 階層型設定システム · [権限](https://code.claude.com/docs/en/permissions) · [モデル設定](https://code.claude.com/docs/en/model-config) · [出力スタイル](https://code.claude.com/docs/en/output-styles) · [サンドボックス](https://code.claude.com/docs/en/sandboxing) · [キーバインド](https://code.claude.com/docs/en/keybindings) · [高速モード](https://code.claude.com/docs/en/fast-mode) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](../../!/tags/implemented.svg)](../../.claude/settings.json) コンテキスト使用量、モデル、コスト、セッション情報を表示するカスタマイズ可能なステータスバー |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-memory.md) [![Implemented](../../!/tags/implemented.svg)](../../CLAUDE.md) CLAUDE.md ファイルと `@path` インポートによる永続コンテキスト · [自動メモリ](https://code.claude.com/docs/en/memory) · [ルール](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | 自動（git ベース） | ファイル編集の自動追跡と巻き戻し（`Esc Esc` または `/rewind`）および選択的な要約 |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-cli-startup-flags.md) Claude Code 起動時のコマンドラインフラグ、サブコマンド、環境変数 · [インタラクティブモード](https://code.claude.com/docs/en/interactive-mode) |
| **AI Terms** | | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) Agentic Engineering · Context Engineering · Vibe Coding |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | 公式ベストプラクティス · [プロンプトエンジニアリング](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [Claude Code の拡張](https://code.claude.com/docs/en/features-overview) |

### 🔥 注目

| 機能 | 場所 | 説明 |
|------|------|------|
| [**Power-ups**](../../best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](../../!/tags/best-practice.svg)](../../best-practice/claude-power-ups.md) Claude Code の機能をアニメーションデモで学べるインタラクティブレッスン（v2.1.90） |
| [**Ultraplan**](https://code.claude.com/docs/en/ultraplan) ![beta](../../!/tags/beta.svg) | `/ultraplan` | クラウドでプランを作成し、ブラウザベースのレビュー、インラインコメント、柔軟な実行が可能 — リモートまたはターミナルへのテレポート |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](../../!/tags/beta.svg) | `claude.ai/code` | クラウドインフラでタスクを実行 — 長時間タスク、PR 自動修正、ローカルセットアップ不要の並列セッション · [スケジュールタスク](https://code.claude.com/docs/en/web-scheduled-tasks) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](../../!/tags/beta.svg) | `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) マウスサポート、安定したメモリ、アプリ内スクロールを備えたフリッカーフリーの代替画面レンダリング — オプトインのリサーチプレビュー |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](../../!/tags/beta.svg) | `computer-use` MCP server | Claude が画面を操作 — macOS でアプリを開く、クリック、入力、スクリーンショットを撮影 · [デスクトップ](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) ![beta](../../!/tags/beta.svg) | `claude --enable-auto-mode` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) バックグラウンドの安全分類器が手動の権限プロンプトを置き換え — Claude が安全性を判断しつつ、プロンプトインジェクションやリスクの高いエスカレーションをブロック · `claude --enable-auto-mode`（または `--permission-mode auto`）で起動、セッション中は `Shift+Tab` で切り替え · [ブログ](https://claude.com/blog/auto-mode) |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](../../!/tags/beta.svg) | `--channels`, plugin ベース | Telegram、Discord、Webhook からのイベントを実行中のセッションにプッシュ — 不在中でも Claude が対応 · [リファレンス](https://code.claude.com/docs/en/channels-reference) |
| [**Slack**](https://code.claude.com/docs/en/slack) | Slack で `@Claude` | チームチャットで @Claude にコーディングタスクをメンション — バグ修正、コードレビュー、並列タスク実行のために Claude Code Web セッションへルーティング |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](../../!/tags/beta.svg) | GitHub App（マネージド） | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) バグ、セキュリティ脆弱性、リグレッションを検出するマルチエージェント PR 分析 · [ブログ](https://claude.com/blog/code-review) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | CI/CD パイプラインで PR レビュー、Issue トリアージ、コード生成を自動化 · [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) ![beta](../../!/tags/beta.svg) | `--chrome`, extension | [![Best Practice](../../!/tags/best-practice.svg)](../../reports/claude-in-chrome-v-chrome-devtools-mcp.md) Claude in Chrome によるブラウザ自動化 — Web アプリのテスト、コンソールデバッグ、フォーム自動化、ページからのデータ抽出 |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, cron tools | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-scheduled-tasks-implementation.md) `/loop` はローカルで定期的にプロンプトを実行（最大3日） · [`/schedule`](https://code.claude.com/docs/en/web-scheduled-tasks) はクラウドで Anthropic インフラ上でプロンプトを実行 — マシンがオフでも動作 · [アナウンス](https://x.com/noahzweben/status/2036129220959805859) |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](../../!/tags/beta.svg) | `/voice` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) 20言語対応のプッシュトゥトーク音声入力で、起動キーの再バインドが可能 |
| [**Simplify & Batch**](https://code.claude.com/docs/en/skills#bundled-skills) | `/simplify`, `/batch` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) コード品質と一括操作のための組み込み Skill — simplify は再利用と効率のためにリファクタリング、batch はファイル横断でコマンドを実行 |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](../../!/tags/beta.svg) | 組み込み（環境変数） | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](../../!/tags/implemented.svg)](../../implementation/claude-agent-teams-implementation.md) 共有タスク協調で同じコードベース上の複数エージェントが並列動作 |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) 任意のデバイスからローカルセッションを継続 — スマホ、タブレット、ブラウザ · [ヘッドレスモード](https://code.claude.com/docs/en/headless) |
| [**Git Worktrees**](https://code.claude.com/docs/en/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) | 組み込み | [![Best Practice](../../!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) 並列開発のための分離された git ブランチ — 各エージェントが独自の作業コピーを持つ |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](../../!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26) 長時間タスクのための自律開発ループ — 完了まで反復 |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="../../orchestration-workflow/orchestration-workflow.md"><img src="../../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

<img src="../../!/tags/c.svg" height="14"> **Command** → <img src="../../!/tags/a.svg" height="14"> **Agent** → <img src="../../!/tags/s.svg" height="14"> **Skill** パターンの実装詳細は [orchestration-workflow](../../orchestration-workflow/orchestration-workflow.md) を参照。


<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

<p align="center">
  <img src="../../orchestration-workflow/orchestration-workflow.gif" alt="Orchestration Workflow Demo" width="600">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ⚙️ 開発ワークフロー

すべての主要ワークフローは同じアーキテクチャパターンに収束する: **調査 → 計画 → 実行 → レビュー → リリース**

| 名前 | ★ | 独自性 | 計画 | <img src="../../!/tags/a.svg" height="14"> | <img src="../../!/tags/c.svg" height="14"> | <img src="../../!/tags/s.svg" height="14"> |
|------|---|--------|------|---|---|---|
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 146k | ![instinct scoring](https://img.shields.io/badge/instinct_scoring-ddf4ff) ![AgentShield](https://img.shields.io/badge/AgentShield-ddf4ff) ![multi-lang rules](https://img.shields.io/badge/multi--lang_rules-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [planner](https://github.com/affaan-m/everything-claude-code/blob/main/agents/planner.md) | 47 | 82 | 182 |
| [Superpowers](https://github.com/obra/superpowers) | 141k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![whole-plan review](https://img.shields.io/badge/whole--plan_review-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 3 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 86k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![22+ tools](https://img.shields.io/badge/22%2B_tools-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [speckit.plan](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md) | 0 | 9+ | 0 |
| [gstack](https://github.com/garrytan/gstack) | 67k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 0 | 37 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 49k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![XML plans](https://img.shields.io/badge/XML_plans-ddf4ff) | <img src="../../!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 24 | 68 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 44k | ![full SDLC](https://img.shields.io/badge/full_SDLC-ddf4ff) ![agent personas](https://img.shields.io/badge/agent_personas-ddf4ff) ![22+ platforms](https://img.shields.io/badge/22%2B_platforms-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [bmad-create-prd](https://github.com/bmad-code-org/BMAD-METHOD/tree/main/src/bmm-skills/2-plan-workflows/bmad-create-prd) | 0 | 0 | 39 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 38k | ![delta specs](https://img.shields.io/badge/delta_specs-ddf4ff) ![brownfield](https://img.shields.io/badge/brownfield-ddf4ff) ![artifact DAG](https://img.shields.io/badge/artifact_DAG-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [opsx:propose](https://github.com/Fission-AI/OpenSpec/blob/main/src/commands/workflow/new-change.ts) | 0 | 11 | 0 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 26k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![skill auto-inject](https://img.shields.io/badge/skill_auto--inject-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-claudecode/tree/main/skills/ralplan) | 19 | 0 | 37 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 14k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![Plugin Marketplace](https://img.shields.io/badge/Plugin_Marketplace-ddf4ff) | <img src="../../!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 51 | 4 | 44 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 10k | ![RPI](https://img.shields.io/badge/RPI-ddf4ff) ![context engineering](https://img.shields.io/badge/context_engineering-ddf4ff) ![300k+ LOC](https://img.shields.io/badge/300k%2B_LOC-ddf4ff) | <img src="../../!/tags/c.svg" height="14"> [create_plan](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md) | 6 | 27 | 0 |

### その他
- [Cross-Model (Claude Code + Codex) Workflow](../../development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/cross-model-workflow/cross-model-workflow.md)
- [RPI](../../development-workflows/rpi/rpi-workflow.md) [![Implemented](../../!/tags/implemented.svg)](../../development-workflows/rpi/rpi-workflow.md)
- [Ralph Wiggum Loop](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](../../!/tags/implemented.svg)](https://github.com/shanraisshan/novel-llm-26)
- [Andrej Karpathy（OpenAI 創設メンバー）のワークフロー](https://x.com/karpathy/status/2015883857489522876)
- [Peter Steinberger（OpenClaw 作者）のワークフロー](https://youtu.be/8lF7HmQ_RgY?t=2582)
- Boris Cherny（Claude Code 作者）のワークフロー — [13のヒント](../../tips/claude-boris-13-tips-03-jan-26.md) · [10のヒント](../../tips/claude-boris-10-tips-01-feb-26.md) · [12のヒント](../../tips/claude-boris-12-tips-12-feb-26.md) · [2つのヒント](../../tips/claude-boris-2-tips-25-mar-26.md) · [15のヒント](../../tips/claude-boris-15-tips-30-mar-26.md) [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny)

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 💡 ヒントとコツ (69)

🚫👶 = 監視不要

[プロンプティング](#tips-prompting) · [計画](#tips-planning) · [CLAUDE.md](#tips-claudemd) · [Agents](#tips-agents) · [Commands](#tips-commands) · [Skills](#tips-skills) · [Hooks](#tips-hooks) · [ワークフロー](#tips-workflows) · [上級](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [デバッグ](#tips-debugging) · [ユーティリティ](#tips-utilities) · [日常](#tips-daily)

![Community](../../!/tags/community.svg)

<a id="tips-prompting"></a>■ **プロンプティング (3)**

| ヒント | ソース |
|--------|--------|
| Claude に挑戦する — 「この変更について厳しくチェックして、テストに合格するまで PR を作らないで」や「これが動くことを証明して」と言い、main とブランチの差分を確認させる 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| いまいちな修正の後 — 「今わかっていることすべてを踏まえて、これを捨ててエレガントな解決策を実装して」 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude はほとんどのバグを自分で修正する — バグを貼り付けて「修正して」と言うだけ、方法は指示しない 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **計画/仕様 (6)**

| ヒント | ソース |
|--------|--------|
| 常に [plan mode](https://code.claude.com/docs/en/common-workflows) から始める | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| 最小限の仕様やプロンプトから始めて、Claude に [AskUserQuestion](https://code.claude.com/docs/en/cli-reference) ツールを使ってインタビューさせ、新しいセッションで仕様を実行する | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| 常にフェーズごとのゲート付き計画を作成し、各フェーズに複数のテスト（ユニット、自動化、統合）を含める | |
| 2つ目の Claude を起動してスタッフエンジニアとして計画をレビューさせるか、[cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) をレビューに使う | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| 作業を渡す前に詳細な仕様を書き、あいまいさを減らす — 具体的であるほど出力品質が向上する | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| プロトタイプ > PRD — 仕様を書くより20-30バージョンを作る、構築コストが低いので多くの試行を | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-claudemd"></a>■ **CLAUDE.md (7)**

| ヒント | ソース |
|--------|--------|
| [CLAUDE.md](https://code.claude.com/docs/en/memory) はファイルあたり [200行以下](https://code.claude.com/docs/en/memory#write-effective-instructions) を目標に。[humanlayer では60行](https://www.humanlayer.dev/blog/writing-a-good-claude-md)（[それでも100%保証ではない](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)） | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](../../!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| ドメイン固有の CLAUDE.md ルールを [\<important if="..."\> タグ](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md)で囲み、ファイルが長くなっても Claude が無視しないようにする | [![Dex](../../!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| モノレポには[複数の CLAUDE.md](../../best-practice/claude-memory.md) を使う — 祖先 + 子孫ローディング | |
| [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) で大きな指示を分割する | |
| [memory.md](https://code.claude.com/docs/en/memory)、constitution.md は何も保証しない | |
| どの開発者でも Claude を起動して「テストを実行して」と言えば初回で動くべき — 動かないなら、CLAUDE.md に必須のセットアップ/ビルド/テストコマンドが不足している | [![Dex](../../!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| コードベースをクリーンに保ちマイグレーションを完了させる — 部分的にマイグレーションされたフレームワークはモデルを混乱させ、間違ったパターンを選ぶ可能性がある | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| ハーネスが強制する動作（アトリビューション、権限、モデル）には [settings.json](../../best-practice/claude-settings.md) を使う — `attribution.commit: ""` が決定的なのに CLAUDE.md に「絶対に Co-Authored-By を追加しない」と書かない | [![davila7](../../!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="../../!/tags/a.svg" height="14"> **Agents (4)**

| ヒント | ソース |
|--------|--------|
| 汎用的な qa や backend engineer ではなく、[Skills](https://code.claude.com/docs/en/skills)（プログレッシブ・ディスクロージャー）を持つ機能固有の [Sub-agents](https://code.claude.com/docs/en/sub-agents)（追加コンテキスト）を使う | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| 「subagents を使って」と言えば問題により多くの計算リソースを投入できる — メインコンテキストをクリーンに保つためにタスクをオフロード 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| 並列開発には [tmux による agent teams](https://code.claude.com/docs/en/agent-teams) と [git worktrees](https://x.com/bcherny/status/2025007393290272904) を使う | |
| [テスト時の計算量](https://code.claude.com/docs/en/sub-agents)を活用 — 別々のコンテキストウィンドウが結果を改善する。一つのエージェントがバグを起こしても、別のエージェント（同じモデル）がそれを見つけられる | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="../../!/tags/c.svg" height="14"> **Commands (3)**

| ヒント | ソース |
|--------|--------|
| ワークフローには [Sub-agents](https://code.claude.com/docs/en/sub-agents) ではなく [Commands](https://code.claude.com/docs/en/slash-commands) を使う | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 1日に何度も行う「インナーループ」ワークフローには [Slash Commands](https://code.claude.com/docs/en/slash-commands) を使う — 繰り返しのプロンプティングを省き、コマンドは `.claude/commands/` に置いて git にコミット | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 1日に2回以上やることは [Skill](https://code.claude.com/docs/en/skills) か [Command](https://code.claude.com/docs/en/slash-commands) にする — `/techdebt`、コンテキストダンプ、分析コマンドを作る | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="../../!/tags/s.svg" height="14"> **Skills (9)**

| ヒント | ソース |
|--------|--------|
| [context: fork](https://code.claude.com/docs/en/skills) を使って Skill を分離された Subagent で実行 — メインコンテキストには最終結果のみ表示され、中間ツール呼び出しは見えない。agent フィールドで Subagent タイプを設定可能 | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| モノレポには[サブフォルダ内の Skills](../../reports/claude-skills-for-larger-mono-repos.md) を使う | |
| Skills はファイルではなくフォルダ — references/、scripts/、examples/ サブディレクトリで[プログレッシブ・ディスクロージャー](https://code.claude.com/docs/en/skills)を活用 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| すべての Skill に Gotchas セクションを作る — 最も価値の高いコンテンツ、Claude の失敗ポイントを時間とともに追加 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| Skill の description フィールドは要約ではなくトリガー — モデル向けに書く（「いつ発火すべきか？」） | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| Skills に当たり前のことを書かない — Claude のデフォルト動作から外れる部分に焦点を当てる 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| Skills で Claude を制約しすぎない — 詳細なステップバイステップではなく、目標と制約を与える 🚫👶 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| Skills にスクリプトやライブラリを含め、Claude がボイラープレートを再構築するのではなく合成できるようにする | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| SKILL.md に `` !`command` `` を埋め込んで動的なシェル出力をプロンプトに注入 — Claude が呼び出し時に実行し、モデルは結果のみ参照 | [![Lydia](../../!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **Hooks (5)**

| ヒント | ソース |
|--------|--------|
| Skills の[オンデマンド Hooks](https://code.claude.com/docs/en/skills) を使う — /careful で破壊的コマンドをブロック、/freeze でディレクトリ外の編集をブロック | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| PreToolUse Hook で [Skill の使用状況を測定](https://code.claude.com/docs/en/skills)し、人気の Skill や発火不足の Skill を発見 | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| [PostToolUse Hook](https://code.claude.com/docs/en/hooks) でコードを自動整形 — Claude は十分整形されたコードを生成し、Hook が残り10%を処理して CI 失敗を回避 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| [権限リクエスト](https://code.claude.com/docs/en/hooks)を Hook 経由で Opus にルーティング — 攻撃をスキャンし安全なものを自動承認させる 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| [Stop Hook](https://code.claude.com/docs/en/hooks) を使って、ターンの終わりに Claude に作業の続行や検証を促す | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **ワークフロー (7)**

| ヒント | ソース |
|--------|--------|
| エージェントのダムゾーンを避け、最大50%で手動 [/compact](https://code.claude.com/docs/en/interactive-mode) を実行。タスク切り替え時は [/clear](https://code.claude.com/docs/en/cli-reference) でコンテキストをリセット | |
| 小さなタスクならどんなワークフローよりもバニラの Claude Code が優れている | |
| [/model](https://code.claude.com/docs/en/model-config) でモデルと推論を選択、[/context](https://code.claude.com/docs/en/interactive-mode) でコンテキスト使用量を確認、[/usage](https://code.claude.com/docs/en/costs) でプラン制限を確認、[/extra-usage](https://code.claude.com/docs/en/interactive-mode) でオーバーフロー課金を設定、[/config](https://code.claude.com/docs/en/settings) で設定変更 — plan mode には Opus、コードには Sonnet を使って両方の長所を活用 | [![Cat](../../!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| 常に[シンキングモード](https://code.claude.com/docs/en/model-config)を true（推論を見るため）、[出力スタイル](https://code.claude.com/docs/en/output-styles)を Explanatory（★ Insight ボックス付きの詳細出力）に設定し、[/config](https://code.claude.com/docs/en/settings) で Claude の判断をよりよく理解する | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| プロンプトで ultrathink キーワードを使い[高い推論努力](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices)を得る | |
| 重要なセッションには [/rename](https://code.claude.com/docs/en/cli-reference)（例：[TODO - リファクタリング]）し、後で [/resume](https://code.claude.com/docs/en/cli-reference) する — 複数の Claude を同時実行時は各インスタンスにラベルを付ける | [![Cat](../../!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |
| Claude が脱線したら同じコンテキストで修正を試みず [Esc Esc や /rewind](https://code.claude.com/docs/en/checkpointing) で元に戻す | |

<a id="tips-workflows-advanced"></a>■ **ワークフロー上級 (6)**

| ヒント | ソース |
|--------|--------|
| アーキテクチャ理解に ASCII ダイアグラムを多用する | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| ローカルの定期監視には [/loop](https://code.claude.com/docs/en/scheduled-tasks)（最大3日） · マシンオフでも動くクラウドベースの定期タスクには [/schedule](https://code.claude.com/docs/en/web-scheduled-tasks) を使う | |
| 長時間の自律タスクには [Ralph Wiggum plugin](https://github.com/shanraisshan/novel-llm-26) を使う | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| dangerously-skip-permissions の代わりにワイルドカード構文で [/permissions](https://code.claude.com/docs/en/permissions)（Bash(npm run *)、Edit(/docs/**)） | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing) でファイルとネットワークの分離により権限プロンプトを削減 — 内部で84%削減を達成 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](../../!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| [製品検証](https://code.claude.com/docs/en/skills) Skill（signup-flow-driver、checkout-verifier）に投資する — 完璧にするのに1週間かける価値がある | [![Thariq](../../!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**

| ヒント | ソース |
|--------|--------|
| PR は小さく集中させる — [p50は118行](../../tips/claude-boris-2-tips-25-mar-26.md)（1日で141 PR、45K行変更）、1 PR につき1機能、レビューとリバートが容易 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| 常に PR を [squash merge](../../tips/claude-boris-2-tips-25-mar-26.md) する — クリーンなリニア履歴、機能ごとに1コミット、git revert と git bisect が容易 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| 頻繁にコミットする — 少なくとも1時間に1回、タスク完了後すぐにコミット | ![Shayan](../../!/tags/community-shayan.svg) |
| 同僚の PR に [@claude](https://github.com/apps/claude) をタグして、繰り返しのレビューフィードバックの lint ルールを自動生成 — コードレビューを自動化する 🚫👶 | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| [/code-review](https://code.claude.com/docs/en/code-review) でマルチエージェント PR 分析 — マージ前にバグ、セキュリティ脆弱性、リグレッションを検出 | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **デバッグ (7)**

| ヒント | ソース |
|--------|--------|
| 問題で行き詰まったらスクリーンショットを撮って Claude に共有する習慣をつける | ![Shayan](../../!/tags/community-shayan.svg) |
| MCP（[Claude in Chrome](https://code.claude.com/docs/en/chrome)、[Playwright](https://github.com/microsoft/playwright-mcp)、[Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp)）を使って Claude に Chrome のコンソールログを自分で見させる | |
| デバッグを改善するため、ログを見たいターミナルは常にバックグラウンドタスクとして実行するよう Claude に指示 | |
| インストール、認証、設定の問題を診断するには [/doctor](https://code.claude.com/docs/en/cli-reference) を使う | |
| コンパクション中のエラーは [/model](https://code.claude.com/docs/en/model-config) で 1M トークンモデルを選択し、[/compact](https://code.claude.com/docs/en/interactive-mode) を実行すれば解決できる | |
| QA には [cross-model](../../development-workflows/cross-model-workflow/cross-model-workflow.md) を使う — 例：計画と実装レビューに [Codex](https://github.com/shanraisshan/codex-cli-best-practice) を使用 | |
| エージェント検索（glob + grep）は RAG に勝る — Claude Code はベクターデータベースを試したが、コードの同期ずれと権限の複雑さのため廃止した | [![Boris](../../!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](../../!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **ユーティリティ (5)**

| ヒント | ソース |
|--------|--------|
| IDE（[VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)）ではなく [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) ターミナルを使う | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| 音声プロンプティングには [/voice](https://code.claude.com/docs/en/voice-dictation) か [Wispr Flow](https://wisprflow.ai)（生産性10倍） | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| Claude のフィードバックには [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) | ![Shayan](../../!/tags/community-shayan.svg) |
| コンテキスト把握と素早いコンパクションには [status line](https://github.com/shanraisshan/claude-code-status-line) | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](../../!/tags/community-shayan.svg) |
| [settings.json](../../best-practice/claude-settings.md) の機能を探索 — [Plans Directory](../../best-practice/claude-settings.md#plans-directory)、[Spinner Verbs](../../best-practice/claude-settings.md#display--ux) でパーソナライズ | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **日常 (2)**

| ヒント | ソース |
|--------|--------|
| Claude Code を毎日[アップデート](https://code.claude.com/docs/en/setup)する | ![Shayan](../../!/tags/community-shayan.svg) |
| 1日の始まりに[変更履歴](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)を読む | ![Shayan](../../!/tags/community-shayan.svg) |

![Boris Cherny + Team](../../!/tags/claude.svg)

| 記事 / ツイート | ソース |
|-----------------|--------|
| [Claude Code の15の隠れた活用不足機能 (Boris) \| 2026/3/30](../../tips/claude-boris-15-tips-30-mar-26.md) | [ツイート](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merge と PR サイズ分布 (Boris) \| 2026/3/25](../../tips/claude-boris-2-tips-25-mar-26.md) | [ツイート](https://x.com/bcherny/status/2038552880018538749) |
| [Claude Code 構築からの教訓: Skills の使い方 (Thariq) \| 2026/3/17](../../tips/claude-thariq-tips-17-mar-26.md) | [記事](https://x.com/trq212/status/2033949937936085378) |
| [Code Review とテスト時計算量 (Boris) \| 2026/3/10](../../tips/claude-boris-2-tips-10-mar-26.md) | [ツイート](https://x.com/bcherny/status/2031089411820228645) |
| /loop — 最大3日間の定期タスクスケジュール (Boris) \| 2026年3月7日 | [ツイート](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Thariq) \| 2026年2月28日 | [ツイート](https://x.com/trq212/status/2027543858289250472) |
| エージェントの目で見る - Claude Code 構築からの教訓 (Thariq) \| 2026年2月28日 | [記事](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees - Boris の5つの活用法 \| 2026年2月21日 | [ツイート](https://x.com/bcherny/status/2025007393290272904) |
| Claude Code 構築からの教訓: プロンプトキャッシングがすべて (Thariq) \| 2026年2月20日 | [記事](https://x.com/trq212/status/2024574133011673516) |
| [12通りの Claude カスタマイズ方法 (Boris) \| 2026/2/12](../../tips/claude-boris-12-tips-12-feb-26.md) | [ツイート](https://x.com/bcherny/status/2021699851499798911) |
| [チームからの Claude Code 使用10のヒント (Boris) \| 2026/2/1](../../tips/claude-boris-10-tips-01-feb-26.md) | [ツイート](https://x.com/bcherny/status/2017742741636321619) |
| [Claude Code の使い方 — 驚くほどシンプルなセットアップから13のヒント (Boris) \| 2026/1/3](../../tips/claude-boris-13-tips-03-jan-26.md) | [ツイート](https://x.com/bcherny/status/2007179832300581177) |
| AskUserQuestion ツールで Claude にインタビューさせる (Thariq) \| 2025/12/28 | [ツイート](https://x.com/trq212/status/2005315275026260309) |
| 常に plan mode を使い、検証手段を与え、/code-review を使う (Boris) \| 2025/12/27 | [ツイート](https://x.com/bcherny/status/2004711722926616680) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🎬 動画 / ポッドキャスト

| 動画 / ポッドキャスト | ソース | YouTube |
|-----------------------|--------|---------|
| Research-Plan-Implement で間違えたこと全部 (Dex) \| 2026年3月24日 \| MLOps Community | [![Dex](../../!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| Boris Cherny と Claude Code を作る (Boris) \| 2026年3月4日 \| The Pragmatic Engineer | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| Claude Code 責任者: コーディングが解決された後に何が起こるか (Boris) \| 2026年2月19日 \| Lenny's Podcast | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| Claude Code の内側をクリエイター Boris Cherny と共に (Boris) \| 2026年2月17日 \| Y Combinator | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Claude Code クリエイター Boris Cherny のキャリア成長論 (Boris) \| 2025年12月15日 \| Ryan Peterman | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| Claude Code を作ったエンジニアたちの秘密 (Cat) \| 2025年10月29日 \| Every | [![Boris](../../!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🔔 購読

| ソース | 名前 | バッジ |
|--------|------|--------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](../../!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Jesse Kriss](https://x.com/obra) ([Superpowers](https://github.com/obra/superpowers)), [Affaan Mustafa](https://x.com/affaanmustafa) ([ECC](https://github.com/affaan-m/everything-claude-code)), [Garry Tan](https://x.com/garrytan) ([gstack](https://github.com/garrytan/gstack)), [Dex Horthy](https://x.com/dexhorthy) ([HumanLayer](https://github.com/humanlayer/humanlayer)), [Kieran Klaassen](https://x.com/kieranklaassen) ([Compound Eng](https://github.com/EveryInc/compound-engineering-plugin)), [Tabish Gilani](https://x.com/0xTab) ([OpenSpec](https://github.com/Fission-AI/OpenSpec)), [Brian McAdams](https://x.com/BMadCode) ([BMAD](https://github.com/bmad-code-org/BMAD-METHOD)), [Lex Christopherson](https://x.com/official_taches) ([GSD](https://github.com/gsd-build/get-shit-done)), [Dani Avila](https://x.com/dani_avila7) ([CC Templates](https://github.com/davila7/claude-code-templates)), [Dan Shipper](https://x.com/danshipper) ([Every](https://every.to/)), [Andrej Karpathy](https://x.com/karpathy) ([AutoResearch](https://x.com/karpathy/status/2015883857489522876)), [Peter Steinberger](https://x.com/steipete) ([OpenClaw](https://x.com/openclaw)), [Sigrid Jin](https://x.com/realsigridjin) ([claw-code](https://github.com/ultraworkers/claw-code)), [Yeachan Heo](https://x.com/bellman_ych) ([oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)) | ![Community](../../!/tags/community.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](../../!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Lenny's Podcast](https://www.youtube.com/@LennysPodcast), [Y Combinator](https://www.youtube.com/@ycombinator), [The Pragmatic Engineer](https://www.youtube.com/@pragmaticengineer), [Ryan Peterman](https://www.youtube.com/@ryanlpeterman), [Every](https://www.youtube.com/@every_media), [MLOps Community](https://www.youtube.com/@MLOps) | ![Community](../../!/tags/community.svg) |

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ☠️ スタートアップ / ビジネス

| Claude | 代替された製品 |
|-|-|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|YC AI ラッパースタートアップ ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](../../!/tags/billion-dollar-questions.svg)

*これらの質問は [Issues](https://github.com/noya21th/ClaudeCode-BestPractice/issues) で議論しましょう*

**メモリと指示 (4)**

1. CLAUDE.md に何を入れるべきで、何を入れないべきか？
2. 既に CLAUDE.md があるなら、constitution.md や rules.md は本当に必要か？
3. CLAUDE.md をどのくらいの頻度で更新すべきで、古くなったことをどう判断するか？
4. CLAUDE.md の指示に MUST と大文字で書いても、なぜ Claude は無視するのか？（[reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)）

**Agents、Skills、ワークフロー (6)**

1. Command vs Agent vs Skill をいつ使うべきか — そしてバニラの Claude Code の方が良いのはいつか？
2. モデルの改善に伴い、Agents、Commands、ワークフローをどのくらいの頻度で更新すべきか？
3. Subagent に詳細なペルソナを与えると品質が向上するか？調査/QA 用 Subagent の「完璧なペルソナ/プロンプト」はどんなものか？
4. Claude Code の組み込み plan mode に頼るべきか、チームのワークフローを強制する独自の計画 Command/Agent を構築すべきか？
5. 個人用 Skill（例：コーディングスタイル付き /implement）がある場合、コミュニティの Skill（例：/simplify）とどう統合し、意見が食い違った時はどちらが優先か？
6. 既存のコードベースを仕様に変換し、コードを削除して、AI がその仕様だけから全く同じコードを再生成できるか？

**仕様とドキュメント (3)**

1. リポジトリのすべての機能にマークダウンファイルの仕様を持つべきか？
2. 新機能実装時に仕様が陳腐化しないよう、どのくらいの頻度で更新が必要か？
3. 新機能を実装する際、他の機能の仕様への波及効果にどう対処するか？

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## レポート

<p align="center">
  <a href="../../reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="../../reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="../../reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="../../reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="../../reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="../../reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="../../reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="../../reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="../../reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
</p>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

![How to Use](../../!/tags/how-to-use.svg)

```
1. このリポジトリをコースのように読み、Commands、Agents、Skills、Hooks が何かを理解してから使い始める。
2. リポジトリをクローンして例を試す。/weather-orchestrator を試し、Hook のサウンドを聴き、Agent Teams を実行して、実際の動作を確認する。
3. 自分のプロジェクトに移り、このリポジトリからどのベストプラクティスを追加すべきか Claude に提案させる。このリポジトリを参照として渡し、何が可能かを知らせる。
```

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="../../!/thumbnail/video-1.png" alt="YouTube で視聴" width="300"></a>

<p align="center">
  <img src="../../!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 謝辞

[shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) をベースに開発。原作者 Shayan Rais のオープンソース貢献に感謝します。
