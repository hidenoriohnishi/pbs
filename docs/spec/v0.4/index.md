# Page Block Syntax (PBS) Specification **v0.4**
*Last updated: 2025‑05‑15*

---

## 目的
PBS (Page Block Syntax) は、アプリケーションの設計に関わる多様な関心事を、必要とされる粒度で、少ない労力と短い記述で正確に表現することを目的とした軽量マークアップ言語です。

特に、アプリケーションを**ページ単位**で捉え、各ページの**Layout (構造と配置) / Flow (状態遷移とインタラクション) / State (保持するデータ)** の3つの主要観点を、Markdown風の直感的な記法で整理します。これにより、人間同士のコミュニケーションはもちろん、人間とLLM（大規模言語モデル）間の情報伝達コストを最小限に抑え、プロトタイピングや反復的な設計プロセス（スパイラル開発）における試行錯誤を、より正確かつ迅速に進めることを支援します。

---

## 設計原則
PBSの設計は、以下の5つの原則に基づいています。これらは `@Concept.md` に記述されたPBSの設計思想を具体化したものです。

1.  **表現力と簡潔性 (Expressiveness & Conciseness)**
    設計に必要な情報を、適切な詳細度で、かつ無駄なく記述できることを目指します。少ない記述量で多くの情報を正確に伝えることを重視します。

2.  **制約による思考負荷の低減と記述容易性 (Reduced Cognitive Load by Constraints & Ease of Writing)**
    最小限の文法ルールが「考え方の型」を示し、選択肢を整理することで設計判断の迷いを減らします。記号の種類を絞り、一貫性を持たせることで、学習しやすく、書きやすい記法を実現します。

3.  **ゆるやかな構造フレーム (Flexible Structural Framework)**
    プログラミング言語のような明確な構造を持ちつつも、文脈から意味が読み取れる範囲での"クリエイティブな崩し"や省略を許容します。これにより、厳格なパース（構文解析）を常に前提とせず、初期のアイデア発想段階から詳細設計まで、幅広いフェーズで利用可能です。

4.  **構造一貫性と拡張性 (Structural Consistency & Extensibility)**
    衝突しにくいラベル体系と行指向のフォーマットにより、要素の再参照や変更追跡（差分フレンドリネス）を容易にします。また、ページの追加、分割、再統合といった変更にも柔軟に対応できる拡張性を持ちます。

5.  **LLM親和性と視覚的明瞭性 (LLM-Friendliness & Visual Clarity)**
    予約語や境界記号を厳選し、トークン効率を考慮した独自のシグナル（例: `@Block`, `<ComponentName>`）を用いることで、LLMによる解釈・生成の精度を高めます。同時に、人間にとっても速読に適した視覚的なフォーマットを提供し、可読性を確保します。

---

## ドキュメント構造
PBSは、原則として1ファイルで1つのアプリケーションの設計を記述します。

```pbs
<file*>
└─ # App                         0..1
   ├─ @State                   0..*
   ├─ @Templates                     0..1
   ├─ @Components                      0..*
   ├─ @Flow               0..*
   └─ ## Page                          1..*
       ├─ @State                       0..*
       ├─ @Layout                     1..*
       └─ @Flow                        0..*
```

---

## 構文・記法の厳密な定義

PBSは「ゆるやかな構造フレーム」を設計原則の一つとしており、必ずしも決定的なパースを前提としません。ここで示すEBNFは、PBSの主要な構造と推奨される記述形式を理解するための一助として提供されるものであり、全ての許容される記述や"崩し"を網羅するものではありません。

