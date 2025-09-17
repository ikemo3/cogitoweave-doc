---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "cicd-tech-selection"
---

# CI/CD 技術選択

<!-- PREMISE_BEGIN: git-service-selection -->

Git サービスとして GitHub を採用する。

<!-- PREMISE_END: git-service-selection -->

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

Development View で GitHub を Git サービスとして選定済みであり、Physical View でセルフホスト方式を採用済みである。技術選択基準により慣れた技術の優先が確立されている。

### 確立された事実

- GitHub リポジトリが主要な開発基盤である。
- セルフホスト環境(自宅サーバー)へのデプロイが必要である。
- 外部 CI/CD サービスから自宅サーバーへの直接アクセスは制限される。
- GitHub Actions の基本的な使用経験が存在する。

### 判断基準

- GitHub との統合性を重視する。
- 既存の運用知識を活用できることを優先する。
- 設定・保守の複雑度を最小化することを確認する。

### スコープ境界

本判断では CI/CD プラットフォームの基本選択のみを扱い、具体的なワークフロー設定やデプロイ手順は扱わない。

## 論理

第一に、セルフホスト環境では外部 CI/CD サービスからの直接アクセスが制限され、GitHub Actions の Self-hosted runners が最適解となる。Jenkins や GitLab CI 等の外部サービスから自宅サーバーへの SSH 接続は、ファイアウォール設定・VPN 構築等の複雑なネットワーク設定が必要である。GitHub Actions の Self-hosted runners は自宅サーバー内から GitHub に接続するため、外部からのアクセス許可が不要である。

第二に、GitHub リポジトリとの統合により、追加のツール連携や認証設定が不要である。リポジトリ内でのワークフロー設定管理、プルリクエストとの自動連携、GitHub Secrets による安全な認証情報管理により、CI/CD パイプライン全体を GitHub エコシステム内で完結できる。

第三に、GitHub Actions は既存の運用知識を活用でき、技術選択基準の「慣れた技術の優先」に適合する。YAML 設定による CI/CD パイプライン定義、基本的なワークフロー構築の経験が既に存在し、追加の習得負荷が最小である。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: cicd-tech-selection -->

CogitoWeave システムの CI/CD 技術として、GitHub Actions を採用する。

<!-- GLOBAL_CONCLUSION_END: cicd-tech-selection -->

### 論理的根拠

セルフホスト環境において、GitHub Actions の Self-hosted runners により外部からのアクセス制限問題を回避し、自宅サーバー内から GitHub への接続で CI/CD を実現できる。GitHub エコシステム内での完結により、追加のツール連携・認証設定・ネットワーク設定が不要であり、既存の運用知識を活用して週 7 時間の時間制約に適合する効率的な運用が可能である。
