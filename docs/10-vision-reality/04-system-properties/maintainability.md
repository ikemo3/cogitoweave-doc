---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "usage-context"
    - "development-constraints"
    - "budget-time-constraints"
  produces:
    - "code-quality-requirements"
    - "documentation-requirements"
    - "testing-automation-requirements"
    - "technology-selection-criteria"
    - "architecture-stability-requirements"
---

# 保守性 (Maintainability)

## 前提

<!-- PREMISE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- PREMISE_END: usage-context -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

<!-- PREMISE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- PREMISE_END: budget-time-constraints -->

### 保守方針特性

- **単独保守前提**: 作成者のみによる長期保守・引き継ぎは想定しない
- **品質重視**: コードの美しさ・保守性を重視した継続的改善
- **実験兼用**: 設計書作成実験としての詳細ドキュメント化
- **自動化重視**: 手動オペレーションの最小化とスクリプト化

## 論理

### 単独保守による一貫性確保と負荷制限

作成者のみによる保守であるため、複数人での保守を前提とした複雑な規約・プロセスは不要である。むしろ、一人で理解・保守可能な適度な複雑度に制限し、シンプルで一貫性のある設計を優先すべきである。

### 品質重視による継続的リファクタリング前提

「リファクタリングが息をするようなもの」である作成者の特性により、継続的なコード改善が前提となる。従って、初期実装の完璧性より、改善しやすい設計・テスト可能性・明確な責任分離を優先すべきである。

### 実験兼用による詳細ドキュメント化

このプロジェクトは設計書作成手法の実験としての側面を持つため、通常の開発よりも詳細なドキュメント化が必要である。詳細ドキュメント化により保守性の向上と実験データの収集を同時に実現できるため、ドキュメント作成にかかるコストを許容すべきである。

### 時間制約による自動化・テスト重視

週 7 時間という限られた時間制約により、手動オペレーションによる保守負荷は最小化すべきである。バックエンド 100%自動テスト・復旧スクリプト化・マイグレーション自動化等により、保守作業の効率化を図るべきである。

### アーキテクチャ安定性による変更負荷軽減

適切な設計により、アーキテクチャ変更が必要となる状況は回避可能である。個人利用による要求変更リスクの排除、疎結合設計による外部依存変更への対応、成熟技術選択による破壊的変更リスクの最小化により、アーキテクチャレベルの変更は不要となる。機能追加時の既存コード大幅変更は設計不備の表れとして、継続的な設計改善で予防すべきである。

## 結論

### コード品質要件

<!-- GLOBAL_CONCLUSION_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- GLOBAL_CONCLUSION_END: code-quality-requirements -->

### ドキュメント要件

<!-- GLOBAL_CONCLUSION_BEGIN: documentation-requirements -->

- **詳細な設計書を作成**: 設計手法の実験として、通常よりも詳しい設計ドキュメントを書く
- **誰でも理解できるように書く**: 作成者以外でも読んで分かるレベルの説明を心がける
- **継続的に更新する**: コードを変更したときに合わせてドキュメントも更新する仕組みを作る

<!-- GLOBAL_CONCLUSION_END: documentation-requirements -->

### テスト・自動化要件

<!-- GLOBAL_CONCLUSION_BEGIN: testing-automation-requirements -->

- **バックエンドは完全に自動テスト**: ほぼ 100%のテストで品質を担保する
- **フロントエンドは状況に応じて使い分け**: ブラウザでの複雑な操作やデザインの確認は手動テスト、ロジック部分は自動テストを行う
- **障害対応をスクリプト化**: 復旧作業を自動化して保守の負担を減らす
- **データ移行を自動化**: データベース構造を変更するときは自動で処理する

<!-- GLOBAL_CONCLUSION_END: testing-automation-requirements -->

### 技術選択基準

<!-- GLOBAL_CONCLUSION_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- GLOBAL_CONCLUSION_END: technology-selection-criteria -->

### アーキテクチャの安定性要件

<!-- GLOBAL_CONCLUSION_BEGIN: architecture-stability-requirements -->

- **変更が必要になる原因を事前に排除**: 疎結合な設計と安定した技術を選んで、アーキテクチャを変更する必要がないようにする
- **技術的な負債を予防**: 機能を追加するときに既存コードを大幅に変更する必要がある場合は設計の問題として、継続的な改善で対応する
- **局所的な変更で機能拡張**: 新機能は既存のアーキテクチャの枠組みの中で、部分的な変更で実現する

<!-- GLOBAL_CONCLUSION_END: architecture-stability-requirements -->
