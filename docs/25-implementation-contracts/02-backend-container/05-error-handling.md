---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "request-processing-design"
  produces:
    - "backend-error-handling-specification"
---

# エラーハンドリング仕様

## 概要

バックエンドのエラーハンドリングの具体的な実装方針を定義する。統一的な例外クラス設計、HTTP ステータスコードマッピング、エラーレスポンス形式の標準化により、保守しやすく理解しやすいエラーハンドリングを実現する。

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

## 論理

### 例外クラス設計の論理

階層的ログレベル管理と緊急度分類を実現するには、例外の種類を明確に分類する必要がある。

**例外の分類**:

- **ビジネスルール違反**(400 系): ユーザー入力エラー、バリデーションエラー、重複エラー等
- **リソース不在**(404): エンティティが存在しない
- **一時的障害**(500 系): データベース接続エラー、外部 API 障害等
- **予期しないエラー**(500): バグ、想定外の状態等

各分類に対応する例外クラスを定義し、HTTP ステータスコードとログレベルを自動的にマッピングする。

### エラーレスポンス形式の論理

フロントエンド側でエラーを適切に表示するため、統一的なエラーレスポンス形式が必要である。

**必要な情報**:

- エラーメッセージ(ユーザー向け)
- 詳細情報(開発者向け、任意)
- HTTP ステータスコード(自動設定)

この形式により、フロントエンドは一貫した方法でエラーを処理できる。

### ログ出力の論理

階層的ログレベル管理により、開発者は緊急度に応じた対応を行える。

**ログレベルとマッピング**:

- ERROR: 500 系エラー(即座対応が必要)
- WARN: 4xx 系エラーの一部(計画的改善が必要)
- INFO: 正常系の動作確認
- DEBUG: 開発時の詳細情報

例外クラスから自動的に適切なログレベルを選択する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-error-handling-specification -->

### 例外クラス階層

```python
# shared/exceptions.py
from fastapi import HTTPException

class AppException(Exception):
    """アプリケーション例外の基底クラス"""
    def __init__(
        self,
        message: str,
        status_code: int,
        details: list[str] | None = None
    ):
        self.message = message
        self.status_code = status_code
        self.details = details or []
        super().__init__(self.message)

class ValidationError(AppException):
    """バリデーションエラー (400 Bad Request)"""
    def __init__(self, message: str, details: list[str] | None = None):
        super().__init__(message, 400, details)

class ConflictError(AppException):
    """リソース競合エラー (409 Conflict)"""
    def __init__(self, message: str):
        super().__init__(message, 409)

class NotFoundError(AppException):
    """リソース不在エラー (404 Not Found)"""
    def __init__(self, message: str):
        super().__init__(message, 404)

class ExternalServiceError(AppException):
    """外部サービスエラー (503 Service Unavailable)"""
    def __init__(self, message: str):
        super().__init__(message, 503)

class InternalServerError(AppException):
    """内部サーバーエラー (500 Internal Server Error)"""
    def __init__(self, message: str, details: list[str] | None = None):
        super().__init__(message, 500, details)
```

### エラーレスポンス形式

```python
# shared/schemas/error.py
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    error: str
    details: list[str] = []

    class Config:
        json_schema_extra = {
            "example": {
                "error": "概念名が既に存在します",
                "details": []
            }
        }
```

### FastAPI グローバル例外ハンドラー

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from loguru import logger
from shared.exceptions import AppException

app = FastAPI()

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    """アプリケーション例外のグローバルハンドラー"""
    # ログレベルの決定
    if exc.status_code >= 500:
        logger.error(f"{exc.message} | details={exc.details}")
    elif exc.status_code >= 400:
        logger.warning(f"{exc.message} | details={exc.details}")

    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": exc.message,
            "details": exc.details
        }
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    """予期しない例外のグローバルハンドラー"""
    logger.error(f"Unexpected error: {exc}", exc_info=True)

    return JSONResponse(
        status_code=500,
        content={
            "error": "内部サーバーエラーが発生しました",
            "details": []
        }
    )
```

### 使用例

```python
# core/application/concept_service.py
from shared.exceptions import ValidationError, ConflictError, NotFoundError

class ConceptService:
    def create_concept(self, name: str, description: str) -> Concept:
        # バリデーション
        if len(name) > 100:
            raise ValidationError(
                "概念名が長すぎます",
                details=["概念名は100文字以内で入力してください"]
            )

        # 重複チェック
        existing = self.concept_repo.get_by_name(name)
        if existing:
            raise ConflictError("概念名が既に存在します")

        return self.concept_repo.create(Concept(name=name, description=description))

    def get_concept(self, concept_id: int) -> Concept:
        concept = self.concept_repo.get_by_id(concept_id)
        if not concept:
            raise NotFoundError("概念が見つかりません")

        return concept
```

### トランザクションエラーハンドリング

```python
# shared/database/session.py
from contextlib import contextmanager
from sqlalchemy.orm import Session
from loguru import logger
from shared.exceptions import InternalServerError

@contextmanager
def transaction_context(db: Session):
    """トランザクションコンテキストマネージャー"""
    try:
        yield db
        db.commit()
    except Exception as e:
        db.rollback()
        logger.error(f"Transaction failed: {e}", exc_info=True)
        raise InternalServerError(
            "データベース処理中にエラーが発生しました",
            details=[str(e)]
        )
    finally:
        db.close()
```

### ログ出力マッピング

| 例外クラス            | HTTP ステータス | ログレベル | 緊急度     |
| --------------------- | --------------- | ---------- | ---------- |
| ValidationError       | 400             | WARN       | 計画的改善 |
| NotFoundError         | 404             | WARN       | 計画的改善 |
| ConflictError         | 409             | WARN       | 計画的改善 |
| ExternalServiceError  | 503             | ERROR      | 即座対応   |
| InternalServerError   | 500             | ERROR      | 即座対応   |
| Exception(予期しない) | 500             | ERROR      | 即座対応   |

### 実装原則

1. **統一的な例外クラス**: AppException を基底とした階層構造
2. **自動 HTTP マッピング**: 例外クラスから HTTP ステータスコードを自動設定
3. **グローバルハンドラー**: FastAPI の exception_handler で一括処理
4. **階層的ログ**: 緊急度に応じたログレベル自動選択
5. **ユーザーフレンドリー**: エラーメッセージは平易な日本語
6. **開発者支援**: details フィールドで技術的詳細を提供

<!-- GLOBAL_CONCLUSION_END: backend-error-handling-specification -->
