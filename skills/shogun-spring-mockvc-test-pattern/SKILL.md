---
name: spring-mockvc-test-pattern
description: Spring Boot MockMvc テストパターンを提供するスキル。MCP ToolのHTTPエンドポイントテスト、コントローラーテスト、統合テストのテンプレートを生成する。「MockMvcのテストパターンを教えて」「Spring BootのコントローラーテストをMockMvcで書いて」「MCPエンドポイントのテストコードを生成して」「REST APIのテストテンプレートを作って」といった要望に対応する。
---

# Spring Boot MockMvc Test Pattern

## Overview

Spring BootアプリケーションにおけるHTTPエンドポイント（RESTコントローラー、MCPサーバーのTool等）のテストを、MockMvcを使って効率的に実装するためのテストパターン集を提供するスキル。

**MockMvcの特徴:**

| 項目 | 説明 |
|------|------|
| 目的 | サーブレットコンテナを起動せずにSpring MVCをテスト |
| 利点 | 高速な実行、HTTPリクエスト/レスポンスの完全検証 |
| 用途 | コントローラー単体テスト、統合テスト |

**テスト分類:**

| テスト種別 | アノテーション | 用途 |
|------------|----------------|------|
| コントローラー単体テスト | `@WebMvcTest` | コントローラーのみを軽量にテスト |
| 統合テスト | `@SpringBootTest` + `@AutoConfigureMockMvc` | アプリケーション全体をテスト |
| セキュリティテスト | `@WebMvcTest` + `@WithMockUser` | 認証・認可のテスト |

**本スキルが提供するもの:**

1. **基本テストパターン**: GET/POST/PUT/DELETE の各HTTPメソッド
2. **認証テストパターン**: Spring Securityとの連携
3. **エラーケーステスト**: バリデーションエラー、404、500等
4. **JSONレスポンス検証**: JSONPath による詳細検証
5. **MCP Tool テスト**: MCPサーバーのToolエンドポイントテスト
6. **WireMock連携**: 外部APIのモック化

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「MockMvcのテストパターンを教えて」
- 「Spring BootのコントローラーテストをMockMvcで書いて」
- 「MCPエンドポイントのテストコードを生成して」
- 「REST APIのテストテンプレートを作って」
- 「@WebMvcTestの使い方を教えて」
- 「MockMvcでJSON検証する方法は？」
- 「認証付きAPIのテストを書きたい」
- 「WireMockとMockMvcを組み合わせたい」
- 「ファイルアップロードAPIのテストを書きたい」
- Spring Boot REST APIのテストコードを生成する必要がある場合
- MCPサーバーのToolをHTTP経由でテストする必要がある場合
- 既存のコントローラーにテストを追加する必要がある場合

**トリガーキーワード**: MockMvc, @WebMvcTest, コントローラーテスト, REST API テスト, Spring Boot テスト, JSONPath, @WithMockUser, WireMock

## Instructions

### Phase 1: テスト対象の分析

#### Step 1.1: コントローラーの確認

```
【分析手順】

1. テスト対象のコントローラークラスを特定
   Grep: "@RestController" OR "@Controller"

2. エンドポイントのマッピングを確認
   - @GetMapping, @PostMapping, @PutMapping, @DeleteMapping
   - @RequestMapping のパス
   - パスパラメータ（@PathVariable）
   - クエリパラメータ（@RequestParam）
   - リクエストボディ（@RequestBody）

3. レスポンス形式を確認
   - ResponseEntity<?> の戻り値型
   - @ResponseStatus アノテーション
   - エラーハンドリング（@ExceptionHandler）

4. 依存サービスを確認
   - @Autowired されているサービスクラス
   - モック化が必要な依存関係
```

#### Step 1.2: テストケース設計

```
【テストケース設計テンプレート】

□ 正常系テストケース
  □ 必須パラメータのみ指定
  □ オプションパラメータ全指定
  □ 境界値（最大長、最小値等）

□ 異常系テストケース
  □ 必須パラメータ欠落
  □ 不正な型のパラメータ
  □ バリデーションエラー
  □ 存在しないリソースへのアクセス（404）
  □ 認証・認可エラー（401, 403）

□ エッジケース
  □ 空文字列
  □ null値
  □ 特殊文字を含む値
  □ 日本語を含む値
```

### Phase 2: MockMvc 基本設定

#### Step 2.1: @WebMvcTest（コントローラー単体テスト）

