<!-- markdownlint-disable MD024 -->

# Container レベル実装契約集

## 概要

C4 モデルの Container レベルで定義された実装契約を統合する。フロントエンド Container、バックエンド Container、データベース Container の 3 つの実行可能単位における設計判断を集約し、実装の全体像を提供する。

## デプロイメント契約

<!-- REFERENCE_BEGIN: deployment-specification -->

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

<!-- REFERENCE_END: deployment-specification -->

## 結論

Container レベルの実装契約として、フロントエンド・バックエンド・データベースの 3 つの実行可能単位における設計判断を統合した。各 Container の技術スタック、アーキテクチャ方針、実装制約が明確化され、実装フェーズへの移行が可能となる。
