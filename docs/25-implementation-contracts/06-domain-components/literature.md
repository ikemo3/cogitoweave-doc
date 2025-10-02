---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "literature-entity"
    - "domain-model"
    - "literature-table-schema"
    - "backend-layer-structure"
  produces:
    - "literature-entity-specification"
---

# 文献エンティティ仕様

## 前提

<!-- PREMISE_BEGIN: literature-entity -->

文献エンティティ(Literature):

- 外部文献の書誌情報管理
- 基本属性: ID、URL、タイトル、取得日時
- 複数の概念観点から文献メモが生成される元情報

<!-- PREMISE_END: literature-entity -->

<!-- PREMISE_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- PREMISE_END: domain-model -->

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

### エンティティ責務

文献エンティティは外部文献の書誌情報を管理する。複数の概念観点から文献メモが生成される元情報として機能し、外部情報を個人の理解に変換するプロセスの起点となる。

### ビジネスルール

#### 書誌情報のシンプル性

文献エンティティは URL、タイトル、取得日時のみを持つ。文献の内容そのものは保持せず、文献メモが個人の理解として保持する責務である。

#### 複数観点からの読み直し

同一文献を複数の概念の観点で読み直すことを許可する。これにより、一つの文献から複数の文献メモが生成され、多面的な知識統合が実現される。

### 関係性制約

#### 文献メモとの関係

文献は 0 個以上の文献メモを持つ(一対多)。文献削除時は紐づく文献メモの扱いをアプリケーション層で制御する必要がある(DB 制約は ON DELETE RESTRICT)。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: literature-entity-specification -->

文献エンティティ仕様として以下を定義する。

### 文献エンティティのビジネス上の責務

外部文献の書誌情報を管理し、複数の概念観点から文献メモが生成される元情報として機能する。外部情報を個人の理解に変換するプロセスの起点となる。

### ドメイン層が実装するビジネスロジック

- 書誌情報のシンプル性: URL、タイトル、取得日時のみを持つ。文献の内容そのものは保持せず、文献メモが個人の理解として保持する。
- 複数観点からの読み直し: 同一文献を複数の概念の観点で読み直すことを許可し、多面的な知識統合を実現する。

<!-- GLOBAL_CONCLUSION_END: literature-entity-specification -->
