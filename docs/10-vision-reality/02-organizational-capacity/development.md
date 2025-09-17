---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "development-constraints"
    - "publication-constraints"
---

# 開発方針

## 前提

CogitoWeave は完全な個人開発プロジェクトである。全ての設計・実装・テスト・保守を一人で実施し、外部からの協力・貢献は原則受け付けない。

## 論理

個人開発における組織制約を論理的に導出する。

### 開発・保守体制の制約

個人開発では限られた時間の中で全ての作業を一人で行う必要がある。また、時間が経つと実装の詳細や設計意図を忘れてしまう。そのため、シンプルで理解しやすい構造でなければ、後から見返した時に理解できず、保守が困難になる。

一人で管理可能な複雑度に制限することで、認知的負荷を軽減し、持続可能な開発・保守体制を確保する。

### 公開方針の制約

個人開発のため、コードレビューや品質標準などの組織的な配慮は不要である。一方で、設計思想をオープンにすることで社会に価値を還元したいというポリシーがある。

ただし、公開するのは設計書のみとし、ソースコードは非公開とする。これにより、設計思想の共有と個人開発の自由度を両立する。

## 結論

CogitoWeave の組織制約として、以下の設計判断を行う:

### 開発・保守体制制約

<!-- GLOBAL_CONCLUSION_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- GLOBAL_CONCLUSION_END: development-constraints -->

### 公開方針制約

<!-- GLOBAL_CONCLUSION_BEGIN: publication-constraints -->

- 複雑なコードレビュー・品質標準・セキュリティ監査等の配慮は不要である。
- 設計書は CC BY ライセンスで公開する。

<!-- GLOBAL_CONCLUSION_END: publication-constraints -->

プライベート一人開発の特性を活かし、複雑な組織的配慮を排除した効率的な開発体制を確保する。
