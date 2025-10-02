<!-- markdownlint-disable MD024 -->

# Container レベル実装契約集

## 概要

C4 モデルの Container レベルで定義された実装契約を統合する。フロントエンド Container、バックエンド Container、データベース Container の 3 つの実行可能単位における設計判断を集約し、実装の全体像を提供する。

## フロントエンド Container 契約

### パッケージ構造

<!-- REFERENCE_BEGIN: frontend-package-structure -->

フロントエンドは単一 package.json によるモノリシック構成とし、機能別ディレクトリ構造により責務を分離する。

#### ディレクトリ構造

```plaintext
src/
├── features/
│   ├── concept/          # 概念管理機能
│   ├── literature-memo/  # 文献メモ管理機能
│   ├── relationship/     # 関係性管理機能
│   └── question/         # 質問探索機能
├── pages/                # 画面の組み立て
├── shared/
│   ├── components/       # 共通UIコンポーネント
│   └── api/              # API通信基盤
└── app/                  # アプリケーション全体の初期化・設定
```

#### 設計原則

- 機能ごとに凝集度を高め、機能追加・削除を容易にする
- 機能ディレクトリ内部でのレイヤー別分割を妨げない
- 画面は複数機能を組み合わせる場所として pages/ に独立配置する
- 横断的な関心事は責務が明確な名前で shared/ に配置する
- util/, common/, lib/ のような曖昧な名前は使用しない

<!-- REFERENCE_END: frontend-package-structure -->

### ルーター構成

<!-- REFERENCE_BEGIN: frontend-router-configuration -->

TanStack Router のファイルベースルーティングを採用し、ルート単位で遅延ロードを行う。

#### ルート定義配置

```plaintext
src/
├── routes/
│   ├── __root.tsx       # ルートレイアウト
│   ├── index.tsx        # メイン画面 (/)
│   ├── question.tsx     # 質問探索画面 (/question)
│   └── spatial.tsx      # 空間配置画面 (/spatial)
└── app/
    └── router.ts        # Router インスタンス生成
```

#### 設計原則

- ファイル構造とルートを一致させ、直感的な理解を実現する
- ルート単位で遅延ロードし、初期バンドルサイズを最小化する
- TanStack Router の規約に従い、設定を最小限に保つ
- ルーティングエラーは `__root.tsx` のエラー境界で捕捉する

<!-- REFERENCE_END: frontend-router-configuration -->

### テスト構成

<!-- REFERENCE_BEGIN: frontend-test-configuration -->

DOM 環境シミュレーションに happy-dom を採用し、ユニットテストはコロケーション、E2E テストは別フォルダ管理とする。

#### テスト環境

- DOM 環境シミュレーション: happy-dom
- ユニットテストフレームワーク: Vitest + Testing Library
- E2E テストフレームワーク: Playwright
- API モック: fetch のモック化(具体的手法は実装時に決定)

#### テストファイル配置

```plaintext
src/
├── features/
│   └── concept/
│       ├── ConceptList.tsx
│       └── ConceptList.test.tsx    # ユニットテスト(コロケーション)
├── pages/
│   ├── MainPage.tsx
│   └── MainPage.test.tsx           # ユニットテスト(コロケーション)
└── shared/
    └── components/
        ├── Button.tsx
        └── Button.test.tsx         # ユニットテスト(コロケーション)

tests/
└── e2e/
    ├── main-flow.spec.ts           # E2E テスト(別フォルダ)
    └── question-flow.spec.ts
```

#### 設計原則

- 軽量な DOM シミュレーションでテスト実行速度を最適化する
- ユニットテストはコード近傍に配置し、関連性を明確化する
- E2E テストは画面シナリオとして独立管理する
- API 通信はモック化し、外部依存を排除する

<!-- REFERENCE_END: frontend-test-configuration -->

## バックエンド Container 契約

### レイヤー構造

<!-- REFERENCE_BEGIN: backend-layer-structure -->

バックエンドは 4 層構成のレイヤードアーキテクチャを採用する。

#### レイヤー構成

- **API 層**: FastAPI エンドポイント、リクエスト/レスポンスの Pydantic モデル
- **Application 層**: ユースケース実行、トランザクション制御
- **Domain 層**: エンティティ(純粋な Python)、エンティティ内で閉じるビジネスロジック
- **Repository 層**: SQLAlchemy ORM、データアクセス、外部 API 通信、Domain ⇔ ORM 変換

#### 各層の責務

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

#### 設計原則

- 薄いドメイン層アプローチ: DOA とデータベース制約により、意図的に Domain 層を薄く保つ
- Active Record パターンを避ける: ビジネスロジックと DB 技術を分離
- 基本 1:1 対応: Domain エンティティと ORM エンティティは構造を揃え、変換コストを最小化
- Repository で吸収しない: ドメインロジックは Domain 層に配置し、Repository は変換のみ担当

<!-- REFERENCE_END: backend-layer-structure -->

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

## データベース Container 契約

### Concepts テーブルスキーマ

<!-- REFERENCE_BEGIN: concept-table-schema -->

```sql
-- ============================================
-- Concept (概念) テーブル
-- ============================================
CREATE TABLE concept (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(1000) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_concept_name ON concept(name);
CREATE INDEX idx_concept_description ON concept(description);
```

<!-- REFERENCE_END: concept-table-schema -->

### Literatures テーブルスキーマ

<!-- REFERENCE_BEGIN: literature-table-schema -->

```sql
-- ============================================
-- Literature (文献) テーブル
-- ============================================
CREATE TABLE literature (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    url VARCHAR(2048) NOT NULL,
    title VARCHAR(500) NOT NULL,
    fetched_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_literature_url ON literature(url);
```

<!-- REFERENCE_END: literature-table-schema -->

### Literature Memos テーブルスキーマ

<!-- REFERENCE_BEGIN: literature-memo-table-schema -->

```sql
-- ============================================
-- LiteratureMemo (文献メモ) テーブル
-- ============================================
CREATE TABLE literature_memo (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_id INTEGER NOT NULL,
    literature_id INTEGER NOT NULL,
    title VARCHAR(500) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_literature_memo_concept_id ON literature_memo(concept_id);
CREATE INDEX idx_literature_memo_literature_id ON literature_memo(literature_id);
```

<!-- REFERENCE_END: literature-memo-table-schema -->

### Relationships テーブルスキーマ

<!-- REFERENCE_BEGIN: relationship-table-schema -->

```sql
-- ============================================
-- Relationship (関係性) テーブル
-- ============================================
CREATE TABLE relationship (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_a_id INTEGER NOT NULL,
    concept_b_id INTEGER NOT NULL,
    relationship_name VARCHAR(100) NOT NULL,
    description VARCHAR(500) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_a_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (concept_b_id) REFERENCES concept(id) ON DELETE RESTRICT,
    CHECK (concept_a_id < concept_b_id),
    UNIQUE (concept_a_id, concept_b_id, relationship_name)
);

CREATE INDEX idx_relationship_concept_a_id ON relationship(concept_a_id);
CREATE INDEX idx_relationship_concept_b_id ON relationship(concept_b_id);
```

<!-- REFERENCE_END: relationship-table-schema -->

### 実装契約詳細

データベース Container の詳細な実装契約は以下で定義する:

- [スキーマ設計](25-implementation-contracts/03-database-container/README.md)

## 結論

Container レベルの実装契約として、フロントエンド・バックエンド・データベースの 3 つの実行可能単位における設計判断を統合した。各 Container の技術スタック、アーキテクチャ方針、実装制約が明確化され、実装フェーズへの移行が可能となる。
