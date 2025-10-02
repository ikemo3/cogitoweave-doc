---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "backend-layer-structure"
    - "backend-module-structure"
    - "backend-dependency-injection"
  produces:
    - "backend-test-configuration"
---

# 04-test-configuration - テスト構成

## 概要

バックエンドのテスト環境構築とテストファイル配置の実装契約を定義する。

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

<!-- PREMISE_BEGIN: backend-module-structure -->

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

<!-- PREMISE_END: backend-module-structure -->

<!-- PREMISE_BEGIN: backend-dependency-injection -->

バックエンドは API 層と Application 層で FastAPI の依存性注入を使用し、コンストラクタ注入により依存を明示的にする。

### 注入対象レイヤー

- **API 層**: Application サービスを注入
- **Application 層**: Repository を注入
- **Repository 層**: 注入不要(純粋なデータアクセス)
- **Domain 層**: 注入不要(純粋なビジネスロジック)

### ライフサイクル管理

- **DB セッション**: リクエストスコープ(Depends で注入)
- **Repository**: リクエストスコープ(Depends で注入)
- **Application サービス**: リクエストスコープ(Depends で注入)

### 実装パターン

**DB セッション管理:**

```python
# shared/database/session.py
def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

**Repository 注入:**

```python
# core/repository/concept_repository.py
class ConceptRepository:
    def __init__(self, db: Session):
        self.db = db

# core/dependencies.py
def get_concept_repository(
    db: Session = Depends(get_db)
) -> ConceptRepository:
    return ConceptRepository(db)
```

**Application サービス注入:**

```python
# core/application/concept_service.py
class ConceptService:
    def __init__(self, concept_repo: ConceptRepository):
        self.concept_repo = concept_repo

# core/dependencies.py
def get_concept_service(
    concept_repo: ConceptRepository = Depends(get_concept_repository)
) -> ConceptService:
    return ConceptService(concept_repo)
```

**API 層での使用:**

```python
# core/api/concept.py
@router.get("/")
def list_concepts(
    service: ConceptService = Depends(get_concept_service)
):
    return service.list_concepts()
```

### 依存性注入の原則

- コンストラクタ注入により依存を明示的にする
- リクエストスコープでステートレス実装を保つ
- FastAPI の Depends を一貫して使用し、パターンを統一する
- 過剰な抽象化を避け、必要な層のみで注入を使用する
- dependencies.py に依存関係の構築ロジックを集約する

<!-- PREMISE_END: backend-dependency-injection -->

## 論理

### テスト戦略の選択

依存性注入によりコンストラクタで依存を明示的にしているため、テスト時に実装を差し替える柔軟性が確保されている。テストにおけるモックの使用方針として、ロンドン派(モック主義)とデトロイト派(古典派、状態検証主義)がある。

ロンドン派は外部依存を全てモック化し、各層を完全に独立してテストする。テストは高速だが、モック実装と実際の実装がずれるリスクがある。

デトロイト派はモックを最小限にし、実際の実装を使う。実際の動作に近く、リファクタリングに強い。前提制約に「トランザクション境界検証」があり、ORM の動作確認とトランザクション境界の検証が重要である。「DOA とデータベース制約により、意図的に Domain 層を薄く保つ」という設計方針から、DB 制約の動作確認が設計の本質的な部分である。SQLite インメモリは高速であり、実 DB を使用するコストは低い。

外部 API はネットワーク依存、課金リスク、レート制限があるため、モック必須である。DB は原則として実物を使用し、外部 API のみモック化するデトロイト派(古典派)を採用する。ただし、Repository のロジックを特に検証したい場合など、依存を明示的にしているため DB をモック化することも妨げない。

### テストファイル配置

Python のコミュニティでは `tests/` ディレクトリが標準的であり、pytest のデフォルト設定も `tests/` を探す。`src/` と `tests/` を分離することで、パッケージングが明確になる。

テストファイルは `src/` の構造をミラーする形で配置する。`tests/unit/` と `tests/integration/` のような分離は行わず、ファイル配置自体はミラー構造にして、テストの内容で単体テスト/統合テストを区別する。この方が一般的であり、ファイルを探しやすい。

### レイヤーごとのテスト方針

Domain 層は純粋な Python であり、外部依存がないため単体テストのみである。

Repository 層は実 DB(SQLite インメモリ)を使用する統合テストである。トランザクション境界検証と ORM の動作確認が目的である。

Application 層は実 Repository + 実 DB を使用し、外部 API のみモック化する統合テストである。ユースケース全体の動作を実際に近い環境で検証する。

API 層は FastAPI TestClient + 実 Repository + 実 DB を使用し、外部 API のみモック化する統合テストである。HTTP レイヤーからエンドツーエンドで検証する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-test-configuration -->

バックエンドはデトロイト派(古典派)のテスト戦略を採用し、原則として DB は実物を使用する。

### テスト戦略

- モック方針: デトロイト派(古典派、状態検証主義)
- DB: 原則として実物(SQLite インメモリ)を使用
- 外部 API: 必ずモック化(httpx-mock 使用)
- 実際の動作に近いテストを重視
- 柔軟性: 依存を明示的にしているため、特定のケースで DB をモック化することも妨げない

### テストディレクトリ構造

```plaintext
tests/
├── core/
│   ├── domain/
│   │   ├── test_concept.py              # 単体テスト(依存なし)
│   │   └── test_literature_memo.py
│   ├── repository/
│   │   ├── test_concept_repository.py   # 統合テスト(実DB)
│   │   └── ...
│   ├── application/
│   │   ├── test_concept_service.py      # 統合テスト(実DB + 外部APIモック)
│   │   └── ...
│   └── api/
│       ├── test_concept_api.py          # 統合テスト(FastAPI TestClient + 実DB + 外部APIモック)
│       └── ...
├── llm/
│   └── repository/
│       └── test_llm_client.py           # 統合テスト(外部APIモック)
├── content/
│   └── repository/
│       └── test_content_fetcher.py      # 統合テスト(外部APIモック)
└── conftest.py                          # pytest共通設定、フィクスチャ
```

### 各層のテスト方針

**Domain 層**:

- 単体テスト(依存なし)
- 純粋な Python のビジネスロジックをテスト

**Repository 層**:

- 統合テスト(実 DB)
- SQLite インメモリを使用
- トランザクション境界検証
- ORM の動作確認

**Application 層**:

- 統合テスト(実 Repository + 実 DB + 外部 API モック)
- ユースケース全体の動作検証
- httpx-mock で外部 API をモック化

**API 層**:

- 統合テスト(FastAPI TestClient + 実 Repository + 実 DB + 外部 API モック)
- HTTP レイヤーからエンドツーエンドで検証
- FastAPI TestClient を使用

### テスト環境の原則

- tests/ は src/ のミラー構造とし、ファイルを探しやすくする
- 原則として DB は実物を使用し、実際の動作に近いテストを実現する
- 依存を明示的にしているため、必要に応じて DB をモック化することも可能
- SQLite インメモリにより、実 DB を使用しても高速なテストを維持する
- pytest の共通設定・フィクスチャは conftest.py に集約する

<!-- GLOBAL_CONCLUSION_END: backend-test-configuration -->
