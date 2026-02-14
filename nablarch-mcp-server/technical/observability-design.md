# Nablarch MCP Server - オブザーバビリティ（可観測性）評価レポート

評価日: 2026-02-10
評価対象: nablarch-mcp-server（Phase 3完了時点）
関連: オブザーバビリティ設計

---

## 1. 評価サマリ

| 評価項目 | スコア | 状態 |
|---------|-------|------|
| **1. ログ（Logging）** | ⭐⭐⭐☆☆ 3/5 | 基本実装あり、構造化ログ未対応 |
| **2. メトリクス（Metrics）** | ⭐☆☆☆☆ 1/5 | 未実装（Actuator/Micrometer未導入） |
| **3. トレーシング（Tracing）** | ⭐☆☆☆☆ 1/5 | 未実装（OpenTelemetry未導入） |
| **4. ヘルスチェック（Health Check）** | ⭐☆☆☆☆ 1/5 | 未実装（Actuator未導入） |
| **5. デバッグ・診断** | ⭐⭐☆☆☆ 2/5 | 基本ログのみ、診断機能なし |
| **6. 運用監視** | ⭐☆☆☆☆ 1/5 | 未実装（外部連携なし） |
| **7. ベストプラクティスとの比較** | ⭐⭐☆☆☆ 2/5 | 乖離大 |
| **総合スコア** | **⭐⭐☆☆☆ 1.6/5** | Phase 3は機能実装優先、可観測性は未着手 |

### 総括

nablarch-mcp-serverはPhase 3完了時点で機能面（10 Tools + 8 Resources + 6 Prompts）は充実しているが、オブザーバビリティは最低限のSLF4Jログ出力に留まる。Spring Boot Actuator、Micrometer、OpenTelemetryのいずれも**依存に含まれておらず、メトリクス・ヘルスチェック・分散トレーシングは一切実装されていない**。Phase 4（本番デプロイ）に向けて、可観測性の強化は必須である。

---

## 2. 現行実装の正確な把握

### 2.1 pom.xml 依存関係

**オブザーバビリティ関連の依存:**

| 依存 | 有無 | 備考 |
|------|------|------|
| `spring-boot-starter-actuator` | **なし** | ヘルスチェック・メトリクスの基盤が不在 |
| `micrometer-core` | **なし** | メトリクス収集基盤が不在 |
| `micrometer-registry-prometheus` | **なし** | Prometheus連携不可 |
| `micrometer-tracing` | **なし** | 分散トレーシング基盤が不在 |
| `opentelemetry-*` | **なし** | OpenTelemetry連携不可 |

**実際に含まれる依存（関連するもの）:**
- `spring-boot-starter-web` — Webサーバ（HTTPプロファイル時）
- `spring-boot-starter-data-jpa` — DB接続（Hibernate）
- `spring-boot-starter-webflux` — WebClient（Embedding API呼び出し用）
- `spring-boot-starter-validation` — バリデーション

### 2.2 application.yaml 設定

**STDIOモード（デフォルト）:**
```yaml
logging:
  pattern:
    console:       # 空欄（Spring Bootデフォルトパターン使用）
  level:
    root: WARN
    com.tis.nablarch.mcp: INFO
```
- ログパターンが空欄のため、Spring Bootデフォルトフォーマット（タイムスタンプ+スレッド名+レベル+ロガー名+メッセージ）が使用される
- rootレベルがWARNのため、Spring Bootフレームワーク自体のINFOログは出力されない
- アプリケーションパッケージ（`com.tis.nablarch.mcp`）はINFOレベル

**HTTPモード（httpプロファイル）:**
```yaml
logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  level:
    root: INFO
    com.tis.nablarch.mcp: DEBUG
```
- 明示的なログパターン指定あり（プレーンテキスト形式）
- HTTPモードではrootがINFO、アプリケーションがDEBUGに引き上げられている

### 2.3 ログ出力の実装

