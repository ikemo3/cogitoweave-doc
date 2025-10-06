---
doc_type: "unknown"
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

### エンティティ構造

```mermaid
classDiagram
    class Literature {
        +id: ID
        +kind: String(10)
    }

    class WebArticle {
        +literature_id: ForeignKey
        +url: String(2048)
        +title: String(500)
    }

    class Book {
        +literature_id: ForeignKey
        +isbn: String(20)
        +author: String(200)
        +publisher: String(200)
        +publication_year: Integer
    }

    Literature <|-- WebArticle : inherits
    Literature <|-- Book : inherits
    Literature "1" -- "0..*" LiteratureMemo : has
```

### 属性制約

**Literature (基底)**:

- **kind**: 'web' または 'book'、必須
- **id**: 自動設定

**WebArticle (派生)**:

- **url**: 空文字列禁止、最大 2048 文字
- **title**: 空文字列禁止、最大 500 文字
- **literature_id**: PRIMARY KEY、外部キー(Literature)

**Book (派生)**:

- **isbn**: 空文字列禁止、最大 20 文字、一意
- **author**: 空文字列禁止、最大 200 文字
- **publisher**: 空文字列禁止、最大 200 文字
- **publication_year**: 必須
- **literature_id**: PRIMARY KEY、外部キー(Literature)

### 不変条件

- kind は 'web' または 'book'
- kind = 'web' なら web_article レコードが存在
- kind = 'book' なら book レコードが存在
- WebArticle: url と title は空文字列でない
- Book: isbn, author, publisher は空文字列でない
- Book: isbn は一意

### ビジネスルール

#### 継承テーブル構造

CTI(Class Table Inheritance)により、基底テーブル(Literature)と派生テーブル(WebArticle/Book)に分離。型安全性と拡張性を確保。

#### 書誌情報のシンプル性

文献の内容そのものは保持せず、文献メモが個人の理解として保持する責務である。

#### 複数観点からの読み直し

同一文献を複数の概念の観点で読み直すことを許可する。一つの文献から複数の文献メモが生成され、多面的な知識統合が実現される。

### 関係性制約

- **文献メモとの関係**: 1 対多、文献削除時は ON DELETE RESTRICT でアプリケーション層が制御

<!-- PREMISE_END: literature-entity-specification -->

<!-- PREMISE_BEGIN: literature-table-schema -->

```sql
-- ============================================
-- Literature (文献) テーブル - 基底
-- ============================================
CREATE TABLE literature (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    kind VARCHAR(10) NOT NULL CHECK (kind IN ('web', 'book'))
);
```

<!-- PREMISE_END: literature-table-schema -->

<!-- PREMISE_BEGIN: layered-architecture-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、複数機能にまたがる処理の調整、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

<!-- PREMISE_END: layered-architecture-structure -->

<!-- PREMISE_BEGIN: repository-layer-responsibility -->

### Repository 層の責務

- SQLAlchemy ORM 定義(テーブル構造)
- データベースアクセス
- 外部 API 通信(LLM、記事取得)
- Domain エンティティ ⇔ ORM エンティティ変換
- クエリ実行、トランザクション境界

<!-- PREMISE_END: repository-layer-responsibility -->

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

```mermaid
classDiagram
    class LiteratureRepository {
        +create(literature: Literature) Literature
        +get_by_id(literature_id: integer) Literature|None
    }
```

### 文献 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 文献の作成と読み取りのみを提供する(更新・削除は優先度低)
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- GLOBAL_CONCLUSION_END: literature-repository-specification -->
