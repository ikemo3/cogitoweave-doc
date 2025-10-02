---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "orm-selection"
    - "http-client-tech-selection"
    - "config-management-tech-selection"
    - "testing-automation-requirements"
  produces:
    - "backend-testing-tech-selection"
---

# バックエンドテストツール選択

## 概要

Python + FastAPI エコシステムにおけるテストフレームワーク・モックツールの選択。3 つのバックエンドサービス全体を対象とし、ほぼ 100%の自動テストカバレッジと CI/CD 統合を前提とした最適解を決定する。

### 前提

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: orm-selection -->

ORM/ODM ツールとして SQLAlchemy + Alembic を採用する。

<!-- PREMISE_END: orm-selection -->

<!-- PREMISE_BEGIN: http-client-tech-selection -->

HTTP クライアントライブラリとして httpx を採用する。

<!-- PREMISE_END: http-client-tech-selection -->

<!-- PREMISE_BEGIN: config-management-tech-selection -->

バックエンドの設定管理技術として環境変数方式を採用し、os.getenv による実装を行う。

<!-- PREMISE_END: config-management-tech-selection -->

<!-- PREMISE_BEGIN: testing-automation-requirements -->

- **バックエンドは完全に自動テスト**: ほぼ 100%のテストで品質を担保する
- **フロントエンドは状況に応じて使い分け**: ブラウザでの複雑な操作やデザインの確認は手動テスト、ロジック部分は自動テストを行う
- **障害対応をスクリプト化**: 復旧作業を自動化して保守の負担を減らす
- **データ移行を自動化**: データベース構造を変更するときは自動で処理する

<!-- PREMISE_END: testing-automation-requirements -->

## 選択肢と判断基準

3 つのバックエンドサービス(コア知識管理・LLM 統合・コンテンツ取得)の自作部分を対象とし、外部依存はモック化、SQLite は実 DB 使用での統合テスト環境を構築する。

### 対象テスト範囲

- **単体テスト**: 各サービスの個別機能・クラス・関数レベル
- **統合テスト**: API エンドポイント、データベース操作、サービス間連携
- **パフォーマンステスト**: 応答時間制約・負荷テスト (一応含むが後で除外予定)
- **外部依存モック**: LLM API・Web スクレイピング対象サイト
- **除外範囲**: E2E テスト(フロントエンド側で実施)、外部ライブラリの内部動作

### 環境構成

- **テスト実行環境**: CI/CD での自動実行 + ローカル開発環境
- **データベース**: SQLite インメモリ(:memory:)による高速テスト実行
- **外部 API**: モックサーバー・レスポンス固定による安定テスト
- **カバレッジ目標**: ほぼ 100%(ただし例外あり、100%は目指さない)

### 判断基準

- FastAPI 親和性 (優先度最高) - FastAPI エコシステムとの統合の自然さ
- Python 標準性 (優先度高) - Python 標準的なテスト手法への適合
- モック機能 (優先度高) - 外部依存の効果的なモック化能力
- CI/CD 統合 (優先度高) - 自動実行・レポート生成の容易さ
- 保守性 - 一人開発における理解しやすさと管理負荷
- パフォーマンス - テスト実行速度と開発効率

### 選択肢

- pytest エコシステム: Python 標準的なテストフレームワーク群
- unittest + mock: Python 標準ライブラリベース
- FastAPI TestClient: FastAPI 専用テストクライアント
- 外部テストサービス: SaaS 型テスト実行環境

## 各選択肢の詳細分析

Python + FastAPI 環境での包括的テスト体制構築に向けた主要選択肢を評価する。

### pytest エコシステム

Python における事実上の標準テストフレームワークとその関連ツール群。

- 利点
  - Python テスト界のデファクトスタンダード
  - 豊富なプラグインエコシステム(pytest-asyncio, pytest-cov, pytest-mock 等)
  - FastAPI との統合が自然(TestClient 対応)
  - パラメータ化テスト・フィクスチャによる効率的テスト記述
  - カバレッジレポート・CI 統合が標準的
  - SQLAlchemy テストパターンが確立
- 欠点
  - 複数プラグインの組み合わせによる設定複雑化
  - 高機能ゆえの学習コストがある
  - プラグイン依存による保守負荷

FastAPI 対応:

- pytest-asyncio による非同期テスト対応
- TestClient による API エンドポイントテスト
- pytest-mock による外部依存モック化

### unittest + mock

Python 標準ライブラリによるテストフレームワークとモック機能。

- 利点
  - Python 標準ライブラリで外部依存なし
  - 学習リソースが豊富で安定している
  - unittest.mock による標準的モック機能
  - シンプルで理解しやすい構造
  - 長期的な互換性が保証される
- 欠点
  - 現代的なテスト記述パターンに対応困難
  - パラメータ化テスト・フィクスチャが貧弱
  - FastAPI との統合に手動実装が必要
  - 非同期テストサポートが限定的
  - カバレッジ・CI 統合に追加作業

FastAPI 対応:

- TestClient の手動統合が必要
- 非同期テストは asyncio.run での実装
- モック化は unittest.mock で対応可能

### FastAPI TestClient

FastAPI 専用のテストクライアントとエコシステム統合。

