# Development View

## 1. フロントエンド 技術選択

### 1. 状態管理方針決定

<!-- REFERENCE_BEGIN: state-management-policy -->

CogitoWeave の状態管理方針として以下を採用する。

- 画面間状態共有は実装しない。
  - 各画面は独立したワークフローを持つため、状態共有の必要性がない。
- 各画面で独立した状態管理を行う。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- 編集中データは一時的にローカルストレージに保存する。
  - ユーザーの誤操作によるデータ消失を防止する。

<!-- REFERENCE_END: state-management-policy -->

### 2. パフォーマンス最適化方針決定

<!-- REFERENCE_BEGIN: performance-optimization-policy -->

CogitoWeave のパフォーマンス最適化方針として以下を採用する。

- 概念ページ読み込み 1.5 秒以内を最優先とする。
  - 最も頻繁な日常操作の快適性確保を最重要視する。
- 初回ロードの遅さは許容し、ページ遷移の高速性を重視する。
  - CSR 特性を活かした継続利用の効率性を優先する。
- 複雑なパフォーマンス最適化は実装しない。
  - 軽量なコンポーネント指向アーキテクチャにより設計段階で最適化を行う。
- ツリーシェイキング対応ライブラリを優先する。
  - 元々軽量で最適化しやすい技術を選択し、開発効率と軽量性を両立する。

<!-- REFERENCE_END: performance-optimization-policy -->

### 3. アプリケーション構造評価

<!-- REFERENCE_BEGIN: app-structure-evaluation -->

CogitoWeave のアプリケーション構造として以下を採用する。

- 基本構造は MPA または SPA のいずれも可能とする。
  - 画面間状態共有が不要で、各画面の独立性を重視するため、どちらの技術でも要件を満たせる。
- 概念間移動の高速化を最優先とする。
  - 概念ページ読み込み最優先要件を満たすため、技術選択に応じて最適な実装を行う。
- 各画面の独立性を確保する。
  - MPA 選択時は別ページ遷移、SPA 選択時は状態管理の分離により実現する。
- 3 画面という小規模構成を活かしたシンプルな設計とする。
  - 一人開発制約下での理解しやすさと保守性を維持するため。

<!-- REFERENCE_END: app-structure-evaluation -->

### 4. フレームワーク調査

<!-- REFERENCE_BEGIN: framework-research -->

State of JS 2024 のデータに基づくフレームワーク候補の調査結果は以下の通りである。

#### フレームワーク最優先候補

- **Preact**: 軽量な React 代替として満足度 80%、期待値ギャップ 55%（最高）を記録し、React エコシステムを活用できる。
- **Vue**: バランスの取れたフレームワークとして満足度 87%（大幅上昇）、使用率 51%（適度）を示し、学習曲線も緩やかである。
- **Svelte**: 高満足度 88% を誇り、コンパイル時最適化による軽量性を提供するが、関心度は減少傾向にある。

#### フレームワーク次点候補

- **Solid**: 最高満足度 90%、期待値ギャップ 35% を記録する革新的なフレームワークだが、使用率 9%で情報が限定的である。
- **React 18**: 使用率 82%で情報は豊富であり、React 19 の CSR における Suspense 遅延バグを回避できる安定版として選択可能である。

#### フレームワーク要注意候補