**SLF4J + Logback（Spring Boot標準）:**
- logback.xml/logback-spring.xmlは**存在しない**（Spring Bootデフォルト設定のみ）
- 19クラスでSLF4Jを使用してログ出力を実装

**ログ出力クラス一覧（全19クラス）:**

| パッケージ | クラス | ログ用途 |
|-----------|-------|---------|
| `tools` | `SemanticSearchTool` | エラー時のみ（`log.error`） |
| `tools` | `TroubleshootTool` | デバッグログ（入力パラメータ） |
| `tools` | `CodeGenerationTool` | info（生成開始） |
| `tools` | `TestGenerationTool` | info（生成開始） |
| `knowledge` | `NablarchKnowledgeBase` | info（初期化開始/完了、YAMLロード）、warn（キー不在） |
| `rag.search` | `HybridSearchService` | debug（検索実行）、warn（片方失敗時）、info（両方結果なし） |
| `rag.search` | `BM25SearchService` | debug（検索実行、SQL） |
| `rag.search` | `VectorSearchService` | debug（検索実行、SQL） |
| `rag.search` | `MetadataFilteringService` | ログ出力あり |
| `rag.rerank` | `CrossEncoderReranker` | warn（API失敗時のdegraded mode） |
| `rag.query` | `QueryAnalyzer` | ログ出力あり |
| `rag.ingestion` | `FintanIngester` | ログ出力あり |
| `rag.ingestion` | `OfficialDocsIngester` | ログ出力あり |
| `embedding` | `JinaEmbeddingClient` | debug（完了）、warn（リトライ） |
| `embedding` | `VoyageEmbeddingClient` | ログ出力あり |
| `embedding.local` | `AbstractOnnxEmbeddingClient` | info（モデルロード/アンロード）、debug（完了）、warn（クリーンアップエラー） |
| `codegen` | `DefaultCodeGenerator` | info（生成開始）、warn（知識ベース検索エラー） |
| `http.config` | `StreamableHttpTransportConfig` | info（トランスポート初期化、ルーティング登録） |
| `http.config` | `McpCorsConfig` | ログ出力あり |

### 2.4 エラーハンドリングの実装

- **@ExceptionHandler / @ControllerAdvice**: 未実装
- **グローバルエラーハンドラ**: 未実装
- **ツール単位のtry-catch**: 各Toolクラスで個別にtry-catchを実装
  - `SemanticSearchTool`: catchでログ出力後、RuntimeExceptionをthrow
  - `CrossEncoderReranker`: catchでfallback（元スコア順返却）
  - `JinaEmbeddingClient`: リトライロジック内でcatch、最大リトライ後にEmbeddingExceptionをthrow
  - `HybridSearchService`: CompletableFuture.exceptionallyでグレースフルデグレード
  - `DefaultCodeGenerator`: 知識ベース検索失敗時にフォールバック規約使用

### 2.5 既存のメトリクス・ヘルスチェック実装

**メトリクス:** 未実装。唯一、`SemanticSearchTool.doSearch()`内で`System.currentTimeMillis()`による手動計時あり（レスポンスのMarkdown出力に「検索時間: XXXms」として含まれるのみ。メトリクス基盤には連携していない）。

**ヘルスチェック:** 未実装。docker-compose.ymlでPostgreSQLのhealthcheckは定義されているが、アプリケーション自体のヘルスチェックエンドポイントは存在しない。

---

## 3. 7項目の詳細評価

### 3.1 ログ（Logging）— ⭐⭐⭐☆☆ 3/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| ログフレームワーク | ✅ 実装済 | SLF4J + Logback（Spring Bootデフォルト） |
| ログレベルの使い分け | ✅ 適切 | ERROR/WARN/INFO/DEBUGを用途に応じて使い分け |
| リクエスト/レスポンスログ | ❌ 未実装 | MCP JSON-RPCリクエスト/レスポンスのログ記録なし |
| エラーログ | ⚠️ 部分的 | 各ツールで個別にcatch→log、統一フォーマットなし |
| 構造化ログ（JSON） | ❌ 未対応 | プレーンテキスト出力のみ |
| ログのフィルタリング | ⚠️ 基本的 | パッケージ単位のレベル設定のみ |
| logback設定ファイル | ❌ なし | カスタムAppender、ローテーション設定なし |
| リクエストID/相関ID | ❌ 未実装 | 同一リクエストのログを紐付ける仕組みなし |

