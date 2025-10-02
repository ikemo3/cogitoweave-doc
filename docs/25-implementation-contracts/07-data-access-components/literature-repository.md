---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "literature-entity-specification"
    - "literature-table-schema"
    - "backend-layer-structure"
  produces:
    - "literature-repository-specification"
---

# 文献 Repository 仕様

## 前提

<!-- PREMISE_BEGIN: literature-entity-specification -->

文献エンティティ仕様として以下を定義する。

### 文献エンティティのビジネス上の責務

外部文献の書誌情報を管理し、複数の概念観点から文献メモが生成される元情報として機能する。外部情報を個人の理解に変換するプロセスの起点となる。

### ドメイン層が実装するビジネスロジック

- 書誌情報のシンプル性: URL、タイトル、取得日時のみを持つ。文献の内容そのものは保持せず、文献メモが個人の理解として保持する。
- 複数観点からの読み直し: 同一文献を複数の概念の観点で読み直すことを許可し、多面的な知識統合を実現する。

<!-- PREMISE_END: literature-entity-specification -->

<!-- PREMISE_BEGIN: literature-table-schema -->

```sql
-- ============================================
-- Literature (文献) テーブル
-- ============================================
CREATE TABLE literature (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    url VARCHAR(2048) NOT NULL,
    title VARCHAR(500) NOT NULL,
    fetched_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_literature_url ON literature(url);
```

<!-- PREMISE_END: literature-table-schema -->

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

文献 Repository は文献エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作を実装する。

### ORM モデル定義

SQLAlchemy ORM モデルは literatures テーブル定義と 1:1 で対応する。Domain エンティティと構造を揃え、変換コストを最小化する。

### Repository インターフェース

文献 Repository が提供するメソッド:

- `create(literature: Literature) -> Literature`: 文献を作成する
- `get_by_id(literature_id: int) -> Literature | None`: ID で文献を取得する

### 変換ロジック

Domain エンティティ ⇔ ORM エンティティの変換は 1:1 対応で行う。属性名、型、構造を揃えることで、変換ロジックをシンプルに保つ。

### クエリ実装方針

- 文献の作成と読み取りのみを提供する(更新・削除は優先度低)
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: literature-repository-specification -->

文献 Repository 仕様として以下を定義する。

### 文献 Repository の責務

文献エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作を実装する。

### 文献 Repository のインターフェース

- `create(literature: Literature) -> Literature`: 文献を作成する
- `get_by_id(literature_id: int) -> Literature | None`: ID で文献を取得する

### 文献 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 文献の作成と読み取りのみを提供する(更新・削除は優先度低)
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- GLOBAL_CONCLUSION_END: literature-repository-specification -->