- 利点
  - FastAPI との完全統合による自然な API テスト
  - Starlette TestClient ベースの高品質実装
  - 非同期対応・リクエスト/レスポンステストが簡潔
  - FastAPI アプリケーション全体のテスト実行
  - dependency_overrides による依存関係モック化
- 欠点
  - FastAPI 専用でテストフレームワーク機能は限定的
  - 単体テスト・モック機能は別途必要
  - 複雑なテストシナリオには機能不足
  - パフォーマンステストには対応困難

FastAPI 対応:

- TestClient そのものが FastAPI 専用
- pytest との併用が一般的
- 依存関係の上書きによる効率的モック化

### 外部テストサービス

GitHub Actions、CircleCI 等の SaaS 型 CI/CD でのテスト実行環境。

- 利点
  - クラウド実行による環境の一貫性
  - 並列実行・分散テストによる高速化
  - 高度なレポート・通知機能
  - インフラ管理負荷の軽減
- 欠点
  - 外部依存による実行コスト・障害リスク
  - ローカル開発環境との差異発生可能性
  - 一人開発には過剰なスペック
  - デバッグ・トラブルシューティングが困難

FastAPI 対応:

- コンテナベースでの実行環境構築
- テストフレームワークは別途選択が必要
- 結果の可視化・通知機能が充実

## 判断基準による評価

### FastAPI 親和性の比較

pytest + FastAPI TestClient の組み合わせが最も自然で効率的である。pytest の豊富な機能と TestClient の FastAPI 専用最適化により、API テスト・非同期処理・依存関係モックが統合的に実現される。

### Python 標準性の比較

pytest は Python テスト界のデファクトスタンダードとして、最も広く採用されている。豊富な学習リソース・コミュニティサポート・企業採用実績により、長期的な技術選択として安全である。

### モック機能の比較

pytest-mock (unittest.mock のラッパー) により、Python 標準のモック機能を pytest の記述スタイルで効率的に利用できる。外部 API・ファイルシステム・時間等の幅広いモック化に対応し、テストの安定性と実行速度を両立する。

### CI/CD 統合の比較

pytest はカバレッジレポート・JUnit XML 出力・並列実行等の CI/CD 統合機能が充実している。GitHub Actions・CircleCI 等での標準的な実行パターンが確立され、自動テスト体制構築の負荷が最小化される。

### 保守性の比較

pytest エコシステムは一人開発制約下でも理解しやすく、段階的な機能追加が可能である。基本的なテストから開始して、必要に応じてプラグイン追加・設定カスタマイズで拡張できる柔軟性がある。

### パフォーマンスの比較

pytest の並列実行機能(pytest-xdist)と SQLite インメモリ実行により、高速なテスト実行が実現される。開発効率を重視した設計で、頻繁なテスト実行による開発体験向上が期待できる。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: backend-testing-tech-selection -->

バックエンドテストツールとして以下を採用する：

- pytest
- FastAPI TestClient
- pytest-asyncio
- pytest-mock
- httpx-mock
- time-machine
- pytest-cov
- pytest-xdist

<!-- GLOBAL_CONCLUSION_END: backend-testing-tech-selection -->

### 選択根拠

Python + FastAPI エコシステムでの最適解として、pytest の豊富な機能と FastAPI TestClient の専用最適化を組み合わせることが決定要因である。Python テスト界のデファクトスタンダードによる学習・保守負荷の軽減、豊富なプラグインエコシステムによる機能拡張性、CI/CD 統合の標準化により、ほぼ 100%のテストカバレッジと自動実行体制を効率的に構築できる。SQLite インメモリと httpx-mock による HTTP モック機能の組み合わせで、外部依存を排除した安定・高速なテスト環境を実現する。

### 技術スタック詳細

#### コアテストフレームワーク

- **pytest**: メインテストフレームワーク
- **pytest-asyncio**: 非同期処理テスト対応
- **FastAPI TestClient**: API エンドポイントテスト

#### モック・テストユーティリティ

- **pytest-mock**: モック機能(unittest.mock のラッパー)
- **httpx-mock**: HTTP リクエストモック化
- **time-machine**: 時間モック化(日時依存テスト)

#### カバレッジ・レポート

- **pytest-cov**: カバレッジ計測・レポート生成
- **coverage[toml]**: カバレッジ設定・HTML レポート

#### パフォーマンス・CI 統合

- **pytest-xdist**: 並列テスト実行
- **pytest-html**: HTML テストレポート生成

#### データベーステスト

- **SQLite インメモリ(:memory:)**: 高速テスト実行
- **SQLAlchemy テストパターン**: セッション管理・ロールバック

### 実装方針

#### モック戦略

- **LLM API**: httpx-mock による HTTP レスポンス固定
- **Web スクレイピング**: httpx-mock + HTML ファイル固定
- **時間依存**: time-machine による固定時刻テスト
- **設定**: pytest fixture による環境変数上書き

#### テスト実行方針

- **データベース**: SQLite インメモリ(:memory:)による高速実行
- **並列実行**: pytest-xdist による効率的テスト実行
- **カバレッジ**: pytest-cov による測定とレポート生成
- **CI 統合**: 自動実行とレポート出力の標準化
