---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "network-tech-selection"
---

# ネットワーク技術選択

<!-- PREMISE_BEGIN: hosting-method-selection -->

CogitoWeave システムのホスティング方式として、セルフホスト + Cloudflare サービスの組み合わせを採用する。

<!-- PREMISE_END: hosting-method-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 前提

### 他設計書との接続点

Physical View でセルフホスト + Cloudflare サービスの組み合わせが採用済みであり、技術選択基準により慣れた技術の優先が確立されている。

### 確立された事実

- Cloudflare Tunnel が自宅サーバーに既に設置済みである。
- 個人用途のみでアクセス制御が必要である。
- CDN 機能は個人用途では不要である。

### 判断基準

- 既存のインフラ構成を最大限活用することを重視する。
- セキュリティと利便性のバランスを確保することを優先する。
- 追加コスト・複雑度を最小化することを確認する。

### スコープ境界

本判断ではネットワーク・セキュリティ技術の基本選択のみを扱い、具体的な設定や運用手順は扱わない。

## 論理

第一に、既存の Cloudflare Tunnel により外部からの安全なアクセス経路が確立されており、追加のネットワーク設定が不要である。VPN 構築・ポート開放・ファイアウォール設定等の複雑なネットワーク構成を回避し、Cloudflare の管理画面から簡潔にアクセス制御を実現できる。

第二に、Cloudflare Access により個人用途に最適化されたゼロトラストアクセス制御を実現できる。Google OAuth 等の外部認証プロバイダーとの統合により、パスワード管理の負荷を削減しつつ、強固な認証を提供する。Basic Auth 等の単純な認証と比較して、セキュリティレベルが大幅に向上する。

第三に、この組み合わせは Cloudflare エコシステム内で完結し、既存の運用知識を活用できる。CDN・WAF・Load Balancing 等の高度な機能は個人用途では過剰であり、Tunnel + Access の組み合わせで必要十分なネットワーク機能を提供する。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: network-tech-selection -->

CogitoWeave システムのネットワーク技術として、Cloudflare Tunnel + Cloudflare Access を採用する。

<!-- GLOBAL_CONCLUSION_END: network-tech-selection -->

### 論理的根拠

既存の Cloudflare Tunnel により安全な外部アクセス経路を活用し、Cloudflare Access により個人用途に最適化されたゼロトラストアクセス制御を実現できる。Cloudflare エコシステム内での完結により追加のネットワーク設定・認証設定が不要であり、既存の運用知識を活用して必要十分なセキュリティレベルを効率的に提供する。
