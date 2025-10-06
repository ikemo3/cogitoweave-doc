---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "distributed-communication-design"
    - "infrastructure-integrated-constraints"
  produces:
    - "backend-security-specification"
---

# セキュリティ仕様

## 概要

バックエンドのセキュリティ実装の具体的な方針を定義する。入力バリデーション、SQL インジェクション対策、API キー管理、CORS 設定を統一的に実装し、最小限かつ効果的なセキュリティを実現する。

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

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

<!-- PREMISE_BEGIN: infrastructure-integrated-constraints -->

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

<!-- PREMISE_END: infrastructure-integrated-constraints -->

## 論理

### セキュリティレイヤーの論理

セキュリティ対策を 3 つのレイヤーで実装する必要がある。

**3 層セキュリティ**:

1. **ネットワーク層**(Caddy/Cloudflare): HTTPS、セキュリティヘッダー、アクセス制御
2. **アプリケーション層**(FastAPI): CORS、入力バリデーション、SQL インジェクション対策
3. **データ層**(SQLAlchemy): パラメータ化クエリ、トランザクション制御

この 3 層により、多層防御(Defense in Depth)を実現する。

### 最小限セキュリティの論理

前提制約から「認証認可機能は除外する」ため、一人用システムとして最小限のセキュリティに絞る。

**実装するもの**:

- 入力バリデーション(既に Pydantic で実装済み)
- SQL インジェクション対策(SQLAlchemy のパラメータ化クエリで実現)
- CORS 設定(フロントエンドからのアクセス許可)
- セキュリティヘッダー(Caddy で設定、バックエンドからも補助的に設定)

**実装しないもの**:

- 認証・認可(一人用のため不要)
- レート制限(一人用のため不要)
- CSRF トークン(認証なしのため不要)

### API キー管理の論理

外部 API(LLM、記事取得等)の API キーを安全に管理する必要がある。

**管理方針**:

- 環境変数で管理(ソースコードにハードコードしない)
- `.env`ファイルを`.gitignore`に追加(Git にコミットしない)
- 利用規約遵守(API キーの適切な使用)

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-security-specification -->

### CORS 設定

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# CORS設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173",  # Vite開発サーバー
        "http://localhost:3000",  # 本番フロントエンド(Caddyプロキシ経由)
    ],
    allow_credentials=True,
    allow_methods=["*"],  # GET, POST等すべて許可
    allow_headers=["*"],  # すべてのヘッダー許可
)
```

### セキュリティヘッダー

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.responses import Response

app = FastAPI()

@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    """セキュリティヘッダーを追加"""
    response: Response = await call_next(request)

    # XSS対策
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"

    # コンテンツセキュリティポリシー(開発環境では緩く設定)
    response.headers["Content-Security-Policy"] = "default-src 'self'"

    return response
```

注: 本番環境では Caddy でより厳格なセキュリティヘッダーを設定する。

### SQL インジェクション対策

```python
# core/repository/concept_repository.py
from sqlalchemy.orm import Session
from core.repository.models import ConceptModel

class ConceptRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_by_name(self, name: str) -> ConceptModel | None:
        """概念名で検索(パラメータ化クエリ)"""
        # ✅ 安全: SQLAlchemyがパラメータ化クエリを生成
        return self.db.query(ConceptModel).filter(
            ConceptModel.name == name
        ).first()

    def search_by_keyword(self, keyword: str) -> list[ConceptModel]:
        """キーワードで部分一致検索(パラメータ化クエリ)"""
        # ✅ 安全: LIKEもパラメータ化される
        return self.db.query(ConceptModel).filter(
            ConceptModel.name.like(f"%{keyword}%")
        ).all()

    # ❌ 危険: 生SQLは使用しない
    # def unsafe_search(self, keyword: str):
    #     return self.db.execute(
    #         f"SELECT * FROM concept WHERE name LIKE '%{keyword}%'"
    #     ).fetchall()
```

