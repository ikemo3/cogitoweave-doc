<!-- markdownlint-disable MD024 -->

# C4 Model

## 簡略化リスト

<!-- REFERENCE_BEGIN: development-constraint-simplified-list -->

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

<!-- REFERENCE_END: development-constraint-simplified-list -->

## Container 図

<!-- REFERENCE_BEGIN: development-container-diagram -->

```mermaid
C4Container
    title CogitoWeave システム Development 制約における技術構成

    Person(user, "ユーザー", "思想家・学習者")

    System_Boundary(app, "CogitoWeave システム") {
        Container(frontend, "Frontend Container", "Solid + TanStack Router", "ユーザーインターフェース・思考整理UI")
        Container(backend, "Backend Container", "Python + FastAPI", "ビジネスロジック・API提供")
        ContainerDb(database, "Database Container", "SQLite", "思考データ永続化")
    }

    System_Ext(github, "GitHub", "ソースコード管理・CI/CD")
    System_Ext(monitoring, "監視システム", "Sentry・Google Cloud Monitoring")

    Rel(user, frontend, "思考整理操作")
    Rel(frontend, backend, "API呼び出し", "JSON/HTTPS")
    Rel(backend, database, "データアクセス", "SQLite接続")
    Rel(backend, monitoring, "メトリクス・ログ送信")
    Rel(github, backend, "CI/CDデプロイ")
    Rel(github, frontend, "CI/CDデプロイ")
```

<!-- REFERENCE_END: development-container-diagram -->
