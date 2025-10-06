---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "concept-repository-specification"
    - "backend-layer-structure"
  produces:
    - "llm-summary-service-specification"
---

# LLM 要約生成サービス仕様

## 前提

<!-- PREMISE_BEGIN: concept-repository-specification -->

概念 Repository 仕様として以下を定義する。

### 概念 Repository の責務

概念エンティティのデータアクセスを提供する。SQLAlchemy ORM モデルと Domain エンティティの変換、CRUD 操作、名前・説明による検索を実装する。

### 概念 Repository のインターフェース

```mermaid
classDiagram
    class ConceptRepository {
        +create(concept: Concept) Concept
        +get_by_id(concept_id: integer) Concept|None
        +get_all() List~Concept~
        +update(concept: Concept) Concept
        +search_by_name_or_description(query: string) List~Concept~
    }
```

### 概念 Repository の実装方針

- ORM モデルと Domain モデルは 1:1 対応で構造を揃える
- インデックスを活用した効率的な検索を実現する
- トランザクション境界は Application 層で制御し、Repository は単一操作のみを提供する

<!-- PREMISE_END: concept-repository-specification -->

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

<!-- PREMISE_BEGIN: application-layer-responsibility -->

#### Application 層の責務

- ユースケース実装
- トランザクション制御
- 複数機能にまたがる処理の調整
- Repository 層を使った外部システム協調

<!-- PREMISE_END: application-layer-responsibility -->

## 論理

第一に、LLM 要約生成サービスは、特定の概念の観点で記事内容を要約する Application 層のコンポーネントである。概念 Repository から概念情報を取得し、LLM に記事内容と概念情報を渡して要約を生成する。

第二に、概念 Repository の get_by_id メソッドを使用して、指定された概念を取得する。概念の名前と説明を LLM のプロンプトに含めることで、概念の観点に沿った要約を生成する。

第三に、LLM Repository を使用して記事内容の要約を生成する。記事タイトル、記事内容、概念情報をプロンプトに含め、文献メモのタイトルと内容を生成する。

第四に、LLM API の一時的な障害に対するエラーハンドリングを担当する。リトライ処理やタイムアウト処理は Application 層で実装する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: llm-summary-service-specification -->

LLM 要約生成サービス仕様として以下を定義する。

### LLM 要約生成サービスの責務

- 概念の観点で記事を要約するユースケース実装を行う。
- 概念 Repository を使用して概念情報を取得する。
- LLM Repository を使用して要約を生成する。
- エラーハンドリングと例外処理を行う。

### LLM 要約生成サービスのインターフェース

- `generate_summary(concept_id: int, article_title: str, article_content: str) -> Summary`: 概念の観点で記事を要約する

### LLM 要約生成サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 概念 Repository と LLM Repository への依存を持つ。
- LLM API の障害に対するエラーハンドリングを実装する。

<!-- GLOBAL_CONCLUSION_END: llm-summary-service-specification -->
