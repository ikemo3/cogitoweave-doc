---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "tracing-selection"
---

# トレーシング技術選択

<!-- PREMISE_BEGIN: apm-selection -->

CogitoWeave システムの APM 技術として、Sentry を採用する。

<!-- PREMISE_END: apm-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- PREMISE_END: budget-time-constraints -->

## 前提

### 他設計書との接続点

Physical View で Sentry を APM 技術として採用済みである。技術選択基準により慣れた技術の優先と最小限監視の方針が確立されており、予算・時間制約が存在する。

### 確立された事実

- Sentry の運用経験が既に存在する。
- 個人開発プロジェクトでのトレーシング必要性を検討する必要がある。
- 分散システムにおける処理追跡機能が求められる。
- 月 1000 円の予算制約内での運用が必要である。

### 判断基準

- 既存の運用知識を活用できることを重視する。
- 個人開発に適した最小限の監視レベルを確保することを優先する。
- 追加のインフラ構築負荷を最小化することを確認する。

### スコープ境界

本判断ではトレーシング技術の基本選択のみを扱い、具体的な設定やトレース分析手順は扱わない。

## 論理

第一に、Sentry Performance は既存の Sentry 基盤を活用でき、技術選択基準の「慣れた技術の優先」に適合する。エラートラッキングと統合されたトレーシング機能により、追加の技術習得や運用知識の蓄積が不要である。

第二に、Sentry Performance は個人開発に適したシンプルなトレーシングアプローチを提供し、技術選択基準の「最小限監視」に適合する。OpenTelemetry + Jaeger のような複雑な分散トレーシング基盤と比較して、SDK だけで完結するシンプルな構成により、インフラ構築・保守負荷を最小化できる。

第三に、Sentry Performance の無料プランにより月 1000 円の予算制約内での運用が可能である。基本的なトレーシング機能は追加コストなしで利用でき、APM 機能との統合により一元的な監視体制を効率的に構築可能である。週 7 時間の時間制約下でも、既存 Sentry 設定の拡張のみで実現できる。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: tracing-selection -->

CogitoWeave システムのトレーシング技術として、Sentry Performance を採用する。

<!-- GLOBAL_CONCLUSION_END: tracing-selection -->

### 論理的根拠

Sentry Performance により既存の Sentry 基盤を活用しつつ、個人開発に適したシンプルなトレーシングアプローチで最小限監視を実現できる。SDK だけで完結する構成により追加のインフラ構築負荷を回避し、無料プランにより予算制約内で APM 機能との統合的な監視体制を効率的に構築する。
