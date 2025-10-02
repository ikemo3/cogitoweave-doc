---
doc_type: "pattern-research"
status: "draft"
depends:
  contracts:
    - "frontend-rendering-style"
    - "testing-automation-requirements"
    - "technology-selection-criteria"
    - "code-quality-requirements"
    - "system-test-strategy"
  produces:
    - "testing-tools-research"
---

# フロントエンドテストツール調査

## 前提

<!-- PREMISE_BEGIN: frontend-rendering-style -->

CogitoWeave のフロントエンドレンダリングスタイルとして**CSR(Client-Side Rendering)**を採用する。3 画面間の状態共有効率と軽量なコンポーネント指向の特性を活かし、一人での開発における保守性と理解しやすさを確保する。

<!-- PREMISE_END: frontend-rendering-style -->

<!-- PREMISE_BEGIN: testing-automation-requirements -->

- **バックエンドは完全に自動テスト**: ほぼ 100%のテストで品質を担保する
- **フロントエンドは状況に応じて使い分け**: ブラウザでの複雑な操作やデザインの確認は手動テスト、ロジック部分は自動テストを行う
- **障害対応をスクリプト化**: 復旧作業を自動化して保守の負担を減らす
- **データ移行を自動化**: データベース構造を変更するときは自動で処理する

<!-- PREMISE_END: testing-automation-requirements -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: code-quality-requirements -->

- **継続的リファクタリング**: 息をするような自然な改善プロセスを組み込む
- **シンプルな設計を優先**: 一人で理解して保守できる程度の複雑さに留める
- **各機能の役割を明確に分ける**: 機能を変更するときに影響する範囲を限定的にする

<!-- PREMISE_END: code-quality-requirements -->

<!-- PREMISE_BEGIN: system-test-strategy -->

- **CI 統合疎通テスト**: フロントエンドから 3 サービスへの基本疎通確認のみを CI 環境で実行する。
- **外部 API 依存をモック**: バックエンドサービス内でモック実装によりダミーデータを返却して検証する。
- **バックエンド実起動**: フロントエンドからの HTTP 通信を受け付けるため、フロントエンド側でのモック機能は不要となる。
- **正常系とセキュリティをカバー**: エラーハンドリングや複雑な E2E シナリオは単体テストレベルで対応済みとして除外する。
- **CI 失敗時の迅速復旧**: PR マージ停止、修正または revert による迅速復旧を基本とする。

<!-- PREMISE_END: system-test-strategy -->

## 調査

### 情報源

State of JS 2024 のテストツール調査結果を分析する。

調査対象は以下 3 つの条件のいずれかを満たすツールに絞り込んだ。

- シェアが高い（使用率 30% 以上）: Jest、Vitest、Mocha、Storybook、Cypress、Playwright、Puppeteer、Testing Library
- 満足度が高い（満足度 85% 以上）: Vitest、Node Test Runner、Testing Library、Playwright、Puppeteer、Mock Service Worker
- 要件へのマッチ度が高い: Vitest、Node Test Runner、Testing Library、Playwright、Storybook

除外対象: Selenium、WebdriverIO、TestCafe は技術的適合性が低く、モック・テストユーティリティカテゴリはシステムテスト戦略によりフロントエンド側でのモック機能が不要と確定しているため調査対象外とした。

### 使用率 (Usage)

#### 単体テストフレームワークの使用率

- Jest: 74% → 72% (高使用率、微減)
- Vitest: 33% → 40% (中使用率、上昇)
- Mocha: 45% → 40% (中使用率、減少)
- Node Test Runner: 7% (新規、低使用率)

#### コンポーネントテストツールの使用率

- Storybook: 52% → 52% (中使用率、安定)
- Testing Library: 40% → 37% (中使用率、微減)

#### E2E テストツールの使用率

- Cypress: 47% → 45% (高使用率、微減)
- Playwright: 28% → 36% (中使用率、上昇)
- Puppeteer: 39% → 36% (中使用率、減少)

### 関心度 (Interest)

#### 単体テストフレームワークの関心度

- Vitest: 79% → 82% (高関心、上昇)
- Node Test Runner: 76% (新規、高関心)
- Jest: 67% → 52% (中関心、減少)
- Mocha: 38% → 23% (低関心、減少)

#### コンポーネントテストツールの関心度

- Storybook: 69% → 63% (高関心、減少)
- Testing Library: 51% → 55% (中関心、上昇)

#### E2E テストツールの関心度

- Playwright: 76% → 77% (最高関心、微上昇)
- Cypress: 61% → 47% (中関心、減少)
- Puppeteer: 52% → 43% (中関心、減少)

### 満足度 (Satisfaction)

