---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "code-quality-requirements"
    - "technology-selection-criteria"
  produces:
    - "http-client-tech-selection"
---

# HTTP クライアント選択

## 概要

Python + FastAPI エコシステムにおける外部 API 通信ライブラリの選択。LLM API 呼び出し、Web スクレイピング対象の外部通信を担当し、型安全性とテスト容易性を重視した最適解を決定する。

### 前提

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- PREMISE_END: code-quality-requirements -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

## 選択肢と判断基準

外部 API 通信ライブラリとして、LLM API 呼び出しと Web スクレイピングでの利用を前提とした HTTP クライアントを選択する。型安全性とテスト容易性を重視し、一人開発制約下での保守性を最優先とする。

### 対象通信

- **LLM API 呼び出し**: OpenAI API 等の外部 LLM サービスとの通信 (ただし専用ライブラリ経由)
- **Web スクレイピング**: 外部サイトのコンテンツ取得と HTML 解析
- **内部サービス間通信**: 将来的にプロセス分離した場合の内部 API 通信 (未確定)

### 技術制約

- **同期処理前提**: 非同期処理は必須ではない
- **HTTPS 対応**: 外部 API・サイト通信での SSL/TLS 対応は必須
- **パフォーマンス**: 個人用途のため同時接続数・スループットは重視しない
- **タイムアウト制御**: LLM API の 10 秒制約に対応した制御が必要

### 判断基準

- 型安全性 (優先度最高) - 型ヒント対応と IDE サポートによる開発効率向上
- テスト容易性 (優先度最高) - モック化・テスト実行の簡潔性
- Python 標準性 - Python エコシステムでの採用実績と情報量
- FastAPI 親和性 - FastAPI との統合の自然さ
- 保守性 - 一人開発における理解しやすさと管理負荷
- 学習コスト - 既存スキル活用と新規学習負荷のバランス

### 選択肢

- httpx: 現代的な HTTP クライアント (同期・非同期両対応)
- requests: Python での HTTP 通信デファクトスタンダード
- aiohttp: 非同期専用 HTTP クライアント
- urllib/urllib3: Python 標準ライブラリ

## 各選択肢の詳細分析

Python + FastAPI 環境での HTTP 通信ライブラリを型安全性とテスト容易性の観点で詳細評価する。

### httpx

現代的な HTTP クライアントライブラリで、requests 互換の API と型安全性を両立。

- 利点
  - requests 互換 API により学習コストが最小
  - 最初から型ヒント込みで設計されており型安全性が優秀
  - 同期・非同期両対応で将来的な拡張性を確保
  - HTTP/2 サポートによる現代的な通信プロトコル対応
  - FastAPI コミュニティでの推奨度が高い
  - mypy・pyright での型チェックが完璧に動作
- 欠点
  - requests より新しく情報量がやや少ない
  - 学習リソースが requests より限定的
  - エコシステムが成長中で一部機能が未成熟

テスト対応:

- httpx-mock や respx による効率的モック化
- pytest との統合が自然
- 同期・非同期両対応のテストパターン

### requests

Python における HTTP 通信のデファクトスタンダードライブラリ。

- 利点
  - Python HTTP 通信の圧倒的デファクトスタンダード
  - 学習リソース・コミュニティサポートが最も豊富
  - 圧倒的な採用実績による安定性
  - シンプルで直感的な API 設計
  - 豊富な拡張ライブラリとエコシステム
- 欠点
  - 型ヒントが本体に含まれず types-requests が必要
  - types-requests の型定義が不完全で型チェックエラーが発生
  - 同期処理のみで非同期拡張性がない
  - 古いアーキテクチャによる設計制約

テスト対応:

- requests-mock による成熟したモック化機能
- pytest との標準的統合パターン
- 豊富なテスト事例とベストプラクティス

### aiohttp

非同期処理に特化した HTTP クライアント・サーバーライブラリ。

- 利点
  - 非同期処理に特化した高性能
  - FastAPI と同じ非同期エコシステム
  - 型ヒント対応が比較的良好
  - WebSocket・サーバー機能も統合提供
