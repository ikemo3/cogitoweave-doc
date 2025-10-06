---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "infrastructure-integrated-constraints"
    - "backend-deployment-decision"
    - "frontend-deployment-decision"
    - "development-production-environment"
    - "cicd-tech-selection"
    - "infra-management-selection"
  produces:
    - "deployment-specification"
---

# 環境設定・デプロイ仕様

## 概要

CogitoWeave システムの環境設定とデプロイメントの具体的な実装方針を定義する。開発環境セットアップ、Docker Compose 構成、CI/CD パイプライン、Ansible 構成管理を統一的に実装し、開発から本番まで一貫した環境を実現する。

## 前提制約

<!-- PREMISE_BEGIN: infrastructure-integrated-constraints -->

### 物理配置

- Cloudflare 環境: Tunnel + Access によるネットワーク技術
- セルフホスト環境
  - Docker 環境: Caddy(リバースプロキシ)、Fluent Bit(ログ収集)
  - Self-hosted Runner: Ansible(インフラ管理)
  - Backup: sqlite3+gzip+rclone
- ローカル開発環境: Vite proxy(開発環境プロキシ)
- GitHub Actions 環境: CI 実行
- 外部監視サービス群: Cloud Monitoring(監視・ログ)、Sentry(APM・トレーシング)

### CI/CD

- Git サービス: GitHub
- CI 環境での自動テスト実行
- PR マージ停止機能
- 修正・revert 機能
- E2E テストによる疎通確認(Playwright)

### ビルド・開発環境

- ビルドツール: Vite
- 開発環境プロキシ: Vite proxy

### ログ・監視

- ログ収集: Fluent Bit
- 構造化ログ分析ツール
- ログ集約・検索システム
- 監視・ログ: Google Cloud Monitoring
- システム稼働監視
- パフォーマンス監視
- APM・トレーシング: Sentry
- エラートラッキング
- 障害検知

### ネットワーク・セキュリティ

- HTTPS 通信・CORS 設定: Caddy
- リバースプロキシ: Caddy
- ネットワークゲートウェイ: Cloudflare Tunnel + Access

### コンテナ管理

- セルフホスト環境: 単一物理サーバーでの運用
- Docker 環境: コンテナレベルでの統合管理、単純なコンテナ構成での管理

### インフラ管理

- Ansible によるインフラ管理
- Self-hosted Runner による CD 実行環境

### バックアップ

- sqlite3+gzip+rclone による定期バックアップ
- ファイルシステムによるバックアップファイルの検証

### 不足技術要素

- 外部アラートシステム(PagerDuty)
- 定期実行スケジューラー(cron・systemd timer・Task Scheduler)

<!-- PREMISE_END: infrastructure-integrated-constraints -->

<!-- PREMISE_BEGIN: backend-deployment-decision -->

CogitoWeave システムのバックエンド・データベース配置方式として、B1「自宅サーバーで API を提供」+ D1「自宅サーバーで SQLite」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- PREMISE_END: backend-deployment-decision -->

<!-- PREMISE_BEGIN: frontend-deployment-decision -->

CogitoWeave システムのフロントエンド配置方式として、選択肢 F1「自宅サーバーで静的配信」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- PREMISE_END: frontend-deployment-decision -->

<!-- PREMISE_BEGIN: development-production-environment -->

CogitoWeave システムのフロントエンド・バックエンド統合技術として、開発時は Vite proxy、本番時は Caddy(Web サーバー兼リバースプロキシ)を採用する。

<!-- PREMISE_END: development-production-environment -->

## 論理

### Docker Compose 構成の論理

前提制約から、以下のサービスを Docker Compose で構成する必要がある。

**必須サービス**:

- **Caddy**: フロントエンド静的配信 + バックエンドリバースプロキシ
- **Backend**: FastAPI + uvicorn
- **Fluent Bit**: ログ収集・転送

**データ永続化**:

- SQLite データベースファイル用ボリューム
- バックアップファイル用ボリューム

### 開発環境セットアップの論理

開発者が初めてプロジェクトを clone したときの手順を明確化する必要がある。

**セットアップステップ**:

1. 前提ソフトウェアインストール(Docker, Node.js, Python)
2. 環境変数設定(.env.example からコピー)
3. 依存パッケージインストール(npm, pip)
4. データベース初期化(Alembic マイグレーション)

### CI/CD パイプラインの論理

CI(テスト・ビルド)と CD(デプロイ)を分離し、安全で効率的なデプロイを実現する。

**CI フロー**(通常ランナー):

1. テスト実行(pytest, vitest)
2. 型チェック(mypy, TypeScript)
3. リント(ruff, eslint)
4. ビルド(Vite, Docker)

