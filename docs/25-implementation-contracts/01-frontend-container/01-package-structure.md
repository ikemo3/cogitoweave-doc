---
doc_type: "unknown"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
  produces:
    - "frontend-package-structure"
---

# 01-package-structure - パッケージ分割方針

## 概要

フロントエンドのパッケージ構成と、ディレクトリ構造による責務分離の実装契約を定義する。

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

## 論理

### パッケージ管理方針の検討

一人開発であり、パッケージ間の独立した再利用やチーム間での分離が不要である。複数パッケージに分割すると、package.json の管理、ビルド設定の複雑化、型定義の共有などのオーバーヘッドが発生する。単一の package.json で管理するモノリシック構成であれば、これらの複雑性を排除しつつ、ディレクトリ構造で責務を明確に分離できる。

### ディレクトリ構造パターンの検討

機能境界が明確である(概念管理、文献メモ管理、関係性管理、質問探索)。機能別構造では、関連するコンポーネント、状態管理、API 通信が機能ごとに集約されるため、機能の追加・修正・削除時に影響範囲が明確になる。

レイヤー別構造では、1 つの機能を実装する際に components/, api/, hooks/ など複数のディレクトリを横断する必要があり、関連ファイルを探す手間が増える。機能別構造であれば、1 つの機能に関連するファイルが同じディレクトリにまとまっているため、変更時の認知負荷が低い。

Feature-Sliced Design を過去に試したが、構造の複雑さがオーバーヘッドとなり、機能別構造の方が効率的であるという結論に至った経験がある。

### ディレクトリ配置の具体化

features/ には機能ごとの再利用可能な部品を配置する。機能が複雑化した場合は、機能ディレクトリ内部でレイヤー別にディレクトリを分割することを妨げない。

pages/ には画面の組み立てを配置する。前提制約の画面構成(メイン画面、質問探索画面、空間配置画面)は複数機能を組み合わせる場所となるため、features/ とは独立した pages/ として責務を分離する。

shared/ には横断的な関心事を責務別に配置する。components/(共通 UI コンポーネント)、api/(API 通信基盤)を基本とし、必要に応じて validation/, formatting/, constants/ などの責務が明確なディレクトリを追加する。util/, common/, lib/ のような曖昧な名前は使用しない。

app/ にはアプリケーション全体の初期化・設定を配置する。具体的な配置内容は TanStack Router 構成で決定する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-package-structure -->

フロントエンドは単一 package.json によるモノリシック構成とし、機能別ディレクトリ構造により責務を分離する。

### ディレクトリ構造

```plaintext
src/
├── features/
│   ├── concept/          # 概念管理機能
│   ├── literature-memo/  # 文献メモ管理機能
│   ├── relationship/     # 関係性管理機能
│   └── question/         # 質問探索機能
├── pages/                # 画面の組み立て
├── shared/
│   ├── components/       # 共通UIコンポーネント
│   └── api/              # API通信基盤
└── app/                  # アプリケーション全体の初期化・設定
```

### 設計原則

- 機能ごとに凝集度を高め、機能追加・削除を容易にする
- 機能ディレクトリ内部でのレイヤー別分割を妨げない
- 画面は複数機能を組み合わせる場所として pages/ に独立配置する
- 横断的な関心事は責務が明確な名前で shared/ に配置する
- util/, common/, lib/ のような曖昧な名前は使用しない

<!-- GLOBAL_CONCLUSION_END: frontend-package-structure -->
