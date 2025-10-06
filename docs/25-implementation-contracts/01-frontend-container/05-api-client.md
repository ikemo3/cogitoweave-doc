---
doc_type: "pattern-logic"
status: "draft"
depends:
  contracts:
    - "frontend-container-integrated-constraints"
    - "distributed-communication-design"
    - "request-processing-design"
  produces:
    - "frontend-api-client-specification"
---

# API 通信基盤仕様

## 概要

フロントエンドの HTTP API 通信基盤の具体的な実装方針を定義する。TypeScript 型生成、リトライ機構、エラーハンドリングを統一的に実装し、バックエンドとの安全で効率的な通信を実現する。

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

<!-- PREMISE_BEGIN: distributed-communication-design -->

- **HTTP API 直接通信**: フロントエンドから 3 サービスへの直接 HTTP/HTTPS 通信を行う。
- **JSON データフォーマット**: GET/POST による CQS ベースの API 設計で保守しやすい通信を実現する。
- **サービス特性ごとのタイムアウト設定**: 各サービスの処理特性に応じた段階的設定で応答時間管理を行う。
- **HTTP ステータスコードエラーハンドリング**: 4xx 系でクライアントエラー、5xx 系でサーバーエラーを表現し、サービス特性ごとのリトライ戦略により障害処理を行う。
- **最小限セキュリティ設計**: HTTPS 通信と CORS 設定による基本セキュリティを確保し、認証認可機能は除外する。

<!-- PREMISE_END: distributed-communication-design -->

<!-- PREMISE_BEGIN: request-processing-design -->

- **同期処理ベース**: 基本的に同期処理でシンプルな処理フローを実現し、複雑な業務処理は 1 トランザクション内で完結させる。
- **ログレベル設定**: 開発者対応の緊急度に応じたログレベル（ERROR=即座対応、WARN=計画的改善、INFO=動作確認、DEBUG=開発詳細）を採用し、ユーザーには理解しやすいエラーメッセージのみを提供する。
- **トランザクション分離**: データベース操作のみを対象とし、外部 API 呼び出しは分離する。
- **リクエスト検証**: 型チェック、範囲チェック、形式チェックを含む定型的な検証を実施し、ビジネスロジック検証は別レイヤーで処理する。
- **統一タイムアウト**: 全 API に 10 秒の統一タイムアウトを設定し、外部 API エラー時はフロントエンド側リトライで対応する。
- **性能最適化**: N+1 問題対策とクエリ最適化により性能要件を満たす。

<!-- PREMISE_END: request-processing-design -->

## 論理

### TypeScript 型生成の論理

バックエンドの API 仕様とフロントエンドの TypeScript 型定義の不整合を防ぐため、OpenAPI 仕様書からの自動型生成が必要である。FastAPI は自動的に OpenAPI 仕様書を生成するため、これを活用する。

1. **型安全性**: バックエンドの API 変更を型エラーとして検出
2. **自動化**: CI/CD で型生成を実行し、差分があれば PR でエラー
3. **保守性**: 手動での型定義メンテナンスを排除

### リトライ機構の論理

統一タイムアウト 10 秒と「外部 API エラー時はフロントエンド側リトライで対応する」という前提から、リトライ対象とリトライ戦略を決定する必要がある。

**リトライ対象**:

- ネットワークエラー(接続失敗、タイムアウト): リトライで復旧する可能性が高い
- 5xx 系ステータスコード(サーバーエラー): 一時的な障害の可能性がある

**リトライしないもの**:

- 4xx 系ステータスコード(クライアントエラー): リトライしても結果は変わらない
- 認証エラー(将来拡張時): リトライすべきでない

**リトライ戦略**:

- 最大リトライ回数: 3 回(過剰なリトライによるサーバー負荷を回避)
- リトライ間隔: 指数バックオフ(1 秒 → 2 秒 → 4 秒)(サーバー復旧時間を確保)
- タイムアウト: 10 秒(統一)

### エラーハンドリング UI の論理

「明確なエラー伝達: 障害種別・影響範囲・対応手順を分かりやすく表示」という前提から、エラー種別による UI 表示方法を分類する必要がある。

**エラー種別と表示方法**:

- 入力エラー(4xx 系): インライン表示(フォーム近傍で即座にフィードバック)
- 一時的エラー(5xx 系、ネットワーク): トースト通知(画面右上で進行状況を通知)
- 致命的エラー: エラーページ遷移(エラー境界でキャッチして専用画面)

この分類により、ユーザーの文脈に応じた適切なエラー表示を実現する。

## 結論

<!-- GLOBAL_CONCLUSION_BEGIN: frontend-api-client-specification -->

### TypeScript 型生成

**openapi-typescript を使用して OpenAPI 仕様書から TypeScript 型定義を自動生成する**。

#### 実装手順

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

#### CI/CD での型チェック

```yaml
# .github/workflows/ci.yml
- name: Generate API types
  run: npm run generate:api-types
- name: Check for type changes
  run: |
    git diff --exit-code src/shared/api/schema.ts || \
    (echo "API types have changed. Please commit the updated schema.ts" && exit 1)
```

### API クライアント実装

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

### リトライ仕様

- **最大リトライ回数**: 3 回
- **リトライ間隔**: 指数バックオフ(1 秒 → 2 秒 → 4 秒)
- **タイムアウト**: 10 秒(統一)

**リトライ対象**:

- ネットワークエラー(接続失敗、タイムアウト)
- 5xx 系ステータスコード(サーバーエラー)

**リトライしないもの**:

- 4xx 系ステータスコード(クライアントエラー)

### エラーハンドリング

#### エラー型定義

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

#### UI 表示方法

| エラー種別                         | 表示方法         | 例                                          |
| ---------------------------------- | ---------------- | ------------------------------------------- |
| 入力エラー(4xx 系)                 | インライン表示   | フォーム下に赤文字「概念名は必須です」      |
| 一時的エラー(5xx 系、ネットワーク) | トースト通知     | 画面右上に「サーバーエラー。リトライ中...」 |
| 致命的エラー                       | エラーページ遷移 | エラー境界でキャッチ → 専用エラー画面       |

#### トースト実装

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

### 実装原則

1. **型安全性**: OpenAPI 仕様書から自動生成した型を使用
2. **リトライ戦略**: ネットワークエラーと 5xx 系のみリトライ、指数バックオフ
3. **統一タイムアウト**: 全 API 10 秒
4. **明確なエラー伝達**: エラー種別に応じた適切な UI 表示
5. **シンプルさ優先**: 複雑なキャッシュ機構やインターセプター不要

<!-- GLOBAL_CONCLUSION_END: frontend-api-client-specification -->
