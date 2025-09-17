---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "main-screen-integration"
    - "main-screen-structure"
    - "question-search-screen-integration"
    - "question-search-screen-structure"
    - "spatial-arrangement-screen-structure"
    - "screen-transition-design"
    - "functional-specifications"
    - "efficiency-focused-operation"
    - "device-optimized-interface"
    - "personal-optimization-flexibility"
  produces:
    - "concept-visual-priority"
    - "information-type-visual-differentiation"
    - "importance-visual-weighting"
    - "visual-grouping-strategy"
    - "progressive-disclosure"
    - "visual-consistency"
    - "mac-visual-density-optimization"
    - "ipad-visual-balance"
    - "iphone-visual-simplification"
    - "operability-visual-indication"
    - "operation-progress-display"
    - "operation-result-feedback"
---

# 視覚デザイン

## 前提

### 画面構成設計からの前提

#### メイン画面統合

<!-- PREMISE_BEGIN: main-screen-integration -->

メイン画面は概念管理、文献メモ管理、関係性管理を 1 画面に統合して提供する。概念・文献メモ・関係性は統合的に利用されるため一体化した画面構成が効率的である。

<!-- PREMISE_END: main-screen-integration -->

#### メイン画面構成要素

<!-- PREMISE_BEGIN: main-screen-structure -->

- 概念選択インターフェース - 左サイドバーに配置し概念一覧と検索機能を提供
- 概念カード UI - メイン表示エリアに選択概念の詳細情報を表示
- 文献メモ一覧表示機能 - 概念カード下部に関連メモ一覧をタブ形式で配置
- メモ詳細表示機能 - メモ選択時にモーダルまたは右パネルで詳細表示
- URL 入力フィールド - 新規文献追加用にヘッダー部に配置
- 概念作成インターフェース - 概念追加ボタンからモーダル表示
- 関係定義・意味付け機能 - 概念間関係作成時にモーダル表示
- 要約修正エディタ - 文献メモ編集時に専用エディタを表示

<!-- PREMISE_END: main-screen-structure -->

#### 質問探索画面統合

<!-- PREMISE_BEGIN: question-search-screen-integration -->

質問探索画面は質問から概念発見とメモ参照を独立画面で提供する。質問探索は他の機能と利用パターンが異なるため独立画面が適切である。

<!-- PREMISE_END: question-search-screen-integration -->

#### 質問探索画面構成要素

<!-- PREMISE_BEGIN: question-search-screen-structure -->

- 質問テキスト入力フィールド - 画面上部にメイン入力エリアとして配置
- 質問から概念候補提示機能 - 質問入力後に候補一覧を表示
- 概念候補選択インターフェース - 候補選択のためのリスト形式 UI
- 文献メモ一覧表示機能 - 選択概念に関連するメモ一覧表示
- メモ詳細表示機能 - メモ詳細の参照表示

<!-- PREMISE_END: question-search-screen-structure -->

#### 空間配置画面統合

<!-- REFERENCE_BEGIN: spatial-arrangement-screen-integration -->

空間配置画面は概念の空間配置と関係性構築を専用画面で提供する。複雑な直接操作が必要なためフルスクリーン専用画面が適切である。

<!-- REFERENCE_END: spatial-arrangement-screen-integration -->

#### 空間配置画面構成要素

<!-- PREMISE_BEGIN: spatial-arrangement-screen-structure -->

- 空間配置インターフェース - 全画面を使用した概念配置キャンバス
- 概念カード UI - 配置可能な概念カードの表示
- 近接度ベースクラスタ認識・境界表示機能 - 自動クラスタ認識と視覚的境界表示
- グループ名・説明入力機能 - クラスタ形成時の命名インターフェース
- 関係性ネットワーク記録機能 - 定義した関係性の保存処理

<!-- PREMISE_END: spatial-arrangement-screen-structure -->

#### 画面遷移設計

<!-- PREMISE_BEGIN: screen-transition-design -->

シンプルなタブ切り替えによる 3 画面間の移動を提供する。操作の複雑度を抑制し直感的なナビゲーションを実現する。

<!-- PREMISE_END: screen-transition-design -->

### 画面仕様からの前提

<!-- PREMISE_BEGIN: functional-specifications -->

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

<!-- PREMISE_END: functional-specifications -->

### 使用性制約からの前提

#### 効率性重視の操作設計

<!-- PREMISE_BEGIN: efficiency-focused-operation -->