### API キー管理

```python
# shared/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """環境変数設定"""

    # データベース設定
    database_url: str = "sqlite:///./cogitoweave.db"

    # LLM API設定
    llm_api_key: str
    llm_api_base_url: str = "https://api.openai.com/v1"

    # 記事取得API設定(必要に応じて)
    article_api_key: str | None = None

    # 開発環境設定
    debug: bool = False

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

# シングルトンインスタンス
settings = Settings()
```

`.env`ファイル例:

```bash
# .env (Gitにコミットしない!)
LLM_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
LLM_API_BASE_URL=https://api.openai.com/v1
DEBUG=true
```

`.gitignore`に追加:

```gitignore
.env
.env.*
!.env.example
```

### 入力バリデーション統合

バリデーション仕様で定義した Pydantic スキーマにより、入力バリデーションは自動的に実行される。

```python
# core/api/concept.py
from fastapi import APIRouter, Depends
from core.api.schemas.concept import ConceptCreateRequest

router = APIRouter()

@router.post("/")
async def create_concept(
    request: ConceptCreateRequest,  # ← Pydanticが自動バリデーション
    service: ConceptService = Depends(get_concept_service)
):
    # リクエスト到達時点でバリデーション済み
    return service.create_concept(
        name=request.name,
        description=request.description
    )
```

### LLM API 使用時のセキュリティ

```python
# llm/repository/llm_client.py
import httpx
from shared.config import settings
from shared.exceptions import ExternalServiceError

class LLMClient:
    def __init__(self):
        self.api_key = settings.llm_api_key
        self.base_url = settings.llm_api_base_url

    async def generate_summary(self, prompt: str) -> str:
        """LLM APIで要約を生成"""
        async with httpx.AsyncClient() as client:
            try:
                response = await client.post(
                    f"{self.base_url}/chat/completions",
                    headers={
                        "Authorization": f"Bearer {self.api_key}",
                        "Content-Type": "application/json"
                    },
                    json={
                        "model": "gpt-4",
                        "messages": [{"role": "user", "content": prompt}],
                        "max_tokens": 500
                    },
                    timeout=10.0
                )
                response.raise_for_status()
                return response.json()["choices"][0]["message"]["content"]
            except httpx.HTTPError as e:
                raise ExternalServiceError(
                    "LLMサービスが一時的に利用できません"
                )
```

### セキュリティチェックリスト

#### アプリケーション層(FastAPI)

- [x] CORS 設定(フロントエンドからのアクセス許可)
- [x] 入力バリデーション(Pydantic)
- [x] SQL インジェクション対策(SQLAlchemy パラメータ化クエリ)
- [x] API キー管理(環境変数、.gitignore)
- [x] セキュリティヘッダー(補助的)
- [ ] 認証・認可(一人用のため不要)
- [ ] レート制限(一人用のため不要)

#### インフラ層(Caddy)

- [ ] HTTPS 設定(Caddy で自動設定)
- [ ] セキュリティヘッダー(Caddy で厳格設定)
- [ ] リバースプロキシ(Caddy で設定)

#### データ層(SQLAlchemy)

- [x] パラメータ化クエリ(SQLAlchemy デフォルト)
- [x] トランザクション制御(session 管理)

### 実装原則

1. **多層防御**: ネットワーク・アプリケーション・データの 3 層で防御
2. **最小限実装**: 一人用システムとして必要最小限のセキュリティに絞る
3. **環境変数管理**: API キーはソースコードにハードコードしない
4. **パラメータ化クエリ**: 生 SQL は使用せず、ORM のパラメータ化クエリを使用
5. **バリデーション統合**: Pydantic スキーマで入力バリデーションを自動化
6. **インフラ分離**: 厳格なセキュリティヘッダーは Caddy に委譲

<!-- GLOBAL_CONCLUSION_END: backend-security-specification -->
