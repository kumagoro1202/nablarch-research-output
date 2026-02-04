---
name: mcp-server-scaffold
description: MCP（Model Context Protocol）サーバーのResource/Prompt/Tool登録コードをスキャフォールド自動生成するスキル。Spring AI MCP Server + Spring Boot 3.xを前提に、McpServerConfig Bean定義テンプレート、@Component Prompt/Toolクラス、ResourceProvider実装、知識YAMLファイル、テストクラスを一括生成する。「新しいMCP Resourceを追加して」「Promptテンプレートを生成して」「MCP Toolのスキャフォールドを作って」といった要望に対応する。また、HTTP/SSE Transport設計・実装パターン、Nablarch向けMCP Tool実装パターン、MockMvc/WireMock/JUnit5を組み合わせたToolテストパターンも生成可能。
---

# MCP Server Scaffold

## Overview

Spring AI MCP Server（spring-ai-starter-mcp-server）を基盤としたMCPサーバーにおいて、Resource / Prompt / Tool の登録コード一式をスキャフォールド（雛形）として自動生成するスキル。

nablarch-mcp-serverの実装で確立された以下のパターンを再利用可能なテンプレートとして体系化する：

**MCPサーバーの3プリミティブ:**

| プリミティブ | 役割 | nablarch-mcp-server実績 |
|---|---|---|
| Resource | 知識データの読み取り提供 | 12リソース（Handler 6種 + Guide 6種） |
| Prompt | テンプレート付き対話フロー | 6プロンプト（handler-queue, action, config-review等） |
| Tool | 検索・バリデーション等の操作 | 2ツール（SearchApi, ValidateHandlerQueue） |

**McpServerConfig.java の登録パターン:**

```java
@Configuration
public class McpServerConfig {
    // Tool: MethodToolCallbackProvider で @Tool メソッドを自動登録
    @Bean
    public ToolCallbackProvider tools(XxxTool tool1, YyyTool tool2) {
        return MethodToolCallbackProvider.builder()
                .toolObjects(tool1, tool2).build();
    }

    // Resource: SyncResourceSpecification のリストで登録
    @Bean
    public List<McpServerFeatures.SyncResourceSpecification> resources(
            XxxResourceProvider provider) {
        return List.of(createResourceSpec("key", "Name", "desc", provider));
    }

    // Prompt: SyncPromptSpecification のリストで登録
    @Bean
    public List<McpServerFeatures.SyncPromptSpecification> prompts(
            XxxPrompt prompt) {
        return List.of(promptSpec("name", "desc",
                List.of(arg("param", "desc", true)), prompt::execute));
    }
}
```

**本スキルの特長:**
- nablarch-mcp-serverの実装パターンを完全にテンプレート化
- Resource / Prompt / Tool の3種全てに対応
- McpServerConfig Bean定義の自動生成
- 知識YAMLファイルのスケルトン生成
- JUnitテストクラスの自動生成
- Spring AI BOM + spring-ai-starter-mcp-server の依存関係設定テンプレート

**用途:**
- 新規MCP Serverプロジェクトの初期セットアップ
- 既存MCP Serverへの新規Resource/Prompt/Tool追加
- Nablarch以外のフレームワーク向けMCPサーバー構築
- MCPプリミティブの実装パターン学習

**実績:**
- nablarch-mcp-server Phase 1: Resource 12種、Prompt 6種、Tool 2種を実装
- McpServerConfig.java: 3つの@Bean メソッドで全プリミティブを統合管理
- 6 Promptクラス: @Component + @PostConstruct パターンで知識YAML参照

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「新しいMCP Resourceを追加して」
- 「MCP Promptテンプレートを生成して」
- 「MCP Toolのスキャフォールドを作って」
- 「Spring AI MCP Serverの初期設定をセットアップして」
- 「McpServerConfigに新しいBeanを追加して」
- 「○○フレームワーク用のMCPサーバーを構築して」
- 「HTTP/SSE Transportの設計書を作って」
- 「HTTP Transport実装のスキャフォールドを生成して」
- 「Nablarch向けのMCP Toolを実装して」
- 「MCP ToolのMockMvc/WireMockテストを作って」
- MCP ServerのResource/Prompt/Toolを新規追加する必要がある場合
- Spring AI MCP Serverプロジェクトを新規作成する場合
- 既存のMCPサーバーに新しいプリミティブを追加する場合
- STDIO TransportからHTTP/SSE Transportへ移行する場合
- Nablarch固有のMCP Tool実装パターンが必要な場合
- MCP Toolの統合テスト・単体テストを整備する場合

**トリガーキーワード**: MCP Server, Resource登録, Prompt登録, Tool登録, スキャフォールド, McpServerConfig, Spring AI MCP, HTTP Transport, SSE Transport, Nablarch Tool, MockMvc, WireMock, MCP Test

## Instructions

### Phase 1: プロジェクト構成の確認

対象プロジェクトの構成を確認し、生成するコードの仕様を決定する。

#### Step 1.1: 既存プロジェクト構成の調査

```
【プロジェクト調査】

1. プロジェクトルートを確認
   Glob: build.gradle.kts OR pom.xml
   - Spring Boot バージョン確認
   - Spring AI BOM バージョン確認
   - spring-ai-starter-mcp-server 依存の有無

2. 既存のMcpServerConfig確認
   Grep: "@Configuration" + "McpServerConfig" OR "McpServer"
   - 既存のBean定義数
   - 使用中のMCP SDK型

3. パッケージ構造確認
   Glob: src/main/java/**/*.java
   - config/ パッケージの有無
   - resources/ prompts/ tools/ パッケージの有無
   - 既存のResourceProvider/Prompt/Toolクラスの確認

4. 知識ファイル確認
   Glob: src/main/resources/knowledge/*.yaml
   - 既存の知識ファイル一覧
   - YAMLスキーマパターンの確認
```

#### Step 1.2: 生成対象の決定

```
【生成対象リスト】

ユーザーのリクエストに基づき、以下の生成物を決定する:

□ Resource関連:
  □ ResourceProvider クラス（@Component）
  □ McpServerConfig の nablarchResources() Bean
  □ createXxxResourceSpec() ヘルパーメソッド
  □ 知識YAMLファイルのスケルトン

□ Prompt関連:
  □ Prompt クラス（@Component + @PostConstruct）
  □ McpServerConfig の nablarchPrompts() Bean
  □ promptSpec() / arg() ヘルパーメソッド
  □ 知識YAMLファイルのスケルトン

□ Tool関連:
  □ Tool クラス（@Component + @Tool メソッド）
  □ McpServerConfig の nablarchTools() Bean
  □ MethodToolCallbackProvider 設定

□ 共通:
  □ build.gradle.kts 依存関係
  □ application.properties / application.yml 設定
  □ JUnit テストクラス
```

### Phase 2: Resource スキャフォールド生成

MCP Resource の登録コードを生成する。

#### Step 2.1: ResourceProvider クラス生成

```
【ResourceProvider テンプレート】

/**
 * {対象ドメイン}のMCPリソースプロバイダ。
 *
 * <p>knowledge/{yaml_file} から{対象データ}を読み込み、
 * Markdown形式でMCPリソースとして提供する。</p>
 */
@Component
public class {Name}ResourceProvider {

    private Map<String, {DataType}> dataMap;

    /**
     * 知識ファイルを読み込み、データマップを初期化する。
     */
    @PostConstruct
    public void init() {
        try (InputStream is = getClass().getResourceAsStream(
                "/knowledge/{yaml_file}")) {
            ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
            // YAML構造に応じたデシリアライズ
            Map<String, Object> root = mapper.readValue(is,
                new TypeReference<Map<String, Object>>() {});
            // データマップの構築
            this.dataMap = buildDataMap(root);
        } catch (IOException e) {
            throw new UncheckedIOException(
                "{yaml_file}の読み込みに失敗しました", e);
        }
    }

    /**
     * 指定キーに対応するMarkdownコンテンツを返す。
     *
     * @param key リソースキー
     * @return Markdown形式のコンテンツ
     */
    public String getMarkdown(String key) {
        {DataType} data = dataMap.get(key);
        if (data == null) {
            return "# " + key + "\n\n該当するデータが見つかりません。";
        }
        return formatMarkdown(data);
    }

    // --- private methods ---
}
```

#### Step 2.2: McpServerConfig Resource Bean 生成

