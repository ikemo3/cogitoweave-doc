---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-layer-structure"
  produces:
    - "backup-service-specification"
---

# バックアップサービス仕様

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

第一に、バックアップサービスは、データベースの全データをバックアップファイルに出力する Application 層のコンポーネントである。バックアップ Repository を使用してデータベースの内容を取得し、ファイルに書き出す。

第二に、バックアップの実行では、トランザクションを使用してデータの一貫性を保証する。バックアップ中にデータが変更されないよう、読み取り専用トランザクションを使用する。

第三に、バックアップファイルのパス生成とファイル出力は Application 層で行う。バックアップファイル名にはタイムスタンプを含め、複数バックアップの管理を容易にする。

第四に、バックアップの検証機能を提供する。バックアップファイルの整合性を確認し、復元可能であることを保証する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backup-service-specification -->

バックアップサービス仕様として以下を定義する。

### バックアップサービスの責務

- データベースのバックアップを実行するユースケース実装を行う。
- バックアップファイルのパス生成とファイル出力を行う。
- バックアップの検証を行う。
- トランザクション制御を担当する。
- エラーハンドリングと例外処理を行う。

### バックアップサービスのインターフェース

- `execute_backup() -> BackupResult`: バックアップを実行する
- `verify_backup(backup_path: str) -> VerifyResult`: バックアップを検証する

### バックアップサービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- バックアップ Repository への依存のみを持つ。
- 読み取り専用トランザクションを使用してデータの一貫性を保証する。
- バックアップファイル名にタイムスタンプを含める。

<!-- GLOBAL_CONCLUSION_END: backup-service-specification -->
