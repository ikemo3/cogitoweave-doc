---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "concept-entity"
    - "domain-model"
    - "concept-table-schema"
    - "backend-layer-structure"
  produces:
    - "concept-entity-specification"
---

# 概念エンティティ仕様

## 前提

<!-- PREMISE_BEGIN: concept-entity -->

概念エンティティ(Concept):

- 思考の最小単位として知識体系の中心に位置する
- 基本属性: ID、名前、説明、作成日時、更新日時
- 他の概念との関係性によって意味を形成する
- 外部情報を個人の理解に変換する観点として機能する

<!-- PREMISE_END: concept-entity -->

<!-- PREMISE_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- PREMISE_END: domain-model -->

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

### エンティティ責務

概念エンティティは思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能する。個人の理解体系を構成する基本要素であり、他の概念との関係性によって意味を形成する。

### 属性仕様

#### id (整数型、主キー)

- 自動採番による一意識別子
- システム内部で使用する
- ユーザーから直接参照されない

#### name (文字列型、最大 100 文字、必須、一意制約)

- 概念の名称
- 個人が認識しやすい自然な言葉で表現する
- 一意制約により同名の概念は作成できない
- 検索インデックスにより高速な名前検索を実現する

**バリデーション:**

- 必須: 空文字列は許可しない
- 最大長: 100 文字以内
- 一意性: 既存の概念名と重複不可

#### description (文字列型、最大 1000 文字、任意)

- 概念の説明文
- 個人が概念の意味を思い出すための補助情報
- 検索対象として機能する

**バリデーション:**

- 任意: NULL 許可
- 最大長: 1000 文字以内

#### created_at (タイムスタンプ型、必須、デフォルト値あり)

- 概念の作成日時
- 自動設定される
- 更新されない

#### updated_at (タイムスタンプ型、必須、デフォルト値あり)

- 概念の最終更新日時
- 作成時は created_at と同じ値
- 更新時に自動更新される

### ビジネスルール

#### 名前の一意性

概念の名前は知識体系内で一意である。同名の概念が複数存在すると、個人の理解体系が曖昧になり、文献メモや関係性の紐付けが混乱するためである。

#### 最小限の属性

概念エンティティは ID、名前、説明、タイムスタンプのみを持つ。概念自体はシンプルな抽象化であり、具体的な知識内容は文献メモが保持する責務である。

### 関係性制約

#### 文献メモとの関係

概念は 0 個以上の文献メモを持つ。文献メモは必ず 1 つの概念に紐づく。概念が削除される場合、紐づく文献メモの扱いをアプリケーション層で制御する必要がある(DB 制約は ON DELETE RESTRICT)。

#### 関係性との関係

概念は 0 個以上の他の概念と関係性を持つ。関係性は無向グラフとして表現され、概念 A と概念 B の関係は一方向のみ記録される(正規化制約)。

### 不変条件

概念エンティティが常に満たすべき条件:

- 名前は必ず存在し、空文字列ではない
- 名前は知識体系内で一意である
- 作成日時と更新日時は必ず存在する
- 更新日時は作成日時以降である

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: concept-entity-specification -->

概念エンティティ仕様:

概念エンティティ仕様として以下を定義する。

### 概念エンティティのビジネス上の責務

思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能する。個人の理解体系を構成する基本要素であり、他の概念との関係性によって意味を形成する。

### ドメイン層が実装するビジネスロジック

- 最小限の属性構成: ID、名前、説明、タイムスタンプのみを持つ。概念自体はシンプルな抽象化であり、具体的な知識内容は文献メモが保持する責務である。

<!-- GLOBAL_CONCLUSION_END: concept-entity-specification -->
