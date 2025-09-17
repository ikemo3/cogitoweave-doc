---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-rendering-style"
    - "performance-requirements"
    - "development-constraints"
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
  - 概念ページ読み込み: 2 秒以内(既存ツールレベル)
  - 検索結果表示: 1 秒以内(現代の検索体験期待)
  - LLM API 呼び出し: 10 秒以内(生成 AI 市場期待)
  - データベース応答: ミリ秒オーダー(現代ソフトウェア基本期待)
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

## 論理

CogitoWeave のパフォーマンス最適化方針は以下の論理により決定される。

概念ページ読み込み 2 秒以内が最重要パフォーマンス要件である。これは日常的な操作で最も頻繁に発生し、ユーザー体験に直接影響するためである。検索結果表示はサーバー処理が多いため、フロントエンド最適化の優先度は低い。

CSR 特性により初回ロードは遅くなる傾向があるが、継続利用での効率性を重視する個人用途では許容できる。ページ遷移の高速性がより重要であり、CSR の利点を活かす方向で最適化を行う。

一人開発制約により、複雑な最適化手法は採用しない。コード分割やバンドル最適化などの高度な技術は、保守性とのトレードオフを考慮して最小限に留める。

無駄に大きなライブラリは避けるが、必要な機能を提供するライブラリは適切に採用する。バンドルサイズよりも開発効率と保守性を優先する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: performance-optimization-policy -->

CogitoWeave のパフォーマンス最適化方針として以下を採用する。

- 概念ページ読み込み 2 秒以内を最優先とする。
  - 最も頻繁な日常操作の快適性確保を最重要視する。
- 初回ロードの遅さは許容し、ページ遷移の高速性を重視する。
  - CSR 特性を活かした継続利用の効率性を優先する。
- 複雑なパフォーマンス最適化は実装しない。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- バンドルサイズよりも開発効率を重視する。
  - 必要な機能を提供するライブラリは適切に採用する。

<!-- GLOBAL_CONCLUSION_END: performance-optimization-policy -->
