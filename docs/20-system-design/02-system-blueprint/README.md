---
doc_type: "pattern-step"
status: "draft"
depends:
  contracts: []
  produces: []
---

# システム青写真 (How 系・理想)

## 目次

TBD

## 概要

CogitoWeave システムの How/What/How 軸における「How 系の理想」として、美しい理想的なアーキテクチャの観点から内部構造・アーキテクチャ設計判断を行っている。Kruchten 4+1 アーキテクチャビューモデルに基づく設計判断を行い、設計判断体系の一部として、システムの論理構造・実装アーキテクチャ・統合ワークフローを体系的に定義する。

## 処理フロー

TBD

## ステップ 1: Logical View

### Kruchten 4+1 アーキテクチャビューモデルについて

CogitoWeave システムの内部構造・アーキテクチャ設計判断を、Kruchten 4+1 アーキテクチャビューモデルに基づいて段階的に定義する。各ビューは異なる関心事・視点・時間軸から システムを分析し、包括的で整合性のあるアーキテクチャ設計を実現する。

<https://www.cs.ubc.ca/~gregor/teaching/papers/4%2B1view-architecture.pdf>

### Logical View

> The logical architecture primarily supports the functional requirements—what the system should provide in terms of services to its users. The system is decomposed into a set of key abstractions, taken (mostly) from the problem domain, in the form of objects or object classes. They exploit the principles of abstraction, encapsulation, and inheritance. This decomposition is not only for the sake of functional analysis, but also serves to identify common mechanisms and design elements across the various parts of the system. We use the Rational/Booch approach for representing the logical architecture, by means of class diagrams and class templates.
> A class diagram shows a set of classes and their logical relationships: association, usage, composition, inheritance, and so forth. Sets of related classes can be grouped into class categories. Class templates focus on each individual class; they emphasize the main class operations, and identify key object characteristics. If it is important to define the internal behavior of an object, this is done with state transition diagrams, or state charts. Common mechanisms or services are defined in class utilities.
> Alternatively to an OO approach, an application that is very data-driven may use some other form of logical view, such as E-R diagrams.

### Process View

> The process architecture takes into account some non-functional requirements, such as performance and availability. It addresses issues of concurrency and distribution, of system's integrity, of fault-tolerance, and how the main abstractions from the logical view fit within the process architecture—on which thread of control is an operation for an object actually executed.
> The process architecture can be described at several levels of abstraction, each level addressing different concerns. At the highest level, the process architecture can be viewed as a set of independently executing logical networks of communicating programs (called "processes"), distributed across a set of hardware resources connected by a LAN or a WAN. Multiple logical networks may exist simultaneously, sharing the same physical resources.
> A process is a grouping of tasks that form an executable unit. Processes represent the level at which the process architecture can be tactically controlled (i.e., started, recovered, reconfigured, and shut down). In addition, processes can be replicated for increased distribution of the processing load, or for improved availability.
> The software is partitioned into a set of independent tasks. A task is a separate thread of control, that can be scheduled individually on one processing node.
> We can distinguish then: major tasks, that are the architectural elements that can be uniquely addressed and minor tasks, that are additional tasks introduced locally for implementation reasons (cyclical activities, buffering, time-outs, etc.). They can be implemented as Ada tasks for example, or light-weight threads.
> Major tasks communicate via a set of well-defined inter-task communication mechanisms: synchronous and asynchronous message-based communication services, remote procedure calls, event broadcast, etc. Minor tasks may communicate by rendezvous or shared memory. Major tasks shall not make assumptions about their collocation in the same process or processing node.

### Development View

> The development architecture focuses on the actual software module organization on the software development environment. The software is packaged in small chunks —program libraries, or subsystems- that can be developed by one or a small number of developers. The subsystems are organized in a hierarchy of layers, each layer providing a narrow and well-defined interface to the layers above it.
> The development architecture of the system is represented by module and subsystem diagrams, showing the 'export' and 'import' relationships. The complete development architecture can only be described when all the elements of the software have been identified. It is, however, possible to list the rules that govern the development architecture: partitioning, grouping, visibility.

### Physical View

> The physical architecture takes into account primarily the non-functional requirements of the system such as availability, reliability (fault-tolerance), performance (throughput), and scalability. The software executes on a network of computers, or processing nodes (or just nodes for short). The various elements identified —networks, processes, tasks, and objects— need to be mapped onto the various nodes. We expect that several different physical configurations will be used: some for development and testing, others for the deployment of the system for various sites or for different customers. The mapping of the software to the nodes therefore needs to be highly flexible and have a minimal impact on the source code itself.

#### 重複項目の区別

