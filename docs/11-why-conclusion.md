# Why 系設計基盤 - 全結論まとめ

CogitoWeave システムの「なぜ作るのか」に関する全ての設計判断を統合し、What 系(機能設計)への前提条件として提供する。内部理想・外部制約・統合解の 3 要素から構成される設計基盤を整理する。

## 前提条件とスコープ

本文書は [10-vision-reality](10-vision-reality/README.md) の全結論を抽出・統合し、[20-system-design](20-system-design/README.md) が参照する設計基盤として機能する。

## 01-core-vision：理想 - ビジョン&スコープ

### 問題意識

<!-- REFERENCE_BEGIN: problem-statement -->

1. **思考の体系化**: 断片的な思考を価値ある知識体系に発展させる仕組みの欠如
2. **継続的な成長**: 一度作った知識を継続的に育て、深化させる仕組みの必要性
3. **意味的関係性の可視化**: 複雑な関係性を直感的に管理できるインターフェースの重要性
4. **効率性と個性の両立**: ゼロからの知識構築は困難だが、完全自動化では個人の視点が失われる

<!-- REFERENCE_END: problem-statement -->

### 設計境界

<!-- REFERENCE_BEGIN: design-boundries -->

1. **思考支援に特化**: 万能ツールではなく、考えをまとめることだけに集中
2. **人間が主導**: LLM は叩き台提供のみ、考える楽しみは人間が味わう
3. **完全に個人的**: 他人の評価ではなく、自分の問題解決が価値基準
4. **自然な関係性**: 学術的分類ではなく、個人の思考パターンを重視

<!-- REFERENCE_END: design-boundries -->

### ビジョン

<!-- REFERENCE_BEGIN: product-vision -->

「全ての考える人(小さな思想家)が自分の考えを体系化し、自らの問題解決に役立てるための仕組みを作る」

<!-- REFERENCE_END: product-vision -->

### 解決アプローチ

<!-- REFERENCE_BEGIN: solution-approach -->

- **概念中心設計**: 小さな思想家の思考の最小単位である概念を軸とした知識構築
- **段階的知識深化**: 概念に文献メモを積み重ね、永続メモとして深化させるプロセスを支援
- **視覚的・空間的アプローチ**: 複雑な関係性の直感的理解・管理による思考地図提供
- **LLM 協働**: 「LLM に作らせて、ツッコミを入れる」効率的な個性発揮の実現

<!-- REFERENCE_END: solution-approach -->

<!-- REFERENCE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- REFERENCE_END: usage-context -->

### ユーザーシナリオ

#### シナリオ 1: 気になる記事 → 自分なりの洞察

<!-- REFERENCE_BEGIN: scenario-1 -->

**CogitoWeave での解決体験**: **概念中心設計**により記事が断片的情報ではなく、個人の概念体系内で「パフォーマンス最適化」「React フック」といった既存概念との関係で意味を持つ知識として位置づけられる。**LLM 協働**により「LLM の叩き台＋個人のツッコミ」で効率的に個性を注入でき、ゼロからの要約作成負荷を削減しながら自分の体験・視点を反映できる。この組み合わせで「記事をブックマークして終わり」から「個人の概念体系に根ざした実用的な文献メモの即座作成」への体験変革が実現される。

<!-- REFERENCE_END: scenario-1 -->

#### シナリオ 2: 散らばった概念 → 整理された思考地図

<!-- REFERENCE_BEGIN: scenario-2 -->

**CogitoWeave での解決体験**: **視覚的・空間的アプローチ**により概念間の複雑な関係性を空間配置で直感的に理解・操作でき、**概念中心設計**により個人の関係語彙で関係性を定義できる。この組み合わせで「頭の中でモヤモヤ」から「空間上での明確な関係性整理」への体験変革が実現され、複雑な思考が視覚的に管理可能な思考地図として外化される。

<!-- REFERENCE_END: scenario-2 -->

#### シナリオ 3: ふとした疑問 → 関係性の発見

<!-- REFERENCE_BEGIN: scenario-3 -->

**CogitoWeave での解決体験**: **概念中心設計**により 2 つの概念間の関係性を「適用タイミング」「アプローチ」「共通目標」といった個人的な関係軸で体系的に定義でき、漠然とした疑問が明確な判断基準として整理される。この関係性の明示化により「なんとなく似てる」から「具体的な使い分け基準の確立」への体験変革が実現される。

<!-- REFERENCE_END: scenario-3 -->

