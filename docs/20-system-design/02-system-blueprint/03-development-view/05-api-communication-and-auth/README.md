---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "platform-constraints"
    - "data-access"
  produces:
    - "api-communication-auth"
---

# API 通信方式・認証認可

## 前提

フロントエンドとバックエンド間、または外部サービスとの通信方式と認証認可方式を決定する。API を使わないサーバサイドレンダリングの場合は認証認可の仕組みが変わるため、通信方式と認証認可をセットで検討する。

### 既存決定事項

<!-- PREMISE_BEGIN: platform-constraints -->

1. **Mac 環境制約**: Chrome 上で情報整理機能が完全動作すること
2. **iPhone 環境制約**: Safari 上で情報参照機能が基本動作すること
3. **Web 標準採用基準**: Baseline Newly available を技術選択の判断基準とすること
4. **マルチデバイス前提**: Web 技術による複数デバイス間データ共有を前提とすること

これ以外の基本セキュリティ対策（HTTPS、SQLi/XSS 対策等）、現代 Web 標準性能要件、個人情報保護法等の法規制遵守、個人環境限定によるセキュリティリスク軽減は現代 Web 開発標準に準拠する。

<!-- PREMISE_END: platform-constraints -->

<!-- PREMISE_BEGIN: data-access -->

**ORM 経由・同期処理・シンプル整合性管理**を採用する。

- データアクセス: ORM 経由
- 処理方式: バックエンド内は同期処理
- トランザクション: 考慮不要
- データ整合性: 参照制約による関連データ削除制限
- アクセス制御: 不要（Cloudflare 保護のみ）

<!-- PREMISE_END: data-access -->

## 論理

### レンダリング・通信方式選択基準

CogitoWeave の要件:

1. **視覚的・空間的 UI**: 概念管理ページでリッチなインタラクティブ機能が必要
2. **一人開発・開発コスト重視**: 複雑な構成は避ける
3. **個人使用**: 複雑な認証認可は不要

### レンダリング方式比較

1. **SSG + CSR なし**: 純粋 SSG
2. **SSG + CSR あり**: SSG + CSR
3. **SSR + CSR なし**: 純粋 SSR
4. **SSR + CSR あり**: SSR + CSR
5. **静的 HTML + CSR なし**: 完全静的サイト
6. **静的 HTML + CSR あり**: 静的 HTML + CSR
7. **ハイブリッド + CSR あり**: ページごとに SSG/SSR 選択（Astro 等）

**選択**: 静的 HTML + CSR

理由: 一人開発・リソース制約のため依存関係管理の負荷を避ける必要があり、ビルドプロセスを排除することで開発・デプロイの複雑性を最小化し、LLM による開発支援を受けやすいシンプルな技術構成とし、同時に視覚的 UI に必要な JavaScript 対応を実現する。

### アプリケーション構成比較

- **MPA**: 複数ページ間の遷移
- **SPA**: 単一ページ内でのコンテンツ切り替え
- **MPA + ページ内 SPA**: ページ間は MPA、必要な部分のみ SPA 的機能

**選択**: MPA + ページ内 SPA

理由: 一人開発・週 7 時間の制約下で開発コストを最小化する必要があり、全画面 SPA では状態管理の複雑性が増大するため、ページ分離により管理負荷を軽減し、視覚的・空間的 UI 要件がある概念管理ページのみ SPA 的機能を提供する。

### API 通信・認証認可

**選択**: REST API・認証認可なし

理由: 個人使用のためユーザー管理・権限制御が不要である。CQRS 設計において各操作の独立性を重視するため REST の操作別エンドポイントが適合する。GraphQL の複雑なリレーション統合機能は要件に対して過剰である。Cloudflare 保護により基本的な外部攻撃対策は実現される。

## 結論

<!-- FOUNDATION_BEGIN: api-communication-auth -->

**静的 HTML + CSR・MPA + ページ内 SPA・REST API・認証認可なし**を採用する。

- レンダリング: 静的 HTML + CSR（手書き HTML + クライアントサイドインタラクティブ）
- 構成: MPA + ページ内 SPA（概念管理ページのみリッチ UI）
- API 通信: REST API
- 認証認可: なし（Cloudflare 保護のみ）

<!-- FOUNDATION_END: api-communication-auth -->

### 開発コスト最適化

1. **シンプル構成**: SSG + MPA で管理負荷軽減
2. **部分的リッチ UI**: 必要な機能のみ SPA 的実装
3. **認証省略**: 個人使用による複雑性排除
