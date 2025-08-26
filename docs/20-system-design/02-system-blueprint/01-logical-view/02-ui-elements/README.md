---
doc_type: "pattern-parallel"
status: "draft"
depends:
  contracts:
    - "user-desires"
  produces: []
---

# UI 要素

## 目次

TBD

## 概要

TBD

## 処理フロー

TBD

## 直交分割

### 前提となるUI要素

01-user-desires より、次の UI 要素が導き出されました。

<!-- PREMISE_BEGIN: user-desires -->

#### テキスト入力 (5 要素)

- URL 入力フィールド - 外部記事の URL 入力 <!-- REFERENCE_INLINE: url-input-field -->
- 概念作成インターフェース - 新規概念作成と関連概念入力の統合機能 <!-- REFERENCE_INLINE: concept-creation-interface -->
- 質問テキスト入力フィールド - 質問内容を受け取るインターフェース <!-- REFERENCE_INLINE: question-text-field -->
- グループ名・説明入力機能 - 形成されたグループへの名前や説明の後付け入力 <!-- REFERENCE_INLINE: group-naming-function -->
- 関係定義・意味付け機能 - 個人的な関係語彙による直感的な関係表現と意味付け <!-- REFERENCE_INLINE: relationship-definition-function -->

#### 選択操作 (2 要素)

- 概念選択インターフェース - 用途に応じた概念の検索・選択機能 <!-- REFERENCE_INLINE: concept-selection-interface -->
- 概念候補選択インターフェース - 提示された概念候補から関連する概念を選択 <!-- REFERENCE_INLINE: concept-candidates-selection -->

#### 直接操作 (1 要素)

- 空間配置インターフェース - 概念の直感的な配置機能 <!-- REFERENCE_INLINE: spatial-arrangement-interface -->

#### 表示・視覚化 (4 要素)

- 概念カード UI - 概念を視覚的に操作可能なカード形式で表示 <!-- REFERENCE_INLINE: concept-cards-ui -->
- 文献メモ一覧表示機能 - 選択した概念に紐付いたメモ一覧表示 <!-- REFERENCE_INLINE: memo-list-display -->
- メモ詳細表示機能 - 各メモの詳細内容を参照 <!-- REFERENCE_INLINE: memo-detail-display -->
- 近接度ベースクラスタ認識・境界表示機能 - 近くに置かれた概念のクラスタ認識と視覚的境界表示 <!-- REFERENCE_INLINE: cluster-recognition-display -->

#### 自動処理 (1 要素)

- 記事取得・解析処理 - URL 指定による外部記事の取得と内容解析

#### LLM 生成・推論 (2 要素)

- LLM による観点別要約生成機能 - 選択した概念の観点で記事内容を要約
- 質問から概念候補提示機能 - 質問内容から関連しそうな概念候補を提示

#### 編集操作 (1 要素)

- 要約修正エディタ - LLM が作成した要約を個人の理解に合わせて修正 <!-- REFERENCE_INLINE: summary-editor -->

#### データ保存 (2 要素)

- 文献メモ保存機能 - 修正した内容を個人の知識体系に保存 <!-- REFERENCE_INLINE: literature-memo-save -->
- 関係性ネットワーク記録機能 - 発見した関係性の記録・保存 <!-- REFERENCE_INLINE: relationship-network-record -->

#### 機能実行 (1 要素)

- 複数概念適用機能 - 同一記事を別の概念の観点でも読み直し

<!-- PREMISE_END: user-desires -->

### シナリオ項目を含まない理由

各 UI 要素ファイルには**シナリオ項目を記載していない**。これは以下の設計判断に基づく：

1. **責務の分離**: シナリオとの対応関係は上位の`docs/20-system-design/01-user-desires/README.md`で管理済み
2. **汎用性の確保**: 各 UI 要素は複数シナリオで使用されるため、特定シナリオに依存しない汎用的な責務定義が適切
3. **情報の重複排除**: 同一情報を複数箇所で管理することによる整合性リスクを回避
4. **論理ビューの本質**: 「どのシナリオで使うか」より「何をする UI 要素か」が論理設計の核心

