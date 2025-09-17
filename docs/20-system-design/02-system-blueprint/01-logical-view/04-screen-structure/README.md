---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "functional-specifications"
    - "device-optimized-interface"
  produces:
    - "main-screen-integration"
    - "main-screen-structure"
    - "question-search-screen-integration"
    - "question-search-screen-structure"
    - "spatial-arrangement-screen-integration"
    - "spatial-arrangement-screen-structure"
    - "screen-transition-design"
---

# 画面構成設計

## 前提

機能仕様で定義された 19 個の UI 要素を基に、実際の画面レイアウトと構成を設計する。

<!-- PREMISE_BEGIN: functional-specifications -->

### テキスト入力 (5 要素)

- URL 入力フィールド - 外部記事の URL 入力
- 概念作成インターフェース - 新規概念作成と関連概念入力の統合機能
- 質問テキスト入力フィールド - 質問内容を受け取るインターフェース
- グループ名・説明入力機能 - 形成されたグループへの名前や説明の後付け入力
- 関係定義・意味付け機能 - 個人的な関係語彙による直感的な関係表現と意味付け

### 選択操作 (2 要素)

- 概念選択インターフェース - 用途に応じた概念の検索・選択機能
- 概念候補選択インターフェース - 提示された概念候補から関連する概念を選択

### 直接操作 (1 要素)

- 空間配置インターフェース - 概念の直感的な配置機能

### 表示・視覚化 (4 要素)

- 概念カード UI - 概念を視覚的に操作可能なカード形式で表示
- 文献メモ一覧表示機能 - 選択した概念に紐付いたメモ一覧表示
- メモ詳細表示機能 - 各メモの詳細内容を参照
- 近接度ベースクラスタ認識・境界表示機能 - 近くに置かれた概念のクラスタ認識と視覚的境界表示

### 自動処理 (1 要素)

- 記事取得・解析処理 - URL 指定による外部記事の取得と内容解析

### LLM 生成・推論 (2 要素)

- LLM による観点別要約生成機能 - 選択した概念の観点で記事内容を要約
- 質問から概念候補提示機能 - 質問内容から関連しそうな概念候補を提示

### 編集操作 (1 要素)

- 要約修正エディタ - LLM が作成した要約を個人の理解に合わせて修正

### データ保存 (2 要素)

- 文献メモ保存機能 - 修正した内容を個人の知識体系に保存
- 関係性ネットワーク記録機能 - 発見した関係性の記録・保存

### 機能実行 (1 要素)

- 複数概念適用機能 - 同一記事を別の概念の観点でも読み直し

<!-- PREMISE_END: functional-specifications -->

### デバイス最適化インターフェース

<!-- PREMISE_BEGIN: device-optimized-interface -->

- **Mac 環境**: マルチウィンドウ・複雑レイアウト活用による情報整理専用 UI
- **iPhone 環境**: シンプル・タッチ最適化による情報参照専用 UI
- **レスポンシブ設計**: デバイス切り替え時の文脈継続性確保

<!-- PREMISE_END: device-optimized-interface -->

## 論理

画面構成設計では利用パターンと操作複雑度に基づく画面分割を行う。概念管理・文献メモ管理・関係性管理は統合的に利用されるため 1 画面に統合する。質問探索は利用パターンが異なるため独立画面とする。空間配置は操作複雑度が高いためフルスクリーン専用画面とする。Mac 環境では複雑レイアウトを活用し、iPhone 環境はシンプルな参照専用 UI とする。

## 結論

CogitoWeave システムは 3 つの主要画面で構成する。

### メイン画面統合

<!-- GLOBAL_CONCLUSION_BEGIN: main-screen-integration -->

メイン画面は概念管理、文献メモ管理、関係性管理を 1 画面に統合して提供する。概念・文献メモ・関係性は統合的に利用されるため一体化した画面構成が効率的である。

<!-- GLOBAL_CONCLUSION_END: main-screen-integration -->

### メイン画面構成要素

<!-- GLOBAL_CONCLUSION_BEGIN: main-screen-structure -->

- 概念選択インターフェース - 左サイドバーに配置し概念一覧と検索機能を提供
- 概念カード UI - メイン表示エリアに選択概念の詳細情報を表示
- 文献メモ一覧表示機能 - 概念カード下部に関連メモ一覧をタブ形式で配置
- メモ詳細表示機能 - メモ選択時にモーダルまたは右パネルで詳細表示
- URL 入力フィールド - 新規文献追加用にヘッダー部に配置
- 概念作成インターフェース - 概念追加ボタンからモーダル表示
- 関係定義・意味付け機能 - 概念間関係作成時にモーダル表示
- 要約修正エディタ - 文献メモ編集時に専用エディタを表示

<!-- GLOBAL_CONCLUSION_END: main-screen-structure -->

### 質問探索画面統合

<!-- GLOBAL_CONCLUSION_BEGIN: question-search-screen-integration -->

質問探索画面は質問から概念発見とメモ参照を独立画面で提供する。質問探索は他の機能と利用パターンが異なるため独立画面が適切である。

<!-- GLOBAL_CONCLUSION_END: question-search-screen-integration -->

### 質問探索画面構成要素

<!-- GLOBAL_CONCLUSION_BEGIN: question-search-screen-structure -->

- 質問テキスト入力フィールド - 画面上部にメイン入力エリアとして配置
- 質問から概念候補提示機能 - 質問入力後に候補一覧を表示
- 概念候補選択インターフェース - 候補選択のためのリスト形式 UI
- 文献メモ一覧表示機能 - 選択概念に関連するメモ一覧表示
- メモ詳細表示機能 - メモ詳細の参照表示

<!-- GLOBAL_CONCLUSION_END: question-search-screen-structure -->

### 空間配置画面統合

<!-- GLOBAL_CONCLUSION_BEGIN: spatial-arrangement-screen-integration -->

空間配置画面は概念の空間配置と関係性構築を専用画面で提供する。複雑な直接操作が必要なためフルスクリーン専用画面が適切である。

<!-- GLOBAL_CONCLUSION_END: spatial-arrangement-screen-integration -->

### 空間配置画面構成要素

<!-- GLOBAL_CONCLUSION_BEGIN: spatial-arrangement-screen-structure -->

- 空間配置インターフェース - 全画面を使用した概念配置キャンバス
- 概念カード UI - 配置可能な概念カードの表示
- 近接度ベースクラスタ認識・境界表示機能 - 自動クラスタ認識と視覚的境界表示
- グループ名・説明入力機能 - クラスタ形成時の命名インターフェース
- 関係性ネットワーク記録機能 - 定義した関係性の保存処理

<!-- GLOBAL_CONCLUSION_END: spatial-arrangement-screen-structure -->

### 画面遷移設計

<!-- GLOBAL_CONCLUSION_BEGIN: screen-transition-design -->

シンプルなタブ切り替えによる 3 画面間の移動を提供する。操作の複雑度を抑制し直感的なナビゲーションを実現する。

<!-- GLOBAL_CONCLUSION_END: screen-transition-design -->
