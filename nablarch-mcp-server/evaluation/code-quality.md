# Nablarch MCP Server コード品質評価レポート

**評価日**: 2026-02-10
**評価者**: 担当者B

**対象**: ~/nablarch-mcp-server（Phase 3完了時点）

---

## 1. 評価サマリ

| 評価項目 | スコア (5段階) | 概要 |
|----------|:---:|------|
| コード構造・設計パターン | ★★★★☆ (4) | 明確なパッケージ分離、適切な設計パターン使用 |
| 可読性・保守性 | ★★★★☆ (4) | 日本語Javadoc充実、命名一貫、recordの活用 |
| テスト品質 | ★★☆☆☆ (2) | テスト設計は良好だが大量のNoClassDefFoundErrorで実行不能 |
| 依存関係の健全性 | ★★★★☆ (4) | 適切な依存構成、Spring AI BOM管理 |
| セキュリティ | ★★★☆☆ (3) | SQLインジェクション対策済み、DB認証情報にハードコード残存 |

**総合スコア: 3.4 / 5.0**

---

## 2. コード構造分析

### 2.1 プロジェクト規模

| 指標 | 値 |
|------|------|
| メインソースファイル数 | 98 |
| テストソースファイル数 | 72 |
| メインソース総行数 | 約14,249行 |
| パッケージ数 | 16 |

### 2.2 パッケージ構成

```
com.tis.nablarch.mcp
├── codegen/           # コード生成機能（CodeGenerator, NamingConventionHelper）
├── common/            # 共通ユーティリティ
├── config/            # Spring設定（McpServerConfig）
├── db/                # DB層（entity, repository, dto）
│   ├── dto/
│   ├── entity/
│   └── repository/
├── embedding/         # Embedding機能（Jina, Voyage, ONNX）
│   ├── config/
│   └── local/
├── http/              # HTTPトランスポート
│   └── config/
├── knowledge/         # 知識ベース（YAMLロード・検索）
│   └── model/
├── prompts/           # MCP Promptクラス（6種）
├── rag/               # RAGパイプライン
│   ├── chunking/
│   ├── ingestion/
│   ├── parser/
│   ├── query/
│   ├── rerank/
│   └── search/
├── resources/         # MCPリソースプロバイダ（10種）
└── tools/             # MCPツール（10種）
    └── testgen/
```

**評価**: パッケージ構成は論理的で、責務が明確に分離されている。`rag/`パッケージ配下のparser→chunking→search→rerankというパイプライン構造が直感的。`db/`のentity/repository/dto分離もJPAベストプラクティスに準拠。

### 2.3 設計パターンの使用

| パターン | 使用箇所 | 評価 |
|----------|----------|------|
| Strategy | `EmbeddingClient`インターフェース（Jina/Voyage/ONNX実装切替） | ◎ 適切 |
| Template Method | `AbstractOnnxEmbeddingClient`（BGE-M3/CodeSage共通処理） | ◎ 適切 |
| Conditional Bean | `@ConditionalOnProperty`（provider: local/api切替） | ◎ 適切 |
| Builder | `MethodToolCallbackProvider.builder()`でTool登録 | ◎ 適切 |
| Graceful Degradation | `HybridSearchService`（片方失敗時に他方で応答） | ◎ 良設計 |
| Configuration Properties | `EmbeddingProperties`, `McpHttpProperties` | ◎ Spring Boot標準 |

### 2.4 Spring Bootベストプラクティス準拠

- **コンストラクタインジェクション**: ほぼ全クラスで使用（◎）
- **`@Autowired(required=false)`**: SemanticSearchToolのQueryAnalyzerのみ（将来拡張に備えた任意注入、妥当）
- **`open-in-view: false`**: 設定済み（◎ パフォーマンスベストプラクティス）
- **プロファイル分離**: `application.yaml`（STDIO）/ `application-http.yaml`（HTTP）分離（◎）
- **ConfigurationProperties**: 型安全な設定値バインディング（◎）

