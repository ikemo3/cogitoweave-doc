---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "concept-entity"
    - "distributed-communication-design"
  produces:
    - "question-to-concepts-api"
---

<!-- markdownlint-disable MD024 -->

# 質問から概念候補提示 API 仕様

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

質問内容から関連する概念候補を提示する。LLM による推論により、既存個人概念体系からの検索・発見を支援し、質問に対する適切な概念の選択を可能にする。

## API 一覧

### 1. 質問から概念候補提示 API

<!-- GLOBAL_CONCLUSION_BEGIN: question-to-concepts-api -->

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

<!-- GLOBAL_CONCLUSION_END: question-to-concepts-api -->

## 結論

質問から概念候補提示 API として、LLM による推論を活用した概念発見を提供する単一エンドポイントを定義した。POST メソッドによるコマンド実行として設計し、適切な HTTP ステータスコードとエラーハンドリングを提供する。LLM API 障害時には 503 エラーを返却し、フロントエンド側でのリトライ戦略を可能にする。
