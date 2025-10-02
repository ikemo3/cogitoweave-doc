---
doc_type: "pattern-composite"
status: "draft"
depends:
  contracts:
    - "backend-layer-structure"
  produces: []
---

# データアクセスコンポーネント仕様

## 概要

C4 モデルの Component レベルにおける Repository 層のコンポーネント仕様を定義する。CogitoWeave システムの 4 つのエンティティ(概念、文献メモ、関係性、文献)について、Repository インターフェース、ORM モデル、変換ロジック、クエリ実装方針を明確化する。

## Repository 一覧

### コア Repository

- [概念 Repository](concept-repository.md): 概念エンティティのデータアクセスを提供する
- [文献メモ Repository](literature-memo-repository.md): 文献メモエンティティのデータアクセスを提供する
- [関係性 Repository](relationship-repository.md): 関係性エンティティのデータアクセスを提供する
- [文献 Repository](literature-repository.md): 文献エンティティのデータアクセスを提供する

### 設計方針

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

## 結論

データアクセスコンポーネント仕様として、4 つのエンティティの Repository 詳細仕様を個別ファイルで定義した。各 Repository は Domain エンティティと ORM エンティティを 1:1 で対応させ、変換のみを担当する。
