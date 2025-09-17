---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-rendering-style"
    - "state-management-policy"
    - "performance-optimization-policy"
    - "screen-transition-design"
  produces:
    - "app-structure-evaluation"
---

# アプリケーション構造評価

## 前提

<!-- PREMISE_BEGIN: frontend-rendering-style -->

CogitoWeave のフロントエンドレンダリングスタイルとして**CSR(Client-Side Rendering)**を採用する。3 画面間の状態共有効率と軽量なコンポーネント指向の特性を活かし、一人での開発における保守性と理解しやすさを確保する。

<!-- PREMISE_END: frontend-rendering-style -->

<!-- PREMISE_BEGIN: state-management-policy -->

CogitoWeave の状態管理方針として以下を採用する。

- 画面間状態共有は実装しない。
  - 各画面は独立したワークフローを持つため、状態共有の必要性がない。
- 各画面で独立した状態管理を行う。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- 編集中データは一時的にローカルストレージに保存する。
  - ユーザーの誤操作によるデータ消失を防止する。

<!-- PREMISE_END: state-management-policy -->

<!-- PREMISE_BEGIN: performance-optimization-policy -->

CogitoWeave のパフォーマンス最適化方針として以下を採用する。

- 概念ページ読み込み 2 秒以内を最優先とする。
  - 最も頻繁な日常操作の快適性確保を最重要視する。
- 初回ロードの遅さは許容し、ページ遷移の高速性を重視する。
  - CSR 特性を活かした継続利用の効率性を優先する。
- 複雑なパフォーマンス最適化は実装しない。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- バンドルサイズよりも開発効率を重視する。
  - 必要な機能を提供するライブラリは適切に採用する。

<!-- PREMISE_END: performance-optimization-policy -->

<!-- PREMISE_BEGIN: screen-transition-design -->

シンプルなタブ切り替えによる 3 画面間の移動を提供する。操作の複雑度を抑制し直感的なナビゲーションを実現する。

<!-- PREMISE_END: screen-transition-design -->

## 論理

CogitoWeave のアプリケーション構造は以下の論理により評価される。

画面間状態共有を実装しないため、SPA の主要メリットである画面間での状態共有効率は不要となる。各画面が独立したワークフローを持つ特性は、MPA の独立性に適合する。

概念ページ読み込み 2 秒以内の最優先要件により、概念間移動の高速性が重要となる。メイン画面内での概念 A から概念 B への移動は頻繁に発生するため、この部分のみ SPA 的な実装による高速化が有効である。

一人開発制約により理解しやすさと保守性を重視するため、シンプルな構造を選好する。3 画面の完全な独立性を保ちながら、必要な部分のみ最適化するハイブリッドアプローチが適切である。

初回ロードの遅さは許容するが、継続利用での効率性を重視する方針により、各画面の軽量性とメイン画面での概念間移動の高速性を両立する構成が最適である。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: app-structure-evaluation -->

CogitoWeave のアプリケーション構造として以下を採用する。

- 基本構造は MPA とする。
  - 画面間状態共有が不要で、各画面の独立性を重視するため。
- メイン画面内の概念間移動のみ SPA 的実装を適用する。
  - 概念ページ読み込み最優先要件を満たすため。
- 3 画面間の移動は別ページ遷移とする。
  - 各画面の軽量性と独立性を確保するため。
- ハイブリッドアプローチにより必要な部分のみ最適化する。
  - 一人開発制約下での理解しやすさと保守性を維持するため。

<!-- GLOBAL_CONCLUSION_END: app-structure-evaluation -->
