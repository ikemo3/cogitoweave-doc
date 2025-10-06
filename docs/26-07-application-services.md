<!-- markdownlint-disable MD024 -->

# Application

## Application サービスコンポーネント契約

### 概念サービス仕様

<!-- REFERENCE_BEGIN: concept-service-specification -->

概念サービス仕様として以下を定義する。

#### 概念サービスの責務

- 概念に関するユースケース実装を行う。
- トランザクション制御を担当する。
- 概念 Repository を使用したデータアクセスを行う。
- エラーハンドリングと例外処理を行う。

#### 概念サービスのインターフェース

- `create_concept(name: str, description: str) -> Concept`: 概念を作成する
- `get_concept(concept_id: int) -> Concept`: ID で概念を取得する
- `list_concepts() -> list[Concept]`: 全概念を取得する
- `update_concept(concept_id: int, name: str | None, description: str | None) -> Concept`: 概念を更新する
- `search_concepts(query: str) -> list[Concept]`: 名前・説明で検索する

#### 概念サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 概念 Repository への依存のみを持つ。
- トランザクション境界を明確に管理する。

<!-- REFERENCE_END: concept-service-specification -->

### 文献メモサービス仕様

<!-- REFERENCE_BEGIN: literature-memo-service-specification -->

文献メモサービス仕様として以下を定義する。

#### 文献メモサービスの責務

- 文献メモに関するユースケース実装を行う。
- 外部キー制約の整合性確認を行う。
- トランザクション制御を担当する。
- 文献メモ Repository を使用したデータアクセスを行う。
- エラーハンドリングと例外処理を行う。

#### 文献メモサービスのインターフェース

- `create_literature_memo(concept_id: int, literature_id: int, title: str, content: str) -> LiteratureMemo`: 文献メモを作成する
- `get_literature_memo(literature_memo_id: int) -> LiteratureMemo`: ID で文献メモを取得する
- `list_literature_memos_by_concept(concept_id: int) -> list[LiteratureMemo]`: 概念別文献メモ一覧を取得する
- `update_literature_memo(literature_memo_id: int, title: str | None, content: str | None) -> LiteratureMemo`: 文献メモを更新する

#### 文献メモサービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 文献メモ Repository への依存のみを持つ。
- 外部キー制約の整合性を Application 層で確認する。
- トランザクション境界を明確に管理する。

<!-- REFERENCE_END: literature-memo-service-specification -->

### 関係性サービス仕様

<!-- REFERENCE_BEGIN: relationship-service-specification -->

関係性サービス仕様として以下を定義する。

#### 関係性サービスの責務

- 関係性に関するユースケース実装を行う。
- 無向グラフの正規化制約を維持する。
- トランザクション制御を担当する。
- 関係性 Repository を使用したデータアクセスを行う。
- エラーハンドリングと例外処理を行う。

#### 関係性サービスのインターフェース

- `create_relationship(concept_a_id: int, concept_b_id: int, relationship_name: str, description: str) -> Relationship`: 関係性を作成する
- `get_relationship(relationship_id: int) -> Relationship`: ID で関係性を取得する
- `list_relationships_by_concept(concept_id: int) -> list[Relationship]`: 概念別関係性一覧を取得する
- `update_relationship(relationship_id: int, relationship_name: str | None, description: str | None) -> Relationship`: 関係性を更新する

#### 関係性サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 関係性 Repository への依存のみを持つ。
- 無向グラフの正規化制約を Application 層で維持する。
- トランザクション境界を明確に管理する。

<!-- REFERENCE_END: relationship-service-specification -->

### 文献サービス仕様

<!-- REFERENCE_BEGIN: literature-service-specification -->

文献サービス仕様として以下を定義する。

#### 文献サービスの責務

- 文献に関するユースケース実装を行う。
- トランザクション制御を担当する。
- 文献 Repository を使用したデータアクセスを行う。
- エラーハンドリングと例外処理を行う。

#### 文献サービスのインターフェース

- `create_web_article(url: str, title: str) -> Literature`: Web 記事を作成する
- `create_book(isbn: str, author: str, publisher: str, publication_year: int) -> Literature`: 書籍を作成する
- `get_literature(literature_id: int) -> Literature`: ID で文献を取得する

#### 文献サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 文献 Repository への依存のみを持つ。
- 作成と読み取りのみを提供する。
- トランザクション境界を明確に管理する。

