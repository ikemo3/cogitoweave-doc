---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "existing-resource-constraints"
    - "budget-time-constraints"
    - "service-boundaries-decision"
    - "foundation-tech-selection"
  produces:
    - "hosting-options-analysis"
---

# ホスティング方式選択肢分析

<!-- PREMISE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- PREMISE_END: existing-resource-constraints -->

<!-- PREMISE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- PREMISE_END: budget-time-constraints -->

<!-- PREMISE_BEGIN: service-boundaries-decision -->

技術的観点からの最適解は**3 サービス完全分離**となる：

- **コア知識管理サービス**: 安定した CRUD 処理を独立プロセスで実行し、長期安定リリースサイクルで運用
- **LLM 統合サービス**: 外部 API 依存を独立プロセスで分離し、技術進歩に応じた中期リリースサイクルで運用
- **コンテンツ取得サービス**: Web スクレイピングを独立プロセスで分離し、サイト変更に応じた短期リリースサイクルで運用

この構成により実行時の責務分離とリリースサイクルの最適化を両立し、各サービスの独立性を確保できる。

<!-- PREMISE_END: service-boundaries-decision -->

<!-- PREMISE_BEGIN: foundation-tech-selection -->

選定技術スタック:

- フレームワーク: Solid + TanStack Router
- ビルドツール: Vite
- CSS 手法: Tailwind CSS + CSS Modules ハイブリッド構成
- テスト: Vitest + Testing Library + Playwright
- UI コンポーネント: Tailwind CSS ベースのカスタム実装

パフォーマンス要件による客観的評価、外部ライブラリ制限、個人最適化による柔軟性を段階的に適用した結果、新しい技術体験とシンプルで直接的な操作体系を提供する技術スタックを採用する。

<!-- PREMISE_END: foundation-tech-selection -->

## 概要

Why 系制約(既存リソース・予算制約)、Process View(サービス分離構成)、Development View(技術スタック)の制約を統合し、利用可能なホスティング選択肢を特定する。本分析では既存リソース制約下での利用可能な選択肢の洗い出しのみを行い、具体的な選択判断は後続ステップで実施する。

## 選択肢と判断基準

### 判断基準

- **既存リソース制約**: 既存リソース制約の範囲内であること
- **技術適合性**: 技術スタックとの適合性があること
- **運用バランス**: 運用負荷とコストのバランスが適切であること

## 各選択肢の詳細分析

### フロントエンド配置選択肢

既存リソース制約と CSR 技術スタックにより、以下の選択肢が利用可能である。

#### 選択肢 F1: 自宅サーバーで静的配信

- **配置方法**: 自宅サーバーで静的ファイルを直接配信
- **技術的適合性**: Vite ビルド成果物の配信に完全対応
- **既存リソース活用**: 自宅サーバーの Web サーバー機能を活用

#### 選択肢 F2: 自宅サーバー + Cloudflare CDN

- **配置方法**: 自宅サーバーを Origin として Cloudflare CDN 経由で配信
- **技術的適合性**: 静的ファイルの CDN 配信に完全対応
- **既存リソース活用**: 自宅サーバーと Cloudflare サービスの組み合わせ

#### 選択肢 F3: Cloudflare Pages

- **配置方法**: Cloudflare Pages による静的サイトホスティング
- **技術的適合性**: Vite ビルド成果物のデプロイに完全対応
- **既存リソース活用**: Cloudflare サービスの Pages 機能を活用

### バックエンド配置選択肢

既存リソース制約と Python + FastAPI 技術スタックにより、以下の選択肢が利用可能である。

#### 選択肢 B1: 自宅サーバーで API を提供

- **配置方法**: 自宅サーバーで 3 サービス構成による API 提供
- **技術的適合性**: Python + FastAPI + SQLite 構成に完全対応
- **既存リソース活用**: 自宅サーバーのランタイム環境を活用

#### 選択肢 B2: Cloudflare Workers で API を提供

- **配置方法**: Cloudflare Workers による API 提供
- **技術的適合性**: JavaScript ランタイムのため Python 技術スタックとの適合性に課題
- **既存リソース活用**: Cloudflare サービスの Workers 機能を活用

### データベース配置選択肢

既存リソース制約と技術スタック決定により、以下の選択肢が利用可能である。

#### 選択肢 D1: 自宅サーバーで SQLite

- **配置方法**: 自宅サーバーで SQLite ファイルによるデータ管理
- **技術的適合性**: 決定済み SQLite 技術スタックに完全対応
- **既存リソース活用**: 自宅サーバーのストレージを活用

#### 選択肢 D2: Cloudflare D1

- **配置方法**: Cloudflare D1 による分散 SQLite データベース
- **技術的適合性**: SQLite 互換性を持つクラウドデータベース
- **既存リソース活用**: Cloudflare サービスの D1 機能を活用

## 判断基準による評価

本ステップでは選択肢の洗い出しのみを行い、具体的な評価は後続ステップで実施する。

### 選択肢の特定結果

すべての選択肢が既存リソース制約の範囲内で利用可能であり、技術スタックとの適合性も確認された。

## 最終選択と根拠

### 分析結果

<!-- GLOBAL_CONCLUSION_BEGIN: hosting-options-analysis -->

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

<!-- GLOBAL_CONCLUSION_END: hosting-options-analysis -->

### 結論

確定した選択肢を基に、バックエンド・データベース配置決定(ステップ 2)とフロントエンド配置決定(ステップ 3)において具体的な配置方式を決定する。
