<!-- markdownlint-disable MD024 -->

# C4 Model

## Physical 制約検出における技術配置

### 全体概要図

<!-- REFERENCE_BEGIN: physical-environment-overview-diagram -->

```mermaid
C4Deployment
    title 物理環境による配置概要図

    Person(user, "ユーザー")

    Deployment_Node(cloudflare, "Cloudflare 環境", "外部サービス") {
        Container(tunnel, "Tunnel + Access", "ネットワーク技術")
    }

    Deployment_Node(selfhost, "セルフホスト環境", "自宅サーバー") {
        Deployment_Node(docker, "Docker環境", "コンテナランタイム") {
            Container(frontend, "Frontend", "静的配信")
            Container(backend, "Backend + DB", "API + SQLite")
            Container(caddy, "Caddy", "リバースプロキシ")
            Container(fluent, "Fluent Bit", "ログ収集")
        }
        Deployment_Node(runner_env, "Self-hosted Runner", "CD実行環境") {
            Container(ansible, "Ansible", "インフラ管理")
        }
        Container(backup, "Backup", "sqlite3+gzip+rclone")
    }

    Deployment_Node(local_dev, "ローカル開発環境", "開発者環境") {
        Container(vite_proxy, "Vite proxy", "開発環境プロキシ")
    }

    Deployment_Node(github, "GitHub Actions 環境", "外部サービス") {
        Container(actions, "GitHub Actions", "CI実行")
    }

    Deployment_Node(monitoring, "外部監視サービス群", "外部サービス") {
        Container(gcp_mon, "Cloud Monitoring", "監視・ログ")
        Container(sentry, "Sentry", "APM・トレーシング")
    }

    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="2")
```

<!-- REFERENCE_END: physical-environment-overview-diagram -->

### 分割された専門特化図

#### 1. ネットワーク・セキュリティ

<!-- REFERENCE_BEGIN: physical-network-infrastructure-diagram -->

```mermaid
C4Deployment
    title ネットワーク・セキュリティ

    Person(user, "ユーザー", "思想家・学習者")

    Deployment_Node(cloudflare, "Cloudflare", "外部サービス") {
        Container(tunnel, "Tunnel+Access", "セキュリティゲートウェイ")
    }

    Deployment_Node(selfhost, "セルフホスト環境", "自宅サーバー") {
        Deployment_Node(docker, "Docker環境", "コンテナランタイム") {
            Container(caddy, "Caddy", "リバースプロキシ")
            Container(frontend, "Frontend Container", "静的配信")
            Container(backend, "Backend Container", "API提供")
        }
    }

    Rel(user, tunnel, "HTTPS接続")
    Rel(tunnel, caddy, "プロキシ経由")
    Rel(caddy, frontend, "静的ファイル配信")
    Rel(caddy, backend, "API プロキシ")

    UpdateRelStyle(user, tunnel, $offsetY="-20")
    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-network-infrastructure-diagram -->

#### 2. アプリケーション基盤

<!-- REFERENCE_BEGIN: physical-application-infrastructure-diagram -->

```mermaid
C4Deployment
    title アプリケーション基盤

    Deployment_Node(selfhost, "セルフホスト環境", "自宅サーバー") {
        Deployment_Node(docker, "Docker環境", "コンテナランタイム") {
            Container(frontend, "Frontend Container", "静的ファイル配信")
            Container(backend, "Backend Container", "ビジネスロジック処理")
            ContainerDb(database, "Database Container", "データストレージ")
        }
    }

    Rel(frontend, backend, "API呼び出し")
    Rel(backend, database, "データアクセス")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-application-infrastructure-diagram -->

#### 3. 開発基盤

<!-- REFERENCE_BEGIN: physical-development-infrastructure-diagram -->