```java
/**
 * コントローラー単体テストの基本設定。
 *
 * <p>対象コントローラーのみをロードし、依存サービスはモック化する。</p>
 */
@WebMvcTest(TargetController.class)
class TargetControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private TargetService targetService;  // 依存サービスをモック化

    @Autowired
    private ObjectMapper objectMapper;  // JSON変換用

    // テストメソッド...
}
```

**設定ポイント:**
- `@WebMvcTest(対象クラス)` でテスト対象を限定
- `@MockBean` で依存サービスをモック化
- `ObjectMapper` はJSONシリアライズ/デシリアライズに使用
- コントローラーのみをロードするため高速

#### Step 2.2: @SpringBootTest + @AutoConfigureMockMvc（統合テスト）

```java
/**
 * 統合テストの基本設定。
 *
 * <p>アプリケーション全体をロードし、実際のサービス層も含めてテストする。</p>
 */
@SpringBootTest
@AutoConfigureMockMvc
@Transactional  // テスト後にDBロールバック
class TargetControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    // 実際のサービス・リポジトリが注入される
    @Autowired
    private TargetService targetService;

    // テストメソッド...
}
```

**設定ポイント:**
- `@SpringBootTest` でアプリケーション全体をロード
- `@AutoConfigureMockMvc` でMockMvc自動設定
- `@Transactional` でテスト後にDBロールバック（テストデータ汚染防止）
- 実際のDBアクセスを含む場合は `@TestPropertySource` でテストDB設定

### Phase 3: HTTPメソッド別テストパターン

#### Step 3.1: GET リクエストテスト

```java
/**
 * GETリクエストのテストパターン。
 */

// パターン1: 単純なGET
@Test
void getResource_正常系_リソース取得成功() throws Exception {
    // Arrange
    TargetDto expected = new TargetDto(1L, "test");
    when(targetService.findById(1L)).thenReturn(expected);

    // Act & Assert
    mockMvc.perform(get("/api/resources/{id}", 1L)
                    .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("test"));
}

// パターン2: クエリパラメータ付きGET
@Test
void searchResources_正常系_検索成功() throws Exception {
    // Arrange
    List<TargetDto> expected = List.of(
            new TargetDto(1L, "test1"),
            new TargetDto(2L, "test2"));
    when(targetService.search("keyword", 0, 10)).thenReturn(expected);

    // Act & Assert
    mockMvc.perform(get("/api/resources")
                    .param("query", "keyword")
                    .param("page", "0")
                    .param("size", "10")
                    .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2))
            .andExpect(jsonPath("$[0].name").value("test1"));
}

// パターン3: リソースが存在しない場合（404）
@Test
void getResource_異常系_リソースが存在しない() throws Exception {
    // Arrange
    when(targetService.findById(999L))
            .thenThrow(new ResourceNotFoundException("Resource not found"));

    // Act & Assert
    mockMvc.perform(get("/api/resources/{id}", 999L)
                    .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.error").value("Resource not found"));
}
```

#### Step 3.2: POST リクエストテスト

```java
/**
 * POSTリクエストのテストパターン。
 */

// パターン1: JSONボディ付きPOST
@Test
void createResource_正常系_リソース作成成功() throws Exception {
    // Arrange
    CreateRequest request = new CreateRequest("new resource");
    TargetDto created = new TargetDto(1L, "new resource");
    when(targetService.create(any())).thenReturn(created);

    // Act & Assert
    mockMvc.perform(post("/api/resources")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("new resource"));
}

// パターン2: バリデーションエラー
@Test
void createResource_異常系_バリデーションエラー() throws Exception {
    // Arrange
    CreateRequest request = new CreateRequest("");  // 空文字は不正

    // Act & Assert
    mockMvc.perform(post("/api/resources")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors[0].field").value("name"))
            .andExpect(jsonPath("$.errors[0].message").exists());
}

// パターン3: フォームデータPOST
@Test
void submitForm_正常系_フォーム送信成功() throws Exception {
    mockMvc.perform(post("/api/forms")
                    .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                    .param("username", "testuser")
                    .param("email", "test@example.com"))
            .andExpect(status().isOk());
}
```

#### Step 3.3: PUT リクエストテスト

