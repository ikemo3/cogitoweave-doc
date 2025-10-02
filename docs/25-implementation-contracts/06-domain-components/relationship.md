---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "relationship-entity"
    - "domain-model"
    - "relationship-table-schema"
    - "backend-layer-structure"
  produces:
    - "relationship-entity-specification"
---

# 関係性エンティティ仕様

## 前提

<!-- PREMISE_BEGIN: relationship-entity -->

関係性エンティティ(Relationship):

- 概念間の無向グラフ構造による知識ネットワーク形成
- 基本属性: ID、関係名、説明、作成日時
- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援

<!-- PREMISE_END: relationship-entity -->

<!-- PREMISE_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- PREMISE_END: domain-model -->

<!-- PREMISE_BEGIN: relationship-table-schema -->

```sql
-- ============================================
-- Relationship (関係性) テーブル
-- ============================================
CREATE TABLE relationship (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_a_id INTEGER NOT NULL,
    concept_b_id INTEGER NOT NULL,
    relationship_name VARCHAR(100) NOT NULL,
    description VARCHAR(500) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_a_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (concept_b_id) REFERENCES concept(id) ON DELETE RESTRICT,
    CHECK (concept_a_id < concept_b_id),
    UNIQUE (concept_a_id, concept_b_id, relationship_name)
);

CREATE INDEX idx_relationship_concept_a_id ON relationship(concept_a_id);
CREATE INDEX idx_relationship_concept_b_id ON relationship(concept_b_id);
```

<!-- PREMISE_END: relationship-table-schema -->

<!-- PREMISE_BEGIN: backend-layer-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

#### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

#### 各層の責務

**API 層**:

- FastAPI ルーター定義
- HTTP リクエスト/レスポンス処理
- Pydantic モデルによる入力バリデーション
- HTTP ステータスコードエラーハンドリング

**Application 層**:

- ユースケース実装
- トランザクション制御
- 複数機能にまたがる処理の調整
- Repository 層を使った外部システム協調

**Domain 層**:

- ビジネスエンティティ定義(純粋な Python)
- エンティティ内で閉じるビジネスロジック
- データ構造から自然に導出されるロジック
- ドメインサービスはほぼ不要(DB 制約で整合性管理)

**Repository 層**:

- SQLAlchemy ORM 定義(テーブル構造)
- データベースアクセス
- 外部 API 通信(LLM、記事取得)
- Domain エンティティ ⇔ ORM エンティティ変換
- クエリ実行、トランザクション境界

#### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

<!-- PREMISE_END: backend-layer-structure -->

## 論理

### エンティティ責務

関係性エンティティは概念間の関連性を表現し知識ネットワークを形成する。個人的な関係語彙による柔軟な関係定義を可能にし、空間配置による直感的な関係性発見を支援する。概念間の無向グラフ構造により、断片的知識を構造化された思考地図に変換する。

### ビジネスルール

#### 無向グラフの正規化

関係性は無向グラフとして表現され、概念 A と概念 B の関係は一方向のみ記録される。CHECK 制約により concept_a_id < concept_b_id が保証され、重複する関係性の登録が防止される。

#### 個人的関係語彙

関係名は個人が自由に定義できる。形式的なオントロジーに縛られず、個人の理解に基づいた自然な表現を使用することで、思考の流れを妨げない知識整理が実現される。

#### 同一概念間の複数関係許可

同一の概念ペア間に複数の異なる関係性を定義できる。多面的な関係性の表現を可能にし、知識ネットワークの豊かさを確保する。

### 関係性制約

#### 概念との関係

関係性は必ず 2 つの概念に紐づく(多対多)。概念削除時は紐づく関係性の扱いをアプリケーション層で制御する必要がある(DB 制約は ON DELETE RESTRICT)。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: relationship-entity-specification -->

関係性エンティティ仕様として以下を定義する。

### 関係性エンティティのビジネス上の責務

概念間の関連性を表現し知識ネットワークを形成する。個人的な関係語彙による柔軟な関係定義を可能にし、空間配置による直感的な関係性発見を支援する。概念間の無向グラフ構造により、断片的知識を構造化された思考地図に変換する。

### ドメイン層が実装するビジネスロジック

- 無向グラフの正規化: CHECK 制約により concept_a_id < concept_b_id が保証され、重複する関係性の登録が防止される。
- 個人的関係語彙: 形式的なオントロジーに縛られず、個人の理解に基づいた自然な表現を使用する。
- 同一概念間の複数関係許可: 同一の概念ペア間に複数の異なる関係性を定義でき、多面的な関係性の表現を可能にする。

<!-- GLOBAL_CONCLUSION_END: relationship-entity-specification -->
