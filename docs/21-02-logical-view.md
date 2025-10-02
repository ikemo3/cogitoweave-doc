# Logical View

## 1. 主要抽象化設計

<!-- REFERENCE_BEGIN: key-abstractions -->

CogitoWeave システムは**概念ベース知識体系構築システム**として抽象化される。

中核的価値は、個人の思考を「概念」という抽象化を通じて構造化し、外部情報を自分の理解に統合する仕組みの提供にある。

### 主要抽象化

#### 1. 概念 (Concept)

思考の最小単位として知識体系の中心に位置する抽象化

- 外部情報を個人の理解に変換するフィルター
- 知識統合の基点として機能する
- 他の概念との関係によって意味を形成する

#### 2. 文献メモ (Literature Memo)

概念に紐づく具体的知識内容を表現する抽象化

- 外部文献を個人の観点で解釈した結果
- LLM 協働により効率的に作成され個人の理解で修正される
- 概念の理解を深化させる具体例や体験を蓄積する

#### 3. 関係性 (Relationship)

概念間の関連性を表現し知識ネットワークを形成する抽象化

- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援
- 断片的知識を構造化された思考地図に変換する

### 抽象化の相互作用

- **外部文献** → **概念フィルター** → **文献メモ** (情報の個人化プロセス)
- **概念** ↔ **関係性** ↔ **概念** (知識ネットワーク形成プロセス)
- **空間配置** → **関係性発見** → **思考地図構築** (直感的構造化プロセス)

これらの抽象化により、外部情報の受動的消費から、個人の理解体系への能動的統合への転換が実現される。

<!-- REFERENCE_END: key-abstractions -->

## 2. 機能境界と責務

<!-- REFERENCE_BEGIN: core-knowledge-boundary -->

- コア知識管理境界は概念管理、文献メモ管理、関係性管理、質問探索を統合して担当する。
  - 各機能の役割を明確に分け、統合的に管理することで保守負荷を軽減するため。

<!-- REFERENCE_END: core-knowledge-boundary -->

<!-- REFERENCE_BEGIN: llm-integration-boundary -->

- LLM 統合境界は LLM API による要約生成、概念候補提示、協働プロセス制御を担当する。
  - 変わりやすい技術を分離してシンプルな設計を維持するため。

<!-- REFERENCE_END: llm-integration-boundary -->

<!-- REFERENCE_BEGIN: content-acquisition-boundary -->

- コンテンツ取得境界は外部記事取得、内容解析、品質チェックを担当する。
  - 独立した処理として分離し機能変更時の影響範囲を限定するため。

<!-- REFERENCE_END: content-acquisition-boundary -->

## 3. 画面構成設計

<!-- REFERENCE_BEGIN: functional-specifications -->

### テキスト入力 (5 要素)

- URL 入力フィールド - 外部記事の URL 入力 <!-- REFERENCE_INLINE: url-input-field -->
- 概念作成インターフェース - 新規概念作成と関連概念入力の統合機能 <!-- REFERENCE_INLINE: concept-creation-interface -->
- 質問テキスト入力フィールド - 質問内容を受け取るインターフェース <!-- REFERENCE_INLINE: question-text-field -->
- グループ名・説明入力機能 - 形成されたグループへの名前や説明の後付け入力 <!-- REFERENCE_INLINE: group-naming-function -->
- 関係定義・意味付け機能 - 個人的な関係語彙による直感的な関係表現と意味付け <!-- REFERENCE_INLINE: relationship-definition-function -->

### 選択操作 (2 要素)

- 概念選択インターフェース - 用途に応じた概念の検索・選択機能 <!-- REFERENCE_INLINE: concept-selection-interface -->
- 概念候補選択インターフェース - 提示された概念候補から関連する概念を選択 <!-- REFERENCE_INLINE: concept-candidates-selection -->

### 直接操作 (1 要素)

- 空間配置インターフェース - 概念の直感的な配置機能 <!-- REFERENCE_INLINE: spatial-arrangement-interface -->