### EBNF（主要構造の定義）
```ebnf
file             = app_block , { page_block } ;
app_name         = identifier ;
page_name        = identifier ;
identifier       = ? sequence of alphanumeric characters and underscores, starting with a letter ? ;
string_literal   = '"' , ? any characters except '"' ? , '"' ;
number_literal   = ? sequence of digits, optionally with a decimal point ? ;
boolean_literal  = "true" | "false" ;
app_block        = "#" , app_name , { block } ;
page_block       = "##" , page_name , [ "uses" , "<" , template_name , ">" ] , { block } ;
block            = state_block | templates_block | components_block | layout_block | flow_block | structure_block | comment ;
state_block      = "@State" , { state_definition | comment } ;
state_definition = identifier , ":" , type_name , [ "=" , default_value_expression ] ;
type_name        = identifier | string_literal ; // Examples: string, boolean, 'light' | 'dark'
default_value_expression = ? any sequence of characters representing a value, not starting with a comment marker ? ;
templates_block  = "@Templates" , { template_definition | comment } ;
template_name    = identifier ;
template_definition = "<" , template_name , ">" , { layout_item } ;
components_block = "@Components" , { component_definition | comment } ;
component_name   = identifier ;
component_definition = "<" , component_name , ">" , { layout_item } ;
layout_block     = "@Layout" , [ "for" , placeholder_name ] , { layout_item } ;
placeholder_name = identifier ;
layout_item      = container | element | component_usage | placeholder_usage | comment | "..." ;
container        = "(" , container_type , ")" , [ "#" , id ] , [ ":" , label_value ] , { layout_item } ;
container_type   = "H" | "V" | "Z" | "Div" | "-" ;
id               = identifier ;
label_value      = string_literal | identifier ;
element          = "[" , element_type , "]" , [ "#" , id ] , [ ":" , label_value ] , [ "->" , action_description_line ] ;
element_type     = "Text" | "Input" | "Select" | "Option" | "CheckBox" | "Radio" | "Icon" | "Image" | "Button" | "Logo" | "Link" | "HeadLine" ;
component_usage  = "<" , component_name , ">" ;
placeholder_usage = "{{" , placeholder_name , "}}" ;
flow_block       = "@Flow" , { flow_statement | comment } ;
flow_statement   = event_binding_statement | conditional_flow_statement ;
event_binding_statement = "On" , event_type , element_selector , "->" , action_statement ;
event_type       = identifier ;
element_selector = "[" , element_type , "]" , [ "#" , id ] ;
conditional_flow_statement = "?" , expression , { "|" , case_condition , "=" , action_statement } ;
expression       = ? any text representing a condition or variable ? ;
case_condition   = ? any text representing a case ? ;
action_statement = action_description_line | multi_step_action_description | conditional_flow_statement ;
action_description_line = ? any text describing a single action, may include element interactions like (Div)#Modal ? ;
multi_step_action_description = { "-" , action_description_line } ;
structure_block  = "@Structure" , { structure_item | comment } ;
structure_item   = "-" , ( page_name | section_name ) , { structure_item } ;
section_name     = "(" , identifier , ")" ;
comment          = inline_comment | block_comment ;
inline_comment   = "//" , ? any characters until end of line ? ;
block_comment    = "/*" , ? any characters including newlines, not "*/" ? , "*/" ;
```

### NG例
- `#AppName` ← `# AppName` のようにスペースが必要
- `[Button]Add` ← `[Button]#Add` のようにIDは`#`で区切る
- `(V)[Text]` ← `(V) [Text]` のように適切なスペースが必要
- `@Layout(V)` ← `@Layout\n  (V)` のようにブロック開始後は必ず改行とインデントが必要

---

## 省略・崩し記法のガイドライン
PBSは「ゆるやかな構造フレーム」の原則に基づき、ある程度の記述の省略や"崩し"を許容します。これは、記述の効率性と柔軟性を高めるためですが、構造の曖昧さを招かない範囲で行う必要があります。

- **許容される例**:
  - `...` による繰り返しや詳細部分の省略の明示
  - ワンライナー表記（例: `(V) [Text]: "名前"`）
  - `On Click [Button]#Add -> TodoDetailに遷移` のような、文脈から理解可能な簡略化された処理記述
- **許容されない例**:
  - インデントや必須記号（例: `#`, `[]`, `()`, `@`）の省略による、基本的な構造の破壊
  - 必須ラベル（例: アプリ名、ページ名）や、参照に必要なIDの省略
  - バインディング記号（`:`、`->`）の意図的な省略による意味の曖昧化
- **原則**: 記述の意図が読み手（人またはLLM）に明確に伝わることが最優先です。迷った場合は、より明示的な記述を選択してください。特にLLMによる自動生成や解釈を考慮する場合、省略は慎重に行い、可能な限り構造を明確にする記述が推奨されます。

---

## ラベル・参照・命名規則
要素の識別や再利用を容易にするため、以下の規則を推奨します。

- **ラベルは英数字・アンダースコアのみ。先頭は英字。**
- **ID（#...）は同一スコープ内で一意。**
- **参照は`<ComponentName>`や`#ID`で明示。**
- **命名例**:
  - コンポーネント: `MainLayout`, `TodoList`
  - ID: `add_btn`, `search_field`
  - 変数（State内）: `user_id`, `isCompleted`
- **スコープ**: コンポーネント名はファイル全体でグローバルに参照可能です。IDは、それが定義されたブロック（例: `@Layout`内、`@Components`内の特定のコンポーネント定義内）のスコープで一意である必要があります。
- **重複禁止**: 例えば、同一ページ内の異なる要素に同じID（例: `#Add`）を付与することは避けてください。

