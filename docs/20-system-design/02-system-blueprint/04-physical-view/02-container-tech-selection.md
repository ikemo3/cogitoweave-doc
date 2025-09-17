---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "container-tech-selection"
---

# コンテナ技術選択

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 前提

### 他設計書との接続点

既存リソース制約により自宅サーバー環境での運用が前提であり、技術選択基準により慣れた技術の優先が確立されている。

### 確立された事実

- Docker Compose の運用経験が既に存在する。
- 自宅サーバー(Ubuntu)での単一ノード運用である。
- 複雑なオーケストレーション機能は不要である。

### 判断基準

- 既存の運用知識を活用できることを重視する。
- 単一ノード環境に適した技術であることを確認する。
- 設定・保守の複雑度を最小化することを優先する。

### スコープ境界

本判断ではコンテナランタイムとオーケストレーション技術の基本選択のみを扱い、具体的な設定や運用手順は扱わない。

## 論理

第一に、自宅サーバー環境での単一ノード運用において、Kubernetes レベルのオーケストレーション機能は過剰である。複数ノード間の負荷分散、クラスタ管理、高可用性機能は個人用途では不要であり、設定・保守の複雑度のみが増加する。

第二に、Docker Compose は既存の運用知識を活用でき、技術選択基準の「慣れた技術の優先」に適合する。コンテナ定義、ネットワーク設定、ボリューム管理の経験が既に蓄積されており、追加の習得が不要である。

第三に、Docker Compose は単一ノード環境に最適化された技術であり、自宅サーバー制約に適合する。シンプルな YAML 設定によりマルチコンテナアプリケーションを効率的に管理でき、開発・テスト・本番環境の統一も容易である。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: container-tech-selection -->

CogitoWeave システムのコンテナ技術として、Docker + Docker Compose を採用する。

<!-- GLOBAL_CONCLUSION_END: container-tech-selection -->

### 論理的根拠

Docker Compose により既存の運用知識を活用しつつ、自宅サーバー環境での単一ノード運用に最適化されたコンテナ管理を実現できる。Kubernetes 等の複雑なオーケストレーション技術は個人用途では過剰であり、Docker Compose のシンプルな設定・保守が週 7 時間の時間制約に適合する。