### 表示・視覚化 (4 要素)

- 概念カード UI - 概念を視覚的に操作可能なカード形式で表示 <!-- REFERENCE_INLINE: concept-cards-ui -->
- 文献メモ一覧表示機能 - 選択した概念に紐付いたメモ一覧表示 <!-- REFERENCE_INLINE: memo-list-display -->
- メモ詳細表示機能 - 各メモの詳細内容を参照 <!-- REFERENCE_INLINE: memo-detail-display -->
- 近接度ベースクラスタ認識・境界表示機能 - 近くに置かれた概念のクラスタ認識と視覚的境界表示 <!-- REFERENCE_INLINE: cluster-recognition-display -->

### 自動処理 (1 要素)

- 記事取得・解析処理 - URL 指定による外部記事の取得と内容解析

### LLM 生成・推論 (2 要素)

- LLM による観点別要約生成機能 - 選択した概念の観点で記事内容を要約
- 質問から概念候補提示機能 - 質問内容から関連しそうな概念候補を提示

### 編集操作 (1 要素)

- 要約修正エディタ - LLM が作成した要約を個人の理解に合わせて修正 <!-- REFERENCE_INLINE: summary-editor -->

### データ保存 (2 要素)

- 文献メモ保存機能 - 修正した内容を個人の知識体系に保存 <!-- REFERENCE_INLINE: literature-memo-save -->
- 関係性ネットワーク記録機能 - 発見した関係性の記録・保存 <!-- REFERENCE_INLINE: relationship-network-record -->

### 機能実行 (1 要素)

- 複数概念適用機能 - 同一記事を別の概念の観点でも読み直し

<!-- REFERENCE_END: functional-specifications -->

## 4. 画面構成設計

### メイン画面統合

<!-- REFERENCE_BEGIN: main-screen-integration -->

メイン画面は概念管理、文献メモ管理、関係性管理を 1 画面に統合して提供する。概念・文献メモ・関係性は統合的に利用されるため一体化した画面構成が効率的である。

<!-- REFERENCE_END: main-screen-integration -->

### メイン画面構成要素

<!-- REFERENCE_BEGIN: main-screen-structure -->

- 概念選択インターフェース - 左サイドバーに配置し概念一覧と検索機能を提供
- 概念カード UI - メイン表示エリアに選択概念の詳細情報を表示
- 文献メモ一覧表示機能 - 概念カード下部に関連メモ一覧をタブ形式で配置
- メモ詳細表示機能 - メモ選択時にモーダルまたは右パネルで詳細表示
- URL 入力フィールド - 新規文献追加用にヘッダー部に配置
- 概念作成インターフェース - 概念追加ボタンからモーダル表示
- 関係定義・意味付け機能 - 概念間関係作成時にモーダル表示
- 要約修正エディタ - 文献メモ編集時に専用エディタを表示

<!-- REFERENCE_END: main-screen-structure -->

### 質問探索画面統合

<!-- REFERENCE_BEGIN: question-search-screen-integration -->

質問探索画面は質問から概念発見とメモ参照を独立画面で提供する。質問探索は他の機能と利用パターンが異なるため独立画面が適切である。

<!-- REFERENCE_END: question-search-screen-integration -->

### 質問探索画面構成要素

<!-- REFERENCE_BEGIN: question-search-screen-structure -->

- 質問テキスト入力フィールド - 画面上部にメイン入力エリアとして配置
- 質問から概念候補提示機能 - 質問入力後に候補一覧を表示
- 概念候補選択インターフェース - 候補選択のためのリスト形式 UI
- 文献メモ一覧表示機能 - 選択概念に関連するメモ一覧表示
- メモ詳細表示機能 - メモ詳細の参照表示

<!-- REFERENCE_END: question-search-screen-structure -->

### 空間配置画面統合

<!-- REFERENCE_BEGIN: spatial-arrangement-screen-integration -->

