---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts: []
  produces:
    - "log-collection-selection"
---

# ログ収集・転送技術選択

<!-- PREMISE_BEGIN: monitoring-tech-selection -->

CogitoWeave システムの監視・ログ技術として、Google Cloud Monitoring を採用する。

<!-- PREMISE_END: monitoring-tech-selection -->

<!-- PREMISE_BEGIN: container-tech-selection -->

CogitoWeave システムのコンテナ技術として、Docker + Docker Compose を採用する。

<!-- PREMISE_END: container-tech-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 前提

### 他設計書との接続点

Physical View で Google Cloud Monitoring を監視基盤として採用済みであり、Docker Compose をコンテナ技術として採用済みである。技術選択基準により慣れた技術の優先が確立されている。

### 確立された事実

- Google Cloud Monitoring への送信が必要である。
- Docker Compose 環境でのコンテナログ収集が必要である。
- Fluent Bit の運用経験が既に存在する。
- 自宅サーバー(Ubuntu)での動作が必要である。

### 判断基準

- 既存の運用知識を活用できることを重視する。
- Docker Compose 環境との親和性を確保することを優先する。
- 設定・保守の複雑度を適切なレベルに抑えることを確認する。

### スコープ境界

本判断ではログ収集・転送技術の基本選択のみを扱い、具体的な設定ファイルやログフォーマットは扱わない。

## 論理

第一に、Fluent Bit は既存の運用経験があり、技術選択基準の「慣れた技術の優先」に適合する。設定ファイル記述、プラグイン管理、トラブルシューティングの知識が既に蓄積されており、追加の習得負荷が最小である。

第二に、Fluent Bit は Docker Compose 環境で効率的に動作し、コンテナログの自動収集が可能である。Docker の標準ログドライバーとの統合により、アプリケーションコードの変更なしでログ収集を実現できる。サイドカーパターンでの配置により、各サービスのログを統一的に管理可能である。

第三に、Fluent Bit は Google Cloud Monitoring との連携が確立されており、専用プラグインにより効率的なログ転送を実現できる。Google Cloud Ops Agent と比較して設定の柔軟性が高く、カスタムログ処理やフィルタリングが容易である。軽量性により自宅サーバーでのリソース使用量も最小化される。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: log-collection-selection -->

CogitoWeave システムのログ収集・転送技術として、Fluent Bit を採用する。

<!-- GLOBAL_CONCLUSION_END: log-collection-selection -->

### 論理的根拠

Fluent Bit により既存の運用経験を活用しつつ、Docker Compose 環境でのコンテナログを効率的に収集・転送できる。Google Cloud Monitoring との連携により統一的なログ管理を実現し、軽量性と柔軟性を両立して自宅サーバー環境での最適なログ収集基盤を構築する。
