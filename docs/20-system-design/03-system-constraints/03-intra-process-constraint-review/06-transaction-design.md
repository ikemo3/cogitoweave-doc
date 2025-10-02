---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "transaction-design"
  produces:
    - "intra-process-transaction-technology-mapping"
    - "intra-process-transaction-missing-technologies"
---

# サービス内 Process 制約の見直し - トランザクション設計

## Physical + Development 制約の技術スタック

各コンテナの技術スタックは以下のとおりである。この技術構成に Process 制約を適合させる必要がある。

<!-- PREMISE_BEGIN: development-constraint-simplified-list -->

- Frontend(Container)
  - フレームワーク：Solid + TanStack Router
  - CSS 手法：Tailwind CSS + CSS Modules ハイブリッド構成
  - UI コンポーネント：Tailwind CSS ベースカスタム実装
  - グラフ可視化：D3
- Backend(Container)
  - 言語・フレームワーク：Python + FastAPI
  - アプリケーションサーバー：uvicorn
  - ORM・マイグレーション：SQLAlchemy + Alembic
  - API ドキュメント生成：FastAPI 内蔵 Swagger UI・ReDoc
  - バリデーション：FastAPI 標準 Pydantic
  - HTTP クライアント：httpx
  - 設定管理：環境変数方式
  - ログ・モニタリング：loguru
- DB(Container)
  - データベース：SQLite
- Vite proxy(Container)
  - ビルドツール：Vite
- GitHub Actions(Container)
  - Git サービス：GitHub
  - フロントエンドテスト：Vitest + Testing Library + Playwright
  - バックエンドテスト
    - テストフレームワーク：pytest
    - API テスト：FastAPI TestClient
    - 非同期テスト：pytest-asyncio
    - モック機能：pytest-mock
    - HTTP モック：httpx-mock
    - 時間モック：time-machine
    - カバレッジ：pytest-cov
    - 並列実行：pytest-xdist
- Fluent Bit(Container)
  - ログ収集：Fluent Bit
- Cloud Monitoring(Container)
  - 監視・ログ：Google Cloud Monitoring
- Sentry(Container)
  - APM・トレーシング：Sentry

<!-- PREMISE_END: development-constraint-simplified-list -->

## Process 制約の整理

Physical 制約・Development 制約との適合性を検討する前に、対象となる Process 制約の内容を整理する。

### トランザクション設計

<!-- PREMISE_BEGIN: transaction-design -->

- **トランザクション境界**: 1 つの API リクエスト内で関連するデータ更新を 1 つのトランザクションで完結させ、部分失敗時は全てロールバックする原子性を確保する。
- **外部 API 分離**: 外部 API 呼び出しはトランザクション外で処理し、データベース操作のみをトランザクション境界とする。
- **READ_COMMITTED 分離レベル**: ギャップロック・ネクストキーロックによるデッドロック危険性を回避する。
- **テーブルアクセス順序統一**: デッドロック発生を予防し、万一発生した場合は例外処理でエラーレスポンスを提供する。
- **データベース制約活用**: データベース制約による整合性管理を活用し、シンプルで理解しやすいトランザクション制御を実現する。

<!-- PREMISE_END: transaction-design -->

## Physical 制約との適合性検討

Physical・Development 制約では Backend Container と Database Container での実装が確定している。

Process 制約「トランザクション設計」は Backend Container での SQLAlchemy による ORM 操作と Database Container の SQLite でのトランザクション管理を前提としており、Physical 制約と完全に整合している。矛盾は発生しない。

## Physical 制約との調整

Physical 制約との矛盾が存在しないため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### トランザクション境界の検討

**Process 制約内容**: 1 つの API リクエスト内で関連するデータ更新を 1 つのトランザクションで完結させ、部分失敗時は全てロールバックする原子性を確保する。

**Development 制約との適合性**:

SQLAlchemy のセッション管理による API リクエスト単位でのトランザクション境界設定が実現可能である。FastAPI の依存性注入による セッション管理とコミット・ロールバック制御により原子性を確保できる。部分失敗時の全体ロールバックは SQLAlchemy のトランザクション機能で自動実現される。

**使用技術および設定**:

- SQLAlchemy: セッション管理とトランザクション制御
- FastAPI: 依存性注入によるセッションライフサイクル管理
- SQLite: ACID 特性によるトランザクション保証

### 外部 API 分離の検討

**Process 制約内容**: 外部 API 呼び出しはトランザクション外で処理し、データベース操作のみをトランザクション境界とする。

**Development 制約との適合性**:

外部 API 分離は httpx による外部通信をトランザクション開始前または終了後に実行する設計で実現可能である。SQLAlchemy のトランザクション境界内にはデータベース操作のみを含め、httpx による外部 API 呼び出しは分離して実行できる。失敗時の補償トランザクション（Saga パターン）も実装可能である。

