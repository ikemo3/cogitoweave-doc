---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "concept-entity"
    - "concept-table-schema"
    - "distributed-communication-design"
  produces:
    - "concept-create-api"
    - "concept-get-single-api"
    - "concept-get-list-api"
    - "concept-update-api"
---

<!-- markdownlint-disable MD024 -->

# 概念 API 仕様

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

<!-- PREMISE_BEGIN: concept-entity -->

概念エンティティ(Concept):

- 思考の最小単位として知識体系の中心に位置する
- 基本属性: ID、名前、説明、作成日時、更新日時
- 他の概念との関係性によって意味を形成する
- 外部情報を個人の理解に変換する観点として機能する

<!-- PREMISE_END: concept-entity -->

<!-- PREMISE_BEGIN: concept-table-schema -->

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

<!-- PREMISE_END: concept-table-schema -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

概念の作成、読み取り、更新を提供する。概念は知識体系の中心に位置し、外部情報を個人の理解に変換するフィルターとして機能する。

## API 一覧

### 1. 概念作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: concept-create-api -->

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

<!-- GLOBAL_CONCLUSION_END: concept-create-api -->

### 2. 概念読み取り API(単一)

<!-- GLOBAL_CONCLUSION_BEGIN: concept-get-single-api -->

#### エンドポイント

- パス: `/api/concepts/{id}`
- メソッド: GET

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 概念 ID
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

- 404 Not Found
  - 原因: 指定 ID の概念が存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "概念が見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: concept-get-single-api -->

### 3. 概念読み取り API(一覧)

<!-- GLOBAL_CONCLUSION_BEGIN: concept-get-list-api -->

#### エンドポイント

- パス: `/api/concepts`
- メソッド: GET

#### クエリパラメータ

なし(全件取得)

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- concepts
  - 型: array
  - 説明: 概念のリスト
  - 要素:
    - id
      - 型: integer
      - 説明: 概念 ID
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

- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: concept-get-list-api -->

### 4. 概念更新 API

<!-- GLOBAL_CONCLUSION_BEGIN: concept-update-api -->

#### エンドポイント

- パス: `/api/concepts/{id}`
- メソッド: POST

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID
  - バリデーション: 正の整数

#### リクエストスキーマ

- name
  - 型: string
  - 必須/任意: 任意
  - 説明: 概念名
  - バリデーション: 1-100 文字、重複不可
- description
  - 型: string
  - 必須/任意: 任意
  - 説明: 概念の説明
  - バリデーション: 最大 1000 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 概念 ID
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
  - 原因: バリデーションエラー(文字数超過)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 404 Not Found
  - 原因: 指定 ID の概念が存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "概念が見つかりません"
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

<!-- GLOBAL_CONCLUSION_END: concept-update-api -->

## 結論

概念 API として、作成・読み取り(単一・一覧)・更新の 4 つのエンドポイントを定義した。削除 API は優先度低として除外した。各エンドポイントは GET/POST による CQS ベースの RESTful 設計に従い、適切な HTTP ステータスコードとエラーハンドリングを提供する。レスポンススキーマでは null を使用せず、description フィールドは常に存在し値がない場合は空文字列を返す設計とした。
