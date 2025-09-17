---
doc_type: "unknown"
status: "draft"
depends:
  contracts: []
  produces: []
---

# 設計書体系ディレクトリ構造

## 概要

構造化 4 軸設計フレームワーク(Why/What/Contract/How)の実際のディレクトリ構成と、各ファイル間の依存関係を定義します。この文書は [design-structure.md](design-structure.md) で説明された論理的根拠を、具体的なファイル配置として実装するためのガイドです。

## 構造と依存関係

### 1. 全体構造：軸間の依存関係

```mermaid
flowchart TD
    Why[10-vision-reality<br/>Why系]
    What[20-system-design<br/>What系]
    Contract[25-implementation-contracts<br/>契約系]
    How[30-delivery-methods<br/>How系]

    Why --> What
    What --> Contract
    What --> How
    Contract --> How

    How -.フィードバック.-> What
    What -.フィードバック.-> Why
    Contract -.設計フィードバック.-> What
```

### 2. Why 系内部：理想・現実制約・統合解

```mermaid
flowchart TD
    subgraph "10-vision-reality"
        A[01-core-vision<br/>コアビジョン]
        B[02-organizational-capacity<br/>組織能力]
        C[03-external-environment<br/>外部環境分析]
        D[04-system-properties<br/>システム制約]

        A --> B
        A --> C
        A --> D
        B --> C
        B --> D
        C --> D
    end
```

### 3. What 系内部：ユーザー・システム軸の正反合

```mermaid
flowchart TD
    subgraph "20-system-design"
        D[01-user-desires<br/>主体：ユーザーの欲求]
        E[02-system-blueprint<br/>主体：美しいアーキテクチャ]
        F[03-system-constraints<br/>客体：アーキテクチャ制約]
        G[04-user-constraints<br/>客体：技術制約込み仕様]
        H[05-design-synthesis<br/>統合：最終設計]

        D -.機能要件の主体vs客体.- G
        E -.アーキテクチャの主体vs客体.- F
        D --> E
        F --> G
        D --> H
        E --> H
        F --> H
        G --> H
    end
```

### 4. 契約系内部：Design by Contract

```mermaid
flowchart TD
    subgraph "25-implementation-contracts"
        I[01-interface-contracts<br/>API・プロトコル契約]
        J[02-security-contracts<br/>認証・認可契約]
        K[03-performance-contracts<br/>性能・SLA契約]
        L[04-data-contracts<br/>データ形式・整合性契約]
        M[05-unified-contracts<br/>統合：実装契約書]

        I --> M
        J --> M
        K --> M
        L --> M
    end
```

### 5. How 系内部：複数パターン

#### パターン A: QCDS 分割モデル(個人開発向け)

```mermaid
flowchart TD
    subgraph "30-delivery-methods-qcds"
        N[01-quality-cost-ideals<br/>主体：品質・コスト理想]
        O[02-delivery-scope-limits<br/>客体：納期・スコープ制約]
        P[03-qcds-synthesis<br/>統合：実現可能なプロセス]

        N -.QCDS主体vs客体.- O
        N --> P
        O --> P
    end
```

#### パターン B: People/Platform モデル(チーム開発向け)

```mermaid
flowchart TD
    subgraph "30-delivery-methods-people-platform"
        Q[01-craft-driven-flow<br/>主体：理想的なチーム・技能・文化]
        R[02-pipeline-blueprint<br/>主体：理想的な自動化・ツールチェーン]
        S[03-people-limits<br/>客体：スキルギャップ・組織制約]
        T[04-platform-limits<br/>客体：レガシー・インフラ・予算制約]
        U[05-adaptive-value-stream<br/>統合：制約最適化プロセス]

        Q -.人間系主体vs客体.- S
        R -.機械系主体vs客体.- T
        Q --> U
        R --> U
        S --> U
        T --> U
    end
```

#### パターン C: Learning/Delivery モデル(DevOps 向け)

```mermaid
flowchart TD
    subgraph "30-delivery-methods-learning-delivery"
        V[01-continuous-learning<br/>主体：仮説検証・実験・改善]
        W[02-continuous-delivery<br/>主体：リードタイム最短フロー]
        X[03-risk-guardrails<br/>客体：品質・安全・法令制約]
        Y[04-resource-caps<br/>客体：工数・コスト・設備上限]
        Z[05-resilient-process<br/>統合：学習+デリバリ最適化]

        V -.学習系主体vs客体.- X
        W -.デリバリ系主体vs客体.- Y
        V --> Z
        W --> Z
        X --> Z
        Y --> Z
    end
```

#### パターン D: PDCA 二段モデル(改善重視組織向け)

