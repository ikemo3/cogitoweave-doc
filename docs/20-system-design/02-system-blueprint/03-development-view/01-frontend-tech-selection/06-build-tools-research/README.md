---
doc_type: "pattern-research"
status: "draft"
depends:
  contracts:
    - "app-structure-evaluation"
    - "performance-optimization-policy"
  produces:
    - "build-tools-research"
---

# ビルドツール調査

## 前提

<!-- PREMISE_BEGIN: app-structure-evaluation -->

CogitoWeave のアプリケーション構造として以下を採用する。

- 基本構造は MPA とする。
  - 画面間状態共有が不要で、各画面の独立性を重視するため。
- メイン画面内の概念間移動のみ SPA 的実装を適用する。
  - 概念ページ読み込み最優先要件を満たすため。
- 3 画面間の移動は別ページ遷移とする。
  - 各画面の軽量性と独立性を確保するため。
- ハイブリッドアプローチにより必要な部分のみ最適化する。
  - 一人開発制約下での理解しやすさと保守性を維持するため。

<!-- PREMISE_END: app-structure-evaluation -->

<!-- PREMISE_BEGIN: performance-optimization-policy -->

CogitoWeave のパフォーマンス最適化方針として以下を採用する。

- 概念ページ読み込み 2 秒以内を最優先とする。
  - 最も頻繁な日常操作の快適性確保を最重要視する。
- 初回ロードの遅さは許容し、ページ遷移の高速性を重視する。
  - CSR 特性を活かした継続利用の効率性を優先する。
- 複雑なパフォーマンス最適化は実装しない。
  - 一人開発制約により、理解しやすさと保守性を優先する。
- バンドルサイズよりも開発効率を重視する。
  - 必要な機能を提供するライブラリは適切に採用する。

<!-- PREMISE_END: performance-optimization-policy -->

<!-- REFERENCE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- REFERENCE_END: technology-selection-criteria -->

## 調査

### 情報源

State of JS 2024 のビルドツール調査結果を分析する。
<https://2024.stateofjs.com/en-US/libraries/build-tools/>

調査対象は 12 個のビルドツールのうち、以下 3 つの条件のいずれかを満たす 6 個に絞り込んだ。

- シェアが高い（使用率 70% 以上）: webpack、Vite
- 満足度が高い（満足度 85% 以上）: Vite、esbuild、Rolldown、Rspack
- 要件へのマッチ度が高い: Vite、esbuild、Parcel、Rolldown

除外対象: tsc CLI、SWC、Turbopack、tsup、Biome、Rollup は上記条件を満たさないため調査対象外とした。

### 使用率 (Usage)

- webpack: 90% → 86% (高使用率、減少傾向)
- Vite: 73% → 78% (高使用率、上昇傾向)
- esbuild: 50% → 49% (中使用率、安定)
- Parcel: 28% → 23% (低使用率、減少)
- Rspack: 3% (新規、低使用率)
- Rolldown: 1% (新規、低使用率)

### 関心度 (Interest)

- Rolldown: 86% (新規、最高関心)
- Vite: 78% → 82% (高関心、上昇)
- Rspack: 73% (新規、高関心)
- esbuild: 66% → 64% (高関心、微減)
- webpack: 45% → 33% (低関心、減少)
- Parcel: 40% → 28% (低関心、減少)

### 満足度 (Satisfaction)

- Vite: 98% → 98% (最高満足度、安定)
- esbuild: 91% → 91% (高満足度、安定)
- Rolldown: 91% (新規、高満足度)
- Rspack: 89% (新規、高満足度)
- Parcel: 56% → 68% (中満足度、上昇)
- webpack: 46% → 35% (低満足度、減少)

### 期待値と現実のギャップ (Appreciation)

- Parcel: 16% → 40% (期待より大幅に良い)
- esbuild: 24% → 26% (期待より良い)
- Rspack: 16% (新規、期待通り)
- Vite: 19% → 16% (期待通り)
- Rolldown: 5% (新規、期待通り)
- webpack: 1% → 1% (期待通り)

### 全体的評判 (Positivity)

- Vite: 93% → 95% (最高評判、上昇)
- esbuild: 81% → 78% (高評判、微減)
- Rolldown: 61% (新規、中評判)
- Rspack: 53% (新規、中評判)
- Parcel: 46% → 41% (低評判、微減)
- webpack: 46% → 34% (低評判、減少)

## 論理

State of JS 2024 のデータから、ビルドツールの候補を以下の 3 つの軸で評価する。

第一に、シェアが高いツールとして webpack（使用率 86%）と Vite（使用率 78%）が挙げられる。webpack は従来の主流ツールだが満足度 35%と開発者の支持を失っており、複雑な設定と遅いビルド速度が一人開発における生産性阻害要因となる。対照的に Vite は満足度 98%で最高水準を示し、開発サーバーの高速性と設定の簡素性により CSR 特性を活かした継続利用の効率性を実現している。

第二に、満足度が高いツールとして Vite（98%）、esbuild（91%）、Rolldown（91%）、Rspack（89%）が優秀である。esbuild は圧倒的なビルド速度により概念ページの読み込み要件に直接貢献する。Rolldown は Rollup の次世代版として Vite との統合を前提とした革新的設計により、ハイブリッドアプローチと高速ビルドという要件に適合する。Rspack は webpack 互換でありながら高速化を実現するが、webpack の複雑な設定をそのまま継承するため、一人開発制約（理解しやすさ・保守性重視）に不適合である。

第三に、要件へのマッチ度が高いツールとして Vite、esbuild、Parcel、Rolldown が候補となる。Vite はハイブリッドアプローチに対応し開発サーバーが高速である。esbuild はビルド速度で概念ページ読み込み要件に直結する。Parcel はゼロコンフィグで一人開発向きであり、満足度 68%（上昇傾向）と期待値ギャップ 40%（大幅改善）により再評価が必要である。Rolldown は革新的技術により要件適合性が高いが、新興ツールである点を考慮する必要がある。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: build-tools-research -->

State of JS 2024 のデータに基づくビルドツール候補の調査結果は以下の通りである。

### ビルドツール最優先候補

- **Vite**: 使用率 78%、満足度 98% でシェアと満足度を両立し、開発サーバーの高速性と設定の簡素性が一人開発に最適である。
- **esbuild**: 満足度 91% で圧倒的なビルド速度を誇り、概念ページの読み込み要件に直接的に貢献する。

### ビルドツール次点候補

- **Parcel**: ゼロコンフィグで一人開発向きであり、満足度 68%（上昇傾向）と期待値ギャップ 40%（大幅改善）により再評価が必要である。
- **Rolldown**: Rollup の次世代版として Vite との統合を前提とし、革新的技術により要件適合性が高いが新興ツールである。

### ビルドツール除外候補

- **webpack**: 使用率 86% で最高シェアを持つが、満足度 35% と開発者支持を失っており、複雑な設定と遅いビルド速度により除外対象である。
- **Rspack**: 満足度 89% で webpack 互換でありながら高速化を実現するが、webpack の複雑な設定をそのまま継承するため一人開発制約に不適合である。

ハイブリッドアプローチ、一人開発の制約、パフォーマンスの要件を総合すると、Vite と esbuild が最有力の候補として浮上し、Parcel と Rolldown が次点の候補、webpack と Rspack が除外すべき候補となる。

<!-- GLOBAL_CONCLUSION_END: build-tools-research -->