```mermaid
C4Deployment
    title 開発基盤

    Deployment_Node(devserver, "開発環境", "ローカル開発") {
        Container(frontend_dev, "Frontend Dev", "開発中フロントエンド")
        Container(vite_proxy, "Vite proxy", "APIプロキシ")
        Container(backend_dev, "Backend Dev", "開発中バックエンド")
    }

    Rel(frontend_dev, vite_proxy, "開発時APIアクセス")
    Rel(vite_proxy, backend_dev, "APIプロキシ")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-development-infrastructure-diagram -->

#### 4. テスト・CI 基盤

<!-- REFERENCE_BEGIN: physical-testing-ci-infrastructure-diagram -->

```mermaid
C4Deployment
    title テスト・CI基盤

    Deployment_Node(github, "ソースコード管理", "コード管理") {
        Container(repo, "リポジトリ", "ソースコード")
    }

    Deployment_Node(cicd, "CI/CD", "GitHub Actions") {
        Container(actions, "GitHub Actions", "ビルド・テスト")
    }

    Deployment_Node(testenv, "テスト環境", "CI専用環境") {
        Container(frontend, "Frontend Container", "テスト対象")
        Container(backend, "Backend Container", "テスト対象")
    }

    Rel(actions, repo, "コード取得")
    Rel(actions, frontend, "フロントエンドテスト実行")
    Rel(actions, backend, "バックエンドテスト実行")

    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-testing-ci-infrastructure-diagram -->

#### 5. デプロイ基盤

<!-- REFERENCE_BEGIN: physical-deployment-infrastructure-diagram -->

```mermaid
C4Deployment
    title デプロイ基盤

    Deployment_Node(cicd, "CI/CD", "自動化") {
        Container(actions, "Self-hosted runner", "デプロイ実行")
    }

    Deployment_Node(selfhost, "デプロイ先", "自宅サーバー") {
        Deployment_Node(infra, "インフラ管理", "Ansible") {
            Container(ansible_playbook, "Ansible Playbook", "自動化")
        }
        Deployment_Node(docker, "Docker環境", "デプロイ対象") {
            Container(frontend, "Frontend Container", "デプロイ対象")
            Container(backend, "Backend Container", "デプロイ対象")
            Container(caddy, "Caddy", "デプロイ対象")
        }
    }

    Rel(actions, ansible_playbook, "デプロイ指示")
    Rel(ansible_playbook, frontend, "フロントエンド配置")
    Rel(ansible_playbook, backend, "バックエンド配置")
    Rel(ansible_playbook, caddy, "Caddy設定配置")

    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-deployment-infrastructure-diagram -->

#### 6. 監視基盤

<!-- REFERENCE_BEGIN: physical-monitoring-infrastructure-diagram -->

```mermaid
C4Deployment
    title 監視基盤

    Deployment_Node(selfhost, "本番環境", "自宅サーバー") {
        Container(frontend, "Frontend Container", "監視対象UI")
        Container(backend, "Backend Container", "監視対象API")
        Container(fluent, "Fluent Bit", "ログ収集エージェント")
    }

    Deployment_Node(monitoring, "監視システム", "外部サービス") {
        Container(sentry, "Sentry", "APM・トレーシング")
        Container(gcp, "Cloud Monitoring", "システム監視")
    }

    Rel(backend, fluent, "ログ出力")
    Rel(fluent, gcp, "ログ転送")
    Rel(backend, sentry, "バックエンドメトリクス・トレース送信")
    Rel(frontend, sentry, "フロントエンドエラー・パフォーマンス送信")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

<!-- REFERENCE_END: physical-monitoring-infrastructure-diagram -->

#### 7. 運用基盤

<!-- REFERENCE_BEGIN: physical-operations-infrastructure-diagram -->

```mermaid
C4Deployment
    title 運用基盤

    Deployment_Node(backup, "バックアップシステム", "sqlite3+gzip+rclone") {
        Container(backup_process, "バックアップ処理", "データ保護")
    }

    Deployment_Node(selfhost, "本番環境", "自宅サーバー") {
        ContainerDb(database, "Database Container", "バックアップ対象")
    }

    Rel(backup_process, database, "SQLite DBバックアップ")
```

<!-- REFERENCE_END: physical-operations-infrastructure-diagram -->
