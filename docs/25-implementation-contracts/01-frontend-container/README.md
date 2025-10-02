---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
    - "intra-process-frontend-performance-technology-mapping"
    - "http-api-selection"
  produces: []
---

# 01-frontend-container - フロントエンドパッケージ設計

## 概要

Container レベルでのフロントエンド実装契約を定義する。What 系で決定された設計判断を、実行可能単位レベルの具体的な実装契約に落とし込む。

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

<!-- PREMISE_BEGIN: http-api-selection -->

HTTP 通信・API 連携技術として fetch API を採用する。

<!-- PREMISE_END: http-api-selection -->

## 論理

前提制約から、Container レベルの実装契約を具体化するには3つの観点が必要である。

### パッケージ分割方針が必要な理由

軽量コンポーネント指向アーキテクチャとコンテナコンポーネントパターンを実現するには、コードの物理的な配置と依存関係の制御が必要である。パッケージ構造により、責務の境界を明確化し、意図しない依存を防止する。

### TanStack Router 構成が必要な理由

SPA 構造とコード分割・バンドル最適化を実現するには、ルーティングの具体的な構成方針が必要である。TanStack Router によるページ初期化管理と遷移管理を、パフォーマンスバジェットと両立させる設計契約を定義する必要がある。

### テスト構成が必要な理由

不足技術要素に DOM 環境シミュレーション、TanStack Router のテスト環境構築手法、API モック化手法が挙げられている。これらを具体化するテスト構成の実装契約を定義する必要がある。

## ディレクトリ構成

実装契約の詳細は以下のファイルで定義する。

- [パッケージ分割方針](01-package-structure.md)
- [TanStack Router 構成](02-router-configuration.md)
- [テスト構成](03-test-configuration.md)

## 結論

Container レベルでのフロントエンド実装契約を、パッケージ構造・ルーティング・テストの3つの観点から定義する。
