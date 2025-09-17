---
doc_type: "pattern-step"
status: "draft"
depends:
  contracts: []
  produces:
    - "physical-view"
---

# Physical View

## 前提

Kruchten 4+1 アーキテクチャビューモデルの Physical View は非機能要件の物理的実現に焦点を当てる。

> The physical architecture takes into account primarily the non-functional requirements of the system such as availability, reliability (fault-tolerance), performance (throughput), and scalability. The software executes on a network of computers, or processing nodes (or just nodes for short). The various elements identified —networks, processes, tasks, and objects— need to be mapped onto the various nodes.

- **関心事**: 非機能要件の物理的実現 - 可用性、信頼性、性能、拡張性
- **時間軸**: 配備・運用時
- **視点**: インフラエンジニア、運用担当者
- **成果物**: デプロイメント図、ネットワーク図、ノード構成

## 論理

Physical View は論理設計と実行時設計を物理的な配置に変換するプロセスである。クラウドネイティブなインフラ設計により、非機能要件を満たす物理的な基盤を構築する。

プロセス設計と開発構造を物理リソースにマッピングし、可用性・性能・コストを最適化した運用可能なシステムを実現する。

## 結論

Physical View では以下の成果物を段階的に作成する:

### インフラ技術選択

1. **[ホスティング方式選択](01-hosting-method-selection.md)** - セルフホスト/マネージドサービス/分散構成等の運用方式を選択する
2. **[コンテナ技術選択](02-container-tech-selection.md)** - コンテナランタイムとオーケストレーション技術を選択する
3. **[CI/CD 技術選択](03-cicd-tech-selection.md)** - 継続的インテグレーション・デプロイメント技術を選択する
4. **[インフラ管理技術選択](04-infra-management-selection.md)** - インフラストラクチャ管理・自動化技術を選択する
5. **[監視・ログ技術選択](05-monitoring-tech-selection.md)** - システム監視・ログ管理技術を選択する
6. **[ネットワーク技術選択](06-network-tech-selection.md)** - 負荷分散・CDN・セキュリティ技術を選択する
7. **[ログ収集・転送技術選択](07-log-collection-selection.md)** - アプリケーションログを監視基盤に送信する技術を選択する
8. **[APM 技術選択](08-apm-selection.md)** - アプリケーション性能監視機能の提供技術を選択する
9. **[トレーシング技術選択](09-tracing-selection.md)** - 分散トレーシング機能の提供技術を選択する
10. **[バックアップ方法技術選択](10-backup-method-selection.md)** - データ保護のためのバックアップ実行技術を選択する
11. **バックアップ先技術選択** - バックアップデータの保存先技術を選択する
12. **シークレット管理技術選択** - 認証情報・API キー等の安全な管理技術を選択する
13. **SSL 証明書管理技術選択** - HTTPS 通信のための証明書管理技術を選択する

### 本プロジェクトでの適用範囲

上記のうち、7-10 については通常通り検討・実装する。11-13 については個人開発プロジェクトの性質上、本設計書では詳細な検討は省略している。ただし、企業システムや大規模システムでは重要な選択項目となるため、技術選択項目として記録している。

Process View での設計要件と Development View での技術選択を受けて、物理的な運用基盤に必要な具体的技術を選定する。C4 のデプロイメント設計は別途契約ファイルで実施する。