```java
/**
 * PUTリクエストのテストパターン。
 */

// パターン1: リソース更新
@Test
void updateResource_正常系_更新成功() throws Exception {
    // Arrange
    UpdateRequest request = new UpdateRequest("updated name");
    TargetDto updated = new TargetDto(1L, "updated name");
    when(targetService.update(eq(1L), any())).thenReturn(updated);

    // Act & Assert
    mockMvc.perform(put("/api/resources/{id}", 1L)
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("updated name"));
}

// パターン2: 存在しないリソースの更新（404）
@Test
void updateResource_異常系_リソースが存在しない() throws Exception {
    // Arrange
    UpdateRequest request = new UpdateRequest("updated name");
    when(targetService.update(eq(999L), any()))
            .thenThrow(new ResourceNotFoundException("Resource not found"));

    // Act & Assert
    mockMvc.perform(put("/api/resources/{id}", 999L)
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isNotFound());
}
```

#### Step 3.4: DELETE リクエストテスト

```java
/**
 * DELETEリクエストのテストパターン。
 */

// パターン1: リソース削除
@Test
void deleteResource_正常系_削除成功() throws Exception {
    // Arrange
    doNothing().when(targetService).delete(1L);

    // Act & Assert
    mockMvc.perform(delete("/api/resources/{id}", 1L))
            .andExpect(status().isNoContent());

    verify(targetService).delete(1L);
}

// パターン2: 削除時の存在確認
@Test
void deleteResource_異常系_リソースが存在しない() throws Exception {
    // Arrange
    doThrow(new ResourceNotFoundException("Resource not found"))
            .when(targetService).delete(999L);

    // Act & Assert
    mockMvc.perform(delete("/api/resources/{id}", 999L))
            .andExpect(status().isNotFound());
}
```

### Phase 4: 認証・認可テスト

#### Step 4.1: Spring Security 連携テスト

```java
/**
 * 認証テストの基本設定。
 */
@WebMvcTest(SecuredController.class)
@Import(SecurityConfig.class)  // セキュリティ設定をインポート
class SecuredControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SecuredService securedService;

    // パターン1: 認証ユーザーでのアクセス
    @Test
    @WithMockUser(username = "testuser", roles = {"USER"})
    void getSecuredResource_正常系_認証済みユーザー() throws Exception {
        mockMvc.perform(get("/api/secured"))
                .andExpect(status().isOk());
    }

    // パターン2: 未認証でのアクセス（401）
    @Test
    void getSecuredResource_異常系_未認証() throws Exception {
        mockMvc.perform(get("/api/secured"))
                .andExpect(status().isUnauthorized());
    }

    // パターン3: 権限不足でのアクセス（403）
    @Test
    @WithMockUser(username = "testuser", roles = {"USER"})
    void getAdminResource_異常系_権限不足() throws Exception {
        // ADMIN ロールが必要なエンドポイント
        mockMvc.perform(get("/api/admin"))
                .andExpect(status().isForbidden());
    }

    // パターン4: ADMINロールでのアクセス
    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    void getAdminResource_正常系_管理者() throws Exception {
        mockMvc.perform(get("/api/admin"))
                .andExpect(status().isOk());
    }

    // パターン5: カスタムユーザー情報の設定
    @Test
    void getResource_正常系_カスタム認証情報() throws Exception {
        mockMvc.perform(get("/api/secured")
                        .with(user("customuser")
                                .password("password")
                                .roles("USER", "MANAGER")))
                .andExpect(status().isOk());
    }

    // パターン6: CSRF トークン付きPOST
    @Test
    @WithMockUser
    void createResource_正常系_CSRF付き() throws Exception {
        mockMvc.perform(post("/api/secured")
                        .with(csrf())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"name\":\"test\"}"))
                .andExpect(status().isCreated());
    }
}
```

#### Step 4.2: JWT認証テスト

```java
/**
 * JWT認証のテストパターン。
 */
@WebMvcTest(JwtSecuredController.class)
@Import({SecurityConfig.class, JwtTokenProvider.class})
class JwtSecuredControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private JwtTokenProvider jwtTokenProvider;

    // パターン1: 有効なJWTトークン
    @Test
    void getResource_正常系_有効なJWT() throws Exception {
        String token = jwtTokenProvider.createToken("testuser", List.of("ROLE_USER"));

        mockMvc.perform(get("/api/jwt-secured")
                        .header("Authorization", "Bearer " + token))
                .andExpect(status().isOk());
    }

    // パターン2: 無効なJWTトークン
    @Test
    void getResource_異常系_無効なJWT() throws Exception {
        mockMvc.perform(get("/api/jwt-secured")
                        .header("Authorization", "Bearer invalid.token.here"))
                .andExpect(status().isUnauthorized());
    }

    // パターン3: JWTトークンなし
    @Test
    void getResource_異常系_JWTなし() throws Exception {
        mockMvc.perform(get("/api/jwt-secured"))
                .andExpect(status().isUnauthorized());
    }
}
```

