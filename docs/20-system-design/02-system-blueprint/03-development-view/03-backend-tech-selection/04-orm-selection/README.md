---
doc_type: "pattern-choice"
status: "draft"
depends:
  contracts:
    - "language-framework-selection"
    - "sqlite-selection"
    - "technology-selection-criteria"
    - "testing-automation-requirements"
  produces:
    - "orm-selection"
---

# ORM/ODM・マイグレーション選択

## 概要

Python + FastAPI バックエンドでのデータアクセス層技術選択。SQLite を前提とし、データ移行の自動化と完全な自動テスト要件を満たすORM/ODMとマイグレーションツールを決定する。

<!-- PREMISE_BEGIN: language-framework-selection -->

Python + FastAPI を採用する。既存スキル活用、技術選択基準「大差ないときは慣れた技術を選ぶ」への適合、疎結合設計による密結合リスク回避により、最適な選択である。

<!-- PREMISE_END: language-framework-selection -->

<!-- PREMISE_BEGIN: sqlite-selection -->

CogitoWeave システムでは **SQLite** を選択する。

<!-- PREMISE_END: sqlite-selection -->

<!-- PREMISE_BEGIN: technology-selection-criteria -->

- **最適な技術を選ぶ**: ただし大差ないときは慣れた技術やメジャーなものを選ぶ
- **外部ライブラリは適度に制限**: 依存を少なくして、自分でコントロールできる範囲を広げる
- **監視は最小限**: システムが生きているかどうかを確認する程度の基本的な監視のみ
- **ログは少なめ**: テストでしっかり品質を保つので、ログに頼りすぎない

<!-- PREMISE_END: technology-selection-criteria -->

<!-- PREMISE_BEGIN: testing-automation-requirements -->

- **バックエンドは完全に自動テスト**: ほぼ 100%のテストで品質を担保する
- **フロントエンドは状況に応じて使い分け**: ブラウザでの複雑な操作やデザインの確認は手動テスト、ロジック部分は自動テストを行う
- **障害対応をスクリプト化**: 復旧作業を自動化して保守の負担を減らす
- **データ移行を自動化**: データベース構造を変更するときは自動で処理する

<!-- PREMISE_END: testing-automation-requirements -->

## 選択肢と判断基準

Python + FastAPI バックエンド環境での ORM/ODM とマイグレーションツールの選択を行う。SQLite をデータベースとして使用し、データ移行の自動化と完全な自動テスト要件を満たすソリューションを決定する。

### 判断基準

- 既存スキル活用 (優先度高)
- マイグレーション機能の充実度 (優先度高)
- FastAPI との統合性
- パフォーマンス特性
- ドキュメント・コミュニティの充実度
- 一人開発での保守性

### 選択肢

- SQLAlchemy + Alembic: Python の定番 ORM、Core/ORM 両対応、充実したマイグレーション機能
- SQLModel: FastAPI 作者が開発、Pydantic 統合、新しいが将来性高い
- Tortoise ORM + Aerich: async/await 完全対応、Django ORM ライクな設計
- Peewee + Peewee-migrate: 軽量シンプルな ORM、小規模プロジェクト向け
- Raw SQL + Databases: ORM なし、生 SQL アプローチ、最大限の制御

### ODM について

ODM (Object Document Mapping) は NoSQL データベース (MongoDB、CouchDB 等) 用のマッピング技術である。CogitoWeave では RDBMS 採用方針のため ODM は使用しないが、将来的なデータベース選択肢として MongoEngine、Motor、Beanie 等が存在する。

## 各選択肢の詳細分析

各選択肢のアプローチと特性を分析し、CogitoWeave のバックエンド要件への適合性を評価する。

### SQLAlchemy + Alembic

Python エコシステムで最も成熟した ORM。Core (低レベル) と ORM (高レベル) の両方の API を提供し、Alembic による強力なマイグレーション機能を持つ。

- 利点
  - 15 年以上の実績と安定性
  - 充実したマイグレーション機能 (自動生成、ロールバック、ブランチマージ)
  - FastAPI との良好な統合性
  - 豊富なドキュメントとコミュニティ
  - 複雑なクエリから単純な CRUD まで幅広くカバー
