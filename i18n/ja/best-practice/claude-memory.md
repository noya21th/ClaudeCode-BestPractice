🌍 [English](../../../best-practice/claude-memory.md) | [中文](../../zh/best-practice/claude-memory.md) | **日本語** | [Français](../../fr/best-practice/claude-memory.md) | [Русский](../../ru/best-practice/claude-memory.md) | [العربية](../../ar/best-practice/claude-memory.md)

# Claude Memory

CLAUDE.md ファイルによる永続コンテキスト — 書き方とモノレポでのロード方法。

<table width="100%">
<tr>
<td><a href="../">← Claude Code ベストプラクティスに戻る</a></td>
<td align="right"><img src="../../../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. 良い CLAUDE.md の書き方

適切に構造化された CLAUDE.md は、プロジェクトにおける Claude Code の出力品質を最も効果的に改善する方法です。Humanlayer が、何を含めるべきか、どう構造化するか、よくある落とし穴について優れたガイドを公開しています。

- [Humanlayer - 良い Claude.md の書き方](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. 大規模モノレポでの CLAUDE.md

モノレポで Claude Code を使用する際、CLAUDE.md ファイルがどのようにコンテキストにロードされるかを理解することが、プロジェクト指示を効果的に整理するために重要です。

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="../../../best-practice/assets/claude-memory/claude-memory-monorepo.jpg" alt="モノレポでの CLAUDE.md ローディング" width="600"></a>
</p>

### 2つのローディングメカニズム

Claude Code は CLAUDE.md ファイルのロードに2つの異なるメカニズムを使用します:

#### 祖先ローディング（ディレクトリツリーを上方向に）

Claude Code を起動すると、現在の作業ディレクトリからファイルシステムのルートに向かって**上方向**に走査し、途中で見つかるすべての CLAUDE.md をロードします。これらのファイルは**起動時に即座にロード**されます。

#### 子孫ローディング（ディレクトリツリーを下方向に）

現在の作業ディレクトリより下のサブディレクトリにある CLAUDE.md ファイルは**起動時にはロードされません**。セッション中に Claude がそれらのサブディレクトリ内のファイルを読み込んだ時にのみ含まれます。これは**遅延ローディング**として知られています。

### モノレポ構造の例

異なるコンポーネント用に別々のディレクトリを持つ典型的なモノレポを考えてみましょう:

```
/mymonorepo/
├── CLAUDE.md          # ルートレベルの指示（全コンポーネント共通）
├── frontend/
│   └── CLAUDE.md      # フロントエンド固有の指示
├── backend/
│   └── CLAUDE.md      # バックエンド固有の指示
└── api/
    └── CLAUDE.md      # API 固有の指示
```

### シナリオ 1: ルートディレクトリから Claude Code を実行

`/mymonorepo/` から Claude Code を実行する場合:

```bash
cd /mymonorepo
claude
```

| ファイル | 起動時にロード？ | 理由 |
|---------|----------------|------|
| `/mymonorepo/CLAUDE.md` | はい | 現在の作業ディレクトリ |
| `/mymonorepo/frontend/CLAUDE.md` | いいえ | `frontend/` のファイルを読み書きした時のみロード |
| `/mymonorepo/backend/CLAUDE.md` | いいえ | `backend/` のファイルを読み書きした時のみロード |
| `/mymonorepo/api/CLAUDE.md` | いいえ | `api/` のファイルを読み書きした時のみロード |

### シナリオ 2: コンポーネントディレクトリから Claude Code を実行

`/mymonorepo/frontend/` から Claude Code を実行する場合:

```bash
cd /mymonorepo/frontend
claude
```

| ファイル | 起動時にロード？ | 理由 |
|---------|----------------|------|
| `/mymonorepo/CLAUDE.md` | はい | 祖先ディレクトリ |
| `/mymonorepo/frontend/CLAUDE.md` | はい | 現在の作業ディレクトリ |
| `/mymonorepo/backend/CLAUDE.md` | いいえ | ディレクトリツリーの別ブランチ |
| `/mymonorepo/api/CLAUDE.md` | いいえ | ディレクトリツリーの別ブランチ |

### 重要なポイント

1. **祖先は常に起動時にロード** — Claude はディレクトリツリーを上方向に走査し、見つかったすべての CLAUDE.md ファイルをロードします。これにより、ルートレベルのリポジトリ全体の指示に常にアクセスできます。

2. **子孫は遅延ロード** — サブディレクトリの CLAUDE.md ファイルは、そのサブディレクトリのファイルとやり取りした時にのみロードされます。これにより、無関係なコンテキストがセッションを肥大化させるのを防ぎます。

3. **兄弟はロードされない** — `frontend/` で作業中、`backend/CLAUDE.md` や `api/CLAUDE.md` はコンテキストにロードされません。

4. **グローバル CLAUDE.md** — ホームフォルダの `~/.claude/CLAUDE.md` に CLAUDE.md を配置することもでき、プロジェクトに関係なくすべての Claude Code セッションに適用されます。

### モノレポでこの設計が機能する理由

- **共有指示が下方に伝播** — ルートレベルの CLAUDE.md にはすべての場所に適用されるリポジトリ全体の規約、コーディング標準、共通パターンが含まれます。

- **コンポーネント固有の指示が分離される** — フロントエンド開発者はバックエンド固有の指示でコンテキストが散らかることなく、その逆も同様です。

- **コンテキストが最適化される** — 子孫の CLAUDE.md ファイルを遅延ロードすることで、Claude Code は起動時に潜在的に数百キロバイトの無関係な指示のロードを回避します。

### ベストプラクティス

1. **共有の規約はルート CLAUDE.md に配置** — コーディング標準、コミットメッセージ形式、PR テンプレート、その他のリポジトリ全体のガイドライン。

2. **コンポーネント固有の指示はコンポーネント CLAUDE.md に配置** — フレームワーク固有のパターン、コンポーネントアーキテクチャ、そのコンポーネント独自のテスト規約。

3. **個人の好みには CLAUDE.local.md を使用** — チームと共有すべきでない指示のため `.gitignore` に追加。

---

## ソース

- [Claude Code ドキュメント - メモリの検索方法](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny on X - CLAUDE.md ローディングの説明](https://x.com/bcherny/status/2016339448863355206)
- [Humanlayer - 良い Claude.md の書き方](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