### Phase 5: JSONレスポンス検証

#### Step 5.1: JSONPath による詳細検証

```java
/**
 * JSONPathを使った詳細検証パターン。
 */

// パターン1: 単純なフィールド検証
@Test
void response_JSONフィールド検証() throws Exception {
    mockMvc.perform(get("/api/resources/1"))
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("test"))
            .andExpect(jsonPath("$.active").value(true))
            .andExpect(jsonPath("$.count").value(42));
}

// パターン2: ネストしたオブジェクト検証
@Test
void response_ネストオブジェクト検証() throws Exception {
    mockMvc.perform(get("/api/resources/1"))
            .andExpect(jsonPath("$.author.id").value(10))
            .andExpect(jsonPath("$.author.name").value("John"))
            .andExpect(jsonPath("$.metadata.createdAt").exists());
}

// パターン3: 配列検証
@Test
void response_配列検証() throws Exception {
    mockMvc.perform(get("/api/resources"))
            .andExpect(jsonPath("$.length()").value(3))
            .andExpect(jsonPath("$[0].id").value(1))
            .andExpect(jsonPath("$[1].id").value(2))
            .andExpect(jsonPath("$[*].name").exists());
}

// パターン4: 条件付き検証
@Test
void response_条件付き検証() throws Exception {
    mockMvc.perform(get("/api/resources"))
            .andExpect(jsonPath("$[?(@.active == true)]").isNotEmpty())
            .andExpect(jsonPath("$[?(@.count > 10)]").exists());
}

// パターン5: Matcher使用
@Test
void response_Matcher検証() throws Exception {
    mockMvc.perform(get("/api/resources/1"))
            .andExpect(jsonPath("$.name", is("test")))
            .andExpect(jsonPath("$.name", containsString("est")))
            .andExpect(jsonPath("$.count", greaterThan(0)))
            .andExpect(jsonPath("$.tags", hasSize(3)))
            .andExpect(jsonPath("$.tags", hasItem("important")));
}

// パターン6: フィールドの存在/非存在確認
@Test
void response_フィールド存在確認() throws Exception {
    mockMvc.perform(get("/api/resources/1"))
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.name").isNotEmpty())
            .andExpect(jsonPath("$.deletedAt").doesNotExist())
            .andExpect(jsonPath("$.optionalField").hasJsonPath());  // null含む
}

// パターン7: 型検証
@Test
void response_型検証() throws Exception {
    mockMvc.perform(get("/api/resources/1"))
            .andExpect(jsonPath("$.id").isNumber())
            .andExpect(jsonPath("$.name").isString())
            .andExpect(jsonPath("$.active").isBoolean())
            .andExpect(jsonPath("$.tags").isArray())
            .andExpect(jsonPath("$.metadata").isMap());
}
```

### Phase 6: 特殊なテストケース

#### Step 6.1: ファイルアップロードテスト

```java
/**
 * ファイルアップロードのテストパターン。
 */

// パターン1: 単一ファイルアップロード
@Test
void uploadFile_正常系_単一ファイル() throws Exception {
    MockMultipartFile file = new MockMultipartFile(
            "file",                          // パラメータ名
            "test.txt",                      // ファイル名
            MediaType.TEXT_PLAIN_VALUE,      // Content-Type
            "Hello, World!".getBytes()       // ファイル内容
    );

    mockMvc.perform(multipart("/api/upload")
                    .file(file))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.filename").value("test.txt"));
}

// パターン2: 複数ファイルアップロード
@Test
void uploadFiles_正常系_複数ファイル() throws Exception {
    MockMultipartFile file1 = new MockMultipartFile(
            "files", "file1.txt", "text/plain", "Content 1".getBytes());
    MockMultipartFile file2 = new MockMultipartFile(
            "files", "file2.txt", "text/plain", "Content 2".getBytes());

    mockMvc.perform(multipart("/api/upload/multiple")
                    .file(file1)
                    .file(file2))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.uploadedCount").value(2));
}

// パターン3: ファイル + JSONパラメータ
@Test
void uploadWithMetadata_正常系() throws Exception {
    MockMultipartFile file = new MockMultipartFile(
            "file", "data.csv", "text/csv", "a,b,c\n1,2,3".getBytes());
    MockMultipartFile metadata = new MockMultipartFile(
            "metadata", "", "application/json",
            "{\"description\":\"Test file\"}".getBytes());

    mockMvc.perform(multipart("/api/upload/with-metadata")
                    .file(file)
                    .file(metadata))
            .andExpect(status().isOk());
}
```

