---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "fault-tolerance-strategy"
  produces:
    - "inter-process-fault-tolerance-technology-mapping"
    - "inter-process-fault-tolerance-missing-technologies"
---

# サービス間 Process 制約の見直し - 耐障害性設計

## Physical + Development 制約の技術スタック

各コンテナの技術スタックは以下のとおりである。この技術構成に Process 制約を適合させる必要がある。

<!-- PREMISE_BEGIN: development-constraint-simplified-list -->

- Frontend(Container)
  - フレームワーク：Solid + TanStack Router
  - CSS 手法：Tailwind CSS + CSS Modules ハイブリッド構成
  - UI コンポーネント：Tailwind CSS ベースカスタム実装
  - グラフ可視化：D3
- Backend(Container)
  - 言語・フレームワーク：Python + FastAPI
  - アプリケーションサーバー：uvicorn
  - ORM・マイグレーション：SQLAlchemy + Alembic
  - API ドキュメント生成：FastAPI 内蔵 Swagger UI・ReDoc
  - バリデーション：FastAPI 標準 Pydantic
  - HTTP クライアント：httpx
  - 設定管理：環境変数方式
  - ログ・モニタリング：loguru
- DB(Container)
  - データベース：SQLite
- Vite proxy(Container)
  - ビルドツール：Vite
- GitHub Actions(Container)
  - Git サービス：GitHub
  - フロントエンドテスト：Vitest + Testing Library + Playwright
  - バックエンドテスト
    - テストフレームワーク：pytest
    - API テスト：FastAPI TestClient
    - 非同期テスト：pytest-asyncio
    - モック機能：pytest-mock
    - HTTP モック：httpx-mock
    - 時間モック：time-machine
    - カバレッジ：pytest-cov
    - 並列実行：pytest-xdist
- Fluent Bit(Container)
  - ログ収集：Fluent Bit
- Cloud Monitoring(Container)
  - 監視・ログ：Google Cloud Monitoring
- Sentry(Container)
  - APM・トレーシング：Sentry

<!-- PREMISE_END: development-constraint-simplified-list -->

## Process 制約の整理

Physical 制約・Development 制約との適合性を検討する前に、対象となる Process 制約の内容を整理する。

### 耐障害性設計

<!-- PREMISE_BEGIN: fault-tolerance-strategy -->

- **データ保護を最優先**: 定期バックアップとバックアップ検証により、概念・文献メモ・関係性データの消失を絶対に防止する。
- **部分機能の継続**: コア知識管理機能を中心とした基本機能の継続動作により、外部 API 障害時でも知識体系の閲覧・編集・保存を維持する。
- **手動復旧を基本**: 自動復旧機構の複雑さを回避し、明確な手動復旧手順と充実したエラー情報により個人での障害対応を支援する。
- **明確なエラー伝達**: 障害種別・影響範囲・対応手順を分かりやすく表示し、開発者自身の技術的判断を支援する。

<!-- PREMISE_END: fault-tolerance-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

耐障害性設計は技術選択に依存しない設計方針であり、Physical・Development 制約でも適用可能である。単一 Backend Container 構成でも、データ保護・部分機能継続・手動復旧・エラー伝達の各要素を実現できる。

## Physical 制約との調整

Physical 制約との矛盾は発生しなかったため、調整は不要である。

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各耐障害性要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### データ保護を最優先の検討

**Process 制約内容**: 定期バックアップとバックアップ検証により、概念・文献メモ・関係性データの消失を絶対に防止する。

**Development 制約との適合性**:

定期バックアップは Database Container の SQLite により実現可能である。SQLite はファイルベースであり、定期的なバックアップが容易である。バックアップ検証も標準的なファイルチェック手法で実現可能である。

**使用技術および設定**:

- SQLite: ファイルベースデータベースでの定期バックアップ
- ファイルシステム: バックアップファイルの検証

### 部分機能の継続の検討

**Process 制約内容**: コア知識管理機能を中心とした基本機能の継続動作により、外部 API 障害時でも知識体系の閲覧・編集・保存を維持する。

**Development 制約との適合性**:

部分機能の継続は Backend Container 内でのモジュール分離により実現可能である。コア知識管理機能を独立したモジュールとして実装し、外部 API 依存機能と分離することで継続動作を確保できる。

**使用技術および設定**:

- Backend Container: モジュール分離による機能独立
- Python + FastAPI: 機能別モジュール実装

### 手動復旧を基本の検討

**Process 制約内容**: 自動復旧機構の複雑さを回避し、明確な手動復旧手順と充実したエラー情報により個人での障害対応を支援する。

**Development 制約との適合性**:

手動復旧は Backend Container の loguru により実現可能である。充実したエラー情報の出力とログ記録により、個人での障害対応を支援できる。複雑な自動復旧機構は不要である。

**使用技術および設定**:

- loguru: エラー情報出力・ログ記録
- Backend Container: 手動復旧手順の実装

### 明確なエラー伝達の検討

**Process 制約内容**: 障害種別・影響範囲・対応手順を分かりやすく表示し、開発者自身の技術的判断を支援する。

**Development 制約との適合性**:

明確なエラー伝達は Backend Container の loguru と Fluent Bit により実現可能である。障害種別・影響範囲・対応手順の情報を構造化してログ出力し、開発者の技術的判断を支援できる。

**使用技術および設定**:

- loguru: 構造化エラー情報出力
- Fluent Bit: ログ収集・転送

## Development 制約との適合性検討結果の抽出

各耐障害性要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術を明確化する。

- データ保護を最優先の検討
  - SQLite: ファイルベースデータベースでの定期バックアップ
  - ファイルシステム: バックアップファイルの検証
- 部分機能の継続の検討
  - Backend Container: モジュール分離による機能独立
  - Python + FastAPI: 機能別モジュール実装
- 手動復旧を基本の検討
  - loguru: エラー情報出力・ログ記録
  - Backend Container: 手動復旧手順の実装
- 明確なエラー伝達の検討
  - loguru: 構造化エラー情報出力
  - Fluent Bit: ログ収集・転送

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-fault-tolerance-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- SQLite
  - ファイルベースデータベースでの定期バックアップ
- ファイルシステム
  - バックアップファイルの検証
- Backend Container
  - モジュール分離による機能独立
  - 手動復旧手順の実装
- Python + FastAPI
  - 機能別モジュール実装
- loguru
  - エラー情報出力・ログ記録
  - 構造化エラー情報出力
- Fluent Bit
  - ログ収集・転送

<!-- GLOBAL_CONCLUSION_END: inter-process-fault-tolerance-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない耐障害性要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: inter-process-fault-tolerance-missing-technologies -->

現在の技術スタックでは全ての耐障害性設計要素が実現可能である。不足している技術要素はない。

<!-- GLOBAL_CONCLUSION_END: inter-process-fault-tolerance-missing-technologies -->