空間配置画面は概念の空間配置と関係性構築を専用画面で提供する。複雑な直接操作が必要なためフルスクリーン専用画面が適切である。

<!-- REFERENCE_END: spatial-arrangement-screen-integration -->

### 空間配置画面構成要素

<!-- REFERENCE_BEGIN: spatial-arrangement-screen-structure -->

- 空間配置インターフェース - 全画面を使用した概念配置キャンバス
- 概念カード UI - 配置可能な概念カードの表示
- 近接度ベースクラスタ認識・境界表示機能 - 自動クラスタ認識と視覚的境界表示
- グループ名・説明入力機能 - クラスタ形成時の命名インターフェース
- 関係性ネットワーク記録機能 - 定義した関係性の保存処理

<!-- REFERENCE_END: spatial-arrangement-screen-structure -->

### 画面遷移設計

<!-- REFERENCE_BEGIN: screen-transition-design -->

シンプルなタブ切り替えによる 3 画面間の移動を提供する。操作の複雑度を抑制し直感的なナビゲーションを実現する。

<!-- REFERENCE_END: screen-transition-design -->

## 5. 視覚デザイン

LOCAL_CONCLUSION のみなので省略

## 6. UI 技術方針

<!-- REFERENCE_BEGIN: ui-tech-approach -->

CogitoWeave の UI 技術方針として、Web 技術による柔軟な実装アプローチを採用する。プラットフォーム制約によりネイティブアプリ開発は除外され、マルチデバイス対応とユーザーインタラクション特性を考慮した選択肢から最適解を選択する。

### 除外される技術選択肢

- **ネイティブアプリ**: プラットフォーム制約により除外済み
- **SSG(Static Site Generation)**: 編集機能中心に矛盾し、マルチデバイス共有に適さない
- **ローカルオンリー**: マルチデバイス前提制約に違反

### 許容される技術選択肢

- **レンダリング方式**: CSR、SSR、またはハイブリッドアプローチのいずれも可能である
  - プライベート用途により SEO 要件がないため、CSR の制約がない
  - 初回ロード速度重視により SSR の利点もある
- **アプリケーション構造**: SPA、MPA、または SPA ベース+ページ分割のハイブリッドアプローチのいずれも可能である
  - 作業完結型の操作パターンにより MPA に適合する
  - 効率性重視の継続利用により SPA の利点もある
- **コンポーネント方針**: 軽量コンポーネント指向を基本とし、必要に応じてページ指向も選択可能である
  - UI 要素の再利用性と一貫性確保が必要
  - デザインシステムレベルまでは不要

### 技術方針決定の基準

具体的な技術選択は、Development View において以下の基準で決定する:

- 保守性と理解しやすさの優先
- 一人での管理可能な複雑度
- Mac 環境での情報整理機能と iPhone 環境での情報参照機能の最適化

<!-- REFERENCE_END: ui-tech-approach -->

## 7. UI コンポーネント責務

<!-- REFERENCE_BEGIN: component-granularity-design -->

CogitoWeave の UI コンポーネント責務として、階層的な粒度設計による軽量なコンポーネント指向アーキテクチャを採用する。

- **最小粒度**: ボタン・入力フィールド・ラベル等の基本 UI 要素レベル
  - 単一責務の原則により、一つの操作または表示に特化する。
- **統合粒度**: カード・パネル・モーダル等の機能単位レベル
  - 複数の基本要素を組み合わせた意味のある機能を提供する。
- **階層アプローチ**: 統合コンポーネント内で基本コンポーネントを組み合わせる
  - 外部からは統合コンポーネントとして使用し、内部実装では基本コンポーネントで構成する。

<!-- REFERENCE_END: component-granularity-design -->

<!-- REFERENCE_BEGIN: state-management-responsibilities -->

状態管理は 3 層に分離して責務を明確化する。

- **コンポーネント内部状態**: 表示状態・入力値・開閉状態等の局所的な制御
  - コンポーネント固有の UI 状態管理に限定する