- **マウス操作中心**: 文字入力以外はマウス操作を基本とした直感的インターフェース
- **GUI 重視**: 複雑な操作もクリック・ドラッグ等の視覚的操作で完結
- **操作学習前提**: 初回直感性より継続利用での効率性を重視

<!-- PREMISE_END: efficiency-focused-operation -->

#### デバイス最適化インターフェース

<!-- PREMISE_BEGIN: device-optimized-interface -->

- **Mac 環境**: マルチウィンドウ・複雑レイアウト活用による情報整理専用 UI
- **iPhone 環境**: シンプル・タッチ最適化による情報参照専用 UI
- **レスポンシブ設計**: デバイス切り替え時の文脈継続性確保

<!-- PREMISE_END: device-optimized-interface -->

#### 個人最適化による柔軟性

<!-- PREMISE_BEGIN: personal-optimization-flexibility -->

- **決め打ち設計**: 作成者の操作パターンに最初から最適化した固定インターフェース
- **標準的ガイダンス排除**: 一般的なヘルプ・チュートリアル等は実装しない
- **シンプルな操作体系**: 複雑な設定や履歴機能なしの直接的操作

<!-- PREMISE_END: personal-optimization-flexibility -->

## 論理

### 概念情報を視覚的最上位に配置する理由

概念・文献メモ・関係性という異なる情報タイプが統合画面で混在表示されるため、ユーザーが情報タイプを誤認識し操作対象を間違える可能性が高い。情報タイプの誤認識は操作ミスと認知負荷増加を引き起こすため、視覚的差別化による即座の識別支援が不可欠である。

システムの機能設計において、概念は知識体系の中心的存在であり、文献メモは概念の観点で作成され、関係性は概念間を結ぶものとして定義されている。この機能的な中心性をユーザーが直感的に理解するためには、概念を視覚的階層の最上位に配置し、他の情報を段階的に表示することで情報の依存関係を視覚的に表現する必要がある。概念が視覚的に最も目立つ位置にあることで、ユーザーは「まず概念を選択し、その後で関連する文献メモや関係性を操作する」という正しい操作フローを自然に学習できる。

### 情報タイプごとの異なる視覚的特徴が必要な理由

統合画面において概念・文献メモ・関係性が同時表示されるため、ユーザーがどの要素がどの情報タイプかを瞬時に判別できない場合、操作対象を間違えて非効率な操作を行う可能性が高い。特に思考支援システムでは集中した思考状態を維持することが重要であるため、情報タイプの判別に認知リソースを消費することは思考効率の大幅な低下を招く。

異なる情報タイプは操作方法も異なるため、視覚的特徴による事前の識別がユーザーの操作準備を支援する。概念は選択・作成・配置操作が中心、文献メモは参照・編集操作が中心、関係性は定義・意味付け操作が中心であるため、それぞれに適した視覚的アフォーダンスを提供することで操作効率が向上する。

### 重要度に応じた視覚的重み付けが必要な理由

知識体系操作において、現在の作業文脈に関連する情報と関連しない情報が同時に表示されるため、ユーザーの注意が重要でない情報に分散し、作業効率が低下する可能性がある。思考支援システムでは認知リソースの効率的配分が思考の質に直接影響するため、現在アクティブな概念、選択状態にある要素、関連度の高い情報を視覚的に強調することで、ユーザーの認知リソースを適切な対象に集中させる必要がある。

視覚的重み付けにより、ユーザーは重要な情報を優先的に処理し、重要でない情報は意識の周辺に置くことができるため、思考の流れを妨げることなく必要な情報にアクセスできる。

### 視覚的グルーピング戦略が必要な理由

思考支援システムでは複雑な知識体系を扱うため、多数の情報要素が画面に表示される。これらの要素がランダムに配置されている場合、ユーザーは要素間の関係性を理解するために多大な認知的努力を要し、思考の本質的な作業に集中できなくなる。

関連する機能や情報を視覚的にグループ化することで、ユーザーの認知的負荷を論理的な単位に分割できる。人間の認知特性上、関連する要素がまとまって提示されると理解しやすくなるため、概念とその関連文献メモ、関連する機能セット、同じ操作フローの要素などを視覚的に近接させることで理解効率が向上する。

### プログレッシブディスクロージャーが必要な理由

知識体系操作では多層的な情報構造を扱うため、すべての情報を同時に表示すると情報過多により認知的混乱が発生する。思考支援システムとして、不要な情報による認知的干渉を最小化し、ユーザーが現在の思考段階に必要な情報のみに集中できる環境を提供する必要がある。

