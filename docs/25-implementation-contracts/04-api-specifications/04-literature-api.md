---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "literature-entity"
    - "literature-table-schema"
    - "distributed-communication-design"
  produces:
    - "literature-create-api"
    - "literature-get-single-api"
    - "literature-get-by-url-api"
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
4. **文献 API**: 作成・読み取り (URL・タイトル・引用が必須)

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
- 基本属性: ID、URL、タイトル、取得日時
- 複数の概念観点から文献メモが生成される元情報

<!-- PREMISE_END: literature-entity -->

<!-- PREMISE_BEGIN: literature-table-schema -->

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

<!-- PREMISE_END: literature-table-schema -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

文献の作成、読み取りを提供する。文献は外部文献の書誌情報を管理し、複数の概念観点から文献メモが生成される元情報として機能する。更新・削除は不要とし、シンプルな書誌情報管理を実現する。

## API 一覧

### 1. 文献作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: literature-create-api -->

#### エンドポイント

- パス: `/api/literatures`
- メソッド: POST

#### リクエストスキーマ

- url
  - 型: string
  - 必須/任意: 必須
  - 説明: 文献の URL
  - バリデーション: 1-2048 文字、URL 形式
- title
  - 型: string
  - 必須/任意: 必須
  - 説明: 文献のタイトル
  - バリデーション: 1-500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された文献の ID
- url
  - 型: string
  - 説明: 文献の URL
- title
  - 型: string
  - 説明: 文献のタイトル
- fetched_at
  - 型: string
  - 説明: 取得日時(ISO 8601 形式)

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

<!-- GLOBAL_CONCLUSION_END: literature-create-api -->

### 2. 文献読み取り API(単一)

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
- id
  - 型: integer
  - 説明: 文献 ID
- url
  - 型: string
  - 説明: 文献の URL
- title
  - 型: string
  - 説明: 文献のタイトル
- fetched_at
  - 型: string
  - 説明: 取得日時(ISO 8601 形式)

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

### 3. 文献読み取り API(URL 検索)

<!-- GLOBAL_CONCLUSION_BEGIN: literature-get-by-url-api -->

#### エンドポイント

- パス: `/api/literatures/search`
- メソッド: GET

#### クエリパラメータ

- url
  - 型: string
  - 必須/任意: 必須
  - 説明: 検索する URL(完全一致)
  - バリデーション: 1-2048 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- literatures
  - 型: array
  - 説明: 文献のリスト(完全一致する URL の文献のみ)
  - 要素:
    - id
      - 型: integer
      - 説明: 文献 ID
    - url
      - 型: string
      - 説明: 文献の URL
    - title
      - 型: string
      - 説明: 文献のタイトル
    - fetched_at
      - 型: string
      - 説明: 取得日時(ISO 8601 形式)

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(url パラメータ欠如、文字数超過)
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

<!-- GLOBAL_CONCLUSION_END: literature-get-by-url-api -->

## 結論

文献 API として、作成・読み取り(単一・URL 検索)の 3 つのエンドポイントを定義した。更新・削除 API は不要として除外した。各エンドポイントは GET/POST による CQS ベースの RESTful 設計に従い、適切な HTTP ステータスコードとエラーハンドリングを提供する。URL 検索機能により、同一 URL の文献の重複登録を防止できる。
