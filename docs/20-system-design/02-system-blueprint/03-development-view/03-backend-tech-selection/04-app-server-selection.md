---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "technology-selection-criteria"
  produces:
    - "app-server-selection"
---

# アプリケーションサーバー選択

## 概要

Python + FastAPI を基盤として、アプリケーションサーバーを選択する段階。FastAPI アプリケーションを本番環境で実行するための ASGI サーバーを決定する。

### 前提

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 選択肢と判断基準

FastAPI の ASGI 仕様に対応したサーバーから、設定と運用のシンプルさを重視して選択する。

### 判断基準

- シンプルさ (優先度高): 設定ファイル不要、コマンド一発で起動可能
- FastAPI 統合: 公式推奨度と実際の統合のしやすさ
- パフォーマンス: リクエスト処理能力とメモリ効率

### 選択肢

- uvicorn: FastAPI 開発元推奨の標準 ASGI サーバー
- Hypercorn: HTTP/2 対応の高機能 ASGI サーバー
- Daphne: Django Channels 開発元による ASGI サーバー

## 各選択肢の詳細分析

各 ASGI サーバーの基本的な特徴と、FastAPI アプリケーション実行における適性を分析する。

### uvicorn

FastAPI の開発元が推奨する標準的な ASGI サーバー。シンプルな設計でデバッグ機能も充実している。

- 利点
  - コマンド一発で起動: `uvicorn main:app --reload`
  - 設定ファイル不要で即座に動作開始
  - FastAPI 公式ドキュメントで標準的に使用
  - ホットリロード機能による開発効率向上
- 欠点
  - HTTP/2 未対応
  - 高度な負荷分散機能は別途必要

### Hypercorn

HTTP/2 と HTTP/3 に対応した高機能な ASGI サーバー。企業向けの機能が充実している。

- 利点
  - HTTP/2 および HTTP/3 対応
  - TOML 設定ファイルによる詳細設定
  - プロセス管理機能内蔵
- 欠点
  - 設定ファイルの作成が推奨される
  - 機能が多く設定項目が複雑
  - FastAPI との統合例が uvicorn より少ない

### Daphne

Django Channels の開発元による ASGI サーバー。WebSocket 処理に特化した設計。

- 利点
  - WebSocket 処理が高度
  - Django エコシステムとの統合
- 欠点
  - Django 向けの設計で FastAPI には過剰
  - 設定が Django 寄りで複雑
  - FastAPI 公式での言及が少ない

## 判断基準による評価

### シンプルさの比較

uvicorn が圧倒的に優秀。設定ファイル不要でコマンド一発起動が可能で、技術選択基準の「外部ライブラリは適度に制限」に最も適合する。Hypercorn は TOML 設定が推奨され、Daphne は Django 向けの複雑な設定が必要。

### FastAPI 統合の比較

uvicorn が最も優秀。FastAPI 公式ドキュメントで標準的に使用され、開発元も推奨している。Hypercorn も対応するが事例が少なく、Daphne は Django 向けで FastAPI との親和性が低い。

### パフォーマンスの比較

単純な HTTP/1.1 処理では uvicorn と Hypercorn に大差なし。HTTP/2 が必要なら Hypercorn が優秀だが、CogitoWeave の要件では HTTP/1.1 で十分。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: app-server-selection -->

アプリケーションサーバーとして uvicorn を採用する。

<!-- GLOBAL_CONCLUSION_END: app-server-selection -->

### 選択根拠

シンプルさの判断基準において uvicorn が圧倒的に優秀であり、技術選択基準「外部ライブラリは適度に制限」「監視は最小限」と完全に合致する。FastAPI 公式推奨により統合も確実で、設定ファイル不要の即座起動により開発効率も最大化される。
