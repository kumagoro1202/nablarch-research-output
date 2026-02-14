# Nablarch MCP Server 情報提供精度評価レポート


**評価者**: 担当者A
**評価日**: 2026-02-10
**対象リポジトリ**: ~/nablarch-mcp-server

---

## 1. 評価サマリ

### 総合スコア: 3 / 5（改善必要）

| 評価軸 | スコア | 説明 |
|--------|--------|------|
| 知識ファイル（YAML）精度 | 3/5 | FQCN誤り5件、バージョン情報に重大な誤り複数 |
| Resources応答精度 | 3/5 | YAMLの誤りをそのまま伝搬。構造的には健全 |
| Prompts応答精度 | 3/5 | アプリタイプ対応が不完全（http-messaging/jakarta-batch欠落） |
| Tools応答精度 | 2/5 | DesignHandlerQueueToolにハードコードされた独自FQCN・順序がYAMLと不整合 |

**総評**: 知識ファイルの構造設計・カバレッジは良好だが、FQCN（完全修飾クラス名）の正確性に系統的な問題がある。特に `nablarch.fw.web.*` と `nablarch.common.web.*` の混同が繰り返されており、AI コーディングツールが誤ったimport文を生成するリスクが高い。DesignHandlerQueueToolのハードコードデータはYAMLと二重管理状態にあり、不整合が深刻。

---

## 2. 知識ファイル精度検証結果（ファイル別正誤表）

### 全10ファイル検証結果一覧

| # | ファイル | 項目数 | 正確 | 不正確 | 要確認 | 精度率 |
|---|---------|--------|------|--------|--------|--------|
| 1 | handler-catalog.yaml | ~60 handlers | 57+ | 3 | 0 | ~95% |
| 2 | api-patterns.yaml | ~20 patterns | 18 | 2 | 0 | ~90% |
| 3 | handler-constraints.yaml | ~23 constraints | 22 | 1 | 0 | ~96% |
| 4 | design-patterns.yaml | 11 patterns | 11 | 0 | 0 | 100% |
| 5 | error-catalog.yaml | 17 errors | 17 | 0 | 0 | 100% |
| 6 | module-catalog.yaml | 20+ modules | 20 | 0 | 0 | ~100% |
| 7 | config-templates.yaml | ~10 templates | 10 | 0 | 0 | 100% |
| 8 | example-catalog.yaml | 4 examples | 4 | 0 | 0 | 100% |
| 9 | antipattern-catalog.yaml | 7 antipatterns | 7 | 0 | 0 | 100% |
| 10 | version-info.yaml | ~30 items | 22 | 8 | 0 | ~73% |

### 検出された不正確情報の詳細

#### 2.1 handler-catalog.yaml

| # | 記載FQCN | 正しいFQCN | 根拠 |
|---|----------|-----------|------|
| 1 | `nablarch.fw.web.handler.multipart.MultipartHandler` | `nablarch.fw.web.upload.MultipartHandler` | nablarch-fw-web リポジトリ src/main/java/nablarch/fw/web/upload/MultipartHandler.java |
| 2 | `nablarch.fw.web.handler.StatusCodeConvertHandler`（REST用） | `nablarch.fw.handler.StatusCodeConvertHandler` | nablarch-fw-standalone リポジトリ。web.handler パッケージには存在しない |
| 3 | `nablarch.fw.web.handler.HttpAccessLogHandler` | `nablarch.common.web.handler.HttpAccessLogHandler` | nablarch-fw-web リポジトリ src/main/java/nablarch/common/web/handler/ |

#### 2.2 api-patterns.yaml

| # | 記載FQCN | 正しいFQCN | 根拠 |
|---|----------|-----------|------|
| 4 | `nablarch.fw.web.interceptor.InjectForm` | `nablarch.common.web.interceptor.InjectForm` | nablarch-fw-web リポジトリ + 公式ドキュメント |
| 5 | `nablarch.fw.web.download.StreamResponse` | `nablarch.common.web.download.StreamResponse` | nablarch-fw-web-extension リポジトリ |

#### 2.3 handler-constraints.yaml

| # | 問題 | 詳細 |
|---|------|------|
| 6 | PackageMappingのFQCN | `nablarch.integration.router.RoutesMapping` と記載。PackageMappingという名前のhandlerを指すならFQCNが不一致。RoutesMapping自体は正しいクラスだが、handler名との対応が不明確 |

#### 2.4 version-info.yaml（重大な誤り多数）

| # | 項目 | 記載値 | 正しい値 | 根拠 |
|---|------|--------|---------|------|
| 7 | release_date | "2024-09" | "2025-03"（6u3は2025-03-24リリース） | Maven Central公開日 |
| 8 | WildFly版 | "30" | "35.0.1.Final"以降 | WildFly公式（2025年版対応表） |
| 9 | Open Liberty版 | "24.0" | "25.0.0.2"以降 | OpenLiberty公式 |
| 10 | 5u系最新版 | "5u24" | "5u26" | Nablarch公式リリース |
| 11 | PostgreSQL版 | ["14","15","16"] | 12,13,17が欠落 | Nablarch動作確認マトリクス |
| 12 | サポートDB | Oracle/PostgreSQL/H2のみ | IBM Db2, SQL Serverが欠落 | 公式動作確認マトリクス |
| 13 | APサーバー | Tomcat/WildFly/Open Libertyのみ | WebSphere Liberty, JBoss EAPが欠落 | 公式動作確認マトリクス |
| 14 | H2 Database | "2.x"（動作確認済み扱い） | 公式サポート対象外（開発用のみ） | 公式動作確認マトリクスに非掲載 |

