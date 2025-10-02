---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "literature-memo-entity"
    - "literature-memo-table-schema"
    - "distributed-communication-design"
  produces:
    - "literature-memo-create-api"
    - "literature-memo-get-single-api"
    - "literature-memo-get-by-concept-api"
    - "literature-memo-update-api"
---

<!-- markdownlint-disable MD024 -->

# 文献メモ API 仕様

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

<!-- PREMISE_BEGIN: literature-memo-entity -->

文献メモエンティティ(LiteratureMemo):

- ツェッテルカステン方式による概念ベースの知識蓄積
- 基本属性: ID、タイトル、内容、作成日時、更新日時
- 特定の概念の観点で外部文献を要約・解釈したもの
- LLM 協働により効率的に作成され、個人の理解で修正される

<!-- PREMISE_END: literature-memo-entity -->

<!-- PREMISE_BEGIN: literature-memo-table-schema -->

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

<!-- PREMISE_END: literature-memo-table-schema -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

文献メモの作成、読み取り、更新、概念別一覧取得を提供する。文献メモは特定の概念の観点で外部文献を要約・解釈したものであり、LLM 協働により効率的に作成され個人の理解で修正される。

## API 一覧

### 1. 文献メモ作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-create-api -->

#### エンドポイント

- パス: `/api/literature-memos`
- メソッド: POST

#### リクエストスキーマ

- concept_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID
  - バリデーション: 正の整数、外部キー制約(concepts テーブルに存在すること)
- literature_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 文献 ID
  - バリデーション: 正の整数、外部キー制約(literatures テーブルに存在すること)
- title
  - 型: string
  - 必須/任意: 必須
  - 説明: 文献メモのタイトル
  - バリデーション: 1-500 文字
- content
  - 型: string
  - 必須/任意: 必須
  - 説明: 文献メモの内容
  - バリデーション: 1 文字以上

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された文献メモの ID
- concept_id
  - 型: integer
  - 説明: 概念 ID
- literature_id
  - 型: integer
  - 説明: 文献 ID
- title
  - 型: string
  - 説明: 文献メモのタイトル
- content
  - 型: string
  - 説明: 文献メモの内容
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)
- updated_at
  - 型: string
  - 説明: 更新日時(ISO 8601 形式)

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(文字数超過、必須フィールド欠如、外部キー制約違反)
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

<!-- GLOBAL_CONCLUSION_END: literature-memo-create-api -->

### 2. 文献メモ読み取り API(単一)

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-get-single-api -->

#### エンドポイント

- パス: `/api/literature-memos/{id}`
- メソッド: GET

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 文献メモ ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 文献メモ ID
- concept_id
  - 型: integer
  - 説明: 概念 ID
- literature_id
  - 型: integer
  - 説明: 文献 ID
- title
  - 型: string
  - 説明: 文献メモのタイトル
- content
  - 型: string
  - 説明: 文献メモの内容
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)
- updated_at
  - 型: string
  - 説明: 更新日時(ISO 8601 形式)

#### エラーレスポンス

- 404 Not Found
  - 原因: 指定 ID の文献メモが存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "文献メモが見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: literature-memo-get-single-api -->

### 3. 文献メモ読み取り API(概念別一覧)

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-get-by-concept-api -->

#### エンドポイント

- パス: `/api/concepts/{concept_id}/literature-memos`
- メソッド: GET

#### パスパラメータ

- concept_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- literature_memos
  - 型: array
  - 説明: 文献メモのリスト
  - 要素:
    - id
      - 型: integer
      - 説明: 文献メモ ID
    - concept_id
      - 型: integer
      - 説明: 概念 ID
    - literature_id
      - 型: integer
      - 説明: 文献 ID
    - title
      - 型: string
      - 説明: 文献メモのタイトル
    - content
      - 型: string
      - 説明: 文献メモの内容
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

<!-- GLOBAL_CONCLUSION_END: literature-memo-get-by-concept-api -->

### 4. 文献メモ更新 API

<!-- GLOBAL_CONCLUSION_BEGIN: literature-memo-update-api -->

#### エンドポイント

- パス: `/api/literature-memos/{id}`
- メソッド: POST

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 文献メモ ID
  - バリデーション: 正の整数

#### リクエストスキーマ

- title
  - 型: string
  - 必須/任意: 任意
  - 説明: 文献メモのタイトル
  - バリデーション: 1-500 文字
- content
  - 型: string
  - 必須/任意: 任意
  - 説明: 文献メモの内容
  - バリデーション: 1 文字以上

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 文献メモ ID
- concept_id
  - 型: integer
  - 説明: 概念 ID
- literature_id
  - 型: integer
  - 説明: 文献 ID
- title
  - 型: string
  - 説明: 文献メモのタイトル
- content
  - 型: string
  - 説明: 文献メモの内容
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
  - 原因: 指定 ID の文献メモが存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "文献メモが見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: literature-memo-update-api -->

## 結論

文献メモ API として、作成・読み取り(単一・概念別一覧)・更新の 4 つのエンドポイントを定義した。削除 API は優先度低として除外した。各エンドポイントは GET/POST による CQS ベースの RESTful 設計に従い、適切な HTTP ステータスコードとエラーハンドリングを提供する。概念別一覧取得により、特定の概念に関連する文献メモを効率的に取得できる。
