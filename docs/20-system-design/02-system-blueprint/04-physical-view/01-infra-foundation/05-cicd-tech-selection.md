---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "git-service-selection"
    - "hosting-options-analysis"
    - "technology-selection-criteria"
  produces:
    - "cicd-tech-selection"
---

# CI/CD 技術選択

## 前提

<!-- PREMISE_BEGIN: git-service-selection -->

Git サービスとして GitHub を採用する。

<!-- PREMISE_END: git-service-selection -->

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

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 論理

第一に、CI と CD の要件の違いにより、実行環境を分離する必要がある。CI はテスト・ビルド処理であり外部環境で実行可能だが、CD は自宅サーバーへのファイル配置・Docker Compose 実行が必要なため物理的なアクセスが必須である。

第二に、CI については GitHub Actions の通常ランナーが最適である。テスト・ビルド処理は外部環境で完結し、高速な実行環境と豊富なプリインストールツールにより効率的な品質保証が実現できる。セキュリティ面でも外部実行により自宅サーバーへの不要なアクセスを回避できる。

第三に、CD については GitHub Actions の Self-hosted runners が必須である。自宅サーバーへのファイル配置・Docker Compose 実行は物理的なアクセスが必要であり、外部サービスからの SSH 接続ではファイアウォール設定・VPN 構築等の複雑なネットワーク設定が必要となる。Self-hosted runners は自宅サーバー内から GitHub に接続するため、外部からのアクセス許可が不要である。

第四に、GitHub エコシステム内での統合により、CI・CD の連携が簡潔である。GitHub Actions により CI から CD への自然な移行、GitHub Secrets による安全な認証情報管理、既存の運用知識の活用が可能である。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: cicd-tech-selection -->

CogitoWeave システムの CI/CD 技術として、GitHub Actions を採用する。CI は通常ランナー、CD は self-hosted runners を使用する。

<!-- GLOBAL_CONCLUSION_END: cicd-tech-selection -->

### 論理的根拠

CI と CD の要件の違いに応じて実行環境を最適化することで、効率性とセキュリティを両立する。CI は外部環境の高速実行環境を活用し、CD は self-hosted runners により自宅サーバーへの物理的アクセス制限問題を解決する。GitHub エコシステム内での完結により、CI から CD への自然な移行と既存運用知識の活用が可能である。
