---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "request-processing-design"
  produces:
    - "intra-process-request-processing-technology-mapping"
    - "intra-process-request-processing-missing-technologies"
---

# サービス内 Process 制約の見直し - リクエスト処理設計

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

### リクエスト処理設計

<!-- PREMISE_BEGIN: request-processing-design -->

- **同期処理ベース**: 基本的に同期処理でシンプルな処理フローを実現し、複雑な業務処理は 1 トランザクション内で完結させる。
- **ログレベル設定**: 開発者対応の緊急度に応じたログレベル（ERROR=即座対応、WARN=計画的改善、INFO=動作確認、DEBUG=開発詳細）を採用し、ユーザーには理解しやすいエラーメッセージのみを提供する。
- **トランザクション分離**: データベース操作のみを対象とし、外部 API 呼び出しは分離する。
- **リクエスト検証**: 型チェック、範囲チェック、形式チェックを含む定型的な検証を実施し、ビジネスロジック検証は別レイヤーで処理する。
- **統一タイムアウト**: 全 API に 10 秒の統一タイムアウトを設定し、外部 API エラー時はフロントエンド側リトライで対応する。
- **性能最適化**: N+1 問題対策とクエリ最適化により性能要件を満たす。

<!-- PREMISE_END: request-processing-design -->

## Physical 制約との適合性検討

Physical・Development 制約では Backend Container での API 処理と Database Container でのデータアクセスが確定している。

Process 制約「リクエスト処理設計」は Backend Container での FastAPI による API 処理を前提としており、Physical 制約と完全に整合している。矛盾は発生しない。

## Physical 制約との調整

Physical 制約との矛盾が存在しないため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 同期処理ベースの検討

**Process 制約内容**: 基本的に同期処理でシンプルな処理フローを実現し、複雑な業務処理は 1 トランザクション内で完結させる。

**Development 制約との適合性**:

同期処理ベースは FastAPI の同期エンドポイント定義により実現可能である。シンプルな処理フローは Python の逐次実行による理解しやすいコード構造で実現できる。複雑な業務処理の 1 トランザクション内完結は SQLAlchemy のトランザクション管理により実現可能である。

**使用技術および設定**:

- FastAPI: 同期エンドポイント定義
- Python: 逐次実行による同期処理
- SQLAlchemy: トランザクション管理

### ログレベル設定の検討

**Process 制約内容**: 開発者対応の緊急度に応じたログレベル（ERROR=即座対応、WARN=計画的改善、INFO=動作確認、DEBUG=開発詳細）を採用し、ユーザーには理解しやすいエラーメッセージのみを提供する。

**Development 制約との適合性**:

ログレベル設定は loguru の階層的ログレベル管理により実現可能である。開発者対応の緊急度分類は loguru の設定で実現できる。ユーザーへの理解しやすいエラーメッセージは FastAPI の例外ハンドラーによる適切なエラーレスポンス生成で実現可能である。

**使用技術および設定**:

- loguru: 階層的ログレベル管理（ERROR、WARN、INFO、DEBUG）
- FastAPI: 例外ハンドラーによる適切なエラーメッセージ生成

### トランザクション分離の検討

**Process 制約内容**: データベース操作のみを対象とし、外部 API 呼び出しは分離する。

**Development 制約との適合性**:

トランザクション分離は SQLAlchemy のトランザクション境界内にデータベース操作のみを含める設計で実現可能である。外部 API 呼び出しは httpx による通信をトランザクション外で実行することで分離できる。FastAPI のエンドポイント内でのフロー制御により適切な実行順序を管理できる。

**使用技術および設定**:

- SQLAlchemy: データベース操作のみのトランザクション境界
- httpx: トランザクション外での外部 API 呼び出し
- FastAPI: エンドポイント内フロー制御

### リクエスト検証の検討

**Process 制約内容**: 型チェック、範囲チェック、形式チェックを含む定型的な検証を実施し、ビジネスロジック検証は別レイヤーで処理する。

**Development 制約との適合性**:

リクエスト検証は Pydantic による型チェック・範囲チェック・形式チェックの自動実行で実現可能である。定型的な検証は FastAPI の自動バリデーション機能により実現される。ビジネスロジック検証の別レイヤー処理は FastAPI のエンドポイント内でのサービス層分離により実現可能である。

