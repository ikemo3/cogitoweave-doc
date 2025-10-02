---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "usage-context"
    - "development-constraints"
  produces:
    - "api-doc-selection"
---

# API ドキュメント生成選択

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- PREMISE_END: usage-context -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

## 前提

Python + FastAPI の言語・フレームワーク選択により、FastAPI が提供する OpenAPI 仕様の自動生成機能を前提とする。CogitoWeave は個人利用システムであり、外部連携用の高度な API 仕様書は不要である。フロントエンド開発において API 仕様の確認とテスト実行が主な用途となり、一人開発制約により保守負荷の軽減が重要である。

## 論理

第一に、CogitoWeave の API 仕様書用途は限定的である。個人利用システムのため外部連携は発生せず、主な利用者はフロントエンド開発時の自分自身となる。この用途では基本的な API 仕様の確認とインタラクティブなテスト実行ができれば十分であり、複雑なドキュメント機能は過剰である。

第二に、FastAPI は型ヒントベースの自動 OpenAPI 仕様生成を標準機能として提供している。Python の型ヒントから自動的に Swagger UI と ReDoc の両方を生成し、リクエスト・レスポンスの詳細仕様、認証情報、インタラクティブなテスト機能まで包含している。この機能はコードファーストアプローチを完全に実現し、コードと仕様の同期を自動的に維持する。

第三に、外部ツールの導入は一人開発制約に反する。Stoplight Studio、Insomnia、Postman 等の外部ツールは高機能だが、追加の学習コスト、設定・保守負荷、ツール間の同期作業が発生する。これらの追加負荷は個人開発において明確な不利益となり、技術選択基準の「シンプルで理解しやすい構造」に矛盾する。

第四に、FastAPI 内蔵機能による充足性は実証されている。Swagger UI によるインタラクティブな API 仕様確認とテスト実行、ReDoc による読みやすいドキュメント表示、機械可読な OpenAPI JSON 出力により、フロントエンド開発に必要な全機能が提供される。追加ツールによる機能拡張の必要性は認められない。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: api-doc-selection -->

FastAPI 内蔵の Swagger UI・ReDoc 機能を使用し、外部 API 仕様書生成ツールは導入しない。

<!-- GLOBAL_CONCLUSION_END: api-doc-selection -->

### 論理的根拠

FastAPI の型ヒントベース自動 OpenAPI 仕様生成により、コードファーストアプローチでコードと仕様の同期を自動的に維持できる。Swagger UI と ReDoc によりインタラクティブな API 仕様確認とテスト実行が可能であり、外部ツール導入による追加の学習コスト・設定・保守負荷・ツール間同期作業を回避して一人開発制約に適合する。
