---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "technology-selection-criteria"
    - "foundation-tech-selection"
    - "development-constraints"
  produces:
    - "http-api-selection"
---

# HTTP 通信・API 連携選択

## 前提

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- PREMISE_END: foundation-tech-selection -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

TanStack Router は純粋なルーティングライブラリであり、内蔵のキャッシュ機能を持つが HTTP 通信ライブラリは含まれない。SolidJS も HTTP 通信機能は内蔵していないため、別途選択が必要である。SolidJS のリアクティブシステムは fetch API と相性が良く、createResource や createSignal を使用して非同期データの状態管理を効率的に行える。

## 論理

第一に、TanStack Router の公式ドキュメントによると、内蔵のルーターキャッシュ機能が小規模から中規模のアプリケーションに適しており、重複排除、プリロード、バックグラウンド更新を自動で提供する。このキャッシュ機能は「ルート間でのデータ共有が少ないアプリケーションに最適」であり、CogitoWeave の状態管理方針「画面間状態共有は実装しない」と完全に一致する。

第二に、技術選択基準である「外部ライブラリは適度に制限」「シンプルで理解しやすい構造」の要件を満たすためには、追加の外部依存を避けることが重要である。fetch API はブラウザ標準であり、ゼロ依存でありながら現代的な Promise ベースの API を提供する。axios や ky のような外部ライブラリは機能は豊富だが、TanStack Router の内蔵キャッシュと組み合わせる場合、機能重複により複雑性が増大する。

第三に、TanStack Router の内蔵キャッシュと fetch API の組み合わせは、シンプルな技術構成でありながら必要十分な機能を提供する。fetch API による HTTP 通信と TanStack Router による自動キャッシュ・重複排除により、追加のライブラリなしで効率的なデータ管理が実現される。さらに SolidJS の createResource は fetch API を直接受け取ることができ、リアクティブなデータフェッチングとサスペンス機能を自動で提供する。この技術統合により「一人で管理可能な複雑度」の制約を満たしながら、現代的な開発体験を提供する。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: http-api-selection -->

HTTP 通信・API 連携技術として fetch API を採用する。

<!-- GLOBAL_CONCLUSION_END: http-api-selection -->

### 論理的根拠

技術選択基準の「外部ライブラリは適度に制限」「シンプルで理解しやすい構造」への適合により、ブラウザ標準の fetch API が最適である。TanStack Router の内蔵キャッシュ機能との組み合わせにより、追加の外部依存なしで重複排除・プリロード・バックグラウンド更新が自動提供される。この構成は CogitoWeave の「画面間状態共有なし」の方針と完全に一致し、一人開発での管理可能性を保ちながら必要十分な機能を実現する。
