---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "foundation-tech-selection"
    - "language-framework-selection"
  produces:
    - "git-service-selection"
---

# Git サービス選定

## 概要

CogitoWeave プロジェクトの開発効率性と運用安定性を両立する Git サービスを選定する。選定済みの技術スタックとの親和性、CI/CD 機能の充実度、ホスティング機能の品質を重視して決定する。

### 前提

<!-- PREMISE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- PREMISE_END: foundation-tech-selection -->

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

## 選択肢と判断基準

CogitoWeave プロジェクトの開発効率性と運用安定性を両立する Git サービスの選定を行う。一人開発環境における CI/CD 機能の充実度、エコシステムの豊富さ、開発者体験の优秀さ、ホスティング機能の品質を重視して決定する。

### 判断基準

- CI/CD 機能の充実度 (優先度高) - 継続的インテグレーション・デプロイメント機能
- エコシステムの豊富さ - ツール・サービス連携と技術スタックとの親和性
- 開発者体験 - 日常的な Git 操作の効率性と使いやすさ
- ホスティング機能 - 静的サイトホスティングなどの付加機能

### 選択肢

- GitHub: Microsoft が提供する世界最大の Git ホスティングサービス
- GitLab: GitLab Inc. が提供する統合 DevOps プラットフォーム
- Bitbucket: Atlassian が提供する Git リポジトリ管理サービス

## 各選択肢の詳細分析

各選択肢のアプローチと特性を分析し、CogitoWeave プロジェクトの要件への適合性を評価する。

### GitHub

Microsoft が運営する世界最大の Git ホスティングサービス。GitHub Actions による強力な CI/CD 機能と豊富なエコシステムを提供する。

- 利点
  - GitHub Actions による充実した CI/CD 機能
  - 豊富なアクションマーケットプレイスと簡潔な YAML 設定
  - 業界最大のサードパーティ連携と将来的な拡張性
  - 既存の使用経験による学習曲線の緩やかさ
  - GitHub Pages による優秀な静的ホスティング機能
- 欠点
  - 特に重大な欠点はない

### GitLab

GitLab Inc. が提供する統合 DevOps プラットフォーム。コード管理から CI/CD、監視まで一体化したサービスを提供する。

- 利点
  - GitLab CI/CD による包括的な機能と企業レベルの要件対応
  - 統合的なプラットフォームで追加ツールへの依存最小化
  - セキュリティスキャンや監視機能の充実
  - GitLab Pages による静的ホスティング機能
- 欠点
  - 設定の複雑度が GitHub Actions より高い
  - 多機能なインターフェースによる学習曲線の急峻さ
  - 外部サービスとの連携が GitHub より限定的

### Bitbucket

Atlassian が提供する Git リポジトリ管理サービス。Jira、Confluence との統合を重視した設計。

- 利点
  - Bitbucket Pipelines による Docker ベースの CI/CD 機能
  - Atlassian 製品群との強力な統合
  - Node.js と Python の十分なサポート
- 欠点
  - CI/CD 機能が GitHub Actions や GitLab CI/CD より限定的
  - 外部サービス連携が GitHub より少ない
  - 静的サイトホスティング機能が提供されていない

## 判断基準による評価

### CI/CD 機能の比較

GitHub Actions は最も充実している。豊富なアクション・マーケットプレイス・簡潔な YAML 設定により強力な CI/CD 環境を提供する。GitLab CI/CD も高機能だが設定がやや複雑である。Bitbucket Pipelines は機能的に劣る。

### エコシステムの比較

選定技術スタック(Solid、TanStack Router、Vite、Python、FastAPI)との連携において大差はない。3 サービス全てが Node.js と Python の CI/CD 環境を十分にサポートしており、必要な連携機能は確保されている。

### ホスティング機能の比較

GitHub Pages が最も優秀である。設定が簡潔で多様な静的サイトジェネレーターに対応している。GitLab Pages も同等の機能を提供するが設定がやや複雑である。Bitbucket は静的ホスティング機能が限定的である。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: git-service-selection -->

Git サービスとして GitHub を採用する。

<!-- GLOBAL_CONCLUSION_END: git-service-selection -->

### 選択根拠

GitHub Actions による充実した CI/CD 機能、豊富なアクションマーケットプレイス、業界最大のサードパーティ連携、既存の使用経験による学習曲線の緩やかさ、GitHub Pages による優秀な静的ホスティング機能により、CogitoWeave プロジェクトの要件を最適に満たす。GitLab は統合性に優れるが設定が複雑であり、Bitbucket は静的ホスティング機能が無く CI/CD 機能も限定的なため、GitHub が最適選択となる。