#### Step 6.2: 非同期リクエストテスト

```java
/**
 * 非同期リクエストのテストパターン。
 */

// パターン1: DeferredResult
@Test
void asyncRequest_正常系_DeferredResult() throws Exception {
    MvcResult mvcResult = mockMvc.perform(get("/api/async"))
            .andExpect(request().asyncStarted())
            .andReturn();

    mockMvc.perform(asyncDispatch(mvcResult))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.result").value("completed"));
}

// パターン2: Callable
@Test
void asyncRequest_正常系_Callable() throws Exception {
    MvcResult mvcResult = mockMvc.perform(get("/api/async/callable"))
            .andExpect(request().asyncStarted())
            .andReturn();

    mockMvc.perform(asyncDispatch(mvcResult))
            .andExpect(status().isOk());
}
```

#### Step 6.3: ストリーミングレスポンステスト

```java
/**
 * ストリーミングレスポンスのテストパターン。
 */

// パターン1: SSE (Server-Sent Events)
@Test
void sseEndpoint_正常系() throws Exception {
    mockMvc.perform(get("/api/events")
                    .accept(MediaType.TEXT_EVENT_STREAM))
            .andExpect(status().isOk())
            .andExpect(content().contentTypeCompatibleWith(MediaType.TEXT_EVENT_STREAM));
}

// パターン2: ファイルダウンロード
@Test
void downloadFile_正常系() throws Exception {
    mockMvc.perform(get("/api/download/1"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_OCTET_STREAM))
            .andExpect(header().string("Content-Disposition",
                    containsString("attachment; filename=")));
}
```

### Phase 7: WireMock連携

#### Step 7.1: WireMock設定

```java
/**
 * WireMockとMockMvcの連携パターン。
 *
 * <p>外部APIをモック化してコントローラーをテストする。</p>
 */
@SpringBootTest
@AutoConfigureMockMvc
@WireMockTest(httpPort = 8089)  // WireMock起動ポート
class ExternalApiIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    // application-test.yml で external.api.url=http://localhost:8089 を設定

    // パターン1: 外部API成功レスポンスのモック
    @Test
    void callExternalApi_正常系() throws Exception {
        // WireMock スタブ設定
        stubFor(get(urlPathEqualTo("/external/data"))
                .willReturn(aResponse()
                        .withStatus(200)
                        .withHeader("Content-Type", "application/json")
                        .withBody("{\"value\": \"external data\"}")));

        // MockMvc でコントローラーをテスト
        mockMvc.perform(get("/api/aggregate"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.externalValue").value("external data"));
    }

    // パターン2: 外部APIエラーのモック
    @Test
    void callExternalApi_異常系_外部APIエラー() throws Exception {
        stubFor(get(urlPathEqualTo("/external/data"))
                .willReturn(aResponse()
                        .withStatus(500)
                        .withBody("{\"error\": \"Internal Server Error\"}")));

        mockMvc.perform(get("/api/aggregate"))
                .andExpect(status().isServiceUnavailable());
    }

    // パターン3: 外部APIタイムアウトのモック
    @Test
    void callExternalApi_異常系_タイムアウト() throws Exception {
        stubFor(get(urlPathEqualTo("/external/data"))
                .willReturn(aResponse()
                        .withStatus(200)
                        .withFixedDelay(5000)));  // 5秒遅延

        mockMvc.perform(get("/api/aggregate"))
                .andExpect(status().isGatewayTimeout());
    }

    // パターン4: リクエスト検証
    @Test
    void callExternalApi_リクエスト検証() throws Exception {
        stubFor(post(urlPathEqualTo("/external/create"))
                .willReturn(aResponse().withStatus(201)));

        mockMvc.perform(post("/api/trigger")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"name\":\"test\"}"))
                .andExpect(status().isOk());

        // WireMock に送信されたリクエストを検証
        verify(postRequestedFor(urlPathEqualTo("/external/create"))
                .withHeader("Content-Type", equalTo("application/json"))
                .withRequestBody(matchingJsonPath("$.name", equalTo("test"))));
    }
}
```

