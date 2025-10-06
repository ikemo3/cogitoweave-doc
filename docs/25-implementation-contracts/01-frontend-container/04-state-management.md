---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
    - "state-management-responsibilities"
  produces:
    - "frontend-state-management-specification"
---

# 状態管理仕様

## 概要

フロントエンドの状態管理の具体的な実装方針を定義する。3 層状態管理(コンポーネント/ページ/アプリケーション)を Solid Signal で実現し、軽量で理解しやすい状態管理を提供する。

## 前提制約

<!-- PREMISE_BEGIN: frontend-container-integrated-constraints -->

### 物理配置

- セルフホスト環境の Docker コンテナとして静的配信

### 技術スタック

- フレームワーク: Solid + TanStack Router
- スタイリング: Tailwind CSS + CSS Modules
- グラフ可視化: D3
- ビルドツール: Vite
- テストフレームワーク: Vitest + Testing Library + Playwright

### アーキテクチャ方針

- CSR(Client-Side Rendering)採用
- SPA(Single Page Application)構造
- 軽量コンポーネント指向アーキテクチャ
- 3 層状態管理(コンポーネント/ページ/アプリケーション)
- コンテナコンポーネントパターンと単方向データフロー

### 画面構成

- メイン画面: 概念管理、文献メモ管理、関係性管理を統合
- 質問探索画面: 質問から概念発見とメモ参照を独立提供
- 空間配置画面: 概念の空間配置と関係性構築を専用提供
- タブ切り替えによる画面間移動

### パフォーマンス

- システム初回起動パフォーマンスバジェット対応
- 概念ページ読み込みパフォーマンスバジェット対応
- コード分割とバンドル最適化
- 軽量再レンダリング機構
- 必要時のグラフ描画最適化

### レスポンシブ対応

- Mac 環境: 情報整理機能最適化
- iPhone 環境: 情報参照機能最適化

### エラーハンドリング

- エラー境界コンポーネント
- ルーティングエラーハンドリング
- location.reload による強制リロード

### 監視・トレーシング

- Sentry による APM・エラートラッキング
- クライアントサイド監視・トレーシング
- 障害検知

### サービス間通信

- バックエンドへの依存(一方向)
- HTTP クライアントによる直接呼び出し
- データ統合責務

### 不足技術要素

- DOM 環境シミュレーション(jsdom/happy-dom)
- TanStack Router のテスト環境構築手法
- API モック化手法
- TypeScript 型生成(openapi-typescript)
- API 仕様変更の自動検知
- メモリ使用量監視(Chrome DevTools Memory)
- リトライ機構の実装パターン

<!-- PREMISE_END: frontend-container-integrated-constraints -->

<!-- PREMISE_BEGIN: state-management-responsibilities -->

状態管理は 3 層に分離して責務を明確化する。

- **コンポーネント内部状態**: 表示状態・入力値・開閉状態等の局所的な制御
  - コンポーネント固有の UI 状態管理に限定する
- **ページレベル状態**: 選択中概念・検索フィルタ・編集中データ等の複数コンポーネント間共有
  - 状態管理ライブラリを使用してページ内グローバル状態を管理する
- **アプリケーション全体状態**: ユーザー設定・表示設定・大きなコンポーネント間同期データ
  - 永続化データはブラウザの localStorage を使用する
  - ページ境界を越えた大きなコンポーネント間の同期はクライアント側状態管理で対応する

<!-- PREMISE_END: state-management-responsibilities -->

## 論理

### 状態管理ライブラリ選択の論理

前提制約から、「軽量コンポーネント指向アーキテクチャ」「最小限状態管理」を実現する必要がある。Solid は既に Signal ベースの高性能リアクティブシステムを内蔵しており、追加の状態管理ライブラリは以下の理由により不要である。

1. **Signal の十分性**: `createSignal()`はコンポーネント/ページ/アプリケーション全層で使用可能
2. **軽量性の維持**: 追加ライブラリ不要でバンドルサイズ最小化
3. **学習コストの削減**: Solid 標準 API のみで一貫した実装が可能
4. **過剰機能の回避**: TanStack Query のキャッシュ戦略・無効化・再取得等の複雑性は一人用システムでは不要

したがって、状態管理ライブラリとして Solid Signal のみを採用する。

### サーバー状態管理の論理

API から取得したデータ(概念リスト、文献メモ等)の管理について、以下の理由により単純な fetch + Signal パターンで十分である。

