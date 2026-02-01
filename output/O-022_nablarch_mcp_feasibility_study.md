# Nablarch MCPサーバー実装可能性調査レポート

> **作成日**: 2026-02-01
> **タスクID**: subtask_027 (parent: cmd_011)
> **作成者**: ashigaru8（シニアソフトウェアエンジニア ペルソナ）
> **ステータス**: 完了

---

## 1. エグゼクティブサマリ

### 結論

**Nablarch上でのMCPサーバー実装は「条件付きで可能」である。ただし、アプローチによって実現性と実用性が大きく異なる。**

| アプローチ | 実現性 | 推奨度 | 概要 |
|-----------|--------|--------|------|
| **A: SDK + Servletサイドカー** | ◎ 高 | ★★★★★ | SDK内蔵のServletトランスポートをNablarchと同一コンテナにデプロイ |
| **B: STDIOトランスポート専用** | ◎ 高 | ★★★★☆ | ローカル用途限定だが最も簡単。Nablarch自体は不要 |
| **C: Nablarchハンドラキュー統合** | △ 低〜中 | ★★☆☆☆ | SSE/非同期の壁が大きい。技術的挑戦が多い |
| **D: フルスクラッチ（SDK不使用）** | × 非推奨 | ★☆☆☆☆ | 開発コスト膨大、SDK更新への追随困難 |

**最も推奨するアプローチはA（SDK + Servletサイドカー）とB（STDIO）の組み合わせ**である。理由は以下の通り：

1. MCP Java SDKは**Spring非依存のコアモジュール**を持ち、Jakarta Servletベースのトランスポートが内蔵されている（事実）
2. NablarchはJakarta Servlet 6.0+準拠のコンテナ上で動作するため、SDKのServletトランスポートと同一コンテナで共存可能（事実）
3. NablarchにはSSE/WebSocket/リアクティブのネイティブサポートがなく、ハンドラキュー経由でのMCPプロトコル処理は技術的に困難（事実）

**「Nablarch自体でMCPサーバーを実装する」ことの意義は限定的**であり、Nablarchの知識・ツールをMCPサーバーの**コンテンツ**として提供することに価値がある。フレームワーク自体をトランスポート層に使う必要はない。

---

## ⚠️ 殿の方針決定（2026-02-01 cmd_016）

> **MCPサーバーのNablarch実装は断念。Spring Bootで確定。**

