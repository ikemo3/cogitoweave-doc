---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-rendering-style"
    - "main-screen-integration"
    - "question-search-screen-integration"
    - "spatial-arrangement-screen-integration"
  produces:
    - "state-management-policy"
---

# 状態管理方針決定

## 前提

<!-- PREMISE_BEGIN: frontend-rendering-style -->

CogitoWeave のフロントエンドレンダリングスタイルとして**CSR(Client-Side Rendering)**を採用する。3 画面間の状態共有効率と軽量なコンポーネント指向の特性を活かし、一人での開発における保守性と理解しやすさを確保する。

<!-- PREMISE_END: frontend-rendering-style -->

<!-- PREMISE_BEGIN: main-screen-integration -->

メイン画面は概念管理、文献メモ管理、関係性管理を 1 画面に統合して提供する。概念・文献メモ・関係性は統合的に利用されるため一体化した画面構成が効率的である。

<!-- PREMISE_END: main-screen-integration -->

<!-- PREMISE_BEGIN: question-search-screen-integration -->

質問探索画面は質問から概念発見とメモ参照を独立画面で提供する。質問探索は他の機能と利用パターンが異なるため独立画面が適切である。

<!-- PREMISE_END: question-search-screen-integration -->

<!-- PREMISE_BEGIN: spatial-arrangement-screen-integration -->

空間配置画面は概念の空間配置と関係性構築を専用画面で提供する。複雑な直接操作が必要なためフルスクリーン専用画面が適切である。

<!-- PREMISE_END: spatial-arrangement-screen-integration -->

## 論理

CogitoWeave の状態管理方針は以下の論理により決定される。

3 つの画面はそれぞれ独立した機能を提供するため、画面間での状態共有の必要性は存在しない。メイン画面での概念管理、質問探索画面での質問処理、空間配置画面での関係性構築は、それぞれ独立したワークフローとして機能する。

一人開発における保守性と理解しやすさを重視する制約により、各画面で独立した状態管理を行う方が、コードの理解しやすさと保守性を確保できる。

編集中データの保護については、ユーザーの誤操作によるリロードでデータが消失するリスクを回避する必要がある。一時的なローカルストレージへの保存により、データ消失を防止しながら、サーバーへの過度な通信負荷を避けることができる。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: state-management-policy -->

CogitoWeave の状態管理方針として以下を採用する。

- 画面間状態共有は実装しない。
  - 各画面は独立したワークフローを持つため、状態共有の必要性がない。
- 各画面で独立した状態管理を行う。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- 編集中データは一時的にローカルストレージに保存する。
  - ユーザーの誤操作によるデータ消失を防止する。

<!-- GLOBAL_CONCLUSION_END: state-management-policy -->
