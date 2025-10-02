---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "distributed-communication-design"
  produces: []
---

<!-- markdownlint-disable MD024 -->

# API 設計仕様

## 概要

C4 モデルの Component レベルで定義された API 仕様を提供する。CogitoWeave システムにおけるフロントエンド・バックエンド間の HTTP API 契約を明文化し、各 API エンドポイントのリクエスト・レスポンス仕様、エラーハンドリング、バリデーションルールを定義する。

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

## API 仕様一覧

### エンティティ操作 API

- [概念 API](01-concept-api.md) - 概念の作成・読み取り・更新
- [文献メモ API](02-literature-memo-api.md) - 文献メモの作成・読み取り・更新・概念別一覧取得
- [関係性 API](03-relationship-api.md) - 関係性の作成・読み取り・更新
- [文献 API](04-literature-api.md) - 文献の作成・読み取り

### 検索・発見 API

- [概念検索 API](05-concept-search-api.md) - 名前・説明での部分一致検索
- [質問から概念候補提示 API](06-question-to-concepts-api.md) - 質問内容からの概念候補提示

### 協働・処理 API

- [LLM 要約生成 API](07-llm-summary-api.md) - 記事内容と概念 ID からの観点別要約生成
- [記事取得 API](08-article-fetch-api.md) - URL からのタイトル・内容取得
- [バックアップ API](09-backup-api.md) - Webhook 化による自動実行支援

## 共通設計方針

### HTTP メソッド

- **GET**: 読み取り専用操作
- **POST**: 作成・更新・コマンド実行

CQS ベースの設計により、PUT/PATCH/DELETE は使用しない。

### エラーハンドリング

- **4xx 系**: クライアントエラー(バリデーションエラー、リソース未発見等)
- **5xx 系**: サーバーエラー(内部エラー、外部 API 障害等)

全てのエラーレスポンスは以下の形式で統一する:

- error
  - 型: string
  - 説明: エラーメッセージ
- details (オプション)
  - 型: array
  - 要素の型: string
  - 説明: エラー詳細メッセージリスト

### データ型表現

- **null 禁止**: レスポンススキーマでは null を返さず、値がない場合は空文字列またはフィールド省略とする
- **タイムスタンプ**: ISO 8601 形式(例: `2024-01-01T12:00:00Z`)
- **ID**: 正の整数(integer)

### バリデーション

各 API で定義されたバリデーションルールに違反した場合、400 Bad Request を返す。

## 結論

API 設計仕様として、9 つの論理的 API 群に対応する仕様ファイルを提供する。各 API は GET/POST による CQS ベースの RESTful 設計に従い、統一されたエラーハンドリングとデータ型表現により保守性と理解しやすさを確保する。
