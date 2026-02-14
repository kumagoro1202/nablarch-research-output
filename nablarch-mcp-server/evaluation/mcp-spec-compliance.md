# nablarch-mcp-server MCP仕様準拠評価レポート

**評価日**: 2026-02-10
**評価者**: 担当者C
**対象**: ~/nablarch-mcp-server
**MCP仕様バージョン**: 2025-03-26
**MCP Java SDK**: io.modelcontextprotocol.sdk:mcp:0.10.0
**Spring AI**: 1.0.0

---

## 1. 評価サマリ

| 評価軸 | スコア | 判定 |
|--------|--------|------|
| **総合MCP仕様準拠度** | **3 / 5** | 主要部分は準拠、重要な非準拠箇所あり |
| Tools準拠度 | 3 / 5 | 8/10ツール登録漏れ、isError未対応 |
| Resources準拠度 | 2 / 5 | 12/推定30+リソース登録漏れ、テンプレート未使用 |
| Prompts準拠度 | 4 / 5 | 6/6登録済み、構造的に準拠 |
| JSON-RPC準拠度 | 4 / 5 | SDKにより概ね準拠 |
| Transport準拠度 | 3 / 5 | STDIO準拠、HTTP Streamable一部非準拠 |
| エラーハンドリング | 2 / 5 | isError未活用、標準エラーコード未使用 |

**総評**: MCP Java SDK + Spring AI MCPの組み合わせにより、JSON-RPCプロトコル層は概ね自動的に準拠している。しかし、アプリケーション層でのTool/Resource登録漏れ、エラーハンドリングの仕様非準拠が目立つ。特にToolの`isError`フラグ未対応とResource登録漏れは、MCPクライアントの利用体験に直接影響する重要な問題である。

---

## 2. MCP仕様バージョンとの照合結果

### 使用SDK

| コンポーネント | バージョン | 備考 |
|---------------|-----------|------|
| MCP Java SDK | 0.10.0 | `io.modelcontextprotocol.sdk:mcp` |
| Spring AI MCP | 1.0.0 | `spring-ai-mcp` |
| Spring AI MCP WebMVC | 0.10.0 | `mcp-spring-webmvc` |
| Spring Boot | 3.4.2 | |

### 仕様バージョン対応状況

MCP仕様 2025-03-26 の主要な変更点との照合:

| 仕様要件 | 対応状況 | 備考 |
|----------|---------|------|
| Tools capability宣言 | ✅ 準拠 | SDK自動処理 |
| Resources capability宣言 | ✅ 準拠 | SDK自動処理 |
| Prompts capability宣言 | ✅ 準拠 | SDK自動処理 |
| JSON-RPC 2.0メッセージ形式 | ✅ 準拠 | SDK自動処理 |
| STDIOトランスポート | ✅ 準拠 | Spring AI Starter |
| Streamable HTTPトランスポート | ⚠️ 部分的 | SSEベース実装、Streamable HTTP非完全対応の可能性 |
| Originヘッダ検証（HTTP MUST要件） | ❌ 非準拠 | Phase 4で対応予定、現在無効 |
| Capability Negotiation | ✅ 準拠 | SDK自動処理 |
| Pagination | ⚠️ 未確認 | resources/list, tools/list のページネーション |

---

## 3. Tools準拠度分析

### 3.1 登録状況

`McpServerConfig.nablarchTools()` に登録されたTool: **8個**

| # | Toolメソッド名 | クラス | 登録 | @Tool description |
|---|---------------|--------|------|-------------------|
| 1 | `searchApi` | SearchApiTool | ✅ | ✅ あり |
| 2 | `validateHandlerQueue` | ValidateHandlerQueueTool | ✅ | ✅ あり |
| 3 | `semanticSearch` | SemanticSearchTool | ✅ | ✅ あり |
| 4 | `generateCode` | CodeGenerationTool | ✅ | ✅ あり |
| 5 | `design` | DesignHandlerQueueTool | ✅ | ✅ あり |
| 6 | `recommend` | RecommendPatternTool | ✅ | ✅ あり |
| 7 | `optimize` | OptimizeHandlerQueueTool | ✅ | ✅ あり |
| 8 | `troubleshoot` | TroubleshootTool | ✅ | ✅ あり |
| 9 | `analyzeMigration` | MigrationAnalysisTool | ❌ **未登録** | ✅ あり |
| 10 | `generateTest` | TestGenerationTool | ❌ **未登録** | ✅ あり |