#### シナリオ 4: 質問への回答 → 過去の学びの活用

<!-- REFERENCE_BEGIN: scenario-4 -->

**CogitoWeave での解決体験**: **概念中心設計**により質問内容が関連概念で自動分類され、**段階的知識深化**により各概念に蓄積された文献メモが統合的に活用できる。この組み合わせで「過去の学びが思い出せない」から「概念軸での知識資産の即座活用」への体験変革が実現され、散発的だった学びが現在の問題解決で統合的に機能する。

<!-- REFERENCE_END: scenario-4 -->

#### シナリオ 5: 漠然とした考え → 論理的な記事

<!-- REFERENCE_BEGIN: scenario-5 -->

**CogitoWeave での解決体験**: **視覚的・空間的アプローチ**により断片的思考を関係性マップとして空間配置でき、**概念中心設計**により「選択肢の多さ」→「長期視点の欠如」→「人気に流される」といった論理的関係連鎖を明確化できる。この組み合わせで「漠然とした考えの集合」から「論理構造を持った価値ある情報発信」への体験変革が実現され、断片的思考が体系的な論理として外化される。

<!-- REFERENCE_END: scenario-5 -->

## 02-organizational-capacity - 組織能力

### リソース管理

#### 予算・時間制約活用

<!-- REFERENCE_BEGIN: budget-time-constraints -->

限られた予算(月 1000 円)と時間(週 7 時間)に適応した効率的な開発手法を選択する。

<!-- REFERENCE_END: budget-time-constraints -->

#### 既存リソース制約

<!-- REFERENCE_BEGIN: existing-resource-constraints -->

自宅サーバー(Ubuntu)と Cloudflare サービスの最大限活用により追加コストを最小化する。

<!-- REFERENCE_END: existing-resource-constraints -->

### 市場ポジション

<!-- REFERENCE_BEGIN: market-constraints -->

#### 既存ツール代替可能性制約

- 既存ツールでは実現困難な独自価値の提供が必須である。

#### 個人開発の市場環境制約

- 他者からの評価や社会的な標準に合わせる必要がない。

<!-- REFERENCE_END: market-constraints -->

### 開発方針

#### 開発・保守体制制約

<!-- REFERENCE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- REFERENCE_END: development-constraints -->

#### 公開方針制約

<!-- REFERENCE_BEGIN: publication-constraints -->

- 複雑なコードレビュー・品質標準・セキュリティ監査等の配慮は不要である。
- 設計書は CC BY ライセンスで公開する。

<!-- REFERENCE_END: publication-constraints -->

## 03-external-environment

外部環境は考慮しないため省略。

## 04-system-properties

### 性能効率性

<!-- REFERENCE_BEGIN: data-migration-policy -->

他のナレッジツールからのデータ移行機能は提供しない。開発・保守の複雑度制限とコア機能への開発リソース集中を優先する。

<!-- REFERENCE_END: data-migration-policy -->

<!-- REFERENCE_BEGIN: platform-compatibility -->

プラットフォーム互換性は Web 配信のみを前提とする。既存インフラの活用とコスト最小化を優先し、複数プラットフォーム対応による保守負荷増大を避ける。

<!-- REFERENCE_END: platform-compatibility -->

<!-- REFERENCE_BEGIN: export-functionality -->

エクスポート機能はバックアップ用途と LLM に渡すための Markdown 出力のみ提供する。個人の思考支援に特化した独自価値の実現を優先する。

<!-- REFERENCE_END: export-functionality -->

### 使用性 (Usability)

#### 効率性重視の操作設計

<!-- REFERENCE_BEGIN: efficiency-focused-operation -->

- **マウス操作中心**: 文字入力以外はマウス操作を基本とした直感的インターフェース
- **GUI 重視**: 複雑な操作もクリック・ドラッグ等の視覚的操作で完結
- **操作学習前提**: 初回直感性より継続利用での効率性を重視

<!-- REFERENCE_END: efficiency-focused-operation -->

#### デバイス最適化インターフェース

<!-- REFERENCE_BEGIN: device-optimized-interface -->

- **Mac 環境**: マルチウィンドウ・複雑レイアウト活用による情報整理専用 UI
- **iPhone 環境**: シンプル・タッチ最適化による情報参照専用 UI
- **レスポンシブ設計**: デバイス切り替え時の文脈継続性確保

<!-- REFERENCE_END: device-optimized-interface -->

