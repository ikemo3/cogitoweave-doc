---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "data-consistency-strategy"
  produces:
    - "inter-process-data-consistency-technology-mapping"
    - "inter-process-data-consistency-missing-technologies"
---

# サービス間 Process 制約の見直し - データ整合性設計

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

### データ整合性設計

<!-- PREMISE_BEGIN: data-consistency-strategy -->

- **単一データストア責務**: コア知識管理サービスのみがデータ永続化を担当する。
- **ステートレスサービス分離**: LLM 統合サービスとコンテンツ取得サービスは外部データ取得のみを担当し、データを永続化しない。
- **一方向データフロー**: 「外部取得 → フロントエンド統合 → コア保存」の流れに限定する。

<!-- PREMISE_END: data-consistency-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

データ整合性設計は Physical・Development 制約と整合している。単一 Backend Container が全ての永続化を担当し、Database Container(SQLite) が単一データストアとして機能する。

## Physical 制約との調整

Physical 制約との矛盾は発生しなかったため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各データ整合性要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 単一データストア責務の検討

**Process 制約内容**: コア知識管理サービスのみがデータ永続化を担当する。

**Development 制約との適合性**:

単一データストア責務は Backend Container の SQLAlchemy + Alembic と Database Container の SQLite により実現可能である。Backend Container 内でコア知識管理機能のみがデータベースアクセスを担当し、他の機能はステートレスに動作する。

**使用技術および設定**:

- SQLAlchemy + Alembic: ORM・マイグレーション管理
- SQLite: 単一データストア

### ステートレスサービス分離の検討

**Process 制約内容**: LLM 統合サービスとコンテンツ取得サービスは外部データ取得のみを担当し、データを永続化しない。

**Development 制約との適合性**:

ステートレスサービス分離は Backend Container 内でのモジュール分離により実現可能である。LLM 統合機能とコンテンツ取得機能をステートレスなモジュールとして実装し、データ永続化を行わない設計が可能である。

**使用技術および設定**:

- Backend Container: モジュール分離によるステートレス実装

### 一方向データフローの検討

**Process 制約内容**: 「外部取得 → フロントエンド統合 → コア保存」の流れに限定する。

**Development 制約との適合性**:

一方向データフローは Frontend Container、Backend Container、Database Container の連携により実現可能である。フロントエンドが外部データを統合し、バックエンドのコア知識管理機能がデータベースに保存する一方向の流れを実現できる。

**使用技術および設定**:

- Frontend Container: データ統合
- Backend Container: データ保存制御
- Database Container: データ永続化

## Development 制約との適合性検討結果の抽出

各データ整合性要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- 単一データストア責務の検討
  - SQLAlchemy + Alembic: ORM・マイグレーション管理
  - SQLite: 単一データストア
- ステートレスサービス分離の検討
  - Backend Container: モジュール分離によるステートレス実装
- 一方向データフローの検討
  - Frontend Container: データ統合
  - Backend Container: データ保存制御
  - Database Container: データ永続化

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-data-consistency-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- SQLAlchemy + Alembic
  - ORM・マイグレーション管理
- SQLite
  - 単一データストア
- Backend Container
  - モジュール分離によるステートレス実装
  - データ保存制御
- Frontend Container
  - データ統合
- Database Container
  - データ永続化

<!-- GLOBAL_CONCLUSION_END: inter-process-data-consistency-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないデータ整合性要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-data-consistency-missing-technologies -->

現在の技術スタックでは全てのデータ整合性設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-data-consistency-missing-technologies -->
