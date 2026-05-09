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

ルールが導入される章は以下の通り（PDF 目次より）:

| 章 | 導入されるルール |
|----|------|
| 第3章 Shatter long functions | Rule 1 Five Lines / Rule 2 Either Call Or Pass / Rule 3 If Only At The Start |
| 第4章 Make type codes work | Rule 4 Never Use If With Else / Rule 5 Never Use Switch / Rule 6 Only Inherit From Interfaces |
| 第5章 Fuse similar code together | Rule 7 Use Pure Conditions / Rule 8 No Interface With Only One Implementation |
| 第6章 Defend the data | Rule 9 Do Not Use Getters Or Setters / Rule 10 Never Have Common Affixes（書籍参照で要確認） |

### 書籍が提示するリファクタリングパターン（第1部）

ルールは「何が悪いか」を判定し、パターンは「どう直すか」を与える。第1部 (3〜6章) で導入されるパターンは以下（書籍 inside front cover より）:

| 章 | パターン名 | 1行説明 |
|----|----------|--------|
| 3章 | Extract Method | 関数の一部を別関数へ抽出 |
| 4章 | Replace Type Code With Classes | enum をインターフェース＋クラス群に変換 |
| 4章 | Push Code Into Classes | 振る舞いをクラスに移す（Replace Type Code の自然な続き） |
| 4章 | Inline Method | 可読性に寄与しなくなった関数を展開 |
| 4章 | Specialize Method | 不要な汎用性を取り除く |
| 4章 | Try Delete Then Compile | スコープが分かっている未使用メソッドを削除 |
| 5章 | Unify Similar Classes | 定数メソッド差分のクラス群を統合 |
| 5章 | Combine Ifs | 同一本体の連続 if を結合 |
| 5章 | Introduce Strategy Pattern | if による分岐をクラスのインスタンス化に置換 |
| 5章 | Extract Interface From Implementation | クラス依存をインターフェース依存へ |
| 6章 | Eliminate Getter Or Setter | データに振る舞いを寄せて getter/setter を消す |
| 6章 | Encapsulate Data | 変数に関わる不変条件を局所化 |
| 6章 | Enforce Sequence | 実行順序をコンパイラに保証させる |

コミットメッセージにはルール番号とパターン名の両方を含める（後述「コミット運用」参照）。

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

具体的な良い例 / 悪い例:

- 悪い例: 「Extract Method ついでに `playerx` を `playerX` にリネーム」→ 1コミットが2つの目的を持ち、書籍の段階追従が崩れる
- 良い例: Extract Method のみ実施 → コミット → 次の指示で rename を別コミット

### Claude のリファクタリング進行ワークフロー

1. **提案前にルール宣言**: 変更を提案する際、適用するルール番号とパターン名を先に書く（コミットメッセージと同じ語彙）
2. **書籍を読んでから手を動かす**: コードを変更する前に必ず書籍 PDF の該当箇所を読み、「これは○章○節・パターン名『XXX』に対応する」と特定してから実施する。宣言なしにコードを変更しない
3. **1ステップ = 1パターン = 1コミット**: Extract Method の最中に、変数リネーム / 型注釈追加 / 別ルール違反の修正 を混ぜない。`diff` を最小に保つ
4. **振る舞いを変えない**: スクリーンショット (`game.png`, `goal.png`) と同じ動作を維持
5. **次のスメル示唆は1個まで**: 1コミット完了時、副作用として見えるようになった次のスメルを **1個だけ** 言及してよい。ただし手は出さない（ユーザーが指示するまで待つ）
6. **能動的に直し始めない**: ユーザーが章/ルール/対象関数を指定するまで、Claude 側からスメル除去を開始しない
7. **書きながら解説する**: コード変更を実施する各ステップで「これが何の操作か (パターン名/ルール番号)」「なぜこの順序なのか」「どこが学習上のポイントか」を **編集と同じターン内で日本語で説明する**。ユーザーの目的は「コードを書き終わること」ではなく「書籍が示す思考過程を追体験すること」なので、無言で diff を出すのは目的に反する。各 Edit/Write の前後に短く意図を述べる、コミット直後に「このコミットで身についた観点」を 1〜3 行で要約する、書籍と現状コードのギャップ (「書籍はここで X と言っているが、本リポジトリは既に Y まで進んでいるので Z だけ実施」) を明示する、といった形で**ユーザーが今何を学んでいるかを言語化**する

### PDF (原書) を参照する基準

`docs/five-lines-of-code.pdf` は『Five Lines of Code』原書 (610 ページ) の全文 PDF。
**深掘りが必要なときだけ**読み、普段は CLAUDE.md の要約で済ませる。

**参照すべきタイミング:**

- ユーザーが「もう少し詳しく」「書籍ではどう説明している?」「原書では?」など**根拠を求めた**とき
- ルールやパターンの**意図 / Why / 例外条件**が CLAUDE.md の1行説明では足りないとき
- Claude 自身が提案内容に**確信が持てない**とき (例: あるパターンの適用順序、ルール間の優先順位)
- ユーザーが**章番号 / ルール番号 / パターン名を直接指定**して質問したとき

**参照しなくてよいケース:**

- 単純な Extract Method など CLAUDE.md の要約だけで判断できるリファクタリング指示
- コミットメッセージの体裁、コマンド実行など本書の主題から外れた作業
- ユーザーが「PDF は見なくていい」と明示したとき

**読み方の制約:**

- PDF は 1 リクエスト最大 20 ページ。`Read` ツールに `pages: "X-Y"` を必ず指定する (省略すると失敗)
- 該当箇所を狙い撃ちで読む。先頭から流し読みしない
- 章番号と PDF ページ番号は一致しないので、目次 (前付近) で当たりを付けてから本文を読む

**引用・提示のしかた:**

- **ページ番号付きで要約**して提示する。例:「PDF p.87-88 の要約: Extract Method を適用する判断基準は…」
- 原文の長文コピーは避ける (著作物のため)。短いキーワード/定義の抜粋に留める
- 引用とユーザーの状況に応じた解釈を区別して書く ("書籍はこう述べている" / "今回のコードに当てはめると")
- 章節番号 (例: P3.2.1) が分かるなら併記すると、ユーザーが紙の本でも追える

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