```
【Resource Bean テンプレート】

/**
 * {対象ドメイン}のMCPリソースを登録する。
 *
 * <p>{N}種のリソースを登録する。</p>
 *
 * @param provider リソースプロバイダ
 * @return MCPサーバ自動構成用のリソース仕様リスト
 */
@Bean
public List<McpServerFeatures.SyncResourceSpecification> {domain}Resources(
        {Name}ResourceProvider provider) {
    return List.of(
        createResourceSpec("{key1}", "{Name1}", "{Description1}", provider),
        createResourceSpec("{key2}", "{Name2}", "{Description2}", provider)
        // ... 必要な数だけ追加
    );
}

private static McpServerFeatures.SyncResourceSpecification createResourceSpec(
        String key, String name, String description,
        {Name}ResourceProvider provider) {
    String uri = "{domain}://{category}/" + key;
    return new McpServerFeatures.SyncResourceSpecification(
        new McpSchema.Resource(uri, name, description, "text/markdown", null),
        (exchange, request) -> new McpSchema.ReadResourceResult(
            List.of(new McpSchema.TextResourceContents(
                request.uri(), "text/markdown",
                provider.getMarkdown(key))))
    );
}
```

### Phase 3: Prompt スキャフォールド生成

MCP Prompt の登録コードを生成する。

#### Step 3.1: Prompt クラス生成

```
【Prompt テンプレート】

/**
 * {Prompt名}のMCP Promptクラス。
 *
 * <p>{機能説明}。
 * knowledge/{yaml_files} を参照して結果を生成する。</p>
 */
@Component
public class {Name}Prompt {

    private Map<String, Object> knowledgeData;

    /**
     * 知識ファイルを読み込み、データを初期化する。
     */
    @PostConstruct
    public void init() {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        try (InputStream is = getClass().getResourceAsStream(
                "/knowledge/{yaml_file}")) {
            this.knowledgeData = mapper.readValue(is,
                new TypeReference<Map<String, Object>>() {});
        } catch (IOException e) {
            throw new UncheckedIOException(
                "{yaml_file}の読み込みに失敗しました", e);
        }
    }

    /**
     * Promptを実行し、結果を返す。
     *
     * @param arguments Prompt引数のマップ
     * @return Prompt実行結果
     * @throws IllegalArgumentException 必須引数が不足している場合
     */
    public McpSchema.GetPromptResult execute(Map<String, String> arguments) {
        // 引数バリデーション
        String param1 = arguments.get("{param1}");
        if (param1 == null || param1.isBlank()) {
            throw new IllegalArgumentException("{param1}は必須です");
        }

        // 知識データからのフィルタリング・整形
        String content = buildContent(param1);

        return new McpSchema.GetPromptResult(
            "{prompt-description}",
            List.of(new McpSchema.PromptMessage(
                McpSchema.Role.USER,
                new McpSchema.TextContent(content)))
        );
    }

    // --- private methods ---
}
```

#### Step 3.2: McpServerConfig Prompt Bean 生成

```
【Prompt Bean テンプレート】

/**
 * {対象ドメイン}のMCP Promptを登録する。
 *
 * <p>{N}種のPromptテンプレートを登録し、各Promptクラスに処理を委譲する。</p>
 */
@Bean
public List<McpServerFeatures.SyncPromptSpecification> {domain}Prompts(
        {Name1}Prompt prompt1,
        {Name2}Prompt prompt2) {
    return List.of(
        promptSpec("{prompt-name-1}",
            "{Description1}",
            List.of(arg("{param}", "{param desc}", true)),
            prompt1::execute),
        promptSpec("{prompt-name-2}",
            "{Description2}",
            List.of(
                arg("{param1}", "{param1 desc}", true),
                arg("{param2}", "{param2 desc}", true)),
            prompt2::execute)
    );
}

// --- ヘルパーメソッド ---

private static McpServerFeatures.SyncPromptSpecification promptSpec(
        String name, String description,
        List<McpSchema.PromptArgument> arguments,
        Function<Map<String, String>, McpSchema.GetPromptResult> handler) {
    return new McpServerFeatures.SyncPromptSpecification(
        new McpSchema.Prompt(name, description, arguments),
        (exchange, request) -> {
            Map<String, String> args = new java.util.HashMap<>();
            if (request.arguments() != null) {
                request.arguments().forEach(
                    (k, v) -> args.put(k, v != null ? v.toString() : null));
            }
            return handler.apply(args);
        }
    );
}

private static McpSchema.PromptArgument arg(
        String name, String description, boolean required) {
    return new McpSchema.PromptArgument(name, description, required);
}
```

### Phase 4: Tool スキャフォールド生成

MCP Tool の登録コードを生成する。

#### Step 4.1: Tool クラス生成

```
【Tool テンプレート】

/**
 * {Tool名}のMCP Toolクラス。
 *
 * <p>{機能説明}。</p>
 */
@Component
public class {Name}Tool {

    /**
     * {操作の説明}。
     *
     * @param param1 {パラメータ説明}
     * @return {戻り値説明}
     */
    @Tool(description = "{Tool description for MCP client}")
    public String {methodName}(
            @ToolParam(description = "{param description}") String param1) {
        // ツール実装
        return result;
    }
}
```

#### Step 4.2: McpServerConfig Tool Bean 生成

```
【Tool Bean テンプレート】

/**
 * MCPツールをSpring AI ToolCallbackとして登録する。
 *
 * @param tool1 {Tool1の説明}
 * @param tool2 {Tool2の説明}
 * @return MCPサーバ自動構成用のToolCallbackProvider
 */
@Bean
public ToolCallbackProvider {domain}Tools(
        {Name1}Tool tool1,
        {Name2}Tool tool2) {
    return MethodToolCallbackProvider.builder()
            .toolObjects(tool1, tool2)
            .build();
}
```

### Phase 5: プロジェクト設定とテスト生成

依存関係設定とテストクラスを生成する。

#### Step 5.1: build.gradle.kts 設定

```
【Gradle設定テンプレート】

plugins {
    java
    id("org.springframework.boot") version "{spring-boot-version}"
    id("io.spring.dependency-management") version "1.1.7"
}

dependencyManagement {
    imports {
        mavenBom("org.springframework.ai:spring-ai-bom:{spring-ai-version}")
    }
}

dependencies {
    // MCP Server
    implementation("org.springframework.ai:spring-ai-starter-mcp-server")

    // YAML parsing
    implementation("com.fasterxml.jackson.dataformat:jackson-dataformat-yaml")

    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}
```

#### Step 5.2: application.properties 設定

```
【アプリケーション設定テンプレート】

# MCP Server Configuration
spring.ai.mcp.server.name={server-name}
spring.ai.mcp.server.version={version}

# STDIO Transport (Phase 1)
spring.main.web-application-type=none
spring.main.banner-mode=off
logging.pattern.console=
```

#### Step 5.3: テストクラス生成

```
【テストテンプレート — Promptクラス】

class {Name}PromptTest {

    private {Name}Prompt prompt;

    @BeforeEach
    void setUp() {
        prompt = new {Name}Prompt();
        prompt.init();
    }

    @Test
    void execute_正常系_{param}指定() {
        Map<String, String> args = Map.of("{param}", "{value}");
        McpSchema.GetPromptResult result = prompt.execute(args);

        assertNotNull(result);
        assertFalse(result.messages().isEmpty());
        String content = ((McpSchema.TextContent)
            result.messages().get(0).content()).text();
        assertTrue(content.contains("{expected}"));
    }

    @Test
    void execute_異常系_{param}null() {
        Map<String, String> args = new HashMap<>();
        args.put("{param}", null);

        assertThrows(IllegalArgumentException.class,
            () -> prompt.execute(args));
    }

    @Test
    void execute_異常系_{param}空文字() {
        Map<String, String> args = Map.of("{param}", "");

        assertThrows(IllegalArgumentException.class,
            () -> prompt.execute(args));
    }
}

【テストテンプレート — ResourceProvider】

class {Name}ResourceProviderTest {

    private {Name}ResourceProvider provider;

    @BeforeEach
    void setUp() {
        provider = new {Name}ResourceProvider();
        provider.init();
    }

    @Test
    void getMarkdown_正常系_存在するキー() {
        String markdown = provider.getMarkdown("{existing_key}");
        assertNotNull(markdown);
        assertFalse(markdown.isEmpty());
        assertTrue(markdown.contains("{expected_content}"));
    }

    @Test
    void getMarkdown_正常系_存在しないキー() {
        String markdown = provider.getMarkdown("nonexistent");
        assertTrue(markdown.contains("該当するデータが見つかりません"));
    }
}

【テストテンプレート — Toolクラス】

@SpringBootTest
class {Name}ToolTest {

    @Autowired
    private {Name}Tool tool;

    @Test
    void {methodName}_正常系() {
        String result = tool.{methodName}("{valid_input}");
        assertNotNull(result);
        // 期待される出力を検証
    }

    @Test
    void {methodName}_異常系_不正入力() {
        String result = tool.{methodName}("{invalid_input}");
        assertTrue(result.contains("エラー") || result.contains("見つかりません"));
    }
}
```

