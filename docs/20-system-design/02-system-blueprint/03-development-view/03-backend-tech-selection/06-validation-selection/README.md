---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "orm-selection"
    - "api-tech-approach"
  produces:
    - "validation-tech-selection"
---

# バリデーション技術選択

## 概要

FastAPI エコシステム内でのバリデーション技術選択。セキュリティ要件とパフォーマンス制約を満たし、既存の技術スタックとの連携を考慮した最適解を決定する。

### 前提

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: orm-selection -->

ORM/ODM ツールとして SQLAlchemy + Alembic を採用する。

<!-- PREMISE_END: orm-selection -->

<!-- PREMISE_BEGIN: api-tech-approach -->

API 技術方針として REST API を採用する。

<!-- PREMISE_END: api-tech-approach -->

## 選択肢と判断基準

FastAPI エコシステム内でのバリデーション技術の選択を行う。セキュリティ要件とパフォーマンス制約を満たし、SQLAlchemy との連携を考慮した最適解を決定する。

### 判断基準

- FastAPI 親和性 (優先度高) - FastAPI との統合の自然さ
- セキュリティ対応 (優先度高) - SQL インジェクション・XSS 対策への貢献
- パフォーマンス - データベース応答ミリ秒オーダー制約への適合
- 保守性 - 一人開発における理解しやすさと管理負荷
- SQLAlchemy 連携 - データベース制約との役割分担の明確さ

### 選択肢

- FastAPI 標準 Pydantic: FastAPI 内蔵のバリデーション機能
- Pydantic + カスタムバリデータ: Pydantic を拡張したカスタム検証
- marshmallow: Python の定番シリアライゼーション・バリデーションライブラリ
- cerberus: 軽量なバリデーション専用ライブラリ

## 各選択肢の詳細分析

Python エコシステムにおける主要なバリデーションライブラリを FastAPI との親和性と機能特性で比較評価する。

### FastAPI 標準 Pydantic

FastAPI に内蔵された Pydantic による型安全なバリデーション機能。

- 利点
  - FastAPI との完全な統合により追加設定が不要
  - 型ヒントベースの直感的なバリデーション定義
  - 自動 OpenAPI スキーマ生成による API ドキュメント連携
  - Rust ベース pydantic-core による高速処理
  - SQLAlchemy モデルとの型共有が可能
- 欠点
  - 複雑なビジネスロジック検証には限界がある
  - カスタムバリデーション追加時の柔軟性が制限される
  - 多層バリデーション構成が困難

### Pydantic + カスタムバリデータ

FastAPI 標準 Pydantic に独自のバリデーション関数を追加した構成。

- 利点
  - FastAPI との統合を維持しながら柔軟性を確保
  - validator デコレータによる段階的なバリデーション拡張
  - 型安全性と複雑なロジック検証を両立
  - 既存の Pydantic エコシステムを活用可能
- 欠点
  - カスタムバリデータのテスト・保守負荷が増加
  - バリデーション複雑化による理解困難性
  - エラーハンドリング設計の追加負荷

### marshmallow

Python エコシステムで長期間使用されているシリアライゼーション・バリデーションライブラリ。

- 利点
  - 豊富なバリデーションルールと拡張性
  - 成熟したライブラリによる安定性
  - SQLAlchemy との連携パターンが確立
  - 複雑なネストしたデータ構造への対応
- 欠点
  - FastAPI との統合に追加の接続コードが必要
  - Pydantic より学習曲線が急峻
  - パフォーマンスが Pydantic より劣る
  - 型安全性の恩恵を受けにくい

### cerberus

バリデーション専用の軽量ライブラリで、シンプルなルールベース検証を提供。

- 利点
  - 軽量でシンプルな API による理解しやすさ
  - 宣言的なスキーマ定義による保守性
  - 他ライブラリとの依存関係が少ない
  - カスタムバリデータの追加が容易
- 欠点
  - FastAPI との統合に手動実装が必要
  - 型安全性のサポートが限定的
  - エコシステムが Pydantic より小規模
  - SQLAlchemy との連携パターンが未確立

## 判断基準による評価

### FastAPI 親和性の比較

FastAPI 標準 Pydantic が圧倒的に優位である。内蔵機能のため追加設定不要で、自動 OpenAPI スキーマ生成、型安全性、パフォーマンスを自然に活用できる。他の選択肢は追加の統合作業が必要となる。

### セキュリティ対応の比較

全選択肢とも基本的なセキュリティ検証(型チェック・範囲チェック・形式チェック)は対応可能である。Pydantic 系は型安全性による静的チェックでセキュリティホールを予防でき、marshmallow は豊富なバリデーションルールでより詳細な検証が可能である。

### パフォーマンスの比較

Pydantic 系が最高性能を示す。Rust ベース pydantic-core による最適化で、データベース応答ミリ秒オーダー制約を余裕で満たす。marshmallow と cerberus は Python 実装のため相対的に劣るが、個人用途では実用的な範囲である。

### 保守性の比較

FastAPI 標準 Pydantic が最も保守しやすい。一人開発制約下で、追加ライブラリの学習・保守負荷を避け、FastAPI エコシステム内で完結できる利点は決定的である。

### SQLAlchemy 連携の比較

Pydantic 系は SQLAlchemy モデルからの自動スキーマ生成により、データベース制約と API バリデーションの役割分担が明確化される。marshmallow も確立された連携パターンがあるが、追加実装が必要である。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: validation-tech-selection -->

バリデーション技術として FastAPI 標準 Pydantic を採用する。

<!-- GLOBAL_CONCLUSION_END: validation-tech-selection -->

### 選択根拠

FastAPI との完全統合による開発効率と保守性が決定要因である。追加ライブラリ不要でセキュリティ要件を満たし、型安全性による静的チェックで SQL インジェクション・XSS 対策に貢献する。Rust ベース pydantic-core の高速処理でパフォーマンス制約を満たし、一人開発制約下での理解しやすさと管理負荷の軽減を実現する。SQLAlchemy との型共有により、データベース制約と API バリデーションの役割分担が自然に確立される。
