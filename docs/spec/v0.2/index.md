# Page Block Syntax (PBS) Specification v0.2

> **変更概要 (v0.2)**
>
> * ステートブロックのネスト例を追加
> * ページ間状態共有 (StateRef) の構文例を追加
> * グローバル vs ローカル State の使い分け指針を明記
> * コンポーネント定義 & 再利用セクション (5.2) を新設
> * EBNF を暫定完成版に更新

---

## 1. はじめに

Page Block Syntax (以下 **PBS**) は、アプリケーション仕様を **ページ (Page)** 単位で記述し、**レイアウト・フロー・ステート** の 3 観点で整理する軽量マークアップです。*人→人* と *人→LLM／LLM→人* の情報伝達における曖昧さを最小化し、スパイラル型設計を高速に回すことを目的とします。

---

## 2. 設計目標

1. **可読性** - 非技術者でも眺めて理解できるプレーンテキスト。
2. **記述容易性** - Markdown ライクな最小限の記号で完結。
3. **拡張性** - ページ追加・分割・結合が容易。
4. **LLM フレンドリー** - 構文が一意にパースでき、生成 AI が補完しやすい。

---

## 3. ドキュメント階層

```
<ファイル*>
└─ (App)                アプリケーション定義       0..1
    ├─ [Page]           ページ定義                 1..*
    │   ├─ [[Layout]]   ページレイアウト            0..1
    │   ├─ (Layout){}   レイアウトブロック          0..*
    │   ├─ (Flow){}     フローブロック             0..*
    │   └─ (State){}    ステートブロック           0..*
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
    - Logo : component
    - Spacer
    - (Button) Login : primary
```

* インデント = 親子関係
* 左→右, 上→下 の順で列挙
* `VStack` / `HStack` / `Spacer` で Flex 方向を制御
* `:` でメモ・装飾・振る舞いを追記

### 5.1 ページレイアウトの適用

```pbs
[[DefaultLayout]]
  - (VStack)
    - Header
    - ContentArea
    - Footer

[Dashboard] : uses [[DefaultLayout]]
  (Layout) ContentArea
    - ChartArea
    - RecentActivities
```

### 5.2 コンポーネント定義 & 再利用

コンポーネントは `(Component)` ブロックで宣言し、以後任意のレイアウトで名前参照が可能。

```pbs
(Component) Logo
  // SVG や画像 URL などをメタ情報に
  src: /assets/logo.svg

(Layout) Header
  - Logo        // 宣言済みコンポーネントを直接利用
  - Spacer
  - (Button) Login
```

* コンポーネントは UI 部品カタログとして再利用
* レイアウト内参照時は単に名前を記載 (慣例的に先頭大文字)

---

## 6. フローブロック `(Flow)`

ビジネスロジックや状態遷移を箇条書きで記述。

```pbs
(Flow) LoginFlow
  - On Click(Button Login) ->
      ValidateCredentials
      ? success -> Navigate [Dashboard]
      ? failure -> ShowError "Invalid credentials"
```

---

## 7. ステートブロック `(State)`

状態やドメインモデルを JSON ライクにリスト化。

```pbs
(State) User
  id        : UUID
  name      : String
  isLoggedIn: Boolean = false

// ▼ ネスト & 配列の例
(State) Order
  id        : UUID
  user      : UserRef
  items[]   :
    - productId : UUID
      qty       : Int
  shipping  :
    address : String
    eta     : Date
```

* インデントでネストを表現
* `items[]` のように `[]` を付けると配列
* `=` で初期値を指定

---

## 8. ページ間の状態共有

### 8.1 Global State とローカル State

| 用途                 | ブロック             | 推奨スコープ               |
| ------------------ | ---------------- | -------------------- |
| アプリ全体で共有したい永続的データ  | `(Global State)` | 例: 認証情報、テーマ設定        |
| ページ固有、またはページ間限定で共有 | `(State)`        | 例: 一時入力データ、ウィザード途中状態 |

* **原則**: 迷ったらローカル State とし、複数ページからアクセスが必要になったら Global へ昇格。

### 8.2 State 参照 (StateRef) 構文

```pbs
// Cart ページ (発生元)
[Cart]
  (State) CartState
    items[] : ProductRef

// Checkout ページ (参照先)
[Checkout]
  (StateRef) CartState from [Cart] : readonly
  (Flow) CheckoutFlow
    - On Click(Button Pay) ->
        ChargePayment CartState.items
        Navigate [Thanks]
```

* `(StateRef)` で **参照 (read‑write / readonly)** を明示
* `from [Page]` で元ページを指定し、LLM が依存グラフを追跡可能

---

## 9. コメント & メタ情報

行頭 `//` は PBS パーサが無視します。
任意で `@author` `@version` などタグを追加可能。

---

## 10. サンプル: TODO アプリ

```pbs
(App) TinyTODO : 個人用 To‑Do 管理アプリ

[[MainLayout]]
  - (VStack)
    - Header
    - Content
    - Spacer
    - Footer

// ▼ ページ
[Home] : uses [[MainLayout]]
  (Layout) Content
    - (HStack)
      - (Input) NewTodoText : placeholder="タスクを入力"
      - (Button) Add : primary
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

| 目的      | 書式                            | 例                                  |
| ------- | ----------------------------- | ---------------------------------- |
| ページ定義   | `[PageName]`                  | `[Settings]`                       |
| レイアウト挿入 | `(Layout) Name`               | `(Layout) Sidebar`                 |
| フロー定義   | `(Flow) Name`                 | `(Flow) AuthFlow`                  |
| ステート定義  | `(State) Name`                | `(State) CartItem`                 |
| ステート参照  | `(StateRef) Name from [Page]` | `(StateRef) CartState from [Cart]` |
| 配列      | `items[]`                     | `products[]`                       |
| ページ遷移   | `Navigate [Page]`             | `Navigate [Home]`                  |

---

## 12. EBNF (暫定完成版)

```ebnf
document     = { app | page | pagelayout | block | comment } ;
app          = "(App)" , text , [ ":" , text ] , nl ;
page         = "[" , text , "]" , [ ":" , text ] , nl ;
pagelayout   = "[[" , text , "]]" , nl ;
block        = layout | flow | state | stateref | component | globalstate ;
layout       = "(Layout)" , text , nl , item+ ;
flow         = "(Flow)" , text , nl , flowline+ ;
state        = "(State)" , text , nl , stateline+ ;
stateref     = "(StateRef)" , text , "from" , pageRef , [":" , "readonly" ] , nl ;
component    = "(Component)" , text , nl , { property } ;
globalstate  = "(Global State)" , text , nl , stateline+ ;
item         = indent , "-" , ( element | layout | componentRef ) , [":" , text ] , nl ;
flowline     = indent , ( "On" , trigger , "->" , actionSeq ) | ( "?" , condition , "->" , actionSeq ) , nl ;
stateline    = indent , key , ":" , type , [ "=" , literal ] , nl ;
comment      = "//" , .* , nl ;
text         = /[^\n\r]+/ ;
indent       = { "  " } ;            // 2 spaces per level
nl           = /\r?\n/ ;
```

*簡潔さ重視のため、文字列・識別子の細部は正規表現で省略しています。*