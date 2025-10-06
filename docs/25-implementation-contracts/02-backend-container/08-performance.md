---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "request-processing-design"
    - "scaling-strategy"
  produces:
    - "backend-performance-specification"
---

# パフォーマンス仕様

## 概要

バックエンドのパフォーマンス最適化の具体的な実装方針を定義する。N+1 問題対策、インデックス最適化、worker プロセス数調整、タイムアウト設定を統一的に実装し、パフォーマンスバジェットを満たす応答性能を実現する。

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

<!-- PREMISE_BEGIN: request-processing-design -->

- **同期処理ベース**: 基本的に同期処理でシンプルな処理フローを実現し、複雑な業務処理は 1 トランザクション内で完結させる。
- **ログレベル設定**: 開発者対応の緊急度に応じたログレベル（ERROR=即座対応、WARN=計画的改善、INFO=動作確認、DEBUG=開発詳細）を採用し、ユーザーには理解しやすいエラーメッセージのみを提供する。
- **トランザクション分離**: データベース操作のみを対象とし、外部 API 呼び出しは分離する。
- **リクエスト検証**: 型チェック、範囲チェック、形式チェックを含む定型的な検証を実施し、ビジネスロジック検証は別レイヤーで処理する。
- **統一タイムアウト**: 全 API に 10 秒の統一タイムアウトを設定し、外部 API エラー時はフロントエンド側リトライで対応する。
- **性能最適化**: N+1 問題対策とクエリ最適化により性能要件を満たす。

<!-- PREMISE_END: request-processing-design -->

<!-- PREMISE_BEGIN: scaling-strategy -->

- **単一サーバー構成**: 水平スケーリングの複雑さを回避し、必要に応じた垂直スケーリングにより性能向上を図る。
- **数万件規模への対応**: 概念・文献メモ・関係性データを含めて数万件規模まで対応し、現代の一般的なサーバーで十分な性能を実現する。
- **シンプル運用を優先**: 複雑なスケーリング機構や最適化は実装せず、理解しやすく保守しやすい構成を維持する。

<!-- PREMISE_END: scaling-strategy -->

## 論理

### N+1 問題対策の論理

ORM を使用する際、関連データを取得するループで個別にクエリが発行される N+1 問題が発生しやすい。

**問題例**:

```python
# ❌ N+1問題: 文献メモ100件取得時に101回クエリが発行される
memos = db.query(LiteratureMemo).all()  # 1回目
for memo in memos:
    concept = memo.concept  # 100回(N回)
```

これを解決するため、SQLAlchemy の`joinedload`で eager loading を実装する。

### インデックス戦略の論理

データベーススキーマで定義されたインデックスを活用し、検索性能を最適化する必要がある。

**インデックス活用方針**:

- 外部キー制約カラムは自動インデックス作成済み
- 検索 API で使用するカラム(name, description)はインデックス作成済み
- ORM クエリでインデックスを活用する実装を行う

### worker プロセス数の論理

単一サーバー構成で最適な worker プロセス数を決定する必要がある。

**worker プロセス数の決定**:

- 推奨値: `(CPU コア数 * 2) + 1`
- 一人用システムなので過剰な worker 不要
- 開発環境: 1 worker(デバッグしやすさ優先)
- 本番環境: 2-4 workers(応答性能優先)

### タイムアウト設定の論理

統一タイムアウト 10 秒を実現するため、uvicorn と httpx で一貫したタイムアウト設定が必要である。

**タイムアウト階層**:

- uvicorn: リクエスト全体のタイムアウト
- httpx: 外部 API 呼び出しのタイムアウト
- SQLite: クエリタイムアウト(通常は不要)

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-performance-specification -->

### N+1 問題対策

```python
# core/repository/literature_memo_repository.py
from sqlalchemy.orm import Session, joinedload
from core.repository.models import LiteratureMemoModel

class LiteratureMemoRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_by_concept_id(self, concept_id: int) -> list[LiteratureMemoModel]:
        """概念IDで文献メモ一覧を取得(eager loading)"""
        # ✅ N+1問題対策: joinedloadで関連データを一度に取得
        return self.db.query(LiteratureMemoModel).options(
            joinedload(LiteratureMemoModel.concept),
            joinedload(LiteratureMemoModel.literature)
        ).filter(
            LiteratureMemoModel.concept_id == concept_id
        ).all()

    def get_all(self) -> list[LiteratureMemoModel]:
        """全文献メモを取得(eager loading)"""
        return self.db.query(LiteratureMemoModel).options(
            joinedload(LiteratureMemoModel.concept),
            joinedload(LiteratureMemoModel.literature)
        ).all()
```

### インデックス活用

```python
# core/repository/concept_repository.py
from sqlalchemy.orm import Session
from core.repository.models import ConceptModel

class ConceptRepository:
    def __init__(self, db: Session):
        self.db = db

    def search_by_name_or_description(self, query: str) -> list[ConceptModel]:
        """名前・説明で部分一致検索(インデックス活用)"""
        # ✅ インデックス活用: idx_concept_name, idx_concept_descriptionを使用
        return self.db.query(ConceptModel).filter(
            (ConceptModel.name.like(f"%{query}%")) |
            (ConceptModel.description.like(f"%{query}%"))
        ).all()
```

### worker プロセス数設定

