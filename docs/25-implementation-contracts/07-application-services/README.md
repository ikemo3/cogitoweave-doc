---
doc_type: "pattern-composite"
status: "draft"
depends:
  contracts: []
  produces: []
---

# Application サービス仕様

## 概要

Application 層のサービスコンポーネントの契約を定義する。各サービスはユースケース実装、トランザクション制御、複数 Repository の協調を担当し、ビジネスロジックの実行を制御する。

## サービス一覧

### エンティティ操作サービス

- [概念サービス](concept-service.md)
- [文献メモサービス](literature-memo-service.md)
- [関係性サービス](relationship-service.md)
- [文献サービス](literature-service.md)

### 検索・発見サービス

- [概念検索サービス](concept-search-service.md)
- [質問から概念候補提示サービス](question-to-concepts-service.md)

### 協働・処理サービス

- [LLM 要約生成サービス](llm-summary-service.md)
- [記事取得サービス](article-fetch-service.md)
- [バックアップサービス](backup-service.md)

## 共通原則

### Application サービスの責務

- ユースケース実装
- トランザクション制御
- 複数 Repository の協調
- Domain エンティティのライフサイクル管理
- ビジネスプロセスの制御

### Application サービスの設計方針

- ステートレス実装
- リクエストスコープのライフサイクル
- Repository への依存のみ(Domain 層への直接依存なし)
- 例外処理と適切なエラーハンドリング
- トランザクション境界の明確化

## 結論

Application サービスの契約により、API 層と Repository 層の橋渡しを行い、ビジネスロジックの実行制御を提供する。
