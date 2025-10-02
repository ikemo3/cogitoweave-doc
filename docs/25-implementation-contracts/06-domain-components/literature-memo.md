---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "literature-memo-entity"
    - "domain-model"
    - "literature-memo-table-schema"
    - "backend-layer-structure"
  produces:
    - "literature-memo-entity-specification"
---

# 文献メモエンティティ仕様

## 前提

<!-- PREMISE_BEGIN: literature-memo-entity -->

文献メモエンティティ(LiteratureMemo):

- ツェッテルカステン方式による概念ベースの知識蓄積
- 基本属性: ID、タイトル、内容、作成日時、更新日時
- 特定の概念の観点で外部文献を要約・解釈したもの
- LLM 協働により効率的に作成され、個人の理解で修正される

<!-- PREMISE_END: literature-memo-entity -->

<!-- PREMISE_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- PREMISE_END: domain-model -->

<!-- PREMISE_BEGIN: literature-memo-table-schema -->

```sql
-- ============================================
-- LiteratureMemo (文献メモ) テーブル
-- ============================================
CREATE TABLE literature_memo (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_id INTEGER NOT NULL,
    literature_id INTEGER NOT NULL,
    title VARCHAR(500) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_literature_memo_concept_id ON literature_memo(concept_id);
CREATE INDEX idx_literature_memo_literature_id ON literature_memo(literature_id);
```

<!-- PREMISE_END: literature-memo-table-schema -->

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

文献メモエンティティは概念に紐づく具体的知識内容を表現する。特定の概念の観点で外部文献を要約・解釈したものであり、LLM 協働により効率的に作成され個人の理解で修正される。ツェッテルカステン方式による概念ベースの知識蓄積を実現する。

### ビジネスルール

#### 概念との必須関連

文献メモは必ず 1 つの概念に紐づく。概念なしの文献メモは存在しない。これは概念ベースの知識体系構築という中核的価値を保証するためである。

#### 文献との必須関連

文献メモは必ず 1 つの文献に紐づく。文献なしの文献メモは存在しない。これは外部情報を個人の理解に変換するというプロセスの透明性を保証するためである。

#### 内容の個人化

文献メモの内容は個人の理解で修正される。LLM が生成した要約をそのまま保存するのではなく、個人が理解した形に編集することで、真の知識統合が実現される。

### 関係性制約

#### 概念との関係

文献メモは必ず 1 つの概念に紐づく(多対一)。文献メモ削除時は制約がないが、概念削除時は紐づく文献メモの扱いをアプリケーション層で制御する必要がある(DB 制約は ON DELETE RESTRICT)。

#### 文献との関係

文献メモは必ず 1 つの文献に紐づく(多対一)。同一文献を複数の概念の観点で読み直すことで、複数の文献メモが生成される。文献削除時は紐づく文献メモの扱いをアプリケーション層で制御する必要がある(DB 制約は ON DELETE RESTRICT)。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-entity-specification -->

文献メモエンティティ仕様として以下を定義する。

### 文献メモエンティティのビジネス上の責務

概念に紐づく具体的知識内容を表現し、特定の概念の観点で外部文献を要約・解釈する。LLM 協働により効率的に作成され個人の理解で修正されることで、ツェッテルカステン方式による概念ベースの知識蓄積を実現する。

### ドメイン層が実装するビジネスロジック

- 概念との必須関連: 文献メモは必ず 1 つの概念に紐づく。概念なしの文献メモは存在しない。
- 文献との必須関連: 文献メモは必ず 1 つの文献に紐づく。文献なしの文献メモは存在しない。
- 内容の個人化: LLM が生成した要約をそのまま保存するのではなく、個人が理解した形に編集することで真の知識統合が実現される。

<!-- GLOBAL_CONCLUSION_END: literature-memo-entity-specification -->
