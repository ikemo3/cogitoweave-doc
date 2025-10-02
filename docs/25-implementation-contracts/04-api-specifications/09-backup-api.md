---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "api-contract"
    - "batch-specification"
    - "distributed-communication-design"
  produces:
    - "backup-execute-api"
    - "backup-verify-api"
---

<!-- markdownlint-disable MD024 -->

# バックアップ API 仕様

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

<!-- PREMISE_BEGIN: batch-specification -->

CogitoWeave のバッチ仕様は日次データバックアップのみを実装する。全データの完全バックアップを日次実行し、システム障害時の迅速復旧を支援する。自動分析・推奨機能、データクリーンアップ、統計処理等は設計方針により除外する。

<!-- PREMISE_END: batch-specification -->

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

## 概要

Webhook 化による自動実行支援を提供する。外部スケジューラーからの定期実行により、日次データバックアップを自動化する。

## API 一覧

### 1. バックアップ実行 API

<!-- GLOBAL_CONCLUSION_BEGIN: backup-execute-api -->

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

<!-- GLOBAL_CONCLUSION_END: backup-execute-api -->

### 2. バックアップ検証 API

<!-- GLOBAL_CONCLUSION_BEGIN: backup-verify-api -->

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

<!-- GLOBAL_CONCLUSION_END: backup-verify-api -->

## 結論

バックアップ API として、バックアップ実行とバックアップ検証の 2 つのエンドポイントを定義した。各エンドポイントは POST メソッドによるコマンド実行として設計し、適切な HTTP ステータスコードとエラーハンドリングを提供する。Webhook 化により外部スケジューラーからの定期実行を可能にし、日次データバックアップの自動化を実現する。