#### 単体テストフレームワークの満足度

- Vitest: 97% → 98% (最高満足度、微上昇)
- Node Test Runner: 87% (新規、高満足度)
- Jest: 79% → 73% (中満足度、減少)
- Mocha: 52% → 61% (中満足度、上昇)

#### コンポーネントテストツールの満足度

- Testing Library: 90% → 91% (最高満足度、微上昇)
- Storybook: 81% → 71% (高満足度、減少)

#### E2E テストツールの満足度

- Playwright: 95% → 94% (最高満足度、微減)
- Puppeteer: 72% → 74% (高満足度、上昇)
- Cypress: 71% → 64% (中満足度、減少)

#### 調査から除外した項目

以下のツールは技術的適合性が低いため除外:

- **Selenium**: 使用率 34%、満足度 25%、関心度 21% - 満足度・関心度が極端に低い
- **WebdriverIO**: 使用率 8%、満足度 30%、関心度 24% - 全指標が低水準
- **TestCafe**: 使用率 3%、満足度 39%、関心度 11% - 使用率・関心度が極端に低い

## 論理

第一に、State of JS 2024 の調査データから読み取れる基本的な傾向として、テストツール領域では明確な世代交代が進行している。従来の主流ツール(Jest、Cypress)に対して、新世代ツール(Vitest、Playwright)が満足度と関心度の両面で大幅に上回る状況が確認できる。特に Vitest の満足度 98%、Playwright の満足度 94%という数値は、技術的な優位性を明確に示している。

第二に、前提条件との照合において、一人での開発制約と保守性重視の要件に対して、各カテゴリで最適な候補が異なる傾向が見られる。単体テストフレームワークでは Vitest が満足度・関心度で圧倒的優位を示し、シンプルな設計と高速実行により一人開発に適合する。コンポーネントテストでは Testing Library が満足度 91%で安定した品質を提供し、E2E テストでは Playwright が技術的革新性と満足度の両面で最優位となっている。

第三に、従来主流ツールの技術的制約が選択判断に重要な影響を与えている。Jest は使用率 72% で最高シェアを持つが、満足度が 79%→73% に減少している背景として、ESM 対応の不完全性による制限事項があり、開発者コミュニティから Vitest への移行を推奨する声が多い。この技術的制約により、現代的な JavaScript 開発環境において Jest の選択は慎重な検討が必要となる。

第四に、制約条件下での最適解として、フロントエンドテストの「状況に応じた使い分け」要件を満たすためには、各カテゴリから高満足度・高関心度の候補を選択し、統合的なテスト戦略を構築することが必要である。外部ライブラリの適度な制限という制約を考慮すると、過度に複雑な構成は避け、必要最小限の高品質ツールによる組み合わせが最適となる。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: testing-tools-research -->

State of JS 2024 の調査結果に基づくテストツール候補の調査結果は以下の通りである。

### テストツール最優先候補

- **単体テストフレームワーク**
  - **Vitest**: 満足度 98% で最高水準を示し、ESM 対応と高速実行により現代的な開発環境に最適化されている。
- **コンポーネントテストツール**
  - **Testing Library**: 満足度 91% でコンポーネントテストの安定した品質を提供し、シンプルなテスト記述により保守性重視の要件に適合する。
- **E2E テストツール**
  - **Playwright**: 満足度 94% で E2E テストの技術革新を牽引し、関心度 77% と将来性も高い。

### テストツール次点候補

- **単体テストフレームワーク**
  - **Node Test Runner**: 満足度 87%、関心度 76% と高評価であり、依存関係を最小化する軽量な選択肢として注目される。
- **コンポーネントテストツール**
  - **Storybook**: 使用率 52% でコンポーネントテストツールの主流であり、関心度 63% と継続的な注目を集めている。
- **E2E テストツール**
  - **Puppeteer**: 満足度 74% で安定した品質を提供し、使用率 36% で実績がある。

### テストツール除外候補

- **単体テストフレームワーク**
  - **Jest**: 使用率 72% で最高シェアを持つが、満足度 73% と技術革新性で劣り、ESM 対応の不完全性により現代的な開発環境での制約がある。
- **E2E テストツール**
  - **Cypress**: 使用率 45% で E2E テストツールの実績を持つが、満足度 64% と技術革新性で新世代ツールに劣る。

従来の主流ツールから新世代ツールへの世代交代が明確であり、Vitest、Testing Library、Playwright が最有力の候補として浮上し、Node Test Runner、Storybook、Puppeteer が次点の候補、Jest と Cypress が除外すべき候補となる。

<!-- GLOBAL_CONCLUSION_END: testing-tools-research -->