### Phase 6: 統合と動作確認

生成したコードをプロジェクトに統合し、動作を確認する。

#### Step 6.1: ファイル配置

```
【ファイル配置】

src/main/java/{base_package}/
├── config/
│   └── McpServerConfig.java          # Bean定義（追加 or 更新）
├── resources/
│   └── {Name}ResourceProvider.java    # Resource用
├── prompts/
│   └── {Name}Prompt.java             # Prompt用
└── tools/
    └── {Name}Tool.java               # Tool用

src/main/resources/
├── knowledge/
│   └── {category}.yaml               # 知識YAML（スケルトン）
└── application.properties             # MCP設定

src/test/java/{base_package}/
├── resources/
│   └── {Name}ResourceProviderTest.java
├── prompts/
│   └── {Name}PromptTest.java
└── tools/
    └── {Name}ToolTest.java
```

#### Step 6.2: ビルド・テスト実行

```
【動作確認】

1. コンパイル確認
   ./gradlew compileJava

2. テスト実行
   ./gradlew test

3. 全体ビルド
   ./gradlew clean build

4. MCP Server起動テスト（STDIOモード）
   echo '{"jsonrpc":"2.0","method":"initialize","params":{"capabilities":{}},"id":1}' | \
     java -jar build/libs/{project}.jar
```

### Phase 7: MCP HTTP Transport設計パターン

HTTP/SSE Transport への移行を計画するための設計ドキュメントを生成する。

#### Step 7.1: Transport設計ドキュメント生成

```
【HTTP Transport設計ドキュメント テンプレート】

# MCP HTTP/SSE Transport 設計書

## 1. 概要

### 1.1 背景
- STDIO Transport の制約（Claude Desktop等のローカル実行のみ）
- HTTP Transport によるネットワーク経由アクセスの必要性
- SSE（Server-Sent Events）によるストリーミング対応

### 1.2 設計方針
- Spring Boot の組み込みサーバー（Tomcat/Netty）を活用
- SSE エンドポイントで MCP JSON-RPC メッセージをストリーミング
- セキュリティ（認証・認可）の組み込み

## 2. エンドポイント設計

### 2.1 MCP メッセージングエンドポイント

| エンドポイント | メソッド | 説明 |
|---------------|---------|------|
| `/mcp/sse` | GET | SSE接続確立（ストリーミング） |
| `/mcp/message` | POST | クライアント→サーバーメッセージ送信 |
| `/mcp/health` | GET | ヘルスチェック |

### 2.2 リクエスト/レスポンス形式

**POST /mcp/message リクエスト:**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "search-api",
    "arguments": {"query": "Handler"}
  },
  "id": "req-001"
}
```

**SSE ストリーム形式:**
```
event: message
data: {"jsonrpc":"2.0","result":{...},"id":"req-001"}

event: message
data: {"jsonrpc":"2.0","method":"notifications/resources/updated"}
```

## 3. 認証設計

### 3.1 認証方式の選択肢

| 方式 | 特徴 | ユースケース |
|------|------|-------------|
| API Key | シンプル、静的トークン | 内部システム連携 |
| Bearer Token (JWT) | 標準的、有効期限あり | 外部クライアント |
| mTLS | 相互認証、高セキュリティ | エンタープライズ |
| OAuth 2.0 | 委譲認証、スコープ制御 | SaaS連携 |

### 3.2 推奨構成（API Key + Rate Limiting）

```yaml
# application.yml
mcp:
  server:
    transport: http
    http:
      port: 8080
      auth:
        type: api-key
        header-name: X-MCP-API-Key
        keys:
          - name: client-1
            key: ${MCP_API_KEY_CLIENT1}
            rate-limit: 100/min
```

## 4. エラーハンドリング設計

### 4.1 MCP JSON-RPC エラーコード

| コード | 名前 | 説明 |
|--------|------|------|
| -32700 | Parse error | JSON パースエラー |
| -32600 | Invalid Request | リクエスト形式不正 |
| -32601 | Method not found | メソッド未定義 |
| -32602 | Invalid params | パラメータ不正 |
| -32603 | Internal error | サーバー内部エラー |
| -32000〜-32099 | Server error | アプリケーション固有エラー |

### 4.2 HTTP ステータスコードとのマッピング

| HTTPステータス | MCPエラーコード | 状況 |
|---------------|----------------|------|
| 200 OK | - | 正常（エラーもJSON-RPCで返却） |
| 400 Bad Request | -32700, -32600 | リクエスト不正 |
| 401 Unauthorized | -32000 | 認証エラー |
| 429 Too Many Requests | -32001 | レート制限超過 |
| 500 Internal Server Error | -32603 | サーバーエラー |

### 4.3 エラーレスポンス例

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": {
      "param": "app_type",
      "reason": "必須パラメータが指定されていません"
    }
  },
  "id": "req-001"
}
```

## 5. セキュリティ考慮事項

### 5.1 必須対策
- [ ] HTTPS強制（本番環境）
- [ ] CORS設定（許可オリジン限定）
- [ ] レート制限（DoS対策）
- [ ] 入力バリデーション（インジェクション対策）
- [ ] ログ記録（監査証跡）

### 5.2 推奨対策
- [ ] API Key のローテーション機能
- [ ] リクエストサイズ制限
- [ ] タイムアウト設定
- [ ] 異常検知アラート

## 6. 移行計画

### 6.1 Phase 1: 並行運用
- STDIO Transport を維持しつつ HTTP Transport を追加
- 設定で Transport を切り替え可能に

### 6.2 Phase 2: HTTP 本格運用
- 認証・認可の本番設定
- 監視・アラートの整備

### 6.3 Phase 3: STDIO 廃止（オプション）
- HTTP のみの運用に移行
- Claude Desktop 以外のクライアント対応完了
```

#### Step 7.2: 設計レビューチェックリスト

```
【HTTP Transport設計レビューチェックリスト】

□ エンドポイント設計
  □ RESTful な URL 設計か
  □ SSE エンドポイントの再接続戦略は定義されているか
  □ ヘルスチェックエンドポイントはあるか

□ 認証設計
  □ 認証方式は要件に適合しているか
  □ 認証情報の安全な保管方法は定義されているか
  □ トークン失効・ローテーション方針はあるか

□ エラーハンドリング
  □ MCP JSON-RPC エラーコードは網羅されているか
  □ エラーメッセージは適切に国際化されているか
  □ スタックトレースの漏洩防止は考慮されているか

□ セキュリティ
  □ OWASP Top 10 の対策は含まれているか
  □ TLS 設定は適切か（TLS 1.2以上）
  □ ログに機密情報は含まれていないか
```

### Phase 8: MCP HTTP Transport実装パターン

HTTP/SSE Transport のスキャフォールドコードを生成する。

#### Step 8.1: Spring Boot SSE実装テンプレート

