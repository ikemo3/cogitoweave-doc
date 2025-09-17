---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "batch-specification"
    - "component-granularity-design"
    - "browser-support"
    - "architecture-stability-requirements"
  produces:
    - "api-tech-approach"
---

# API 技術方針

## 前提

<!-- PREMISE_BEGIN: batch-specification -->

CogitoWeave のバッチ仕様は日次データバックアップのみを実装する。全データの完全バックアップを日次実行し、システム障害時の迅速復旧を支援する。自動分析・推奨機能、データクリーンアップ、統計処理等は設計方針により除外する。

<!-- PREMISE_END: batch-specification -->

<!-- PREMISE_BEGIN: component-granularity-design -->

CogitoWeave の UI コンポーネント責務として、階層的な粒度設計による軽量なコンポーネント指向アーキテクチャを採用する。

- **最小粒度**: ボタン・入力フィールド・ラベル等の基本 UI 要素レベル
  - 単一責務の原則により、一つの操作または表示に特化する。
- **統合粒度**: カード・パネル・モーダル等の機能単位レベル
  - 複数の基本要素を組み合わせた意味のある機能を提供する。
- **階層アプローチ**: 統合コンポーネント内で基本コンポーネントを組み合わせる
  - 外部からは統合コンポーネントとして使用し、内部実装では基本コンポーネントで構成する。

<!-- PREMISE_END: component-granularity-design -->

<!-- PREMISE_BEGIN: browser-support -->

- 必須対応
  - 対応ブラウザ: iOS Safari(iPhone)、Chrome(Mac)
  - 採用 Web 標準: Baseline Newly available を基準とした機能のみ使用
  - 対応方針: 確定デバイスでの完全動作確保
- 任意対応
  - 他環境での動作は Baseline Newly available による副次的効果として期待
- 除外対応
  - Internet Explorer・古いブラウザとの互換性は考慮しない
  - Baseline Newly available 未対応の機能は使用しない

<!-- PREMISE_END: browser-support -->

<!-- PREMISE_BEGIN: architecture-stability-requirements -->

- **変更が必要になる原因を事前に排除**: 疎結合な設計と安定した技術を選んで、アーキテクチャを変更する必要がないようにする
- **技術的な負債を予防**: 機能を追加するときに既存コードを大幅に変更する必要がある場合は設計の問題として、継続的に改善で対応する
- **局所的な変更で機能拡張**: 新機能は既存のアーキテクチャの枠組みの中で、部分的な変更で実現する

<!-- PREMISE_END: architecture-stability-requirements -->

CogitoWeave のデータアクセスは CRUD 中心で、複雑なクエリは GraphRAG 的な知識取得程度に限定される。バッチ操作は不要で、リアルタイム性要件もない。

## 論理

第一に、CogitoWeave のデータアクセスパターンは単純明快である。4 つのエンティティに対する基本的な作成・読み取り・更新・削除操作が中心であり、複雑な関係性クエリは稀な処理に留まる。

第二に、クライアント環境の統一性により API 設計が単純化される。Mac での更新・参照操作と iPhone での参照操作は、同一の API エンドポイントで対応可能であり、クライアント固有の API 分岐は不要である。

第三に、リアルタイム性要件の不在により、同期的な HTTP ベースの通信で十分である。WebSocket や Server-Sent Events 等の複雑な通信プロトコルは必要なく、標準的な HTTP リクエスト・レスポンスで要件を満たせる。

第四に、要件に対する技術適合性の観点から REST API が最適である。GraphQL は作成者には経験が多く学習負荷なしで実装可能だが、CRUD 中心のシンプルなデータアクセスに対してはオーバーエンジニアリングとなる。REST API の標準的なエンドポイント設計で十分要件を満たせる。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: api-tech-approach -->

API 技術方針として REST API を採用する。

<!-- GLOBAL_CONCLUSION_END: api-tech-approach -->