### 2.5 問題点

1. **`BM25SearchService.parseMetadata`（:231行）**: `new ObjectMapper()`を毎回生成。ObjectMapperは生成コストが高く、Bean注入またはstatic finalフィールドとすべき
2. **`VectorSearchService`のQualifier名不整合**: コンストラクタで`@Qualifier("jinaEmbeddingClient")`を指定しているが、実装クラス側は`@Qualifier("document")`で登録。テスト環境以外で問題になる可能性あり
3. **`DesignHandlerQueueTool`の静的FQCNマップ**: ハンドラFQCNをハードコードしており、知識ベース（handler-catalog.yaml）との二重管理になっている

---

## 3. 可読性・保守性分析

### 3.1 命名規則

| 対象 | 一貫性 | 備考 |
|------|:---:|------|
| クラス名 | ◎ | PascalCase、役割を明確に表現（例: `HybridSearchService`, `DesignHandlerQueueTool`） |
| メソッド名 | ◎ | camelCase、動詞で開始（`search`, `embed`, `validate`） |
| 変数名 | ○ | 明瞭。略語は`sb`（StringBuilder）、`rs`（ResultSet）等の慣用的なもののみ |
| 定数名 | ◎ | UPPER_SNAKE_CASE（`DEFAULT_RRF_K`, `CANDIDATE_K`, `FTS_CONFIG`） |
| パッケージ名 | ◎ | 小文字、意味的に明確 |

### 3.2 Javadoc・コメント

- **クラスレベルJavadoc**: ほぼ全クラスに記述あり（◎）
- **メソッドレベルJavadoc**: publicメソッドに@param/@return/@see記述あり（◎）
- **日本語Javadoc**: プロジェクト方針（プロジェクト方針）に準拠（◎）
- **TODOコメント**: `SemanticSearchTool:145`に1件（QueryAnalyzer統合、Phase進行上妥当）
- **セクションコメント**: `NablarchKnowledgeBase`で`// ========== 内部: xxx ==========`形式でセクション分け（○ 可読性向上）

### 3.3 メソッドの長さ・複雑度

| 長いメソッド | 行数 | 評価 |
|-------------|:---:|------|
| `NablarchKnowledgeBase.search` | 48行 | △ 5カテゴリの検索を1メソッドで処理。各カテゴリをprivateメソッドに抽出するとより良い |
| `NablarchKnowledgeBase.validateHandlerQueue` | 66行 | △ 検証ロジック+結果組み立てが混在。分離推奨 |
| `SetupHandlerQueuePrompt.execute` | 80行 | △ YAML展開処理が長い。セクション別のメソッド抽出推奨 |
| `DesignHandlerQueueTool.applyOrderingConstraints` | 70行 | △ ソートアルゴリズムが複雑。テスタビリティのためメソッド分割推奨 |

### 3.4 コードスメル

| 問題 | 箇所 | 深刻度 |
|------|------|:---:|
| ObjectMapper毎回生成 | `BM25SearchService:236` | 中 |
| `@SuppressWarnings("unchecked")` | `SetupHandlerQueuePrompt` 2箇所 | 低（YAML Map操作上やむを得ない） |
| unchecked operations警告 | `BM25SearchServiceTest` | 低 |
| 静的マップのハードコード | `DesignHandlerQueueTool.HANDLER_FQCN_MAP` | 低（知識YAMLとの二重管理） |

### 3.5 Java機能の活用

- **record**: `SearchResult`, `SearchFilters`, `ExtendedSearchFilters`, `HandlerInfo`, `AnalyzedQuery`等で活用（◎ イミュータブルなデータクラスとして適切）
- **switch式**: `SemanticSearchTool.parseMode`, `DesignHandlerQueueTool.getHandlerDescription`等（◎ Java 17+機能の適切な活用）
- **var**: `SetupHandlerQueuePrompt.execute`内で使用（○ 型が明確な場面での使用）
- **Stream API**: 検索・フィルタ・ソートで広範に使用（◎）
- **CompletableFuture**: `HybridSearchService`の並列検索（◎）

