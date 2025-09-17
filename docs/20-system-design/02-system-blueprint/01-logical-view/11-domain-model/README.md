---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "database-approach"
    - "concept-entity"
    - "literature-memo-entity"
    - "relationship-entity"
    - "literature-entity"
    - "code-quality-requirements"
  produces:
    - "domain-model"
---

# ドメインモデル

## 前提

### データベース方針

<!-- PREMISE_BEGIN: database-approach -->

CogitoWeave では RDBMS を採用する。リレーショナル構造によるデータ整合性管理、成熟技術による保守性確保、データ規模への十分な対応能力が主な選択理由である。具体的な製品選択は Development View で決定する。

<!-- PREMISE_END: database-approach -->

### 1. 概念エンティティ (Concept)

<!-- PREMISE_BEGIN: concept-entity -->

- 思考の最小単位として知識体系の中心に位置する
- 基本属性: ID、名前、説明、作成日時、更新日時
- 他の概念との関係性によって意味を形成する
- 外部情報を個人の理解に変換する観点として機能する

<!-- PREMISE_END: concept-entity -->

### 2. 文献メモエンティティ (LiteratureMemo)

<!-- PREMISE_BEGIN: literature-memo-entity -->

- ツェッテルカステン方式による概念ベースの知識蓄積
- 基本属性: ID、タイトル、内容、元 URL、作成日時、更新日時
- 特定の概念の観点で外部文献を要約・解釈したもの
- LLM 協働により効率的に作成され、個人の理解で修正される

<!-- PREMISE_END: literature-memo-entity -->

### 3. 関係性エンティティ (Relationship)

<!-- PREMISE_BEGIN: relationship-entity -->

- 概念間の無向グラフ構造による知識ネットワーク形成
- 基本属性: ID、関係名、説明、作成日時
- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援

<!-- PREMISE_END: relationship-entity -->

### 4. 文献エンティティ (Literature)

<!-- PREMISE_BEGIN: literature-entity -->

- 外部文献の書誌情報管理
- 基本属性: ID、URL、タイトル、取得日時
- 複数の概念観点から文献メモが生成される元情報

<!-- PREMISE_END: literature-entity -->

### コード品質要件

<!-- PREMISE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- PREMISE_END: code-quality-requirements -->

DOA アプローチによりデータモデルからビジネスロジックを導出する。個人開発制約により複雑なドメイン設計は避け、薄いドメイン層とする。

## 論理

第一に、CogitoWeave は概念中心の思考支援システムであり、ドメインロジックはデータ構造から自然に導出される。4 つのエンティティ間の関係性とビジネス制約が、システムの本質的な振る舞いを定義する。

第二に、個人開発による制約から、複雑なドメイン設計は保守負荷を増大させる。DDD の重厚な集約設計やドメインサービスの複雑な責務分担よりも、データベース制約に依存したシンプルなモデルが適している。

第三に、外部 API との協調が重要な要素である。LLM 協働やコンテンツ取得は将来的にマイクロサービス化の可能性があり、ドメイン層からは分離すべき処理である。

第四に、RDBMS の参照制約により、エンティティ間の整合性は自動的に保証される。概念削除時のカスケード処理や関係性の整合性管理は、データベース層で実現可能である。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- GLOBAL_CONCLUSION_END: domain-model -->
