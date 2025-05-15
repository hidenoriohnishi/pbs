# Page Block Syntax (PBS) Specification **v0.3**
*Last updated: 2025‑05‑14*

## 1. 目的
PBS はアプリケーションを **ページ単位** で設計し、**Layout / Flow / State** の 3 観点をMarkdown風の軽量記法で整理することで、*人<->人* と *人<->LLM* の情報伝達コストを最小化して、プロトタイピングや設計での試行錯誤（スパイラルな設計）を正確かつ高速にすることを狙うもの。

---

## 2. 設計原則
1. **可読性** — エンジニアが俯瞰しやすい平易なテキスト  
2. **記述容易性** — 記号の種類を最小限に、また一貫性を持たせる 
3. **拡張性** — ページの追加・分割・再統合が可能かつ簡単  
4. **LLM フレンドリー** — LLMが解釈＆生成しやすい構文であること  
5. **視覚的明瞭さ** — 速読に適した視覚的なフォーマット

---

## ドキュメント構造

```
<file*>
└─ # App                         0..1
   ├─ @Structure                 0..1
   ├─ @State                  (global) 0..*
   ├─ @Templates                       0..1
   ├─ @Components                      0..*
   ├─ @Flow              (global flow) 0..*
   └─ ## Page                          1..*
       ├─ @State                       0..*
       ├─ @Layout                      1..*
       ├─ @Components                  0..*
       └─ @Flow                        0..*
```

* グローバルブロック（State / Templates / Components / Flow）は **ファイル内の出現位置自由**
* ページブロックは `@Layout → @Components → @Flow → @State` の順を推奨（必須ではない）。  
* 複数ファイルに分割する時、@Structureはインデックスとして作用する。
  * @Structureを含めたページを`_app_.pbs.txt`として配置し、インデックスの代わりに使う。
  * 各ページは一つ以上のページを含み慣例として一つのページを1ファイルに対応させる。例：`Home.pbs.txt`


```pbs
# App: My Todo
@Structure
  - Home
  - (Auth): 認証後の領域
    - TodoList
    - TodoDetail
// 以下省略
```

```pbs
// Home.pbs.txt
# App: My Todo
## Home uses <LPLayout>
  @Layout for Content
    (V)
      (H): Navbar
      ...
```
---

## 記法の早見表（一部）

| 記号 | 用途 | 書式例 |
|------|------|--------|
| `#` / `##` | アプリ / ページ定義 | `# TaskManager`, `## Login` |
| `@Block` | ブロック開始 | `@Layout`, `@State` |
| `<Name>` | テンプレートやコンポーネントの定義と参照 | `<MainLayout>` |
| `[Type]` | 実態要素 | `[Button]`, `[Text]` |
| `(H)(V)[-]` | 予約レイアウト要素 | `(H)`, `[-]` |
| `(Custom)` | 任意コンテナ／カテゴリ要素 | `(Header)`, `(Sidebar)` |
| `{{Placeholder}}` | テンプレートの穴 | `{{Content}}` |
| `On Event -> ` | フローのイベントバインド | `On Click([Button Add]) ->` |
| `? cond \|case -> expr.` | 条件分岐(ガード文) | `? Todo.completed \|true -> "トグルする"` |
| `//` / `/* … */` | 行 / ブロックコメント | `// note` |

---

## テンプレート & コンポーネント

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
        [Text]: Taskmanager
        [-]
        [Icon]: "Settings Gearのアイコン" -> Settings 
```

---

## @Layout ブロック例

```pbs
## Home uses <MainLayout>
  @Layout for Content
```

---

## @Flow ブロック例

```pbs
  @Flow
```

---

## @State ブロック例

```pbs
  @State
```

## フルサンプル

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
          // ログイン状態でラベルと挙動をトグル
          [Button]:
            ? User.id
              | true  -> "ログアウト" -> Logout
              | false -> "ログイン"  -> Login
        (H)
          (V): SideMenu
            [Link]: ホーム -> Home
            [Link]: Todo一覧 -> TodoList
          {{Content}}

  @Components
    <TodoRow>
      (H)
        [CheckBox]: 完了 -> "completed をトグル"
        [Text]: {{title}}
        [Text]: {{tags.join(',')}}
        [Button]: 詳細 -> TodoDetail

## Home
  @Layout
    (V)
      [HeadLine]: "TODOアプリへようこそ"
      // ログイン状態によって遷移先が変わる
      [Button]:
        ? User.id
          | true  -> "Todo追加" -> TodoDetail
          | false -> "ログイン" -> Login

## TodoList uses <MainFrame>
  @Layout for Content
    (V)
      [HeadLine]: "TODO一覧"
      [Button]: 新規作成 -> TodoDetail
      (V) // ここは全Todo分だけ繰り返し描画する
        <TodoRow>
  @Flow
    On Click [Button]: 新規作成 -> TodoDetail

## TodoDetail uses <MainFrame>
  @Layout for Content
    (V)
      [HeadLine]:
        ? Todo.id
          | true  -> "TODO編集"
          | false -> "新規 TODO"
      [Input]#Title: タイトル
      [Input]#Tags: タグ（カンマ区切り）
      (H)
        [Button]#Save: 保存
        ? Todo.id
          | true -> [Button]#Delete: 削除
  @Flow
    On Click [Button]#Save ->
      - 入力チェックして保存、TodoListに遷移
    On Click [Button]#Delete ->
      - 削除確認ダイアログを出し、OKなら削除しTodoListに遷移
```


## EBNF (暫定完成版)

```ebnf

```