---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "frontend-deployment-decision"
    - "backend-deployment-decision"
    - "foundation-tech-selection"
    - "technology-selection-criteria"
  produces:
    - "development-production-environment"
---

# フロントエンド・バックエンド統合技術選択

## 概要

フロントエンド(F1: 自宅サーバー)とバックエンド(B1: 自宅サーバー + Docker Compose)の配置方式決定を受け、Solid + Vite フロントエンドと FastAPI × 3 サービスバックエンドの統合技術を選択する。開発時と本番時のフロントエンド・バックエンド間 API 通信を実現する技術的手段を決定することが目的である。

### 前提

<!-- PREMISE_BEGIN: frontend-deployment-decision -->

CogitoWeave システムのフロントエンド配置方式として、選択肢 F1「自宅サーバーで静的配信」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- PREMISE_END: frontend-deployment-decision -->

<!-- PREMISE_BEGIN: backend-deployment-decision -->

CogitoWeave システムのバックエンド・データベース配置方式として、B1「自宅サーバーで API を提供」+ D1「自宅サーバーで SQLite」を採用し、Docker + Docker Compose によるコンテナ化配置を行う。

<!-- PREMISE_END: backend-deployment-decision -->

<!-- PREMISE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- PREMISE_END: foundation-tech-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 選択肢と判断基準

フロントエンド・バックエンド統合において、開発時の API 通信方式と本番時の Web サーバー・プロキシ技術を選択する。

### 判断基準

- **開発効率性**: ホットリロード速度、CORS 問題回避、設定簡素性 (優先度高)
- **統合簡素性**: フロントエンド・バックエンド間の通信設定の簡潔性 (優先度高)
- **運用負荷**: 本番環境でのプロキシ設定複雑度、SSL 管理負荷 (優先度高)
- **週 7 時間制約適合性**: 学習コスト、トラブルシューティング容易性

### 選択肢

- **開発時統合方式**: Vite proxy vs Docker Compose 内統合
- **本番時統合技術**: nginx vs Caddy (Web サーバー兼リバースプロキシ)

## 各選択肢の詳細分析

開発時統合方式と本番時統合技術について、それぞれの特徴と適用性を分析する。

### 開発時統合方式の選択肢

#### Vite proxy

Vite dev server の proxy 機能により、フロントエンドからの API 通信を Docker で動作する FastAPI サービスに転送する方式。

- 利点
  - Vite dev server の軽快なホットリロードを活用
  - CORS 問題を設定レベルで解決
  - Mac のネイティブ Node.js 環境で高速動作
  - フロントエンド・バックエンドの独立開発が可能
- 欠点
  - 本番環境との差異が存在
  - Vite 設定ファイルでの proxy 設定が必要
  - Mac 環境と Docker 環境の混在管理

#### Docker Compose 内統合

フロントエンドも含む全サービスを Docker Compose 内で動作させ、コンテナ間通信で API 連携する方式。

- 利点
  - 開発・本番環境の完全一致
  - 全サービスの統一的な管理
  - 環境差異によるトラブル回避
- 欠点
  - Vite のホットリロードが重くなる可能性
  - ボリュームマウント設定が複雑
  - Docker 環境でのフロントエンド開発体験低下

### 本番時統合技術の選択肢

#### nginx

高性能・実績豊富な Web サーバーによる静的ファイル配信と API リバースプロキシ。

- 利点
  - 高いパフォーマンスと安定性
  - 豊富なドキュメントとコミュニティ
  - 詳細な設定制御が可能
- 欠点
  - 設定ファイルの複雑性
  - SSL 証明書の手動管理が必要
  - リバースプロキシ設定の学習コスト

#### Caddy

設定簡素性を重視したモダン Web サーバー兼リバースプロキシ。

- 利点
  - 設定ファイルが非常にシンプル
  - SSL 証明書の自動取得・更新
  - 静的配信と API プロキシの統合設定が直感的
- 欠点
  - nginx 比でパフォーマンスが劣る可能性
  - コミュニティサイズが小さい
  - 詳細制御の柔軟性に限界

## 判断基準による評価

### 開発効率性の比較

**Vite proxy**が優位である。Mac ネイティブ環境での Vite dev server の軽快性は開発体験に直結し、CORS 問題の解決も設定レベルで完結する。Docker Compose 内統合では Vite ホットリロードが重くなり、開発効率が低下する。

### 統合簡素性の比較

**Vite proxy + Caddy**が優位である。開発時は vite.config.ts での proxy 設定、本番時は Caddyfile での静的配信+API プロキシ設定と、両者とも設定が簡潔である。nginx の複雑な設定に対し、統合設定の簡素性が際立つ。

### 運用負荷の比較

**Caddy**が優位である。nginx の複雑な設定に対し、Caddy は最小限の設定で Web サーバーとリバースプロキシを統合実現する。SSL 証明書の自動管理により、週 7 時間制約下での運用負荷を大幅に軽減する。

### 週 7 時間制約適合性の比較

**Vite proxy + Caddy**の組み合わせが最適である。Vite proxy は追加学習コストが最小限で、Caddy は設定トラブルの発生確率が低い。nginx の設定習得は時間制約に対してコストが高い。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: development-production-environment -->

CogitoWeave システムのフロントエンド・バックエンド統合技術として、開発時は Vite proxy、本番時は Caddy(Web サーバー兼リバースプロキシ)を採用する。

<!-- GLOBAL_CONCLUSION_END: development-production-environment -->

### 選択根拠

開発効率性(Vite proxy)と統合簡素性(Caddy)が判断の決定要因である。週 7 時間制約下において、Vite dev server の軽快性と Caddy の設定簡素性により、フロントエンド・バックエンド統合の両面で最適な技術選択を実現できる。nginx の高性能性よりも設定簡素性を優先し、Docker Compose 内統合の環境統一性よりも開発効率性を優先する選択である。