**重大な非準拠**: MigrationAnalysisToolとTestGenerationToolは`@Service`として存在し`@Tool`アノテーションも持つが、`McpServerConfig.nablarchTools()`のMethodToolCallbackProviderに渡されていない。MCPクライアントからは発見・呼び出し不可能。

### 3.2 Tool名の仕様準拠

MCP仕様ではTool名は`name`フィールドで定義。Spring AIの`@Tool`アノテーションはデフォルトでメソッド名をTool名として使用する。

| 期待されるTool名（context.md記載） | 実際のTool名（メソッド名） | 一致 |
|----------------------------------|--------------------------|------|
| `semantic_search` | `semanticSearch` | ❌ |
| `design_handler_queue` | `design` | ❌ |
| `generate_code` | `generateCode` | ❌ |
| `generate_test` | `generateTest` | ❌ |
| `validate_config` | `validateHandlerQueue` | ❌ |
| `troubleshoot` | `troubleshoot` | ✅ |
| `analyze_migration` | `analyzeMigration` | ❌ |
| `recommend_pattern` | `recommend` | ❌ |
| `optimize_handler_queue` | `optimize` | ❌ |
| `search_api` | `searchApi` | ❌ |

**問題**: Spring AI 1.0.0では`@Tool`にname属性を指定しない場合、Javaメソッド名がそのままTool名になる。ドキュメント記載のsnake_case名と実際のcamelCase名が不一致。MCPクライアントが参照する名前と外部ドキュメントが乖離する。

### 3.3 inputSchema（パラメータ定義）の正確性

MCP仕様ではTool定義に`inputSchema`（JSON Schema）を含む。Spring AIが`@ToolParam`から自動生成する。

| Tool | 必須パラメータ | required=false指定 | 問題 |
|------|--------------|-------------------|------|
| semanticSearch | query | なし（全7パラメータ必須扱い） | ❌ appType, module, source, sourceType, topK, modeは任意だが必須として公開される |
| design | appType | requirements, includeComments | ✅ |
| generateCode | type, name | なし | ❌ appType, specificationsは任意だが必須として公開される |
| validateHandlerQueue | 全2パラメータ | なし | ✅ |
| troubleshoot | errorMessage | なし | ❌ stackTrace, errorCode, environmentは任意だが必須として公開される |
| searchApi | keyword | なし | ❌ categoryは任意だが必須として公開される |
| recommend | requirement | appType, constraints, maxResults | ✅ |
| optimize | currentXml | appType, concern | ✅ |
| analyzeMigration | codeSnippet | sourceVersion, targetVersion, analysisScope | ✅ |
| generateTest | targetClass, testType | なし | ❌ format, testCases, includeExcel, coverageTargetは任意だが必須として公開される |

**問題**: 5個のToolで任意パラメータに`required = false`が指定されておらず、MCPクライアントに対して不正確なinputSchemaが公開される。MCPクライアントはこれらのパラメータを必須と判断し、未指定時にエラーとする可能性がある。

### 3.4 Tool実行結果のフォーマット

MCP仕様では`tools/call`レスポンスは以下の構造:
```json
{
  "content": [{"type": "text", "text": "..."}],
  "isError": false
}
```

**現状**: 全Toolは`String`を返却し、Spring AI MCPが`CallToolResult`にラップする。

| シナリオ | MCP仕様 | 現実装 | 準拠 |
|---------|---------|--------|------|
| 正常系 | content + isError: false | String → TextContent | ✅ |
| ツール実行エラー | content + **isError: true** | **エラー文字列をそのまま返却（isError: false）** | ❌ |
| 不正引数 | JSON-RPC error -32602 | エラー文字列をそのまま返却 | ❌ |

**重大な非準拠**: 全Toolがcatch節でエラーメッセージを通常の文字列として返却している。MCP仕様では、ツール実行中のエラーは`isError: true`を設定すべき（SHOULD）。現実装ではエラーも正常レスポンスと区別不可能。