1. **一人用システム**: 同時編集やキャッシュ無効化の複雑性が不要
2. **データ規模**: 数万件規模でも fetch は十分高速
3. **理解しやすさ**: 複雑なキャッシュ機構よりシンプルな実装を優先

基本方針として、ページ表示時に毎回 fetch して常に最新データを取得する。例外として、編集中データのみページレベル Signal で保持し、離脱時に確認ダイアログを表示する。

### localStorage 永続化の論理

アプリケーション全体状態のうち、永続化が必要なものと不要なものを明確に区別する必要がある。

**永続化対象**:

- アプリケーション設定(ダークモード、画面表示設定等)
- 最後に選択していた画面タブ

**永続化しないもの**:

- 選択中の概念 ID(ページリロード時はリセットして混乱を防ぐ)
- 編集中データ(明示的な保存のみ許可してデータ整合性を保つ)

この区別により、永続化によるバグ(古いデータの残存)を防止し、ユーザーの意図しない動作を回避する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-state-management-specification -->

### 状態管理ライブラリ

**Solid Signal のみを使用する**。追加の状態管理ライブラリは導入しない。

### コンポーネント内部状態の実装

`createSignal()`で局所状態を管理する。

```typescript
// 例: モーダルの開閉状態
function ConceptModal() {
  const [isOpen, setIsOpen] = createSignal(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>開く</button>
      <Show when={isOpen()}>
        <div>モーダル内容</div>
      </Show>
    </>
  );
}
```

### ページレベル状態の実装

ページコンポーネントで`createSignal()`を定義し、props で子コンポーネントに配布する。

```typescript
// 例: 選択中の概念ID
function MainPage() {
  const [selectedConceptId, setSelectedConceptId] = createSignal<number | null>(
    null
  );

  return (
    <>
      <ConceptList
        selectedId={selectedConceptId()}
        onSelect={setSelectedConceptId}
      />
      <ConceptDetail conceptId={selectedConceptId()} />
    </>
  );
}
```

### アプリケーション全体状態の実装

`createSignal()` + localStorage 永続化のカスタムフックで実装する。

```typescript
// shared/hooks/usePersistedSignal.ts
export function usePersistedSignal<T>(key: string, initialValue: T) {
  const stored = localStorage.getItem(key);
  const [value, setValue] = createSignal<T>(
    stored ? JSON.parse(stored) : initialValue
  );

  const setPersistedValue = (newValue: T) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };

  return [value, setPersistedValue] as const;
}

// 使用例
function App() {
  const [theme, setTheme] = usePersistedSignal("app:theme", "light");
  const [lastTab, setLastTab] = usePersistedSignal("app:lastTab", "main");

  return <div data-theme={theme()}>...</div>;
}
```

### サーバー状態管理の実装

fetch + Signal + 単純キャッシュパターンで実装する。

```typescript
// 基本パターン
function useConcepts() {
  const [concepts, setConcepts] = createSignal<Concept[]>([]);
  const [loading, setLoading] = createSignal(false);
  const [error, setError] = createSignal<Error | null>(null);

  async function fetchConcepts() {
    setLoading(true);
    setError(null);
    try {
      const data = await apiClient.get("/api/concepts");
      setConcepts(data);
    } catch (e) {
      setError(e as Error);
    } finally {
      setLoading(false);
    }
  }

  return { concepts, loading, error, fetchConcepts };
}
```

### キャッシュ戦略

- **基本方針**: ページ表示時に毎回 fetch(常に最新データを取得)
- **例外**: 編集中データのみページレベル Signal で保持し、離脱時に確認ダイアログを表示

### localStorage 永続化の対象

**永続化するもの**:

- ダークモード設定(将来拡張)
- 画面表示設定(グラフ表示/リスト表示等)
- 最後に選択していた画面タブ

**永続化しないもの**:

- 選択中の概念 ID(ページリロード時はリセット)
- 編集中データ(明示的な保存のみ)

### 実装原則

1. **Solid Signal のみ使用**: 追加ライブラリ不要
2. **単方向データフロー**: props による下方向、callback による上方向
3. **コンテナコンポーネントパターン**: データ取得は上位コンポーネントに集約
4. **シンプルさ優先**: 複雑なキャッシュ機構より理解しやすさを重視

<!-- GLOBAL_CONCLUSION_END: frontend-state-management-specification -->