---

## 3. Resources/Prompts/Tools応答の精度検証結果

### 3.1 ResourceProvider応答精度（8クラス）

| ResourceProvider | 参照YAML | 応答形式 | 精度評価 | 備考 |
|-----------------|----------|----------|----------|------|
| HandlerResourceProvider | handler-catalog.yaml, handler-constraints.yaml | Markdown | 3/5 | YAML内のFQCN誤りをそのまま伝搬 |
| ApiResourceProvider | module-catalog.yaml, api-patterns.yaml | JSON | 3/5 | api-patternsのFQCN誤りを伝搬 |
| VersionResourceProvider | version-info.yaml, module-catalog.yaml | JSON | 2/5 | version-infoの重大な誤りを伝搬 |
| PatternResourceProvider | design-patterns.yaml | Markdown | 5/5 | 正確 |
| ConfigResourceProvider | config-templates.yaml | Text(XML) | 5/5 | 正確 |
| GuideResourceProvider | 全YAML | Markdown | 3/5 | 複数YAMLの誤りを統合伝搬するリスク |
| ExampleResourceProvider | example-catalog.yaml | JSON | 5/5 | 正確 |
| AntipatternResourceProvider | antipattern-catalog.yaml | JSON | 5/5 | 正確 |

**構造的所見**: ResourceProviderはYAMLを忠実に読み込んでMarkdown/JSONに変換する設計であり、コード自体にロジックバグはない。問題はすべて上流のYAMLデータに起因する。YAMLを修正すれば自動的に応答が正確になる良い設計。

### 3.2 Prompt応答精度（2クラス確認）

| Prompt | 精度評価 | 問題点 |
|--------|----------|--------|
| SetupHandlerQueuePrompt | 3/5 | 対応アプリタイプが web/rest/batch/messaging のみ。http-messaging と jakarta-batch が欠落。handler-catalog.yaml では6タイプ定義されているのに Prompt 側で4タイプに制限 |
| CreateActionPrompt | 4/5 | 構造は健全。ただし生成テンプレートのimport文にapi-patternsのFQCN誤りが伝搬する可能性 |

### 3.3 Tool応答精度（2クラス確認）

| Tool | 精度評価 | 問題点 |
|------|----------|--------|
| DesignHandlerQueueTool | **2/5** | **最重大問題**。後述の詳細参照 |
| SemanticSearchTool | 4/5 | RAGパイプライン。精度はインデックスされた知識データ依存。コード構造は健全 |

---

## 4. 検出された不正確情報の一覧

### 4.1 DesignHandlerQueueTool の重大問題（詳細）

このToolはハードコードされたFQCNマップとハンドラ順序を持ち、知識YAMLとは**独立した二重管理**状態にある。以下の不整合を検出：

#### ハードコードFQCNの誤り

| 行 | ハンドラ名 | ハードコード値 | 正しい値 |
|----|-----------|-------------|---------|
| ~101 | RequestPathJavaPackageMapping | `nablarch.integration.router.RequestPathJavaPackageMapping` | `nablarch.fw.handler.RequestPathJavaPackageMapping` |
| ~102 | MultipartHandler | `nablarch.fw.web.handler.MultipartHandler` | `nablarch.fw.web.upload.MultipartHandler` |
| ~105 | PackageMapping | `nablarch.integration.router.PackageMapping` | `nablarch.integration.router.RoutesMapping` |
| ~119 | RequestThreadLoopHandler | `nablarch.fw.messaging.handler.RequestThreadLoopHandler` | `nablarch.fw.handler.RequestThreadLoopHandler` |

#### DEFAULT_HANDLER_ORDERS の不整合

DesignHandlerQueueToolの "web" 用ハンドラ順序がhandler-catalog.yamlのweb定義と大きく異なる：

**DesignHandlerQueueTool（ハードコード）の web 順序**:
```
HttpCharacterEncodingHandler → GlobalErrorHandler → HttpResponseHandler →
SecureHandler → MultipartHandler → SessionStoreHandler →
NormalizationHandler → ForwardingHandler → HttpAccessLogHandler →
ThreadContextHandler → ThreadContextClearHandler →
DbConnectionManagementHandler → TransactionManagementHandler → PackageMapping
```

**handler-catalog.yaml の web 順序**:
```
HttpCharacterEncodingHandler → GlobalErrorHandler → HttpResponseHandler →
StatusCodeConvertHandler → SecureHandler → MultipartHandler →
SessionStoreHandler → NormalizationHandler → ThreadContextHandler →
ThreadContextClearHandler → ForwardingHandler →
HttpAccessLogHandler → DbConnectionManagementHandler →
TransactionManagementHandler → PackageMapping
```