```
【HTTP Transport Controller テンプレート】

/**
 * MCP HTTP/SSE Transportのコントローラ。
 *
 * <p>SSEストリームでMCPメッセージを配信し、
 * POSTエンドポイントでクライアントからのメッセージを受信する。</p>
 */
@RestController
@RequestMapping("/mcp")
public class McpHttpTransportController {

    private final McpServer mcpServer;
    private final SseEmitterRegistry emitterRegistry;

    /**
     * コンストラクタ。
     *
     * @param mcpServer MCPサーバーインスタンス
     * @param emitterRegistry SSEエミッター管理
     */
    public McpHttpTransportController(McpServer mcpServer,
            SseEmitterRegistry emitterRegistry) {
        this.mcpServer = mcpServer;
        this.emitterRegistry = emitterRegistry;
    }

    /**
     * SSE接続を確立する。
     *
     * <p>クライアントはこのエンドポイントに接続し、
     * サーバーからのメッセージをストリームで受信する。</p>
     *
     * @return SSEエミッター
     */
    @GetMapping(value = "/sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter connect() {
        SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);
        String sessionId = UUID.randomUUID().toString();

        emitter.onCompletion(() -> emitterRegistry.remove(sessionId));
        emitter.onTimeout(() -> emitterRegistry.remove(sessionId));
        emitter.onError(e -> emitterRegistry.remove(sessionId));

        emitterRegistry.register(sessionId, emitter);

        // 接続確立メッセージを送信
        try {
            emitter.send(SseEmitter.event()
                .name("connected")
                .data(Map.of("sessionId", sessionId)));
        } catch (IOException e) {
            emitter.completeWithError(e);
        }

        return emitter;
    }

    /**
     * クライアントからのMCPメッセージを受信する。
     *
     * @param sessionId SSEセッションID
     * @param request JSON-RPCリクエスト
     * @return JSON-RPCレスポンス
     */
    @PostMapping(value = "/message",
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<String> handleMessage(
            @RequestHeader("X-MCP-Session-Id") String sessionId,
            @RequestBody String request) {

        try {
            String response = mcpServer.handleMessage(request);

            // SSE経由でも通知（双方向通信用）
            emitterRegistry.sendToSession(sessionId, "message", response);

            return ResponseEntity.ok(response);
        } catch (McpException e) {
            return ResponseEntity.badRequest()
                .body(createErrorResponse(e));
        }
    }

    /**
     * ヘルスチェック。
     *
     * @return ヘルスステータス
     */
    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        return ResponseEntity.ok(Map.of(
            "status", "UP",
            "server", mcpServer.getServerInfo(),
            "timestamp", Instant.now().toString()
        ));
    }

    private String createErrorResponse(McpException e) {
        return String.format(
            "{\"jsonrpc\":\"2.0\",\"error\":{\"code\":%d,\"message\":\"%s\"},\"id\":null}",
            e.getCode(), e.getMessage());
    }
}
```

#### Step 8.2: SSEエミッター管理クラス

```
【SseEmitterRegistry テンプレート】

/**
 * SSEエミッターの登録・管理を行うレジストリ。
 *
 * <p>セッションIDとSSEエミッターのマッピングを管理し、
 * 特定セッションへのメッセージ送信機能を提供する。</p>
 */
@Component
public class SseEmitterRegistry {

    private final ConcurrentMap<String, SseEmitter> emitters =
        new ConcurrentHashMap<>();

    /**
     * SSEエミッターを登録する。
     *
     * @param sessionId セッションID
     * @param emitter SSEエミッター
     */
    public void register(String sessionId, SseEmitter emitter) {
        emitters.put(sessionId, emitter);
    }

    /**
     * SSEエミッターを削除する。
     *
     * @param sessionId セッションID
     */
    public void remove(String sessionId) {
        emitters.remove(sessionId);
    }

    /**
     * 特定セッションにメッセージを送信する。
     *
     * @param sessionId セッションID
     * @param eventName イベント名
     * @param data 送信データ
     */
    public void sendToSession(String sessionId, String eventName, Object data) {
        SseEmitter emitter = emitters.get(sessionId);
        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event()
                    .name(eventName)
                    .data(data));
            } catch (IOException e) {
                remove(sessionId);
            }
        }
    }

    /**
     * 全セッションにブロードキャストする。
     *
     * @param eventName イベント名
     * @param data 送信データ
     */
    public void broadcast(String eventName, Object data) {
        emitters.forEach((sessionId, emitter) -> {
            try {
                emitter.send(SseEmitter.event()
                    .name(eventName)
                    .data(data));
            } catch (IOException e) {
                remove(sessionId);
            }
        });
    }

    /**
     * 接続中のセッション数を返す。
     *
     * @return セッション数
     */
    public int getActiveSessionCount() {
        return emitters.size();
    }
}
```

#### Step 8.3: 認証フィルター

```
【API Key認証フィルター テンプレート】

/**
 * API Key認証フィルター。
 *
 * <p>X-MCP-API-Key ヘッダーを検証し、
 * 有効なAPI Keyを持つリクエストのみ許可する。</p>
 */
@Component
public class McpApiKeyAuthFilter extends OncePerRequestFilter {

    private final McpSecurityProperties securityProperties;

    public McpApiKeyAuthFilter(McpSecurityProperties securityProperties) {
        this.securityProperties = securityProperties;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {

        // ヘルスチェックは認証スキップ
        if (request.getRequestURI().equals("/mcp/health")) {
            filterChain.doFilter(request, response);
            return;
        }

        String apiKey = request.getHeader("X-MCP-API-Key");

        if (apiKey == null || apiKey.isBlank()) {
            sendError(response, HttpServletResponse.SC_UNAUTHORIZED,
                "API Key is required");
            return;
        }

        if (!securityProperties.isValidApiKey(apiKey)) {
            sendError(response, HttpServletResponse.SC_UNAUTHORIZED,
                "Invalid API Key");
            return;
        }

        filterChain.doFilter(request, response);
    }

    private void sendError(HttpServletResponse response, int status,
            String message) throws IOException {
        response.setStatus(status);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.getWriter().write(String.format(
            "{\"jsonrpc\":\"2.0\",\"error\":{\"code\":-32000,\"message\":\"%s\"},\"id\":null}",
            message));
    }
}

/**
 * MCPセキュリティ設定プロパティ。
 */
@ConfigurationProperties(prefix = "mcp.server.http.auth")
public class McpSecurityProperties {

    private String type = "api-key";
    private String headerName = "X-MCP-API-Key";
    private List<ApiKeyConfig> keys = new ArrayList<>();

    /**
     * API Keyが有効か検証する。
     *
     * @param apiKey 検証対象のAPI Key
     * @return 有効な場合true
     */
    public boolean isValidApiKey(String apiKey) {
        return keys.stream()
            .anyMatch(k -> k.getKey().equals(apiKey));
    }

    // getters/setters 省略

    public static class ApiKeyConfig {
        private String name;
        private String key;
        private String rateLimit;
        // getters/setters 省略
    }
}
```

#### Step 8.4: application.yml HTTP Transport設定

```
【HTTP Transport設定テンプレート】

# application.yml
spring:
  main:
    web-application-type: servlet  # HTTP Transport用（STDIOはnone）

server:
  port: 8080

mcp:
  server:
    name: "{server-name}"
    version: "{version}"
    transport: http  # stdio | http
    http:
      auth:
        type: api-key
        header-name: X-MCP-API-Key
        keys:
          - name: default
            key: ${MCP_API_KEY:dev-key-change-in-production}
            rate-limit: 100/min

# CORS設定
spring:
  web:
    cors:
      allowed-origins: ${MCP_CORS_ORIGINS:http://localhost:3000}
      allowed-methods: GET,POST
      allowed-headers: Content-Type,X-MCP-API-Key,X-MCP-Session-Id
```

### Phase 9: Nablarch MCP Tool実装パターン

Nablarch MCPサーバー向けの特化したTool実装スキャフォールドを生成する。

#### Step 9.1: Nablarch Tool実装テンプレート

