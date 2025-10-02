---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "literature-memo-entity-specification"
    - "literature-memo-table-schema"
    - "backend-layer-structure"
  produces:
    - "literature-memo-repository-specification"
---

# 文献メモ Repository 仕様

## 前提

<!-- PREMISE_BEGIN: literature-memo-entity-specification -->

文献メモエンティティ仕様として以下を定義する。

### 文献メモエンティティのビジネス上の責務

概念に紐づく具体的知識内容を表現し、特定の概念の観点で外部文献を要約・解釈する。LLM 協働により効率的に作成され個人の理解で修正されることで、ツェッテルカステン方式による概念ベースの知識蓄積を実現する。

### ドメイン層が実装するビジネスロジック

- 概念との必須関連: 文献メモは必ず 1 つの概念に紐づく。概念なしの文献メモは存在しない。
- 文献との必須関連: 文献メモは必ず 1 つの文献に紐づく。文献なしの文献メモは存在しない。
- 内容の個人化: LLM が生成した要約をそのまま保存するのではなく、個人が理解した形に編集することで真の知識統合が実現される。

<!-- PREMISE_END: literature-memo-entity-specification -->

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

### Repository 責務

文献メモ Repository は文献メモエンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、概念別一覧取得を実装する。

### ORM モデル定義

SQLAlchemy ORM モデルは literature_memos テーブル定義と 1:1 で対応する。Domain エンティティと構造を揃え、変換コストを最小化する。外部キー(concept_id, literature_id)による関連を定義する。

### Repository インターフェース

文献メモ Repository が提供するメソッド:

- `create(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを作成する
- `get_by_id(literature_memo_id: int) -> LiteratureMemo | None`: ID で文献メモを取得する
- `get_by_concept_id(concept_id: int) -> list[LiteratureMemo]`: 概念 ID で文献メモ一覧を取得する
- `update(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを更新する

### 変換ロジック

Domain エンティティ ⇔ ORM エンティティの変換は 1:1 対応で行う。外部キー参照は ID のみを保持し、関連エンティティの取得は必要に応じて別途行う。

### クエリ実装方針

- 概念別一覧取得ではインデックス(idx_literature_memos_concept_id)を活用する
- N+1 問題を避けるため、必要に応じて eager loading を使用する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-repository-specification -->

文献メモ Repository 仕様として以下を定義する。

### 文献メモ Repository の責務

文献メモエンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、概念別一覧取得を実装する。

### 文献メモ Repository のインターフェース

- `create(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを作成する
- `get_by_id(literature_memo_id: int) -> LiteratureMemo | None`: ID で文献メモを取得する
- `get_by_concept_id(concept_id: int) -> list[LiteratureMemo]`: 概念 ID で文献メモ一覧を取得する
- `update(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを更新する

### 文献メモ Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 外部キー参照は ID のみを保持し、関連エンティティの取得は必要に応じて別途行う
- インデックスを活用した効率的な概念別取得を実現する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- GLOBAL_CONCLUSION_END: literature-memo-repository-specification -->
