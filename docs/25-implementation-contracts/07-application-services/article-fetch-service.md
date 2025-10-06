---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-layer-structure"
  produces:
    - "article-fetch-service-specification"
---

# 記事取得サービス仕様

## 前提

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

<!-- PREMISE_BEGIN: application-layer-responsibility -->

### Application 層の責務

- ユースケース実装
- トランザクション制御
- 複数機能にまたがる処理の調整
- Repository 層を使った外部システム協調

<!-- PREMISE_END: application-layer-responsibility -->

## 論理

第一に、記事取得サービスは、指定された URL から記事内容を取得する Application 層のコンポーネントである。記事取得 Repository を使用して外部 API 通信を行い、記事のタイトルと本文を取得する。

第二に、URL のバリデーションと正規化は Application 層で行う。不正な URL や到達不可能な URL に対するエラーハンドリングを担当する。

第三に、記事取得 Repository を使用して、Web スクレイピングまたは記事取得 API を通じて記事内容を取得する。取得した HTML から本文を抽出し、テキスト形式で返す。

第四に、記事取得サービスの一時的な障害やタイムアウトに対するエラーハンドリングを担当する。リトライ処理は Application 層で実装する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: article-fetch-service-specification -->

記事取得サービス仕様として以下を定義する。

### 記事取得サービスの責務

- 指定 URL から記事内容を取得するユースケース実装を行う。
- URL のバリデーションと正規化を行う。
- 記事取得 Repository を使用して記事内容を取得する。
- エラーハンドリングと例外処理を行う。

### 記事取得サービスのインターフェース

- `fetch_article(url: str) -> Article`: 指定 URL から記事を取得する

### 記事取得サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 記事取得 Repository への依存のみを持つ。
- URL のバリデーションを Application 層で行う。
- 記事取得サービスの障害に対するエラーハンドリングを実装する。

<!-- GLOBAL_CONCLUSION_END: article-fetch-service-specification -->
