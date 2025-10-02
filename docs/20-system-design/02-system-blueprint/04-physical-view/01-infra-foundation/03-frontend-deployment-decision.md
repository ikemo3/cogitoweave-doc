---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "existing-resource-constraints"
    - "technology-selection-criteria"
    - "hosting-options-analysis"
    - "security-requirements"
    - "foundation-tech-selection"
    - "backend-deployment-decision"
  produces:
    - "frontend-deployment-decision"
---

# フロントエンド配置決定

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

<!-- PREMISE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- PREMISE_END: foundation-tech-selection -->

<!-- PREMISE_BEGIN: backend-deployment-decision -->

CogitoWeave システムのバックエンド・データベース配置方式として、B1「自宅サーバーで API を提供」+ D1「自宅サーバーで SQLite」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- PREMISE_END: backend-deployment-decision -->

## 論理

第一に、プライバシー要件の観点で、F1(自宅サーバー)が最適である。完全プライベートなシステムにおいて、F2(CDN 併用)や F3(Cloudflare Pages)では外部インフラに配信が依存するため、「配信されたら困る」という要件に適合しない。F1 により完全な自己完結性を実現できる。

第二に、バックエンド配置との統合性において、F1(自宅サーバー)はバックエンドと同一環境での統一管理を可能にする。B1(自宅サーバー)と組み合わせることで、フロントエンド・バックエンドの一括運用により週 7 時間の時間制約に適合する。

第三に、既存リソース活用の観点で、F1(自宅サーバー)は追加コストを発生させない。F3(Cloudflare Pages)は無料枠内でも外部依存となり、既存リソース制約の最大限活用に反する。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-deployment-decision -->

CogitoWeave システムのフロントエンド配置方式として、選択肢 F1「自宅サーバーで静的配信」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- GLOBAL_CONCLUSION_END: frontend-deployment-decision -->

### 論理的根拠

完全プライベートシステムの要件、バックエンドとの統一管理、既存リソースの最大限活用により、F1 が最適な配置方式である。