主な差分:
1. StatusCodeConvertHandler がToolのweb順序に不在
2. ForwardingHandler / HttpAccessLogHandler の位置が異なる
3. ThreadContext系ハンドラの位置が異なる

### 4.2 系統的エラーパターン分析

確認した5件のFQCN誤りのうち、4件が同一パターン：

| 誤パターン | 正パターン | 該当件数 |
|-----------|-----------|---------|
| `nablarch.fw.web.*` | `nablarch.common.web.*` | 3件（InjectForm, StreamResponse, HttpAccessLogHandler） |
| `nablarch.fw.web.handler.multipart.*` | `nablarch.fw.web.upload.*` | 1件（MultipartHandler） |
| `nablarch.fw.web.handler.*` | `nablarch.fw.handler.*` | 1件（StatusCodeConvertHandler） |

**根本原因推定**: Nablarchのパッケージ構造で `fw.web`（フレームワークWeb）と `common.web`（共通Web）の区分が直感的でなく、知識ファイル作成時に `fw.web` に統一してしまった可能性が高い。

---

## 5. 改善提案（優先度付き）

### 優先度 P1（即座に対応すべき / リリースブロッカー）

#### P1-1: DesignHandlerQueueToolの二重管理解消
- **問題**: ハードコードされたFQCNマップ + DEFAULT_HANDLER_ORDERSがYAMLと不整合
- **対策**: DesignHandlerQueueToolをYAML参照に切り替え、ハードコードを排除。handler-catalog.yaml + handler-constraints.yaml を Single Source of Truth とする
- **影響範囲**: `DesignHandlerQueueTool.java` のFQCNマップ（約40行）とDEFAULT_HANDLER_ORDERS（約60行）

#### P1-2: FQCN誤りの一括修正
- **問題**: 5件の確認済みFQCN誤り
- **対策**: 以下のファイルを修正
  - `handler-catalog.yaml`: MultipartHandler, StatusCodeConvertHandler, HttpAccessLogHandler
  - `api-patterns.yaml`: InjectForm, StreamResponse
  - `handler-constraints.yaml`: PackageMapping記述の明確化
- **検証方法**: 修正後、GitHub APIでクラス存在を自動検証するテストを追加

#### P1-3: version-info.yaml の全面更新
- **問題**: リリース日、APサーバー版、DB対応等に重大な誤り8件
- **対策**: Nablarch公式動作確認マトリクスに基づき全面改訂
- **特に重要**: release_date "2024-09" → "2025-03"、5u24 → 5u26

### 優先度 P2（早期に対応すべき）

#### P2-1: SetupHandlerQueuePromptのアプリタイプ拡張
- **問題**: web/rest/batch/messagingの4タイプのみ対応。http-messaging/jakarta-batchが欠落
- **対策**: handler-catalog.yamlの6タイプすべてに対応するよう拡張

#### P2-2: FQCN自動検証テストの導入
- **問題**: 手動検証に依存しており、再発リスクが高い
- **対策**: JUnit/統合テストで全YAMLのFQCNを実クラスロードまたはGitHub API照合で検証

### 優先度 P3（中期的に対応）

#### P3-1: version-info.yaml の自動更新機構
- **問題**: バージョン情報が手動管理で陳腐化しやすい
- **対策**: CI/CDでNablarch BOMの最新バージョンを定期チェックし、差分があればPR自動作成

#### P3-2: DesignHandlerQueueToolのテスト強化
- **問題**: ハンドラ順序の正確性を検証するテストがない
- **対策**: handler-constraints.yamlの制約を満たしているかを自動検証するユニットテスト追加

---

## 付録A: 検証手法

| 手法 | 対象 | ツール |
|------|------|--------|
| GitHub API照合 | FQCN存在確認 | `gh api repos/nablarch/{repo}/contents/{path}` |
| GitHub Search | パッケージ構造確認 | `gh search code` |
| WebSearch | 公式ドキュメント・リリース情報 | WebSearch (Nablarch公式) |
| WebFetch | Maven Central、公式ドキュメント | WebFetch |
| ソースコード静的解析 | ResourceProvider/Prompt/Tool実装 | Read tool（全ファイル通読） |

## 付録B: 確認したファイル一覧

### 知識YAML（10/10ファイル確認済み）
1. handler-catalog.yaml
2. api-patterns.yaml
3. handler-constraints.yaml
4. design-patterns.yaml
5. error-catalog.yaml
6. module-catalog.yaml
7. config-templates.yaml
8. example-catalog.yaml
9. antipattern-catalog.yaml
10. version-info.yaml

### Javaソースコード（12ファイル確認済み）
- ResourceProvider: 8クラス（HandlerResourceProvider, ApiResourceProvider, VersionResourceProvider, PatternResourceProvider, ConfigResourceProvider, GuideResourceProvider, ExampleResourceProvider, AntipatternResourceProvider）
- Prompt: 2クラス（SetupHandlerQueuePrompt, CreateActionPrompt）
- Tool: 2クラス（DesignHandlerQueueTool, SemanticSearchTool）
