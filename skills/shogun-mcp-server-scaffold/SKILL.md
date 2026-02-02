---
name: mcp-server-scaffold
description: MCP（Model Context Protocol）サーバーのResource/Prompt/Tool登録コードをスキャフォールド自動生成するスキル。Spring AI MCP Server + Spring Boot 3.xを前提に、McpServerConfig Bean定義テンプレート、@Component Prompt/Toolクラス、ResourceProvider実装、知識YAMLファイル、テストクラスを一括生成する。「新しいMCP Resourceを追加して」「Promptテンプレートを生成して」「MCP Toolのスキャフォールドを作って」といった要望に対応する。
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
- MCP ServerのResource/Prompt/Toolを新規追加する必要がある場合
- Spring AI MCP Serverプロジェクトを新規作成する場合
- 既存のMCPサーバーに新しいプリミティブを追加する場合

**トリガーキーワード**: MCP Server, Resource登録, Prompt登録, Tool登録, スキャフォールド, McpServerConfig, Spring AI MCP

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
```

## Output Format

```
# 生成されるファイル一覧

{project_root}/
├── src/main/java/{base_package}/
│   ├── config/McpServerConfig.java       # @Configuration Bean定義
│   ├── resources/{Name}ResourceProvider.java
│   ├── prompts/{Name}Prompt.java
│   └── tools/{Name}Tool.java
├── src/main/resources/
│   ├── knowledge/{category}.yaml         # 知識YAMLスケルトン
│   └── application.properties            # MCP Server設定
├── src/test/java/{base_package}/
│   ├── resources/{Name}ResourceProviderTest.java
│   ├── prompts/{Name}PromptTest.java
│   └── tools/{Name}ToolTest.java
└── build.gradle.kts                      # 依存関係設定
```

**各ファイルの内容:**
- **McpServerConfig.java**: 全プリミティブの@Bean定義を集約。ヘルパーメソッド含む
- **ResourceProvider**: @Component + @PostConstruct。知識YAML読み込み + Markdown変換
- **Prompt**: @Component + @PostConstruct。引数バリデーション + 知識データ参照 + 結果生成
- **Tool**: @Component + @Tool。ツールメソッド実装
- **知識YAML**: スキーマに従ったスケルトン（データは別途S-017で充填）
- **テスト**: 正常系・異常系・コンテンツ検証の3カテゴリ

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

7. **テストはSpringコンテキストなしで動作させること**
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
