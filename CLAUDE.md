# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの位置づけ

Christian Clausen 著『**Five Lines of Code**』(Manning, 2021) / 邦訳『**実践で学ぶコード改善の極意 — 5行ルールで強く美しくリファクタリングする**』(松田晃一 訳, マイナビ出版, 2025) の第1部の題材となる 2D パズルゲーム ("Gatherer") のスターターコード。

ユーザーは書籍を読み進めながら手を動かして学習する目的で使用する。したがって作業は基本的に **`index.ts` のリファクタリング** であり、新機能追加や別アプリ化ではない。

書籍の章構成・10 のルール・パターン表・既知のスメル一覧などの参照情報は **[docs/book-reference.md](docs/book-reference.md)** にまとめてある。Claude は必要時にこのファイルを Read する。

## Claude の行動規範

書籍は **「1ステップ = 1 リファクタリングパターン適用 = 即コミット」** という非常に細かい粒度を推奨する。複数のルール違反を一気に直そうとすると学習価値が下がるため、以下を厳守する。

1. **書籍を読んでから手を動かす**: コードを変更する前に必ず PDF (`docs/five-lines-of-code.pdf`) の該当箇所を読み、「これは○章○節・パターン名『XXX』に対応する」と特定してから実施する。宣言なしにコードを変更しない。
2. **ルール番号 + パターン名を変更前に宣言する**: 提案時に適用するルール番号とパターン名を先に書く（コミットメッセージと同じ語彙）。
3. **1ステップ = 1パターン = 1コミット**: Extract Method の最中に、変数リネーム / 型注釈追加 / 別ルール違反の修正 を混ぜない。`diff` を最小に保つ。
4. **書きながら日本語で解説する**: 各 Edit/Write の前後に「これが何の操作か（パターン名/ルール番号）」「なぜこの順序なのか」「学習上のポイント」を短く述べる。ユーザーの目的は「書籍が示す思考過程を追体験すること」なので、無言で diff を出すのは目的に反する。
5. **振る舞いを変えない**: スクリーンショット (`game.png`, `goal.png`) と同じ動作を維持する。
6. **能動的に直し始めない**: ユーザーが章 / ルール / 対象関数を指定するまで、Claude 側からスメル除去を開始しない。
7. **次のスメル示唆は1個まで**: 1 コミット完了時、副作用として見えるようになった次のスメルを **1 個だけ** 言及してよい。ただし手は出さない（ユーザーが指示するまで待つ）。

例: 「第3章のルール 1 (Five Lines) を `moveHorizontal` に適用して」と言われたら、Extract Method を 1〜2 回行うだけで止め、別のルール違反は触らない。

- 悪い例: 「Extract Method ついでに `playerx` を `playerX` にリネーム」→ 1 コミットが 2 つの目的を持ち、書籍の段階追従が崩れる
- 良い例: Extract Method のみ実施 → コミット → 次の指示で rename を別コミット

## コミット運用

ユーザーは学習履歴として残したいので、**「適用したリファクタリングパターン名 + 違反していたルール番号」をコミットメッセージに含める**:

```
Extract moveStone from moveHorizontal (Rule 1 / Extract Method)
Replace Tile enum branch with polymorphism (Rule 4-5 / Replace Type Code with Classes)
```

## 開発コマンド

```bash
mise install                    # Node 24.12.0 をインストール（mise 利用時）
npm install                     # devDependency の TypeScript 6.0.3 を取得
npm run build                   # tsc 実行 → index.js を生成
npm run watch                   # ファイル変更を監視して自動コンパイル
npm run serve                   # python3 -m http.server 8080（http://localhost:8080）
```

ブラウザで `index.html` を開けばゲームが動く。**`index.ts` を編集するたびに `tsc` を再実行**しないと反映されない（`watch` を別ペインで回しておくのが楽）。

## 動作確認

テストフレームワークは無い。動作確認はブラウザでの手動プレイのみ。Claude が変更した場合は、ユーザーに「ブラウザで動作確認してください」と促す。スクリーンショット (`game.png`, `goal.png`) のとおりに動くことが正解。
