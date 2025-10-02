---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "batch-processing-design"
  produces:
    - "intra-process-batch-processing-constraint-adjustment"
    - "intra-process-batch-processing-technology-mapping"
    - "intra-process-batch-processing-missing-technologies"
---

# サービス内 Process 制約の見直し - バッチ処理設計

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

### バッチ処理設計

<!-- PREMISE_BEGIN: batch-processing-design -->

- **深夜単発バックアップ**: 日次データバックアップを深夜時間帯に単一バッチとして実行し、全データの完全バックアップを取得する。
- **既存通知との連携**: 既存のバックアップ通知仕組みと連携し、失敗時のメール通知により迅速な障害対応を支援する。
- **シンプルなスケジューリング**: 複雑なスケジューラーや並列処理は実装せず、確実な実行を基本とする。
- **データ整合性の確保**: バックアップ実行時点での一貫性を確保し、実行中のユーザーアクセスは停止しない前提で設計する。

<!-- PREMISE_END: batch-processing-design -->

## Physical 制約との適合性検討

Physical・Development 制約では Backend Container と Database Container での実装が確定している。

Process 制約「バッチ処理設計」は Backend Container での Python スクリプト実行と Database Container の SQLite データバックアップを前提としており、Physical 制約と完全に整合している。矛盾は発生しない。

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-batch-processing-constraint-adjustment -->

- **loguru によるログ出力**: バックアップ処理の成功・失敗をログとして出力し、迅速な障害対応を支援する。

<!-- GLOBAL_CONCLUSION_END: intra-process-batch-processing-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 深夜単発バックアップの検討

**Process 制約内容**: 日次データバックアップを深夜時間帯に単一バッチとして実行し、全データの完全バックアップを取得する。

**Development 制約との適合性**:

日次データバックアップは Python スクリプトによる SQLite データベースファイルのコピー処理で実現可能である。深夜時間帯の実行は cron ジョブまたはシステムタスクスケジューラーによる定期実行で実現可能である。全データの完全バックアップは SQLite の単一ファイル特性を活用したファイルコピーで効率的に実現できる。

**使用技術および設定**:

- Python: バックアップスクリプト実行
- SQLite: 単一ファイルデータベースのバックアップ
- **[技術不足]**: スケジューラー（cron、systemd timer 等）

### loguru によるログ出力の検討

**Process 制約内容**: バックアップ処理の成功・失敗をログとして出力し、迅速な障害対応を支援する。

**Development 制約との適合性**:

loguru によるログ出力は現在の技術スタックに含まれており、バックアップ処理の成功・失敗を適切なログレベル（INFO、ERROR）で出力可能である。ファイル出力とローテーション機能により長期的なログ保存も実現可能である。迅速な障害対応のためのログ検索・集約も loguru の機能で基本的に対応できる。

**使用技術および設定**:

- loguru: バックアップ処理ログ出力
- loguru: ログレベル管理（INFO=成功、ERROR=失敗）
- loguru: ファイル出力とローテーション

### シンプルなスケジューリングの検討

**Process 制約内容**: 複雑なスケジューラーや並列処理は実装せず、確実な実行を基本とする。

**Development 制約との適合性**:

シンプルなスケジューリングは外部スケジューラー（cron、systemd timer）による単一プロセス実行で実現可能である。複雑なスケジューラーの除外により、Python の標準的なスクリプト実行のみに依存した確実な実行を実現できる。並列処理の除外により、単一プロセスでの順次処理による理解しやすい構成を実現できる。

**使用技術および設定**:

- Python: 単一プロセススクリプト実行
- **[技術不足]**: 外部スケジューラー（cron、systemd timer 等）

### データ整合性確保の検討

**Process 制約内容**: バックアップ実行時点での一貫性を確保し、実行中のユーザーアクセスは停止しない前提で設計する。

**Development 制約との適合性**:

データ整合性確保は SQLAlchemy による読み取り専用トランザクションでバックアップ時点の一貫性を保証可能である。SQLite の WAL（Write-Ahead Logging）モードにより、バックアップ実行中でもユーザーアクセスを継続できる。Python の shutil.copy2 による原子的ファイルコピーで確実なバックアップを実現できる。

**使用技術および設定**:

- SQLAlchemy: 読み取り専用トランザクション
- SQLite: WAL モードによる並行アクセス対応
- Python: shutil.copy2 による原子的ファイルコピー

## Development 制約との適合性検討結果の抽出

各要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 深夜単発バックアップの検討
  - Python: バックアップスクリプト実行
  - SQLite: 単一ファイルデータベースのバックアップ
  - **[技術不足]**: スケジューラー（cron、systemd timer 等）
- loguru によるログ出力の検討
  - loguru: バックアップ処理ログ出力
  - loguru: ログレベル管理（INFO=成功、ERROR=失敗）
  - loguru: ファイル出力とローテーション
- シンプルなスケジューリングの検討
  - Python: 単一プロセススクリプト実行
  - **[技術不足]**: 外部スケジューラー（cron、systemd timer 等）
- データ整合性確保の検討
  - SQLAlchemy: 読み取り専用トランザクション
  - SQLite: WAL モードによる並行アクセス対応
  - Python: shutil.copy2 による原子的ファイルコピー

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-batch-processing-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- Python
  - バックアップスクリプト実行
  - 単一プロセススクリプト実行
  - shutil.copy2 による原子的ファイルコピー
- SQLite
  - 単一ファイルデータベースのバックアップ
  - WAL モードによる並行アクセス対応
- SQLAlchemy
  - 読み取り専用トランザクション
- loguru
  - バックアップ処理ログ出力
  - ログレベル管理（INFO=成功、ERROR=失敗）
  - ファイル出力とローテーション

<!-- GLOBAL_CONCLUSION_END: intra-process-batch-processing-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できない要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-batch-processing-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- スケジューラー: cron、systemd timer、Task Scheduler 等の定期実行機構

<!-- GLOBAL_CONCLUSION_END: intra-process-batch-processing-missing-technologies -->
