---
doc_type: "pattern-process-technology-mapping"
status: "draft"
depends:
  contracts:
    - "development-constraint-simplified-list"
    - "backend-test-strategy"
    - "frontend-test-strategy"
    - "api-contract-test-strategy"
  produces:
    - "intra-process-test-constraint-adjustment"
    - "intra-service-test-technology-mapping"
    - "intra-service-test-missing-technologies"
---

# サービス内 Process 制約の見直し - サービス内テスト戦略

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

### バックエンドテスト戦略決定

<!-- PREMISE_BEGIN: backend-test-strategy -->

- **サービス特性別テスト**: コア知識管理サービスはデータベース操作とトランザクション境界を検証し、LLM 統合サービスは外部 API の契約テストと異常ケース（タイムアウト、エラーレスポンス）対応を実装し、コンテンツ取得サービスは構造変更への適応性を検証する。
- **バックエンドテスト環境**: インメモリ SQLite を使用した高速なテスト実行とテスト毎のクリーンな状態確保、HTTP 通信レベルでの外部 API モック化による安定したテスト実行を実現する。

<!-- PREMISE_END: backend-test-strategy -->

### フロントエンドテスト戦略決定

<!-- PREMISE_BEGIN: frontend-test-strategy -->

- **フロントエンドユニットテスト**: ユーティリティ関数やビジネスロジック関数の正確性を検証し、データ変換・バリデーション・計算処理等の純粋関数の動作を検証する。
- **フロントエンドコンポーネントテスト**: 個別のコンポーネントの動作と表示の正確性を検証し、プロパティによる表示変化・イベントハンドリング・内部状態管理を検証する。
- **フロントエンド統合テスト**: 複数のコンポーネント間の連携を検証し、状態管理・画面遷移・ルーティング・イベント処理の統合動作を検証する。
- **フロントエンドテスト環境**: ユニットテスト用の軽量実行環境・コンポーネントテスト用の DOM 環境・統合テスト用のルーティング状態管理環境を提供し、バックエンド API 依存をモック化する。

<!-- PREMISE_END: frontend-test-strategy -->

### API 契約テスト戦略決定

<!-- PREMISE_BEGIN: api-contract-test-strategy -->

- **フロントエンド・バックエンド間 API 契約**: リクエスト・レスポンス仕様を明文化し、インターフェース契約として文書化する。バックエンド側での API 仕様準拠検証とフロントエンド側での TypeScript 型定義による契約検証により、API 仕様変更の早期検知を実現する。

<!-- PREMISE_END: api-contract-test-strategy -->

## Physical 制約との適合性検討

Physical・Development 制約では単一 Backend Container での実装が確定している。

Process 制約「サービス特性別テスト」は「3 つのサービス」を前提としているが、Physical 制約では単一バックエンドコンテナでの実装となるため矛盾が発生する。

- **サービス特性別テスト**: コア知識管理サービスはデータベース操作とトランザクション境界を検証し、LLM 統合サービスは外部 API の契約テストと異常ケース対応を実装し、コンテンツ取得サービスは構造変更への適応性を検証する。

## Physical 制約との調整

この制約を Physical 制約に適合させるため、次のように変更した。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-process-test-constraint-adjustment -->

- **単一バックエンド内での機能別テスト**: コア知識管理機能はデータベース操作とトランザクション境界を検証し、LLM 統合機能は外部 API の契約テストと異常ケース対応を実装し、コンテンツ取得機能は構造変更への適応性を検証する。

<!-- GLOBAL_CONCLUSION_END: intra-process-test-constraint-adjustment -->

## Development 制約との適合性検討

**完全引用のルール**: Process 制約内容は要約せず、元の文書から完全に引用する。これにより誤解や解釈の相違を防ぎ、正確な制約内容に基づいた技術適用の検討を行う。

Process 制約の各テスト戦略要素について、Development 制約との適合性を検討し、技術要素への対応を確認する。

### 単一バックエンド内での機能別テストの検討