**良い点:**
- 19クラスでSLF4Jを統一的に使用（ロギングライブラリの混在なし）
- 起動時の知識ベースロード状況がINFOレベルで記録される
- リトライ/フォールバック発生時にWARNログで記録される
- HTTPプロファイルでは独自のログパターン定義あり

**課題:**
- MCP JSON-RPCのリクエスト/レスポンスが一切ログに記録されない（どのToolが何回呼ばれたか追跡不可）
- 構造化ログ（JSON形式）未対応のため、ログ集約システム（ELK、Loki等）への連携が困難
- logback-spring.xml未定義のため、ファイル出力・ローテーション・環境別設定ができない
- 相関ID未実装のため、並行リクエスト時にログを紐付けられない

### 3.2 メトリクス（Metrics）— ⭐☆☆☆☆ 1/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| Spring Boot Actuator | ❌ 未導入 | pom.xmlに依存なし |
| Micrometer | ❌ 未導入 | メトリクス収集基盤なし |
| MCP固有メトリクス | ❌ 未実装 | Tool呼び出し数/時間/エラー率の計測なし |
| JVMメトリクス | ❌ 未取得 | ヒープ/GC/スレッド情報の公開なし |
| DBメトリクス | ❌ 未取得 | コネクションプール/クエリ統計なし |
| Embeddingメトリクス | ❌ 未計測 | 推論時間/バッチサイズの統計なし |
| カスタムメトリクス | ❌ 未定義 | ビジネスメトリクスの定義なし |

**唯一の計測箇所:** `SemanticSearchTool.doSearch()`での手動計時（`System.currentTimeMillis()`）。これはMarkdown出力に「検索時間」として表示されるのみで、メトリクス基盤には一切連携していない。

### 3.3 トレーシング（Tracing）— ⭐☆☆☆☆ 1/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| 分散トレーシング | ❌ 未導入 | OpenTelemetry/Zipkin/Jaeger等なし |
| リクエストID伝播 | ❌ 未実装 | MCPセッション→Tool→RAG→DB間のID伝播なし |
| Tool呼び出しトレース | ❌ 未実装 | Tool実行の開始/終了/所要時間の追跡不可 |
| RAGパイプライントレース | ❌ 未実装 | BM25→Vector→RRF→Rerankの各ステージの所要時間不明 |
| Embedding APIトレース | ❌ 未実装 | 外部API呼び出しのレイテンシ追跡不可 |

**影響:** MCPクライアント（Claude Code等）からTool呼び出しが遅い場合、RAGパイプラインのどの段階がボトルネックかを特定する手段がない。

### 3.4 ヘルスチェック（Health Check）— ⭐☆☆☆☆ 1/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| Actuator /health | ❌ なし | Actuator未導入 |
| DB接続チェック | ❌ なし | PostgreSQL死活監視なし（docker-compose側のhealthcheckのみ） |
| 知識ファイルチェック | ❌ なし | YAMLファイルの読み込み成否を外部から確認する手段なし |
| Embedding APIチェック | ❌ なし | Jina/Voyage APIの到達性確認手段なし |
| ONNXモデルチェック | ❌ なし | ローカルモデルのロード成否を外部から確認する手段なし |
| Liveness/Readiness | ❌ なし | Kubernetes Probe対応なし |

**備考:** STDIOモードではHTTPエンドポイントが使えないため、ヘルスチェックの必要性は低い。ただしHTTPモードでは必須。

