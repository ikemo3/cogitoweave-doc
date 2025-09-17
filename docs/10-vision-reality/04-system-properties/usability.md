---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "usage-context"
    - "target-platforms"
  produces:
    - "efficiency-focused-operation"
    - "device-optimized-interface"
    - "personal-optimization-flexibility"
---

# 使用性 (Usability)

## 前提

<!-- PREMISE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- PREMISE_END: usage-context -->

<!-- PREMISE_BEGIN: target-platforms -->

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

<!-- PREMISE_END: target-platforms -->

### 利用者特性

- **技術的熟練度**: 高度な技術リテラシーを持つ開発者
- **利用頻度**: ヘビーユーザー前提での継続的利用
- **操作習得**: 操作習得のための時間投資は不要(作成者自身のため)

### 操作環境特性

- **Mac 環境**: マウス・キーボード併用による情報整理作業
- **iPhone 環境**: タッチ操作による情報参照・軽微な編集
- **マルチタスク**: Mac 環境でのマルチウィンドウ・複数アプリ並行利用
- **レスポンシブ対応**: デバイス間シームレス移行が前提

## 論理

### 単一ユーザー専用による操作性特化

単一ユーザー専用システムであるため、一般的なユーザビリティ配慮(直感的アイコン、詳細ヘルプ、操作ガイド等)は不要である。むしろ、作成者の操作パターンと思考プロセスに完全最適化した効率的インターフェースを提供すべきである。

### 技術的熟練度による操作複雑度許容

利用者が高度な技術リテラシーを持つため、複雑な画面レイアウトや多機能なインターフェース等の効率性重視の操作が許容される。むしろ、単純化による操作効率低下を避けるべきである。

### デバイス特性による役割分離設計

Mac 環境では情報整理(思考の体系化・関係性の整理)、iPhone 環境では情報参照(外出先での確認・軽微な追記)という役割分離により、各デバイスの操作特性を活かした専用インターフェースが適切である。

### 継続利用による操作学習効果活用

ヘビーユーザー前提のため、初回利用時の直感性より継続利用時の効率性を重視すべきである。操作に慣れることで高速化される複雑な操作系の採用が適切である。

## 結論

### 効率性重視の操作設計

<!-- GLOBAL_CONCLUSION_BEGIN: efficiency-focused-operation -->

- **マウス操作中心**: 文字入力以外はマウス操作を基本とした直感的インターフェース
- **GUI 重視**: 複雑な操作もクリック・ドラッグ等の視覚的操作で完結
- **操作学習前提**: 初回直感性より継続利用での効率性を重視

<!-- GLOBAL_CONCLUSION_END: efficiency-focused-operation -->

### デバイス最適化インターフェース

<!-- GLOBAL_CONCLUSION_BEGIN: device-optimized-interface -->

- **Mac 環境**: マルチウィンドウ・複雑レイアウト活用による情報整理専用 UI
- **iPhone 環境**: シンプル・タッチ最適化による情報参照専用 UI
- **レスポンシブ設計**: デバイス切り替え時の文脈継続性確保

<!-- GLOBAL_CONCLUSION_END: device-optimized-interface -->

### 個人最適化による柔軟性

<!-- GLOBAL_CONCLUSION_BEGIN: personal-optimization-flexibility -->

- **決め打ち設計**: 作成者の操作パターンに最初から最適化した固定インターフェース
- **標準的ガイダンス排除**: 一般的なヘルプ・チュートリアル等は実装しない
- **シンプルな操作体系**: 複雑な設定や履歴機能なしの直接的操作

<!-- GLOBAL_CONCLUSION_END: personal-optimization-flexibility -->