例: SemanticSearchTool:91-127
```java
try {
    return doSearch(query, filters, effectiveTopK, effectiveMode);
} catch (Exception e) {
    log.error("semantic_search実行中にエラーが発生: {}", e.getMessage(), e);
    return "検索中にエラーが発生しました。search_apiツールをお試しください。"; // isError: false として返却される
}
```

---

## 4. Resources準拠度分析

### 4.1 登録状況

`McpServerConfig.nablarchResources()` に登録されたResource: **12個**（2プロバイダ）

| # | URI | プロバイダ | 登録 | name | description | mimeType |
|---|-----|----------|------|------|-------------|----------|
| 1 | nablarch://handler/web | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 2 | nablarch://handler/rest | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 3 | nablarch://handler/batch | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 4 | nablarch://handler/messaging | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 5 | nablarch://handler/http-messaging | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 6 | nablarch://handler/jakarta-batch | HandlerResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 7 | nablarch://guide/setup | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 8 | nablarch://guide/testing | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 9 | nablarch://guide/validation | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 10 | nablarch://guide/database | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 11 | nablarch://guide/handler-queue | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |
| 12 | nablarch://guide/error-handling | GuideResourceProvider | ✅ | ✅ | ✅ | text/markdown |

### 4.2 未登録のResourceプロバイダ

以下の6プロバイダはソースコードに存在するが`McpServerConfig`に未登録:

| プロバイダ | 期待されるURIパターン | @Component | 登録 |
|-----------|---------------------|-----------|------|
| ApiResourceProvider | nablarch://api/{module}/{class} | ✅ | ❌ **未登録** |
| PatternResourceProvider | nablarch://pattern/{name} | ✅ | ❌ **未登録** |
| ExampleResourceProvider | nablarch://example/{type} | ✅ | ❌ **未登録** |
| ConfigResourceProvider | nablarch://config/{name} | ✅ | ❌ **未登録** |
| AntipatternResourceProvider | nablarch://antipattern/{name} | ✅ | ❌ **未登録** |
| VersionResourceProvider | nablarch://version | ✅ | ❌ **未登録** |

**重大な非準拠**: context.mdで8 URIパターンと記載されているが、実際にMCPクライアントから利用可能なのは2パターン（handler, guide）のみ。6パターン分のリソースがMCPクライアントに公開されていない。

### 4.3 Resource定義の仕様照合

MCP仕様が定める必須/任意フィールド:

| フィールド | MCP仕様 | 登録済みリソース | 準拠 |
|-----------|---------|----------------|------|
| uri | 必須 | ✅ 全リソースに設定 | ✅ |
| name | 必須 | ✅ 全リソースに設定 | ✅ |
| description | 任意 | ✅ 全リソースに設定 | ✅ |
| mimeType | 任意 | ✅ "text/markdown" | ✅ |
| size | 任意 | null | ✅（任意） |

### 4.4 Resource Templateの未使用

MCP仕様 2025-03-26 では`resources/templates/list`によるURIテンプレートの公開をサポート。handler/{app_type}やguide/{topic}はRFC 6570 URIテンプレートとして登録すべきだが、現実装では各値をハードコードした個別リソースとして登録している。

**影響**: MCPクライアントは動的にパラメータを指定してリソースを取得する手段がなく、固定のURI一覧からしか選択できない。テンプレートを使用すれば、クライアントがapp_typeやtopicを動的に補完可能になる。

### 4.5 ReadResourceResultの正確性

```java
new McpSchema.ReadResourceResult(
    List.of(new McpSchema.TextResourceContents(
        request.uri(), "text/markdown", provider.getHandlerMarkdown(type))))
```

- ✅ ReadResourceResultのcontentsリストに1要素
- ✅ TextResourceContentsにuri, mimeType, textを指定
- ✅ request.uri()で元のリクエストURIをそのまま返却

---

## 5. Prompts準拠度分析

### 5.1 登録状況

`McpServerConfig.nablarchPrompts()` に登録されたPrompt: **6個**

