<!-- markdownlint-disable MD024 -->

# API 契約統合

## 概要

C4 モデルの Component レベルで定義された API 仕様を統合する。CogitoWeave システムにおける 9 つの論理的 API 群、21 のエンドポイントの完全な契約定義を提供し、フロントエンド・バックエンド間の HTTP API 契約を明文化する。

## エンティティ操作 API

### 概念 API

<!-- REFERENCE_BEGIN: concept-create-api -->

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

<!-- REFERENCE_END: concept-create-api -->

<!-- REFERENCE_BEGIN: concept-get-single-api -->

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

<!-- REFERENCE_END: concept-get-single-api -->

<!-- REFERENCE_BEGIN: concept-get-list-api -->

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

<!-- REFERENCE_END: concept-get-list-api -->

<!-- REFERENCE_BEGIN: concept-update-api -->

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

<!-- REFERENCE_END: concept-update-api -->

### 文献メモ API

<!-- REFERENCE_BEGIN: literature-memo-create-api -->

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

<!-- REFERENCE_END: literature-memo-create-api -->

<!-- REFERENCE_BEGIN: literature-memo-get-single-api -->

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

<!-- REFERENCE_END: literature-memo-get-single-api -->

<!-- REFERENCE_BEGIN: literature-memo-get-by-concept-api -->

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

<!-- REFERENCE_END: literature-memo-get-by-concept-api -->

<!-- REFERENCE_BEGIN: literature-memo-update-api -->

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

<!-- REFERENCE_END: literature-memo-update-api -->

### 関係性 API

<!-- REFERENCE_BEGIN: relationship-create-api -->

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

<!-- REFERENCE_END: relationship-create-api -->

<!-- REFERENCE_BEGIN: relationship-get-single-api -->

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

<!-- REFERENCE_END: relationship-get-single-api -->

<!-- REFERENCE_BEGIN: relationship-get-by-concept-api -->

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

<!-- REFERENCE_END: relationship-get-by-concept-api -->

<!-- REFERENCE_BEGIN: relationship-update-api -->

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

<!-- REFERENCE_END: relationship-update-api -->

### 文献 API

<!-- REFERENCE_BEGIN: literature-create-api -->

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

<!-- REFERENCE_END: literature-create-api -->

<!-- REFERENCE_BEGIN: literature-get-single-api -->

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

<!-- REFERENCE_END: literature-get-single-api -->

<!-- REFERENCE_BEGIN: literature-get-by-url-api -->

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

<!-- REFERENCE_END: literature-get-by-url-api -->

## 検索・発見 API

### 概念検索 API

<!-- REFERENCE_BEGIN: concept-search-api -->

#### エンドポイント

- パス: `/api/concepts/search`
- メソッド: GET

#### クエリパラメータ

- query
  - 型: string
  - 必須/任意: 必須
  - 説明: 検索クエリ(名前・説明での部分一致検索)
  - バリデーション: 1-200 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- concepts
  - 型: array
  - 説明: 検索結果の概念リスト
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

- 400 Bad Request
  - 原因: バリデーションエラー(query パラメータ欠如、文字数超過)
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

<!-- REFERENCE_END: concept-search-api -->

### 質問から概念候補提示 API

<!-- REFERENCE_BEGIN: question-to-concepts-api -->

#### エンドポイント

- パス: `/api/questions/suggest-concepts`
- メソッド: POST

#### リクエストスキーマ

- question
  - 型: string
  - 必須/任意: 必須
  - 説明: 質問内容
  - バリデーション: 1-2000 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- concepts
  - 型: array
  - 説明: 提示された概念候補のリスト
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
    - relevance_reason
      - 型: string
      - 説明: この概念が関連する理由(LLM による推論結果)

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(question フィールド欠如、文字数超過)
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
- 503 Service Unavailable
  - 原因: LLM API の一時的な障害
  - レスポンス:
    - error
      - 型: string
      - 説明: "LLM サービスが一時的に利用できません"

<!-- REFERENCE_END: question-to-concepts-api -->

## 協働・処理 API

### LLM 要約生成 API

<!-- REFERENCE_BEGIN: llm-summary-api -->

#### エンドポイント

- パス: `/api/llm/summarize`
- メソッド: POST

#### リクエストスキーマ

