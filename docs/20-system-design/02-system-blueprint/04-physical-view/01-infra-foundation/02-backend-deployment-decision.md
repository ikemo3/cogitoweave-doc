---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "existing-resource-constraints"
    - "technology-selection-criteria"
    - "hosting-options-analysis"
    - "security-requirements"
    - "language-framework-selection"
  produces:
    - "backend-deployment-decision"
---

# バックエンド・データベース配置決定

## 前提

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: hosting-options-analysis -->

CogitoWeave システムのホスティング方式として、フロントエンド 3 選択肢(F1-F3)、バックエンド 2 選択肢(B1-B2)、データベース 2 選択肢(D1-D2)の組み合わせによる配置パターンが利用可能である。

- フロントエンド
  - 選択肢 F1: 自宅サーバーで静的配信
  - 選択肢 F2: 自宅サーバー + Cloudflare CDN
  - 選択肢 F3: Cloudflare Pages
- バックエンド
  - 選択肢 B1: 自宅サーバーで API を提供
  - 選択肢 B2: Cloudflare Workers で API を提供
- データベース
  - 選択肢 D1: 自宅サーバーで SQLite
  - 選択肢 D2: Cloudflare D1

<!-- PREMISE_END: hosting-options-analysis -->

<!-- PREMISE_BEGIN: security-requirements -->

- 必須対応として実装する
  - 外部 API 接続時の HTTPS 通信(LLM API・MCP 統合)
  - SQL インジェクションと XSS に対する基本的な対策の実装
  - 個人情報保護法に基づくプライバシー配慮設計
  - LLM API 利用規約・各種サービス規約の遵守
  - 攻撃対象面の限定(作成者個人環境のみ)によるリスク軽減
- 任意対応として実装する
  - 作成者本人のみがアクセスできる制御を Cloudflare Access で実装
  - 全通信の HTTPS 暗号化
- 実装対象から除外する
  - 高度なユーザー認証機能(プロトタイプ段階)
  - データ暗号化(プロトタイプ段階)
  - 詳細なアクセス制御(プロトタイプ段階)

<!-- PREMISE_END: security-requirements -->

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

## 論理

第一に、Python + FastAPI 技術スタックとの適合性を考慮すると、バックエンド選択肢 B2(Cloudflare Workers)が除外される。B2 の Python 対応はベータ段階であり、決定済みの Python + FastAPI 技術スタックの安定運用には適さない。この時点でバックエンドは B1(自宅サーバー)に確定する。

第二に、B1(自宅サーバー)確定により、データベース選択肢の合理性が変化する。D2(Cloudflare D1)は自宅サーバーとの地理的分離によりレイテンシが発生し、3 つの FastAPI サービスからの頻繁なアクセスに対して性能劣化を生じる。D1(自宅サーバーで SQLite)による同一環境内配置が最適である。

第三に、セキュリティ要件において、どちらの組み合わせも Cloudflare Access により個人環境限定を実現できるため、セキュリティ観点での差異はない。

第四に、既存リソース活用の観点で、B1 + D1 の組み合わせは既存の Ubuntu 環境を最大限活用できる。追加コストを発生させず、週 7 時間の時間制約に適合する。

第五に、3 サービス分離構成の管理において、Docker Compose による統一的なコンテナ管理が最適である。直接デプロイでは各サービスの依存関係管理が複雑になるが、Docker Compose により効率的な運用を実現できる。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: backend-deployment-decision -->

CogitoWeave システムのバックエンド・データベース配置方式として、B1「自宅サーバーで API を提供」+ D1「自宅サーバーで SQLite」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- GLOBAL_CONCLUSION_END: backend-deployment-decision -->

### 論理的根拠

Python + FastAPI 技術スタックの安定運用、同一環境内でのレイテンシ最小化、既存リソースの最大限活用、Docker Compose による効率的な管理により、B1 + D1 + コンテナ化が最適な配置方式である。