| # | Prompt名 | 引数 | 必須指定 | 実装クラス | 登録 |
|---|---------|------|---------|-----------|------|
| 1 | setup-handler-queue | app_type | ✅ required=true | SetupHandlerQueuePrompt | ✅ |
| 2 | create-action | app_type, action_name | ✅ 両方required=true | CreateActionPrompt | ✅ |
| 3 | review-config | config_xml | ✅ required=true | ReviewConfigPrompt | ✅ |
| 4 | explain-handler | handler_name | ✅ required=true | ExplainHandlerPrompt | ✅ |
| 5 | migration-guide | from_version, to_version | ✅ 両方required=true | MigrationGuidePrompt | ✅ |
| 6 | best-practices | topic | ✅ required=true | BestPracticesPrompt | ✅ |

### 5.2 Prompt定義の仕様照合

| フィールド | MCP仕様 | 実装 | 準拠 |
|-----------|---------|------|------|
| name | 必須 | ✅ 全Promptに設定 | ✅ |
| description | 任意 | ✅ 全Promptに設定 | ✅ |
| arguments | 任意 | ✅ 全Promptに引数定義 | ✅ |
| arguments[].name | 必須 | ✅ | ✅ |
| arguments[].description | 任意 | ✅ 全引数に設定 | ✅ |
| arguments[].required | 任意 | ✅ 全引数にtrue設定 | ✅ |

### 5.3 GetPromptResult構造

```java
new McpSchema.GetPromptResult(
    description,
    List.of(new McpSchema.PromptMessage(
        McpSchema.Role.USER,
        new McpSchema.TextContent(content)
    ))
);
```

| フィールド | MCP仕様 | 実装 | 準拠 |
|-----------|---------|------|------|
| description | 任意 | ✅ 設定あり | ✅ |
| messages | 必須 | ✅ リスト1要素 | ✅ |
| messages[].role | 必須（"user"/"assistant"） | ✅ USER | ✅ |
| messages[].content | 必須 | ✅ TextContent | ✅ |
| content.type | 必須 | ✅ "text" | ✅ |
| content.text | 必須 | ✅ | ✅ |

### 5.4 エラーハンドリング

Promptクラスは不正入力時に`IllegalArgumentException`をスロー。MCP仕様ではJSON-RPCエラー`-32602`（Invalid params）を推奨。

**判定**: Spring AI MCP SDKがIllegalArgumentExceptionをJSON-RPCエラーに変換する可能性があるが、明示的なエラーコード指定は行われていない。Promptの名前自体が不正な場合（`prompts/get`で存在しないPrompt名を指定）のハンドリングはSDK依存。

---

## 6. JSON-RPC準拠度分析

### 6.1 JSON-RPC 2.0基本要件

| 要件 | 準拠 | 備考 |
|------|------|------|
| JSON-RPC 2.0メッセージ形式 | ✅ | MCP Java SDK 0.10.0が処理 |
| UTF-8エンコーディング | ✅ | SDK + JVMデフォルト |
| jsonrpc: "2.0"フィールド | ✅ | SDK自動付与 |
| id フィールド（リクエスト） | ✅ | SDK管理 |
| method フィールド | ✅ | SDK管理 |
| result / error 排他 | ✅ | SDK処理 |

### 6.2 MCP固有のJSON-RPCメソッド

| メソッド | 対応 | 備考 |
|---------|------|------|
| initialize | ✅ | SDK + Spring AI |
| tools/list | ✅ | SDK自動生成 |
| tools/call | ✅ | @Tool → MethodToolCallback |
| resources/list | ✅ | SyncResourceSpecification |
| resources/read | ✅ | SyncResourceSpecification |
| resources/templates/list | ⚠️ 未確認 | テンプレートリソース未使用のため |
| prompts/list | ✅ | SyncPromptSpecification |
| prompts/get | ✅ | SyncPromptSpecification |
| notifications/tools/list_changed | ⚠️ 未確認 | 動的Tool変更なし |
| notifications/resources/list_changed | ⚠️ 未確認 | 動的Resource変更なし |

### 6.3 サーバCapability宣言

`application.yaml` の設定:
```yaml
spring.ai.mcp.server:
  name: nablarch-mcp-server
  version: 0.2.0
  type: SYNC
  stdio: true
```

