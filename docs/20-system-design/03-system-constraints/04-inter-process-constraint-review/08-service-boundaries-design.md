---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "service-boundaries-decision"
  produces:
    - "service-boundaries-adjustment"
    - "inter-process-service-boundaries-technology-mapping"
    - "inter-process-service-boundaries-missing-technologies"
---

# サービス間 Process 制約の見直し - サービス境界設計

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

### サービス境界設計

<!-- PREMISE_BEGIN: service-boundaries-decision -->

技術的観点からの最適解は**3 サービス完全分離**となる：

- **コア知識管理サービス**: 安定した CRUD 処理を独立プロセスで実行し、長期安定リリースサイクルで運用
- **LLM 統合サービス**: 外部 API 依存を独立プロセスで分離し、技術進歩に応じた中期リリースサイクルで運用
- **コンテンツ取得サービス**: Web スクレイピングを独立プロセスで分離し、サイト変更に応じた短期リリースサイクルで運用

この構成により実行時の責務分離とリリースサイクルの最適化を両立し、各サービスの独立性を確保できる。

<!-- PREMISE_END: service-boundaries-decision -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

サービス境界設計は Physical・Development 制約との矛盾が発生している。Process 制約の「3 サービス完全分離」は Physical 制約の「単一 Backend Container」と矛盾する。単一 Backend Container 内でのモジュール分離に変更する必要がある。

## Physical 制約との調整

Process 制約「3 サービス完全分離」を「単一バックエンド内でのモジュール分離」に変更する。

<!-- GLOBAL_CONCLUSION_BEGIN: service-boundaries-adjustment -->

- **単一バックエンド内でのモジュール分離**: コア知識管理機能、LLM 統合機能、コンテンツ取得機能を単一バックエンド内でモジュール分離で実現する。

<!-- GLOBAL_CONCLUSION_END: service-boundaries-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

調整後の Process 制約について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 単一バックエンド内でのモジュール分離の検討

**Process 制約内容**: コア知識管理機能、LLM 統合機能、コンテンツ取得機能を単一バックエンド内でモジュール分離で実現する。

**Development 制約との適合性**:

単一バックエンド内でのモジュール分離は Backend Container の Python + FastAPI により実現可能である。FastAPIはルーター機能により機能別モジュール分離をサポートしている。Pythonのモジュール設計により責務分離を実現できる。

**使用技術および設定**:

- Python + FastAPI: モジュール分離・ルーター機能

## Development 制約との適合性検討結果の抽出

サービス境界設計要素の技術適合性結果を整理する。「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- 単一バックエンド内でのモジュール分離の検討
  - Python + FastAPI: モジュール分離・ルーター機能

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-service-boundaries-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- Python + FastAPI
  - モジュール分離・ルーター機能

<!-- GLOBAL_CONCLUSION_END: inter-process-service-boundaries-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないサービス境界設計要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-service-boundaries-missing-technologies -->

現在の技術スタックでは全てのサービス境界設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-service-boundaries-missing-technologies -->