```
【Nablarch Tool テンプレート】

/**
 * {Tool名} - Nablarch {機能領域} Tool。
 *
 * <p>{機能説明}。
 * Nablarchのコンポーネント設定やハンドラキュー情報を活用する。</p>
 */
@Component
public class {Name}Tool {

    private static final Logger LOGGER = LoggerFactory.getLogger({Name}Tool.class);

    private final {Name}KnowledgeProvider knowledgeProvider;

    /**
     * コンストラクタ。
     *
     * @param knowledgeProvider 知識プロバイダ
     */
    public {Name}Tool({Name}KnowledgeProvider knowledgeProvider) {
        this.knowledgeProvider = knowledgeProvider;
    }

    /**
     * {操作の説明}。
     *
     * <p>入力: {入力の説明}
     * 出力: {出力の説明}</p>
     *
     * @param param1 {パラメータ説明}
     * @param param2 {パラメータ説明}（オプション）
     * @return 処理結果のJSON文字列
     */
    @Tool(description = "{English description for MCP client}")
    public String {methodName}(
            @ToolParam(description = "{param1 description}") String param1,
            @ToolParam(description = "{param2 description}", required = false)
                String param2) {

        LOGGER.debug("{methodName} called with param1={}, param2={}",
            param1, param2);

        // 1. 入力バリデーション
        ValidationResult validation = validateInput(param1, param2);
        if (!validation.isValid()) {
            return createErrorResponse(validation.getErrorMessage());
        }

        try {
            // 2. 知識データの検索・フィルタリング
            List<{DataType}> results = knowledgeProvider.search(param1, param2);

            if (results.isEmpty()) {
                return createNotFoundResponse(param1);
            }

            // 3. 結果の整形
            return formatResults(results);

        } catch (Exception e) {
            LOGGER.error("{methodName} failed", e);
            return createErrorResponse("処理中にエラーが発生しました: " + e.getMessage());
        }
    }

    /**
     * 入力パラメータを検証する。
     */
    private ValidationResult validateInput(String param1, String param2) {
        if (param1 == null || param1.isBlank()) {
            return ValidationResult.invalid("param1は必須です");
        }
        // 追加のバリデーションルール
        return ValidationResult.valid();
    }

    /**
     * エラーレスポンスを生成する。
     */
    private String createErrorResponse(String message) {
        return String.format("""
            {
              "success": false,
              "error": "%s"
            }
            """, escapeJson(message));
    }

    /**
     * 検索結果なしレスポンスを生成する。
     */
    private String createNotFoundResponse(String query) {
        return String.format("""
            {
              "success": true,
              "count": 0,
              "message": "'%s' に該当する結果が見つかりませんでした",
              "suggestions": %s
            }
            """, escapeJson(query), getSuggestions(query));
    }

    /**
     * 検索結果を整形する。
     */
    private String formatResults(List<{DataType}> results) {
        StringBuilder sb = new StringBuilder();
        sb.append("{\n");
        sb.append("  \"success\": true,\n");
        sb.append("  \"count\": ").append(results.size()).append(",\n");
        sb.append("  \"results\": [\n");

        for (int i = 0; i < results.size(); i++) {
            sb.append(formatSingleResult(results.get(i)));
            if (i < results.size() - 1) {
                sb.append(",");
            }
            sb.append("\n");
        }

        sb.append("  ]\n");
        sb.append("}");
        return sb.toString();
    }

    private String escapeJson(String value) {
        return value.replace("\\", "\\\\")
                   .replace("\"", "\\\"")
                   .replace("\n", "\\n");
    }
}

/**
 * バリデーション結果。
 */
record ValidationResult(boolean valid, String errorMessage) {
    static ValidationResult valid() {
        return new ValidationResult(true, null);
    }
    static ValidationResult invalid(String message) {
        return new ValidationResult(false, message);
    }
    boolean isValid() {
        return valid;
    }
    String getErrorMessage() {
        return errorMessage;
    }
}
```

#### Step 9.2: Nablarch知識プロバイダテンプレート

```
【Nablarch KnowledgeProvider テンプレート】

/**
 * {領域名}の知識データプロバイダ。
 *
 * <p>knowledge/{yaml_file} から{データ種別}を読み込み、
 * 検索・フィルタリング機能を提供する。</p>
 */
@Component
public class {Name}KnowledgeProvider {

    private Map<String, {DataType}> dataMap;
    private List<String> allKeys;

    /**
     * 知識ファイルを読み込み、データを初期化する。
     */
    @PostConstruct
    public void init() {
        try (InputStream is = getClass().getResourceAsStream(
                "/knowledge/{yaml_file}")) {
            ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
            Map<String, Object> root = mapper.readValue(is,
                new TypeReference<Map<String, Object>>() {});

            this.dataMap = buildDataMap(root);
            this.allKeys = new ArrayList<>(dataMap.keySet());
            Collections.sort(this.allKeys);

        } catch (IOException e) {
            throw new UncheckedIOException(
                "{yaml_file}の読み込みに失敗しました", e);
        }
    }

    /**
     * キーワードで検索する。
     *
     * @param query 検索クエリ
     * @param filter フィルター条件（オプション）
     * @return 検索結果リスト
     */
    public List<{DataType}> search(String query, String filter) {
        String normalizedQuery = normalizeQuery(query);

        return dataMap.entrySet().stream()
            .filter(e -> matchesQuery(e.getKey(), e.getValue(), normalizedQuery))
            .filter(e -> matchesFilter(e.getValue(), filter))
            .map(Map.Entry::getValue)
            .collect(Collectors.toList());
    }

    /**
     * 全キーを取得する。
     *
     * @return キーリスト
     */
    public List<String> getAllKeys() {
        return Collections.unmodifiableList(allKeys);
    }

    /**
     * 類似キーワードを提案する（タイポ対策）。
     *
     * @param query 検索クエリ
     * @return 類似キーワードリスト
     */
    public List<String> suggestSimilar(String query) {
        return allKeys.stream()
            .filter(key -> calculateSimilarity(key, query) > 0.6)
            .limit(5)
            .collect(Collectors.toList());
    }

    private String normalizeQuery(String query) {
        return query.toLowerCase().trim();
    }

    private boolean matchesQuery(String key, {DataType} data, String query) {
        // キー名、説明、タグでマッチング
        return key.toLowerCase().contains(query)
            || data.getDescription().toLowerCase().contains(query)
            || data.getTags().stream()
                .anyMatch(tag -> tag.toLowerCase().contains(query));
    }

    private double calculateSimilarity(String s1, String s2) {
        // レーベンシュタイン距離ベースの類似度
        int distance = levenshteinDistance(s1.toLowerCase(), s2.toLowerCase());
        int maxLen = Math.max(s1.length(), s2.length());
        return 1.0 - ((double) distance / maxLen);
    }
}
```

#### Step 9.3: Nablarch Tool例外処理パターン

```
【Nablarch Tool例外クラス テンプレート】

/**
 * MCP Tool実行時の例外。
 */
public class McpToolException extends RuntimeException {

    private final int code;
    private final String details;

    /**
     * コンストラクタ。
     *
     * @param code エラーコード（JSON-RPC準拠）
     * @param message エラーメッセージ
     */
    public McpToolException(int code, String message) {
        this(code, message, null);
    }

    /**
     * コンストラクタ。
     *
     * @param code エラーコード
     * @param message エラーメッセージ
     * @param details 詳細情報
     */
    public McpToolException(int code, String message, String details) {
        super(message);
        this.code = code;
        this.details = details;
    }

    public int getCode() {
        return code;
    }

    public String getDetails() {
        return details;
    }

    /**
     * JSON-RPCエラーレスポンス形式で返す。
     */
    public String toJsonRpcError() {
        StringBuilder sb = new StringBuilder();
        sb.append("{\"code\":").append(code);
        sb.append(",\"message\":\"").append(escapeJson(getMessage())).append("\"");
        if (details != null) {
            sb.append(",\"data\":{\"details\":\"")
              .append(escapeJson(details)).append("\"}");
        }
        sb.append("}");
        return sb.toString();
    }

    private String escapeJson(String value) {
        return value.replace("\\", "\\\\")
                   .replace("\"", "\\\"")
                   .replace("\n", "\\n");
    }

    // よく使うエラーのファクトリメソッド
    public static McpToolException invalidParams(String message) {
        return new McpToolException(-32602, message);
    }

    public static McpToolException notFound(String resource) {
        return new McpToolException(-32001, resource + " が見つかりません");
    }

    public static McpToolException internal(String message, Throwable cause) {
        return new McpToolException(-32603, message,
            cause != null ? cause.getMessage() : null);
    }
}
```

### Phase 10: MCP Toolテストパターン

MockMvc、WireMock、JUnit5を組み合わせたToolテストスキャフォールドを生成する。

#### Step 10.1: 単体テストテンプレート（JUnit5 + Mockito）

