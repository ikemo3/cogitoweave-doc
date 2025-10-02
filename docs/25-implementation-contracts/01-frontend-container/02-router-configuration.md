---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
    - "intra-process-frontend-performance-technology-mapping"
  produces:
    - "frontend-router-configuration"
---

# 02-router-configuration - TanStack Router 構成

## 概要

TanStack Router のルート定義配置とコード分割の実装契約を定義する。

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

### ルート定義配置の検討

ファイルベースルーティングを採用する。ファイル構造とルートが一致するため、ルート定義を探す手間がなく直感的に理解できる。ルート定義の記述量が少なく、メンテナンス負荷が低い。TanStack Router の公式が推奨している方式であり、コード分割との連携も自動化しやすい。

集中管理型は画面数が増えると 1 ファイルが肥大化し、分散管理型は全体像の把握が難しくなる。

Solid で非ファイルベースルーティングを試した経験があるが、ルート定義の管理が煩雑で扱いづらかった。ファイルベースルーティングの方が規約ベースで明確である。

### コード分割方針の検討

ルート単位で遅延ロードを行う。システム初回起動パフォーマンスバジェット対応が前提制約にあり、初期バンドルサイズの最小化が最優先である。画面数が 3 つ(メイン画面、質問探索画面、空間配置画面)と少ないため、画面遷移時のロード待ちは許容範囲内である。

機能単位でのチャンク分割は設定が複雑になり、画面数が少ないプロジェクトには過剰である。ルート単位の遅延ロードであれば、設定がシンプルで TanStack Router のファイルベースルーティングと自動連携できる。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-router-configuration -->

TanStack Router のファイルベースルーティングを採用し、ルート単位で遅延ロードを行う。

### ルート定義配置

```plaintext
src/
├── routes/
│   ├── __root.tsx       # ルートレイアウト
│   ├── index.tsx        # メイン画面 (/)
│   ├── question.tsx     # 質問探索画面 (/question)
│   └── spatial.tsx      # 空間配置画面 (/spatial)
└── app/
    └── router.ts        # Router インスタンス生成
```

### 設計原則

- ファイル構造とルートを一致させ、直感的な理解を実現する
- ルート単位で遅延ロードし、初期バンドルサイズを最小化する
- TanStack Router の規約に従い、設定を最小限に保つ
- ルーティングエラーは `__root.tsx` のエラー境界で捕捉する

<!-- GLOBAL_CONCLUSION_END: frontend-router-configuration -->