#### 個人最適化による柔軟性

<!-- REFERENCE_BEGIN: personal-optimization-flexibility -->

- **決め打ち設計**: 作成者の操作パターンに最初から最適化した固定インターフェース
- **標準的ガイダンス排除**: 一般的なヘルプ・チュートリアル等は実装しない
- **シンプルな操作体系**: 複雑な設定や履歴機能なしの直接的操作

<!-- REFERENCE_END: personal-optimization-flexibility -->

### 信頼性 (Reliability)

#### データ保護要件

<!-- REFERENCE_BEGIN: data-protection-requirements -->

- **自動バックアップ**: 日次自動バックアップによる合理的なデータ保護
- **迅速復旧**: 障害発生時の当日復旧を目標とした手順整備
- **部分ロス許容**: 完全なゼロリスクは求めず、実用的保護レベルで運用

<!-- REFERENCE_END: data-protection-requirements -->

#### 可用性要件

<!-- REFERENCE_BEGIN: availability-requirements -->

- **自己保守前提**: 作成者による障害検知・復旧対応を前提とした設計
- **メンテナンス時間許容**: 計画的メンテナンス時のダウンタイムは許容
- **シンプル構成優先**: 高可用性より保守性・理解しやすさを重視

<!-- REFERENCE_END: availability-requirements -->

#### エラー処理要件

<!-- REFERENCE_BEGIN: error-handling-requirements -->

- **基本ログ記録**: 障害原因特定のための最小限のログ出力
- **通知機能除外**: リアルタイム障害通知は実装しない
- **監視機能任意**: 基本監視は実装可能だが必須ではない

<!-- REFERENCE_END: error-handling-requirements -->

#### 外部依存耐性

<!-- REFERENCE_BEGIN: external-dependency-resilience -->

- **機能分離設計**: LLM 機能障害時もコア機能(保存・閲覧・編集)は継続動作
- **API 障害部分許容**: 外部 API 障害時は該当機能のみ利用不可で運用継続
- **ブラウザ互換性**: 標準 Web 技術使用による互換性問題の最小化

<!-- REFERENCE_END: external-dependency-resilience -->

#### データ整合性要件

<!-- REFERENCE_BEGIN: data-consistency-requirements -->

- **順次アクセス前提**: 同時編集制御は実装せず、利用パターンで回避
- **確実な同期**: デバイス間データ同期の確実性を優先
- **バージョン管理**: 重要データの履歴保持機能を任意実装

<!-- REFERENCE_END: data-consistency-requirements -->

### セキュリティ

<!-- REFERENCE_BEGIN: security-requirements -->

- 必須対応
  - 外部 API 接続時の HTTPS 通信(LLM API・MCP 統合)
  - SQL インジェクション対策・XSS 対策の基本実装
  - 個人情報保護法に基づくプライバシー配慮設計
  - LLM API 利用規約・各種サービス規約の遵守
  - 攻撃対象面の限定(作成者個人環境のみ)によるリスク軽減
- 任意対応
  - 作成者本人のみアクセス可能なアクセス制御
  - 全通信の HTTPS 暗号化
- 除外対応
  - 高度なユーザー認証機能(プロトタイプ段階)
  - データ暗号化(プロトタイプ段階)
  - 詳細なアクセス制御(プロトタイプ段階)

<!-- REFERENCE_END: security-requirements -->

### 保守性 (Maintainability)

#### コード品質要件

<!-- REFERENCE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- REFERENCE_END: code-quality-requirements -->

#### ドキュメント要件

<!-- REFERENCE_BEGIN: documentation-requirements -->

- **詳細な設計書を作成**: 設計手法の実験として、通常よりも詳しい設計ドキュメントを書く
- **誰でも理解できるように書く**: 作成者以外でも読んで分かるレベルの説明を心がける
- **継続的に更新する**: コードを変更したときに合わせてドキュメントも更新する仕組みを作る

<!-- REFERENCE_END: documentation-requirements -->

#### テスト・自動化要件

<!-- REFERENCE_BEGIN: testing-automation-requirements -->

- **バックエンドは完全に自動テスト**: ほぼ 100%のテストで品質を担保する
- **フロントエンドは状況に応じて使い分け**: ブラウザでの複雑な操作やデザインの確認は手動テスト、ロジック部分は自動テストを行う
- **障害対応をスクリプト化**: 復旧作業を自動化して保守の負担を減らす
- **データ移行を自動化**: データベース構造を変更するときは自動で処理する

