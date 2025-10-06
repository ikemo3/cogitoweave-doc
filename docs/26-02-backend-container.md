<!-- markdownlint-disable MD024 -->

# Backend

## バックエンド Container 契約

<!-- REFERENCE_BEGIN: layered-architecture-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、複数機能にまたがる処理の調整、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

<!-- REFERENCE_END: layered-architecture-structure -->

<!-- REFERENCE_BEGIN: api-layer-responsibility -->

### API 層の責務

- FastAPI ルーター定義
- HTTP リクエスト/レスポンス処理
- Pydantic モデルによる入力バリデーション
- HTTP ステータスコードエラーハンドリング

<!-- REFERENCE_END: api-layer-responsibility -->

<!-- REFERENCE_BEGIN: application-layer-responsibility -->

### Application 層の責務

- ユースケース実装
- トランザクション制御
- 複数機能にまたがる処理の調整
- Repository 層を使った外部システム協調

<!-- REFERENCE_END: application-layer-responsibility -->

<!-- REFERENCE_BEGIN: domain-layer-responsibility -->

### Domain 層の責務

- ビジネスエンティティ定義(純粋な Python)
- エンティティ内で閉じるビジネスロジック
- データ構造から自然に導出されるロジック
- ドメインサービスはほぼ不要(DB 制約で整合性管理)

<!-- REFERENCE_END: domain-layer-responsibility -->

<!-- REFERENCE_BEGIN: repository-layer-responsibility -->

### Repository 層の責務

- SQLAlchemy ORM 定義(テーブル構造)
- データベースアクセス
- 外部 API 通信(LLM、記事取得)
- Domain エンティティ ⇔ ORM エンティティ変換
- クエリ実行、トランザクション境界

<!-- REFERENCE_END: repository-layer-responsibility -->

### モジュール構造

<!-- REFERENCE_BEGIN: backend-module-structure -->

バックエンドは機能境界優先のディレクトリ構造とし、境界内部でレイヤーを分割する。

#### ディレクトリ構造

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

#### モジュール構造の原則

- 機能境界で分割し、境界間の独立性を保つ
- 境界内部では密接に関連する機能を統合管理する
- API 提供は core のみ、llm/content は内部サービスとして Repository 層を提供
- shared は責務別に分割し、曖昧な名前(util/common/lib)を避ける
- レイヤー構造を境界内部で実現し、縦横の責務分離を両立する

<!-- REFERENCE_END: backend-module-structure -->

### 依存性注入

<!-- REFERENCE_BEGIN: backend-dependency-injection -->

バックエンドは API 層と Application 層で FastAPI の依存性注入を使用し、コンストラクタ注入により依存を明示的にする。

#### 注入対象レイヤー

- **API 層**: Application サービスを注入
- **Application 層**: Repository を注入
- **Repository 層**: 注入不要(純粋なデータアクセス)
- **Domain 層**: 注入不要(純粋なビジネスロジック)

#### ライフサイクル管理

- **DB セッション**: リクエストスコープ(Depends で注入)
- **Repository**: リクエストスコープ(Depends で注入)
- **Application サービス**: リクエストスコープ(Depends で注入)

#### 実装パターン

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

#### 依存性注入の原則

- コンストラクタ注入により依存を明示的にする
- リクエストスコープでステートレス実装を保つ
- FastAPI の Depends を一貫して使用し、パターンを統一する
- 過剰な抽象化を避け、必要な層のみで注入を使用する
- dependencies.py に依存関係の構築ロジックを集約する

<!-- REFERENCE_END: backend-dependency-injection -->

### テスト構成

<!-- REFERENCE_BEGIN: backend-test-configuration -->

バックエンドはデトロイト派(古典派)のテスト戦略を採用し、原則として DB は実物を使用する。

#### テスト戦略

- モック方針: デトロイト派(古典派、状態検証主義)
- DB: 原則として実物(SQLite インメモリ)を使用
- 外部 API: 必ずモック化(httpx-mock 使用)
- 実際の動作に近いテストを重視
- 柔軟性: 依存を明示的にしているため、特定のケースで DB をモック化することも妨げない

#### テストディレクトリ構造

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

#### 各層のテスト方針

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

#### テスト環境の原則

