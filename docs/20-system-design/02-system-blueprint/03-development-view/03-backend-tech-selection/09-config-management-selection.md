---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "security-requirements"
  produces:
    - "config-management-tech-selection"
---

# バックエンド設定管理技術選択

## 前提

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: security-requirements -->

- 必須対応として実装する
  - 外部 API 接続時の HTTPS 通信(LLM API・MCP 統合)
  - SQL インジェクションと XSS に対する基本的な対策の実装
  - 個人情報保護法に基づくプライバシー配慮設計
  - LLM API 利用規約・各種サービス規約の遵守
  - 攻撃対象面の限定(作成者個人環境のみ)によるリスク軽減
- 任意対応として実装する
  - 作成者本人のみがアクセスできる制御を Cloudflare Access で実装
  - 全通信の HTTPS 暗号化
- 実装対象から除外する
  - 高度なユーザー認証機能(プロトタイプ段階)
  - データ暗号化(プロトタイプ段階)
  - 詳細なアクセス制御(プロトタイプ段階)

<!-- PREMISE_END: security-requirements -->

CogitoWeave システムのバックエンドにおける環境変数・設定ファイル管理技術の選択。個人用途であり、ローカル開発と本番環境のみの二環境構成で、LLM API、データベース、外部サービス、アプリケーション設定を対象とする。

## 論理

第一に、一人開発制約下では管理負荷の軽減が最重要である。複雑な設定管理システムは理解・保守コストが高く、開発効率を阻害する。

第二に、個人用途でありながら基本的なセキュリティ要件は満たす必要がある。機密情報の環境分離と、開発・本番環境での設定値切り替えの確実性が求められる。

第三に、Python + FastAPI エコシステムとの親和性が高く、追加の習得が必要ない手法が望ましい。業界標準に準拠することで、将来の拡張性も確保できる。

第四に、本番環境でのデプロイ方法によらず一貫して動作する手法が求められる。PaaS 環境では管理画面で環境変数を設定し、Docker 環境では compose.yml が .env ファイルを読み込んで環境変数化し、専用サーバーではプロセス起動時に環境変数を設定する。いずれの場合もアプリケーション実行時には通常の環境変数として利用可能である。

環境変数 + .env ファイル方式は、12-factor app 原則に準拠した最もシンプルな設定管理手法である。実装には os.getenv による環境変数取得のみで十分であり、python-dotenv のような .env ファイル読み込みライブラリは本番環境では不要となる。ローカル開発環境でも .env ファイルから手動で環境変数を設定することで対応でき、追加ライブラリの管理負荷を避けながら機密情報の環境分離を自然に実現できる。

## 結論

### 判断結果

<!-- GLOBAL_CONCLUSION_BEGIN: config-management-tech-selection -->

バックエンドの設定管理技術として環境変数方式を採用し、os.getenv による実装を行う。

<!-- GLOBAL_CONCLUSION_END: config-management-tech-selection -->

### 論理的根拠

一人開発制約下でのシンプルさが決定要因である。12-factor app 原則に準拠した業界標準手法であり、機密情報の環境分離を確実に実現する。os.getenv による標準ライブラリのみの実装は、本番環境のデプロイ方法に関係なく一貫して動作し、追加ライブラリの管理負荷を避けながら、理解しやすく保守負荷の軽い設定管理を実現する。
