---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "budget-time-constraints"
    - "existing-resource-constraints"
    - "development-constraints"
    - "market-constraints"
  produces:
    - "data-migration-policy"
    - "platform-compatibility"
    - "export-functionality"
---

# 互換性 (Compatibility)

## 前提

<!-- PREMISE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- PREMISE_END: budget-time-constraints -->

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

<!-- PREMISE_BEGIN: market-constraints -->

### 既存ツール代替可能性制約

- 既存ツールでは実現困難な独自価値の提供が必須である。

### 個人開発の市場環境制約

- 他者からの評価や社会的な標準に合わせる必要がない。
<!-- PREMISE_END: market-constraints -->

## 論理

### Q1: 他のナレッジツールからのデータ移行について

データ移行機能は提供しない。開発・保守体制制約により一人で管理可能な複雑度に制限する必要があり、データ移行機能の実装・テスト・保守は複雑度を大幅に増加させるため除外する。また予算・時間制約活用により週 7 時間の制約下でコア機能開発に集中する必要があり、移行機能ではなく独自価値の実現にリソースを集中させる。

### Q2: プラットフォーム互換性について

Web 配信のみを前提とする。既存リソース制約により自宅サーバー・Cloudflare で最適化された Web 配信インフラを活用し、追加コストを最小化する必要がある。開発・保守体制制約により複数プラットフォーム対応は保守負荷を指数的に増加させ、一人での管理可能性を超えるため除外する。

### Q3: エクスポート機能について

バックアップ用途と LLM に渡すための Markdown 出力機能は提供する。既存ツール代替可能性制約により独自価値の提供が必須であり、バックアップ機能は既存ツールでは提供されない価値となる。また個人開発の市場環境制約により他者との共有ではなく個人の思考支援に特化でき、蓄積した知識を LLM に効率的に渡す Markdown 形式は独自の使用体験を実現する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: data-migration-policy -->

他のナレッジツールからのデータ移行機能は提供しない。開発・保守の複雑度制限とコア機能への開発リソース集中を優先する。

<!-- GLOBAL_CONCLUSION_END: data-migration-policy -->

<!-- GLOBAL_CONCLUSION_BEGIN: platform-compatibility -->

プラットフォーム互換性は Web 配信のみを前提とする。既存インフラの活用とコスト最小化を優先し、複数プラットフォーム対応による保守負荷増大を避ける。

<!-- GLOBAL_CONCLUSION_END: platform-compatibility -->

<!-- GLOBAL_CONCLUSION_BEGIN: export-functionality -->

エクスポート機能はバックアップ用途と LLM に渡すための Markdown 出力のみ提供する。個人の思考支援に特化した独自価値の実現を優先する。

<!-- GLOBAL_CONCLUSION_END: export-functionality -->
