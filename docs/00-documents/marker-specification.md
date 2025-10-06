---
doc_type: "unknown"
status: "draft"
depends:
  contracts: []
  produces: []
---

# マーカー仕様

ドキュメント間での情報継承や統合作業を保証するため、以下の 3 種類のマーカーを使用します。

## GLOBAL_CONCLUSION マーカー(結論)

結論として結論集に継承する内容を示します。

### GLOBAL_CONCLUSION マーカー用途

- **結論の継承**: 個別・統合・段階型すべてのタイプで結論を結論集に継承
- **新規定義**: このファイルで新たに定義・生産する契約内容
- **基底統合**: Why 系 →11-why-conclusion.md、What 系 →21-01〜05-\*-view.md(各ビュー別)

### GLOBAL_CONCLUSION マーカー記法

```markdown
<!-- GLOBAL_CONCLUSION_BEGIN: example -->

(結論として継承する内容)

<!-- GLOBAL_CONCLUSION_END: example -->
```

## PREMISE マーカー(前提)

前提として利用する契約内容を示します。

### PREMISE マーカー用途

- **前提の取り込み**: 他の文書で定義された契約を前提として利用
- **論理的基盤**: 推論の出発点となる前提条件
- **依存関係**: 結論集からの前提継承や他文書への依存

### PREMISE マーカー記法

```markdown
<!-- PREMISE_BEGIN: example -->

(前提として利用する内容)

<!-- PREMISE_END: example -->
```

## LOCAL_CONCLUSION マーカー(結論・詳細版)

結論として定義するが、結論集には継承しない詳細な内容を示します。

### LOCAL_CONCLUSION マーカー用途

- **完全性チェック**: バリデーションで結論の完全性を保証
- **詳細実装**: 実装レベルの詳細な結論を定義
- **基底継承なし**: 結論集には継承せず、整合性チェックのみ実行

### LOCAL_CONCLUSION マーカー記法

```markdown
<!-- LOCAL_CONCLUSION_BEGIN: example -->

(結論として定義するが結論集継承しない内容)

<!-- LOCAL_CONCLUSION_END: example -->
```

## REFERENCE マーカー(参照)

説明や例示のための引用・参照を示します。

### REFERENCE マーカー用途

- **引用・参照**: 説明や例示のための契約内容の引用
- **統合・まとめ**: 複数契約をまとめる際の参照
- **新規生産しない**: 内容の一致チェックは行うが、結論集には継承しない

### REFERENCE マーカー記法

```markdown
<!-- REFERENCE_BEGIN: example -->

(参照・引用する内容)

<!-- REFERENCE_END: example -->
```

## INLINE マーカー(行内マーカー)

行内で単一の項目や要素を参照・定義するためのマーカーです。

### INLINE マーカー用途

- **行内参照**: 文章中で特定の要素を参照する際の短縮表記
- **項目識別**: 箇条書きや要素の個別識別子として利用
- **実装参照**: コードや実装で参照する際の識別子として利用

### INLINE マーカー記法

```markdown
<!-- PREMISE_INLINE: example -->
<!-- GLOBAL_CONCLUSION_INLINE: example -->
<!-- LOCAL_CONCLUSION_INLINE: example -->
<!-- REFERENCE_INLINE: example -->
```

### INLINE マーカー使用例

```markdown
- URL 入力フィールド - 外部記事の URL 入力 <!-- REFERENCE_INLINE: example -->
- 概念作成インターフェース - 新規概念作成機能 <!-- REFERENCE_INLINE: example -->
```

## マーカー命名規則

マーカー名は由来を明確にするため、以下の命名規則に従う。

### 基本構造

```text
{軸}/{ビュー}:{内容ID}
```

**例外(docs 軸のみ)**:

```text
{軸}:{内容ID}
```

### 軸とビューの定義

**why 系(Why 系: なぜ作るのか)**:

```text
why/core:{内容ID}     - 01-core-vision(コアビジョン)
why/org:{内容ID}      - 02-organizational-capacity(組織能力)
why/ext:{内容ID}      - 03-external-environment(外部環境)
why/sys:{内容ID}      - 04-system-properties(システム制約)
```

**what 系(What 系: 何を作るのか - Kruchten 4+1 ビュー)**:

```text
what/scenario:{内容ID}         - 01-user-desires(Scenario View)
what/logical:{内容ID}          - 02-system-blueprint/01-logical-view(Logical View)
what/process:{内容ID}          - 02-system-blueprint/02-process-view(Process View)
what/development:{内容ID}      - 02-system-blueprint/03-development-view(Development View)
what/physical:{内容ID}         - 02-system-blueprint/04-physical-view(Physical View)
what/sys-constraints:{内容ID}  - 03-system-constraints(システム制約)
what/user-constraints:{内容ID} - 04-user-constraints(ユーザー制約)
```

**docs 系(メタ情報)**:

```text
docs:{内容ID}         - 00-documents(設計書体系のメタ情報)
```

**contract 系(TBD)**:

```text
contract/{ビュー}:{内容ID}    - 25-implementation-contracts(実装契約)
```

**how 系(TBD)**:

```text
how/{ビュー}:{内容ID}         - 30-delivery-methods(実装・運用方法)
```

### 命名例

```text
why/core:mission-statement
what/development:sqlite-selection
what/scenario:usage-context
docs:writing-rules
```

## 共通ルール

- マーカー名は上記の命名規則に従う
- マーカーで囲んだ本文は、見出しレベル(行頭の `#`)を除き**完全一致**が必要
- 一致の判定は空白や記号も含む
- 同一マーカー名を持つ内容は、すべてのファイルで**一字一句同じ**でなければならない

## マーカーの取り扱い禁則

### GLOBAL_CONCLUSION マーカーの移動・削除は絶対禁止

```markdown
<!-- GLOBAL_CONCLUSION_BEGIN: example-id -->

重要な情報内容

<!-- GLOBAL_CONCLUSION_END: example-id -->
```

**重要**: GLOBAL_CONCLUSION マーカーで囲まれた内容の移動・削除は絶対に禁止。理由:

1. **情報継承の生命線**: 結論を結論集にそのまま継承するための技術的仕組み
2. **完全複製原則**: 内容は一字一句「そのまま」持っていく。編集・要約・改変は一切禁止
3. **整合性保証**: 同一マーカー ID を持つ全ファイルで内容が完全一致していることが前提

### PREMISE・REFERENCE マーカーも同様の制約

**前提と参照の整合性**: PREMISE・REFERENCE マーカーも同一マーカー ID で内容完全一致が必要。理由:

1. **論理的一貫性**: 前提が変われば結論も変わる可能性があるため
2. **参照の正確性**: 引用・参照内容の正確性を技術的に保証するため

### 許可される操作

**見出しレベル調整のみ許可**: 親ディレクトリに継承する際、階層に応じた見出しレベル調整は可能

```markdown
<!-- 子ファイル -->

### 予算制約の受容

<!-- 親ファイル -->

##### 予算制約の受容
```

### 禁止される操作

- マーカー内容の移動
- マーカー内容の削除
- マーカー内容の編集・要約・改変
- マーカー ID の変更
- マーカーの部分的な抜粋

違反すると設計書体系の情報継承チェーンが破綻し、結論集での情報欠落・矛盾が発生する。