- ✅ サーバ名・バージョン宣言
- ✅ SYNCタイプ（同期処理）
- ⚠️ listChanged capabilityの明示的設定は不明（SDK依存）

---

## 7. エラーハンドリング分析

### 7.1 MCP仕様が定めるエラー体系

MCP仕様では2種類のエラーメカニズムを定義:

1. **プロトコルエラー**: JSON-RPCエラー（不明Tool、不正引数等）
2. **ツール実行エラー**: `isError: true`を含むToolResult

### 7.2 現実装の問題点

#### 問題1: Tool実行エラーのisError未対応

全10個のToolクラスが例外を捕捉してエラー文字列を返却する実装パターン:

```java
// 典型的なパターン（全Toolで共通）
try {
    return doSomething();
} catch (Exception e) {
    return "エラーが発生しました: " + e.getMessage();
}
```

Spring AIの`@Tool`メソッドがStringを返す場合、SDKは`isError: false`のCallToolResultを生成する。エラー時に`isError: true`を返すには、McpSchemaの例外をスローするか、Spring AI固有のエラーハンドリング機構を使用する必要がある。

**影響**: MCPクライアント（Claude, Cline等）がツールエラーを正常レスポンスとして処理してしまい、エラーリカバリロジックが機能しない。

#### 問題2: 入力バリデーションエラーの非標準処理

```java
// SemanticSearchTool
if (query == null || query.isBlank()) {
    return "検索クエリを指定してください。"; // JSON-RPC -32602ではなく通常レスポンス
}
```

MCP仕様では不正な引数に対してJSON-RPCエラー`-32602`を返すべき。現実装はエラーを通常のテキストレスポンスとして返却。

#### 問題3: Resourceエラーの非標準処理

HandlerResourceProvider:
```java
if (!VALID_APP_TYPES.contains(appType)) {
    return "# Unknown Application Type\n\nUnknown application type: " + appType;
}
```

MCP仕様ではリソース未発見時にJSON-RPCエラー`-32002`を返すべき（SHOULD）。

### 7.3 エラーコード対応表

| シナリオ | MCP仕様推奨 | 現実装 | 準拠 |
|---------|------------|--------|------|
| 不明Tool呼び出し | -32602 | SDK処理 | ✅ |
| Tool引数不正 | -32602 | テキスト返却 | ❌ |
| Tool実行失敗 | isError: true | テキスト返却（isError: false） | ❌ |
| Resource未発見 | -32002 | テキスト返却 | ❌ |
| Prompt名不正 | -32602 | IllegalArgumentException | ⚠️ |
| Prompt引数不足 | -32602 | IllegalArgumentException | ⚠️ |
| サーバ内部エラー | -32603 | SDK処理 | ✅ |

---

## 8. Transport準拠度分析

### 8.1 STDIO Transport

| 要件 | 準拠 | 備考 |
|------|------|------|
| stdin/stdout通信 | ✅ | Spring AI MCP Server Starter |
| 改行区切りメッセージ | ✅ | SDK処理 |
| stdoutにMCPメッセージのみ出力 | ✅ | `banner-mode: off`, ログはstderr |
| stderrへのログ出力 | ✅ | SLF4J → stderr |
| web-application-type: none | ✅ | application.yaml |

### 8.2 HTTP Transport

| 要件 | 準拠 | 備考 |
|------|------|------|
| 単一MCPエンドポイント提供 | ✅ | /mcp |
| POST対応（メッセージ送信） | ✅ | WebMvcSseServerTransportProvider |
| GET対応（SSEストリーム） | ✅ | WebMvcSseServerTransportProvider |
| DELETE対応（セッション終了） | ✅ | WebMvcSseServerTransportProvider |
| **Originヘッダ検証（MUST）** | **❌** | `originValidation.enabled: false` |
| localhostバインド推奨 | ✅ | CORSで localhost:3000, :8080 許可 |
| セッション管理 | ⚠️ | SDK依存、Mcp-Session-Idの明示的処理なし |
| SSEストリーミング | ✅ | WebMvcSseServerTransportProvider |
| Streamable HTTP完全対応 | ⚠️ | WebMvcSseServerTransportProviderは旧SSEパターンベースの可能性 |