---

## エラー・曖昧な記述時の挙動
- **パース不能な場合**: PBSの基本的な構造を著しく損なう記述（例: 対応の取れない括弧、必須のブロック開始記号の欠如など）は、処理系によってエラーとして扱われるべきです。
- **曖昧な場合**: 解釈が複数取りうるような曖昧な記述に対しては、処理系は警告を出すか、最も妥当と推測される解釈を試みつつ、その曖昧さを利用者にフィードバックすることが望ましいです。
- **LLMによる自動生成時**: LLMがPBSを生成する際には、可能な限り曖昧さを排除し、EBNFで示される主要構造に準拠した、明確な記述を心がけるべきです。
- **エラー例（構造的な問題）**:
  - 不正な入れ子構造（例: `@Layout` ブロック内に `## Page` 定義を記述する）
  - 未定義の参照（例: 存在しないコンポーネント `<NonExistentComponent>` を使用する）
  - 構文エラー（例: 条件分岐の `?` に対する `|` が不足している、またはその逆）

---

## アンチパターン・よくある誤り集
以下は、PBSを記述する際に避けるべき、または注意すべき一般的なパターンです。

- **ラベル・IDの重複**: 特にページやコンポーネントを跨いで同じIDを使用してしまい、意図しない挙動を引き起こす。
- **必須ブロックの欠落**: アプリケーションやページを定義する上で、本来必要なブロック（例: ページの `@Layout`）が記述されていない。
- **記号の誤用**: 
  - ID指定の `#` の抜け（例: `[Button]Add` → 正しくは `[Button]#Add`）
  - コロン `:` とアロー `->` の混同（ラベル定義とイベント/アクション指定）
- **インデントによる構造の崩れ**: インデントが不適切で、要素の親子関係や所属ブロックが不明瞭になる。
- **条件式の不正な構文**: 条件分岐の `?` やケース区切りの `|` の記述方法の誤り。
- **未定義コンポーネントの参照**: `@Components` で定義されていないコンポーネント名をレイアウト内で使用する。

---

## 記法の早見表（一部）
| 記号 | 用途 | 書式例 |
|------|------|--------|
| `#` / `##` | アプリ / ページ定義 | `# TaskManager`, `## Login` |
| `@Block` | ブロック開始 | `@Layout`, `@State` |
| `<ComponentName>` | テンプレートやコンポーネントの定義と参照 | `<MainLayout>` |
| `[Type]` | 実態要素（要素一覧参照） | `[Button]`, `[Text]` |
| `...` |  省略。繰り返し表現を省略する | `...` |
| `(H)(V)(Z)(Div)[-]` | 予約レイアウト要素 | `(H)`, `[-]` |
| `{{Placeholder}}` | テンプレートの穴 | `{{Content}}` |
| `On Event -> ` | フローのイベントバインド | `On [Button]#Add ->` |
| `? cond \|case1 = expr or statement \|case N ...` | 条件分岐(ガード文) | `? $Todo.completed \|true = トグルする` |
| `//` / `/* … */` | 行 / ブロックコメント | `// note` |

---

## 要素一覧
以下の要素を使ってUIを構築します：

- `[Text]` - テキスト表示
- `[Input]` - テキスト入力フィールド
- `[Select]` - ドロップダウン選択
- `[Option]` - 選択肢
- `[CheckBox]` - チェックボックス
- `[Radio]` - ラジオボタン
- `[Icon]` - アイコン
- `[Image]` - 画像
- `[Button]` - ボタン

---

## レイアウト・ブロック・フローの出力例

### レイアウト表現例

```pbs
@Layout
  (V): ドロップダウン
    [Text]: "選択肢1"
    [Text]: "選択肢2"
    [Text]: "選択肢3"
```

```pbs
@Layout
  (H): サーチバー
    [Input]: 検索ワード欄
    [Icon]#Search: 虫眼鏡 -> 検索を行う
```

```pbs
@Layout
  (V): テーブル 
    (H): ヘッダー行
      (V) [Text]: "名前" // このようなワンライナー表記は許される
      (V) [Text]: "種類"
    (H): データ行
      (V) [Text]: "iPhone" 
      (V) [Text]: "スマートフォン"
      ...
```

```pbs
@Layout
  (H): パンクズリスト
    [Text]: "Top" -> ホームに遷移
    [Text]: 第二階層のページ名 -> そのページに遷移
    ...
```

### テンプレート & コンポーネント例

