---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "backend-layer-structure"
  produces:
    - "backend-module-structure"
---

# 02-module-structure - モジュール/パッケージ構造

## 概要

レイヤードアーキテクチャと機能境界に基づく水平分割を、ディレクトリ構造で具体化する実装契約を定義する。

## 前提制約

<!-- PREMISE_BEGIN: backend-container-integrated-constraints -->

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

<!-- PREMISE_END: backend-container-integrated-constraints -->

<!-- PREMISE_BEGIN: backend-layer-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

### 各層の責務

**API 層**:

- FastAPI ルーター定義
- HTTP リクエスト/レスポンス処理
- Pydantic モデルによる入力バリデーション
- HTTP ステータスコードエラーハンドリング

**Application 層**:

- ユースケース実装
- トランザクション制御
- 複数機能にまたがる処理の調整
- Repository 層を使った外部システム協調

**Domain 層**:

- ビジネスエンティティ定義(純粋な Python)
- エンティティ内で閉じるビジネスロジック
- データ構造から自然に導出されるロジック
- ドメインサービスはほぼ不要(DB 制約で整合性管理)

**Repository 層**:

- SQLAlchemy ORM 定義(テーブル構造)
- データベースアクセス
- 外部 API 通信(LLM、記事取得)
- Domain エンティティ ⇔ ORM エンティティ変換
- クエリ実行、トランザクション境界

### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

<!-- PREMISE_END: backend-layer-structure -->

## 論理

### ディレクトリ構造の分割方針

レイヤードアーキテクチャ(縦の分割)と機能境界に基づく水平分割を、ディレクトリ構造で表現する必要がある。レイヤー優先、機能優先、機能境界優先の 3 つの選択肢がある。

レイヤー優先では、1 つの機能を実装する際に複数のディレクトリを横断する必要があり、認知負荷が高い。機能優先では、概念、文献メモ、関係性が密接に関連しているため、相互依存が発生しやすい。前提制約でも「コア知識管理境界は概念管理、文献メモ管理、関係性管理、質問探索を統合」とあり、細かく分けすぎると循環依存のリスクがある。

機能境界優先であれば、境界ごとに独立性を保ちつつ、境界内部では密接に関連する機能をまとめて管理できる。機能境界は相互依存を避けて独立性を保つために設計されている。

### API 提供の境界

前提制約に「外部 API 処理の分離」「LLM 協働やコンテンツ取得は将来的にマイクロサービス化の可能性」とある。LLM 統合境界とコンテンツ取得境界は Repository 層の責務(外部システム通信)であり、外部に API を提供しない。コア知識管理境界のみが外部に API を提供する。

LLM 統合境界とコンテンツ取得境界は、コア知識管理境界の Application 層から Repository 層を通じて利用される内部サービスである。

### shared の構成

フロントエンドと同様に、横断的な関心事を責務別に配置する。共通 Pydantic モデル(schemas/)、共通例外クラス(exceptions/)、DB 接続・セッション管理(database/)を基本とし、必要に応じて責務が明確なディレクトリを追加する。util/, common/, lib/ のような曖昧な名前は使用しない。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-module-structure -->

バックエンドは機能境界優先のディレクトリ構造とし、境界内部でレイヤーを分割する。

### ディレクトリ構造

```plaintext
src/
├── core/                    # コア知識管理境界(唯一のAPI提供)
│   ├── api/
│   │   ├── __init__.py      # Router集約
│   │   ├── concept.py
│   │   ├── literature_memo.py
│   │   ├── relationship.py
│   │   └── question.py
│   ├── application/
│   │   ├── concept_service.py
│   │   ├── literature_memo_service.py
│   │   └── ...
│   ├── domain/
│   │   ├── concept.py
│   │   ├── literature_memo.py
│   │   └── ...
│   └── repository/
│       ├── concept_repository.py
│       └── ...
├── llm/                     # LLM統合境界(外部API通信のみ)
│   └── repository/
│       └── llm_client.py
├── content/                 # コンテンツ取得境界(外部API通信のみ)
│   └── repository/
│       └── content_fetcher.py
├── shared/                  # 共通コード
│   ├── schemas/             # 共通Pydanticモデル
│   ├── exceptions/          # 共通例外クラス
│   ├── database/            # DB接続、セッション管理
│   └── (責務が明確になった時点で追加)
└── main.py                  # FastAPIアプリケーション、Router集約
```

### モジュール構造の原則

- 機能境界で分割し、境界間の独立性を保つ
- 境界内部では密接に関連する機能を統合管理する
- API 提供は core のみ、llm/content は内部サービスとして Repository 層を提供
- shared は責務別に分割し、曖昧な名前(util/common/lib)を避ける
- レイヤー構造を境界内部で実現し、縦横の責務分離を両立する

<!-- GLOBAL_CONCLUSION_END: backend-module-structure -->