**重大な非準拠**: MCP仕様 2025-03-26 で Streamable HTTP Transport を実装するサーバは Origin ヘッダ検証が **MUST** 要件。`McpHttpProperties.OriginValidationConfig.enabled = false` であり、DNS rebinding攻撃に脆弱。

### 8.3 CORS設定の適切性

```yaml
cors:
  allowed-origins:
    - "http://localhost:3000"
    - "http://localhost:8080"
  allow-credentials: true
```

- ✅ 開発環境向けのlocalhost限定は適切
- ⚠️ 本番環境向けのOriginリストが未定義（Phase 4事項）

---

## 9. 非準拠箇所の一覧

| # | 重要度 | カテゴリ | ファイル | 行 | 仕様条文 | 現状 | 修正提案 |
|---|--------|---------|--------|-----|---------|------|---------|
| NC-001 | 🔴高 | Tools | McpServerConfig.java | 42-58 | tools/list は全Toolを公開すべき | MigrationAnalysisTool, TestGenerationToolが未登録 | MethodToolCallbackProviderにmigrationAnalysisTool, testGenerationToolを追加 |
| NC-002 | 🔴高 | Resources | McpServerConfig.java | 69-111 | resources/list は全Resourceを公開すべき | 6プロバイダ（Api, Pattern, Example, Config, Antipattern, Version）が未登録 | nablarchResources()に全プロバイダのリソース仕様を追加 |
| NC-003 | 🔴高 | Error | 全Tool（10クラス） | catch節 | Tool実行エラーはisError: trueを設定すべき（SHOULD） | エラー文字列を通常レスポンスとして返却 | Spring AI MCPの`ToolExecutionException`をスローするか、SDK固有のエラーレスポンス機構を使用 |
| NC-004 | 🔴高 | Transport | McpHttpProperties.java | 225 | Streamable HTTPサーバはOriginヘッダを検証しなければならない（MUST） | originValidation.enabled = false | Origin検証を有効化し、許可リストを設定 |
| NC-005 | 🟡中 | Tools | SemanticSearchTool.java他 | @ToolParam | inputSchemaのrequired/optionalが正確であるべき | 5Toolでoptionalパラメータにrequired=false未指定 | 該当@ToolParamにrequired=false追加 |
| NC-006 | 🟡中 | Tools | 全Tool | @Tool | Tool名はドキュメントと一致すべき | メソッド名（camelCase）がドキュメント記載（snake_case）と不一致 | @Toolのname属性（Spring AI 1.0+対応を確認）で明示的に指定するか、ドキュメントを更新 |
| NC-007 | 🟡中 | Error | 全Tool catch節 | - | 不正入力はJSON-RPC -32602を返すべき（SHOULD） | エラー文字列を通常レスポンスとして返却 | 入力バリデーション失敗時に適切なJSON-RPCエラーをスロー |
| NC-008 | 🟡中 | Resources | McpServerConfig.java | 166-190 | Resource TemplateはURIテンプレートで公開推奨 | 全リソースが固定URIで個別登録 | handler/{app_type}, guide/{topic}をResourceTemplateとして登録検討 |
| NC-009 | 🟢低 | Error | HandlerResourceProvider.java | 61-63 | リソース未発見はJSON-RPCエラー-32002を返すべき（SHOULD） | エラーMarkdown文字列を返却 | McpSchema固有の例外をスロー |
| NC-010 | 🟢低 | Transport | StreamableHttpTransportConfig.java | 全体 | Streamable HTTP 2025-03-26完全対応 | WebMvcSseServerTransportProviderはSSEベース実装 | MCP Java SDK更新時にStreamable HTTPへの完全移行を検討 |
| NC-011 | 🟢低 | Prompts | McpServerConfig.java | 134-163 | - | Prompt名がcontext.md記載と不一致（create-web-app等は実在しない） | ドキュメント更新 |

---

## 10. 改善提案（優先度付き）

### 優先度: 高（リリースブロッカー級）

#### P1: 未登録Tool/Resourceの追加

**対象**: McpServerConfig.java
**工数**: 小