**Process 制約内容**: コア知識管理サービスはデータベース操作とトランザクション境界を検証し、LLM 統合サービスは外部 API の契約テストと異常ケース対応を実装し、コンテンツ取得サービスは構造変更への適応性を検証する。

**Development 制約との適合性**:

コア知識管理サービスのデータベース操作検証は SQLAlchemy による ORM 操作テストとして pytest により実現可能である。トランザクション境界検証については、SQLAlchemy の基本的なトランザクション制御は可能だが、詳細なトランザクション境界の検証には pytest-sqlalchemy 等の専用ライブラリが必要であり、現在の技術スタックでは不足している。

LLM 統合サービスの外部 API 契約テストは httpx による API 呼び出しを pytest-mock + httpx-mock でモック化し、期待するレスポンス形式の検証として実現可能である。異常ケース対応（タイムアウト、エラーレスポンス）は httpx-mock でタイムアウト例外や 4xx/5xx ステータスコードを発生させることで実現可能である。

コンテンツ取得サービスの構造変更適応性検証は httpx による Web スクレイピング処理を pytest でテストし、HTML 構造変化への対応を検証可能である。

**使用技術および設定**:

- pytest: ユニットテスト・統合テスト実行フレームワーク
- FastAPI TestClient: FastAPI アプリケーションのテスト環境
- SQLAlchemy: ORM によるデータベース操作
- pytest-asyncio: 非同期処理のテスト実行
- pytest-mock: モック機能提供
- httpx-mock: HTTP 通信のモック化とタイムアウト・エラーレスポンステスト
- time-machine: 時間関連処理のモック化（必要に応じて使用）
- **[技術不足]**: pytest-sqlalchemy 等のトランザクション境界検証専用ライブラリ

### バックエンドテスト環境の検討

**Process 制約内容**: インメモリ SQLite を使用した高速なテスト実行とテスト毎のクリーンな状態確保、HTTP 通信レベルでの外部 API モック化による安定したテスト実行を実現する。

**Development 制約との適合性**:

インメモリ SQLite を使用した高速なテスト実行は、SQLite データベースと SQLAlchemy による ORM 環境で実現可能である。テスト毎のクリーンな状態確保は pytest の fixture 機能と SQLAlchemy のセッション管理により実現可能である。

HTTP 通信レベルでの外部 API モック化は httpx + pytest-mock + httpx-mock により実現可能である。安定したテスト実行は FastAPI TestClient による独立したテスト環境構築で実現可能である。

**使用技術および設定**:

- SQLite: インメモリデータベース
- SQLAlchemy: ORM とセッション管理
- pytest: fixture によるテスト環境分離
- FastAPI TestClient: 独立したテスト環境
- httpx-mock: 外部 API モック化
- pytest-mock: モック機能提供

### フロントエンドユニットテストの検討

**Process 制約内容**: ユーティリティ関数やビジネスロジック関数の正確性を検証し、データ変換・バリデーション・計算処理等の純粋関数の動作を検証する。

**Development 制約との適合性**:

ユーティリティ関数やビジネスロジック関数の正確性検証は Vitest による軽量な JavaScript/TypeScript 関数テスト実行で実現可能である。データ変換・バリデーション・計算処理等の純粋関数の動作検証も Vitest の単体テスト機能により実現可能である。

**使用技術および設定**:

- Vitest: 軽量な JavaScript/TypeScript テスト実行フレームワーク

### フロントエンドコンポーネントテストの検討

**Process 制約内容**: 個別のコンポーネントの動作と表示の正確性を検証し、プロパティによる表示変化・イベントハンドリング・内部状態管理を検証する。

**Development 制約との適合性**:

個別のコンポーネントの動作と表示の正確性検証は Vitest + Testing Library による Solid コンポーネントのレンダリングテストで実現可能である。ただし DOM 環境が必要であり、jsdom または happy-dom による DOM シミュレーションが必要である。プロパティによる表示変化・イベントハンドリング・内部状態管理の検証も Testing Library の DOM 操作とイベント発火機能により実現可能である。

