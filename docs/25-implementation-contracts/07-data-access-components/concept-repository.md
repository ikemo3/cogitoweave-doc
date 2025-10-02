---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "concept-entity-specification"
    - "concept-table-schema"
    - "backend-layer-structure"
  produces:
    - "concept-repository-specification"
---

# 概念 Repository 仕様

## 前提

<!-- PREMISE_BEGIN: concept-entity-specification -->

概念エンティティ仕様:

概念エンティティ仕様として以下を定義する。

### 概念エンティティのビジネス上の責務

思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能する。個人の理解体系を構成する基本要素であり、他の概念との関係性によって意味を形成する。

### ドメイン層が実装するビジネスロジック

- 最小限の属性構成: ID、名前、説明、タイムスタンプのみを持つ。概念自体はシンプルな抽象化であり、具体的な知識内容は文献メモが保持する責務である。

<!-- PREMISE_END: concept-entity-specification -->

<!-- PREMISE_BEGIN: concept-table-schema -->

```sql
-- ============================================
-- Concept (概念) テーブル
-- ============================================
CREATE TABLE concept (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(1000) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_concept_name ON concept(name);
CREATE INDEX idx_concept_description ON concept(description);
```

<!-- PREMISE_END: concept-table-schema -->

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

### Repository 責務

概念 Repository は概念エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、名前・説明による検索を実装する。

### ORM モデル定義

SQLAlchemy ORM モデルは concepts テーブル定義と 1:1 で対応する。Domain エンティティと構造を揃え、変換コストを最小化する。

### Repository インターフェース

概念 Repository が提供するメソッド:

- `create(concept: Concept) -> Concept`: 概念を作成する
- `get_by_id(concept_id: int) -> Concept | None`: ID で概念を取得する
- `get_all() -> list[Concept]`: 全概念を取得する
- `update(concept: Concept) -> Concept`: 概念を更新する
- `search_by_name_or_description(query: str) -> list[Concept]`: 名前・説明で部分一致検索する

### 変換ロジック

Domain エンティティ ⇔ ORM エンティティの変換は 1:1 対応で行う。属性名、型、構造を揃えることで、変換ロジックをシンプルに保つ。

### クエリ実装方針

- 名前・説明検索ではインデックス(idx_concepts_name, idx_concepts_description)を活用する
- 全件取得は頻繁に使用されるため、パフォーマンスを考慮する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: concept-repository-specification -->

概念 Repository 仕様として以下を定義する。

### 概念 Repository の責務

概念エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、名前・説明による検索を実装する。

### 概念 Repository のインターフェース

- `create(concept: Concept) -> Concept`: 概念を作成する
- `get_by_id(concept_id: int) -> Concept | None`: ID で概念を取得する
- `get_all() -> list[Concept]`: 全概念を取得する
- `update(concept: Concept) -> Concept`: 概念を更新する
- `search_by_name_or_description(query: str) -> list[Concept]`: 名前・説明で部分一致検索する

### 概念 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- インデックスを活用した効率的な検索を実現する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- GLOBAL_CONCLUSION_END: concept-repository-specification -->
