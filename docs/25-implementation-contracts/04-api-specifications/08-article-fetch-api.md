---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "distributed-communication-design"
  produces:
    - "article-fetch-api"
---

<!-- markdownlint-disable MD024 -->

# 記事取得 API 仕様

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

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

URL からタイトル・内容を取得する。外部記事の自動取得により、文献メモ作成の効率化を支援する。

## API 一覧

### 1. 記事取得 API

<!-- GLOBAL_CONCLUSION_BEGIN: article-fetch-api -->

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

<!-- GLOBAL_CONCLUSION_END: article-fetch-api -->

## 結論

記事取得 API として、URL からのタイトル・内容取得を提供する単一エンドポイントを定義した。POST メソッドによるコマンド実行として設計し、適切な HTTP ステータスコードとエラーハンドリングを提供する。記事取得サービス障害時には 503 エラーを返却し、フロントエンド側でのリトライ戦略を可能にする。
