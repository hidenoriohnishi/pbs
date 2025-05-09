# Page Block Syntax (PBS) Specification v0.1

## 1. はじめに

Page Block Syntax (以下 **PBS**) は、アプリケーション仕様を **ページ (Page)** 単位で記述し、レイアウト・フロー・ステートという 3 つの観点から整理する軽量マークアップです。*人→人* や *人→LLM*／*LLM→人* の情報伝達で曖昧さを最小化し、スパイラル型の設計プロセスを素早く回すことを目的とします。

---

## 2. 設計目標

1. **可読性** ‑ 非技術者でも眺めて理解できるプレーンテキスト。
2. **記述容易性** ‑ Markdown ライクな最小限の記号で完結。
3. **拡張性** ‑ ページ追加・分割・結合が容易。
4. **LLM フレンドリー** ‑ 構文が一意にパースでき、生成 AI が補完しやすい。

---

## 3. ドキュメント階層

```
<ファイル*>
└─ (App)                アプリケーション定義       0..1
    ├─ [Page]           ページ定義                 1..*
    │   ├─ [[Layout]]   ページレイアウト            0..1
    │   ├─ (Layout){}   レイアウトブロック          0..*
    │   ├─ (Flow){}     フローブロック             0..*
    │   └─ (State){}    ステートブロック           0..*
    └─ (Global State)   アプリ全体の状態            0..*
```

*ファイル分割は自由。複数ファイルでも単一ファイルでも動作します。*

---

## 4. 共通記法

| トークン                       | 意味                  | 例                     |
| -------------------------- | ------------------- | --------------------- |
| `(Name)`                   | セクション / グループ / コンテナ | `(Header)`            |
| `[Page]`                   | ページ                 | `[Login]`             |
| `[[Name]]`                 | ページレイアウト            | `[[DefaultLayout]]`   |
| `:`                        | 直前要素へのノート           | `[Login] : 非ログイン時に表示` |
| `-`                        | 箇条書き (インデントが階層を示す)  | `- (Button) Login`    |
| `VStack`,`HStack`,`Spacer` | レイアウト補助             | `- (HStack)`          |
| `//`                       | 行コメント               | `// メモ`               |

---

## 5. レイアウトブロック `(Layout)`

```
(Layout) Header
  - (HStack)
    - Logo
    - Spacer
    - (Button) Login : primary
```

* インデント = 親子関係
* 左→右, 上→下 の順に並べた時系列で列挙
* `VStack` / `HStack` / `Spacer` で Flex 方向を制御
* 任意の要素に `:` でメモ・装飾・振る舞いを追記

### 5.1 ページレイアウトの適用

```
// レイアウト定義
[[DefaultLayout]]
  - (VStack)
    - Header
    - ContentArea
    - Footer

// ページで利用
[Dashboard] : uses [[DefaultLayout]]
  (Layout) ContentArea
    - ChartArea
    - RecentActivities
```

---

## 6. フローブロック `(Flow)`

ビジネスロジックや状態遷移を箇条書きで記述します。

```
(Flow) LoginFlow
  - On Click(Button Login) ->
      ValidateCredentials
      ? success -> Navigate [Dashboard]
      ? failure -> ShowError "Invalid ID or PW"
```

* `On <Trigger> ->` で開始
* `? <条件> ->` で分岐
* `Navigate [Page]` でページ遷移
* アクションは CamelCase で自由命名

---

## 7. ステートブロック `(State)`

状態やドメインモデルを JSON‑like にリスト化。

```
(State) User
  id        : UUID
  name      : String
  isLoggedIn: Boolean = false
```

* `=` で初期値
* 型は任意の英字列 (String, Int, UUID など)
* ネストはインデントで表現

---

## 8. アプリケーション全体の連携

* グローバルステート `(Global State)` はファイル冒頭や専用ファイルにまとめる。
* ページ間通信は **State の参照** または **Navigate** アクション経由で明示。
* ドキュメント横断リンクはページ名で一意に行う。

---

## 9. コメント & メタ情報

行頭 `//` は PBS パーサが無視します。
特殊タグ `@author` `@version` などを自由に定義可能。

---

## 10. サンプル: シンプルな TODO アプリ

```
(App) TinyTODO : 個人用 To‑Do 管理アプリ

[[MainLayout]]
  - (VStack)
    - Header
    - Content
    - Spacer
    - Footer

[Home] : uses [[MainLayout]]
  (Layout) Content
    - (HStack)
      - (Input) NewTodoText : placeholder="タスクを入力"
      - (Button) Add       : primary
    - (List) TodoItems

  (Flow) HomeFlow
    - On Click(Button Add) ->
        AddTodo NewTodoText
        ClearInput NewTodoText

(State) Todo
  id   : UUID
  text : String
  done : Boolean = false
```

---

## 11. チートシート

| 目的      | 書式                | 例                  |
| ------- | ----------------- | ------------------ |
| ページ定義   | `[PageName]`      | `[Settings]`       |
| レイアウト挿入 | `(Layout) Name`   | `(Layout) Sidebar` |
| フロー定義   | `(Flow) Name`     | `(Flow) AuthFlow`  |
| ステート定義  | `(State) Name`    | `(State) CartItem` |
| ページ遷移   | `Navigate [Page]` | `Navigate [Home]`  |
| 条件分岐    | `? condition ->`  | `? isValid ->`     |

---

## 12. EBNF (参考実装用)

```
document   ::= (app | block | comment)* ;
app        ::= '(App)' text (':' text)? ;
block      ::= layout | flow | state ;
layout     ::= '(Layout)' text newline item+ ;
flow       ::= '(Flow)' text newline flowline+ ;
state      ::= '(State)' text newline stateline+ ;
page       ::= '[' text ']' (':' text)? ;
pagelayout ::= '[[' text ']]' ;
item       ::= indent '-' (pagelayout | page | layout | element) (':' text)? newline ;
comment    ::= '//' .* newline ;
```