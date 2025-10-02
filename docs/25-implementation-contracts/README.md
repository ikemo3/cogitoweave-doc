---
doc_type: "unknown"
status: "draft"
depends:
  contracts: []
  produces: []
---

# 25-implementation-contracts - 実装契約系

## 概要

C4 モデルに基づき、システム実装における各レベルの契約を定義します。What 系で決定された設計判断を具体的な実装契約に落とし込み、設計の論理的根拠と実装の橋渡しを行います。

## ディレクトリ構成

### Container レベル契約

実行可能単位レベルの大きな構造契約を定義します。

- [フロントエンドパッケージ設計](01-frontend-container/README.md)
- [バックエンドモジュール設計](02-backend-container/README.md)
- [スキーマ設計](03-database-container/README.md)

### Component レベル契約

コンポーネント間の詳細な契約を定義します。

- [API設計仕様](04-api-specifications/README.md)
- [UIコンポーネント仕様](05-ui-components/README.md)
- [ドメインコンポーネント仕様](06-domain-components/README.md)
- [データアクセス仕様](07-data-access-components/README.md)

## 結論

C4 モデルの Container と Component レベルで実装契約を階層的に整理し、What 系の設計判断を具体的な実装仕様に変換します。
