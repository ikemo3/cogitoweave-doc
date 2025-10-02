---
doc_type: "pattern-step"
status: "draft"
viz_exclude: true
depends:
  contracts: []
  produces: []
---

# Physical View

## 目次

1. [概要](#概要)
2. [処理フロー](#処理フロー)
3. [ステップ 1: 基盤インフラ技術選択](#ステップ-1-基盤インフラ技術選択)
4. [ステップ 2: 監視・オブザーバビリティ技術選択](#ステップ-2-監視オブザーバビリティ技術選択)
5. [ステップ 3: セキュリティ・ネットワーク技術選択](#ステップ-3-セキュリティネットワーク技術選択)

## 概要

Kruchten 4+1 アーキテクチャビューモデルの Physical View は非機能要件の物理的実現に焦点を当てる。

> The physical architecture takes into account primarily the non-functional requirements of the system such as availability, reliability (fault-tolerance), performance (throughput), and scalability. The software executes on a network of computers, or processing nodes (or just nodes for short). The various elements identified —networks, processes, tasks, and objects— need to be mapped onto the various nodes.

- **関心事**: 非機能要件の物理的実現 - 可用性、信頼性、性能、拡張性
- **時間軸**: 配備・運用時
- **視点**: インフラエンジニア、運用担当者
- **成果物**: デプロイメント図、ネットワーク図、ノード構成

Physical View は論理設計と実行時設計を物理的な配置に変換するプロセスである。クラウドネイティブなインフラ設計により、非機能要件を満たす物理的な基盤を構築する。

プロセス設計と開発構造を物理リソースにマッピングし、可用性・性能・コストを最適化した運用可能なシステムを実現する。

## 処理フロー

TBD

### ステップ 1: 基盤インフラ技術選択

**[基盤インフラ技術選択](01-infra-foundation/README.md)** - ホスティングから CI/CD、バックアップまでの基盤技術を体系的に決定する

### ステップ 2: 監視・オブザーバビリティ技術選択

**[監視・オブザーバビリティ技術選択](02-observability/README.md)** - システムの健全性と性能を継続的に把握するための技術スタックを選定する

### ステップ 3: セキュリティ・ネットワーク技術選択

**[セキュリティ・ネットワーク技術選択](03-security-network/README.md)** - システムの安全性と可用性を確保するための技術スタックを選定する

### 本プロジェクトでの適用範囲

上記 15 項目のうち、ステップ 1 の基盤インフラ技術選択の 1-7 項目とステップ 2 の監視・オブザーバビリティ技術選択の全 4 項目については通常通り検討・実装する。ステップ 1 のバックアップ先技術選択、ステップ 3 のシークレット管理と SSL 証明書管理については個人開発プロジェクトの性質上、本設計書では詳細な検討は省略している。ただし、企業システムや大規模システムでは重要な選択項目となるため、技術選択項目として記録している。

Process View での設計要件と Development View での技術選択を受けて、物理的な運用基盤に必要な具体的技術を選定する。C4 のデプロイメント設計は別途契約ファイルで実施する。