```pbs
# TaskManager
  @Templates
    <MainLayout>
      (V)
        <Header>
        {{Content}}
        <Footer>
  @Components
    <Header>
      (H)
        [Text]: "Taskmanager"
        [-]
        [Icon]: Gearのアイコン -> Settingsに遷移
```

### @State ブロック例

```pbs
  @State
    Todo
      id: uuid
      title: string
      completed: boolean = false
      createdAt: datetime = now()
      tags: string[]
    Settings
      theme: 'light' | 'dark' = 'light'
      language: string = 'ja'
      notifications: boolean = true
```

### @Layout ブロック例

```pbs
## Home uses <MainLayout>
  @Layout for Content
    (V)
      (H): Header
        [Text]: "Todo List"
        [-]
        [Button]#Add: "新規追加"
      (V): TodoList
        [Input]#Search: "検索..."
        (V)#Todos: 全てのTODOを繰り返し描画
          <TodoItem>
          ...
      (H): Footer
        [Text]: "合計: {{todos.length}}件"
        [-]
        [Button]#Clear: "完了済みを削除" -> 完了済みのTODOを削除する
```

### @Flow ブロック例

```pbs
  @Flow
    On Click [Button]#Add ->
      - フォームの内容を元に新しいTodoを追加
      ? 追加の結果
        | 成功 = (Div)#AddModalを閉じる
        | 失敗 = エラーを表示
```

---

## 出力例バリエーション
### シンプルな例
```pbs
# MyApp
  @State
    User
      id: uuid
      name: string
  @Layout
    (V)
      [Text]: "Hello, {{User.name}}!"
```
### 複雑な例
```pbs
# ComplexApp
  @Templates
    <MainLayout>
      (V)
        <Header>
        {{Content}}
        <Footer>
  @Components
    <Header>
      (H)
        [Text]: "ComplexApp"
        [-]
        [Icon]: Gear -> Settingsに遷移
  @State
    Item
      id: uuid
      value: string
      tags: string[]
  @Layout
    (V)
      (H): Header
        <Header>
      (V): Items
        (H) [Text]: "Items"
        (V)#ItemList: 全てのItemを繰り返し描画
          [Text]: {{value}}
          ...
```

---

## 参考: フルサンプル

```pbs
# TodoApp
  @Structure
    - Home
    - (Auth): 認証後の領域
      - TodoList
      - TodoDetail

  @State
    User
      id: uuid
      name: string
      email: string
      password: string
    Todo
      id: uuid
      title: string
      completed: boolean
      tags: string[]
      status: 'waiting' | 'done' | 'archived'

  @Templates
    <MainFrame>
      (V)
        (H): Header
          [Logo]: "TodoApp"
          [-]
          ? $User.idが設定されている
            | 設定されている = [Button]: "ログアウト" -> Logoutに遷移
            | 設定されてない = [Button]: "ログイン"  -> Loginに遷移
        (H)
          (V): SideMenu
            [Link]: "ホーム" -> Homeに遷移
            [Link]: "Todo一覧" -> TodoListに遷移
          (V)
            {{Content}}

  @Components
    <TodoRow>
      (V)
        (H)
          [CheckBox]: "完了" -> completedをトグル //遷移以外の操作も記述できる
          [Text]: {{title}}
        (H)
          [Text]: {{tags.join(',')}}
          [-]
          [Button]: "詳細" -> TodoDetailに遷移

## Home
  @Layout
    (V)
      [HeadLine]: "TODOアプリへようこそ"
      ? $User.id
        | true  = [Button]: "Todo追加" -> TodoDetailに遷移
        | false = [Button]: "ログイン" -> Loginに遷移

## TodoList uses <MainFrame>
  @Layout for Content
    (V)
      [HeadLine]: "TODO一覧"
      [Button]#Add: "新規作成"
      (V): ここは全Todo分だけ繰り返し描画する
        <TodoRow>
  @Flow
    On Click [Button]#Add -> TodoDetailに遷移
## TodoDetail uses <MainFrame>
  @Layout for Content
    (V)
      [HeadLine]: ? $Todo.id | true = "TODO編集" | false = "新規 TODO"
      [Input]#Title: タイトル
      [Input]#Tags: バッジデザインのタグをスペース区切りで並べる
      (H)
        [Button]#Save: 保存
        ? $Todo.id
          | true = [Button]#Delete: "削除"
  @Flow
    On Click [Button]#Save ->
      ? 入力チェック
        | 成功 =
          - データを保存する
          - TodoListに移動する
        | 失敗 = エラー文を出す

    On Click [Button]#Delete ->
      - 削除確認ダイアログを出す
      ? 削除確認ダイアログの結果
        | OK =
          - 削除処理をする
          - TodoListに移動する
``` 