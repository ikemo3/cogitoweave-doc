# Physical View

## 1. 基盤インフラ技術選択

### 1. ホスティング方式選択

<!-- REFERENCE_BEGIN: hosting-options-analysis -->

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

<!-- REFERENCE_END: hosting-options-analysis -->

### 2. バックエンド・データベース配置決定

<!-- REFERENCE_BEGIN: backend-deployment-decision -->

CogitoWeave システムのバックエンド・データベース配置方式として、B1「自宅サーバーで API を提供」+ D1「自宅サーバーで SQLite」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- REFERENCE_END: backend-deployment-decision -->

### 3. フロントエンド配置決定

<!-- REFERENCE_BEGIN: frontend-deployment-decision -->

CogitoWeave システムのフロントエンド配置方式として、選択肢 F1「自宅サーバーで静的配信」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- REFERENCE_END: frontend-deployment-decision -->

### 4. 開発・本番環境構成選択

<!-- REFERENCE_BEGIN: development-production-environment -->

CogitoWeave システムのフロントエンド・バックエンド統合技術として、開発時は Vite proxy、本番時は Caddy(Web サーバー兼リバースプロキシ)を採用する。

<!-- REFERENCE_END: development-production-environment -->

### 5. CI/CD 技術選択

<!-- REFERENCE_BEGIN: cicd-tech-selection -->

CogitoWeave システムの CI/CD 技術として、GitHub Actions を採用する。CI は通常ランナー、CD は self-hosted runners を使用する。

<!-- REFERENCE_END: cicd-tech-selection -->

### 6. インフラ管理技術選択

<!-- REFERENCE_BEGIN: infra-management-selection -->

CogitoWeave システムのインフラ管理技術として、Ansible を採用する。

<!-- REFERENCE_END: infra-management-selection -->

### 7. バックアップ方法技術選択

<!-- REFERENCE_BEGIN: backup-method-tech-selection -->

バックアップ方法技術として **sqlite3 .backup + gzip + rclone** を採用する。

<!-- REFERENCE_END: backup-method-tech-selection -->

## 2. 監視・オブザーバビリティ技術選択

### 1. 監視・ログ技術選択

<!-- REFERENCE_BEGIN: monitoring-tech-selection -->

CogitoWeave システムの監視・ログ技術として、Google Cloud Monitoring を採用する。

<!-- REFERENCE_END: monitoring-tech-selection -->

### 2. ログ収集・転送技術選択

<!-- REFERENCE_BEGIN: log-collection-selection -->

CogitoWeave システムのログ収集・転送技術として、Docker Compose 環境でサイドカーパターンにより配置される Fluent Bit を採用する。

<!-- REFERENCE_END: log-collection-selection -->

### 3. APM 技術選択

<!-- REFERENCE_BEGIN: apm-selection -->

CogitoWeave システムの APM 技術として、Sentry を採用する。

<!-- REFERENCE_END: apm-selection -->

### 4. トレーシング技術選択

<!-- REFERENCE_BEGIN: tracing-selection -->

CogitoWeave システムのトレーシング技術として、Sentry Performance を採用する。

<!-- REFERENCE_END: tracing-selection -->

## 3. セキュリティ・ネットワーク技術選択

### 1. ネットワーク技術選択

<!-- REFERENCE_BEGIN: network-tech-selection -->

CogitoWeave システムのネットワーク技術として、Cloudflare Tunnel + Cloudflare Access を採用する。

<!-- REFERENCE_END: network-tech-selection -->
