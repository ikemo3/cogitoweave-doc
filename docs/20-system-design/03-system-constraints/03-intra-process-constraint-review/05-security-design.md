---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "security-design"
  produces:
    - "intra-process-security-technology-mapping"
    - "intra-process-security-missing-technologies"
---

# サービス内 Process 制約の見直し - セキュリティ設計

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

### セキュリティ設計

<!-- PREMISE_BEGIN: security-design -->

- **認証・認可なし**: 個人環境前提により実装せず、シンプルな構成を維持する。
- **セキュリティヘッダー設定**: CSP、X-Frame-Options、X-Content-Type-Options、X-XSS-Protection により基本的なブラウザレベル防御を確立する。
- **HSTS 設定除外**: 開発・テスト環境への支障を避けるため除外する。
- **多層防御実装**: リクエストレイヤーでの入力バリデーション、データアクセスレイヤーでの SQL インジェクション対策、レスポンスレイヤーでの XSS 対策を実装する。
- **個人用途最適化**: Rate Limiting は個人用途により不要とし、セキュリティログも別レイヤー処理として除外する。
- **法的要件遵守**: LLM API 利用規約遵守により、法的要件を満たしたセキュリティ体制を実現する。

<!-- PREMISE_END: security-design -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約「セキュリティ設計」は Backend Container での実装を前提としており、Physical 制約と完全に整合している。矛盾は発生しない。

## Physical 制約との調整

Physical 制約との矛盾が存在しないため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 認証・認可なしの検討

**Process 制約内容**: 個人環境前提により実装せず、シンプルな構成を維持する。

**Development 制約との適合性**:

認証・認可機能の除外は現在の技術スタックに認証ライブラリ（OAuth、JWT 等）が含まれていないため自然に実現される。FastAPI の dependency injection による認証機構の除外により、シンプルな API エンドポイント設計を実現できる。個人環境での使用により外部からのアクセス制御は不要である。

**使用技術および設定**:

- FastAPI: 認証機構なしのシンプル API 設計
- **[技術不足]**: 認証・認可ライブラリ（意図的に除外）

### セキュリティヘッダー設定の検討

**Process 制約内容**: CSP、X-Frame-Options、X-Content-Type-Options、X-XSS-Protection により基本的なブラウザレベル防御を確立する。

**Development 制約との適合性**:

セキュリティヘッダーの設定は FastAPI のミドルウェアによる HTTP レスポンスヘッダー追加で実現可能である。CSP（Content Security Policy）による XSS 対策、X-Frame-Options による clickjacking 対策、X-Content-Type-Options による MIME タイプスニッフィング対策、X-XSS-Protection による XSS フィルタ有効化が可能である。

**使用技術および設定**:

- FastAPI: セキュリティヘッダー設定ミドルウェア
- **[技術不足]**: リバースプロキシレベルでのヘッダー設定（Caddy 等）

### HSTS 設定除外の検討

**Process 制約内容**: 開発・テスト環境への支障を避けるため除外する。

**Development 制約との適合性**:

HSTS（HTTP Strict Transport Security）の除外は現在の開発環境で HTTP 通信も必要であることを考慮した妥当な判断である。本番環境でも個人用途のため、HTTPS 強制の必要性は低い。開発時の利便性を優先した構成を実現できる。

**使用技術および設定**:

- FastAPI: HSTS ヘッダー設定除外
- 開発環境: HTTP/HTTPS 両対応

### 多層防御実装の検討

**Process 制約内容**: リクエストレイヤーでの入力バリデーション、データアクセスレイヤーでの SQL インジェクション対策、レスポンスレイヤーでの XSS 対策を実装する。

**Development 制約との適合性**:

リクエストレイヤーの入力バリデーションは Pydantic による型チェック・範囲チェック・形式チェックで実現可能である。データアクセスレイヤーの SQL インジェクション対策は SQLAlchemy の ORM による パラメータ化クエリで実現可能である。レスポンスレイヤーの XSS 対策は FastAPI の自動 HTML エスケープとセキュリティヘッダーで実現可能である。

**使用技術および設定**:

- Pydantic: 入力バリデーション（型・範囲・形式チェック）
- SQLAlchemy: パラメータ化クエリによる SQL インジェクション対策
- FastAPI: HTML エスケープと XSS 対策

### 個人用途最適化の検討

**Process 制約内容**: Rate Limiting は個人用途により不要とし、セキュリティログも別レイヤー処理として除外する。

**Development 制約との適合性**:

Rate Limiting の除外は現在の技術スタックに Redis 等の Rate Limiting ライブラリが含まれていないため自然に実現される。セキュリティログの除外により loguru は通常のアプリケーションログに専念できる。個人用途のため大量アクセス対策は不要である。

**使用技術および設定**:

- **[技術不足]**: Rate Limiting ライブラリ（意図的に除外）
- loguru: 通常のアプリケーションログに専念

### 法的要件遵守の検討

**Process 制約内容**: LLM API 利用規約遵守により、法的要件を満たしたセキュリティ体制を実現する。

**Development 制約との適合性**:

LLM API 利用規約遵守は環境変数による API キーの適切な管理で実現可能である。httpx による外部 API 通信時の利用規約に準拠したリクエスト送信を実現できる。個人情報の適切な取り扱いは SQLite による ローカルデータベース保存で実現可能である。

**使用技術および設定**:

- 環境変数方式: API キー管理と利用規約遵守
- httpx: 利用規約準拠の外部 API 通信
- SQLite: ローカルデータ保存による個人情報保護

## Development 制約との適合性検討結果の抽出

各要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 認証・認可なしの検討
  - FastAPI: 認証機構なしのシンプル API 設計
  - **[技術不足]**: 認証・認可ライブラリ（意図的に除外）
- セキュリティヘッダー設定の検討
  - FastAPI: セキュリティヘッダー設定ミドルウェア
  - **[技術不足]**: リバースプロキシレベルでのヘッダー設定（Caddy 等）
- HSTS 設定除外の検討
  - FastAPI: HSTS ヘッダー設定除外
  - 開発環境: HTTP/HTTPS 両対応
- 多層防御実装の検討
  - Pydantic: 入力バリデーション（型・範囲・形式チェック）
  - SQLAlchemy: パラメータ化クエリによる SQL インジェクション対策
  - FastAPI: HTML エスケープと XSS 対策
- 個人用途最適化の検討
  - **[技術不足]**: Rate Limiting ライブラリ（意図的に除外）
  - loguru: 通常のアプリケーションログに専念
- 法的要件遵守の検討
  - 環境変数方式: API キー管理と利用規約遵守
  - httpx: 利用規約準拠の外部 API 通信
  - SQLite: ローカルデータ保存による個人情報保護

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-security-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- FastAPI
  - 認証機構なしのシンプル API 設計
  - セキュリティヘッダー設定ミドルウェア
  - HSTS ヘッダー設定除外
  - HTML エスケープと XSS 対策
- Pydantic
  - 入力バリデーション（型・範囲・形式チェック）
- SQLAlchemy
  - パラメータ化クエリによる SQL インジェクション対策
- 環境変数方式
  - API キー管理と利用規約遵守
- httpx
  - 利用規約準拠の外部 API 通信
- SQLite
  - ローカルデータ保存による個人情報保護
- loguru
  - 通常のアプリケーションログに専念
- 開発環境
  - HTTP/HTTPS 両対応

<!-- GLOBAL_CONCLUSION_END: intra-process-security-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-security-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- リバースプロキシレベルでのセキュリティヘッダー設定: Caddy、nginx 等

<!-- GLOBAL_CONCLUSION_END: intra-process-security-missing-technologies -->