<!-- REFERENCE_END: literature-service-specification -->

### 概念検索サービス仕様

<!-- REFERENCE_BEGIN: concept-search-service-specification -->

概念検索サービス仕様として以下を定義する。

#### 概念検索サービスの責務

- 概念検索に関するユースケース実装を行う。
- 検索クエリのバリデーションを行う。
- 概念 Repository を使用した検索を実行する。
- エラーハンドリングと例外処理を行う。

#### 概念検索サービスのインターフェース

- `search_concepts(query: str) -> list[Concept]`: 名前・説明で概念を検索する

#### 概念検索サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 概念 Repository への依存のみを持つ。
- 検索クエリのバリデーションを Application 層で行う。

<!-- REFERENCE_END: concept-search-service-specification -->

### 質問から概念候補提示サービス仕様

<!-- REFERENCE_BEGIN: question-to-concepts-service-specification -->

質問から概念候補提示サービス仕様として以下を定義する。

#### 質問から概念候補提示サービスの責務

- 質問内容から概念候補を提示するユースケース実装を行う。
- 概念 Repository を使用して全概念を取得する。
- LLM Repository を使用して質問内容を分析する。
- LLM の応答を解析して概念候補リストを生成する。
- エラーハンドリングと例外処理を行う。

#### 質問から概念候補提示サービスのインターフェース

- `suggest_concepts_from_question(question: str) -> list[ConceptSuggestion]`: 質問から概念候補を提示する

#### 質問から概念候補提示サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 概念 Repository と LLM Repository への依存を持つ。
- LLM API の障害に対するエラーハンドリングを実装する。

<!-- REFERENCE_END: question-to-concepts-service-specification -->

### LLM 要約生成サービス仕様

<!-- REFERENCE_BEGIN: llm-summary-service-specification -->

LLM 要約生成サービス仕様として以下を定義する。

#### LLM 要約生成サービスの責務

- 概念の観点で記事を要約するユースケース実装を行う。
- 概念 Repository を使用して概念情報を取得する。
- LLM Repository を使用して要約を生成する。
- エラーハンドリングと例外処理を行う。

#### LLM 要約生成サービスのインターフェース

- `generate_summary(concept_id: int, article_title: str, article_content: str) -> Summary`: 概念の観点で記事を要約する

#### LLM 要約生成サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 概念 Repository と LLM Repository への依存を持つ。
- LLM API の障害に対するエラーハンドリングを実装する。

<!-- REFERENCE_END: llm-summary-service-specification -->

### 記事取得サービス仕様

<!-- REFERENCE_BEGIN: article-fetch-service-specification -->

記事取得サービス仕様として以下を定義する。

#### 記事取得サービスの責務

- 指定 URL から記事内容を取得するユースケース実装を行う。
- URL のバリデーションと正規化を行う。
- 記事取得 Repository を使用して記事内容を取得する。
- エラーハンドリングと例外処理を行う。

#### 記事取得サービスのインターフェース

- `fetch_article(url: str) -> Article`: 指定 URL から記事を取得する

#### 記事取得サービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- 記事取得 Repository への依存のみを持つ。
- URL のバリデーションを Application 層で行う。
- 記事取得サービスの障害に対するエラーハンドリングを実装する。

<!-- REFERENCE_END: article-fetch-service-specification -->

### バックアップサービス仕様

<!-- REFERENCE_BEGIN: backup-service-specification -->

バックアップサービス仕様として以下を定義する。

#### バックアップサービスの責務

- データベースのバックアップを実行するユースケース実装を行う。
- バックアップファイルのパス生成とファイル出力を行う。
- バックアップの検証を行う。
- トランザクション制御を担当する。
- エラーハンドリングと例外処理を行う。

#### バックアップサービスのインターフェース

- `execute_backup() -> BackupResult`: バックアップを実行する
- `verify_backup(backup_path: str) -> VerifyResult`: バックアップを検証する

#### バックアップサービスの実装方針

- ステートレス実装を行う。
- リクエストスコープのライフサイクルを持つ。
- バックアップ Repository への依存のみを持つ。
- 読み取り専用トランザクションを使用してデータの一貫性を保証する。
- バックアップファイル名にタイムスタンプを含める。

<!-- REFERENCE_END: backup-service-specification -->
