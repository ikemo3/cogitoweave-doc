---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "concept-entity"
    - "literature-memo-entity"
    - "relationship-entity"
    - "literature-entity"
    - "technology-selection-criteria"
    - "code-quality-requirements"
  produces:
    - "database-approach"
---

# データベース方針

## 前提

<!-- PREMISE_BEGIN: concept-entity -->

概念エンティティ(Concept):

- 思考の最小単位として知識体系の中心に位置する
- 基本属性: ID、名前、説明、作成日時、更新日時
- 他の概念との関係性によって意味を形成する
- 外部情報を個人の理解に変換する観点として機能する

<!-- PREMISE_END: concept-entity -->

<!-- PREMISE_BEGIN: literature-memo-entity -->

文献メモエンティティ(LiteratureMemo):

- ツェッテルカステン方式による概念ベースの知識蓄積
- 基本属性: ID、タイトル、内容、作成日時、更新日時
- 特定の概念の観点で外部文献を要約・解釈したもの
- LLM 協働により効率的に作成され、個人の理解で修正される

<!-- PREMISE_END: literature-memo-entity -->

<!-- PREMISE_BEGIN: relationship-entity -->

関係性エンティティ(Relationship):

- 概念間の無向グラフ構造による知識ネットワーク形成
- 基本属性: ID、関係名、説明、作成日時
- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援

<!-- PREMISE_END: relationship-entity -->

<!-- PREMISE_BEGIN: literature-entity -->

文献エンティティ(Literature):

- 外部文献の書誌情報管理
- 基本属性: ID、URL、タイトル、取得日時
- 複数の概念観点から文献メモが生成される元情報

<!-- PREMISE_END: literature-entity -->

### 技術選択基準

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

### コード品質要件

<!-- PREMISE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- PREMISE_END: code-quality-requirements -->

データ規模要件として、概念数は数千レベル、文献メモは年間数千件の作成が想定される。同時アクセスは個人ツールのため考慮不要で、複雑な検索やリアルタイム性要件もない。

本文書では、データベース技術の方針レベルの判断を行う。具体的な製品選択(PostgreSQL/MySQL/SQLite 等)は後続の Development View で決定する。

## 論理

第一に、CogitoWeave のデータ構造は本質的にリレーショナルモデルに適合する。4 つのエンティティ間の関係性は、1:N や N:M といった明確なリレーショナル関係で表現される。概念と文献メモの関係、概念と関係性の関係、文献と文献メモの関係は、すべて外部キーによる参照整合性が重要である。

第二に、データ整合性の要求が高い。概念の削除時には関連する文献メモや関係性も適切に処理される必要があり、孤立レコードの発生を防ぐ必要がある。関係性エンティティが参照する概念が削除された場合の整合性管理も重要な要件となる。

第三に、データ規模と性能要件を考慮すると、概念数千レベル、文献メモ年間数千件という規模は、現代的な RDBMS であれば十分に対応可能な範囲である。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: database-approach -->

CogitoWeave では RDBMS を採用する。リレーショナル構造によるデータ整合性管理、成熟技術による保守性確保、データ規模への十分な対応能力が主な選択理由である。具体的な製品選択は Development View で決定する。

<!-- GLOBAL_CONCLUSION_END: database-approach -->