- tests/ は src/ のミラー構造とし、ファイルを探しやすくする
- 原則として DB は実物を使用し、実際の動作に近いテストを実現する
- 依存を明示的にしているため、必要に応じて DB をモック化することも可能
- SQLite インメモリにより、実 DB を使用しても高速なテストを維持する
- pytest の共通設定・フィクスチャは conftest.py に集約する

<!-- REFERENCE_END: backend-test-configuration -->

### エラーハンドリング

<!-- REFERENCE_BEGIN: backend-error-handling-specification -->

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

<!-- REFERENCE_END: backend-error-handling-specification -->

### バリデーション

<!-- REFERENCE_BEGIN: backend-validation-schemas-specification -->

### 概念スキーマ

```python
# core/api/schemas/concept.py
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

class ConceptCreateRequest(BaseModel):
    """概念作成リクエスト"""
    name: str = Field(..., min_length=1, max_length=100, description="概念名")
    description: str = Field(default="", max_length=1000, description="概念の説明")

    @field_validator('name')
    @classmethod
    def name_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('概念名は空白のみにできません')
        return v.strip()

class ConceptUpdateRequest(BaseModel):
    """概念更新リクエスト"""
    name: str | None = Field(None, min_length=1, max_length=100, description="概念名")
    description: str | None = Field(None, max_length=1000, description="概念の説明")

    @field_validator('name')
    @classmethod
    def name_not_empty(cls, v: str | None) -> str | None:
        if v is not None and not v.strip():
            raise ValueError('概念名は空白のみにできません')
        return v.strip() if v else None

class ConceptResponse(BaseModel):
    """概念レスポンス"""
    id: int
    name: str
    description: str
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True  # ORMモデルからの変換を許可
```

### 文献メモスキーマ

```python
# core/api/schemas/literature_memo.py
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

class LiteratureMemoCreateRequest(BaseModel):
    """文献メモ作成リクエスト"""
    concept_id: int = Field(..., gt=0, description="概念ID")
    literature_id: int = Field(..., gt=0, description="文献ID")
    title: str = Field(..., min_length=1, max_length=500, description="タイトル")
    content: str = Field(..., min_length=1, description="内容")

    @field_validator('title', 'content')
    @classmethod
    def not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('空白のみは許可されません')
        return v.strip()

class LiteratureMemoUpdateRequest(BaseModel):
    """文献メモ更新リクエスト"""
    title: str | None = Field(None, min_length=1, max_length=500, description="タイトル")
    content: str | None = Field(None, min_length=1, description="内容")

    @field_validator('title', 'content')
    @classmethod
    def not_empty(cls, v: str | None) -> str | None:
        if v is not None and not v.strip():
            raise ValueError('空白のみは許可されません')
        return v.strip() if v else None

class LiteratureMemoResponse(BaseModel):
    """文献メモレスポンス"""
    id: int
    concept_id: int
    literature_id: int
    title: str
    content: str
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True
```

### 関係性スキーマ

```python
# core/api/schemas/relationship.py
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

class RelationshipCreateRequest(BaseModel):
    """関係性作成リクエスト"""
    concept_a_id: int = Field(..., gt=0, description="概念AのID")
    concept_b_id: int = Field(..., gt=0, description="概念BのID")
    relationship_name: str = Field(..., min_length=1, max_length=100, description="関係名")
    description: str = Field(default="", max_length=500, description="関係の説明")

    @field_validator('relationship_name')
    @classmethod
    def name_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('関係名は空白のみにできません')
        return v.strip()

    @field_validator('concept_b_id')
    @classmethod
    def concepts_different(cls, v: int, info) -> int:
        if 'concept_a_id' in info.data and v == info.data['concept_a_id']:
            raise ValueError('同一概念の関係は作成できません')
        return v

class RelationshipUpdateRequest(BaseModel):
    """関係性更新リクエスト"""
    relationship_name: str | None = Field(None, min_length=1, max_length=100, description="関係名")
    description: str | None = Field(None, max_length=500, description="関係の説明")

    @field_validator('relationship_name')
    @classmethod
    def name_not_empty(cls, v: str | None) -> str | None:
        if v is not None and not v.strip():
            raise ValueError('関係名は空白のみにできません')
        return v.strip() if v else None

class RelationshipResponse(BaseModel):
    """関係性レスポンス"""
    id: int
    concept_a_id: int
    concept_b_id: int
    relationship_name: str
    description: str
    created_at: datetime

    class Config:
        from_attributes = True
```