**使用技術および設定**:

- Vitest: テスト実行フレームワーク
- Testing Library: Solid コンポーネントのレンダリング・イベント・状態テスト
- **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）

### フロントエンド統合テストの検討

**Process 制約内容**: 複数のコンポーネント間の連携を検証し、状態管理・画面遷移・ルーティング・イベント処理の統合動作を検証する。

**Development 制約との適合性**:

複数のコンポーネント間の連携検証は Vitest + Testing Library による複数コンポーネント統合テストで実現可能である。状態管理・イベント処理の統合動作検証も Testing Library により実現可能である。画面遷移・ルーティングの検証については TanStack Router のテスト手法が必要であり、現在の技術スタックでは具体的な手法が不明である。DOM 環境シミュレーションも必要である。

**使用技術および設定**:

- Vitest: テスト実行フレームワーク
- Testing Library: 複数コンポーネント統合テスト
- **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）
- **[技術不足]**: TanStack Router のテスト環境構築手法

### フロントエンドテスト環境の検討

**Process 制約内容**: ユニットテスト用の軽量実行環境・コンポーネントテスト用の DOM 環境・統合テスト用のルーティング状態管理環境を提供し、バックエンド API 依存をモック化する。

**Development 制約との適合性**:

ユニットテスト用の軽量実行環境は Vitest により実現可能である。コンポーネントテスト用の DOM 環境は jsdom または happy-dom による DOM シミュレーションが必要であり、現在の技術スタックでは不足している。統合テスト用のルーティング状態管理環境は TanStack Router のテスト環境構築手法が必要であり、現在の技術スタックでは具体的な手法が不明である。バックエンド API 依存のモック化についても具体的な手法が不明である。

**使用技術および設定**:

- Vitest: 軽量テスト実行環境
- **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）
- **[技術不足]**: TanStack Router のテスト環境構築手法
- **[技術不足]**: API モック化の具体的手法

### 契約テストの検討

**Process 制約内容**: フロントエンド・バックエンド間 API 契約のリクエスト・レスポンス仕様を明文化し、インターフェース契約として文書化する。バックエンド側での API 仕様準拠検証とフロントエンド側での TypeScript 型定義による契約検証により、API 仕様変更の早期検知を実現する。

**Development 制約との適合性**:

API 仕様の明文化は FastAPI 内蔵 Swagger UI・ReDoc により実現可能である。バックエンド側での API 仕様準拠検証は FastAPI の Pydantic バリデーションと pytest による API テストで実現可能である。

フロントエンド側での TypeScript 型定義による契約検証については、OpenAPI 仕様から TypeScript 型定義を自動生成するツール（openapi-typescript 等）が必要であり、現在の技術スタックでは不足している。

API 仕様変更の早期検知については、OpenAPI 仕様の変更検知やスキーマ比較の自動化手法が必要であり、現在の技術スタックでは具体的な手法が不明である。

**使用技術および設定**:

- FastAPI: 内蔵 Swagger UI・ReDoc による API 仕様生成
- Pydantic: バリデーションによる仕様準拠検証
- pytest: API テスト実行
- **[技術不足]**: openapi-typescript 等の TypeScript 型生成ツール
- **[技術不足]**: API 仕様変更の自動検知手法

## Development 制約との適合性検討結果の抽出

各テスト戦略要素の技術適合性結果を整理する。各項目の「使用技術および設定」の結論部分のみを抜き出し、実現可能な技術と技術不足の項目を明確化する。

- 単一バックエンド内での機能別テストの検討
  - pytest: ユニットテスト・統合テスト実行フレームワーク
  - FastAPI TestClient: FastAPI アプリケーションのテスト環境
  - SQLAlchemy: ORM によるデータベース操作
  - pytest-asyncio: 非同期処理のテスト実行
  - pytest-mock: モック機能提供
  - httpx-mock: HTTP 通信のモック化とタイムアウト・エラーレスポンステスト
  - time-machine: 時間関連処理のモック化（必要に応じて使用）
  - **[技術不足]**: pytest-sqlalchemy 等のトランザクション境界検証専用ライブラリ