- 欠点
  - 学習曲線がやや急
  - async/await 対応が後付けで一部制約あり
  - 設定が複雑になる場合がある

### SQLModel

FastAPI 作者 Sebastian Ramirez が開発した新しい ORM。Pydantic と SQLAlchemy を統合し、型安全性を重視した設計。

- 利点
  - FastAPI との完全な統合 (同一作者)
  - Pydantic による型安全性
  - SQLAlchemy の実績を継承
  - 現代的な async/await 設計
  - シンプルで直感的な API
- 欠点
  - 比較的新しく情報が限定的
  - マイグレーション機能は Alembic 依存
  - コミュニティがまだ小さい
  - 複雑なクエリでは SQLAlchemy の知識が必要

### Tortoise ORM + Aerich

Django ORM にインスパイアされた async/await 完全対応の ORM。Aerich によるマイグレーション機能を提供。

- 利点
  - 完全な async/await 対応
  - Django ライクな直感的 API
  - 良好なパフォーマンス
  - 型ヒント対応
  - シンプルなマイグレーション
- 欠点
  - SQLAlchemy と比べて機能が限定的
  - 複雑なクエリ表現が困難
  - コミュニティが小さい
  - FastAPI との統合情報が限定的

### Peewee + Peewee-migrate

軽量でシンプルな ORM。小規模プロジェクト向けに設計され、最小限の設定で動作する。

- 利点
  - 軽量でシンプル
  - 学習曲線が緩やか
  - 一人開発に適したサイズ
  - 基本的なマイグレーション機能
- 欠点
  - async/await 対応が不完全
  - 複雑な要件には不向き
  - FastAPI との統合情報が少ない
  - マイグレーション機能が基本的

### Raw SQL + Databases

ORM を使わず生 SQL と async データベースドライバーを直接使用するアプローチ。

- 利点
  - 最大限のパフォーマンス
  - 完全な制御
  - 軽量な依存関係
  - 複雑なクエリも自由自在
- 欠点
  - マイグレーション機能は別途実装が必要
  - 開発効率が低い
  - SQL インジェクション等のセキュリティリスク
  - 保守性が低い

## 判断基準による評価

### 既存スキル活用の比較

SQLAlchemy が最も優秀である。Python 開発者にとって最も一般的な ORM であり、豊富な学習リソースと実務経験の蓄積がある。SQLModel は SQLAlchemy ベースのため知識が活用できる。

### マイグレーション機能の充実度の比較

SQLAlchemy + Alembic が最も優秀である。自動マイグレーション生成、ロールバック機能、ブランチマージ機能、データマイグレーション支援など、企業レベルの要件を満たす充実した機能を提供する。SQLModel も Alembic を使用するため同等の機能を持つ。

### FastAPI との統合性の比較

SQLModel が最も優秀である。同一作者による設計で完全な統合が保証されている。SQLAlchemy も FastAPI での使用実績が豊富で良好な統合性を持つ。

### パフォーマンス特性の比較

Raw SQL が最高だが、実用的な選択肢としては Tortoise ORM と SQLAlchemy Core が優秀である。async/await 対応により現代的な非同期処理が可能。

### ドキュメント・コミュニティの充実度の比較

SQLAlchemy が最も優秀である。15 年以上の歴史により膨大な情報蓄積とアクティブなコミュニティを持つ。SQLModel は新しいが FastAPI コミュニティの支持がある。

### 一人開発での保守性の比較

SQLModel と SQLAlchemy が優秀である。型安全性、明確な API、豊富な情報により長期保守が容易。Peewee もシンプルさで保守性が高い。

## 最終選択と根拠

### 選択結果

<!-- GLOBAL_CONCLUSION_BEGIN: orm-selection -->

ORM/ODM ツールとして SQLAlchemy + Alembic を採用する。

<!-- GLOBAL_CONCLUSION_END: orm-selection -->

### 選択根拠

既存スキルの活用、充実したマイグレーション機能、FastAPI との良好な統合性、15年以上の実績と安定性により、CogitoWeave のバックエンド要件を最適に満たす。SQLModel は将来性が高いが情報が限定的であり、Tortoise ORM は機能制限があり、Peewee は async/await 対応が不完全であり、Raw SQL はマイグレーションの自作が必要なため、SQLAlchemy + Alembic が最適選択となる。
