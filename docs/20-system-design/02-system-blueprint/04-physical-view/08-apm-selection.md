---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "apm-selection"
---

# APM 技術選択

<!-- PREMISE_BEGIN: monitoring-tech-selection -->

CogitoWeave システムの監視・ログ技術として、Google Cloud Monitoring を採用する。

<!-- PREMISE_END: monitoring-tech-selection -->

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

Physical View で Google Cloud Monitoring を監視基盤として採用済みである。技術選択基準により慣れた技術の優先と最小限監視の方針が確立されており、予算・時間制約が存在する。

### 確立された事実

- Sentry の運用経験が既に存在する。
- 個人開発プロジェクトでの APM 必要性を検討する必要がある。
- エラートラッキング・障害調査機能が求められる。
- 月 1000 円の予算制約内での運用が必要である。

### 判断基準

- 既存の運用知識を活用できることを重視する。
- 個人開発に適した最小限の監視レベルを確保することを優先する。
- 予算制約内で運用可能であることを確認する。

### スコープ境界

本判断では APM 技術の基本選択のみを扱い、具体的な設定やアラート設定は扱わない。

## 論理

第一に、Sentry は既存の運用経験があり、技術選択基準の「慣れた技術の優先」に適合する。エラートラッキング設定、アラート管理、障害調査の知識が既に蓄積されており、追加の習得負荷が最小である。

第二に、Sentry は個人開発に適したエラー中心の APM アプローチを提供し、技術選択基準の「最小限監視」に適合する。New Relic や Datadog のような包括的 APM と比較して、エラートラッキングと基本的な性能監視に特化しており、個人用途での過剰な機能を排除できる。

第三に、Sentry の無料プランにより月 1000 円の予算制約内での運用が可能である。個人プロジェクト規模のエラー発生量・イベント数であれば、追加コストを発生させずに基本的な APM 機能を利用できる。Google Cloud Monitoring との併用により、包括的な監視体制を効率的に構築可能である。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: apm-selection -->

CogitoWeave システムの APM 技術として、Sentry を採用する。

<!-- GLOBAL_CONCLUSION_END: apm-selection -->

### 論理的根拠

Sentry により既存の運用経験を活用しつつ、個人開発に適したエラー中心の APM アプローチで最小限監視を実現できる。無料プランにより月 1000 円の予算制約内で運用可能であり、Google Cloud Monitoring との併用により包括的な監視体制を効率的に構築する。