- バックエンドテスト環境の検討
  - SQLite: インメモリデータベース
  - SQLAlchemy: ORM とセッション管理
  - pytest: fixture によるテスト環境分離
  - FastAPI TestClient: 独立したテスト環境
  - httpx-mock: 外部 API モック化
  - pytest-mock: モック機能提供
- フロントエンドユニットテストの検討
  - Vitest: 軽量な JavaScript/TypeScript テスト実行フレームワーク
- フロントエンドコンポーネントテストの検討
  - Vitest: テスト実行フレームワーク
  - Testing Library: Solid コンポーネントのレンダリング・イベント・状態テスト
  - **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）
- フロントエンド統合テストの検討
  - Vitest: テスト実行フレームワーク
  - Testing Library: 複数コンポーネント統合テスト
  - **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）
  - **[技術不足]**: TanStack Router のテスト環境構築手法
- フロントエンドテスト環境の検討
  - Vitest: 軽量テスト実行環境
  - **[技術不足]**: jsdom または happy-dom（DOM 環境シミュレーション）
  - **[技術不足]**: TanStack Router のテスト環境構築手法
  - **[技術不足]**: API モック化の具体的手法
- 契約テストの検討
  - FastAPI: 内蔵 Swagger UI・ReDoc による API 仕様生成
  - Pydantic: バリデーションによる仕様準拠検証
  - pytest: API テスト実行
  - **[技術不足]**: openapi-typescript 等の TypeScript 型生成ツール
  - **[技術不足]**: API 仕様変更の自動検知手法

## Development 制約の技術単位での整理

先ほどのリストを「Process View の結論 → 技術」順から「技術 → Process View の結論」順に並び替える。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-service-test-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- pytest
  - ユニットテスト・統合テスト実行フレームワーク
  - fixture によるテスト環境分離
  - API テスト実行
- FastAPI TestClient
  - FastAPI アプリケーションのテスト環境
  - 独立したテスト環境
- SQLAlchemy
  - ORM によるデータベース操作
  - ORM とセッション管理
- SQLite
  - インメモリデータベース
- pytest-asyncio
  - 非同期処理のテスト実行
- pytest-mock
  - モック機能提供
- httpx-mock
  - HTTP 通信のモック化とタイムアウト・エラーレスポンステスト
  - 外部 API モック化
- time-machine
  - 時間関連処理のモック化（必要に応じて使用）
- Vitest
  - 軽量な JavaScript/TypeScript テスト実行フレームワーク
  - テスト実行フレームワーク
  - 軽量テスト実行環境
- Testing Library
  - Solid コンポーネントのレンダリング・イベント・状態テスト
  - 複数コンポーネント統合テスト
- FastAPI
  - 内蔵 Swagger UI・ReDoc による API 仕様生成
- Pydantic
  - バリデーションによる仕様準拠検証

<!-- GLOBAL_CONCLUSION_END: intra-service-test-technology-mapping -->

## Development 制約の不足技術

現在の技術スタックでは実現できないテスト戦略要素について、不足している技術とその例を整理する。

<!-- GLOBAL_CONCLUSION_BEGIN: intra-service-test-missing-technologies -->

現在の技術スタックでは以下の技術要素が不足している。

- トランザクション境界検証専用ライブラリ: pytest-sqlalchemy 等
- DOM 環境シミュレーション: jsdom または happy-dom
- TanStack Router のテスト環境構築手法: 具体的な構築方法が不明
- API モック化の具体的手法: フロントエンド側での具体的なモック手法が不明
- TypeScript 型生成ツール: openapi-typescript 等
- API 仕様変更の自動検知手法: 具体的な検知方法が不明

<!-- GLOBAL_CONCLUSION_END: intra-service-test-missing-technologies -->