MigrationAnalysisTool, TestGenerationToolをnablarchTools()に追加。6つの未登録ResourceProviderをnablarchResources()に追加。MCPクライアントに全機能を公開する最も基本的な要件。

#### P2: Tool実行エラーのisError対応

**対象**: 全Tool（10クラス）
**工数**: 中

Spring AI MCPでisError: trueを返す方法を調査し、エラー時のレスポンスを修正。catch節で単純にStringを返すのではなく、SDK固有のエラー機構を使用する。

#### P3: Origin検証の有効化（HTTP Transport）

**対象**: McpHttpProperties.java, McpCorsConfig.java
**工数**: 小

HTTP Transport使用時のOrigin検証を有効化。MCP仕様のMUST要件。Phase 4待ちだが、仕様準拠の観点では優先すべき。

### 優先度: 中

#### P4: @ToolParam required=false追加

**対象**: SemanticSearchTool, CodeGenerationTool, TroubleshootTool, SearchApiTool, TestGenerationTool
**工数**: 小

任意パラメータに`required = false`を追加し、正確なinputSchemaを生成する。

#### P5: Tool名の明示的指定

**対象**: 全Toolクラス
**工数**: 小

`@Tool(name = "semantic_search", description = "...")` のようにname属性を明示的に指定し、ドキュメントと一致させる。Spring AI 1.0での@Toolのname属性サポート状況を要確認。

#### P6: Resource Template対応

**対象**: McpServerConfig.java
**工数**: 中

handler/{app_type}やguide/{topic}をMCP Resource Templateとして登録し、MCPクライアントの動的補完を可能にする。

### 優先度: 低

#### P7: 入力バリデーションのJSON-RPCエラー化

**対象**: 全Tool
**工数**: 中

不正入力時にエラー文字列ではなくJSON-RPC -32602エラーを返すよう修正。

#### P8: ドキュメント整合性

**対象**: context/nablarch-mcp-server.md
**工数**: 小

実際のTool名、Resource数、Prompt名をコードと一致させる。

---

## 付録: 評価対象ファイル一覧

| ファイル | 評価内容 |
|---------|---------|
| McpServerConfig.java | Tool/Resource/Prompt登録、全体構成 |
| SemanticSearchTool.java | @Tool, @ToolParam, エラーハンドリング |
| DesignHandlerQueueTool.java | @Tool, @ToolParam, required指定 |
| CodeGenerationTool.java | @Tool, @ToolParam, エラーハンドリング |
| ValidateHandlerQueueTool.java | @Tool, @ToolParam |
| TroubleshootTool.java | @Tool, @ToolParam, エラーハンドリング |
| SearchApiTool.java | @Tool, @ToolParam |
| RecommendPatternTool.java | @Tool, @ToolParam, required指定 |
| OptimizeHandlerQueueTool.java | @Tool, @ToolParam, required指定 |
| MigrationAnalysisTool.java | @Tool（未登録確認） |
| TestGenerationTool.java | @Tool（未登録確認） |
| HandlerResourceProvider.java | Resource実装、エラーハンドリング |
| GuideResourceProvider.java | Resource実装 |
| ApiResourceProvider.java | 未登録確認 |
| ConfigResourceProvider.java | 未登録確認 |
| ExampleResourceProvider.java | 未登録確認 |
| PatternResourceProvider.java | 未登録確認 |
| AntipatternResourceProvider.java | 未登録確認 |
| VersionResourceProvider.java | 未登録確認 |
| SetupHandlerQueuePrompt.java | Prompt実装、GetPromptResult構造 |
| CreateActionPrompt.java | Prompt実装、引数バリデーション |
| BestPracticesPrompt.java | Prompt実装 |
| ReviewConfigPrompt.java | Prompt実装 |
| ExplainHandlerPrompt.java | Prompt実装 |
| MigrationGuidePrompt.java | Prompt実装 |
| StreamableHttpTransportConfig.java | HTTP Transport実装 |
| McpHttpProperties.java | HTTP設定、Origin検証 |
| McpCorsConfig.java | CORS設定 |
| application.yaml | サーバ設定、STDIO設定 |
| application-http.yaml | HTTP Transport設定 |
| pom.xml | SDK/依存関係バージョン |
