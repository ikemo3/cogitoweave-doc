---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "monitoring-design"
  produces:
    - "intra-process-monitoring-constraint-adjustment"
    - "intra-process-monitoring-technology-mapping"
    - "intra-process-monitoring-missing-technologies"
---

# サービス内 Process 制約の見直し - 監視設計

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

### サービス特性別監視

<!-- PREMISE_BEGIN: monitoring-design -->

- **サービス特性別監視**: コア知識管理サービスはデータ整合性と応答時間を重視し、LLM 統合サービスは外部 API 依存性を監視し、コンテンツ取得サービスは取得成功率と構造変更を追跡する。
- **階層的監視項目**: 必須監視項目で基本動作を保証し、性能監視項目で応答時間制約を管理し、運用監視項目で長期的な運用判断を支援する。
- **緊急度分類アラート**: 緊急度分類による適切な通知とアラート抑制による根本原因の明確化により、一人での迅速な障害対応を可能にする。

<!-- PREMISE_END: monitoring-design -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約「サービス特性別監視」は「3 つのサービス」を前提としているが、Physical 制約では単一バックエンドコンテナでの実装となるため矛盾が発生する。

- **サービス特性別監視**: コア知識管理サービスはデータ整合性と応答時間を重視し、LLM 統合サービスは外部 API 依存性を監視し、コンテンツ取得サービスは取得成功率と構造変更を追跡する。

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-monitoring-constraint-adjustment -->

- **単一バックエンド内での機能別監視**: コア知識管理機能はデータ整合性と応答時間を重視し、LLM 統合機能は外部 API 依存性を監視し、コンテンツ取得機能は取得成功率と構造変更を追跡する。

<!-- GLOBAL_CONCLUSION_END: intra-process-monitoring-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各監視要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 単一バックエンド内での機能別監視の検討

**Process 制約内容**: コア知識管理機能はデータ整合性と応答時間を重視し、LLM 統合機能は外部 API 依存性を監視し、コンテンツ取得機能は取得成功率と構造変更を追跡する。

**Development 制約との適合性**:

コア知識管理機能のデータ整合性監視は loguru による SQL 操作ログ出力で実現可能である。応答時間監視も loguru による処理時間ログで実現可能である。

LLM 統合機能の外部 API 依存性監視は loguru による API 呼び出し結果ログ出力で実現可能である。httpx による通信ログも合わせて実現可能である。

コンテンツ取得機能の取得成功率監視は loguru による取得結果ログ出力で実現可能である。構造変更の追跡についても loguru による構造解析結果ログで実現可能である。

**使用技術および設定**:

- loguru: 機能別ログ出力とログレベル設定
- httpx: 外部 API 通信ログ
- SQLAlchemy: データベース操作ログ
- Fluent Bit: 構造化ログ分析ツール

### 階層的監視項目の検討

**Process 制約内容**: 必須監視項目で基本動作を保証し、性能監視項目で応答時間制約を管理し、運用監視項目で長期的な運用判断を支援する。

**Development 制約との適合性**:

必須監視項目の基本動作保証は loguru による API 正常応答ログで実現可能である。性能監視項目の応答時間制約管理も loguru による処理時間測定ログで実現可能である。

運用監視項目の長期的な運用判断支援は loguru によるメトリクス蓄積で基本的な実現は可能である。詳細な分析は Google Cloud Monitoring と Sentry により実現可能である。

**使用技術および設定**:

- loguru: 階層的ログレベル設定（ERROR、WARN、INFO、DEBUG）
- FastAPI: 応答時間測定用ミドルウェア
- Google Cloud Monitoring: 外部監視システム
- Sentry: APM・トレーシング

### 緊急度分類アラートの検討

**Process 制約内容**: 緊急度分類による適切な通知とアラート抑制による根本原因の明確化により、一人での迅速な障害対応を可能にする。

**Development 制約との適合性**:

緊急度分類による適切な通知は loguru のログレベル設定で基本的な分類は可能だが、実際の通知機能は外部アラートシステムが必要である。アラート抑制による根本原因の明確化は Sentry と Google Cloud Monitoring により部分的に実現可能である。

一人での迅速な障害対応については、Fluent Bit によるログ集約・検索システムにより実現可能である。

**使用技術および設定**:

- loguru: 緊急度分類（ERROR=即座対応、WARN=計画的改善）
- **[技術不足]**: 外部アラートシステム（例：PagerDuty）
- Fluent Bit: ログ集約・検索システム

## Development 制約との適合性検討結果の抽出

各監視要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 単一バックエンド内での機能別監視の検討
  - loguru: 機能別ログ出力とログレベル設定
  - httpx: 外部 API 通信ログ
  - SQLAlchemy: データベース操作ログ
  - Fluent Bit: 構造化ログ分析ツール
- 階層的監視項目の検討
  - loguru: 階層的ログレベル設定（ERROR、WARN、INFO、DEBUG）
  - FastAPI: 応答時間測定用ミドルウェア
  - Google Cloud Monitoring: 外部監視システム
  - Sentry: APM・トレーシング
- 緊急度分類アラートの検討
  - loguru: 緊急度分類（ERROR=即座対応、WARN=計画的改善）
  - **[技術不足]**: 外部アラートシステム（例：Sentry、PagerDuty）
  - Fluent Bit: ログ集約・検索システム

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-monitoring-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- loguru
  - 機能別ログ出力とログレベル設定
  - 階層的ログレベル設定（ERROR、WARN、INFO、DEBUG）
  - 緊急度分類（ERROR=即座対応、WARN=計画的改善）
- httpx
  - 外部 API 通信ログ
- SQLAlchemy
  - データベース操作ログ
- FastAPI
  - 応答時間測定用ミドルウェア
- Fluent Bit
  - 構造化ログ分析ツール
  - ログ集約・検索システム
- Google Cloud Monitoring
  - 外部監視システム
- Sentry
  - APM・トレーシング

<!-- GLOBAL_CONCLUSION_END: intra-process-monitoring-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない監視要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-monitoring-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- 外部アラートシステム: PagerDuty 等の専用アラートサービス

<!-- GLOBAL_CONCLUSION_END: intra-process-monitoring-missing-technologies -->