**使用技術および設定**:

- Pydantic: 型チェック・範囲チェック・形式チェック
- FastAPI: 自動バリデーション機能
- FastAPI: サービス層分離によるビジネスロジック検証

### 統一タイムアウトの検討

**Process 制約内容**: 全 API に 10 秒の統一タイムアウトを設定し、外部 API エラー時はフロントエンド側リトライで対応する。

**Development 制約との適合性**:

統一タイムアウトは FastAPI のミドルウェアによる全 API 共通のタイムアウト設定で実現可能である。10 秒のタイムアウト設定は uvicorn の設定で指定可能である。外部 API エラー時のフロントエンド側リトライは httpx のタイムアウト・リトライ設定と Solid による前端リトライロジックで実現可能である。

**使用技術および設定**:

- FastAPI: ミドルウェアによる統一タイムアウト設定
- uvicorn: タイムアウト設定（10 秒）
- httpx: 外部 API タイムアウト・リトライ設定
- **[技術不足]**: フロントエンド側リトライ機構の詳細実装

### 性能最適化の検討

**Process 制約内容**: N+1 問題対策とクエリ最適化により性能要件を満たす。

**Development 制約との適合性**:

N+1 問題対策は SQLAlchemy の eager loading（joinedload、selectinload）により実現可能である。クエリ最適化は SQLAlchemy のクエリ最適化機能と SQLite のインデックス活用により実現可能である。性能要件達成は FastAPI の高速な API 処理と組み合わせて実現できる。

**使用技術および設定**:

- SQLAlchemy: eager loading による N+1 問題対策
- SQLAlchemy: クエリ最適化機能
- SQLite: インデックス活用
- FastAPI: 高速 API 処理

## Development 制約との適合性検討結果の抽出

各要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 同期処理ベースの検討
  - FastAPI: 同期エンドポイント定義
  - Python: 逐次実行による同期処理
  - SQLAlchemy: トランザクション管理
- ログレベル設定の検討
  - loguru: 階層的ログレベル管理（ERROR、WARN、INFO、DEBUG）
  - FastAPI: 例外ハンドラーによる適切なエラーメッセージ生成
- トランザクション分離の検討
  - SQLAlchemy: データベース操作のみのトランザクション境界
  - httpx: トランザクション外での外部 API 呼び出し
  - FastAPI: エンドポイント内フロー制御
- リクエスト検証の検討
  - Pydantic: 型チェック・範囲チェック・形式チェック
  - FastAPI: 自動バリデーション機能
  - FastAPI: サービス層分離によるビジネスロジック検証
- 統一タイムアウトの検討
  - FastAPI: ミドルウェアによる統一タイムアウト設定
  - uvicorn: タイムアウト設定（10 秒）
  - httpx: 外部 API タイムアウト・リトライ設定
  - **[技術不足]**: フロントエンド側リトライ機構の詳細実装
- 性能最適化の検討
  - SQLAlchemy: eager loading による N+1 問題対策
  - SQLAlchemy: クエリ最適化機能
  - SQLite: インデックス活用
  - FastAPI: 高速 API 処理

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-request-processing-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- FastAPI
  - 同期エンドポイント定義
  - 例外ハンドラーによる適切なエラーメッセージ生成
  - エンドポイント内フロー制御
  - 自動バリデーション機能
  - サービス層分離によるビジネスロジック検証
  - ミドルウェアによる統一タイムアウト設定
  - 高速 API 処理
- SQLAlchemy
  - トランザクション管理
  - データベース操作のみのトランザクション境界
  - eager loading による N+1 問題対策
  - クエリ最適化機能
- Python
  - 逐次実行による同期処理
- loguru
  - 階層的ログレベル管理（ERROR、WARN、INFO、DEBUG）
- httpx
  - トランザクション外での外部 API 呼び出し
  - 外部 API タイムアウト・リトライ設定
- Pydantic
  - 型チェック・範囲チェック・形式チェック
- uvicorn
  - タイムアウト設定（10 秒）
- SQLite
  - インデックス活用

<!-- GLOBAL_CONCLUSION_END: intra-process-request-processing-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-request-processing-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- フロントエンド側リトライ機構: 具体的なリトライライブラリや実装パターン

<!-- GLOBAL_CONCLUSION_END: intra-process-request-processing-missing-technologies -->
