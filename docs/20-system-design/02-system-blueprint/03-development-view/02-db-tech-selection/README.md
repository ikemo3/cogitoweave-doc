---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "database-approach"
    - "usage-context"
    - "performance-requirements"
    - "development-constraints"
    - "existing-resource-constraints"
    - "data-protection-requirements"
    - "scaling-strategy"
  produces:
    - "sqlite-selection"
---

# DB 技術選択

## 前提

<!-- PREMISE_BEGIN: database-approach -->

CogitoWeave では RDBMS を採用する。リレーショナル構造によるデータ整合性管理、成熟技術による保守性確保、データ規模への十分な対応能力が主な選択理由である。具体的な製品選択は Development View で決定する。

<!-- PREMISE_END: database-approach -->

<!-- PREMISE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- PREMISE_END: usage-context -->

<!-- PREMISE_BEGIN: performance-requirements -->

- 必須対応
  - システム初回起動: 3 秒以内(アプリケーション全体の初期化)
  - 概念ページ読み込み: 1.5 秒以内(概念間の移動、継続利用での主要操作)
  - 検索結果表示: 1 秒以内(現代の検索体験として期待される応答時間)
  - LLM API 呼び出し: 10 秒以内(生成 AI 市場で期待される処理時間)
  - データベース応答: ミリ秒オーダー(現代のソフトウェアで基本的に期待される応答時間)
- 任意対応
  - 既存ツールを上回るパフォーマンス最適化
- 除外対応
  - 複数ユーザー同時アクセス: 個人ツール市場では期待されない
  - 大規模データ処理: 個人用途範囲を超える性能は不要
  - 企業向け機能: 個人ツール市場では要求されない

<!-- PREMISE_END: performance-requirements -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: data-protection-requirements -->

- **自動バックアップ**: 日次自動バックアップによる合理的なデータ保護
- **迅速復旧**: 障害発生時の当日復旧を目標とした手順整備
- **部分ロス許容**: 完全なゼロリスクは求めず、実用的保護レベルで運用

<!-- PREMISE_END: data-protection-requirements -->

<!-- PREMISE_BEGIN: scaling-strategy -->

- **単一サーバー構成**: 水平スケーリングの複雑さを回避し、必要に応じた垂直スケーリングにより性能向上を図る。
- **数万件規模への対応**: 概念・文献メモ・関係性データを含めて数万件規模まで対応し、現代の一般的なサーバーで十分な性能を実現する。
- **シンプル運用を優先**: 複雑なスケーリング機構や最適化は実装せず、理解しやすく保守しやすい構成を維持する。

<!-- PREMISE_END: scaling-strategy -->

CogitoWeave システムでは、RDBMS 採用の基本方針の下で、単一ユーザー専用システムとしてのパフォーマンス要件、開発制約、既存リソース制約、データ保護要件、スケーリング戦略が確立されている。これらの制約条件に最適な RDBMS 製品の選択が必要である。

## 論理

### RDBMS 選択肢の特性分析

前提条件を満たすために、主要な RDBMS 選択肢の特性を分析する。

**PostgreSQL**:

- 高機能: JSON サポート、全文検索、拡張性に優れる
- データ整合性: ACID 特性の厳密な実装を提供する
- 運用負荷: デフォルト設定で動作するが、メモリ使用量が多い
- バックアップ: pg_dump/pg_restore による 2 段階手順が必要である

**MySQL**:

- 標準機能: 一般的な RDBMS 機能を提供する
- データ整合性: InnoDB エンジンにより ACID 特性を確保する
- 運用負荷: デフォルト設定で適切に動作する
- バックアップ: mysqldump/mysql による 2 段階手順が必要である

**SQLite**:

- 基本機能: 標準的な RDBMS 機能を提供する
- データ整合性: ACID 特性を完全に実装する
- 運用負荷: サーバー起動・設定が一切不要である
- バックアップ: ファイルコピーによる単一手順で完了する

### 制約条件との適合性評価

各選択肢を前提条件と照合して適合性を評価する。

**自宅サーバー・一人開発制約**:

- PostgreSQL: メモリ使用量が多く、個人環境では過剰である
- MySQL: 適切だが、サーバー管理の負荷がある
- SQLite: サーバー管理が不要で最も適合する

**個人利用・数万件規模制約**:

- PostgreSQL: 規模に対して機能過剰である
- MySQL: 適切な規模感である
- SQLite: 十分な性能を提供する

**データ整合性・バックアップ制約**:

- PostgreSQL: ACID 保証だが、バックアップ手順が複雑である
- MySQL: ACID 保証だが、バックアップ手順が複雑である
- SQLite: ACID 保証かつ最も単純なバックアップ手順である

### 最適解の導出

制約条件との適合性分析により、以下の論理的結論が導出される:

**運用負荷の最小化**: 一人開発制約下では、運用負荷が最小の選択肢が最も合理的である。SQLite はサーバー管理が不要で最小の運用負荷を実現する。

**要件充足性**: 個人利用・数万件規模の制約下では、SQLite の機能・性能で要件を十分に満たせる。PostgreSQL/MySQL の高度な機能は現時点で不要である。

**バックアップ簡易性**: 日次バックアップ要件に対して、SQLite のファイルコピーが最も確実で単純な手順を提供する。

**開発効率性**: 設定不要で即座に開始できる SQLite が、開発初期の効率を最大化する。

これらの論理的分析により、現在の制約条件下では SQLite が最適な選択肢であることが導出される。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: sqlite-selection -->

CogitoWeave システムでは **SQLite** を選択する。

<!-- GLOBAL_CONCLUSION_END: sqlite-selection -->

### 論理的根拠

現在の制約条件下では、SQLite が最適な選択である。一人開発制約においてサーバー管理が不要で運用負荷が最小であり、個人利用・数万件規模の制約下では SQLite の機能・性能で要件を十分に満たせる。日次バックアップ要件に対して、SQLite のファイルコピーが最も確実で単純な手順を提供し、設定不要で即座に開始できる SQLite が開発初期の効率を最大化する。