- **理由**: 費用対効果が見込めず、Nablarchをトランスポート層に使うメリットがない
- **決定**: MCPサーバーは当初計画通り **Spring Boot + MCP Java SDK** で実装する
- **対象プロジェクト**: [nablarch-mcp-server](https://github.com/kumanoGoro/nablarch-mcp-server) (O-020)
- **本レポートの位置付け**: Nablarch実装の可能性調査は完了。結論「条件付き可能だがメリット薄」が殿の判断を裏付けた

---

## 2. MCP仕様要件の整理

### 2.1 プロトコル概要

| 項目 | 仕様 |
|------|------|
| プロトコルバージョン | 2025-03-26 |
| メッセージフォーマット | JSON-RPC 2.0（UTF-8） |
| アーキテクチャ | Client-Host-Server |
| セッション | 1:1のステートフルセッション |
| 機能 | Tools / Resources / Prompts の3プリミティブ |

### 2.2 トランスポート要件

#### STDIO トランスポート
| 要件 | 詳細 |
|------|------|
| 通信方式 | stdin（受信）/ stdout（送信）/ stderr（ログ） |
| メッセージ区切り | 改行（newline-delimited） |
| エンコーディング | UTF-8 |
| 制約 | stdoutにMCPメッセージ以外を出力してはならない |
| 用途 | ローカルサーバー（Claude Desktop, Claude Code等） |

#### Streamable HTTP トランスポート
| 要件 | 詳細 |
|------|------|
| エンドポイント | 単一HTTPエンドポイント（POST + GET対応） |
| Content-Type | `application/json`（単一レスポンス）または `text/event-stream`（SSEストリーム） |
| セッション管理 | `Mcp-Session-Id` HTTPヘッダーによるセッション識別 |
| SSE | サーバーからの非同期メッセージ送信に使用 |
| セキュリティ | Originヘッダー検証必須、localhost限定推奨 |
| 用途 | リモート/チーム共有用 |

### 2.3 ライフサイクル

1. **初期化**: `initialize` リクエスト → レスポンス → `initialized` 通知（3ウェイハンドシェイク）
2. **機能ネゴシエーション**: クライアント/サーバー双方が対応機能を宣言
3. **操作**: 双方向メッセージ交換
4. **シャットダウン**: STDIO（ストリーム閉鎖 → SIGTERM → SIGKILL）/ HTTP（接続クローズ、DELETE）

### 2.4 MCPサーバー実装に必要な技術要件まとめ

| 要件 | STDIO | Streamable HTTP |
|------|-------|-----------------|
| JSON-RPC 2.0パース/生成 | ✅ 必須 | ✅ 必須 |
| stdin/stdout制御 | ✅ 必須 | — |
| HTTPエンドポイント | — | ✅ 必須 |
| SSE（Server-Sent Events） | — | ✅ 必須 |
| セッション管理 | ✅ 必須 | ✅ 必須 |
| 非同期処理 | △ 推奨 | ✅ 必須 |
| バッチメッセージ受信 | ✅ 必須 | ✅ 必須 |

---

## 3. Nablarch実装の実現可能性分析

### 3.1 MCP Java SDKのアーキテクチャ（重要な発見）

調査の結果、MCP Java SDKは**高度にモジュール化されており、Spring非依存で使用可能**であることが判明した。

#### モジュール構成

```
mcp-json                   ← ゼロ依存のJSON抽象API
mcp-json-jackson2          ← Jackson 2.x実装
mcp-json-jackson3          ← Jackson 3.x実装
mcp-core                   ← コアMCP実装（Springへの依存なし）
mcp                        ← 便利バンドル（mcp-core + mcp-json-jackson3）
mcp-spring-webflux         ← Spring WebFlux統合（オプション）
mcp-spring-webmvc          ← Spring WebMVC統合（オプション）
```

#### `mcp-core` のコンパイル依存（Spring非依存の証拠）

| 依存 | スコープ | 備考 |
|------|---------|------|
| `mcp-json` | compile | ゼロ依存のJSON抽象 |
| `slf4j-api` 2.0.16 | compile | ロギングファサード |
| `jackson-annotations` 2.20 | compile | アノテーション型のみ |
| `reactor-core` | compile | Project Reactor |
| `jakarta.servlet-api` 6.1.0 | **provided** | Servletトランスポート用（任意） |

**Spring関連の依存はtest scopeのみ。**

#### コア内蔵のフレームワーク非依存トランスポート

| クラス | 種別 | 依存 |
|--------|------|------|
| `StdioServerTransportProvider` | STDIO サーバー | JDK標準のみ |
| `HttpServletSseServerTransportProvider` | SSE サーバー | Jakarta Servlet（provided） |
| `HttpServletStreamableServerTransportProvider` | Streamable HTTP サーバー | Jakarta Servlet（provided） |
| `StdioClientTransport` | STDIO クライアント | JDK標準のみ |
| `HttpClientSseClientTransport` | SSE クライアント | JDK HttpClient |

**これらはSpringなしで動作する。**

#### トランスポートインターフェース（カスタム実装用）

```java
// サーバートランスポートプロバイダ（レガシーSSE用）
public interface McpServerTransportProvider {
    void setSessionFactory(McpServerSession.Factory sessionFactory);
    Mono<Void> notifyClients(String method, Object params);
    Mono<Void> closeGracefully();
}

// Streamable HTTP用プロバイダ
public interface McpStreamableServerTransportProvider {
    void setSessionFactory(McpStreamableServerSession.Factory sessionFactory);
    Mono<Void> notifyClients(String method, Object params);
    Mono<Void> closeGracefully();
}

// 個別セッショントランスポート
public interface McpServerTransport extends McpTransport {
    Mono<Void> sendMessage(JSONRPCMessage message);
    Mono<Void> closeGracefully();
    <T> T unmarshalFrom(Object data, TypeRef<T> typeRef);
}
```

### 3.2 NablarchのHTTPハンドリング能力の評価

#### JSON-RPC 2.0のリクエスト/レスポンス処理

| 項目 | 評価 | 理由 |
|------|------|------|
| JSONパース | ◎ | Jackson連携可能（`nablarch-jackson-adaptor`） |
| HTTPリクエスト受信 | ◎ | `WebFrontController`（Servlet Filter）で受信 |
| カスタムContent-Type | ○ | `BodyConvertHandler`でカスタムコンバーター実装可能 |
| HTTPレスポンス生成 | ◎ | `JaxRsResponseHandler`で柔軟に対応 |
| リクエストルーティング | ◎ | `RoutesMapping`でURIベースのルーティング |

#### SSE（Server-Sent Events）対応

| 項目 | 評価 | 理由 |
|------|------|------|
| Nablarchネイティブ | **✗ 未対応** | ドキュメント・ソースコードに痕跡なし |
| JAX-RS SSE API | **△ 未検証** | Jersey/RESTEasyは対応するがNablarchが公式にサポートしていない |
| ハンドラキューとの互換性 | **✗ 困難** | 長時間保持のHTTP接続はハンドラキューの設計思想と合致しない |

#### 非同期処理（リアクティブ/ノンブロッキング）

| 項目 | 評価 | 理由 |
|------|------|------|
| Project Reactor | **✗ 未対応** | Nablarchは完全同期・スレッドパーリクエスト |
| CompletableFuture | **✗ 未対応** | 非同期APIなし |
| ノンブロッキングI/O | **✗ 未対応** | Servletのブロッキングモデルに依存 |

### 3.3 STDIOトランスポートの実現性

| 項目 | 評価 | 理由 |
|------|------|------|
| フレームワーク制約 | ◎ 制約なし | STDIOはフレームワーク非依存 |
| SDK対応 | ◎ | `StdioServerTransportProvider` が内蔵 |
| Nablarchの関与 | — | 不要（純粋なJavaプロセスとして実行） |

**STDIOトランスポートはNablarchの制約を一切受けない。** MCP Java SDKの`StdioServerTransportProvider`をそのまま使用でき、Nablarchの知識ベースやツールロジックはJavaコードとして実装するだけである。

### 3.4 Servletサイドカー方式の実現性

MCP Java SDKの`HttpServletSseServerTransportProvider`は`HttpServlet`を継承しており、任意のServletコンテナに直接デプロイできる。

```
Nablarch + MCP サイドカー構成:

┌────────────────────────────────────────────────────┐
│           Servlet Container (Tomcat/Jetty)           │
│                                                      │
│  ┌─────────────────────┐  ┌──────────────────────┐  │
│  │    Nablarch          │  │   MCP Servlet         │  │
│  │  WebFrontController  │  │  (HttpServlet拡張)    │  │
│  │  (/api/*, /web/*)    │  │  (/mcp/*)             │  │
│  │                      │  │                        │  │
│  │  ハンドラキュー       │  │  SDK Transport        │  │
│  │  業務ロジック         │  │  Tools/Resources      │  │
│  └─────────────────────┘  └──────────────────────┘  │
│                                                      │
│  共通: DB接続プール, ログ, 設定                        │
└────────────────────────────────────────────────────┘
```

| 項目 | 評価 | 理由 |
|------|------|------|
| 技術的実現性 | ◎ | Jakarta Servlet準拠、同一コンテナで共存 |
| SDKの活用 | ◎ | `HttpServletSseServerTransportProvider` をそのまま使用 |
| Nablarch資産の活用 | ○ | DB接続、設定管理等をNablarchのSystemRepositoryから取得可能 |
| ハンドラキューの活用 | △ | MCP処理自体はハンドラキュー外だが、業務ロジック呼び出し時に利用可能 |

---

## 4. 不足機能一覧と対策

### 4.1 Nablarchに不足している機能

| 不足機能 | MCP要件との関係 | 新規実装範囲 | 難易度 | 代替策 |
|---------|----------------|-------------|--------|--------|
| **SSE（Server-Sent Events）** | Streamable HTTP必須 | Nablarchのハンドラキューでは困難 | **高** | SDK内蔵のServletトランスポートを直接使用 |
| **WebSocket** | MCP仕様では不使用（参考） | — | — | 不要 |
| **リアクティブ/ノンブロッキング** | SDK内部がReactorベース | Nablarch自体への追加は不可能に近い | **高** | SDK側で完結（Nablarchは呼び出されるだけ） |
| **JSON-RPC 2.0ネイティブ** | コアプロトコル | Nablarch単体では実装が必要 | **中** | SDKが完全に処理（自前実装不要） |
| **セッション管理（MCP用）** | Streamable HTTP必須 | SDKの`McpServerSession`で対応 | **低** | SDK任せ |

### 4.2 MCP Java SDKをNablarch上で動かすために必要な改修

#### アプローチA（Servletサイドカー）の場合

| 必要な作業 | 難易度 | 内容 |
|-----------|--------|------|
| Servlet登録 | 低 | `web.xml`またはServlet API 3.0+の`@WebServlet`でMCPサーブレットを登録 |
| 共通リソースの共有 | 低 | `SystemRepository`経由でDB接続等をMCPサーバー側から参照 |
| ツール実装 | 中 | Nablarchの知識ベースを参照するMCPツール群をJavaで実装 |
| 設定統合 | 低 | Nablarchのプロパティファイル/環境変数をMCPサーバー設定に流用 |

#### アプローチC（ハンドラキュー統合）の場合（推奨せず）

| 必要な作業 | 難易度 | 内容 |
|-----------|--------|------|
| カスタムTransportProvider実装 | **高** | `McpStreamableServerTransportProvider`をNablarchのハンドラキューにブリッジ |
| SSEハンドラ新規実装 | **高** | Nablarchにはない長時間HTTP接続の管理機構を新規作成 |
| Reactor連携 | **高** | Nablarchの同期モデルとReactorの非同期モデルをブリッジ |
| Spring依存部分の切り離し | **不要** | SDKコアにSpring依存はない（事実確認済み） |

---

## 5. Spring実装との比較表

| 観点 | Spring Boot実装 | Nablarchサイドカー実装 | Nablarchハンドラキュー統合 |
|------|----------------|----------------------|--------------------------|
| **開発効率** | ◎ Spring Boot Starterで数行 | ○ Servlet登録+ツール実装 | △ 大量のアダプタコード必要 |
| **SDKサポート** | ◎ 公式Spring Boot Starter | ○ コアSDKのServletトランスポート | △ カスタムTransport実装 |
| **エコシステム** | ◎ Spring AI統合、多数の事例 | ○ Jakarta Servlet標準 | × Nablarch固有、事例ゼロ |
| **性能（非同期）** | ◎ WebFlux対応 | ○ Servlet SSE（ブロッキング） | △ 同期→非同期ブリッジのオーバーヘッド |
| **性能（スループット）** | ◎ ノンブロッキング対応 | ○ スレッドプールベース | △ ハンドラキューの直列処理がボトルネック |
| **保守性** | ◎ SDK更新に即追随 | ◎ コアSDK依存のため追随容易 | △ カスタムTransportの保守コスト |
| **コミュニティ** | ◎ 世界中のSpring開発者 | ○ Java/Servlet標準 | × Nablarch固有 |
| **Nablarchの強み** | × 不使用 | △ 共通リソース共有程度 | ○ ハンドラキュー活用可能 |
| **デモ効果** | × Nablarchの存在感なし | △ 同一コンテナで共存 | ◎ 「NablarchでMCPサーバー」のストーリー |
| **初期コスト** | ◎ 最小 | ○ 低〜中 | × 高 |
| **リスク** | 低 | 低 | **高**（未知の技術課題） |

### 「Nablarch自体で実装する意義」の分析

#### デモンストレーション効果

- **肯定面**: 「NablarchでMCPサーバーが動く」というストーリーは、Nablarchの拡張性と現代性をアピールできる
- **否定面**: MCPプロトコルのトランスポート層をNablarchが処理する技術的必要性がなく、「無理に使っている」印象を与えるリスク
- **評価**: SSE/リアクティブの根本的な不足を露呈する可能性があり、逆効果になりうる

#### ハンドラキューアーキテクチャの活用可能性

- **肯定面**: MCPのリクエスト処理をハンドラキューで制御すれば、認証・ログ・トランザクション管理等の横断的関心事を統一的に適用できる
- **否定面**: MCPプロトコルはJSON-RPC 2.0の双方向メッセージングであり、Nablarchのリクエスト→レスポンスの直列パイプラインモデルと根本的に異なる。SSEの長時間接続はハンドラキューの設計思想（リクエストが来て→ハンドラ群を通って→レスポンスを返す）と相容れない
- **評価**: ハンドラキューの強みを活かせる場面は限定的

#### 既存Nablarchユーザーへのアピール

- **肯定面**: Nablarchユーザーが追加投資なしでAI開発支援を得られる
- **否定面**: サイドカー方式でも同等の効果が得られ、ハンドラキュー統合の追加メリットは薄い
- **評価**: サイドカー方式で十分にアピール可能

---

## 6. 推奨アプローチ

### 第1推奨: ハイブリッド方式（STDIO + Servletサイドカー）

```
┌─────────────────────────────────────────────────────────┐
│                  Nablarch MCPサーバー                      │
│                                                           │
│  【ローカル用】                【リモート/チーム用】          │
│  STDIO トランスポート           Servlet SSE トランスポート    │
│  (StdioServerTransportProvider) (HttpServletSse*Provider)  │
│         │                              │                   │
│         └──────────┬───────────────────┘                   │
│                    │                                       │
│          ┌─────────▼──────────┐                           │
│          │  Nablarch MCP Core │ ← Nablarchの知識・ツール    │
│          │  (Tools/Resources/ │                             │
│          │   Prompts)         │                             │
│          └────────────────────┘                           │
│                    │                                       │
│          ┌─────────▼──────────┐                           │
│          │  Knowledge Base    │ ← 既存のナレッジベース       │
│          │  (output/*.md等)   │                             │
│          └────────────────────┘                           │
└─────────────────────────────────────────────────────────┘
```

#### この方式の利点

1. **MCP Java SDKを最大限活用**: コアモジュールの内蔵トランスポートをそのまま使用
2. **Spring不要**: `mcp-core` + `mcp-json-jackson2` のみで動作
3. **Nablarchとの共存**: 同一Servletコンテナ上でURLパスを分けて共存
4. **デュアルトランスポート**: STDIO（ローカル）とHTTP（リモート）の両対応
5. **SDKの自動更新追随**: カスタムTransport実装が不要なため、SDK更新に自動追随
6. **Nablarch固有の知識**: Tools/Resources/Promptsの実装でNablarchのドメイン知識を提供

#### 必要な開発作業

| 項目 | 内容 | 推定規模 |
|------|------|---------|
| プロジェクト構造 | Mavenプロジェクト作成、依存関係設定 | 小 |
| MCPサーバー基盤 | `McpServer.sync()`によるサーバー構築 | 小 |
| Servlet統合 | `web.xml`でMCPサーブレット登録 | 小 |
| Tools実装 | ハンドラキュー検証、API検索、コード生成等 | 中〜大 |
| Resources実装 | ドキュメント、パターン、設定テンプレート | 中 |
| Prompts実装 | アプリケーション作成ガイド、トラブルシュート | 中 |
| ナレッジベース | 公式ドキュメントのインデックス化 | 中 |
| テスト | MCP Inspector + JUnit | 中 |

#### 最小依存関係

```xml
<!-- MCP Java SDK（Springなし） -->
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-core</artifactId>
</dependency>
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-json-jackson2</artifactId>
</dependency>

<!-- Nablarch（知識ベースアクセス用、オプション） -->
<dependency>
    <groupId>com.nablarch.framework</groupId>
    <artifactId>nablarch-core</artifactId>
</dependency>

<!-- ロギング -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
```

推移的依存: reactor-core, slf4j-api, jackson-annotations, jackson-databind (2.x)

### 第2推奨: Spring Boot実装（既存計画の維持）

既存のMCPサーバー計画（`output/nablarch_mcp_server_plan.md`）のSpring Boot方式は、技術的に最もリスクが低く、エコシステムのサポートが最も厚い。Nablarch固有の実装にこだわらない場合は、この方式が最善である。

### 非推奨: Nablarchハンドラキュー統合

技術的には不可能ではないが、SSE/リアクティブの根本的な不足により、大量のアダプタコードと技術的リスクが発生する。得られるメリット（デモ効果）に対してコストが見合わない。

---

## 7. 参考URL一覧

### MCP仕様・SDK

| リソース | URL |
|---------|-----|
| MCP公式仕様（2025-03-26） | https://modelcontextprotocol.io/specification/2025-03-26 |
| MCP トランスポート仕様 | https://modelcontextprotocol.io/specification/2025-03-26/basic/transports |
| MCP Java SDK（GitHub） | https://github.com/modelcontextprotocol/java-sdk |
| MCP Java SDK Overview | https://modelcontextprotocol.io/sdk/java/mcp-overview |
| MCP Java Server Guide | https://modelcontextprotocol.io/sdk/java/mcp-server |
| Spring AI MCP Reference | https://docs.spring.io/spring-ai-mcp/reference/mcp.html |
| MCP Java SDK (Spring不要) | https://10xdev.blog/build-a-java-mcp-server-in-10-minutes-no-spring-needed/ |
| MCP Raw STDIO解説 | https://foojay.io/today/understanding-mcp-through-raw-stdio-communication/ |
| Quarkus MCP (Streamable HTTP) | https://quarkus.io/blog/streamable-http-mcp/ |

### Nablarch

| リソース | URL |
|---------|-----|
| Nablarch公式ドキュメント | https://nablarch.github.io/docs/LATEST/doc/ |
| Nablarchアーキテクチャ | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html |
| RESTful Webサービス | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html |
| 標準ハンドラ一覧 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html |
| システムリポジトリ | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html |
| JAX-RSアダプタ | https://nablarch.github.io/docs/LATEST/doc/application_framework/adaptors/jaxrs_adaptor.html |
| Nablarch GitHub | https://github.com/nablarch |
| 稼動環境 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/platform.html |

---

## 付録A: MCP Java SDK モジュール依存関係図

```
mcp-json（ゼロ依存）
    │
    ├── mcp-json-jackson2（Jackson 2.x databind）
    │
    └── mcp-json-jackson3（Jackson 3.x databind）
            │
            v
mcp-core（reactor-core, slf4j-api, jackson-annotations,
          jakarta.servlet-api[provided]）
    │
    ├── mcp（= mcp-core + mcp-json-jackson3）
    │
    ├── mcp-spring-webflux（Spring WebFlux 6.2）← オプション
    │
    └── mcp-spring-webmvc（Spring WebMVC 6.2）← オプション
```

## 付録B: Nablarch不足機能の代替策マッピング

| Nablarchに不足 | MCPでの必要性 | 代替策 | 代替の実現性 |
|---------------|-------------|--------|------------|
| SSE | Streamable HTTP | SDK内蔵の`HttpServletSseServerTransportProvider`をサイドカーとして使用 | ◎ |
| リアクティブ | SDK内部（Reactor） | SDK側で完結。Nablarchは同期APIとして呼び出される | ◎ |
| WebSocket | MCP仕様では不使用 | 不要 | — |
| JSON-RPC 2.0 | コアプロトコル | SDKが完全に処理 | ◎ |
| MCP セッション管理 | 必須 | SDKの`McpServerSession`が管理 | ◎ |

---

*本レポートは、MCP公式仕様書、MCP Java SDKソースコード（GitHub）、Nablarch公式ドキュメント、および既存ナレッジベース（output/nablarch_kb_*.md）を一次情報源として作成した。推測と事実は本文中で明確に区別している。Xenlon関連の情報は含まない。*
