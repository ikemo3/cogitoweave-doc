---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "usage-context"
    - "development-constraints"
  produces:
    - "data-protection-requirements"
    - "availability-requirements"
    - "error-handling-requirements"
    - "external-dependency-resilience"
    - "data-consistency-requirements"
---

# 信頼性 (Reliability)

## 前提

<!-- PREMISE_BEGIN: usage-context -->

- **利用者**: 単一ユーザー専用システム
  - 個人の思考プロセスに完全最適化
  - マルチユーザー対応の複雑性を排除
  - 「ツッコミを入れる」個人的判断プロセスの前提

<!-- PREMISE_END: usage-context -->

<!-- PREMISE_BEGIN: development-constraints -->

- シンプルで理解しやすい構造、一人で管理可能な複雑度に制限する。

<!-- PREMISE_END: development-constraints -->

### データ特性

- **蓄積型知識資産**: 思考メモ・概念・関係性等の長期蓄積データ
- **個人依存性**: 作成者以外には価値を持たないが、作成者には高価値
- **段階的成長**: 一度失うと再構築困難な知的資産

### 利用パターン

- **単一デバイス順次利用**: Mac 保存後 iPhone 読み込みの順次アクセス
- **作成者自身による保守**: 障害発生・復旧判断・対応全て作成者が実施
- **外部 API 依存**: LLM 機能は外部 API 障害時の代替手段を想定

## 論理

### 個人用システムによる信頼性要件軽減

単一ユーザー専用システムであるため、マルチユーザー環境の高可用性(99.9%稼働率等)は不要である。むしろ、作成者自身が保守するため、障害発生時の迅速な気づき・対応が可能であり、シンプルな構成による保守性向上を優先すべきである。

### 蓄積型知識資産による部分的データ保護必要性

思考メモ等の知的資産は一度失うと再構築困難であるが、完全なゼロリスクは現実的でない。自動バックアップによる合理的なデータ保護と、迅速な復旧手順の整備により、実用的なデータ保護レベルを実現すべきである。

### 順次利用パターンによる同時性問題回避

Mac 保存後 iPhone 読み込みの利用パターンにより、複雑な同時編集制御は不要である。むしろ、デバイス間の確実なデータ同期と、編集競合が発生しない設計により、シンプルで確実な整合性を実現すべきである。

### 外部依存の部分的許容による機能分離

LLM API 等の外部依存は完全回避不可能だが、コア機能(データ保存・閲覧・編集)は外部依存なしで動作させ、LLM 機能のみ外部障害時に利用不可とすることで、システム全体の可用性を確保すべきである。

## 結論

### データ保護要件

<!-- GLOBAL_CONCLUSION_BEGIN: data-protection-requirements -->

- **自動バックアップ**: 日次自動バックアップによる合理的なデータ保護
- **迅速復旧**: 障害発生時の当日復旧を目標とした手順整備
- **部分ロス許容**: 完全なゼロリスクは求めず、実用的保護レベルで運用

<!-- GLOBAL_CONCLUSION_END: data-protection-requirements -->

### 可用性要件

<!-- GLOBAL_CONCLUSION_BEGIN: availability-requirements -->

- **自己保守前提**: 作成者による障害検知・復旧対応を前提とした設計
- **メンテナンス時間許容**: 計画的メンテナンス時のダウンタイムは許容
- **シンプル構成優先**: 高可用性より保守性・理解しやすさを重視

<!-- GLOBAL_CONCLUSION_END: availability-requirements -->

### エラー処理要件

<!-- GLOBAL_CONCLUSION_BEGIN: error-handling-requirements -->

- **基本ログ記録**: 障害原因特定のための最小限のログ出力
- **通知機能除外**: リアルタイム障害通知は実装しない
- **監視機能任意**: 基本監視は実装可能だが必須ではない

<!-- GLOBAL_CONCLUSION_END: error-handling-requirements -->

### 外部依存耐性

<!-- GLOBAL_CONCLUSION_BEGIN: external-dependency-resilience -->

- **機能分離設計**: LLM 機能障害時もコア機能(保存・閲覧・編集)は継続動作
- **API 障害部分許容**: 外部 API 障害時は該当機能のみ利用不可で運用継続
- **ブラウザ互換性**: 標準 Web 技術使用による互換性問題の最小化

<!-- GLOBAL_CONCLUSION_END: external-dependency-resilience -->

### データ整合性要件

<!-- GLOBAL_CONCLUSION_BEGIN: data-consistency-requirements -->

- **順次アクセス前提**: 同時編集制御は実装せず、利用パターンで回避
- **確実な同期**: デバイス間データ同期の確実性を優先
- **バージョン管理**: 重要データの履歴保持機能を任意実装

<!-- GLOBAL_CONCLUSION_END: data-consistency-requirements -->