以下の項目が複数の View に登場するため、View 間での役割の違いを明確にする：

**セキュリティ**：

- Development View の「セキュリティ（認証実装、脆弱性対策、セキュアコーディング）」
- Physical View の「ネットワーク/セキュリティ（ファイアウォール、VPN、証明書管理）」

**API**：

- Logical View の「API 契約（機能仕様、何をするか）」
- Development View の「API/スキーマ（技術仕様、どう実装するか）」

**監視/観測**：

- Process View の「観測性（ログ、メトリクス、トレース）」
- Physical View の「監視/運用（サーバリソース、ネットワーク、アラート）」

### Logical View の詳細

- **主語（誰の視点）**: ユーザー / 設計者
- **時間（いつの話）**: 設計時
- **関心（何を示す）**: 機能と責務の分割（何を）
- **代表アウトプット**: ドメイン/機能分割図、コンポーネント境界、API 契約
- **判定質問**
  - この機能はどのコンポーネントの責務か？
  - 境界越えの入力/出力は何か？
  - ユースケースを満たすために何と何が協調するか？

この Logical View に相当する設計判断は次のとおり。

<!-- REFERENCE_BEGIN: logical-view -->

- 主要抽象化設計
- 機能境界と責務
- UI 要素
- データモデル
- システムクラス・オブジェクトモデル設計
- ワークフロー
- API 契約（機能仕様、何をするか）
- 品質属性とトレードオフ

<!-- REFERENCE_END: logical-view -->

## ステップ 2: Process View

- **主語（誰の視点）**: 実行時のシステム
- **時間（いつの話）**: 実行時
- **関心（何を示す）**: 並行性・通信・スループット（どう動く）
- **代表アウトプット**: シーケンス図、メッセージフロー、スレッド/ワーカー/キュー設計
- **判定質問**
  - 実行時にどの順序で処理が進み、どこで並列になるか？
  - 失敗時の再試行や整合性確保はどうするか？
  - リクエストは何 Hop・何ミリ秒・どのキューを通るか？

この Process View に相当する設計判断は次のとおり。

<!-- REFERENCE_BEGIN: process-view -->

- 実行時フロー
- 一貫性/整合
- 並行性設計
- 同期メカニズム設計
- 信頼性と制御
- 性能設計
- 観測性（ログ、メトリクス、トレース）

<!-- REFERENCE_END: process-view -->

## ステップ 3: Development View

- **主語（誰の視点）**: 開発者 / 構成管理担当
- **時間（いつの話）**: 実装時
- **関心（何を示す）**: モジュール化・依存・技術選定（どう作る）
- **代表アウトプット**: パッケージ構成、言語/FW/ライブラリ選定、ビルド/テスト戦略
- **判定質問**
  - この機能はどの言語/フレームワーク/パッケージで実装するか？
  - どのレイヤやディレクトリに配置するか？
  - 依存関係や API の具体（型/インターフェイス）はどう定義するか？

この Development View に相当する設計判断は次のとおり(根拠については別途述べる)。

<!-- REFERENCE_BEGIN: development-view -->

- アーキテクチャ設計層
  - アーキテクチャスタイル
  - システム分割
  - 開発チーム構造
- インターフェース設計層
  - データアクセス
  - API 通信方式、認証認可
- 技術実装層
  - セキュリティ（認証実装、脆弱性対策、セキュアコーディング）
  - 言語、フレームワーク、ライブラリ、データベース
  - レイヤー構造
- 品質・運用層
  - テスト、品質
  - CI、CD、リポジトリ管理
- 記録層（並列）
  - ドキュメント、ADR

<!-- REFERENCE_END: development-view -->

## ステップ 4: Physical View

- **主語（誰の視点）**: インフラ / 運用担当
- **時間（いつの話）**: 配備 / 稼働時
- **関心（何を示す）**: 配置・冗長化・ネットワーク（どこで動く）
- **代表アウトプット**: デプロイ/ネットワーク図、ノード/AZ 構成、容量/スケール計画
- **判定質問**
  - どのノード/Pod/AZ/リージョンで動作させるか？
  - どのネットワーク経路を通すか？
  - 冗長化・フェイルオーバー・バックアップはどう構成するか？

この Physical View に相当する設計判断は次のとおり。

<!-- REFERENCE_BEGIN: physical-view -->

- 配置/インフラ
- ネットワーク/セキュリティ（ファイアウォール、VPN、証明書管理）
- 可用性/耐障害
- スケール/容量計画
- 監視/運用（サーバリソース、ネットワーク、アラート）
- コスト管理
- コンプライアンス

<!-- REFERENCE_END: physical-view -->
