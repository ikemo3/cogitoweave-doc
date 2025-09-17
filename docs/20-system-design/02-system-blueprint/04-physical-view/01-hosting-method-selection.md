---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "hosting-method-selection"
---

# ホスティング方式選択

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- PREMISE_END: budget-time-constraints -->

## 前提

### 他設計書との接続点

既存リソース制約と予算・時間制約により、セルフホスト方式が前提として確定している。

### 確立された事実

- 自宅サーバー(Ubuntu)が既に稼働中である。
- Cloudflare サービスアカウントが既に利用可能である。
- 追加インフラコストは月 1000 円以内に制限される。

### 判断基準

- 既存リソースの最大限活用を優先する。
- 追加コストの最小化を重視する。
- 運用負荷の軽減を考慮する。

### スコープ境界

本判断ではホスティング方式の基本構成のみを扱い、具体的なサーバー設定や Cloudflare の詳細設定は扱わない。

## 論理

第一に、既存リソース制約により自宅サーバーの活用が必須である。追加のクラウドインフラ契約は月 1000 円の予算制約に抵触するため、既存の自宅サーバー(Ubuntu)を主要な計算リソースとして使用する必要がある。

第二に、Cloudflare サービスによる CDN・セキュリティ・ドメイン管理機能の活用により、セルフホストの欠点を補完できる。自宅サーバー単体では外部アクセスやセキュリティに課題があるが、Cloudflare との組み合わせにより企業レベルのサービス品質を実現可能である。

第三に、この組み合わせは既存の運用知識を活用でき、新しい技術の習得が不要である。自宅サーバーと Cloudflare の管理は既に確立されているため、週 7 時間の時間制約下でも効率的な運用が可能である。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: hosting-method-selection -->

CogitoWeave システムのホスティング方式として、セルフホスト + Cloudflare サービスの組み合わせを採用する。

<!-- GLOBAL_CONCLUSION_END: hosting-method-selection -->

### 論理的根拠

自宅サーバー(Ubuntu)で主要なアプリケーション処理を実行し、Cloudflare サービスで外部アクセス・CDN・セキュリティ機能を提供する構成により、既存リソースを最大限活用しつつ追加コストを最小化できる。この組み合わせは既存の運用知識を活用でき、週 7 時間の時間制約下でも効率的な運用が可能である。
