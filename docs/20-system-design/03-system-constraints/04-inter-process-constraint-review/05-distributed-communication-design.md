---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "distributed-communication-design"
  produces:
    - "inter-process-distributed-communication-constraint-adjustment"
    - "inter-process-distributed-communication-technology-mapping"
    - "inter-process-distributed-communication-missing-technologies"
---

# サービス間 Process 制約の見直し - 分散通信設計

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

### 分散通信設計

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約は「フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信」を前提としているが、Physical 制約では単一 Backend Container となるため矛盾が発生する。

- **HTTP API 直接通信**: 「3 サービスへの直接通信」は3サービス完全分離を前提としている

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-distributed-communication-constraint-adjustment -->

- **HTTP API 直接通信**: フロントエンドから単一バックエンドへの直接 HTTP/HTTPS 通信を行う

<!-- GLOBAL_CONCLUSION_END: inter-process-distributed-communication-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

**PREMISE マーカー不変ルール**: PREMISE マーカー内の内容は一切変更してはならない。Physical 制約との矛盾が発見された場合でも、PREMISE マーカー内は元の内容のまま保持し、調整結果は「Physical 制約との調整」セクションの GLOBAL_CONCLUSION マーカー内に記述する。

調整後の Process 制約の各分散通信要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### HTTP API 直接通信の検討

**Process 制約内容**: フロントエンドから単一バックエンドへの直接 HTTP/HTTPS 通信を行う。

**Development 制約との適合性**:

HTTP API 直接通信は Frontend Container の Solid + TanStack Router と Backend Container の FastAPI により実現可能である。フロントエンドからバックエンドへの直接 HTTP/HTTPS 通信が可能である。

**使用技術および設定**:

- Solid + TanStack Router: フロントエンド HTTP クライアント
- FastAPI: バックエンド HTTP API サーバー

### JSON データフォーマットの検討

**Process 制約内容**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。

**Development 制約との適合性**:

JSON データフォーマットは Backend Container の FastAPI により実現可能である。FastAPI は標準で JSON 形式のリクエスト・レスポンスに対応している。CQS ベースの API 設計も可能である。

**使用技術および設定**:

- FastAPI: JSON 形式 API・CQS 設計対応

### サービス特性ごとのタイムアウト設定の検討

**Process 制約内容**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。

**Development 制約との適合性**:

タイムアウト設定は Backend Container の httpx により実現可能である。httpx はリクエストごとのタイムアウト設定に対応している。各サービスの処理特性に応じた段階的設定が可能である。

**使用技術および設定**:

- httpx: リクエストタイムアウト設定

### HTTP ステータスコードエラーハンドリングの検討

**Process 制約内容**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。

**Development 制約との適合性**:

HTTP ステータスコードエラーハンドリングは Backend Container の FastAPI により実現可能である。FastAPI は標準で HTTP ステータスコードによるエラーハンドリングに対応している。リトライ戦略も実装可能である。

**使用技術および設定**:

- FastAPI: HTTP ステータスコード・エラーハンドリング

### 最小限セキュリティ設計の検討

**Process 制約内容**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

**Development 制約との適合性**:

最小限セキュリティ設計は Physical View のみの Caddy により実現可能である。Caddy は HTTPS 通信と CORS 設定に標準対応している。認証認可機能は除外可能である。

**使用技術および設定**:

- Caddy: HTTPS 通信・CORS 設定

## Development 制約との適合性検討結果の抽出

各分散通信要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- HTTP API 直接通信の検討
  - Solid + TanStack Router: フロントエンド HTTP クライアント
  - FastAPI: バックエンド HTTP API サーバー
- JSON データフォーマットの検討
  - FastAPI: JSON 形式 API・CQS 設計対応
- サービス特性ごとのタイムアウト設定の検討
  - httpx: リクエストタイムアウト設定
- HTTP ステータスコードエラーハンドリングの検討
  - FastAPI: HTTP ステータスコード・エラーハンドリング
- 最小限セキュリティ設計の検討
  - Caddy: HTTPS 通信・CORS 設定

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-distributed-communication-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- Solid + TanStack Router
  - フロントエンド HTTP クライアント
- FastAPI
  - バックエンド HTTP API サーバー
  - JSON 形式 API・CQS 設計対応
  - HTTP ステータスコード・エラーハンドリング
- httpx
  - リクエストタイムアウト設定
- Caddy
  - HTTPS 通信・CORS 設定

<!-- GLOBAL_CONCLUSION_END: inter-process-distributed-communication-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない分散通信要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-distributed-communication-missing-technologies -->

現在の技術スタックでは全ての分散通信設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-distributed-communication-missing-technologies -->
