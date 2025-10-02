---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "scaling-strategy"
  produces:
    - "inter-process-scaling-technology-mapping"
    - "inter-process-scaling-missing-technologies"
---

# サービス間 Process 制約の見直し - スケーリング設計

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

### スケーリング設計

<!-- PREMISE_BEGIN: scaling-strategy -->

- **単一サーバー構成**: 水平スケーリングの複雑さを回避し、必要に応じた垂直スケーリングにより性能向上を図る。
- **数万件規模への対応**: 概念・文献メモ・関係性データを含めて数万件規模まで対応し、現代の一般的なサーバーで十分な性能を実現する。
- **シンプル運用を優先**: 複雑なスケーリング機構や最適化は実装せず、理解しやすく保守しやすい構成を維持する。

<!-- PREMISE_END: scaling-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

スケーリング設計は Physical・Development 制約と完全に整合している。Physical 制約の単一サーバー構成と Docker 環境が、Process 制約の単一サーバー・垂直スケーリング方針と一致している。

## Physical 制約との調整

Physical 制約との矛盾は発生しなかったため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各スケーリング要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 単一サーバー構成の検討

**Process 制約内容**: 水平スケーリングの複雑さを回避し、必要に応じた垂直スケーリングにより性能向上を図る。

**Development 制約との適合性**:

単一サーバー構成はセルフホスト環境と Docker 環境により実現可能である。垂直スケーリングは物理的なリソース増設で対応可能である。水平スケーリングの複雑さを回避することで運用負荷を軽減できる。

**使用技術および設定**:

- セルフホスト環境: 単一物理サーバーでの運用
- Docker 環境: コンテナレベルでの統合管理

### 数万件規模への対応の検討

**Process 制約内容**: 概念・文献メモ・関係性データを含めて数万件規模まで対応し、現代の一般的なサーバーで十分な性能を実現する。

**Development 制約との適合性**:

数万件規模のデータ処理は SQLAlchemy + Alembic と SQLite により実現可能である。SQLite は数万件規模のデータ処理に十分対応できる。現代の一般的なサーバースペックで動作可能である。

**使用技術および設定**:

- SQLAlchemy + Alembic: データベースアクセス・マイグレーション管理
- SQLite: 軽量データベースエンジン

### シンプル運用を優先の検討

**Process 制約内容**: 複雑なスケーリング機構や最適化は実装せず、理解しやすく保守しやすい構成を維持する。

**Development 制約との適合性**:

シンプル運用は Docker 環境での単純な構成により実現可能である。複雑なスケーリング機構は不要で、基本的なコンテナ管理のみで運用できる。理解しやすく保守しやすい構成を維持できる。

**使用技術および設定**:

- Docker 環境: 単純なコンテナ構成での管理

## Development 制約との適合性検討結果の抽出

各スケーリング要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- 単一サーバー構成の検討
  - セルフホスト環境: 単一物理サーバーでの運用
  - Docker 環境: コンテナレベルでの統合管理
- 数万件規模への対応の検討
  - SQLAlchemy + Alembic: データベースアクセス・マイグレーション管理
  - SQLite: 軽量データベースエンジン
- シンプル運用を優先の検討
  - Docker 環境: 単純なコンテナ構成での管理

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-scaling-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- セルフホスト環境
  - 単一物理サーバーでの運用
- Docker 環境
  - コンテナレベルでの統合管理
  - 単純なコンテナ構成での管理
- SQLAlchemy + Alembic
  - データベースアクセス・マイグレーション管理
- SQLite
  - 軽量データベースエンジン

<!-- GLOBAL_CONCLUSION_END: inter-process-scaling-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないスケーリング要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-scaling-missing-technologies -->

現在の技術スタックでは全てのスケーリング設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-scaling-missing-technologies -->
