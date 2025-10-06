---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "backend-container-integrated-constraints"
    - "concept-create-api"
  produces:
    - "backend-validation-schemas-specification"
---

# バリデーション仕様

## 概要

バックエンドの Pydantic バリデーションスキーマの具体的な実装方針を定義する。API 契約で定義された各エンドポイントのリクエスト/レスポンススキーマを Pydantic モデルとして具体化し、型安全で保守しやすいバリデーションを実現する。

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

<!-- PREMISE_BEGIN: concept-create-api -->

#### エンドポイント

- パス: `/api/concepts`
- メソッド: POST

#### リクエストスキーマ

- name
  - 型: string
  - 必須/任意: 必須
  - 説明: 概念名
  - バリデーション: 1-100 文字、重複不可
- description
  - 型: string
  - 必須/任意: 任意
  - 説明: 概念の説明
  - バリデーション: 最大 1000 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された概念の ID
- name
  - 型: string
  - 説明: 概念名
- description
  - 型: string
  - 説明: 概念の説明(値がない場合は空文字列)
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)
- updated_at
  - 型: string
  - 説明: 更新日時(ISO 8601 形式)

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(文字数超過、必須フィールド欠如)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 409 Conflict
  - 原因: 概念名の重複
  - レスポンス:
    - error
      - 型: string
      - 説明: "概念名が既に存在します"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- PREMISE_END: concept-create-api -->

## 論理

### Pydantic モデル設計の論理

API 契約で定義された各エンドポイントのバリデーション要件を、Pydantic モデルとして実装する必要がある。

**設計方針**:

1. **リクエスト/レスポンスの分離**: リクエストスキーマとレスポンススキーマを別モデルで定義
2. **共通フィールドの再利用**: BaseModel で DRY 原則を実現
3. **バリデーターの活用**: カスタムバリデーションロジックを`@field_validator`で実装
4. **OpenAPI 統合**: FastAPI の OpenAPI 自動生成と連携

### バリデーション戦略の論理

入力バリデーション(型・範囲・形式チェック)を実現するため、3 層のバリデーションを実装する。

**3 層バリデーション**:

1. **型チェック**: Pydantic の型アノテーションによる自動チェック
2. **範囲チェック**: `Field()`で文字列長、数値範囲等を制約
3. **形式チェック**: `@field_validator`で URL 形式、ISBN 形式等をチェック

この 3 層により、エラーハンドリング仕様で定義した ValidationError を適切に発生させる。

### スキーマ構成の論理

各エンティティに対して、CRUD 操作に応じたスキーマを定義する。

**スキーマ種別**:

- **CreateSchema**: 作成時のリクエストスキーマ(ID なし)
- **UpdateSchema**: 更新時のリクエストスキーマ(全フィールド任意)
- **ResponseSchema**: レスポンススキーマ(ID あり、タイムスタンプあり)

この分離により、各操作の要件を明確にし、意図しないフィールド更新を防止する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: backend-validation-schemas-specification -->

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

<!-- GLOBAL_CONCLUSION_END: backend-validation-schemas-specification -->