### Phase 8: MCPサーバー Tool テスト

#### Step 8.1: MCP Tool HTTP エンドポイントテスト

```java
/**
 * MCPサーバーのToolをHTTP経由でテストするパターン。
 *
 * <p>SSE/HTTP Transport を使用するMCPサーバーの場合に適用。</p>
 */
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class McpToolHttpTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    // パターン1: Tool 呼び出し（JSON-RPC形式）
    @Test
    void callTool_正常系_SearchApi() throws Exception {
        Map<String, Object> request = Map.of(
                "jsonrpc", "2.0",
                "method", "tools/call",
                "params", Map.of(
                        "name", "search_api",
                        "arguments", Map.of("query", "Handler")
                ),
                "id", 1
        );

        mockMvc.perform(post("/mcp")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.result").exists())
                .andExpect(jsonPath("$.result.content[0].text").isNotEmpty());
    }

    // パターン2: Tool 呼び出しエラー（不正なパラメータ）
    @Test
    void callTool_異常系_パラメータ不正() throws Exception {
        Map<String, Object> request = Map.of(
                "jsonrpc", "2.0",
                "method", "tools/call",
                "params", Map.of(
                        "name", "search_api",
                        "arguments", Map.of()  // 必須パラメータなし
                ),
                "id", 1
        );

        mockMvc.perform(post("/mcp")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.error").exists())
                .andExpect(jsonPath("$.error.code").value(-32602));  // Invalid params
    }

    // パターン3: Resource 取得
    @Test
    void readResource_正常系() throws Exception {
        Map<String, Object> request = Map.of(
                "jsonrpc", "2.0",
                "method", "resources/read",
                "params", Map.of(
                        "uri", "nablarch://handler/web"
                ),
                "id", 1
        );

        mockMvc.perform(post("/mcp")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.result.contents[0].text").isNotEmpty());
    }

    // パターン4: Prompt 実行
    @Test
    void executePrompt_正常系() throws Exception {
        Map<String, Object> request = Map.of(
                "jsonrpc", "2.0",
                "method", "prompts/get",
                "params", Map.of(
                        "name", "setup-handler-queue",
                        "arguments", Map.of("app_type", "web")
                ),
                "id", 1
        );

        mockMvc.perform(post("/mcp")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.result.messages").isArray())
                .andExpect(jsonPath("$.result.messages[0].content.text").isNotEmpty());
    }
}
```

## Input Format

```yaml
# テストコード生成リクエスト
controller_class: "com.example.api.UserController"  # テスト対象コントローラー
test_type: "unit"  # "unit" (WebMvcTest) | "integration" (SpringBootTest)

# エンドポイント定義
endpoints:
  - method: GET
    path: "/api/users/{id}"
    path_params:
      - { name: "id", type: "Long" }
    response_type: "UserDto"
    test_cases:
      - { name: "正常系_ユーザー取得", id: 1, expected_status: 200 }
      - { name: "異常系_存在しないユーザー", id: 999, expected_status: 404 }

  - method: POST
    path: "/api/users"
    request_body: "CreateUserRequest"
    response_type: "UserDto"
    test_cases:
      - { name: "正常系_ユーザー作成", body: { name: "test" }, expected_status: 201 }
      - { name: "異常系_バリデーションエラー", body: { name: "" }, expected_status: 400 }

# 認証設定（オプション）
security:
  enabled: true
  type: "spring-security"  # "spring-security" | "jwt"
  roles: ["USER", "ADMIN"]

# モック設定
mocks:
  - service: "UserService"
    methods:
      - { name: "findById", return_type: "UserDto" }
      - { name: "create", return_type: "UserDto" }
```

## Output Format

```java
/**
 * {ControllerClass}のMockMvcテスト。
 *
 * <p>{テスト種別}テストとして、各エンドポイントの
 * 正常系・異常系をカバーする。</p>
 */
@{TestAnnotation}
class {ControllerClass}Test {

    @Autowired
    private MockMvc mockMvc;

    // モック設定...

    // テストメソッド群...
}
```

## Examples