---

## 4. テスト品質分析

### 4.1 テスト実行結果

```
Tests run: 286, Failures: 7, Errors: 152, Skipped: 6
BUILD FAILURE
```

| 指標 | 値 |
|------|------|
| テスト総数 | 286 |
| 成功 | 121 |
| 失敗 | 7 |
| エラー | 152 |
| スキップ | 6 |
| 成功率 | **42.3%** |

### 4.2 エラー分析

**152件のエラーの大半はNoClassDefFoundError**。コンパイルは成功する（`mvn compile`/`mvn test-compile`ともにBUILD SUCCESS）が、テスト実行時にクラスロードに失敗する。

影響を受けるクラス:

| エラー対象 | テストクラス数 | 根本原因 |
|-----------|:---:|------|
| `NablarchKnowledgeBase` | 7 | 知識ベースのロード失敗カスケード |
| `SemanticSearchTool` | 3 | 検索ツールの依存注入失敗 |
| `SearchMode` | 6 | HybridSearchServiceTest内のenum参照失敗 |
| `MetadataFilteringService` | 1 | クラスロード失敗 |
| Resource Provider群 | 11 | リソースプロバイダのBean生成失敗 |
| `CodeGenerator` | 1 | インターフェースロード失敗 |
| `MigrationAnalysisTool` | 1 | ツールクラスロード失敗 |

**推定原因**: テスト実行時のクラスパス構成またはSpring Contextの初期化問題。mvn test-compileは成功するため、ソースコードの欠落ではなくランタイムのクラスロード順序またはstaticイニシャライザの失敗がカスケードしている可能性が高い。Phase 3完了時（2026-02-04）は810件中805件成功（5件スキップ）だったため、依存ライブラリのバージョン変更やローカル環境変化が原因と推測される。

### 4.3 成功しているテスト（121件）

以下のテストは正常に実行・成功:

- **Embedding**: JinaEmbeddingClientTest (5), VoyageEmbeddingClientTest (5)
- **Parser**: MarkdownDocumentParser (10), XmlConfigParser (12), JavaSourceParser (11), HtmlDocumentParser (10)
- **Reranking**: CrossEncoderRerankerTest (11), RerankingIntegrationTest (5)
- **Chunking**: ChunkingServiceTest (19)
- **Query**: QueryAnalyzerTest (36), AnalyzedQueryTest, QueryLanguageTest
- **Codegen**: GenerationResultTest, NamingConventionHelperTest

### 4.4 スキップされているテスト（6件）

| テストクラス | スキップ数 | 理由 |
|-------------|:---:|------|
| CodeSageOnnxEmbeddingClientTest | 3 | ONNXモデルファイル未配置（@DisabledIf条件） |
| BgeM3OnnxEmbeddingClientTest | 2 | ONNXモデルファイル未配置（@DisabledIf条件） |
| NablarchMcpServerApplicationTests | 1 | Spring Context起動テスト（DB接続不可のためスキップ） |

**評価**: スキップはONNXモデルのローカル未配置とDB接続が原因であり、環境依存のため許容範囲。

### 4.5 テスト設計の質（コード分析ベース）

成功しているテストの設計は良好:

- **Mockitoの適切な使用**: `DesignHandlerQueueToolTest`でNablarchKnowledgeBaseをMock化（◎）
- **@Nestedクラスによるテスト構造化**: 入力検証/Webアプリ/RESTアプリ/出力形式等のカテゴリ分け（◎）
- **@DisplayNameの日本語記述**: テスト目的が明確（◎）
- **MockWebServerの活用**: Embedding APIテストで外部API依存を排除（◎）
- **正常系/異常系の網羅**: null入力、空文字列、不明なタイプ等のエッジケース（○）
- **境界値テスト**: topK=0, 1, 50等（○ 一部テストで確認）

