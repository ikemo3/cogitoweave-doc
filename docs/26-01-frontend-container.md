<!-- markdownlint-disable MD024 -->

# Frontend

## フロントエンド Container 契約

### パッケージ構造

<!-- REFERENCE_BEGIN: frontend-package-structure -->

フロントエンドは単一 package.json によるモノリシック構成とし、機能別ディレクトリ構造により責務を分離する。

#### ディレクトリ構造

```plaintext
src/
├── features/
│   ├── concept/          # 概念管理機能
│   ├── literature-memo/  # 文献メモ管理機能
│   ├── relationship/     # 関係性管理機能
│   └── question/         # 質問探索機能
├── pages/                # 画面の組み立て
├── shared/
│   ├── components/       # 共通UIコンポーネント
│   └── api/              # API通信基盤
└── app/                  # アプリケーション全体の初期化・設定
```

#### 設計原則

- 機能ごとに凝集度を高め、機能追加・削除を容易にする
- 機能ディレクトリ内部でのレイヤー別分割を妨げない
- 画面は複数機能を組み合わせる場所として pages/ に独立配置する
- 横断的な関心事は責務が明確な名前で shared/ に配置する
- util/, common/, lib/ のような曖昧な名前は使用しない

<!-- REFERENCE_END: frontend-package-structure -->

### ルーター構成

<!-- REFERENCE_BEGIN: frontend-router-configuration -->

TanStack Router のファイルベースルーティングを採用し、ルート単位で遅延ロードを行う。

#### ルート定義配置

```plaintext
src/
├── routes/
│   ├── __root.tsx       # ルートレイアウト
│   ├── index.tsx        # メイン画面 (/)
│   ├── question.tsx     # 質問探索画面 (/question)
│   └── spatial.tsx      # 空間配置画面 (/spatial)
└── app/
    └── router.ts        # Router インスタンス生成
```

#### 設計原則

- ファイル構造とルートを一致させ、直感的な理解を実現する
- ルート単位で遅延ロードし、初期バンドルサイズを最小化する
- TanStack Router の規約に従い、設定を最小限に保つ
- ルーティングエラーは `__root.tsx` のエラー境界で捕捉する

<!-- REFERENCE_END: frontend-router-configuration -->

### テスト構成

<!-- REFERENCE_BEGIN: frontend-test-configuration -->

DOM 環境シミュレーションに happy-dom を採用し、ユニットテストはコロケーション、E2E テストは別フォルダ管理とする。

#### テスト環境

- DOM 環境シミュレーション: happy-dom
- ユニットテストフレームワーク: Vitest + Testing Library
- E2E テストフレームワーク: Playwright
- API モック: fetch のモック化(具体的手法は実装時に決定)

#### テストファイル配置

```plaintext
src/
├── features/
│   └── concept/
│       ├── ConceptList.tsx
│       └── ConceptList.test.tsx    # ユニットテスト(コロケーション)
├── pages/
│   ├── MainPage.tsx
│   └── MainPage.test.tsx           # ユニットテスト(コロケーション)
└── shared/
    └── components/
        ├── Button.tsx
        └── Button.test.tsx         # ユニットテスト(コロケーション)

tests/
└── e2e/
    ├── main-flow.spec.ts           # E2E テスト(別フォルダ)
    └── question-flow.spec.ts
```

#### 設計原則

- 軽量な DOM シミュレーションでテスト実行速度を最適化する
- ユニットテストはコード近傍に配置し、関連性を明確化する
- E2E テストは画面シナリオとして独立管理する
- API 通信はモック化し、外部依存を排除する

<!-- REFERENCE_END: frontend-test-configuration -->

### 状態管理

<!-- REFERENCE_BEGIN: frontend-state-management-specification -->

#### 状態管理ライブラリ

**Solid Signal のみを使用する**。追加の状態管理ライブラリは導入しない。

#### コンポーネント内部状態の実装

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

#### ページレベル状態の実装

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

#### アプリケーション全体状態の実装

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

#### サーバー状態管理の実装

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

#### キャッシュ戦略

- **基本方針**: ページ表示時に毎回 fetch(常に最新データを取得)
- **例外**: 編集中データのみページレベル Signal で保持し、離脱時に確認ダイアログを表示

#### localStorage 永続化の対象

**永続化するもの**:

- ダークモード設定(将来拡張)
- 画面表示設定(グラフ表示/リスト表示等)
- 最後に選択していた画面タブ

**永続化しないもの**:

- 選択中の概念 ID(ページリロード時はリセット)
- 編集中データ(明示的な保存のみ)

#### 実装原則

1. **Solid Signal のみ使用**: 追加ライブラリ不要
2. **単方向データフロー**: props による下方向、callback による上方向
3. **コンテナコンポーネントパターン**: データ取得は上位コンポーネントに集約
4. **シンプルさ優先**: 複雑なキャッシュ機構より理解しやすさを重視

<!-- REFERENCE_END: frontend-state-management-specification -->

### API 通信基盤

<!-- REFERENCE_BEGIN: frontend-api-client-specification -->

#### TypeScript 型生成

**openapi-typescript を使用して OpenAPI 仕様書から TypeScript 型定義を自動生成する**。

##### 実装手順

1. バックエンドの OpenAPI 仕様書をエクスポート(FastAPI 自動生成)
2. `openapi-typescript`で型定義を生成
3. フロントエンドで生成された型を import

