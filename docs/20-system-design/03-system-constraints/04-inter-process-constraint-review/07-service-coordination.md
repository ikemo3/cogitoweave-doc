---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "service-coordination-pattern"
  produces:
    - "inter-process-service-coordination-constraint-adjustment"
    - "inter-process-service-coordination-technology-mapping"
    - "inter-process-service-coordination-missing-technologies"
---

# サービス間 Process 制約の見直し - サービス間協調

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

### サービス間協調

<!-- PREMISE_BEGIN: service-coordination-pattern -->

- **フロントエンド直接呼び出し**: フロントエンドから各サービスを直接呼び出す。
- **一方向依存**: フロントエンドからコンテンツ取得サービスと LLM 統合サービスへの依存のみ存在し、サービス間の相互依存は発生しない。
- **タイムアウトつき同期通信**: フロントエンドから各サービスへの通信はタイムアウトつき同期通信で行う。

<!-- PREMISE_END: service-coordination-pattern -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約は「フロントエンドから各サービスを直接呼び出す」を前提とし、「コンテンツ取得サービスと LLM 統合サービス」への依存を想定しているが、Physical 制約では単一 Backend Container となるため矛盾が発生する。

- **フロントエンド直接呼び出し**: 「各サービスを直接呼び出す」は3サービス完全分離を前提としている
- **一方向依存**: 「コンテンツ取得サービスと LLM 統合サービスへの依存」は複数サービスの存在を前提としている

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-service-coordination-constraint-adjustment -->

- **フロントエンド直接呼び出し**: フロントエンドから単一バックエンドを直接呼び出し、内部でコア知識管理・LLM 統合・コンテンツ取得のモジュール間協調を実現する
- **一方向依存**: フロントエンドから単一バックエンドへの依存のみ存在する
- **タイムアウトつき同期通信**: フロントエンドから単一バックエンドへの通信はタイムアウトつき同期通信で行う

<!-- GLOBAL_CONCLUSION_END: inter-process-service-coordination-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

調整後の Process 制約の各サービス間協調要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### フロントエンド直接呼び出しの検討

**Process 制約内容**: フロントエンドから単一バックエンドを直接呼び出し、内部でコア知識管理・LLM 統合・コンテンツ取得のモジュール間協調を実現する。

**Development 制約との適合性**:

フロントエンド直接呼び出しは Frontend Container の Solid + TanStack Router により実現可能である。単一バックエンド内でのモジュール間協調は Backend Container の Python + FastAPI により実現可能である。

**使用技術および設定**:

- Solid + TanStack Router: フロントエンドからの直接呼び出し
- Python + FastAPI: モジュール間協調

### 一方向依存の検討

**Process 制約内容**: フロントエンドから単一バックエンドへの依存のみ存在する。

**Development 制約との適合性**:

一方向依存は Frontend Container と Backend Container の関係により実現可能である。フロントエンドからバックエンドへの依存のみが存在し、バックエンドからフロントエンドへの依存は発生しない。

**使用技術および設定**:

- Frontend Container: バックエンドへの依存
- Backend Container: フロントエンドからの独立

### タイムアウトつき同期通信の検討

**Process 制約内容**: フロントエンドから単一バックエンドへの通信はタイムアウトつき同期通信で行う。

**Development 制約との適合性**:

タイムアウトつき同期通信は Backend Container の httpx により実現可能である。httpx はリクエストタイムアウト設定に対応している。フロントエンドからバックエンドへの同期通信も実現可能である。

**使用技術および設定**:

- httpx: タイムアウト設定つき同期通信

## Development 制約との適合性検討結果の抽出

各サービス間協調要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- フロントエンド直接呼び出しの検討
  - Solid + TanStack Router: フロントエンドからの直接呼び出し
  - Python + FastAPI: モジュール間協調
- 一方向依存の検討
  - Frontend Container: バックエンドへの依存
  - Backend Container: フロントエンドからの独立
- タイムアウトつき同期通信の検討
  - httpx: タイムアウト設定つき同期通信

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-service-coordination-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- Solid + TanStack Router
  - フロントエンドからの直接呼び出し
- Python + FastAPI
  - モジュール間協調
- Frontend Container
  - バックエンドへの依存
- Backend Container
  - フロントエンドからの独立
- httpx
  - タイムアウト設定つき同期通信

<!-- GLOBAL_CONCLUSION_END: inter-process-service-coordination-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないサービス間協調要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-service-coordination-missing-technologies -->

現在の技術スタックでは全てのサービス間協調設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-service-coordination-missing-technologies -->
