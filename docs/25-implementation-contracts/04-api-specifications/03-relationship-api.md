---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "relationship-entity"
    - "relationship-table-schema"
    - "distributed-communication-design"
  produces:
    - "relationship-create-api"
    - "relationship-get-single-api"
    - "relationship-get-by-concept-api"
    - "relationship-update-api"
---

<!-- markdownlint-disable MD024 -->

# 関係性 API 仕様

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

<!-- PREMISE_BEGIN: relationship-entity -->

関係性エンティティ(Relationship):

- 概念間の無向グラフ構造による知識ネットワーク形成
- 基本属性: ID、関係名、説明、作成日時
- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援

<!-- PREMISE_END: relationship-entity -->

<!-- PREMISE_BEGIN: relationship-table-schema -->

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

<!-- PREMISE_END: relationship-table-schema -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

関係性の作成、読み取り、更新を提供する。関係性は概念間の無向グラフ構造による知識ネットワークを形成し、個人的な関係語彙による柔軟な関係定義を可能にする。概念間の重複関係を許可することで、複数の観点からの関係性表現を実現する。

## API 一覧

### 1. 関係性作成 API

<!-- GLOBAL_CONCLUSION_BEGIN: relationship-create-api -->

#### エンドポイント

- パス: `/api/relationships`
- メソッド: POST

#### リクエストスキーマ

- concept_a_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 A の ID
  - バリデーション: 正の整数、外部キー制約(concepts テーブルに存在すること)
- concept_b_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 B の ID
  - バリデーション: 正の整数、外部キー制約(concepts テーブルに存在すること)、concept_a_id と異なること
- relationship_name
  - 型: string
  - 必須/任意: 必須
  - 説明: 関係名
  - バリデーション: 1-100 文字
- description
  - 型: string
  - 必須/任意: 任意
  - 説明: 関係の説明
  - バリデーション: 最大 500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 201 Created
- id
  - 型: integer
  - 説明: 作成された関係性の ID
- concept_a_id
  - 型: integer
  - 説明: 概念 A の ID(常に concept_b_id より小さい値)
- concept_b_id
  - 型: integer
  - 説明: 概念 B の ID(常に concept_a_id より大きい値)
- relationship_name
  - 型: string
  - 説明: 関係名
- description
  - 型: string
  - 説明: 関係の説明(値がない場合は空文字列)
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(文字数超過、必須フィールド欠如、外部キー制約違反、同一概念の関係指定)
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

<!-- GLOBAL_CONCLUSION_END: relationship-create-api -->

### 2. 関係性読み取り API(単一)

<!-- GLOBAL_CONCLUSION_BEGIN: relationship-get-single-api -->

#### エンドポイント

- パス: `/api/relationships/{id}`
- メソッド: GET

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 関係性 ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 関係性 ID
- concept_a_id
  - 型: integer
  - 説明: 概念 A の ID
- concept_b_id
  - 型: integer
  - 説明: 概念 B の ID
- relationship_name
  - 型: string
  - 説明: 関係名
- description
  - 型: string
  - 説明: 関係の説明(値がない場合は空文字列)
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)

#### エラーレスポンス

- 404 Not Found
  - 原因: 指定 ID の関係性が存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "関係性が見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: relationship-get-single-api -->

### 3. 関係性読み取り API(概念別)

<!-- GLOBAL_CONCLUSION_BEGIN: relationship-get-by-concept-api -->

#### エンドポイント

- パス: `/api/concepts/{concept_id}/relationships`
- メソッド: GET

#### パスパラメータ

- concept_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID
  - バリデーション: 正の整数

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- relationships
  - 型: array
  - 説明: 関係性のリスト
  - 要素:
    - id
      - 型: integer
      - 説明: 関係性 ID
    - concept_a_id
      - 型: integer
      - 説明: 概念 A の ID
    - concept_b_id
      - 型: integer
      - 説明: 概念 B の ID
    - relationship_name
      - 型: string
      - 説明: 関係名
    - description
      - 型: string
      - 説明: 関係の説明(値がない場合は空文字列)
    - created_at
      - 型: string
      - 説明: 作成日時(ISO 8601 形式)

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

<!-- GLOBAL_CONCLUSION_END: relationship-get-by-concept-api -->

### 4. 関係性更新 API

<!-- GLOBAL_CONCLUSION_BEGIN: relationship-update-api -->

#### エンドポイント

- パス: `/api/relationships/{id}`
- メソッド: POST

#### パスパラメータ

- id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 関係性 ID
  - バリデーション: 正の整数

#### リクエストスキーマ

- relationship_name
  - 型: string
  - 必須/任意: 任意
  - 説明: 関係名
  - バリデーション: 1-100 文字
- description
  - 型: string
  - 必須/任意: 任意
  - 説明: 関係の説明
  - バリデーション: 最大 500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- id
  - 型: integer
  - 説明: 関係性 ID
- concept_a_id
  - 型: integer
  - 説明: 概念 A の ID
- concept_b_id
  - 型: integer
  - 説明: 概念 B の ID
- relationship_name
  - 型: string
  - 説明: 関係名
- description
  - 型: string
  - 説明: 関係の説明(値がない場合は空文字列)
- created_at
  - 型: string
  - 説明: 作成日時(ISO 8601 形式)

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
  - 原因: 指定 ID の関係性が存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "関係性が見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ

<!-- GLOBAL_CONCLUSION_END: relationship-update-api -->

## 結論

関係性 API として、作成・読み取り(単一・概念別)・更新の 4 つのエンドポイントを定義した。削除 API は優先度低として除外した。各エンドポイントは GET/POST による CQS ベースの RESTful 設計に従い、適切な HTTP ステータスコードとエラーハンドリングを提供する。無向グラフ構造により concept_a_id < concept_b_id の制約を保ち、概念間の重複関係を許可することで柔軟な知識ネットワーク構築を実現する。