### 3.5 デバッグ・診断 — ⭐⭐☆☆☆ 2/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| 問題原因特定のしやすさ | ⚠️ 部分的 | ログがあれば特定可能だが、情報量が不十分 |
| MCPクライアント側エラー情報 | ⚠️ 最低限 | RuntimeException.messageのみ伝播 |
| 知識ベース状態確認 | ⚠️ 起動ログのみ | 初期化完了ログで件数は把握可能、実行時の状態は確認不可 |
| 開発時デバッグ支援 | ⚠️ 基本的 | DEBUGレベルで検索クエリ/SQLは確認可能 |
| スレッドダンプ | ❌ 手段なし | Actuator/JMX未設定 |
| ヒープダンプ | ❌ 手段なし | Actuator/JMX未設定 |

**良い点:**
- DEBUGレベル有効時、BM25/Vector検索のSQL文とパラメータが出力される
- HybridSearchServiceでグレースフルデグレード発生時にWARNログで理由が記録される
- 知識ベース初期化完了時に各カテゴリの件数がINFOログに出力される

**課題:**
- MCPクライアントに返されるエラー情報が「検索中にエラーが発生しました。search_apiツールをお試しください。」のような汎用メッセージのみ
- ツールごとのエラーハンドリングが統一されておらず、同種のエラーでも返却メッセージが異なる
- 実行時の知識ベース状態（ロード済みエントリ数、最終アクセス時刻等）を確認するAPIがない

### 3.6 運用監視 — ⭐☆☆☆☆ 1/5

**現状:**

| 観点 | 状態 | 詳細 |
|------|------|------|
| Prometheus連携 | ❌ 不可 | Micrometer未導入 |
| Grafana連携 | ❌ 不可 | メトリクスエンドポイントなし |
| アラート設定 | ❌ 不可 | メトリクス基盤なし |
| ダッシュボード | ❌ 不可 | 可視化データなし |
| ログ集約 | ❌ 未対応 | 構造化ログ未実装 |
| Docker監視 | ⚠️ 最低限 | docker-composeでDB healthcheckのみ |

### 3.7 ベストプラクティスとの比較 — ⭐⭐☆☆☆ 2/5

#### Spring Boot MCPサーバーのオブザーバビリティBP

| ベストプラクティス | 現状 | 乖離 |
|-----------------|------|------|
| Actuator必須導入 | 未導入 | **大** |
| /health, /info, /metricsエンドポイント公開 | なし | **大** |
| MicrometerによるTool呼び出しメトリクス | なし | **大** |
| JSON構造化ログ（本番環境） | プレーンテキストのみ | **大** |
| リクエスト相関IDの伝播 | なし | **中** |
| エラーレスポンスの標準化 | 個別実装 | **中** |
| 起動完了の明示的ログ | 知識ベース初期化ログはあり | **小** |

#### 他MCPサーバー実装との比較

Spring AI公式のMCPサーバーサンプルでも、Actuatorは通常含まれていない（STDIOモードではHTTPサーバ不要のため）。ただし、HTTPトランスポートモードで運用する場合はActuator導入が標準的。nablarch-mcp-serverはHTTPモードを既に実装しているため、このモードではActuator導入が望ましい。

#### 12-Factor App / Cloud Native要件

| 要件 | 現状 | 評価 |
|------|------|------|
| **XI. ログ** — イベントストリームとして扱う | ❌ | stdout出力はされるが構造化されていない |
| **XII. 管理プロセス** — ワンオフ管理タスク | ❌ | 診断用エンドポイントなし |
| ヘルスチェックエンドポイント | ❌ | Kubernetes Probe非対応 |
| メトリクスエンドポイント | ❌ | Prometheus Scrape非対応 |
| グレースフルシャットダウン | ⚠️ | Spring Bootデフォルト（ONNXモデルの@PreDestroyは実装済み） |
| 環境変数による設定 | ✅ | DB接続情報、APIキー等は環境変数対応 |

---

## 4. 改善提案（優先度付き）

### P0: 運用に必須（即時対応）

