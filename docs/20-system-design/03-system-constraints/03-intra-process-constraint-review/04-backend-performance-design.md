---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "backend-performance-design"
  produces:
    - "intra-process-backend-performance-technology-mapping"
    - "intra-process-backend-performance-missing-technologies"
---

# サービス内 Process 制約の見直し - バックエンドパフォーマンス設計

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

### バックエンドパフォーマンス設計

<!-- PREMISE_BEGIN: backend-performance-design -->

- **応答時間制約**: API 呼び出し 600ms 以内という制約を満たすため、N+1 問題の対策とクエリの最適化を実装する。
- **最小限の最適化**: 複雑なキャッシュ層や分散処理は除外し、必要に応じたデータベースインデックスの設定による最小限の最適化を行う。
- **単一サーバー構成**: 数万件規模のデータに対応することにより、個人用途に十分な性能を確保する。
- **垂直スケーリング**: 性能が劣化した場合はサーバースペックの向上による垂直スケーリングで対応する。
- **シンプルな構成維持**: 複雑な最適化機構は保守性制約により実装せず、理解しやすい構成で長期安定運用を実現する。

<!-- PREMISE_END: backend-performance-design -->

## Physical 制約との適合性検討

Physical・Development 制約では Backend Container という単一のバックエンドコンテナでの実装が確定している。

Process 制約「バックエンドパフォーマンス設計」は Backend Container での実装を前提としており、Physical 制約と完全に整合している。矛盾は発生しない。

## Physical 制約との調整

Physical 制約との矛盾が存在しないため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 応答時間制約の検討

**Process 制約内容**: API 呼び出し 600ms 以内という制約を満たすため、N+1 問題の対策とクエリの最適化を実装する。

**Development 制約との適合性**:

N+1 問題の対策は SQLAlchemy の eager loading（joinedload、selectinload）による関連データ一括取得で実現可能である。クエリ最適化は SQLAlchemy の query optimization と SQLite のクエリプラン分析で実現可能である。API 応答時間測定は FastAPI のミドルウェアによる処理時間計測で実現可能である。

**使用技術および設定**:

- SQLAlchemy: eager loading による N+1 問題対策
- SQLite: クエリプラン最適化とインデックス活用
- FastAPI: 応答時間測定ミドルウェア
- loguru: パフォーマンスログ出力

### 最小限の最適化の検討

**Process 制約内容**: 複雑なキャッシュ層や分散処理は除外し、必要に応じたデータベースインデックスの設定による最小限の最適化を行う。

**Development 制約との適合性**:

データベースインデックスの設定は SQLite の CREATE INDEX と Alembic によるマイグレーション管理で実現可能である。複雑なキャッシュ層の除外は現在の技術スタックに Redis 等のキャッシュシステムが含まれていないため自然に実現されている。

**使用技術および設定**:

- SQLite: インデックス作成と最適化
- Alembic: インデックスマイグレーション管理
- SQLAlchemy: クエリ最適化機能

### 単一サーバー構成の検討

**Process 制約内容**: 数万件規模のデータに対応することにより、個人用途に十分な性能を確保する。

**Development 制約との適合性**:

単一サーバー構成は現在の Backend Container 単体での運用により実現される。数万件規模のデータ対応は SQLite の軽量データベースとしての特性と SQLAlchemy による効率的なデータアクセスで実現可能である。uvicorn による単一プロセス実行で個人用途に適した構成を実現できる。

**使用技術および設定**:

- uvicorn: 単一プロセス・単一サーバー実行
- SQLite: 軽量データベースによる数万件対応
- SQLAlchemy: 効率的データアクセス

### 垂直スケーリングの検討

**Process 制約内容**: 性能が劣化した場合はサーバースペックの向上による垂直スケーリングで対応する。

**Development 制約との適合性**:

垂直スケーリングは現在の単一コンテナ構成において、ホストマシンの CPU・メモリ・ストレージ増強により実現可能である。uvicorn の worker プロセス数調整による CPU 活用も可能である。SQLite は単一ファイルデータベースのため、ストレージ性能向上の恩恵を直接受けられる。

**使用技術および設定**:

- uvicorn: worker プロセス数調整
- SQLite: ストレージ性能向上対応
- **[技術不足]**: リソース監視ツール（CPU・メモリ使用量）

### シンプルな構成維持の検討

**Process 制約内容**: 複雑な最適化機構は保守性制約により実装せず、理解しやすい構成で長期安定運用を実現する。

**Development 制約との適合性**:

シンプルな構成維持は現在の技術スタックの軽量性により自然に実現される。FastAPI の直感的な API 定義、SQLAlchemy の標準的な ORM 機能、SQLite のファイルベースデータベースにより理解しやすい構成を実現できる。複雑な最適化機構（分散キャッシュ、複数データベース、非同期処理等）は現在のスタックに含まれていない。

**使用技術および設定**:

- FastAPI: 直感的な API 設計
- SQLAlchemy: 標準的な ORM 機能
- SQLite: シンプルなファイルベース DB
- Python: 理解しやすいプログラミング言語

## Development 制約との適合性検討結果の抽出

各要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 応答時間制約の検討
  - SQLAlchemy: eager loading による N+1 問題対策
  - SQLite: クエリプラン最適化とインデックス活用
  - FastAPI: 応答時間測定ミドルウェア
  - loguru: パフォーマンスログ出力
- 最小限の最適化の検討
  - SQLite: インデックス作成と最適化
  - Alembic: インデックスマイグレーション管理
  - SQLAlchemy: クエリ最適化機能
- 単一サーバー構成の検討
  - uvicorn: 単一プロセス・単一サーバー実行
  - SQLite: 軽量データベースによる数万件対応
  - SQLAlchemy: 効率的データアクセス
- 垂直スケーリングの検討
  - uvicorn: worker プロセス数調整
  - SQLite: ストレージ性能向上対応
  - **[技術不足]**: リソース監視ツール（CPU・メモリ使用量）
- シンプルな構成維持の検討
  - FastAPI: 直感的な API 設計
  - SQLAlchemy: 標準的な ORM 機能
  - SQLite: シンプルなファイルベース DB
  - Python: 理解しやすいプログラミング言語

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-backend-performance-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- SQLAlchemy
  - eager loading による N+1 問題対策
  - クエリ最適化機能
  - 効率的データアクセス
  - 標準的な ORM 機能
- SQLite
  - クエリプラン最適化とインデックス活用
  - インデックス作成と最適化
  - 軽量データベースによる数万件対応
  - ストレージ性能向上対応
  - シンプルなファイルベース DB
- FastAPI
  - 応答時間測定ミドルウェア
  - 直感的な API 設計
- uvicorn
  - 単一プロセス・単一サーバー実行
  - worker プロセス数調整
- Alembic
  - インデックスマイグレーション管理
- loguru
  - パフォーマンスログ出力
- Python
  - 理解しやすいプログラミング言語

<!-- GLOBAL_CONCLUSION_END: intra-process-backend-performance-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-backend-performance-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- リソース監視ツール: psutil、htop 等の CPU・メモリ使用量監視ツール

<!-- GLOBAL_CONCLUSION_END: intra-process-backend-performance-missing-technologies -->