<!-- REFERENCE_END: testing-automation-requirements -->

#### 技術選択基準

<!-- REFERENCE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- REFERENCE_END: technology-selection-criteria -->

#### アーキテクチャの安定性要件

<!-- REFERENCE_BEGIN: architecture-stability-requirements -->

- **変更が必要になる原因を事前に排除**: 疎結合な設計と安定した技術を選んで、アーキテクチャを変更する必要がないようにする
- **技術的な負債を予防**: 機能を追加するときに既存コードを大幅に変更する必要がある場合は設計の問題として、継続的に改善で対応する
- **局所的な変更で機能拡張**: 新機能は既存のアーキテクチャの枠組みの中で、部分的な変更で実現する

<!-- REFERENCE_END: architecture-stability-requirements -->

### プラットフォーム制約

#### ブラウザ対応制約

<!-- REFERENCE_BEGIN: browser-support -->

- 必須対応
  - 対応ブラウザ: iOS Safari(iPhone)、Chrome(Mac)
  - 採用 Web 標準: Baseline Newly available を基準とした機能のみ使用
  - 対応方針: 確定デバイスでの完全動作確保
- 任意対応
  - 他環境での動作は Baseline Newly available による副次的効果として期待
- 除外対応
  - Internet Explorer・古いブラウザとの互換性は考慮しない
  - Baseline Newly available 未対応の機能は使用しない

<!-- REFERENCE_END: browser-support -->

#### プラットフォーム制約まとめ

<!-- REFERENCE_BEGIN: platform-constraints -->

1. **Mac 環境制約**: Chrome 上で情報整理機能が完全動作すること
2. **iPhone 環境制約**: Safari 上で情報参照機能が基本動作すること
3. **Web 標準採用基準**: Baseline Newly available を技術選択の判断基準とすること
4. **マルチデバイス前提**: Web 技術による複数デバイス間データ共有を前提とすること

これ以外の基本セキュリティ対策(HTTPS、SQLi/XSS 対策等)、現代 Web 標準性能要件、個人情報保護法等の法規制遵守、個人環境限定によるセキュリティリスク軽減は現代 Web 開発標準に準拠する。

<!-- REFERENCE_END: platform-constraints -->

### パフォーマンス制約

<!-- REFERENCE_BEGIN: performance-requirements -->

- 必須対応
  - 概念ページ読み込み: 2 秒以内(既存ツールレベル)
  - 検索結果表示: 1 秒以内(現代の検索体験期待)
  - LLM API 呼び出し: 10 秒以内(生成 AI 市場期待)
  - データベース応答: ミリ秒オーダー(現代ソフトウェア基本期待)
- 任意対応
  - 既存ツールを上回るパフォーマンス最適化
- 除外対応
  - 複数ユーザー同時アクセス: 個人ツール市場では期待されない
  - 大規模データ処理: 個人用途範囲を超える性能は不要
  - 企業向け機能: 個人ツール市場では要求されない

<!-- REFERENCE_END: performance-requirements -->

### 対象プラットフォーム

<!-- REFERENCE_BEGIN: target-platforms -->

- 必須対応
  - 情報整理機能: Mac (Chrome) - デスクトップ環境中心
  - 情報参照機能: Mac (Chrome) + iPhone (Safari) - 全環境対応
  - 実行環境: Web 技術による実装、複数デバイス間でのデータ共有を前提
- 任意対応
  - 作成者環境外での動作は妨げないが保証しない
  - Web 技術による実装のため、一般的なモダンブラウザで基本動作は期待できる
- 除外対応
  - ネイティブアプリ開発(iOS・macOS 別実装)
  - ローカルストレージのみによる単一デバイス運用

<!-- REFERENCE_END: target-platforms -->

## Bootstrap 的開発方針

<!-- REFERENCE_BEGIN: bootstrap-development-approach -->

- CogitoWeave の設計書作成プロセスは、完成前から思考支援効果を提供する。
  - Bootstrap 的開発により、開発プロセス自体がプロダクト価値となるため。
- 開発プロセスと改善プロセスが継続的に最適化される。
  - ツールを使いながら改善する循環により、プロセス品質が向上し続けるため。
- 思考支援ツール制作を通じて、開発者のメタ認知能力が向上する。
  - 思考プロセスを意識化・体系化する経験により、思考能力そのものが強化されるため。

<!-- REFERENCE_END: bootstrap-development-approach -->
