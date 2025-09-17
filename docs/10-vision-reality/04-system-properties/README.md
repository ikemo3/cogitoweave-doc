---
doc_type: "pattern-parallel"
status: "draft"
viz_exclude: true
depends:
  contracts: []
  produces: []
---

# システム制約 - 非機能要件の網羅的定義

## 目次

TBD

## 概要

CogitoWeave システムの内部品質制約として、ISO/IEC 25010 システム及びソフトウェア品質モデルに基づいた非機能要件を網羅的に定義する。これらの制約はシステム設計の全段階で参照される基盤的な品質基準となる。

## 処理フロー

```mermaid
graph TD
    Q[ISO/IEC 25010<br/>品質特性]
    Q -->|網羅分割| A[機能適合性<br/>※機能要求側で評価]
    Q -->|網羅分割| B[性能効率性<br/>Performance Efficiency]
    Q -->|網羅分割| C[互換性<br/>Compatibility]
    Q -->|網羅分割| D[ユーザビリティ<br/>Usability]
    Q -->|網羅分割| E[信頼性<br/>Reliability]
    Q -->|網羅分割| F[セキュリティ<br/>Security]
    Q -->|網羅分割| G[保守性<br/>Maintainability]
    Q -->|網羅分割| H[移植性<br/>Portability]
    
    style A fill:#f0f0f0,stroke-dasharray: 5 5
```

## 直交分割

TBD

## 並列関係ファイル

### ISO/IEC 25010 品質特性

これらは互いに排他的で網羅的な品質特性であり、決定順序に依存関係はない。また、機能適合性は機能要求の受け入れ基準として評価するため、ここでは非機能特性のみ扱う。

- 機能適合性 - システムが指定された機能を提供する度合い
- [性能効率性](performance.md) - 指定された条件の下で、使用されるリソースの量に関するパフォーマンス
- [互換性](compatibility.md) - 同じハードウェア環境で他システムと情報交換する度合い
- [ユーザビリティ](usability.md) - 指定された利用者が効果・効率・満足を達成する度合い
- [信頼性](reliability.md) - 指定された条件下で指定された期間機能を実行する度合い
- [セキュリティ](security.md) - 情報・データを許可なく参照・変更されない度合い
- [保守性](maintainability.md) - 有効性・効率性をもって修正できる度合い
- [移植性](portability.md) - 1つの環境から他の環境に移すことができる度合い