| # | 提案 | 概要 | 影響範囲 |
|---|------|------|---------|
| P0-1 | **Spring Boot Actuator導入** | pom.xmlに`spring-boot-starter-actuator`追加。/health, /info, /metricsエンドポイントを公開 | pom.xml, application-http.yaml |
| P0-2 | **ヘルスチェック実装** | DB接続、知識ベースロード状態、Embeddingモデル状態のカスタムHealthIndicator実装 | 新規3クラス + application-http.yaml |
| P0-3 | **構造化ログ（JSON）導入** | logback-spring.xml追加。HTTPモード（本番相当）ではJSON形式、STDIOモードではプレーンテキスト | logback-spring.xml新規作成 |

### P1: 品質向上に重要（次フェーズ）

| # | 提案 | 概要 | 影響範囲 |
|---|------|------|---------|
| P1-1 | **Micrometer導入 + MCP固有メトリクス** | Tool呼び出し数（Counter）、レスポンス時間（Timer）、エラー率、RAGパイプライン各ステージの所要時間を計測 | pom.xml + 各Toolクラス + 新規MetricsInterceptor |
| P1-2 | **リクエスト相関ID** | MDC（Mapped Diagnostic Context）を使用し、MCPセッションIDまたはリクエストIDをログに自動付与 | logback-spring.xml + 新規Filter/Interceptor |
| P1-3 | **MCP JSON-RPCリクエスト/レスポンスログ** | Tool呼び出し名、パラメータ（要サニタイズ）、レスポンスサイズ、処理時間を記録 | 新規LoggingInterceptor or AOP |
| P1-4 | **エラーハンドリングの統一** | Tool横断でエラーレスポンスフォーマットを統一。エラーコード体系を定義 | 全Toolクラス + 新規ErrorResponseBuilder |
| P1-5 | **Prometheus連携** | `micrometer-registry-prometheus`追加。/actuator/prometheusエンドポイントでメトリクスをScrape可能に | pom.xml + application-http.yaml |

### P2: あると良い（余裕があれば）

| # | 提案 | 概要 | 影響範囲 |
|---|------|------|---------|
| P2-1 | **分散トレーシング（Micrometer Tracing）** | `micrometer-tracing-bridge-otel`導入。Zipkin/Jaeger連携でTool→RAG→DB→API間のトレースを可視化 | pom.xml + docker-compose.yml(Jaeger追加) |
| P2-2 | **Grafanaダッシュボード** | MCP Server用ダッシュボードテンプレート作成（Tool呼び出し頻度、レスポンス時間分布、エラー率） | docker-compose.yml(Grafana+Prometheus追加) + ダッシュボードJSON |
| P2-3 | **Embedding推論メトリクス** | ONNXモデルの推論時間、バッチサイズ別パフォーマンス、トークン数統計 | AbstractOnnxEmbeddingClient + Micrometer |
| P2-4 | **知識ベース診断エンドポイント** | /actuator/knowledgebase で知識ベースの状態（ロード済みエントリ数、メモリ使用量概算、最終ロード時刻）を公開 | 新規Endpoint |
| P2-5 | **ログローテーション** | logback-spring.xmlでファイル出力 + SizeAndTimeBasedRollingPolicy | logback-spring.xml |

### P3: 将来検討

| # | 提案 | 概要 |
|---|------|------|
| P3-1 | **OpenTelemetry Collector統合** | OTel Collectorを介してJaeger+Prometheus+Lokiにテレメトリデータを統合 |
| P3-2 | **SLO/SLI定義** | Tool呼び出し応答時間のSLO（例: P95 < 3秒）を定義し、アラート設定 |
| P3-3 | **カスタムSpring Boot Admin** | 複数MCPサーバーインスタンスの一元管理ダッシュボード |
| P3-4 | **クライアント側メトリクス伝播** | MCPクライアント（Claude Code等）にレスポンスヘッダーでメトリクス情報（処理時間、RAGヒット数等）を伝播 |

---

## 5. 実装ロードマップ案

### Phase 4-A: 基本的なオブザーバビリティ（P0 + P1の一部）