---

## 5. 依存関係分析

### 5.1 主要依存関係

| ライブラリ | バージョン | 最新安定版(推定) | 評価 |
|-----------|-----------|----------------|:---:|
| Spring Boot | 3.4.2 | 3.4.x | ◎ |
| Spring AI BOM | 1.0.0 | 1.0.x | ◎ |
| PostgreSQL Driver | (Spring Boot管理) | — | ◎ |
| pgvector | 0.1.6 | 0.1.6 | ◎ |
| Flyway | (Spring Boot管理) | — | ◎ |
| ONNX Runtime | 1.20.0 | 1.20.x | ◎ |
| DJL Tokenizers | 0.31.1 | 0.31.x | ◎ |
| Jsoup | 1.18.3 | 1.18.x | ◎ |
| Jackson YAML | (Spring Boot管理) | — | ◎ |
| OkHttp MockWebServer | 4.12.0 | 4.12.x | ◎ |
| H2 Database (test) | (Spring Boot管理) | — | ◎ |

### 5.2 依存関係の健全性

- **BOM管理**: Spring AI依存はBOM経由で一括管理されており、バージョン不整合リスクが低い（◎）
- **Spring Boot Parent**: 依存バージョン管理をSpring Boot Parentに委譲しており保守性が高い（◎）
- **テスト依存の分離**: H2, MockWebServerは`<scope>test</scope>`で適切に分離（◎）
- **不要依存**: 検出されず。全依存が実際に使用されている（◎）

### 5.3 懸念事項

- **spring-boot-starter-webflux**: Embedding API呼び出し用にWebClientのみ使用。`spring-boot-starter-web`と共存しており、リアクティブスタック全体は不要。`WebClient`のみ必要なら依存を軽量化できる可能性あり（軽微）

---

## 6. セキュリティ分析

### 6.1 認証情報管理

| 項目 | 状態 | 評価 |
|------|------|:---:|
| Embedding APIキー | 環境変数（`${JINA_API_KEY:}`）デフォルト空 | ◎ |
| Rerank APIキー | 環境変数（`${JINA_API_KEY:}`）デフォルト空 | ◎ |
| ONNXモデルパス | 環境変数（`${EMBEDDING_MODEL_PATH:}`） | ◎ |
| **DB接続パスワード** | **`nablarch_dev` ハードコード** | **✗ 要改善** |
| DB接続ユーザー名 | `nablarch` ハードコード | △ 環境変数化推奨 |
| テスト用APIキー | `test-jina-key`等（test profile限定） | ○ 許容 |

### 6.2 SQLインジェクション対策

| 箇所 | 対策 | 評価 |
|------|------|:---:|
| BM25SearchService | `NamedParameterJdbcTemplate`+パラメータバインド | ◎ |
| VectorSearchService | `NamedParameterJdbcTemplate`+パラメータバインド | ◎ |
| tsqueryトークン | `sanitizeToken`で特殊文字除去 | ◎ |
| テーブル名 | 文字列連結だが、固定値（`document_chunks`/`code_chunks`）のみ | ○ |
| FTS_CONFIG | 定数 `"japanese"` の直接連結。安全（ユーザー入力ではない） | ○ |

### 6.3 入力値バリデーション

| ツール/メソッド | バリデーション | 評価 |
|----------------|--------------|:---:|
| SemanticSearchTool.semanticSearch | query null/blank チェック、topK範囲チェック | ◎ |
| HybridSearchService.search | query null/blank、topK>=1 チェック | ◎ |
| BM25SearchService.search | query null/blank、topK>=1 チェック | ◎ |
| DesignHandlerQueueTool.design | appType null/blank/不明値チェック | ◎ |
| SetupHandlerQueuePrompt.execute | app_type null/不正値チェック | ◎ |
| NablarchKnowledgeBase.search | keyword null/blank チェック | ◎ |