```mermaid
flowchart TD
    subgraph "30-delivery-methods-pdca"
        AA[01-ideal-plan<br/>主体：理想的計画]
        BB[02-ideal-execution<br/>主体：理想的実行]
        CC[03-plan-constraints<br/>客体：計画制約]
        DD[04-execution-constraints<br/>客体：実行制約]
        EE[05-feasible-pd<br/>統合：実現可能なPlan-Do]

        FF[06-check-act-metrics<br/>横串：QCDS・DORA・SLI監視層]

        AA -.計画系主体vs客体.- CC
        BB -.実行系主体vs客体.- DD
        AA --> EE
        BB --> EE
        CC --> EE
        DD --> EE

        EE --> FF
        FF -.改善フィードバック.- AA
        FF -.改善フィードバック.- BB
    end
```

## Why 系における設計判断 - なぜ作るのか

Why 系では「なぜ作るのか」という根本的な問いに対して、4 つの観点で答えます。

### 01-core-vision：理想 - ビジョン&スコープ

プロダクトビジョン・設計原則・スコープを定義します。Why の理想像は「複数のシナリオ」として表現できます。インクリメンタルに進める場合は、1 つ目のシナリオを end-to-end で実現することが重要です。ただし、全シナリオを包含する骨格(What)は事前に定義し、2 つ目以降のシナリオで整合性が崩れることを防ぎます。

### 02-organizational-capacity：組織能力

開発体制・技術選択・品質基準等の組織の技術的能力、予算・時間・設備等のリソース制約と活用方針、競合・差別化・価値提案等の市場での立ち位置を定義します。これらは組織の特性に応じた直交分割により並列分析されます。

### 03-external-environment：外部環境分析

PEST 分析(Political・Economic・Social・Technological)に基づく外部環境要因を定義します。政治的要因、経済的要因、社会的要因、技術的要因を分析し、組織が直接コントロールできない外部環境がシステム設計に与える影響を評価します。

### 04-system-properties：システム制約

ISO/IEC 25010 システム及びソフトウェア品質モデルに基づく品質制約・非機能要件を定義します。機能適合性、性能効率性、互換性、ユーザビリティ、信頼性、セキュリティ、保守性、移植性の 8 つの品質特性を網羅的に扱います。

## What 系における設計判断 - 何を作るのか

What 系では「何を作るのか」という問いに対して、[20-system-design](../20-system-design/)でユーザーが本当に欲しいものの理想を描き、技術制約込みの実現可能な仕様という現実を扱います。

### 20-system-design：What 系 - システム設計判断

システム設計は Kruchten 4+1 アーキテクチャビューモデルに基づいて構成されています：

- [01-user-desires](../20-system-design/01-user-desires/) - Scenario View(+1)：ユーザーシナリオと要求
- [02-system-blueprint](../20-system-design/02-system-blueprint/) - 4 つのアーキテクチャビュー：
  - Logical View：論理構造とコンポーネント
  - Process View：プロセスと並行性
  - Development View：開発とモジュール構成
  - Physical View：物理配置と環境
- [03-system-constraints](../20-system-design/03-system-constraints/) - アーキテクチャ制約条件
- [04-user-constraints](../20-system-design/04-user-constraints/) - 技術制約込み実現可能仕様

## How 系における設計判断 - どう作るのか

How 系では「どう作るのか」という問いに対して、[25-implementation-contracts](../25-implementation-contracts/)で契約系を、[30-delivery-methods](../30-delivery-methods/)で実装・運用方法を扱います。

### 25-implementation-contracts：契約系 - Design by Contract

C4 モデルによる実装契約の定義では、Context(システム境界と外部システムとの関係)から Container(アプリケーション内部の構成要素間契約)、Component(コンテナ内部のコンポーネント間契約)、Code(クラス・インターフェース・データ契約)まで、階層的に契約を定義します。

TBD - 現在構築中

### 30-delivery-methods：How 系 - 実装・運用方法

実装・運用方法ではアジャイル開発の正しい理解が重要です。アジャイルは What の曖昧化ではなく、Why の変化に応じるための仕組みであることを明確にします。「要件を後から決める」や「仕様を曖昧にしておく」という誤解を避け、Why の変化に対応する柔軟性を持ちながらも、What は整合性のため最初に骨格を定義し、How は継続的に改善するという適切な運用を行います。

TBD - 現在構築中

## ディレクトリ構成の運用ガイド

### ファイル作成時のルール

1. **1 ファイル 1 判断原則**：各ファイルは単一の設計判断のみを扱う
2. **番号順序**：依存関係に基づいた論理的順序で番号を付与
3. **マーカー活用**：ファイル間の情報継承にはマーカーを使用

### 依存関係の管理

- **前方依存**：後続番号のファイルは前番号のファイルに依存可能
- **循環依存禁止**：同一軸内での循環依存は避ける
- **軸間依存**：Why → What → Contract → How の順序を守る

### 変更時の影響範囲

設計変更時は、依存関係図を参照して影響範囲を特定し、関連ファイルの整合性を確認する必要があります。特に統合ファイル(05-design-synthesis, 05-unified-contracts 等)は複数の前提ファイルに依存するため、変更時は注意深い検証が必要です。

## 結論

この実装構造により、論理的根拠に基づいた具体的なディレクトリ配置が実現され、設計書体系の保守性と拡張性が確保されます。