### Example 1: ユーザーAPIコントローラーテスト

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void getUser_正常系_ユーザー取得成功() throws Exception {
        UserDto user = new UserDto(1L, "testuser", "test@example.com");
        when(userService.findById(1L)).thenReturn(user);

        mockMvc.perform(get("/api/users/1")
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.username").value("testuser"))
                .andExpect(jsonPath("$.email").value("test@example.com"));
    }

    @Test
    void createUser_正常系_ユーザー作成成功() throws Exception {
        CreateUserRequest request = new CreateUserRequest("newuser", "new@example.com");
        UserDto created = new UserDto(2L, "newuser", "new@example.com");
        when(userService.create(any())).thenReturn(created);

        mockMvc.perform(post("/api/users")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(2));
    }
}
```

### Example 2: MCP Tool テスト

```java
@SpringBootTest
@AutoConfigureMockMvc
class NablarchMcpServerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void searchApi_正常系_検索成功() throws Exception {
        var request = Map.of(
                "jsonrpc", "2.0",
                "method", "tools/call",
                "params", Map.of(
                        "name", "search_api",
                        "arguments", Map.of("query", "HttpRequest")
                ),
                "id", 1
        );

        mockMvc.perform(post("/mcp")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.result.content").isArray())
                .andExpect(jsonPath("$.result.content[0].text").value(
                        containsString("HttpRequest")));
    }
}
```

## Guidelines

### 必須ルール

1. **テストメソッド名は日本語で記述すること**
   - `getUser_正常系_ユーザー取得成功` のように状況を明確に
   - アンダースコアで区切る: `メソッド名_ケース種別_具体的状況`

2. **Arrange-Act-Assert パターンを使用すること**
   - Arrange: テストデータとモックの準備
   - Act: MockMvc の perform() 実行
   - Assert: andExpect() による検証

3. **モック設定は各テストメソッドで完結させること**
   - `@BeforeEach` での共通設定は最小限に
   - テスト間の依存関係を排除

4. **JSON検証は JSONPath を使用すること**
   - レスポンスボディ全体の文字列比較は避ける
   - フィールド単位で検証することで変更に強くする

5. **HTTP ステータスコードを必ず検証すること**
   - `status().isOk()`, `status().isCreated()`, `status().isBadRequest()` 等
   - ステータスコードの検証は必須

6. **Content-Type を適切に設定すること**
   - リクエスト: `contentType(MediaType.APPLICATION_JSON)`
   - レスポンス検証: `content().contentType(MediaType.APPLICATION_JSON)`

### アンチパターン

1. **テストデータのハードコード**
   - 同じテストデータを複数テストで重複定義
   - 対策: テストデータファクトリを作成

2. **過度なモック設定**
   - 全ての依存関係をモック化してテストの意味が薄れる
   - 対策: 統合テストと単体テストを使い分ける

3. **非決定的なテスト**
   - 日時やランダム値に依存してテストが不安定
   - 対策: `Clock` のモック化、固定シード値の使用

4. **エラーメッセージの検証漏れ**
   - エラー時のレスポンスボディを検証しない
   - 対策: エラーケースでもJSONPath検証を行う

5. **認証テストの省略**
   - セキュリティ設定をテストから除外
   - 対策: `@WithMockUser` や `@Import(SecurityConfig.class)` を使用

6. **WireMock スタブのリセット漏れ**
   - 前のテストのスタブが残って影響を与える
   - 対策: `@AfterEach` で `WireMock.reset()` を呼ぶ

## Troubleshooting

### よくある問題と解決策

| 問題 | 原因 | 解決策 |
|------|------|--------|
| 404 Not Found | コントローラーがロードされていない | `@WebMvcTest(対象クラス)` を確認 |
| MockBean が null | DIされていない | `@MockBean` の配置を確認 |
| JSON パースエラー | Content-Type 不一致 | `contentType(APPLICATION_JSON)` を追加 |
| 認証エラー（401） | SecurityConfig が適用されている | `@WithMockUser` を追加 |
| CSRF エラー（403） | CSRF トークンがない | `.with(csrf())` を追加 |
| 文字化け | エンコーディング設定 | `characterEncoding("UTF-8")` を追加 |

### デバッグ方法

```java
// レスポンス全体を出力
mockMvc.perform(get("/api/test"))
        .andDo(print());  // リクエスト/レスポンス詳細を出力

// レスポンスボディを取得して検証
MvcResult result = mockMvc.perform(get("/api/test"))
        .andReturn();
String body = result.getResponse().getContentAsString();
System.out.println(body);
```