- **React 19**: React の最新版だが CSR における Suspense の[強制 300ms 遅延バグ](https://github.com/facebook/react/issues/31819)が発生し、概念ページの読み込み要件に深刻な影響を与える。

軽量なコンポーネント指向アーキテクチャ、一人開発の制約、パフォーマンスの要件を総合すると、Preact、Vue、Svelte が最有力の候補として浮上し、Solid と React 18 が次点の候補、React 19 が要注意の候補となる。

<!-- REFERENCE_END: framework-research -->

### 5. メタフレームワーク調査

<!-- REFERENCE_BEGIN: metaframework-research -->

State of JS 2024 のデータに基づくメタフレームワーク候補の調査結果は以下の通りである。

#### メタフレームワーク最優先候補

- **Astro**: 最高満足度 94%、関心度 67%（最高）を記録し、軽量で柔軟なハイブリッドアプローチに対応する。
- **SvelteKit**: 高満足度 90%、期待値ギャップ 35% を示し、Svelte の軽量性を活かしたメタフレームワークである。

#### メタフレームワーク次点候補

- **Nuxt**: Vue の生態系において満足度 81%、期待値ギャップ 42%（大幅改善）を達成し、安定したハイブリッド対応を提供する。

#### メタフレームワーク除外候補

- **Next.js**: 満足度が 89%（2022 年）→ 76%（2023 年）→ 68%（2024 年）と 3 年連続で急激に下落しており、開発者からの信頼を完全に失っている。App Router の導入による既存 API の大幅破壊、Vercel エコシステムへの強制的な依存、複雑性の爆発的増大により、シンプルさを重視する一人開発には最悪の選択肢となった。フレームワーク自体の肥大化によりパフォーマンス要件も満たせず、技術的に他の選択肢が明らかに優位である。

軽量なコンポーネント指向アーキテクチャ、CSR の前提、一人開発の制約、パフォーマンスの要件を総合すると、Astro と SvelteKit が最有力の候補として浮上し、Nuxt が次点の候補として選定される。

#### State of JS 2024 調査対象外の例外候補

- **React Router 7**: Remix との統合により従来のルーティングライブラリがメタフレームワーク機能を獲得した選択肢だが、State of JS 2024 の調査対象に含まれていない理由は不明である。CSR 前提とハイブリッドアプローチには適合する可能性がある。
- **TanStack Router**: 型安全性に重点を置いた純粋なクライアントサイドルーティングライブラリとして、メタフレームワークを使わない軽量な選択肢を提供する。Vite 等との組み合わせで CSR 前提の要件に完全適合する可能性がある。

<!-- REFERENCE_END: metaframework-research -->

### 6. ビルドツール調査

<!-- REFERENCE_BEGIN: build-tools-research -->

State of JS 2024 のデータに基づくビルドツール候補の調査結果は以下の通りである。

#### ビルドツール最優先候補

- **Vite**: 使用率 78%、満足度 98% でシェアと満足度を両立し、開発サーバーの高速性と設定の簡素性が一人開発に最適である。
- **esbuild**: 満足度 91% で圧倒的なビルド速度を誇り、概念ページの読み込み要件に直接的に貢献する。

#### ビルドツール次点候補

- **Parcel**: ゼロコンフィグで一人開発向きであり、満足度 68%（上昇傾向）と期待値ギャップ 40%（大幅改善）により再評価が必要である。
- **Rolldown**: Rollup の次世代版として Vite との統合を前提とし、革新的技術により要件適合性が高いが新興ツールである。

#### ビルドツール除外候補

- **webpack**: 使用率 86% で最高シェアを持つが、満足度 35% と開発者支持を失っており、複雑な設定と遅いビルド速度により除外対象である。
- **Rspack**: 満足度 89% で webpack 互換でありながら高速化を実現するが、webpack の複雑な設定をそのまま継承するため一人開発制約に不適合である。

軽量なコンポーネント指向アーキテクチャ、一人開発の制約、パフォーマンスの要件を総合すると、Vite と esbuild が最有力の候補として浮上し、Parcel と Rolldown が次点の候補、webpack と Rspack が除外すべき候補となる。

<!-- REFERENCE_END: build-tools-research -->

### 7. CSS 手法調査

<!-- REFERENCE_BEGIN: css-approach-research-conclusion -->

State of CSS 2025 のデータ分析と CogitoWeave の技術制約を総合した結果、以下の優先順位で CSS 手法の候補を選定する。

#### 第一候補: Tailwind CSS 単体

- State of CSS 2025 の最高シェア（51%）による安定性と情報の豊富性を提供する。
- ユーティリティファーストなアプローチによる軽量なコンポーネント指向への完全な適合を実現する。
- ゼロランタイムによる CSR のパフォーマンス要件を満足する。
- フレームワークに非依存により、どのフレームワーク選択でも対応可能である。
- 一人での開発における習得の容易さと保守の簡潔さが許容範囲である。

#### 条件付き候補: Tailwind CSS と shadcn/ui の組み合わせ

- React フレームワーク選択時の高品質なコンポーネント補完を提供する。
- コピー&ペーストによる必要な部分のみの導入が可能である。
- Radix UI ベースの高品質なコンポーネントを活用できる。
- **前提条件**: React フレームワークの選択が必要である。
- **リスク**: Radix UI のメンテナンス停滞があるが、コピー&ペースト方式により影響は限定的である。

#### フレームワーク非依存候補: CSS Modules と Sass/SCSS の組み合わせ

- CSS Modules（28%）と Sass/SCSS（61%）による高シェアの組み合わせである。
- ゼロランタイムによる軽量性とスコープの分離を両立する。
- 従来の手法による情報の豊富性と習得の容易さを実現する。
- どのフレームワーク選択でも対応可能である。

#### 除外候補

- Bootstrap: 重量級でバンドルサイズの制約に不適合である。
- Styled Components: ランタイムコストが CSR のパフォーマンス要件に抵触する。
- Ant Design: 企業向けの複雑性が一人での開発制約に不適合である。

Tailwind CSS 単体がフレームワーク選択の自由度と技術制約への適合性を両立する最適解として選定される。React 選択時は shadcn/ui との組み合わせがさらに高品質な選択肢となる。

<!-- REFERENCE_END: css-approach-research-conclusion -->

### 8. フロントエンドテストツール調査

<!-- REFERENCE_BEGIN: testing-tools-research -->

State of JS 2024 の調査結果に基づくテストツール候補の調査結果は以下の通りである。

#### テストツール最優先候補

- **単体テストフレームワーク**
  - **Vitest**: 満足度 98% で最高水準を示し、ESM 対応と高速実行により現代的な開発環境に最適化されている。
- **コンポーネントテストツール**
  - **Testing Library**: 満足度 91% でコンポーネントテストの安定した品質を提供し、シンプルなテスト記述により保守性重視の要件に適合する。
- **E2E テストツール**
  - **Playwright**: 満足度 94% で E2E テストの技術革新を牽引し、関心度 77% と将来性も高い。

#### テストツール次点候補

- **単体テストフレームワーク**
  - **Node Test Runner**: 満足度 87%、関心度 76% と高評価であり、依存関係を最小化する軽量な選択肢として注目される。
- **コンポーネントテストツール**
  - **Storybook**: 使用率 52% でコンポーネントテストツールの主流であり、関心度 63% と継続的な注目を集めている。
- **E2E テストツール**
  - **Puppeteer**: 満足度 74% で安定した品質を提供し、使用率 36% で実績がある。

#### テストツール除外候補

- **単体テストフレームワーク**
  - **Jest**: 使用率 72% で最高シェアを持つが、満足度 73% と技術革新性で劣り、ESM 対応の不完全性により現代的な開発環境での制約がある。
- **E2E テストツール**
  - **Cypress**: 使用率 45% で E2E テストツールの実績を持つが、満足度 64% と技術革新性で新世代ツールに劣る。

従来の主流ツールから新世代ツールへの世代交代が明確であり、Vitest、Testing Library、Playwright が最有力の候補として浮上し、Node Test Runner、Storybook、Puppeteer が次点の候補、Jest と Cypress が除外すべき候補となる。

<!-- REFERENCE_END: testing-tools-research -->

### 9. 基盤技術選択

<!-- REFERENCE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- REFERENCE_END: foundation-tech-selection -->

### 10. グラフ可視化選択

<!-- REFERENCE_BEGIN: graph-visualization-selection -->

グラフ可視化ライブラリとして D3 を採用する。

<!-- REFERENCE_END: graph-visualization-selection -->

### 11. HTTP 通信・API 連携選択

<!-- REFERENCE_BEGIN: http-api-selection -->

HTTP 通信・API 連携技術として fetch API を採用する。

<!-- REFERENCE_END: http-api-selection -->

## 2. DB 技術選択

<!-- REFERENCE_BEGIN: sqlite-selection -->

CogitoWeave システムでは **SQLite** を選択する。

<!-- REFERENCE_END: sqlite-selection -->

## 3. バックエンド 技術選択

### 1. 技術制約評価

<!-- REFERENCE_BEGIN: technical-constraints-evaluation -->

全ての主要言語・フレームワーク組み合わせが技術制約条件を満たすため、技術制約による足切りは実質的に発生しない。Ubuntu 環境での動作確保という制約条件の明確化により、既存リソースとの整合性は保証される。

<!-- REFERENCE_END: technical-constraints-evaluation -->

### 2. 非機能要件評価

<!-- REFERENCE_BEGIN: non-functional-requirements-evaluation -->

全ての主要言語・フレームワーク組み合わせが非機能要件を満たすため、非機能要件による足切りは実質的に発生しない。

<!-- REFERENCE_END: non-functional-requirements-evaluation -->

### 3. 言語 × フレームワーク選択

<!-- REFERENCE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- REFERENCE_END: language-framework-selection -->

### 4. アプリケーションサーバー選択

<!-- REFERENCE_BEGIN: app-server-selection -->

アプリケーションサーバーとして uvicorn を採用する。

<!-- REFERENCE_END: app-server-selection -->

### 5. ORM/ODM・マイグレーション選択

<!-- REFERENCE_BEGIN: orm-selection -->

ORM/ODM ツールとして SQLAlchemy + Alembic を採用する。

<!-- REFERENCE_END: orm-selection -->

### 6. API ドキュメント生成選択

<!-- REFERENCE_BEGIN: api-doc-selection -->

FastAPI 内蔵の Swagger UI・ReDoc 機能を使用し、外部 API 仕様書生成ツールは導入しない。

<!-- REFERENCE_END: api-doc-selection -->

### 7. バリデーション技術選択

<!-- REFERENCE_BEGIN: validation-tech-selection -->

バリデーション技術として FastAPI 標準 Pydantic を採用する。

<!-- REFERENCE_END: validation-tech-selection -->

### 8. HTTP クライアント選択

<!-- REFERENCE_BEGIN: http-client-tech-selection -->

HTTP クライアントライブラリとして httpx を採用する。

<!-- REFERENCE_END: http-client-tech-selection -->

### 9. 設定管理技術選択

<!-- REFERENCE_BEGIN: config-management-tech-selection -->

バックエンドの設定管理技術として環境変数方式を採用し、os.getenv による実装を行う。

<!-- REFERENCE_END: config-management-tech-selection -->

### 10. バックエンドテストツール選択

<!-- REFERENCE_BEGIN: backend-testing-tech-selection -->

バックエンドテストツールとして以下を採用する：

- pytest
- FastAPI TestClient
- pytest-asyncio
- pytest-mock
- httpx-mock
- time-machine
- pytest-cov
- pytest-xdist

<!-- REFERENCE_END: backend-testing-tech-selection -->

### 11. ログ・モニタリング選択

<!-- REFERENCE_BEGIN: logging-tech-selection -->

ログライブラリとして loguru を採用する。

<!-- REFERENCE_END: logging-tech-selection -->

## 4. Git サービス選定

<!-- REFERENCE_BEGIN: git-service-selection -->

Git サービスとして GitHub を採用する。

<!-- REFERENCE_END: git-service-selection -->
