---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "literature-entity"
    - "literature-table-schema"
    - "web-article-table-schema"
    - "book-table-schema"
    - "distributed-communication-design"
  produces:
    - "web-article-create-api"
    - "book-create-api"
    - "literature-get-single-api"
---

<!-- markdownlint-disable MD024 -->

# 文献 API 仕様

## 前提

<!-- PREMISE_BEGIN: api-contract -->

CogitoWeave の API 契約は以下の 9 つの論理的 API 群で構成される:

### エンティティ操作 API

1. **概念 API**: 作成・読み取り・更新 (削除は優先度低)
2. **文献メモ API**: 作成・読み取り・更新・概念別一覧取得
3. **関係性 API**: 作成・読み取り・更新 (概念間の重複関係許可)
4. **文献 API**: 作成・読み取り

### 検索・発見 API

1. **概念検索 API**: 名前・説明での部分一致検索
2. **質問から概念候補提示 API**: 既存個人概念体系からの検索・発見支援

### 協働・処理 API

1. **LLM 要約生成 API**: 記事内容と概念 ID から観点別要約を生成
2. **記事取得 API**: URL からタイトル・内容を取得
3. **バックアップ API**: Webhook 化による自動実行支援

これらの API により、概念中心の知識体系構築と効率的な知識活用が実現される。

<!-- PREMISE_END: api-contract -->

<!-- PREMISE_BEGIN: literature-entity -->

文献エンティティ(Literature):

- 外部文献の書誌情報管理
- 基本属性: ID、URL、タイトル(Web 記事の場合)、ISBN、著者、出版社、出版年(書籍の場合)
- 複数の概念観点から文献メモが生成される元情報
- 同一 URL/ISBN であれば同一文献として識別される

<!-- PREMISE_END: literature-entity -->

<!-- PREMISE_BEGIN: literature-table-schema -->

```sql
-- ============================================
-- Literature (文献) テーブル - 基底
-- ============================================
CREATE TABLE literature (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    kind VARCHAR(10) NOT NULL CHECK (kind IN ('web', 'book'))
);
```

<!-- PREMISE_END: literature-table-schema -->

<!-- PREMISE_BEGIN: web-article-table-schema -->

```sql
-- ============================================
-- WebArticle (Web記事) テーブル
-- ============================================
CREATE TABLE web_article (
    literature_id INTEGER PRIMARY KEY,
    url VARCHAR(2048) NOT NULL CHECK(url != ''),
    title VARCHAR(500) NOT NULL CHECK(title != ''),
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_web_article_url ON web_article(url);
```

<!-- PREMISE_END: web-article-table-schema -->

<!-- PREMISE_BEGIN: book-table-schema -->

```sql
-- ============================================
-- Book (書籍) テーブル
-- ============================================
CREATE TABLE book (
    literature_id INTEGER PRIMARY KEY,
    isbn VARCHAR(20) NOT NULL UNIQUE CHECK(isbn != ''),
    author VARCHAR(200) NOT NULL CHECK(author != ''),
    publisher VARCHAR(200) NOT NULL CHECK(publisher != ''),
    publication_year INTEGER NOT NULL,
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_book_isbn ON book(isbn);
```

<!-- PREMISE_END: book-table-schema -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

文献の作成、読み取りを提供する。文献は外部文献の書誌情報を管理し、複数の概念観点から文献メモが生成される元情報として機能する。

文献種別(Web 記事、書籍)ごとに異なるエンドポイントを提供し、型安全で明確な API 設計を実現する。読み取り API では Union 型レスポンスにより kind フィールドで文献種別を判別可能とする。

更新・削除は不要とし、シンプルな書誌情報管理を実現する。

## API 一覧

### 1. Web 記事作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: web-article-create-api -->

#### エンドポイント

- パス: `/api/literatures/web-articles`
- メソッド: POST

#### リクエストスキーマ

- url
  - 型: string
  - 必須/任意: 必須
  - 説明: Web 記事の URL
  - バリデーション: 1-2048 文字、URL 形式
- title
  - 型: string
  - 必須/任意: 必須
  - 説明: Web 記事のタイトル
  - バリデーション: 1-500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された文献の ID
- kind
  - 型: string
  - 説明: 文献種別('web')
- url
  - 型: string
  - 説明: Web 記事の URL
- title
  - 型: string
  - 説明: Web 記事のタイトル

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(文字数超過、必須フィールド欠如、URL 形式不正)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: web-article-create-api -->

### 2. 書籍作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: book-create-api -->

#### エンドポイント

- パス: `/api/literatures/books`
- メソッド: POST

#### リクエストスキーマ

- isbn
  - 型: string
  - 必須/任意: 必須
  - 説明: 書籍の ISBN
  - バリデーション: 1-20 文字
- author
  - 型: string
  - 必須/任意: 必須
  - 説明: 著者名
  - バリデーション: 1-200 文字
- publisher
  - 型: string
  - 必須/任意: 必須
  - 説明: 出版社
  - バリデーション: 1-200 文字
- publication_year
  - 型: integer
  - 必須/任意: 必須
  - 説明: 出版年
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された文献の ID
- kind
  - 型: string
  - 説明: 文献種別('book')
- isbn
  - 型: string
  - 説明: 書籍の ISBN
- author
  - 型: string
  - 説明: 著者名
- publisher
  - 型: string
  - 説明: 出版社
- publication_year
  - 型: integer
  - 説明: 出版年

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(文字数超過、必須フィールド欠如、ISBN 重複)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: book-create-api -->

### 3. 文献読み取り API(単一)

<!-- GLOBAL_CONCLUSION_BEGIN: literature-get-single-api -->

#### エンドポイント

- パス: `/api/literatures/{id}`
- メソッド: GET

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 文献 ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- Union 型: kind フィールドにより Web 記事または書籍を判別
- **Web 記事の場合:**
  - id
    - 型: integer
    - 説明: 文献 ID
  - kind
    - 型: string
    - 説明: 文献種別('web')
  - url
    - 型: string
    - 説明: Web 記事の URL
  - title
    - 型: string
    - 説明: Web 記事のタイトル
- **書籍の場合:**
  - id
    - 型: integer
    - 説明: 文献 ID
  - kind
    - 型: string
    - 説明: 文献種別('book')
  - isbn
    - 型: string
    - 説明: 書籍の ISBN
  - author
    - 型: string
    - 説明: 著者名
  - publisher
    - 型: string
    - 説明: 出版社
  - publication_year
    - 型: integer
    - 説明: 出版年

#### エラーレスポンス

- 404 Not Found
  - 原因: 指定 ID の文献が存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "文献が見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: literature-get-single-api -->

## 結論

文献 API として、文献種別ごとの作成エンドポイントと ID による読み取りエンドポイントを定義した:

- **作成 API**: Web 記事(`/api/literatures/web-articles`)と書籍(`/api/literatures/books`)で別エンドポイントを提供し、型安全性を確保。同一 URL/ISBN の重複登録時は 400 エラーを返す。
- **読み取り API**: ID による単一取得エンドポイントで、Union 型レスポンスにより kind フィールドで文献種別を判別可能。

更新・削除 API は不要として除外した。各エンドポイントは GET/POST による CQS ベースの RESTful 設計に従い、適切な HTTP ステータスコードとエラーハンドリングを提供する。