### UI要素のフィルタリング

上記19個のUI要素から、UI設計対象となる15個の要素を抽出する。

#### 対象外とする4つの要素

- 自動処理: UI を伴わないため対象外
  - 記事取得・解析処理 - URL 指定による外部記事の取得と内容解析
- LLM 生成・推論: UI を伴わないため対象外
  - LLM による観点別要約生成機能 - 選択した概念の観点で記事内容を要約
  - 質問から概念候補提示機能 - 質問内容から関連しそうな概念候補を提示
- 機能実行: 複数の機能があるため対象外
  - 複数概念適用機能 - 同一記事を別の概念の観点でも読み直し

## 並列関係ファイル

各 UI 要素の実装仕様を個別ファイルで管理する。

### テキスト入力ファイル (5 ファイル)

- [URL 入力フィールド](input-01-url-field.md) - 外部記事の URL 入力 <!-- REFERENCE_INLINE: url-input-field -->
- [概念作成インターフェース](input-02-concept-creation.md) - 新規概念作成と関連概念入力の統合機能 <!-- REFERENCE_INLINE: concept-creation-interface -->
- [質問テキスト入力フィールド](input-03-question-field.md) - 質問内容を受け取るインターフェース <!-- REFERENCE_INLINE: question-text-field -->
- [グループ名・説明入力機能](input-04-group-naming.md) - 形成されたグループへの名前や説明の後付け入力 <!-- REFERENCE_INLINE: group-naming-function -->
- [関係定義・意味付け機能](input-05-relationship-definition.md) - 個人的な関係語彙による直感的な関係表現と意味付け <!-- REFERENCE_INLINE: relationship-definition-function -->

### 選択操作ファイル (2 ファイル)

- [概念選択インターフェース](select-01-concept-interface.md) - 用途に応じた概念の検索・選択機能 <!-- REFERENCE_INLINE: concept-selection-interface -->
- [概念候補選択インターフェース](select-02-concept-candidates.md) - 提示された概念候補から関連する概念を選択 <!-- REFERENCE_INLINE: concept-candidates-selection -->

### 直接操作ファイル (1 ファイル)

- [空間配置インターフェース](direct-01-spatial-arrangement.md) - 概念の直感的な配置機能 <!-- REFERENCE_INLINE: spatial-arrangement-interface -->

### 表示・視覚化ファイル (4 ファイル)

- [概念カード UI](display-01-concept-cards.md) - 概念を視覚的に操作可能なカード形式で表示 <!-- REFERENCE_INLINE: concept-cards-ui -->
- [文献メモ一覧表示機能](display-02-memo-list.md) - 選択した概念に紐付いたメモ一覧表示 <!-- REFERENCE_INLINE: memo-list-display -->
- [メモ詳細表示機能](display-03-memo-detail.md) - 各メモの詳細内容を参照 <!-- REFERENCE_INLINE: memo-detail-display -->
- [近接度ベースクラスタ認識・境界表示機能](display-04-cluster-recognition.md) - 近くに置かれた概念のクラスタ認識と視覚的境界表示 <!-- REFERENCE_INLINE: cluster-recognition-display -->

### 編集操作ファイル (1 ファイル)

- [要約修正エディタ](edit-01-summary-editor.md) - LLM が作成した要約を個人の理解に合わせて修正 <!-- REFERENCE_INLINE: summary-editor -->

### データ保存ファイル (2 ファイル)

- [文献メモ保存機能](save-01-literature-memo.md) - 修正した内容を個人の知識体系に保存 <!-- REFERENCE_INLINE: literature-memo-save -->
- [関係性ネットワーク記録機能](save-02-relationship-network.md) - 発見した関係性の記録・保存 <!-- REFERENCE_INLINE: relationship-network-record -->