段階的な情報開示により、ユーザーは自分の思考の進行に合わせて必要な詳細レベルを選択できる。概要から詳細へのドリルダウン、関連情報の段階的表示、操作フローに応じた情報の順次提示により、認知的負荷を適切にコントロールしながら深い思考作業を支援できる。

### 視覚的一貫性が学習負荷軽減に必要な理由

継続利用での効率性を重視する設計制約下では、ユーザーが視覚的パターンを学習し、予測可能な操作体験を構築することが重要である。同じ機能や状態が異なる視覚表現で表示される場合、ユーザーは毎回視覚的手がかりを再解釈する必要があり、学習効果が蓄積されず操作効率が向上しない。

一貫した視覚表現により、ユーザーは「この見た目ならこの機能」「この状態表示ならこの操作が可能」という視覚的語彙を構築できる。この語彙が確立されると、認知的処理が自動化され、思考の本質的作業により多くの認知リソースを配分できるようになる。

### Mac 環境で高情報密度レイアウトが最適な理由

Mac 環境は情報整理作業を主目的とし、大画面でマウス操作による精密な作業が可能である。情報整理では多数の概念、文献メモ、関係性を同時に参照・比較・操作する必要があるため、画面領域を最大限活用した高密度な情報配置が作業効率向上に直結する。

大画面では複雑な視覚階層も認識可能であるため、多層的な情報構造を一覧性を保ちながら表示できる。マウスによる精密な操作が可能なため、密度の高いレイアウトでも誤操作のリスクが低く、効率的な並行作業を支援できる。

### iPad 環境で中間バランスが最適な理由

iPad 環境は Mac と iPhone の中間的な画面サイズを持ち、タッチ操作とマウス操作の両方に対応する必要がある。情報整理と情報参照の両方の用途で利用される可能性があるため、高密度すぎると小さなタッチターゲットで操作性が悪化し、低密度すぎると情報整理作業での効率が低下する。

タッチ操作に必要な最小限のタッチターゲットサイズを確保しつつ、画面サイズを活用した適度な情報密度を実現することで、柔軟な利用シーンに対応する。画面の向きや利用状況に応じて情報密度を動的に調整できる中間的な設計が最も適している。

### iPhone 環境で視覚単純化が最適な理由

iPhone 環境は情報参照を主目的とし、小画面でタッチ操作による簡単な操作のみが現実的である。小画面では複雑な視覚階層や高密度レイアウトが認識困難になり、タッチ操作では精密な操作が困難であるため、シンプルな視覚構成と大きなタッチターゲットが操作性確保に不可欠である。

モバイル環境では集中した長時間作業よりも、移動中や隙間時間での情報確認が主要な利用パターンであるため、単一タスクに集中した設計が使いやすさを最大化する。画面領域の制約下では情報の優先度を明確にし、最も重要な情報のみを表示することで認知負荷を最小限に抑える必要がある。

### 操作可能性の視覚的明示が必要な理由

マウス操作中心の設計において、ユーザーは視覚的手がかりによってドラッグ可能、クリック可能、選択可能な要素を事前に識別し、適切な操作を準備する。操作可能性が視覚的に不明確な場合、ユーザーは試行錯誤による操作確認を行う必要があり、操作効率が大幅に低下する。

思考支援システムでは思考の流れを維持することが重要であるため、操作の可否を確認するための中断は思考効率の低下を招く。操作対象を瞬時に認識できる明確な視覚的アフォーダンスにより、流れるような操作体験を実現し、思考作業への集中を維持できる。

### 操作進行状態の表示が必要な理由

複雑な直接操作では、ドラッグ中、選択中、処理中など複数の操作状態が存在し、それぞれの状態で可能な操作や期待される結果が異なる。操作状態が視覚的に不明確な場合、ユーザーは現在何を行っているか、次に何をすべきかを判断できず、操作ミスや混乱が発生する。

明確な視覚変化による状態表示により、ユーザーは操作の進行状況を把握し、適切なタイミングで次の操作を実行できる。特に空間配置のようなドラッグ操作では、ドラッグ開始、移動中、配置可能領域、配置完了などの状態を視覚的に区別することで操作精度と効率が向上する。

### 操作結果の即座フィードバックが必要な理由

継続的な操作フローでは、各操作の結果を確認してから次の操作を決定するため、操作完了、エラー、成功状態の即座な視覚的伝達が操作リズムの維持に重要である。フィードバックが遅延したり不明確な場合、ユーザーは操作が成功したかを確認するために作業を中断し、操作効率と思考の継続性が損なわれる。