```
【Tool単体テスト テンプレート】

/**
 * {Name}Tool の単体テスト。
 *
 * <p>KnowledgeProviderをモック化し、Tool単体の動作を検証する。</p>
 */
@ExtendWith(MockitoExtension.class)
class {Name}ToolTest {

    @Mock
    private {Name}KnowledgeProvider knowledgeProvider;

    @InjectMocks
    private {Name}Tool tool;

    @Test
    @DisplayName("正常系: 検索結果あり")
    void methodName_正常系_検索結果あり() {
        // Arrange
        List<{DataType}> mockResults = List.of(
            createTestData("key1", "description1"),
            createTestData("key2", "description2")
        );
        when(knowledgeProvider.search("query", null))
            .thenReturn(mockResults);

        // Act
        String result = tool.{methodName}("query", null);

        // Assert
        assertThat(result).contains("\"success\": true");
        assertThat(result).contains("\"count\": 2");
        verify(knowledgeProvider).search("query", null);
    }

    @Test
    @DisplayName("正常系: 検索結果なし")
    void methodName_正常系_検索結果なし() {
        // Arrange
        when(knowledgeProvider.search("unknown", null))
            .thenReturn(Collections.emptyList());
        when(knowledgeProvider.suggestSimilar("unknown"))
            .thenReturn(List.of("known1", "known2"));

        // Act
        String result = tool.{methodName}("unknown", null);

        // Assert
        assertThat(result).contains("\"success\": true");
        assertThat(result).contains("\"count\": 0");
        assertThat(result).contains("suggestions");
    }

    @Test
    @DisplayName("異常系: 必須パラメータnull")
    void methodName_異常系_param1がnull() {
        // Act
        String result = tool.{methodName}(null, null);

        // Assert
        assertThat(result).contains("\"success\": false");
        assertThat(result).contains("param1は必須です");
        verifyNoInteractions(knowledgeProvider);
    }

    @Test
    @DisplayName("異常系: 必須パラメータ空文字")
    void methodName_異常系_param1が空文字() {
        // Act
        String result = tool.{methodName}("", null);

        // Assert
        assertThat(result).contains("\"success\": false");
        assertThat(result).contains("param1は必須です");
    }

    @Test
    @DisplayName("異常系: 内部エラー発生")
    void methodName_異常系_内部エラー() {
        // Arrange
        when(knowledgeProvider.search(anyString(), any()))
            .thenThrow(new RuntimeException("DB connection failed"));

        // Act
        String result = tool.{methodName}("query", null);

        // Assert
        assertThat(result).contains("\"success\": false");
        assertThat(result).contains("処理中にエラーが発生しました");
    }

    private {DataType} createTestData(String key, String description) {
        // テストデータ生成ヘルパー
        return new {DataType}(key, description, List.of("tag1"));
    }
}
```

#### Step 10.2: 統合テストテンプレート（MockMvc）

```
【Tool統合テスト テンプレート（MockMvc）】

/**
 * {Name}Tool の統合テスト。
 *
 * <p>Spring BootコンテキストとMockMvcを使用し、
 * MCPプロトコル経由でのTool呼び出しを検証する。</p>
 */
@SpringBootTest
@AutoConfigureMockMvc
class {Name}ToolIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("MCP tools/call: 正常系")
    void toolsCall_正常系() throws Exception {
        // Arrange
        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {
                "name": "{tool-name}",
                "arguments": {
                  "param1": "query"
                }
              },
              "id": "test-001"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .header("X-MCP-API-Key", "test-api-key")
                .header("X-MCP-Session-Id", "test-session")
                .content(request))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.jsonrpc").value("2.0"))
            .andExpect(jsonPath("$.id").value("test-001"))
            .andExpect(jsonPath("$.result").exists())
            .andExpect(jsonPath("$.error").doesNotExist());
    }

    @Test
    @DisplayName("MCP tools/call: 必須パラメータ不足")
    void toolsCall_異常系_パラメータ不足() throws Exception {
        // Arrange
        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {
                "name": "{tool-name}",
                "arguments": {}
              },
              "id": "test-002"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .header("X-MCP-API-Key", "test-api-key")
                .header("X-MCP-Session-Id", "test-session")
                .content(request))
            .andExpect(status().isOk())  // JSON-RPCはHTTP 200でエラーを返す
            .andExpect(jsonPath("$.error").exists())
            .andExpect(jsonPath("$.error.code").value(-32602));
    }

    @Test
    @DisplayName("認証エラー: API Keyなし")
    void toolsCall_異常系_認証エラー() throws Exception {
        // Arrange
        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {"name": "{tool-name}", "arguments": {}}
              "id": "test-003"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .content(request))
            .andExpect(status().isUnauthorized());
    }
}
```

#### Step 10.3: 外部API統合テストテンプレート（WireMock）

```
【WireMock統合テスト テンプレート】

/**
 * {Name}Tool の外部API統合テスト。
 *
 * <p>WireMockで外部APIをスタブ化し、
 * 外部連携を含むTool動作を検証する。</p>
 */
@SpringBootTest
@AutoConfigureMockMvc
@WireMockTest(httpPort = 8089)
class {Name}ToolExternalApiTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("外部API呼び出し: 正常系")
    void externalApiCall_正常系() throws Exception {
        // Arrange: WireMockで外部APIをスタブ化
        stubFor(get(urlEqualTo("/api/v1/data?q=query"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""
                    {
                      "results": [
                        {"id": "1", "name": "Result 1"},
                        {"id": "2", "name": "Result 2"}
                      ]
                    }
                    """)));

        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {
                "name": "{tool-name}",
                "arguments": {"query": "query"}
              },
              "id": "test-001"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .header("X-MCP-API-Key", "test-api-key")
                .header("X-MCP-Session-Id", "test-session")
                .content(request))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.result").exists());

        // WireMockリクエスト検証
        verify(getRequestedFor(urlEqualTo("/api/v1/data?q=query")));
    }

    @Test
    @DisplayName("外部API呼び出し: タイムアウト")
    void externalApiCall_異常系_タイムアウト() throws Exception {
        // Arrange: 遅延レスポンス
        stubFor(get(urlEqualTo("/api/v1/data?q=slow"))
            .willReturn(aResponse()
                .withStatus(200)
                .withFixedDelay(5000)));  // 5秒遅延

        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {
                "name": "{tool-name}",
                "arguments": {"query": "slow"}
              },
              "id": "test-002"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .header("X-MCP-API-Key", "test-api-key")
                .header("X-MCP-Session-Id", "test-session")
                .content(request))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.error").exists())
            .andExpect(jsonPath("$.error.code").value(-32603));
    }

    @Test
    @DisplayName("外部API呼び出し: 503エラー → リトライ")
    void externalApiCall_異常系_リトライ成功() throws Exception {
        // Arrange: 最初は503、2回目で成功
        stubFor(get(urlEqualTo("/api/v1/data?q=retry"))
            .inScenario("Retry")
            .whenScenarioStateIs(Scenario.STARTED)
            .willReturn(aResponse().withStatus(503))
            .willSetStateTo("First Failure"));

        stubFor(get(urlEqualTo("/api/v1/data?q=retry"))
            .inScenario("Retry")
            .whenScenarioStateIs("First Failure")
            .willReturn(aResponse()
                .withStatus(200)
                .withBody("{\"results\": []}")));

        String request = """
            {
              "jsonrpc": "2.0",
              "method": "tools/call",
              "params": {
                "name": "{tool-name}",
                "arguments": {"query": "retry"}
              },
              "id": "test-003"
            }
            """;

        // Act & Assert
        mockMvc.perform(post("/mcp/message")
                .contentType(MediaType.APPLICATION_JSON)
                .header("X-MCP-API-Key", "test-api-key")
                .header("X-MCP-Session-Id", "test-session")
                .content(request))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.result").exists());

        // 2回呼び出されたことを検証
        verify(2, getRequestedFor(urlEqualTo("/api/v1/data?q=retry")));
    }
}
```

#### Step 10.4: テスト設定ファイル

```
【テスト設定テンプレート】

# src/test/resources/application-test.yml
spring:
  main:
    web-application-type: servlet

mcp:
  server:
    name: "test-mcp-server"
    version: "0.0.1-TEST"
    transport: http
    http:
      auth:
        type: api-key
        keys:
          - name: test
            key: test-api-key

# WireMock外部API設定
external:
  api:
    base-url: http://localhost:8089/api/v1

# テスト用タイムアウト短縮
mcp:
  tool:
    timeout: 2000  # 2秒
```

## Input Format

