---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "system-test-strategy"
  produces:
    - "inter-process-system-test-technology-mapping"
    - "inter-process-system-test-missing-technologies"
---

# サービス間 Process 制約の見直し - システムテスト戦略

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

### システムテスト戦略

<!-- PREMISE_BEGIN: system-test-strategy -->

- **CI 統合疎通テスト**: フロントエンドから 3 サービスへの基本疎通確認のみを CI 環境で実行する。
- **外部 API 依存をモック**: バックエンドサービス内でモック実装によりダミーデータを返却して検証する。
- **バックエンド実起動**: フロントエンドからの HTTP 通信を受け付けるため、フロントエンド側でのモック機能は不要となる。
- **正常系とセキュリティをカバー**: エラーハンドリングや複雑な E2E シナリオは単体テストレベルで対応済みとして除外する。
- **CI 失敗時の迅速復旧**: PR マージ停止、修正または revert による迅速復旧を基本とする。

<!-- PREMISE_END: system-test-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約「フロントエンドから 3 サービスへの基本疎通確認」は「3 つのサービス」を前提としているが、Physical 制約では単一バックエンドコンテナでの実装となるため矛盾が発生する。

- **CI 統合疎通テスト**: フロントエンドから 3 サービスへの基本疎通確認のみを CI 環境で実行する。

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

- **CI 統合疎通テスト**: フロントエンドから単一バックエンドへの基本疎通確認のみを CI 環境で実行する。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各テスト要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### CI 統合疎通テストの検討

**Process 制約内容**: フロントエンドから単一バックエンドへの基本疎通確認のみを CI 環境で実行する。

**Development 制約との適合性**:

フロントエンドから単一バックエンドへの疎通確認は GitHub Actions による CI 環境で実現可能である。フロントエンドテストは Vitest + Testing Library + Playwright により E2E テストとして実現可能である。バックエンドテストは pytest + FastAPI TestClient により API テストとして実現可能である。

**使用技術および設定**:

- GitHub Actions: CI 環境での自動テスト実行
- Playwright: E2E テストによる疎通確認
- FastAPI TestClient: API テスト実行

### 外部 API 依存をモックの検討

**Process 制約内容**: バックエンドサービス内でモック実装によりダミーデータを返却して検証する。

**Development 制約との適合性**:

バックエンドサービス内でのモック実装は pytest-mock + httpx-mock + time-machine により実現可能である。外部 API 依存のモック化により、CI 環境での安定したテスト実行が可能となる。

**使用技術および設定**:

- pytest-mock: モック機能
- httpx-mock: HTTP 通信モック
- time-machine: 時間モック

### バックエンド実起動の検討

**Process 制約内容**: フロントエンドからの HTTP 通信を受け付けるため、フロントエンド側でのモック機能は不要となる。

**Development 制約との適合性**:

バックエンド実起動は FastAPI + uvicorn により実現可能である。フロントエンド側でのモック機能は不要で、実際の HTTP 通信による統合テストが可能である。

**使用技術および設定**:

- FastAPI + uvicorn: バックエンド実起動
- HTTP クライアント通信: 実際の HTTP 通信テスト

### 正常系とセキュリティをカバーの検討

**Process 制約内容**: エラーハンドリングや複雑な E2E シナリオは単体テストレベルで対応済みとして除外する。

**Development 制約との適合性**:

正常系とセキュリティのカバーは Vitest + Testing Library + Playwright によるフロントエンドテストと pytest + FastAPI TestClient によるバックエンドテストで実現可能である。複雑なシナリオは単体テストレベルで対応済みとすることで、CI 環境での実行時間を短縮できる。

**使用技術および設定**:

- Vitest + Testing Library + Playwright: フロントエンド正常系テスト
- pytest + FastAPI TestClient: バックエンド正常系・セキュリティテスト

### CI 失敗時の迅速復旧の検討

**Process 制約内容**: PR マージ停止、修正または revert による迅速復旧を基本とする。

**Development 制約との適合性**:

CI 失敗時の迅速復旧は GitHub の Git サービスにより実現可能である。PR マージ停止は GitHub の branch protection rules により実現可能である。修正または revert による復旧も Git の標準機能で実現可能である。

**使用技術および設定**:

- GitHub: PR マージ停止機能
- Git サービス: 修正・revert 機能

## Development 制約との適合性検討結果の抽出

各テスト要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- CI 統合疎通テストの検討
  - GitHub Actions: CI 環境での自動テスト実行
  - Playwright: E2E テストによる疎通確認
  - FastAPI TestClient: API テスト実行
- 外部 API 依存をモックの検討
  - pytest-mock: モック機能
  - httpx-mock: HTTP 通信モック
  - time-machine: 時間モック
- バックエンド実起動の検討
  - FastAPI + uvicorn: バックエンド実起動
  - HTTP クライアント通信: 実際の HTTP 通信テスト
- 正常系とセキュリティをカバーの検討
  - Vitest + Testing Library + Playwright: フロントエンド正常系テスト
  - pytest + FastAPI TestClient: バックエンド正常系・セキュリティテスト
- CI 失敗時の迅速復旧の検討
  - GitHub: PR マージ停止機能
  - Git サービス: 修正・revert 機能

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-system-test-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- GitHub Actions
  - CI 環境での自動テスト実行
- GitHub
  - PR マージ停止機能
- Git サービス
  - 修正・revert 機能
- Playwright
  - E2E テストによる疎通確認
- FastAPI TestClient
  - API テスト実行
- pytest-mock
  - モック機能
- httpx-mock
  - HTTP 通信モック
- time-machine
  - 時間モック
- FastAPI + uvicorn
  - バックエンド実起動
- HTTP クライアント通信
  - 実際の HTTP 通信テスト
- Vitest + Testing Library + Playwright
  - フロントエンド正常系テスト
- pytest + FastAPI TestClient
  - バックエンド正常系・セキュリティテスト

<!-- GLOBAL_CONCLUSION_END: inter-process-system-test-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないテスト要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-system-test-missing-technologies -->

現在の技術スタックでは全てのシステムテスト戦略要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-system-test-missing-technologies -->