- concept_id
  - 型: integer
  - 必須/任意: 必須
  - 説明: 概念 ID(要約の観点を指定)
  - バリデーション: 正の整数、外部キー制約(concepts テーブルに存在すること)
- article_content
  - 型: string
  - 必須/任意: 必須
  - 説明: 記事の本文内容
  - バリデーション: 1 文字以上
- article_title
  - 型: string
  - 必須/任意: 必須
  - 説明: 記事のタイトル
  - バリデーション: 1-500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- summary_title
  - 型: string
  - 説明: 生成された要約のタイトル
- summary_content
  - 型: string
  - 説明: 生成された要約の内容

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(必須フィールド欠如、外部キー制約違反、文字数超過)
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
- 503 Service Unavailable
  - 原因: LLM API の一時的な障害
  - レスポンス:
    - error
      - 型: string
      - 説明: "LLM サービスが一時的に利用できません"

<!-- REFERENCE_END: llm-summary-api -->

### 記事取得 API

<!-- REFERENCE_BEGIN: article-fetch-api -->

#### エンドポイント

- パス: `/api/articles/fetch`
- メソッド: POST

#### リクエストスキーマ

- url
  - 型: string
  - 必須/任意: 必須
  - 説明: 取得する記事の URL
  - バリデーション: 1-2048 文字、URL 形式

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- title
  - 型: string
  - 説明: 記事のタイトル
- content
  - 型: string
  - 説明: 記事の本文内容

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(url フィールド欠如、文字数超過、URL 形式不正)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 404 Not Found
  - 原因: 指定 URL の記事が見つからない
  - レスポンス:
    - error
      - 型: string
      - 説明: "記事が見つかりません"
- 500 Internal Server Error
  - 原因: サーバー内部エラー
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
- 503 Service Unavailable
  - 原因: 記事取得サービスの一時的な障害
  - レスポンス:
    - error
      - 型: string
      - 説明: "記事取得サービスが一時的に利用できません"

<!-- REFERENCE_END: article-fetch-api -->

### バックアップ API

<!-- REFERENCE_BEGIN: backup-execute-api -->

#### エンドポイント

- パス: `/api/backup/execute`
- メソッド: POST

#### リクエストスキーマ

なし(リクエストボディは不要)

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- backup_path
  - 型: string
  - 説明: 作成されたバックアップファイルのパス
- backup_size
  - 型: integer
  - 説明: バックアップファイルのサイズ(バイト)
- backup_timestamp
  - 型: string
  - 説明: バックアップ実行日時(ISO 8601 形式)
- record_count
  - 型: integer
  - 説明: バックアップされたレコード総数

#### エラーレスポンス

- 500 Internal Server Error
  - 原因: バックアップ処理の失敗
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バックアップ失敗の詳細メッセージリスト

<!-- REFERENCE_END: backup-execute-api -->

<!-- REFERENCE_BEGIN: backup-verify-api -->

#### エンドポイント

- パス: `/api/backup/verify`
- メソッド: POST

#### リクエストスキーマ

- backup_path
  - 型: string
  - 必須/任意: 必須
  - 説明: 検証するバックアップファイルのパス
  - バリデーション: 1-500 文字

#### レスポンススキーマ(成功時)

- HTTP ステータス: 200 OK
- is_valid
  - 型: boolean
  - 説明: バックアップの有効性(true: 有効、false: 無効)
- verified_timestamp
  - 型: string
  - 説明: 検証実行日時(ISO 8601 形式)
- record_count
  - 型: integer
  - 説明: バックアップに含まれるレコード総数

#### エラーレスポンス

- 400 Bad Request
  - 原因: バリデーションエラー(backup_path フィールド欠如、文字数超過)
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バリデーションエラーの詳細メッセージリスト
- 404 Not Found
  - 原因: 指定されたバックアップファイルが存在しない
  - レスポンス:
    - error
      - 型: string
      - 説明: "バックアップファイルが見つかりません"
- 500 Internal Server Error
  - 原因: バックアップ検証処理の失敗
  - レスポンス:
    - error
      - 型: string
      - 説明: エラーメッセージ
    - details
      - 型: array
      - 要素の型: string
      - 説明: バックアップ検証失敗の詳細メッセージリスト

<!-- REFERENCE_END: backup-verify-api -->