- **ページレベル状態**: 選択中概念・検索フィルタ・編集中データ等の複数コンポーネント間共有
  - 状態管理ライブラリを使用してページ内グローバル状態を管理する
- **アプリケーション全体状態**: ユーザー設定・表示設定・大きなコンポーネント間同期データ
  - 永続化データはブラウザの localStorage を使用する
  - ページ境界を越えた大きなコンポーネント間の同期はクライアント側状態管理で対応する

<!-- REFERENCE_END: state-management-responsibilities -->

<!-- REFERENCE_BEGIN: data-flow-design -->

コンテナコンポーネントパターンと単方向データフローによるデータ設計を採用する。

- **コンテナコンポーネントパターン**: データ取得・API 呼び出し責務を上位コンポーネントに集約
  - プレゼンテーショナルコンポーネントは props によるデータ受け取りに特化する
- **単方向データフロー**: props による下方向データ流れと callback による上方向イベント伝播
  - データの流れを予測可能にし、デバッグとテストを容易にする
- **イベント駆動通信**: 子コンポーネントから親コンポーネントへは callback props によるイベント通知
  - 疎結合を保ちながら必要な情報のみを伝達する

<!-- REFERENCE_END: data-flow-design -->

<!-- REFERENCE_BEGIN: device-adaptation-approach -->

デバイス別対応はコンポーネント共通化とレスポンシブ設計による適応的 UI で実現する。

- **コンポーネント共通化**: 同一コンポーネントをデバイス別に CSS で最適化
  - Mac 環境と iPhone 環境で共通のコンポーネントロジックを使用する
- **レスポンシブ設計**: デバイス別の表示密度と操作方法に応じた適応的 UI
  - 情報整理機能と情報参照機能の違いを CSS レベルで調整する

<!-- REFERENCE_END: device-adaptation-approach -->

## 8. データモデル

### 1. 概念エンティティ (Concept)

<!-- REFERENCE_BEGIN: concept-entity -->

概念エンティティ(Concept):

- 思考の最小単位として知識体系の中心に位置する
- 基本属性: ID、名前、説明、作成日時、更新日時
- 他の概念との関係性によって意味を形成する
- 外部情報を個人の理解に変換する観点として機能する

<!-- REFERENCE_END: concept-entity -->

### 2. 文献メモエンティティ (LiteratureMemo)

<!-- REFERENCE_BEGIN: literature-memo-entity -->

文献メモエンティティ(LiteratureMemo):

- ツェッテルカステン方式による概念ベースの知識蓄積
- 基本属性: ID、タイトル、内容、作成日時、更新日時
- 特定の概念の観点で外部文献を要約・解釈したもの
- LLM 協働により効率的に作成され、個人の理解で修正される

<!-- REFERENCE_END: literature-memo-entity -->

### 3. 関係性エンティティ (Relationship)

<!-- REFERENCE_BEGIN: relationship-entity -->

関係性エンティティ(Relationship):

- 概念間の無向グラフ構造による知識ネットワーク形成
- 基本属性: ID、関係名、説明、作成日時
- 個人的な関係語彙による柔軟な関係定義
- 空間配置による直感的な関係性発見を支援

<!-- REFERENCE_END: relationship-entity -->

### 4. 文献エンティティ (Literature)

<!-- REFERENCE_BEGIN: literature-entity -->

文献エンティティ(Literature):

- 外部文献の書誌情報管理
- 基本属性: ID、URL、タイトル、取得日時
- 複数の概念観点から文献メモが生成される元情報

<!-- REFERENCE_END: literature-entity -->

## 9. データベース方針

<!-- REFERENCE_BEGIN: database-approach -->

CogitoWeave では RDBMS を採用する。リレーショナル構造によるデータ整合性管理、成熟技術による保守性確保、データ規模への十分な対応能力が主な選択理由である。具体的な製品選択は Development View で決定する。

