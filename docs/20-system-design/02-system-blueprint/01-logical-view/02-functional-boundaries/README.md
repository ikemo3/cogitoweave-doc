---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "development-constraints"
    - "code-quality-requirements"
  produces:
    - "core-knowledge-boundary"
    - "llm-integration-boundary"
    - "content-acquisition-boundary"
---

# 機能境界と責務

## 前提

CogitoWeave システムの機能を論理的に分割し境界を設定する必要がある。

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

<!-- PREMISE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- PREMISE_END: code-quality-requirements -->

## 論理

境界設定は保守性を判断軸とする。概念・文献メモ・関係性は統合的に管理することで 1 人開発での保守負荷を軽減する。LLM は変わりやすい技術のため分離して保守性を確保する。コンテンツ取得は独立した処理として分離する。

## 結論

CogitoWeave システムは 3 つの機能境界で構成する。一人で管理可能な複雑度に制限し保守性を確保するためである。

<!-- GLOBAL_CONCLUSION_BEGIN: core-knowledge-boundary -->

- コア知識管理境界は概念管理、文献メモ管理、関係性管理、質問探索を統合して担当する。
  - 各機能の役割を明確に分け、統合的に管理することで保守負荷を軽減するため。

<!-- GLOBAL_CONCLUSION_END: core-knowledge-boundary -->

<!-- GLOBAL_CONCLUSION_BEGIN: llm-integration-boundary -->

- LLM 統合境界は LLM API による要約生成、概念候補提示、協働プロセス制御を担当する。
  - 変わりやすい技術を分離してシンプルな設計を維持するため。

<!-- GLOBAL_CONCLUSION_END: llm-integration-boundary -->

<!-- GLOBAL_CONCLUSION_BEGIN: content-acquisition-boundary -->

- コンテンツ取得境界は外部記事取得、内容解析、品質チェックを担当する。
  - 独立した処理として分離し機能変更時の影響範囲を限定するため。

<!-- GLOBAL_CONCLUSION_END: content-acquisition-boundary -->
