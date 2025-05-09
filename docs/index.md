# Page Block Syntax (PBS)

Page Block Syntax (PBS) は、アプリケーション仕様を **ページ単位**で記述し、**レイアウト・フロー・ステート**の3観点で整理する軽量マークアップ言語です。*人→人* と *人→LLM／LLM→人* の情報伝達における曖昧さを最小化し、スパイラル型設計を高速に回すことを目的としています。

## 特徴

1. **可読性** - 非技術者でも理解できるプレーンテキスト
2. **記述容易性** - Markdown ライクな最小限の記号
3. **拡張性** - ページ追加・分割・結合が容易
4. **LLM フレンドリー** - 構文が一意にパースでき、生成 AI が補完しやすい

## 仕様書

- [PBS v0.2 仕様書](spec/v0.2/index.md) - 最新版
- [PBS v0.1 仕様書](spec/v0.1/index.md) - 初版

## サンプル

```pbs
(App) TinyTODO : 個人用 To-Do 管理アプリ

[[MainLayout]]
  - (VStack)
    - Header
    - Content
    - Footer

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
