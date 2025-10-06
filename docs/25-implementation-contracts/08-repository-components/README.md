---
doc_type: "pattern-composite"
status: "draft"
depends:
  contracts:
    - "backend-layer-structure"
  produces: []
---

# Repository コンポーネント仕様

## 概要

C4 モデルの Component レベルにおける Repository 層のコンポーネント仕様を定義する。CogitoWeave システムの Repository は、データベースアクセスと外部 API 通信の両方を担当する。

## Repository 一覧

### データベースアクセス Repository

- [概念 Repository](concept-repository.md): 概念エンティティのデータアクセスを提供する
- [文献メモ Repository](literature-memo-repository.md): 文献メモエンティティのデータアクセスを提供する
- [関係性 Repository](relationship-repository.md): 関係性エンティティのデータアクセスを提供する
- [文献 Repository](literature-repository.md): 文献エンティティのデータアクセスを提供する

### 外部システム統合 Repository

- [LLM クライアント Repository](llm-client-repository.md): LLM API 通信を提供する
- [記事取得 Repository](article-fetcher-repository.md): Web 記事取得を提供する

### 設計方針

<!-- PREMISE_BEGIN: layered-architecture-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

#### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

#### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、複数機能にまたがる処理の調整、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

<!-- PREMISE_END: layered-architecture-structure -->

<!-- PREMISE_BEGIN: repository-layer-responsibility -->

#### Repository 層の責務

- SQLAlchemy ORM 定義(テーブル構造)
- データベースアクセス
- 外部 API 通信(LLM、記事取得)
- Domain エンティティ ⇔ ORM エンティティ変換
- クエリ実行、トランザクション境界

<!-- PREMISE_END: repository-layer-responsibility -->

## 結論

Repository コンポーネント仕様として、データベースアクセス Repository と外部システム統合 Repository の詳細仕様を個別ファイルで定義した。各 Repository は Repository 層の責務に従い、データアクセスまたは外部 API 通信を担当する。
