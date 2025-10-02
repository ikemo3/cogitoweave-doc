---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "technical-constraints-evaluation"
    - "non-functional-requirements-evaluation"
    - "technology-selection-criteria"
    - "api-tech-approach"
  produces:
    - "language-framework-selection"
---

# 言語 × フレームワーク選択

## 前提

<!-- PREMISE_BEGIN: technical-constraints-evaluation -->

全ての主要言語・フレームワーク組み合わせが技術制約条件を満たすため、技術制約による足切りは実質的に発生しない。Ubuntu 環境での動作確保という制約条件の明確化により、既存リソースとの整合性は保証される。

<!-- PREMISE_END: technical-constraints-evaluation -->

<!-- PREMISE_BEGIN: non-functional-requirements-evaluation -->

全ての主要言語・フレームワーク組み合わせが非機能要件を満たすため、非機能要件による足切りは実質的に発生しない。

<!-- PREMISE_END: non-functional-requirements-evaluation -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: api-tech-approach -->

API 技術方針として REST API を採用する。

<!-- PREMISE_END: api-tech-approach -->

技術制約・非機能要件による足切りが発生しないため、既存スキル活用と技術選択基準への適合性が主要な判断基準となる。既存スキルとして Java、Python、TypeScript(バックエンド経験あり)を保有し、フレームワーク経験として Django、FastAPI、Hono を保有している。

## 論理

第一に、技術制約・非機能要件による除外が発生しないため、既存スキル活用が主要な判断基準となる。技術選択基準により「大差ないときは慣れた技術を選ぶ」ことが方針として確立されており、既存経験の活用が合理的である。

第二に、既存スキルから現実的な選択肢を評価する。Python + FastAPI は既存経験により、即座に開発開始が可能である。TypeScript + Hono も既存経験があり、フロントエンド Solid との言語統一によるメリットが期待される。Python + Django は既存経験があるが REST API 開発においてオーバースペックの可能性がある。

第三に、API 技術方針で確立された疎結合設計による密結合リスクの回避を考慮する。TypeScript 統一による型共有は技術的には可能だが、API 技術方針により「型定義や API スキーマを共有する密結合なアプローチは採用しない」ことが確立されており、フロントエンド - バックエンド間の密結合を招く可能性がある。疎結合設計により各層が独立して変更可能な設計を重視し、言語統一による密結合よりも適切な分離を優先する。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- GLOBAL_CONCLUSION_END: language-framework-selection -->

### 論理的根拠

Python + FastAPI の組み合わせは既存経験により即座に開発開始が可能であり、技術選択基準の「大差ないときは慣れた技術を選ぶ」に完全に適合する。API 技術方針で確立された疎結合設計を考慮し、TypeScript 統一による型共有よりもフロントエンド - バックエンド間の適切な分離と独立性を優先する。
