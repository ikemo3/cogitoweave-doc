---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "traceability-strategy"
  produces:
    - "inter-process-traceability-constraint-adjustment"
    - "inter-process-traceability-technology-mapping"
    - "inter-process-traceability-missing-technologies"
---

# サービス間 Process 制約の見直し - トレーサビリティ設計

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

### トレーサビリティ設計

<!-- PREMISE_BEGIN: traceability-strategy -->

- **エラートラッキングとシステム監視を分離**: エラー情報の詳細追跡とシステム全体の稼働監視を分離し、既存インフラとの統合を優先する。
- **フロントエンド起点の一括トレーシング**: CSR アーキテクチャを活用したクライアントサイド監視と 3 つのバックエンドサービスとの統合により、システム全体の動作を把握する。
- **必要最小限の監視項目**: 障害検知・エラー把握・パフォーマンス監視に絞り込み、複雑なメトリクスや分析機能は除外する。

<!-- PREMISE_END: traceability-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約は「3 つのバックエンドサービスとの統合」を前提としているが、Physical 制約では単一 Backend Container となるため矛盾が発生する。

- **フロントエンド起点の一括トレーシング**: 「3 つのバックエンドサービスとの統合」は3サービス完全分離を前提としている

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-traceability-constraint-adjustment -->

- **フロントエンド起点の一括トレーシング**: CSR アーキテクチャを活用したクライアントサイド監視と単一バックエンドとの統合により、システム全体の動作を把握する

<!-- GLOBAL_CONCLUSION_END: inter-process-traceability-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

**PREMISE マーカー不変ルール**: PREMISE マーカー内の内容は一切変更してはならない。Physical 制約との矛盾が発見された場合でも、PREMISE マーカー内は元の内容のまま保持し、調整結果は「Physical 制約との調整」セクションの GLOBAL_CONCLUSION マーカー内に記述する。

調整後の Process 制約の各トレーサビリティ要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### エラートラッキングとシステム監視を分離の検討

**Process 制約内容**: エラー情報の詳細追跡とシステム全体の稼働監視を分離し、既存インフラとの統合を優先する。

**Development 制約との適合性**:

エラー情報の詳細追跡は Backend Container の loguru により実現可能である。システム全体の稼働監視は Sentry・Google Cloud Monitoring により実現可能である。既存インフラとの統合も設定可能である。

**使用技術および設定**:

- loguru: エラー情報の詳細追跡
- Sentry: APM・エラートラッキング
- Google Cloud Monitoring: システム稼働監視

### フロントエンド起点の一括トレーシングの検討

**Process 制約内容**: CSR アーキテクチャを活用したクライアントサイド監視と単一バックエンドとの統合により、システム全体の動作を把握する。

**Development 制約との適合性**:

CSR アーキテクチャはフロントエンドの Solid + TanStack Router により実現可能である。クライアントサイド監視は Sentry により実現可能である。単一バックエンドとの統合は Backend Container により実現可能である。

**使用技術および設定**:

- Solid + TanStack Router: CSR アーキテクチャ
- Sentry: クライアントサイド監視・トレーシング
- Backend Container: 単一バックエンドとの統合

### 必要最小限の監視項目の検討

**Process 制約内容**: 障害検知・エラー把握・パフォーマンス監視に絞り込み、複雑なメトリクスや分析機能は除外する。

**Development 制約との適合性**:

障害検知・エラー把握は Sentry により実現可能である。パフォーマンス監視は Google Cloud Monitoring により実現可能である。複雑なメトリクスや分析機能を除外することで、必要最小限の監視を実現できる。

**使用技術および設定**:

- Sentry: 障害検知・エラー把握
- Google Cloud Monitoring: パフォーマンス監視
- Fluent Bit: ログ収集

## Development 制約との適合性検討結果の抽出

各トレーサビリティ要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- エラートラッキングとシステム監視を分離の検討
  - loguru: エラー情報の詳細追跡
  - Sentry: APM・エラートラッキング
  - Google Cloud Monitoring: システム稼働監視
- フロントエンド起点の一括トレーシングの検討
  - Solid + TanStack Router: CSR アーキテクチャ
  - Sentry: クライアントサイド監視・トレーシング
  - Backend Container: 単一バックエンドとの統合
- 必要最小限の監視項目の検討
  - Sentry: 障害検知・エラー把握
  - Google Cloud Monitoring: パフォーマンス監視
  - Fluent Bit: ログ収集

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-traceability-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- loguru
  - エラー情報の詳細追跡
- Sentry
  - APM・エラートラッキング
  - クライアントサイド監視・トレーシング
  - 障害検知・エラー把握
- Google Cloud Monitoring
  - システム稼働監視
  - パフォーマンス監視
- Solid + TanStack Router
  - CSR アーキテクチャ
- Backend Container
  - 単一バックエンドとの統合
- Fluent Bit
  - ログ収集

<!-- GLOBAL_CONCLUSION_END: inter-process-traceability-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないトレーサビリティ要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-traceability-missing-technologies -->

現在の技術スタックでは全てのトレーサビリティ設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-traceability-missing-technologies -->