### 文献スキーマ

```python
# core/api/schemas/literature.py
from pydantic import BaseModel, Field, field_validator, HttpUrl
from typing import Literal

class WebArticleCreateRequest(BaseModel):
    """Web記事作成リクエスト"""
    url: HttpUrl = Field(..., description="記事URL")
    title: str = Field(..., min_length=1, max_length=500, description="タイトル")

    @field_validator('title')
    @classmethod
    def title_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('タイトルは空白のみにできません')
        return v.strip()

class BookCreateRequest(BaseModel):
    """書籍作成リクエスト"""
    isbn: str = Field(..., min_length=1, max_length=20, description="ISBN")
    author: str = Field(..., min_length=1, max_length=200, description="著者名")
    publisher: str = Field(..., min_length=1, max_length=200, description="出版社")
    publication_year: int = Field(..., gt=0, description="出版年")

    @field_validator('isbn', 'author', 'publisher')
    @classmethod
    def not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('空白のみは許可されません')
        return v.strip()

    @field_validator('publication_year')
    @classmethod
    def year_valid(cls, v: int) -> int:
        if v < 1000 or v > 9999:
            raise ValueError('出版年は4桁の西暦で入力してください')
        return v

class WebArticleResponse(BaseModel):
    """Web記事レスポンス"""
    id: int
    kind: Literal['web']
    url: str
    title: str

    class Config:
        from_attributes = True

class BookResponse(BaseModel):
    """書籍レスポンス"""
    id: int
    kind: Literal['book']
    isbn: str
    author: str
    publisher: str
    publication_year: int

    class Config:
        from_attributes = True
```

### 共通エラーレスポンス

```python
# shared/schemas/error.py
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    """エラーレスポンス"""
    error: str = Field(..., description="エラーメッセージ")
    details: list[str] = Field(default_factory=list, description="詳細メッセージリスト")

    class Config:
        json_schema_extra = {
            "example": {
                "error": "バリデーションエラー",
                "details": ["概念名は100文字以内で入力してください"]
            }
        }
```

### FastAPI での使用例

```python
# core/api/concept.py
from fastapi import APIRouter, Depends, status
from core.api.schemas.concept import (
    ConceptCreateRequest,
    ConceptUpdateRequest,
    ConceptResponse
)
from shared.schemas.error import ErrorResponse

router = APIRouter(prefix="/api/concepts", tags=["concepts"])

@router.post(
    "/",
    response_model=ConceptResponse,
    status_code=status.HTTP_201_CREATED,
    responses={
        400: {"model": ErrorResponse},
        409: {"model": ErrorResponse},
        500: {"model": ErrorResponse}
    }
)
async def create_concept(
    request: ConceptCreateRequest,
    service: ConceptService = Depends(get_concept_service)
):
    """概念を作成する"""
    return service.create_concept(
        name=request.name,
        description=request.description
    )
```

### バリデーションエラーのカスタマイズ

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Pydanticバリデーションエラーのハンドラー"""
    details = []
    for error in exc.errors():
        field = ' -> '.join(str(loc) for loc in error['loc'][1:])
        message = error['msg']
        details.append(f"{field}: {message}")

    return JSONResponse(
        status_code=400,
        content={
            "error": "入力データが不正です",
            "details": details
        }
    )
```

### 実装原則

1. **3 層バリデーション**: 型・範囲・形式チェックの階層的実装
2. **リクエスト/レスポンス分離**: Create/Update/Response 用の専用スキーマ
3. **from_attributes 有効化**: ORM モデルからの自動変換
4. **空白チェック**: `@field_validator`で空白のみの入力を拒否
5. **OpenAPI 統合**: FastAPI の自動ドキュメント生成と連携
6. **明確なエラーメッセージ**: ユーザーフレンドリーな日本語メッセージ

<!-- REFERENCE_END: backend-validation-schemas-specification -->

### セキュリティ

<!-- REFERENCE_BEGIN: backend-security-specification -->

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

<!-- REFERENCE_END: backend-security-specification -->

### パフォーマンス

<!-- REFERENCE_BEGIN: backend-performance-specification -->

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

<!-- REFERENCE_END: backend-performance-specification -->
