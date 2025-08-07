---
doc_type: "premise-logic-conclusion"
status: "draft"
dependency:
  prev_file: "01-platform-constraints.md"
  next_file: "03-performance-requirements.md"
---

# ブラウザ対応制約

## 前提条件とスコープ

この設計判断は CogitoWeave システムのブラウザ対応範囲の選択を扱う。[プラットフォーム制約](01-platform-constraints.md)で決定されたWeb技術による実装を前提として、作成者の実際のデバイス環境（iPhone・Mac 確定、iPad・Chromebook 追加可能性）、開発効率の最適化に基づく対応範囲を決定する。

**このファイルのスコープ**: ブラウザ対応範囲の設計判断のみ。具体的な技術実装はスコープ外。

## 設計判断の論理

CogitoWeave のブラウザ対応制約は、作成者の実際のデバイス環境と Web 技術による実装効率を基準として設定する必要がある。

まず、作成者の確定デバイス環境として、iPhone（iOS Safari）と Mac（Chrome）での動作が必須となる。個人のデジタルツェッテルカステンは日常的に使用するツールであるため、実際に所有しているデバイスでの確実な動作が最優先である。この 2 つの環境での動作確保が基本要件となる。

次に、Web 標準の採用基準として、Baseline Newly available を基準とする。Baseline Newly available は主要ブラウザすべてで対応が確認された最新の Web 標準を示すため、実装の信頼性と最新機能の活用を両立できる。Baseline Newly available で利用可能な機能のみを使用することで、確定デバイス以外でも基本的な動作互換性を期待できる。

この実際のデバイス環境と Baseline Newly available 基準に集中することで、開発効率の最大化を図る。Internet Explorer や古いバージョンのブラウザ対応は完全に除外し、Baseline Newly available で利用可能なモダン Web 標準を積極的に活用する。限定された環境での最適化により、高品質な実装と迅速な開発を両立する。

## 結論

CogitoWeave のブラウザ対応制約を以下のように決定する：

- **必須対応ブラウザ**: iOS Safari（iPhone）、Chrome（Mac）
- **採用 Web 標準**: Baseline Newly available を基準とした機能のみ使用
- **除外対応**: Internet Explorer・古いブラウザとの互換性は考慮しない
- **対応方針**: 確定デバイスでの完全動作確保、他環境での動作は Baseline Newly available による副次的効果