**使用技術および設定**:

- httpx: トランザクション外での外部 API 通信
- SQLAlchemy: データベース操作のみのトランザクション境界
- FastAPI: API 呼び出しフローの制御

### READ_COMMITTED 分離レベルの検討

**Process 制約内容**: ギャップロック・ネクストキーロックによるデッドロック危険性を回避する。

**Development 制約との適合性**:

SQLite は READ_COMMITTED 分離レベルをサポートしており、SQLAlchemy の設定で指定可能である。ギャップロック・ネクストキーロックは MySQL/PostgreSQL の機能であり、SQLite では発生しないため自然に回避される。SQLite のロック機構はシンプルで理解しやすい。

**使用技術および設定**:

- SQLite: READ_COMMITTED 分離レベルサポート
- SQLAlchemy: 分離レベル設定
- **[技術不足]**: 高度なロック制御機構（SQLite の制限）

### テーブルアクセス順序統一の検討

**Process 制約内容**: デッドロック発生を予防し、万一発生した場合は例外処理でエラーレスポンスを提供する。

**Development 制約との適合性**:

テーブルアクセス順序統一は SQLAlchemy のクエリ実行順序の標準化で実現可能である。SQLite は単一ライターモデルのためデッドロックが発生しにくいが、万一の場合は SQLAlchemy の例外処理機構でエラーハンドリングできる。FastAPI の例外ハンドラーによる適切なエラーレスポンス返却も可能である。

**使用技術および設定**:

- SQLAlchemy: テーブルアクセス順序制御と例外処理
- SQLite: 単一ライターモデルによるデッドロック回避
- FastAPI: 例外ハンドラーによるエラーレスポンス

### データベース制約活用の検討

**Process 制約内容**: データベース制約による整合性管理を活用し、シンプルで理解しやすいトランザクション制御を実現する。

**Development 制約との適合性**:

データベース制約活用は SQLite の PRIMARY KEY、FOREIGN KEY、CHECK 制約と Alembic によるマイグレーション管理で実現可能である。SQLAlchemy の ORM モデル定義と連携したデータベース制約により、アプリケーションレベルの複雑な制御を排除してシンプルなトランザクション制御を実現できる。

**使用技術および設定**:

- SQLite: データベース制約（PRIMARY KEY、FOREIGN KEY、CHECK）
- Alembic: 制約定義のマイグレーション管理
- SQLAlchemy: ORM モデルと制約の連携

## Development 制約との適合性検討結果の抽出

各要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- トランザクション境界の検討
  - SQLAlchemy: セッション管理とトランザクション制御
  - FastAPI: 依存性注入によるセッションライフサイクル管理
  - SQLite: ACID 特性によるトランザクション保証
- 外部 API 分離の検討
  - httpx: トランザクション外での外部 API 通信
  - SQLAlchemy: データベース操作のみのトランザクション境界
  - FastAPI: API 呼び出しフローの制御
- READ_COMMITTED 分離レベルの検討
  - SQLite: READ_COMMITTED 分離レベルサポート
  - SQLAlchemy: 分離レベル設定
  - **[技術不足]**: 高度なロック制御機構（SQLite の制限）
- テーブルアクセス順序統一の検討
  - SQLAlchemy: テーブルアクセス順序制御と例外処理
  - SQLite: 単一ライターモデルによるデッドロック回避
  - FastAPI: 例外ハンドラーによるエラーレスポンス
- データベース制約活用の検討
  - SQLite: データベース制約（PRIMARY KEY、FOREIGN KEY、CHECK）
  - Alembic: 制約定義のマイグレーション管理
  - SQLAlchemy: ORM モデルと制約の連携

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-transaction-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- SQLAlchemy
  - セッション管理とトランザクション制御
  - データベース操作のみのトランザクション境界
  - 分離レベル設定
  - テーブルアクセス順序制御と例外処理
  - ORM モデルと制約の連携
- SQLite
  - ACID 特性によるトランザクション保証
  - READ_COMMITTED 分離レベルサポート
  - 単一ライターモデルによるデッドロック回避
  - データベース制約（PRIMARY KEY、FOREIGN KEY、CHECK）
- FastAPI
  - 依存性注入によるセッションライフサイクル管理
  - API 呼び出しフローの制御
  - 例外ハンドラーによるエラーレスポンス
- httpx
  - トランザクション外での外部 API 通信
- Alembic
  - 制約定義のマイグレーション管理

<!-- GLOBAL_CONCLUSION_END: intra-process-transaction-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-transaction-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- 高度なロック制御機構: MySQL/PostgreSQL レベルの行レベルロック等（SQLite の制限）

<!-- GLOBAL_CONCLUSION_END: intra-process-transaction-missing-technologies -->