<!-- REFERENCE_END: database-approach -->

## 10. ドメインモデル

<!-- REFERENCE_BEGIN: domain-model -->

CogitoWeave のドメインモデルは薄いドメイン層アプローチを採用する。4 つのエンティティ（概念、文献メモ、関係性、文献）に対して基本的な CRUD 操作と最小限のビジネス制約のみを実装し、複雑なドメインロジックはアプリケーション層に委ねる。データベース制約による整合性管理を活用し、外部 API 処理は分離する。

<!-- REFERENCE_END: domain-model -->

## 11. バッチ仕様

<!-- REFERENCE_BEGIN: batch-specification -->

CogitoWeave のバッチ仕様は日次データバックアップのみを実装する。全データの完全バックアップを日次実行し、システム障害時の迅速復旧を支援する。自動分析・推奨機能、データクリーンアップ、統計処理等は設計方針により除外する。

<!-- REFERENCE_END: batch-specification -->

## 12. バックエンドアーキテクチャスタイル

<!-- REFERENCE_BEGIN: backend-architecture-style -->

CogitoWeave のバックエンドアーキテクチャスタイルとして**レイヤードアーキテクチャ**を採用する。階層的な責務分離と機能境界に基づく水平分割により、一人開発における保守性と理解しやすさを確保する。

<!-- REFERENCE_END: backend-architecture-style -->

## 13. フロントエンドレンダリングスタイル

<!-- REFERENCE_BEGIN: frontend-rendering-style -->

CogitoWeave のフロントエンドレンダリングスタイルとして**CSR(Client-Side Rendering)**を採用する。3 画面間の状態共有効率と軽量なコンポーネント指向の特性を活かし、一人での開発における保守性と理解しやすさを確保する。

<!-- REFERENCE_END: frontend-rendering-style -->

## 14. API 技術方針

<!-- REFERENCE_BEGIN: api-tech-approach -->

API 技術方針として REST API を採用する。

<!-- REFERENCE_END: api-tech-approach -->

<!-- REFERENCE_BEGIN: performance-budget-allocation -->

**システム初回起動パフォーマンスバジェット**を以下のように配分する。

- HTML ロード: 300ms 以内
- JavaScript バンドルロード: 800ms 以内
- DOM 構築: 200ms 以内
- 初回 API 呼び出し: 600ms 以内
- レンダリング完了: 300ms 以内
- 合計バジェット: 2200ms（要件 3000ms に対して 800ms の余裕）

**概念ページ読み込みパフォーマンスバジェット**を以下のように配分する。

- JavaScript バンドル: 0ms（ブラウザキャッシュ活用）
- API 呼び出し: 600ms 以内
- レンダリング: 300ms 以内
- 合計バジェット: 900ms（要件 1500ms に対して 600ms の余裕）

<!-- REFERENCE_END: performance-budget-allocation -->

## 15. API 契約

<!-- REFERENCE_BEGIN: api-contract -->

CogitoWeave の API 契約は以下の 9 つの論理的 API 群で構成される:

### エンティティ操作 API

1. **概念 API**: 作成・読み取り・更新 (削除は優先度低)
2. **文献メモ API**: 作成・読み取り・更新・概念別一覧取得
3. **関係性 API**: 作成・読み取り・更新 (概念間の重複関係許可)
4. **文献 API**: 作成・読み取り (URL・タイトル・引用が必須)

### 検索・発見 API

1. **概念検索 API**: 名前・説明での部分一致検索
2. **質問から概念候補提示 API**: 既存個人概念体系からの検索・発見支援

### 協働・処理 API

1. **LLM 要約生成 API**: 記事内容と概念 ID から観点別要約を生成
2. **記事取得 API**: URL からタイトル・内容を取得
3. **バックアップ API**: Webhook 化による自動実行支援

これらの API により、概念中心の知識体系構築と効率的な知識活用が実現される。

<!-- REFERENCE_END: api-contract -->