```
1. spring-boot-starter-actuator 追加（P0-1）
2. logback-spring.xml 作成（P0-3）
3. カスタムHealthIndicator 3件実装（P0-2）
   - DatabaseHealthIndicator（DB接続確認）
   - KnowledgeBaseHealthIndicator（知識ベースロード状態）
   - EmbeddingModelHealthIndicator（ONNXモデル状態）
4. リクエスト相関ID導入（P1-2）
5. エラーハンドリング統一（P1-4）
```

推定作業量: 2-3タスク（実装作業）

### Phase 4-B: メトリクス・監視（P1残り + P2の一部）

```
1. Micrometer + Prometheus Registry導入（P1-1, P1-5）
2. Tool呼び出しメトリクスInterceptor実装（P1-1）
3. MCP JSON-RPCログ記録（P1-3）
4. Grafanaダッシュボード（P2-2）
5. docker-compose.yml にPrometheus + Grafana追加（P2-2）
```

推定作業量: 3-4タスク（実装作業）

---

## 6. 参考: ソースコードから確認した事実

### ログ出力パターンの分類

**INFO レベル（運用上有用なログ）:**
- `NablarchKnowledgeBase`: 初期化開始/完了、各YAMLロード完了
- `AbstractOnnxEmbeddingClient`: モデルロード/アンロード
- `StreamableHttpTransportConfig`: HTTPトランスポート初期化
- `DefaultCodeGenerator`: コード生成開始
- `HybridSearchService`: 両検索結果なし時

**WARN レベル（注意が必要な状況）:**
- `NablarchKnowledgeBase`: YAMLキー不在
- `HybridSearchService`: BM25/ベクトル検索片方失敗（グレースフルデグレード）
- `CrossEncoderReranker`: リランキングAPI失敗（degraded mode）
- `JinaEmbeddingClient`: API呼び出しリトライ
- `AbstractOnnxEmbeddingClient`: クリーンアップエラー
- `DefaultCodeGenerator`: 知識ベース検索エラー（フォールバック使用）
- `BM25SearchService`: メタデータパース失敗

**ERROR レベル（エラー状況）:**
- `SemanticSearchTool`: 検索実行中のエラー

**DEBUG レベル（開発時のみ有用）:**
- `TroubleshootTool`: トラブルシューティング開始パラメータ
- `HybridSearchService`: 検索実行パラメータ
- `BM25SearchService`: 検索実行パラメータ、SQL文
- `VectorSearchService`: 検索実行パラメータ、SQL文
- `JinaEmbeddingClient`: Embedding生成完了
- `AbstractOnnxEmbeddingClient`: Embedding生成完了

### エラーハンドリングの現状パターン

| パターン | 使用箇所 | 動作 |
|---------|---------|------|
| catch→log→rethrow | SemanticSearchTool | ログ後RuntimeException |
| catch→fallback | CrossEncoderReranker | ログ後元スコア順返却 |
| catch→retry→rethrow | JinaEmbeddingClient | リトライ後EmbeddingException |
| exceptionally→empty | HybridSearchService | 片方失敗時はもう片方のみで応答 |
| catch→fallback convention | DefaultCodeGenerator | 知識ベース検索失敗時フォールバック規約 |

---

## 7. 結論

nablarch-mcp-serverのオブザーバビリティはPhase 3完了時点で**最低限のログ出力のみ**という状況である。これはPhase 1-3が機能実装（Tools/Resources/Prompts）に注力していたためであり、Phase 4（本番デプロイ）で対応すべき領域として合理的に位置付けられている。

**最優先で取り組むべきはP0の3項目:**
1. **Actuator導入** — 全ての可観測性の基盤
2. **ヘルスチェック** — DB・知識ベース・Embeddingモデルの死活監視
3. **構造化ログ** — ログ集約システム連携の前提条件

これら3項目の実装により、スコアは現状の1.6/5から3.5/5程度まで改善できると見込まれる。

---

*評価完了: 2026-02-10*
*ソースコードからの事実のみ記載。推測は含まない。*