```yaml
# スキャフォールド生成リクエスト
project_root: "/path/to/project"          # プロジェクトルートパス
base_package: "com.example.mcp"           # Javaベースパッケージ
domain: "nablarch"                        # ドメイン名（URI prefix等に使用）
spring_boot_version: "3.4.2"              # Spring Bootバージョン
spring_ai_version: "1.0.0"               # Spring AI BOMバージョン

# 生成するプリミティブ
primitives:
  resources:
    - name: "HandlerResource"
      keys: ["web", "rest", "batch"]
      description: "ハンドラカタログリソース"
      yaml_file: "handler-catalog.yaml"
  prompts:
    - name: "SetupHandlerQueue"
      params:
        - name: "app_type"
          description: "アプリケーション種別"
          required: true
      yaml_files: ["handler-catalog.yaml", "handler-constraints.yaml"]
  tools:
    - name: "SearchApi"
      methods:
        - name: "searchApi"
          description: "Search Nablarch API documentation"
          params:
            - name: "query"
              description: "検索クエリ"

# HTTP Transport設定（オプション、Phase 7-8で使用）
http_transport:
  enabled: true                           # HTTP Transport生成の有無
  port: 8080                              # デフォルトポート
  auth:
    type: "api-key"                       # api-key | jwt | mtls | oauth2
    header_name: "X-MCP-API-Key"          # 認証ヘッダー名
  cors:
    allowed_origins: ["http://localhost:3000"]
  generate_design_doc: true               # 設計ドキュメント生成の有無

# Nablarch Tool拡張設定（オプション、Phase 9で使用）
nablarch_tools:
  enabled: true                           # Nablarch Tool拡張生成の有無
  knowledge_provider: true                # KnowledgeProvider生成
  validation_pattern: true                # ValidationResultパターン生成
  exception_handling: true                # McpToolException生成

# テストパターン設定（オプション、Phase 10で使用）
test_patterns:
  unit_test: true                         # JUnit5 + Mockito単体テスト
  integration_test: true                  # MockMvc統合テスト
  wiremock_test: false                    # WireMock外部API統合テスト（外部連携がある場合のみ）
```

## Output Format

```
# 生成されるファイル一覧

{project_root}/
├── src/main/java/{base_package}/
│   ├── config/McpServerConfig.java       # @Configuration Bean定義
│   ├── resources/{Name}ResourceProvider.java
│   ├── prompts/{Name}Prompt.java
│   ├── tools/{Name}Tool.java
│   ├── tools/{Name}KnowledgeProvider.java    # (nablarch_tools.enabled時)
│   ├── tools/McpToolException.java           # (nablarch_tools.enabled時)
│   ├── transport/McpHttpTransportController.java  # (http_transport.enabled時)
│   ├── transport/SseEmitterRegistry.java          # (http_transport.enabled時)
│   └── security/McpApiKeyAuthFilter.java          # (http_transport.enabled時)
├── src/main/resources/
│   ├── knowledge/{category}.yaml         # 知識YAMLスケルトン
│   ├── application.properties            # MCP Server設定（STDIO）
│   └── application-http.yml              # HTTP Transport設定（http_transport.enabled時）
├── src/test/java/{base_package}/
│   ├── resources/{Name}ResourceProviderTest.java
│   ├── prompts/{Name}PromptTest.java
│   ├── tools/{Name}ToolTest.java                  # 単体テスト
│   ├── tools/{Name}ToolIntegrationTest.java       # (test_patterns.integration_test時)
│   └── tools/{Name}ToolExternalApiTest.java       # (test_patterns.wiremock_test時)
├── docs/
│   └── http-transport-design.md          # (http_transport.generate_design_doc時)
└── build.gradle.kts                      # 依存関係設定
```

**各ファイルの内容:**
- **McpServerConfig.java**: 全プリミティブの@Bean定義を集約。ヘルパーメソッド含む
- **ResourceProvider**: @Component + @PostConstruct。知識YAML読み込み + Markdown変換
- **Prompt**: @Component + @PostConstruct。引数バリデーション + 知識データ参照 + 結果生成
- **Tool**: @Component + @Tool。ツールメソッド実装
- **KnowledgeProvider**: ToolからYAML知識を検索・取得するプロバイダ
- **McpToolException**: JSON-RPCエラーコード対応の例外クラス
- **McpHttpTransportController**: SSE/POSTエンドポイント実装
- **SseEmitterRegistry**: SSEセッション管理
- **McpApiKeyAuthFilter**: API Key認証フィルター
- **知識YAML**: スキーマに従ったスケルトン（データは別途S-017で充填）
- **テスト**: 正常系・異常系・コンテンツ検証の3カテゴリ
- **統合テスト**: MockMvcによるMCPプロトコル経由の検証
- **外部API統合テスト**: WireMockによる外部API連携の検証
- **設計ドキュメント**: HTTP Transport移行計画・認証設計・エラーハンドリング設計

## Examples

### Example 1: Nablarch MCP Server Phase 1 の再現

```
【入力】
project_root: "nablarch-mcp-server"
base_package: "com.tis.nablarch.mcp"
domain: "nablarch"
spring_boot_version: "3.4.2"
spring_ai_version: "1.0.0"
primitives:
  resources:
    - name: "HandlerResource"
      keys: [web, rest, batch, messaging, http-messaging, jakarta-batch]
    - name: "GuideResource"
      keys: [setup, testing, validation, database, handler-queue, error-handling]
  prompts:
    - name: "SetupHandlerQueue"
      params: [{name: app_type, required: true}]
    - name: "CreateAction"
      params: [{name: app_type, required: true}, {name: action_name, required: true}]
    # ... 4 more prompts
  tools:
    - name: "SearchApi"
    - name: "ValidateHandlerQueue"

【生成結果】
- McpServerConfig.java: 3 @Bean メソッド (tools, resources, prompts)
- 2 ResourceProvider + 6 Prompt + 2 Tool クラス = 10 実装クラス
- 7 知識YAMLスケルトン
- 10 テストクラス
- build.gradle.kts 依存関係
- application.properties MCP設定
```

### Example 2: 新規Resource追加（既存プロジェクトへの追加）

```
【入力】
project_root: "nablarch-mcp-server"  # 既存プロジェクト
primitives:
  resources:
    - name: "ModuleResource"
      keys: [core, web, batch, testing]
      yaml_file: "module-catalog.yaml"

【実行フロー】
Phase 1: 既存McpServerConfig.javaを読み込み、nablarchResources()の構造を確認
Phase 2: ModuleResourceProvider.java を生成（@Component + @PostConstruct）
Phase 3-4: スキップ（Resource追加のみ）
Phase 5: ModuleResourceProviderTest.java を生成
Phase 6: McpServerConfig.java の nablarchResources() にModuleResource追加
         ビルド・テスト実行

【差分】
+ ModuleResourceProvider.java（新規）
+ ModuleResourceProviderTest.java（新規）
~ McpServerConfig.java（nablarchResources() に4エントリ追加）
```

### Example 3: Spring Boot向けMCPサーバー新規構築

```
【入力】
project_root: "spring-mcp-server"
base_package: "com.example.spring.mcp"
domain: "spring"
spring_boot_version: "3.4.2"
spring_ai_version: "1.0.0"
primitives:
  resources:
    - name: "StarterResource"
      keys: [web, data-jpa, security, actuator]
      yaml_file: "starter-catalog.yaml"
  prompts:
    - name: "SetupProject"
      params: [{name: project_type, required: true}]
  tools:
    - name: "AnalyzeConfig"

【実行フロー】
Phase 1: 新規プロジェクト → 全ファイルを新規生成
Phase 2: StarterResourceProvider.java（starter-catalog.yaml参照）
Phase 3: SetupProjectPrompt.java（@Component + @PostConstruct）
Phase 4: AnalyzeConfigTool.java（@Component + @Tool）
Phase 5: build.gradle.kts, application.properties, テスト3クラス生成
Phase 6: ./gradlew clean build で動作確認

【生成ファイル数】
- 実装クラス: 4（Config + Provider + Prompt + Tool）
- テストクラス: 3
- 設定ファイル: 3（build.gradle.kts, application.properties, starter-catalog.yaml）
- 合計: 10ファイル
```

### Example 4: HTTP Transport移行（既存STDIOプロジェクトの拡張）

```
【入力】
project_root: "nablarch-mcp-server"
http_transport:
  enabled: true
  port: 8080
  auth:
    type: api-key
    header_name: X-MCP-API-Key
  cors:
    allowed_origins: ["http://localhost:3000", "https://app.example.com"]
  generate_design_doc: true

【実行フロー】
Phase 7: HTTP Transport設計ドキュメント生成（docs/http-transport-design.md）
Phase 8: HTTP Transport実装スキャフォールド生成
  - McpHttpTransportController.java
  - SseEmitterRegistry.java
  - McpApiKeyAuthFilter.java
  - McpSecurityProperties.java
  - application-http.yml

【生成ファイル数】
- 実装クラス: 4（Controller + Registry + Filter + Properties）
- 設定ファイル: 1（application-http.yml）
- ドキュメント: 1（http-transport-design.md）
- 合計: 6ファイル

【使い方】
1. application.propertiesに spring.profiles.active=http を追加
2. 環境変数 MCP_API_KEY でAPI Keyを設定
3. ./gradlew bootRun でHTTPモードで起動
4. curl -H "X-MCP-API-Key: $MCP_API_KEY" http://localhost:8080/mcp/health で動作確認
```

