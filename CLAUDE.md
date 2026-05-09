# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの位置づけ

このリポジトリは Christian Clausen 著『**Five Lines of Code**』(Manning, 2021) / 邦訳『**実践で学ぶコード改善の極意 — 5行ルールで強く美しくリファクタリングする**』(松田晃一 訳, マイナビ出版, 2025) の第1部の題材となる 2D パズルゲーム ("Gatherer") のスターターコードである。書籍はこの `index.ts` を**章ごとに段階的にリファクタリング**していく形式で進む。

ユーザーは書籍を読み進めながら手を動かして学習する目的でこのリポジトリを使う。したがって作業は基本的に **`index.ts` のリファクタリング** であり、新機能追加や別アプリ化ではない。

### 書籍の14章構成（リファクタリングが進む順序）

1. Refactoring refactoring
2. Looking under the hood of refactoring
3. Shatter long functions（長い関数を分割する）
4. Make type codes work（タイプコードを機能させる）
5. Fuse similar code together（似たコードを統合する）
6. Defend the data（データを守る）
7. Collaborate with the compiler（コンパイラと協調する）
8. Stay away from comments（コメントから離れる）
9. Love deleting code（コードを削除することを愛する）
10. Never be afraid to add code（コードを追加することを恐れない）
11. Follow the structure in the code（コードの構造に従う）
12. Avoid optimizations and generality（最適化と汎用性を避ける）
13. Make bad code look bad（悪いコードは悪く見えるようにする）
14. Wrapping up

第1部 (3〜6章) でこの `index.ts` を題材にリファクタリングを学び、第2部 (7〜13章) で実務応用を扱う。

### 書籍が提示する10のルール

リファクタリング判断は「コードスメル」のような直観ではなく、以下の機械的ルールで行う。**学習中はこれらルールを優先し、PR/コミットで違反するルール名を明示する**と振り返りやすい。

1. **Five Lines** — メソッドは5行以内（空白行・`{`/`}` 行は除く）
2. **Either Call Or Pass** — 関数本体では「他関数を呼ぶ」か「引数をそのまま渡す」のどちらかに揃える（同じ抽象レベル）
3. **If Only At The Start** — `if` は関数の先頭でのみ許される（早期 return 専用）
4. **Never Use If With Else** — `if-else` を見たら多態（Strategy/State）に置き換える
5. **Never Use Switch** — `switch` も同様。enum + ポリモーフィズムへ
6. **Only Inherit From Interface** — クラス継承は禁止。継承するならインターフェースのみ
7. **Use Pure Conditions** — 条件式に副作用を持たせない
8. **No Interface With Only One Implementation** — 実装が1つしかないインターフェースは作らない（YAGNI）
9. **Do Not Use Getters Or Setters** — データを「尋ねる」のではなく「振る舞いを依頼する」(Tell, Don't Ask)
10. **Never Have Common Affixes** — 共通の接頭辞・接尾辞は冗長 (`AbstractFooImpl` のような命名を避ける)

第3章 = ルール 1〜3、第4章 = ルール 4〜6、以降の章で残りが順次導入される（厳密な対応は書籍参照）。

## 開発コマンド

```bash
mise install                    # Node 24.12.0 をインストール（mise 利用時）
npm install                     # devDependency の TypeScript 6.0.3 を取得
npm run build                   # tsc 実行 → index.js を生成
npm run watch                   # ファイル変更を監視して自動コンパイル
npm run serve                   # python3 -m http.server 8080（http://localhost:8080）
```

ブラウザで `index.html` を開けばゲームが動く。**`index.ts` を編集するたびに `tsc` を再実行**しないと反映されない（`watch` を別ペインで回しておくのが楽）。テストフレームワークは無い。動作確認はブラウザでの手動プレイのみ。

### TypeScript 設定の癖

`tsconfig.json` は `target: "es5"`, `module: "commonjs"`, `strictNullChecks: false`。書籍の出版時点（TS 4.x）に合わせており、`strictNullChecks` が無効なため**コンパイラが null 起因のバグを検出しない**。リファクタリング 第6〜7章 で型による防御を学ぶ過程で、必要に応じて strict 系フラグを段階的に有効化していくのも有効な学習方法。

`ignoreDeprecations: "6.0"` は TS 6.x で削除予定の `target: es5` を引き続き使うための回避策。

### .gitignore に注意

`*.js` が ignore 対象。`index.js` はコミットしない（`tsc` の生成物）。同様に `*.sh` も ignore。

## このコードベースで Claude が意識すべきこと

### リファクタリングの粒度

書籍は **「1ステップ = 1リファクタリングパターン適用 = 即コミット」** という非常に細かい粒度を推奨する。Claude が複数のルール違反を一気に直そうとすると学習価値が下がる。**ユーザーが章/ルールを指定したら、その範囲だけを最小限に変更し、コミット単位を分けやすい形で提示する**こと。

例: 「第3章のルール 1 (Five Lines) を `moveHorizontal` に適用して」と言われたら、Extract Method を1〜2回行うだけで止め、別のルール違反は触らない。

### 既知のスメル一覧（リファクタ対象の地図）

`index.ts` には学習用に意図的に残されたスメルが多数ある。早回しに直さないこと:

- `moveHorizontal` / `moveVertical` の長大な if-else（ルール1, 3, 4 違反）
- `update` 内の重複したループ＋if-else（ルール5 相当の switch 的構造）
- `Tile` enum を大量の同値比較で扱う「タイプコード」（4章で多態に置き換える）
- `playerx` / `playery` などのモジュールスコープ可変変数（6章）
- `draw` 内の `g.fillStyle` 設定の繰り返し（5章で統合）
- `KEY1/LOCK1` と `KEY2/LOCK2` の対称な重複（5章のテーマ）

### コミット運用

ユーザーは学習履歴として残したいので、**「適用したリファクタリングパターン名 + 違反していたルール番号」をコミットメッセージに含める**スタイルを推奨。例:

```
Extract moveStone from moveHorizontal (Rule 1 / Extract Method)
Replace Tile enum branch with polymorphism (Rule 4-5 / Replace Type Code with Classes)
```

### 「動くこと」を壊さない

書籍の全工程は**振る舞いを変えないリファクタリング**。スクリーンショット (`game.png`, `goal.png`) のとおりに動くことが正解。Claude が変更する場合は、ユーザーに「ブラウザで動作確認してください」と促す（自動テストが無いため）。