```bash
# package.jsonのscriptsに追加
{
  "scripts": {
    "generate:api-types": "openapi-typescript http://localhost:8000/openapi.json -o src/shared/api/schema.ts"
  }
}
```

##### CI/CD での型チェック

```yaml
# .github/workflows/ci.yml
- name: Generate API types
  run: npm run generate:api-types
- name: Check for type changes
  run: |
    git diff --exit-code src/shared/api/schema.ts || \
    (echo "API types have changed. Please commit the updated schema.ts" && exit 1)
```

#### API クライアント実装

**fetch ベースの API クライアントをリトライ機構付きで実装する**。

```typescript
// shared/api/client.ts
import type { paths } from "./schema";

export class ApiError extends Error {
  constructor(
    public status: number,
    public statusText: string,
    public body?: unknown
  ) {
    super(`API Error: ${status} ${statusText}`);
    this.name = "ApiError";
  }
}

async function fetchWithRetry(
  url: string,
  options: RequestInit = {},
  maxRetries = 3
): Promise<Response> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 10000);

      const response = await fetch(url, {
        ...options,
        signal: controller.signal,
      });

      clearTimeout(timeoutId);

      // 成功 or 4xx系はリトライ不要
      if (response.ok || response.status < 500) {
        return response;
      }

      // 5xx系は次のループでリトライ
    } catch (error) {
      // 最後のリトライで失敗
      if (attempt === maxRetries - 1) {
        throw error;
      }

      // 指数バックオフ
      const delay = 1000 * Math.pow(2, attempt);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }

  throw new Error("Max retries exceeded");
}

export const apiClient = {
  async get<T>(path: string): Promise<T> {
    const response = await fetchWithRetry(path, {
      method: "GET",
      headers: { "Content-Type": "application/json" },
    });

    if (!response.ok) {
      const body = await response.json().catch(() => null);
      throw new ApiError(response.status, response.statusText, body);
    }

    return response.json();
  },

  async post<T>(path: string, data: unknown): Promise<T> {
    const response = await fetchWithRetry(path, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      const body = await response.json().catch(() => null);
      throw new ApiError(response.status, response.statusText, body);
    }

    return response.json();
  },
};
```

#### リトライ仕様

- **最大リトライ回数**: 3 回
- **リトライ間隔**: 指数バックオフ(1 秒 → 2 秒 → 4 秒)
- **タイムアウト**: 10 秒(統一)

**リトライ対象**:

- ネットワークエラー(接続失敗、タイムアウト)
- 5xx 系ステータスコード(サーバーエラー)

**リトライしないもの**:

- 4xx 系ステータスコード(クライアントエラー)

#### エラーハンドリング

##### エラー型定義

```typescript
// shared/api/errors.ts
export type ErrorDisplay = {
  type: "inline" | "toast" | "page";
  title: string; // 「保存に失敗しました」
  message: string; // 「概念名が重複しています」
  action?: string; // 「名前を変更してください」(対応手順)
  technicalDetail?: string; // 「409 Conflict」(開発者向け)
};

export function formatApiError(error: ApiError): ErrorDisplay {
  // 4xx系: インライン表示
  if (error.status >= 400 && error.status < 500) {
    return {
      type: "inline",
      title: "入力エラー",
      message: error.body?.error || error.statusText,
      action: error.body?.details?.[0],
      technicalDetail: `${error.status} ${error.statusText}`,
    };
  }

  // 5xx系: トースト通知
  if (error.status >= 500) {
    return {
      type: "toast",
      title: "サーバーエラー",
      message: "しばらく待ってから再度お試しください",
      technicalDetail: `${error.status} ${error.statusText}`,
    };
  }

  // ネットワークエラー: トースト通知
  return {
    type: "toast",
    title: "通信エラー",
    message: "ネットワーク接続を確認してください",
    technicalDetail: error.message,
  };
}
```

##### UI 表示方法

| エラー種別                         | 表示方法         | 例                                          |
| ---------------------------------- | ---------------- | ------------------------------------------- |
| 入力エラー(4xx 系)                 | インライン表示   | フォーム下に赤文字「概念名は必須です」      |
| 一時的エラー(5xx 系、ネットワーク) | トースト通知     | 画面右上に「サーバーエラー。リトライ中...」 |
| 致命的エラー                       | エラーページ遷移 | エラー境界でキャッチ → 専用エラー画面       |

##### トースト実装

```typescript
// shared/components/Toast.tsx
import { toast } from "solid-toast";

export function showErrorToast(error: ErrorDisplay) {
  if (error.type !== "toast") return;

  toast.error(error.message, {
    duration: 10000, // エラーは10秒表示
    position: "top-right",
  });
}

export function showSuccessToast(message: string) {
  toast.success(message, {
    duration: 5000, // 成功は5秒表示
    position: "top-right",
  });
}
```

#### 実装原則

1. **型安全性**: OpenAPI 仕様書から自動生成した型を使用
2. **リトライ戦略**: ネットワークエラーと 5xx 系のみリトライ、指数バックオフ
3. **統一タイムアウト**: 全 API 10 秒
4. **明確なエラー伝達**: エラー種別に応じた適切な UI 表示
5. **シンプルさ優先**: 複雑なキャッシュ機構やインターセプター不要

<!-- REFERENCE_END: frontend-api-client-specification -->