```python
# shared/config.py
from pydantic_settings import BaseSettings
import multiprocessing

class Settings(BaseSettings):
    """環境変数設定"""

    # データベース設定
    database_url: str = "sqlite:///./cogitoweave.db"

    # uvicorn設定
    uvicorn_host: str = "0.0.0.0"
    uvicorn_port: int = 8000
    uvicorn_workers: int = 1  # 開発環境デフォルト

    # 本番環境用workerプロセス数計算
    @property
    def production_workers(self) -> int:
        """本番環境用workerプロセス数: (CPU コア数 * 2) + 1"""
        return (multiprocessing.cpu_count() * 2) + 1

    class Config:
        env_file = ".env"

settings = Settings()
```

起動スクリプト:

```bash
#!/bin/bash
# scripts/start_production.sh

# 本番環境での起動
uvicorn main:app \
  --host 0.0.0.0 \
  --port 8000 \
  --workers 4 \
  --timeout-keep-alive 10 \
  --access-log \
  --log-level info
```

開発環境での起動:

```bash
#!/bin/bash
# scripts/start_dev.sh

# 開発環境での起動(ホットリロード有効)
uvicorn main:app \
  --host 127.0.0.1 \
  --port 8000 \
  --reload \
  --log-level debug
```

### タイムアウト設定

```python
# main.py
from fastapi import FastAPI, Request
from contextlib import asynccontextmanager
import time

app = FastAPI()

@app.middleware("http")
async def timeout_middleware(request: Request, call_next):
    """統一タイムアウトミドルウェア: 10秒"""
    start_time = time.time()

    try:
        response = await asyncio.wait_for(
            call_next(request),
            timeout=10.0
        )
        return response
    except asyncio.TimeoutError:
        process_time = time.time() - start_time
        logger.error(f"Request timeout: {request.url.path} ({process_time:.2f}s)")
        return JSONResponse(
            status_code=504,
            content={"error": "リクエストがタイムアウトしました"}
        )
```

外部 API 呼び出しのタイムアウト:

```python
# llm/repository/llm_client.py
import httpx

class LLMClient:
    async def generate_summary(self, prompt: str) -> str:
        """LLM APIで要約を生成(10秒タイムアウト)"""
        async with httpx.AsyncClient(timeout=10.0) as client:
            response = await client.post(
                f"{self.base_url}/chat/completions",
                headers={"Authorization": f"Bearer {self.api_key}"},
                json={"model": "gpt-4", "messages": [...]},
                # タイムアウト: 10秒
            )
            return response.json()["choices"][0]["message"]["content"]
```

### データベース接続プール

```python
# shared/database/session.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from shared.config import settings

# 接続プール設定
engine = create_engine(
    settings.database_url,
    connect_args={
        "check_same_thread": False,  # SQLiteのマルチスレッド対応
        "timeout": 10.0,  # クエリタイムアウト: 10秒
    },
    pool_size=5,  # 最大接続数
    max_overflow=10,  # プール外の最大接続数
    pool_pre_ping=True,  # 接続の有効性チェック
    echo=False,  # SQLログ出力(開発環境ではTrue)
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

### パフォーマンス監視

```python
# main.py
from fastapi import FastAPI, Request
import time
from loguru import logger

app = FastAPI()

@app.middleware("http")
async def performance_logging_middleware(request: Request, call_next):
    """パフォーマンスログミドルウェア"""
    start_time = time.time()

    response = await call_next(request)

    process_time = time.time() - start_time

    # 1秒以上かかったリクエストを警告
    if process_time > 1.0:
        logger.warning(
            f"Slow request: {request.method} {request.url.path} "
            f"({process_time:.2f}s)"
        )
    else:
        logger.info(
            f"{request.method} {request.url.path} "
            f"({process_time:.3f}s)"
        )

    # レスポンスヘッダーに処理時間を追加
    response.headers["X-Process-Time"] = f"{process_time:.3f}"

    return response
```

### クエリ最適化チェックリスト

#### N+1 問題対策

- [x] `joinedload`で eager loading 実装
- [x] 関連データ取得時は常に joinedload 使用
- [x] `selectinload`は必要に応じて使用(大量データ時)

#### インデックス活用

- [x] 外部キー制約カラムは自動インデックス
- [x] 検索用カラムは明示的インデックス作成済み
- [x] LIKE 検索でもインデックス活用

#### クエリ最適化

- [x] 必要なカラムのみ取得(`load_only`使用可能)
- [x] 大量データ取得時はページネーション実装(優先度低)
- [x] 集計クエリは最小限

### パフォーマンスバジェット

| 項目                 | 目標値     | 測定方法                                  |
| -------------------- | ---------- | ----------------------------------------- |
| システム初回起動     | 2 秒以内   | uvicorn 起動からヘルスチェックまで        |
| 概念一覧取得 API     | 500ms 以内 | `/api/concepts` GET                       |
| 文献メモ一覧取得 API | 500ms 以内 | `/api/concepts/{id}/literature-memos` GET |
| LLM 要約生成 API     | 10 秒以内  | `/api/llm/summarize` POST                 |

### 実装原則

1. **N+1 問題対策**: `joinedload`で関連データを eager loading
2. **インデックス活用**: 検索クエリでインデックスを必ず使用
3. **統一タイムアウト**: 全 API 10 秒タイムアウト
4. **適切な worker 数**: 開発環境 1 worker、本番環境 2-4 workers
5. **パフォーマンスログ**: 1 秒以上のリクエストを警告ログ出力
6. **接続プール**: SQLite でも接続プールを適切に設定

<!-- GLOBAL_CONCLUSION_END: backend-performance-specification -->
