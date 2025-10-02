<!-- markdownlint-disable MD024 -->

# アーキテクチャ制約の統合

## Frontend Container 統合制約

<!-- REFERENCE_BEGIN: frontend-container-integrated-constraints -->

### 物理配置

- セルフホスト環境の Docker コンテナとして静的配信

### 技術スタック

- フレームワーク: Solid + TanStack Router
- スタイリング: Tailwind CSS + CSS Modules
- グラフ可視化: D3
- ビルドツール: Vite
- テストフレームワーク: Vitest + Testing Library + Playwright

### アーキテクチャ方針

- CSR(Client-Side Rendering)採用
- SPA(Single Page Application)構造
- 軽量コンポーネント指向アーキテクチャ
- 3 層状態管理(コンポーネント/ページ/アプリケーション)
- コンテナコンポーネントパターンと単方向データフロー

### 画面構成

- メイン画面: 概念管理、文献メモ管理、関係性管理を統合
- 質問探索画面: 質問から概念発見とメモ参照を独立提供
- 空間配置画面: 概念の空間配置と関係性構築を専用提供
- タブ切り替えによる画面間移動

### パフォーマンス

- システム初回起動パフォーマンスバジェット対応
- 概念ページ読み込みパフォーマンスバジェット対応
- コード分割とバンドル最適化
- 軽量再レンダリング機構
- 必要時のグラフ描画最適化

### レスポンシブ対応

- Mac 環境: 情報整理機能最適化
- iPhone 環境: 情報参照機能最適化

### エラーハンドリング

- エラー境界コンポーネント
- ルーティングエラーハンドリング
- location.reload による強制リロード

### 監視・トレーシング

- Sentry による APM・エラートラッキング
- クライアントサイド監視・トレーシング
- 障害検知

### サービス間通信

- バックエンドへの依存(一方向)
- HTTP クライアントによる直接呼び出し
- データ統合責務

### 不足技術要素

- DOM 環境シミュレーション(jsdom/happy-dom)
- TanStack Router のテスト環境構築手法
- API モック化手法
- TypeScript 型生成(openapi-typescript)
- API 仕様変更の自動検知
- メモリ使用量監視(Chrome DevTools Memory)
- リトライ機構の実装パターン

<!-- REFERENCE_END: frontend-container-integrated-constraints -->

## Backend Container 統合制約

<!-- REFERENCE_BEGIN: backend-container-integrated-constraints -->

### 物理配置

- セルフホスト環境の Docker コンテナとして API + SQLite 提供

### 技術スタック

- 言語・フレームワーク: Python + FastAPI
- アプリケーションサーバー: uvicorn
- ORM・マイグレーション: SQLAlchemy + Alembic
- バリデーション: Pydantic
- HTTP クライアント: httpx
- ログ・モニタリング: loguru
- 設定管理: 環境変数方式
- テストフレームワーク: pytest + FastAPI TestClient + pytest-asyncio + pytest-mock + httpx-mock + time-machine

### アーキテクチャ方針

- レイヤードアーキテクチャ採用
- 階層的な責務分離
- 機能境界に基づく水平分割
- 薄いドメイン層アプローチ
- モジュール分離による機能独立
- ステートレス実装

### API 契約

- エンティティ操作 API: 概念 API、文献メモ API、関係性 API、文献 API
- 検索・発見 API: 概念検索 API、質問から概念候補提示 API
- 協働・処理 API: LLM 要約生成 API、記事取得 API、バックアップ API
- REST API 採用
- JSON 形式 API・CQS 設計対応
- HTTP ステータスコードエラーハンドリング
- API ドキュメント生成(FastAPI 内蔵 Swagger UI・ReDoc)

### 機能境界と責務

- コア知識管理境界: 概念管理、文献メモ管理、関係性管理、質問探索を統合
  - データベース操作とトランザクション境界を重点検証
  - データ整合性と応答時間を監視
- LLM 統合境界: LLM API による要約生成、概念候補提示、協働プロセス制御
  - 外部 API の契約テストと異常ケース対応を強化
  - 外部 API 依存性を監視
- コンテンツ取得境界: 外部記事取得、内容解析、品質チェック
  - 構造変更への適応性を継続検証
  - 取得成功率と構造変更を追跡

### ドメインモデル

- 4 つのエンティティに対して基本的な CRUD 操作と最小限のビジネス制約
- 複雑なドメインロジックはアプリケーション層に委ねる
- ORM モデルと制約の連携
- データベース制約による整合性管理

### データモデル

- 概念エンティティ: 思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能
- 文献メモエンティティ: ツェッテルカステン方式による概念ベースの知識蓄積として、LLM 協働により効率的に作成され個人の理解で修正される
- 関係性エンティティ: 概念間の無向グラフ構造による知識ネットワーク形成として、個人的な関係語彙による柔軟な関係定義と空間配置による直感的な関係性発見を支援
- 文献エンティティ: 外部文献の書誌情報管理として複数の概念観点から文献メモが生成される元情報

### トランザクション制御

- セッション管理とトランザクション制御
- 分離レベル設定
- テーブルアクセス順序制御と例外処理
- パラメータ化クエリによる SQL インジェクション対策

### パフォーマンス

