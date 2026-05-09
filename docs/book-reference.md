# 書籍リファレンス

このファイルは『Five Lines of Code』(Christian Clausen, Manning, 2021) / 邦訳『実践で学ぶコード改善の極意』(松田晃一 訳, マイナビ出版, 2025) に関する**参照用メタ情報**を集約したもの。

`CLAUDE.md` は毎セッションで Claude のコンテキストに自動で載るが、このファイルは載らない。Claude/ユーザーは必要時に明示的に Read する。

## 14 章構成（リファクタリングが進む順序）

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

## 10 のルール

リファクタリング判断は「コードスメル」のような直観ではなく、以下の機械的ルールで行う。コミットメッセージで違反していたルール名を明示すると振り返りやすい。

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

ルールが導入される章:

| 章 | 導入されるルール |
|----|------|
| 第3章 Shatter long functions | Rule 1 Five Lines / Rule 2 Either Call Or Pass / Rule 3 If Only At The Start |
| 第4章 Make type codes work | Rule 4 Never Use If With Else / Rule 5 Never Use Switch / Rule 6 Only Inherit From Interfaces |
| 第5章 Fuse similar code together | Rule 7 Use Pure Conditions / Rule 8 No Interface With Only One Implementation |
| 第6章 Defend the data | Rule 9 Do Not Use Getters Or Setters / Rule 10 Never Have Common Affixes（書籍参照で要確認） |

## 第 1 部のリファクタリングパターン

ルールは「何が悪いか」を判定し、パターンは「どう直すか」を与える（書籍 inside front cover より）。

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

## PDF 参照ガイド

`docs/five-lines-of-code.pdf` は原書 (610 ページ) の全文 PDF。

**参照すべきタイミング:**

- ユーザーが「もう少し詳しく」「書籍ではどう説明している?」など根拠を求めたとき
- ルールやパターンの**意図 / Why / 例外条件**が要約だけでは足りないとき
- 提案内容に確信が持てないとき（例: パターンの適用順序、ルール間の優先順位）
- ユーザーが**章番号 / ルール番号 / パターン名を直接指定**して質問したとき

**参照しなくてよいケース:**

- 単純な Extract Method など要約だけで判断できる指示
- コミットメッセージの体裁、コマンド実行など本書の主題から外れた作業
- ユーザーが「PDF は見なくていい」と明示したとき

**読み方の制約:**

- PDF は 1 リクエスト最大 20 ページ。`Read` ツールに `pages: "X-Y"` を必ず指定する（省略すると失敗）
- 該当箇所を狙い撃ちで読む。先頭から流し読みしない
- 章番号と PDF ページ番号は一致しないので、目次で当たりを付けてから本文を読む

**引用・提示のしかた:**

- ページ番号付きで要約して提示する。例:「PDF p.87-88 の要約: …」
- 原文の長文コピーは避ける（著作物のため）。短いキーワード/定義の抜粋に留める
- 引用とユーザーの状況に応じた解釈を区別する（"書籍はこう述べている" / "今回のコードに当てはめると"）
- 章節番号（例: P3.2.1）が分かるなら併記する

## 既知のスメル一覧（リファクタ対象の地図）

`index.ts` には学習用に意図的に残されたスメルが多数ある。早回しに直さないこと。

- `moveHorizontal` / `moveVertical` の長大な if-else（ルール 1, 3, 4 違反）
- `update` 内の重複したループ＋if-else（ルール 5 相当の switch 的構造）
- `Tile` enum を大量の同値比較で扱う「タイプコード」（4章で多態に置き換える）
- `playerx` / `playery` などのモジュールスコープ可変変数（6章）
- `draw` 内の `g.fillStyle` 設定の繰り返し（5章で統合）
- `KEY1/LOCK1` と `KEY2/LOCK2` の対称な重複（5章のテーマ）

## TypeScript 設定の癖

`tsconfig.json` は `target: "es5"`, `module: "commonjs"`, `strictNullChecks: false`。書籍出版時点（TS 4.x）に合わせており、`strictNullChecks` が無効なため**コンパイラが null 起因のバグを検出しない**。第6〜7章 で型による防御を学ぶ過程で、必要に応じて strict 系フラグを段階的に有効化していくのも有効な学習方法。

`ignoreDeprecations: "6.0"` は TS 6.x で削除予定の `target: es5` を引き続き使うための回避策。

`.gitignore` の注意: `*.js` が ignore 対象。`index.js` はコミットしない（`tsc` の生成物）。同様に `*.sh` も ignore。
