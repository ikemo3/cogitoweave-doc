<!-- markdownlint-disable MD024 -->

# Component レベル実装契約集

## 概要

C4 モデルの Component レベルで定義された実装契約を統合する。ドメインコンポーネント、API 仕様、UI コンポーネント、データアクセスコンポーネントの 4 つの Component 群における設計判断を集約し、実装の詳細仕様を提供する。

## ドメインコンポーネント契約

### 概念エンティティ仕様

<!-- REFERENCE_BEGIN: concept-entity-specification -->

概念エンティティ仕様:

概念エンティティ仕様として以下を定義する。

#### 概念エンティティのビジネス上の責務

思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能する。個人の理解体系を構成する基本要素であり、他の概念との関係性によって意味を形成する。

#### ドメイン層が実装するビジネスロジック

- 最小限の属性構成: ID、名前、説明、タイムスタンプのみを持つ。概念自体はシンプルな抽象化であり、具体的な知識内容は文献メモが保持する責務である。

<!-- REFERENCE_END: concept-entity-specification -->

### 文献メモエンティティ仕様

<!-- REFERENCE_BEGIN: literature-memo-entity-specification -->

文献メモエンティティ仕様として以下を定義する。

#### 文献メモエンティティのビジネス上の責務

概念に紐づく具体的知識内容を表現し、特定の概念の観点で外部文献を要約・解釈する。LLM 協働により効率的に作成され個人の理解で修正されることで、ツェッテルカステン方式による概念ベースの知識蓄積を実現する。

#### ドメイン層が実装するビジネスロジック

- 概念との必須関連: 文献メモは必ず 1 つの概念に紐づく。概念なしの文献メモは存在しない。
- 文献との必須関連: 文献メモは必ず 1 つの文献に紐づく。文献なしの文献メモは存在しない。
- 内容の個人化: LLM が生成した要約をそのまま保存するのではなく、個人が理解した形に編集することで真の知識統合が実現される。

<!-- REFERENCE_END: literature-memo-entity-specification -->

### 関係性エンティティ仕様

<!-- REFERENCE_BEGIN: relationship-entity-specification -->

関係性エンティティ仕様として以下を定義する。

#### 関係性エンティティのビジネス上の責務

概念間の関連性を表現し知識ネットワークを形成する。個人的な関係語彙による柔軟な関係定義を可能にし、空間配置による直感的な関係性発見を支援する。概念間の無向グラフ構造により、断片的知識を構造化された思考地図に変換する。

#### ドメイン層が実装するビジネスロジック

- 無向グラフの正規化: CHECK 制約により concept_a_id < concept_b_id が保証され、重複する関係性の登録が防止される。
- 個人的関係語彙: 形式的なオントロジーに縛られず、個人の理解に基づいた自然な表現を使用する。
- 同一概念間の複数関係許可: 同一の概念ペア間に複数の異なる関係性を定義でき、多面的な関係性の表現を可能にする。

<!-- REFERENCE_END: relationship-entity-specification -->

### 文献エンティティ仕様

<!-- REFERENCE_BEGIN: literature-entity-specification -->

文献エンティティ仕様として以下を定義する。

#### 文献エンティティのビジネス上の責務

外部文献の書誌情報を管理し、複数の概念観点から文献メモが生成される元情報として機能する。外部情報を個人の理解に変換するプロセスの起点となる。

#### ドメイン層が実装するビジネスロジック

- 書誌情報のシンプル性: URL、タイトル、取得日時のみを持つ。文献の内容そのものは保持せず、文献メモが個人の理解として保持する。
- 複数観点からの読み直し: 同一文献を複数の概念の観点で読み直すことを許可し、多面的な知識統合を実現する。

<!-- REFERENCE_END: literature-entity-specification -->

## データアクセスコンポーネント契約

### 概念 Repository 仕様

<!-- REFERENCE_BEGIN: concept-repository-specification -->

概念 Repository 仕様として以下を定義する。

#### 概念 Repository の責務

概念エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、名前・説明による検索を実装する。

#### 概念 Repository のインターフェース

- `create(concept: Concept) -> Concept`: 概念を作成する
- `get_by_id(concept_id: int) -> Concept | None`: ID で概念を取得する
- `get_all() -> list[Concept]`: 全概念を取得する
- `update(concept: Concept) -> Concept`: 概念を更新する
- `search_by_name_or_description(query: str) -> list[Concept]`: 名前・説明で部分一致検索する

#### 概念 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- インデックスを活用した効率的な検索を実現する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- REFERENCE_END: concept-repository-specification -->

### 文献メモ Repository 仕様

<!-- REFERENCE_BEGIN: literature-memo-repository-specification -->

文献メモ Repository 仕様として以下を定義する。

#### 文献メモ Repository の責務

文献メモエンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、概念別一覧取得を実装する。

#### 文献メモ Repository のインターフェース

- `create(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを作成する
- `get_by_id(literature_memo_id: int) -> LiteratureMemo | None`: ID で文献メモを取得する
- `get_by_concept_id(concept_id: int) -> list[LiteratureMemo]`: 概念 ID で文献メモ一覧を取得する
- `update(literature_memo: LiteratureMemo) -> LiteratureMemo`: 文献メモを更新する

#### 文献メモ Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 外部キー参照は ID のみを保持し、関連エンティティの取得は必要に応じて別途行う
- インデックスを活用した効率的な概念別取得を実現する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- REFERENCE_END: literature-memo-repository-specification -->

### 関係性 Repository 仕様

<!-- REFERENCE_BEGIN: relationship-repository-specification -->

関係性 Repository 仕様として以下を定義する。

#### 関係性 Repository の責務

関係性エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、概念 ID による関係性取得を実装する。

#### 関係性 Repository のインターフェース

- `create(relationship: Relationship) -> Relationship`: 関係性を作成する
- `get_by_id(relationship_id: int) -> Relationship | None`: ID で関係性を取得する
- `get_by_concept_id(concept_id: int) -> list[Relationship]`: 概念 ID に関連する関係性一覧を取得する
- `update(relationship: Relationship) -> Relationship`: 関係性を更新する

#### 関係性 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 無向グラフの正規化制約(concept_a_id < concept_b_id)を維持する
- 概念 ID による取得では concept_a_id と concept_b_id の両方を検索する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- REFERENCE_END: relationship-repository-specification -->

### 文献 Repository 仕様

<!-- REFERENCE_BEGIN: literature-repository-specification -->

文献 Repository 仕様として以下を定義する。

#### 文献 Repository の責務

文献エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作を実装する。

#### 文献 Repository のインターフェース

- `create(literature: Literature) -> Literature`: 文献を作成する
- `get_by_id(literature_id: int) -> Literature | None`: ID で文献を取得する

#### 文献 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- 文献の作成と読み取りのみを提供する(更新・削除は優先度低)
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- REFERENCE_END: literature-repository-specification -->