### Example 5: Nablarch Tool拡張（知識プロバイダ + テストパターン）

```
【入力】
project_root: "nablarch-mcp-server"
base_package: "com.tis.nablarch.mcp"
primitives:
  tools:
    - name: "AnalyzeConfig"
      methods:
        - name: "analyzeConfig"
          description: "Analyze Nablarch component configuration XML"
          params:
            - name: "config_path"
              description: "設定ファイルパス"
nablarch_tools:
  enabled: true
  knowledge_provider: true
  validation_pattern: true
  exception_handling: true
test_patterns:
  unit_test: true
  integration_test: true
  wiremock_test: false

【実行フロー】
Phase 4: Tool基本クラス生成
Phase 9: Nablarch Tool拡張
  - AnalyzeConfigTool.java（拡張版: ValidationResult使用）
  - AnalyzeConfigKnowledgeProvider.java
  - McpToolException.java
Phase 10: テストパターン生成
  - AnalyzeConfigToolTest.java（JUnit5 + Mockito）
  - AnalyzeConfigToolIntegrationTest.java（MockMvc）

【生成ファイル数】
- 実装クラス: 3（Tool + KnowledgeProvider + Exception）
- テストクラス: 2（単体 + 統合）
- 合計: 5ファイル

【Nablarch Tool拡張の特徴】
- ValidationResult record による入力検証パターン
- McpToolException によるJSON-RPC準拠エラーハンドリング
- KnowledgeProvider による知識データ検索・類似語提案
- 統一されたJSON形式レスポンス（success/count/results構造）
```

## Guidelines

### 必須ルール

1. **Javadocは全て日本語で記述すること**
   - クラス・メソッド・パラメータのJavadocは日本語
   - @Tool, @ToolParam のdescription は英語（MCPクライアントが読むため）
   - コード内コメントは日本語

2. **McpServerConfig.java は1ファイルに集約すること**
   - 全プリミティブ（Tool/Resource/Prompt）の@Beanを1つの@Configurationクラスに配置
   - プリミティブの種類ごとに1つの@Bean メソッド
   - ヘルパーメソッド（createResourceSpec, promptSpec, arg）はprivate static

3. **MCP SDK型を正しく使用すること**
   - Resource: `McpServerFeatures.SyncResourceSpecification`, `McpSchema.Resource`, `McpSchema.ReadResourceResult`, `McpSchema.TextResourceContents`
   - Prompt: `McpServerFeatures.SyncPromptSpecification`, `McpSchema.Prompt`, `McpSchema.PromptArgument`, `McpSchema.GetPromptResult`, `McpSchema.PromptMessage`, `McpSchema.Role.USER`, `McpSchema.TextContent`
   - Tool: `@Tool`, `@ToolParam` アノテーション、`ToolCallbackProvider`, `MethodToolCallbackProvider`

4. **@Component + @PostConstruct パターンを使用すること**
   - Prompt/ResourceProvider は @Component で Spring管理Bean化
   - 知識ファイルの読み込みは @PostConstruct で初期化時に実行
   - コンストラクタインジェクションではなく @PostConstruct を使用（知識ファイル読み込みの遅延初期化のため）

5. **引数バリデーションを実装すること**
   - Prompt の execute() メソッドで必須引数のnull/空文字チェック
   - 不正な引数には IllegalArgumentException をスロー
   - エラーメッセージは日本語で具体的に記述

6. **Java 17互換を維持すること**
   - `getFirst()` は使用不可（Java 21以降）→ `get(0)` を使用
   - テキストブロック（"""）は使用可能（Java 15以降）
   - record は使用可能（Java 16以降）

7. **HTTP Transportではセキュリティを必須とすること**
   - 認証なしのHTTPエンドポイント公開は禁止
   - 最低でもAPI Key認証を実装
   - 本番環境ではHTTPS必須
   - CORS設定は明示的に許可オリジンを指定

8. **SSEエンドポイントの再接続戦略を実装すること**
   - クライアント側の自動再接続を考慮した設計
   - セッションIDによる状態管理
   - 切断時のリソース解放（onCompletion, onTimeout, onError）

9. **MCP Toolのエラーハンドリングは統一パターンを使用すること**
   - JSON形式のレスポンス（success/error構造）
   - JSON-RPCエラーコード準拠（-32000〜-32099のアプリケーションエラー）
   - エラーメッセージは日本語で具体的に記述
   - スタックトレースは本番環境で漏洩させない

10. **テストはSpringコンテキストなしで動作させること**
   - `new XxxPrompt(); prompt.init();` パターンで直接インスタンス化
   - @SpringBootTest は Tool テストでのみ使用（DI が必要な場合）
   - テストメソッド名は日本語で記述（`execute_正常系_web指定`）

### 効率化のコツ

1. **既存コードのコピー＆修正**
   - nablarch-mcp-server の実装を直接参考にする
   - 特にMcpServerConfig.javaのヘルパーメソッドはそのまま再利用可能

2. **プリミティブ種別ごとの並列生成**
   - Resource/Prompt/Tool は独立しているため並列で生成可能
   - McpServerConfig.java は最後に統合

3. **知識YAMLはスケルトンのみ**
   - スキャフォールドではYAMLの構造（スキーマ）のみ生成
   - 実データの充填はS-017（nablarch-knowledge-builder）で実施

4. **テスト駆動のアプローチ**
   - テストクラスを先に生成し、実装が正しいことを確認しながら進める
   - 各Promptに最低10テスト（正常系+異常系+コンテンツ検証）

### アンチパターン（避けるべきこと）

1. **MCP SDK型の誤用**
   - `McpSchema.Role.ASSISTANT` をPrompt結果に使用する（正しくは `Role.USER`）
   - `SyncResourceSpecification` と `SyncPromptSpecification` を混同する
   - 対策: nablarch-mcp-server の McpServerConfig.java を常に参考にする

2. **ヘルパーメソッドの重複定義**
   - promptSpec() や arg() を複数の@Configuration クラスに定義する
   - 対策: McpServerConfig.java 1ファイルに集約

3. **引数のnull透過**
   - Prompt の execute() で引数のnullチェックを省略し、NullPointerException が発生する
   - 対策: 全必須引数に対して null/blank チェックを実装

4. **知識ファイルのハードコードパス**
   - `/absolute/path/to/knowledge/file.yaml` のように絶対パスを使用する
   - 対策: `getClass().getResourceAsStream("/knowledge/{file}")` でクラスパスから読み込む

5. **テストでのSpringコンテキスト濫用**
   - 全テストクラスに @SpringBootTest を付与する（テスト実行が遅くなる）
   - 対策: 直接インスタンス化 + init() で十分なクラスは @SpringBootTest を使用しない

6. **Java 21 API の使用**
   - `List.getFirst()`, `Map.of()` のnull引数等、Java 17で非対応のAPIを使用する
   - 対策: Java 17互換を常に意識。`get(0)` を使用する

7. **McpServerConfigの分割**
   - Resource用Config, Prompt用Config, Tool用Config のように分割する
   - 対策: 1つの@Configuration クラスに全プリミティブを集約（nablarch-mcp-serverの実績パターン）

8. **HTTP Transportでの認証省略**
   - ヘルスチェック以外のエンドポイントで認証をスキップする
   - 対策: McpApiKeyAuthFilter で全エンドポイントをカバー

9. **SSEエミッターのリソースリーク**
   - 切断時にSseEmitterRegistryから削除しない
   - 対策: onCompletion, onTimeout, onError で必ず remove() を呼ぶ

10. **WireMockポートの競合**
    - 固定ポートでテストを実行し、CI環境で競合する
    - 対策: @WireMockTest でランダムポート or 環境変数で指定

11. **Toolレスポンスの形式不統一**
    - 成功時はテキスト、失敗時はJSON、のように形式が混在する
    - 対策: 常にJSON形式（`{success, count/error, results/suggestions}`）で返却
