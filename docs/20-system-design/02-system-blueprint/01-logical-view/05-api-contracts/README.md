---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces: []
---

# API 契約 (抽象)

## 前提

ワークフローで定義された 3 つの核ワークフロー、データモデルで定義された 5 つの核エンティティ、UI 要素で抽出された 15 個の UI 要素を踏まえ、システム間・コンポーネント間の通信契約を論理的に定義する。

API 契約は役割名ベースで統一し、CQRS の Command（データ変更）と Query（データ参照）を明確に分離する。入力・出力・エラーは概念項目レベルで記述し、実装詳細は含めない。

## 論理

### API 分類と構造

**Command 系 API（データ変更操作）**:

- 概念管理: 概念作成
- 文献管理: 文献作成、文献メモ保存
- 関係性管理: 概念関連作成、概念グループ作成

**Query 系 API（データ参照操作）**:

- 概念参照: 概念検索、概念詳細取得
- メモ参照: 文献メモ一覧取得、文献メモ詳細取得
- 外部連携: 外部記事取得、LLM 要約生成、LLM 概念候補提示
- 空間分析: 空間クラスタ認識

### 個別 API ファイル

#### Command 系（5 ファイル）

- [command-concept-01-create.md](command-concept-01-create.md) - 概念作成 API
- [command-literature-01-create.md](command-literature-01-create.md) - 文献作成 API
- [command-memo-01-save.md](command-memo-01-save.md) - 文献メモ保存 API
- [command-relation-01-create.md](command-relation-01-create.md) - 概念関連作成 API
- [command-group-01-create.md](command-group-01-create.md) - 概念グループ作成 API

#### Query 系（9 ファイル）

**概念参照系**:

- [query-concept-01-search.md](query-concept-01-search.md) - 概念検索 API
- [query-concept-02-get.md](query-concept-02-get.md) - 概念詳細取得 API

**メモ参照系**:

- [query-memo-01-list.md](query-memo-01-list.md) - 文献メモ一覧取得 API
- [query-memo-02-detail.md](query-memo-02-detail.md) - 文献メモ詳細取得 API

**外部連携系**:

- [query-external-01-article-fetch.md](query-external-01-article-fetch.md) - 外部記事取得 API
- [query-llm-01-summary-generate.md](query-llm-01-summary-generate.md) - LLM 要約生成 API
- [query-llm-02-concept-suggest.md](query-llm-02-concept-suggest.md) - LLM 概念候補提示 API

**空間分析系**:

- [query-spatial-01-cluster-recognize.md](query-spatial-01-cluster-recognize.md) - 空間クラスタ認識 API

### API 設計原則

- **役割名ベース**: Service 名で機能境界を明確化
- **CQRS 適用**: Command（データ変更）と Query（参照）を明確に分離
- **概念項目のみ**: 入力・出力・エラーは概念レベルで定義（実装詳細は含めない）
- **境界越え対応**: 外部 API 連携ポイントを明記
- **自己完結性**: 各 API ファイルは単体で理解可能

## 結論

CogitoWeave システムの API 契約は 14 個の API で構成され、CQRS パターンに従って Command（5 個）と Query（9 個）に分類される。

各 API は対応するワークフローと密接に関連し、データモデルの 5 つの核エンティティ（概念、文献メモ、文献、概念関連、概念グループ）の操作を担当する。

外部連携 API（記事取得、LLM 処理）と内部分析 API（空間クラスタ認識）により、システムの境界越えポイントを明確化し、知識管理の自動化を実現する。全 API は概念項目レベルで定義され、実装に依存しない論理的契約として機能する。