**CD フロー**(self-hosted runners):

1. Docker イメージビルド
2. self-hosted runner で Ansible 実行
3. Docker Compose でデプロイ

### Ansible 適用範囲の論理

Ansible でインフラ全体を管理し、手動オペレーションを排除する。

**Ansible 管理対象**:

- Docker Compose の起動/停止/更新
- 環境変数ファイルの配布
- Caddyfile 設定の配布
- バックアップスクリプトの cron 設定
- ログ収集設定の配布

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: deployment-specification -->

### Docker Compose 構成

```yaml
# docker-compose.yml
version: "3.8"

services:
  # Caddy: フロントエンド静的配信 + バックエンドリバースプロキシ
  caddy:
    image: caddy:2-alpine
    container_name: cogitoweave-caddy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./frontend/dist:/usr/share/caddy # フロントエンド静的ファイル
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - backend
    restart: unless-stopped

  # Backend: FastAPI + uvicorn
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: cogitoweave-backend
    environment:
      - DATABASE_URL=sqlite:////data/cogitoweave.db
      - LLM_API_KEY=${LLM_API_KEY}
      - SENTRY_DSN=${SENTRY_DSN}
    volumes:
      - sqlite_data:/data # SQLiteデータベース永続化
      - backup_data:/backup # バックアップファイル
    restart: unless-stopped

  # Fluent Bit: ログ収集・転送
  fluent-bit:
    image: fluent/fluent-bit:latest
    container_name: cogitoweave-fluent-bit
    volumes:
      - ./fluent-bit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    environment:
      - GOOGLE_APPLICATION_CREDENTIALS=/fluent-bit/etc/gcp-credentials.json
    restart: unless-stopped

volumes:
  caddy_data:
  caddy_config:
  sqlite_data: # SQLiteデータベース
  backup_data: # バックアップファイル
```

### Caddyfile

```caddyfile
# Caddyfile
{
    # 開発環境: HTTPのみ
    # 本番環境: 自動HTTPS
}

# フロントエンド静的配信
:80 {
    # ルートはフロントエンド
    root * /usr/share/caddy
    file_server
    try_files {path} /index.html

    # /api/* はバックエンドへプロキシ
    handle /api/* {
        reverse_proxy backend:8000
    }

    # セキュリティヘッダー
    header {
        X-Content-Type-Options "nosniff"
        X-Frame-Options "DENY"
        X-XSS-Protection "1; mode=block"
        Strict-Transport-Security "max-age=31536000; includeSubDomains"
    }
}
```

### 開発環境セットアップ

#### 前提ソフトウェア

```bash
# 必須
- Docker 24.0+
- Docker Compose 2.0+
- Node.js 20+
- Python 3.11+
- Git

# 推奨
- pnpm (npm install -g pnpm)
- uv (pip install uv)
```

#### セットアップ手順

```bash
# 1. リポジトリクローン
git clone https://github.com/your-org/cogitoweave.git
cd cogitoweave

# 2. 環境変数設定
cp .env.example .env
# .envを編集してLLM_API_KEY等を設定

# 3. フロントエンド依存パッケージインストール
cd frontend
pnpm install

# 4. バックエンド依存パッケージインストール
cd ../backend
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt

# 5. データベース初期化
alembic upgrade head

# 6. 開発サーバー起動
# ターミナル1: バックエンド
cd backend
uvicorn main:app --reload --port 8000

# ターミナル2: フロントエンド
cd frontend
pnpm dev
```

`.env.example`:

```bash
# .env.example
# LLM API設定
LLM_API_KEY=your-api-key-here
LLM_API_BASE_URL=https://api.openai.com/v1

# Sentry設定(任意)
SENTRY_DSN=

# データベース
DATABASE_URL=sqlite:///./cogitoweave.db

# 開発環境フラグ
DEBUG=true
```

### CI/CD パイプライン

#### CI ワークフロー

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  frontend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install -g pnpm
      - run: pnpm install
        working-directory: frontend
      - run: pnpm test
        working-directory: frontend
      - run: pnpm type-check
        working-directory: frontend
      - run: pnpm lint
        working-directory: frontend
      - run: pnpm build
        working-directory: frontend

  backend-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install uv
      - run: uv pip install -r requirements.txt
        working-directory: backend
      - run: pytest
        working-directory: backend
      - run: mypy .
        working-directory: backend
      - run: ruff check .
        working-directory: backend

  api-type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install -g pnpm
      - run: pnpm install
        working-directory: frontend
      - run: pnpm run generate:api-types
        working-directory: frontend
      - run: git diff --exit-code
