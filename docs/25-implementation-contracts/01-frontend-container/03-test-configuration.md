---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
    - "intra-process-frontend-performance-technology-mapping"
  produces:
    - "frontend-test-configuration"
---

# 03-test-configuration - テスト構成

## 概要

フロントエンドのテスト環境構築とテストファイル配置の実装契約を定義する。

## 前提制約

<!-- PREMISE_BEGIN: frontend-container-integrated-constraints -->

### 物理配置

- セルフホスト環境の Docker コンテナとして静的配信

### 技術スタック

- フレームワーク: Solid + TanStack Router
- スタイリング: Tailwind CSS + CSS Modules
- グラフ可視化: D3
- ビルドツール: Vite
- テストフレームワーク: Vitest + Testing Library + Playwright

### アーキテクチャ方針

- CSR(Client-Side Rendering)採用
- SPA(Single Page Application)構造
- 軽量コンポーネント指向アーキテクチャ
- 3 層状態管理(コンポーネント/ページ/アプリケーション)
- コンテナコンポーネントパターンと単方向データフロー

### 画面構成

- メイン画面: 概念管理、文献メモ管理、関係性管理を統合
- 質問探索画面: 質問から概念発見とメモ参照を独立提供
- 空間配置画面: 概念の空間配置と関係性構築を専用提供
- タブ切り替えによる画面間移動

### パフォーマンス

- システム初回起動パフォーマンスバジェット対応
- 概念ページ読み込みパフォーマンスバジェット対応
- コード分割とバンドル最適化
- 軽量再レンダリング機構
- 必要時のグラフ描画最適化

### レスポンシブ対応

- Mac 環境: 情報整理機能最適化
- iPhone 環境: 情報参照機能最適化

### エラーハンドリング

- エラー境界コンポーネント
- ルーティングエラーハンドリング
- location.reload による強制リロード

### 監視・トレーシング

- Sentry による APM・エラートラッキング
- クライアントサイド監視・トレーシング
- 障害検知

### サービス間通信

- バックエンドへの依存(一方向)
- HTTP クライアントによる直接呼び出し
- データ統合責務

### 不足技術要素

- DOM 環境シミュレーション(jsdom/happy-dom)
- TanStack Router のテスト環境構築手法
- API モック化手法
- TypeScript 型生成(openapi-typescript)
- API 仕様変更の自動検知
- メモリ使用量監視(Chrome DevTools Memory)
- リトライ機構の実装パターン

<!-- PREMISE_END: frontend-container-integrated-constraints -->

<!-- PREMISE_BEGIN: intra-process-frontend-performance-technology-mapping -->

技術単位での Process 制約対応を以下に示す。

- Vite
  - HTML 配信とビルド最適化
  - コード分割とバンドル最適化
- Solid
  - JavaScript 実行管理と DOM 操作最適化
  - Signal による最小限状態管理
  - 軽量再レンダリング機構
  - エラー境界コンポーネント
  - 基本的な軽量実装
  - 軽量メモリ管理
- TanStack Router
  - ページ初期化管理
  - SPA ルーティングと遷移管理
  - ルーティングエラーハンドリング
- Tailwind CSS
  - シンプルなエラー画面スタイリング
- D3
  - 必要時のグラフ描画最適化
- JavaScript
  - location.reload() による強制リロード

<!-- PREMISE_END: intra-process-frontend-performance-technology-mapping -->

## 論理

### DOM 環境シミュレーションの検討

happy-dom を採用する。前提制約に軽量実装が繰り返し強調されており(軽量コンポーネント指向、軽量再レンダリング、軽量メモリ管理)、DOM シミュレーションも軽量な方が一貫性がある。テスト実行速度が速いことで開発サイクルが高速化される。Vitest の公式ドキュメントでも happy-dom が推奨されている。Solid は軽量フレームワークであり、軽量な DOM シミュレーションとの組み合わせが自然である。

jsdom は安定しているが、一人開発ではトラブルシューティング情報の豊富さは重要度が低い。

### テストファイル配置の検討

ユニットテストはコロケーション、E2E テストは別フォルダ管理のハイブリッド方式を採用する。

ユニットテストはコロケーションの方が、コードとテストを行き来する際の認知負荷が低い。テスト対象のコードとテストコードが同じディレクトリにあることで、関連性が明確になる。

E2E テストは画面全体のシナリオをテストするため、特定のコンポーネントやページに紐づかない。前提制約に Playwright があり、E2E テストは別管理が自然である。

一人開発であるため、ハイブリッド方式のルール周知コストは問題にならない。

### API モック化の検討

不足技術要素に API モック化手法が挙げられている。ユニットテストでは API 通信を実際に行わず、モックで代替する必要がある。fetch API を使用しているため、fetch のモック化手法が必要である。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-test-configuration -->

DOM 環境シミュレーションに happy-dom を採用し、ユニットテストはコロケーション、E2E テストは別フォルダ管理とする。

### テスト環境

- DOM 環境シミュレーション: happy-dom
- ユニットテストフレームワーク: Vitest + Testing Library
- E2E テストフレームワーク: Playwright
- API モック: fetch のモック化(具体的手法は実装時に決定)

### テストファイル配置

```plaintext
src/
├── features/
│   └── concept/
│       ├── ConceptList.tsx
│       └── ConceptList.test.tsx    # ユニットテスト(コロケーション)
├── pages/
│   ├── MainPage.tsx
│   └── MainPage.test.tsx           # ユニットテスト(コロケーション)
└── shared/
    └── components/
        ├── Button.tsx
        └── Button.test.tsx         # ユニットテスト(コロケーション)

tests/
└── e2e/
    ├── main-flow.spec.ts           # E2E テスト(別フォルダ)
    └── question-flow.spec.ts
```

### 設計原則

- 軽量な DOM シミュレーションでテスト実行速度を最適化する
- ユニットテストはコード近傍に配置し、関連性を明確化する
- E2E テストは画面シナリオとして独立管理する
- API 通信はモック化し、外部依存を排除する

<!-- GLOBAL_CONCLUSION_END: frontend-test-configuration -->
