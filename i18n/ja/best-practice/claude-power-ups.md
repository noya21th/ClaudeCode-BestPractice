🌍 [English](../../../best-practice/claude-power-ups.md) | [中文](../../zh/best-practice/claude-power-ups.md) | **日本語** | [Français](../../fr/best-practice/claude-power-ups.md) | [Русский](../../ru/best-practice/claude-power-ups.md) | [العربية](../../ar/best-practice/claude-power-ups.md)

# Power-ups ベストプラクティス

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2002%2C%202026-white?style=flat&labelColor=555)

Claude Code の機能をアニメーションデモで学べるインタラクティブレッスン。各 Power-up は、ほとんどの人が見落としている Claude Code の機能を一つ教えてくれます。v2.1.90 で導入。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 使い方

```bash
claude
/powerup
```

---

## Power-ups (10)

<p align="center">
  <img src="../../../best-practice/assets/claude-power-ups/powerup-menu.png" alt="10のレッスンを表示する Power-ups メニュー" width="700">
</p>

| # | Power-up | トピック |
|---|----------|---------|
| 1 | コードベースと対話する | `@` ファイル、行参照 |
| 2 | モードで操作する | `shift+tab`、plan、auto |
| 3 | 何でも元に戻す | `/rewind`、`Esc-Esc` |
| 4 | バックグラウンドで実行 | tasks、`/tasks` |
| 5 | Claude にルールを教える | `CLAUDE.md`、`/memory` |
| 6 | ツールで拡張する | MCP、`/mcp` |
| 7 | ワークフローを自動化する | Skills、Hooks |
| 8 | 自分を増やす | Subagents、`/agents` |
| 9 | どこからでもコーディング | `/remote-control`、`/teleport` |
| 10 | モデルを調整する | `/model`、`/effort` |

---

## 例: モデルを調整する

最後の Power-up はモデル切り替えと努力レベル制御をアニメーションデモで教えます。

<p align="center">
  <img src="../../../best-practice/assets/claude-power-ups/dial-the-model-1.png" alt="モデルを調整する — 深い思考のデモ" width="700">
</p>

<p align="center">
  <img src="../../../best-practice/assets/claude-power-ups/dial-the-model-2.png" alt="モデルを調整する — 仮説表示のデモ" width="700">
</p>

<p align="center">
  <img src="../../../best-practice/assets/claude-power-ups/dial-the-model-3.png" alt="モデルを調整する — 努力レベルを high に設定するデモ" width="700">
</p>

---

## ソース

- [Changelog — v2.1.90](https://code.claude.com/docs/en/changelog)