- 欠点
  - 同期処理ができず用途が限定的
  - API が独特で requests との互換性なし
  - 学習コストが高く一人開発には過剰
  - Web スクレイピング用途には機能過多

テスト対応:

- aioresponses による非同期モック化
- pytest-asyncio との組み合わせが必要
- テスト記述が複雑化する傾向

### urllib/urllib3

Python 標準ライブラリの HTTP 通信機能。

- 利点
  - Python 標準ライブラリで外部依存なし
  - 最も軽量で依存関係の制約がない
  - 長期的な互換性が保証される
  - 低レベル制御が可能
- 欠点
  - 低レベル API で使いにくく開発効率が劣る
  - 型ヒントサポートが限定的
  - セッション管理・Cookie 処理が手動実装
  - モック化・テストが複雑

テスト対応:

- unittest.mock による手動モック実装が必要
- 低レベル API のため多くのモック設定が必要

## 判断基準による評価

### 型安全性の比較

httpx が最も優秀である。最初から型ヒント込みで設計されており、mypy・pyright での型チェックが完璧に動作する。requests は types-requests が不完全で型エラーが頻発し、aiohttp は部分的対応、urllib は限定的サポートとなる。

### テスト容易性の比較

httpx と requests がともに優秀だが、アプローチが異なる。httpx は httpx-mock による現代的なモック化、requests は requests-mock による成熟したモック化を提供する。aiohttp は非同期テストの複雑性、urllib は低レベル API の手動モック実装が必要である。

### Python 標準性の比較

requests が圧倒的に優位である。Python HTTP 通信の事実上の標準として、最も多くの学習リソース・採用実績・コミュニティサポートを持つ。httpx は急成長中、aiohttp は特定用途での実績、urllib は標準ライブラリの安定性を提供する。

### FastAPI 親和性の比較

httpx が最も親和性が高い。FastAPI 作者グループが関与し、同期・非同期両対応により FastAPI エコシステムで推奨される。requests も問題なく統合可能だが非同期対応なし、aiohttp は非同期のみ、urllib は手動統合が必要である。

### 保守性の比較

httpx と requests がともに優秀である。httpx は requests 互換 API により学習負荷を最小化し、現代的な設計で将来性がある。requests は圧倒的な情報量で問題解決が容易だが、型安全性の課題がある。

### 学習コストの比較

requests が最も低い学習コストを提供する。既存スキル活用と豊富な学習リソースにより、一人開発制約下での効率的な習得が可能である。httpx も requests 互換で学習コストは低く、aiohttp は独特 API、urllib は低レベル API により学習コストが高い。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: http-client-tech-selection -->

HTTP クライアントライブラリとして httpx を採用する。

<!-- GLOBAL_CONCLUSION_END: http-client-tech-selection -->

### 選択根拠

型安全性による開発効率向上が決定要因である。最初から型ヒント込みで設計された httpx により、mypy・pyright での完璧な型チェックと IDE サポートを活用できる。requests 互換 API により学習コストを最小化し、「各機能の役割を明確に分ける」要件を型レベルで支援する。「テストでしっかり品質を保つ」方針に対して httpx-mock による効率的なモック化を提供し、FastAPI との親和性により統合的な開発体験を実現する。将来的な非同期拡張性も確保しながら、一人開発制約下での理解しやすさと保守負荷の軽減を達成する。

### 実装方針

#### 基本利用パターン

- **同期処理**: httpx.get()・httpx.post() による直接呼び出し
- **セッション管理**: httpx.Client() による接続プール・Cookie 管理
- **エラーハンドリング**: httpx.HTTPStatusError・httpx.RequestError の統一処理
- **タイムアウト制御**: timeout パラメータによる適切な制御設定

#### テスト戦略

- **モック化**: httpx-mock による HTTP レスポンス固定
- **pytest 統合**: 標準的な pytest パターンでのテスト実行
- **型安全テスト**: mypy による静的型チェックとテストの品質保証

#### FastAPI 統合

- **依存性注入**: FastAPI の Depends() による httpx.Client の注入
- **非同期拡張**: 将来的に必要に応じて httpx.AsyncClient への移行
- **設定管理**: 環境変数によるタイムアウト・リトライ設定の外部化
