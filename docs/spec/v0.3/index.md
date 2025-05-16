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

PBSは、1ファイルで1アプリケーションを記述する。

```
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

## 記法の早見表（一部）

| 記号 | 用途 | 書式例 |
|------|------|--------|
| `#` / `##` | アプリ / ページ定義 | `# TaskManager`, `## Login` |
| `@Block` | ブロック開始 | `@Layout`, `@State` |
| `<Name>` | テンプレートやコンポーネントの定義と参照 | `<MainLayout>` |
| `[Type]` | 実態要素（要素一覧参照） | `[Button]`, `[Text]` |
| `...` |  省略。繰り返し表現を省略する | `...` |
| `(H)(V)(Z)(Div)[-]` | 予約レイアウト要素 | `(H)`, `[-]` |
| `{{Placeholder}}` | テンプレートの穴 | `{{Content}}` |
| `On Event -> ` | フローのイベントバインド | `On [Button]#Add ->` |
| `? cond \|case1 = expr or statement \|case N ...` | 条件分岐(ガード文) | `? $Todo.completed \|true = トグルする` |
| `//` / `/* … */` | 行 / ブロックコメント | `// note` |

---

## 要素一覧

 ```
 [Text] / [Input] / [Select] / [Option] / [CheckBox] / [Radio]  / [Icon] / [Image] / [Button]
 ```

## レイアウト表現例

```
@Layout
  (V): ドロップダウン
    [Text]: "選択肢1"
    [Text]: "選択肢2"
    [Text]: "選択肢3"
```

```
@Layout
  (H): サーチバー
    [Input]: 検索ワード欄
    [Icon]#Search: 虫眼鏡 -> 検索を行う
```

```
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

```
@Layout
  (H): パンクズリスト
    [Text]: "Top" -> ホームに遷移
    [Text]: 第二階層のページ名 -> そのページに遷移
    ...
```



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
        [Text]: "Taskmanager"
        [-]
        [Icon]: Gearのアイコン -> Settingsに遷移
```

---

## @State ブロック例

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

## @Layout ブロック例

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

---

## @Flow ブロック例

```pbs
  @Flow
    On Click [Button]#Add ->
      - フォームの内容を元に新しいTodoを追加
      ? 追加の結果
        | 成功 = (Div)#AddModalを閉じる
        | 失敗 = エラーを表示
```

---

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


## EBNF (暫定完成版)

```ebnf

```