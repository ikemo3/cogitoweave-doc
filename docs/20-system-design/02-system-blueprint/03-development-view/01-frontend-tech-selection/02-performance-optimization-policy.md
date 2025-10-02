---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-rendering-style"
    - "performance-requirements"
    - "development-constraints"
    - "component-granularity-design"
  produces:
    - "performance-optimization-policy"
---

# パフォーマンス最適化方針決定

## 前提

<!-- PREMISE_BEGIN: frontend-rendering-style -->

CogitoWeave のフロントエンドレンダリングスタイルとして**CSR(Client-Side Rendering)**を採用する。3 画面間の状態共有効率と軽量なコンポーネント指向の特性を活かし、一人での開発における保守性と理解しやすさを確保する。

<!-- PREMISE_END: frontend-rendering-style -->

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

<!-- PREMISE_BEGIN: component-granularity-design -->

CogitoWeave の UI コンポーネント責務として、階層的な粒度設計による軽量なコンポーネント指向アーキテクチャを採用する。

- **最小粒度**: ボタン・入力フィールド・ラベル等の基本 UI 要素レベル
  - 単一責務の原則により、一つの操作または表示に特化する。
- **統合粒度**: カード・パネル・モーダル等の機能単位レベル
  - 複数の基本要素を組み合わせた意味のある機能を提供する。
- **階層アプローチ**: 統合コンポーネント内で基本コンポーネントを組み合わせる
  - 外部からは統合コンポーネントとして使用し、内部実装では基本コンポーネントで構成する。

<!-- PREMISE_END: component-granularity-design -->

## 論理

CogitoWeave のパフォーマンス最適化方針は以下の論理により決定される。

概念ページ読み込み 1.5 秒以内が最重要パフォーマンス要件である。これは日常的な操作で最も頻繁に発生し、ユーザー体験に直接影響するためである。検索結果表示はサーバー処理が多いため、フロントエンド最適化の優先度は低い。

CSR 特性により初回ロードは遅くなる傾向があるが、継続利用での効率性を重視する個人用途では許容できる。ページ遷移の高速性がより重要であり、CSR の利点を活かす方向で最適化を行う。

一人開発制約により、複雑な最適化手法は採用しない。軽量なコンポーネント指向アーキテクチャにより設計段階でバンドルサイズを制御し、後付けの最適化に依存しない。

ツリーシェイキング対応ライブラリを優先し、必要な機能のみを効率的に導入する。元々軽量で最適化しやすい技術を選択することで、開発効率と軽量性を両立させる。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: performance-optimization-policy -->

CogitoWeave のパフォーマンス最適化方針として以下を採用する。

- 概念ページ読み込み 1.5 秒以内を最優先とする。
  - 最も頻繁な日常操作の快適性確保を最重要視する。
- 初回ロードの遅さは許容し、ページ遷移の高速性を重視する。
  - CSR 特性を活かした継続利用の効率性を優先する。
- 複雑なパフォーマンス最適化は実装しない。
  - 軽量なコンポーネント指向アーキテクチャにより設計段階で最適化を行う。
- ツリーシェイキング対応ライブラリを優先する。
  - 元々軽量で最適化しやすい技術を選択し、開発効率と軽量性を両立する。

<!-- GLOBAL_CONCLUSION_END: performance-optimization-policy -->