- システム初回起動パフォーマンスバジェット対応
- eager loading による N+1 問題対策
- 単一プロセス・単一サーバー実行
- worker プロセス数調整
- タイムアウト設定

### 外部 API 通信

- 利用規約準拠の外部 API 通信
- タイムアウト・リトライ設定
- 外部 API 処理の分離

### ログ・監視

- 階層的ログレベル管理(ERROR、WARN、INFO、DEBUG)
- 機能別ログ出力
- 緊急度分類
- パフォーマンスログ出力
- ファイル出力とローテーション

### バッチ仕様

- 日次データバックアップのみを実装
- 全データの完全バックアップを日次実行
- バックアップ処理ログ出力
- システム障害時の迅速復旧を支援

### セキュリティ

- 入力バリデーション(型・範囲・形式チェック)
- API キー管理と利用規約遵守
- HTTP/HTTPS 両対応(開発環境)

### サービス間通信

- 単一バックエンドとの統合
- モジュール間協調
- フロントエンドからの独立
- データ保存制御
- 手動復旧手順の実装

### 不足技術要素

- トランザクション境界検証(pytest-sqlalchemy)
- CPU・メモリ使用量監視(psutil・htop)
- リバースプロキシレベルセキュリティヘッダー設定(Caddy・nginx)
- 高度なロック制御機構(MySQL/PostgreSQL レベル)

<!-- REFERENCE_END: backend-container-integrated-constraints -->

## Database Container 統合制約

<!-- REFERENCE_BEGIN: database-container-integrated-constraints -->

### 物理配置

- セルフホスト環境の Docker コンテナ内で Backend と統合配置

### 技術スタック

- データベース: SQLite

### データベース方針

- RDBMS 採用
- リレーショナル構造によるデータ整合性管理
- 成熟技術による保守性確保
- データ規模への十分な対応能力

### データモデル

- 概念エンティティ: 思考の最小単位として知識体系の中心に位置し、外部情報を個人の理解に変換する観点として機能
- 文献メモエンティティ: ツェッテルカステン方式による概念ベースの知識蓄積として、LLM 協働により効率的に作成され個人の理解で修正される
- 関係性エンティティ: 概念間の無向グラフ構造による知識ネットワーク形成として、個人的な関係語彙による柔軟な関係定義と空間配置による直感的な関係性発見を支援
- 文献エンティティ: 外部文献の書誌情報管理として複数の概念観点から文献メモが生成される元情報

### データ整合性

- データベース制約による整合性保証
- ACID 特性によるトランザクション保証
- READ_COMMITTED 分離レベルサポート
- 単一ライターモデルによるデッドロック回避

### パフォーマンス

- インデックス作成と最適化
- 軽量データベースによる数万件対応
- ストレージ性能向上対応
- WAL モードによる並行アクセス対応
- インメモリデータベース対応(テスト用)

### セキュリティ

- ローカルデータ保存による個人情報保護

### バックアップ

- 単一ファイル構成によるバックアップ
- ファイルベース定期バックアップ
- 日次データバックアップのみを実装
- 全データの完全バックアップを日次実行
- バックアップファイルの検証

### サービス間通信

- 単一データストア
- データ永続化責務

### 不足技術要素

- (なし)

<!-- REFERENCE_END: database-container-integrated-constraints -->

## Infrastructure 統合制約

<!-- REFERENCE_BEGIN: infrastructure-integrated-constraints -->

### 物理配置

- Cloudflare 環境: Tunnel + Access によるネットワーク技術
- セルフホスト環境
  - Docker 環境: Caddy(リバースプロキシ)、Fluent Bit(ログ収集)
  - Self-hosted Runner: Ansible(インフラ管理)
  - Backup: sqlite3+gzip+rclone
- ローカル開発環境: Vite proxy(開発環境プロキシ)
- GitHub Actions 環境: CI 実行
- 外部監視サービス群: Cloud Monitoring(監視・ログ)、Sentry(APM・トレーシング)

### CI/CD

- Git サービス: GitHub
- CI 環境での自動テスト実行
- PR マージ停止機能
- 修正・revert 機能
- E2E テストによる疎通確認(Playwright)

### ビルド・開発環境

- ビルドツール: Vite
- 開発環境プロキシ: Vite proxy

### ログ・監視

- ログ収集: Fluent Bit
- 構造化ログ分析ツール
- ログ集約・検索システム
- 監視・ログ: Google Cloud Monitoring
- システム稼働監視
- パフォーマンス監視
- APM・トレーシング: Sentry
- エラートラッキング
- 障害検知

### ネットワーク・セキュリティ

- HTTPS 通信・CORS 設定: Caddy
- リバースプロキシ: Caddy
- ネットワークゲートウェイ: Cloudflare Tunnel + Access

### コンテナ管理

- セルフホスト環境: 単一物理サーバーでの運用
- Docker 環境: コンテナレベルでの統合管理、単純なコンテナ構成での管理

### インフラ管理

- Ansible によるインフラ管理
- Self-hosted Runner による CD 実行環境

### バックアップ

- sqlite3+gzip+rclone による定期バックアップ
- ファイルシステムによるバックアップファイルの検証

### 不足技術要素

- 外部アラートシステム(PagerDuty)
- 定期実行スケジューラー(cron・systemd timer・Task Scheduler)

<!-- REFERENCE_END: infrastructure-integrated-constraints -->