### 6.4 CORS設定

- **httpプロファイル時**: 許可オリジンを`application-http.yaml`で明示指定（◎）
- **デフォルト**: localhostのみ許可（開発段階として妥当）
- **Origin Validation**: Phase 4で実装予定（`originValidation.enabled = false`）
- **allowCredentials**: `true`設定。ワイルドカードオリジンと併用しないため安全（○）

### 6.5 懸念事項

1. **DB認証情報ハードコード**: `application.yaml`に`username: nablarch`/`password: nablarch_dev`が直接記述されている。Phase 4（本番デプロイ）前に環境変数化が必須
2. **Origin Validation未実装**: Phase 4で対応予定だが、HTTPトランスポート有効化時のセキュリティリスク
3. **認証・認可未実装**: MCPエンドポイントに認証なし。Phase 4（OAuth 2.0）で対応予定

---

## 7. 改善提案（優先度付き）

### 優先度: 高

| # | 項目 | 対象 | 理由 |
|---|------|------|------|
| 1 | **テスト実行エラーの解消** | テスト全般 | 152件のNoClassDefFoundError。ビルド安定性の根幹。依存バージョンの整合性確認、`mvn dependency:tree`での競合調査が必要 |
| 2 | **DB認証情報の環境変数化** | `application.yaml:19` | `password: nablarch_dev`がハードコード。`${DB_PASSWORD:}`形式に変更すべき |

### 優先度: 中

| # | 項目 | 対象 | 理由 |
|---|------|------|------|
| 3 | ObjectMapper使い回し | `BM25SearchService:236` | `new ObjectMapper()`を毎回生成は性能上問題。static finalフィールドまたはBean注入に変更 |
| 4 | Qualifier名整合 | `VectorSearchService:54-55` | `@Qualifier("jinaEmbeddingClient")`と`@Qualifier("document")`の不整合。ConditionalOnPropertyで切り替わるため問題が顕在化しにくいが、潜在バグ |
| 5 | ハンドラFQCNの二重管理解消 | `DesignHandlerQueueTool` | `HANDLER_FQCN_MAP`の静的マップとhandler-catalog.yamlが二重管理。知識ベースから動的取得に一本化推奨 |

### 優先度: 低

| # | 項目 | 対象 | 理由 |
|---|------|------|------|
| 6 | 長メソッドのリファクタリング | `NablarchKnowledgeBase.search`(48行), `validateHandlerQueue`(66行) | 可読性向上。カテゴリ別のprivateメソッド抽出 |
| 7 | WebFlux依存の軽量化 | `pom.xml` | `spring-boot-starter-webflux`はWebClient用のみ。ただし実害は小さい |
| 8 | スキップテストの条件明示 | ONNXテスト6件 | テスト実行ログにスキップ理由を出力する@DisabledIfの条件メッセージ追加 |

---

## 8. 総括

Nablarch MCP Serverのコード品質は、コード構造・設計パターン・可読性の観点では**高品質**である。Spring Bootのベストプラクティスに準拠し、パッケージ分離・コンストラクタインジェクション・ConfigurationProperties・record型の活用など、モダンなJava/Springの設計が実践されている。

最大の課題は**テスト実行の安定性**である。Phase 3完了時（2026-02-04）には810件中805件成功していたが、現時点では286件中152件がNoClassDefFoundErrorで実行不能となっている。コンパイルは成功するため、ランタイムのクラスロード問題（依存ライブラリの変更、環境変化等）が原因と推測される。Phase 4着手前にこの問題の解消が強く推奨される。

セキュリティについては、SQLインジェクション対策・入力バリデーション・APIキーの環境変数管理等は適切だが、DB認証情報のハードコードがPhase 4（本番デプロイ）前に修正必須のリスクとして残っている。

---

*評価完了: 2026-02-10 担当者B*
