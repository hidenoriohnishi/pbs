# Page Block Syntax (PBS)

![バージョン](https://img.shields.io/badge/version-0.2-blue)

Page Block Syntax（PBS）は、アプリケーション仕様を**ページ単位**で記述し、**レイアウト・フロー・ステート**の3観点から整理する軽量マークアップ言語です。

## 概要

PBSは以下の目的で設計されています：

- **人間とAI間のコミュニケーションギャップの解消**
- **設計段階での曖昧さの最小化**
- **スパイラル型開発プロセスの高速化**

## 主な特徴

1. **可読性に優れた構文** - 非技術者でも理解できるプレーンテキスト
2. **記述が容易** - Markdownライクな最小限の記号で完結
3. **拡張性が高い** - ページの追加・分割・結合が容易
4. **LLMフレンドリー** - 構文が一意にパースでき、生成AIが補完しやすい

## 基本構文

PBS文書は以下の要素で構成されます：

- アプリケーション定義 `(App)`
- ページ定義 `[Page]`
- レイアウトブロック `(Layout)`
- フローブロック `(Flow)`
- ステートブロック `(State)`

## 簡単な例

```pbs
(App) TinyTODO : 個人用To-Do管理アプリ

[Home]
  (Layout) Content
    - (HStack)
      - (Input) NewTodoText
      - (Button) Add
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

## 仕様書

詳細な仕様については以下をご参照ください：

- [PBS v0.2 仕様書](docs/spec/v0.2/index.md) - 最新版
- [PBS v0.1 仕様書](docs/spec/v0.1/index.md) - 初版

## 利用方法

1. テキストエディタで`.pbs`拡張子のファイルを作成
2. 仕様に従ってアプリケーション構造を記述
3. PBSパーサでパースして検証（開発中）

## ライセンス

MIT

## 貢献

バグレポートや機能提案は[Issues](https://github.com/yourusername/pbs/issues)にお願いします。 