```

#### CD ワークフロー

```yaml
# .github/workflows/cd.yml
name: CD

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker images
        run: |
          docker compose build

      - name: Run Ansible playbook
        run: |
          ansible-playbook -i ansible/inventory.yml ansible/deploy.yml
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
```

### Ansible 構成

#### ディレクトリ構造

```plaintext
ansible/
├── inventory.yml          # インベントリ
├── deploy.yml            # デプロイplaybook
├── roles/
│   ├── docker/           # Docker関連
│   ├── app/              # アプリケーション配置
│   ├── caddy/            # Caddy設定
│   ├── backup/           # バックアップ設定
│   └── logging/          # ログ収集設定
└── group_vars/
    └── all.yml           # 共通変数
```

#### デプロイ Playbook

```yaml
# ansible/deploy.yml
---
- name: Deploy CogitoWeave
  hosts: production
  become: yes

  roles:
    - role: docker
      tags: [docker]

    - role: app
      tags: [app]

    - role: caddy
      tags: [caddy]

    - role: backup
      tags: [backup]

    - role: logging
      tags: [logging]

  tasks:
    - name: Deploy application with Docker Compose
      docker_compose:
        project_src: /opt/cogitoweave
        state: present
        pull: yes
      tags: [deploy]
```

#### App ロール

```yaml
# ansible/roles/app/tasks/main.yml
---
- name: Create app directory
  file:
    path: /opt/cogitoweave
    state: directory
    mode: "0755"

- name: Copy docker-compose.yml
  template:
    src: docker-compose.yml.j2
    dest: /opt/cogitoweave/docker-compose.yml
    mode: "0644"

- name: Copy .env file
  template:
    src: env.j2
    dest: /opt/cogitoweave/.env
    mode: "0600"

- name: Copy Caddyfile
  template:
    src: Caddyfile.j2
    dest: /opt/cogitoweave/Caddyfile
    mode: "0644"
```

#### Backup ロール

```yaml
# ansible/roles/backup/tasks/main.yml
---
- name: Install backup script
  template:
    src: backup.sh.j2
    dest: /opt/cogitoweave/backup.sh
    mode: "0755"

- name: Setup backup cron
  cron:
    name: "CogitoWeave daily backup"
    minute: "0"
    hour: "3"
    job: "/opt/cogitoweave/backup.sh >> /var/log/cogitoweave-backup.log 2>&1"
```

バックアップスクリプト:

```bash
# ansible/roles/backup/templates/backup.sh.j2
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/opt/cogitoweave/backups"
DATE=$(date +%Y%m%d_%H%M%S)
DB_FILE="/opt/cogitoweave/data/cogitoweave.db"

# SQLiteバックアップ
sqlite3 "$DB_FILE" ".backup ${BACKUP_DIR}/cogitoweave_${DATE}.db"

# gzip圧縮
gzip "${BACKUP_DIR}/cogitoweave_${DATE}.db"

# rcloneでクラウドストレージへアップロード
rclone copy "${BACKUP_DIR}/cogitoweave_${DATE}.db.gz" remote:cogitoweave-backups/

# 7日以上古いバックアップを削除
find "$BACKUP_DIR" -name "cogitoweave_*.db.gz" -mtime +7 -delete

echo "Backup completed: cogitoweave_${DATE}.db.gz"
```

### デプロイメントフロー

#### 本番デプロイ

```bash
# mainブランチへのpushで自動デプロイ
git push origin main

# GitHub Actionsが自動実行:
# 1. CI: テスト・型チェック・ビルド
# 2. CD (self-hosted runner):
#    a. Dockerイメージビルド
#    b. Ansible playbook実行
#    c. Docker Compose更新
```

#### 手動デプロイ

```bash
# Ansible経由
cd ansible
ansible-playbook -i inventory.yml deploy.yml

# または直接Docker Compose
cd /opt/cogitoweave
docker compose pull
docker compose up -d
```

### 実装原則

1. **Docker Compose 構成**: Caddy + Backend + Fluent Bit の 3 サービス
2. **データ永続化**: SQLite 用ボリューム、バックアップ用ボリューム
3. **開発環境**: Vite proxy、ホットリロード
4. **本番環境**: Caddy リバースプロキシ、自動 HTTPS
5. **CI/CD 分離**: CI(通常ランナー)、CD(self-hosted)
6. **Ansible 全管理**: Docker Compose、環境変数、Caddy、バックアップ cron
7. **自動バックアップ**: 日次 3 時、7 日間保持、rclone 自動アップロード

<!-- GLOBAL_CONCLUSION_END: deployment-specification -->