即座の視覚的フィードバックにより、ユーザーは操作結果を瞬時に把握し、成功の場合は次の操作に移行、エラーの場合は修正操作を実行できる。これにより思考の流れを維持しながら効率的な知識体系操作を実現できる。

## 結論

### 情報階層の視覚表現方針

<!-- GLOBAL_CONCLUSION_BEGIN: concept-visual-priority -->

概念を最上位の視覚階層に配置し、関連する文献メモと関係性情報を段階的に表示する。概念が知識体系の中心であり、他の情報はすべて概念を起点として意味を持つため。

<!-- GLOBAL_CONCLUSION_END: concept-visual-priority -->

<!-- GLOBAL_CONCLUSION_BEGIN: information-type-visual-differentiation -->

概念・文献メモ・関係性をそれぞれ異なる視覚的特徴で識別可能にする。統合画面で複数の情報タイプを効率的に操作するため視覚的区別が不可欠であるため。

<!-- GLOBAL_CONCLUSION_END: information-type-visual-differentiation -->

<!-- GLOBAL_CONCLUSION_BEGIN: importance-visual-weighting -->

アクティブな概念・選択状態・関連度の高い情報を視覚的に強調する。認知リソースを重要な情報に集中させ、思考効率を向上させるため。

<!-- GLOBAL_CONCLUSION_END: importance-visual-weighting -->

### 認知負荷軽減のための視覚戦略

<!-- GLOBAL_CONCLUSION_BEGIN: visual-grouping-strategy -->

関連する機能・情報を視覚的にグループ化し、認知的分割を支援する。複雑な知識体系操作において認知的負荷を分散し、理解しやすさを向上させるため。

<!-- GLOBAL_CONCLUSION_END: visual-grouping-strategy -->

<!-- GLOBAL_CONCLUSION_BEGIN: progressive-disclosure -->

必要な情報のみを段階的に表示し、情報過多を防止する。思考支援システムとして、不要な情報による認知的干渉を最小化するため。

<!-- GLOBAL_CONCLUSION_END: progressive-disclosure -->

<!-- GLOBAL_CONCLUSION_BEGIN: visual-consistency -->

同じ機能・状態は常に同じ視覚表現を使用し、学習負荷を軽減する。継続利用での効率性を重視し、予測可能な視覚体験を提供するため。

<!-- GLOBAL_CONCLUSION_END: visual-consistency -->

### デバイス別視覚適応方針

<!-- GLOBAL_CONCLUSION_BEGIN: mac-visual-density-optimization -->

Mac 環境では大画面を活用した高情報密度レイアウトと複雑な視覚階層を提供する。情報整理作業に必要な多数の要素を効率的に配置し、並行作業を支援するため。

<!-- GLOBAL_CONCLUSION_END: mac-visual-density-optimization -->

<!-- GLOBAL_CONCLUSION_BEGIN: ipad-visual-balance -->

iPad 環境では Mac と iPhone の中間的な情報密度と操作性を提供する。タッチ操作と画面サイズのバランスを取り、柔軟な利用シーンに対応するため。

<!-- GLOBAL_CONCLUSION_END: ipad-visual-balance -->

<!-- GLOBAL_CONCLUSION_BEGIN: iphone-visual-simplification -->

iPhone 環境では単一タスクに集中したシンプルな視覚構成と大きなタッチターゲットを提供する。情報参照用途に特化し、モバイル環境での使いやすさを優先するため。

<!-- GLOBAL_CONCLUSION_END: iphone-visual-simplification -->

### インタラクション状態の視覚フィードバック方針

<!-- GLOBAL_CONCLUSION_BEGIN: operability-visual-indication -->

ドラッグ可能・クリック可能・選択可能な要素を視覚的に識別可能にする。マウス操作中心の設計において、操作対象を瞬時に認識できることが効率性に直結するため。

<!-- GLOBAL_CONCLUSION_END: operability-visual-indication -->

<!-- GLOBAL_CONCLUSION_BEGIN: operation-progress-display -->

ドラッグ中・選択中・処理中等の状態を明確な視覚変化で表現する。複雑な直接操作における操作者の認知負荷を軽減し、操作ミスを防止するため。

<!-- GLOBAL_CONCLUSION_END: operation-progress-display -->

<!-- GLOBAL_CONCLUSION_BEGIN: operation-result-feedback -->

操作完了・エラー・成功状態を即座に視覚的に伝達する。継続的な操作フローにおいて、次の行動を迅速に判断できるようにするため。

<!-- GLOBAL_CONCLUSION_END: operation-result-feedback -->
