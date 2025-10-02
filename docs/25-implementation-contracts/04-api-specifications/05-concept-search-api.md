---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "concept-entity"
    - "distributed-communication-design"
  produces:
    - "concept-search-api"
---

<!-- markdownlint-disable MD024 -->

# 概念検索 API 仕様

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

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

概念の名前・説明での部分一致検索を提供する。概念の検索により、既存の知識体系から関連する概念を効率的に発見できる。

## API 一覧

### 1. 概念検索 API

<!-- GLOBAL_CONCLUSION_BEGIN: concept-search-api -->

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

<!-- GLOBAL_CONCLUSION_END: concept-search-api -->

## 結論

概念検索 API として、名前・説明での部分一致検索を提供する単一エンドポイントを定義した。GET メソッドによる読み取り専用操作として設計し、適切な HTTP ステータスコードとエラーハンドリングを提供する。検索結果は概念の全属性を含むリスト形式で返却する。
