# Nablarch MCP Server UX/DX評価レポート

> **評価日**: 2026-02-10
> **評価者**: 担当者D
> **対象**: nablarch-mcp-server（Phase 3完了時点）
> 

---

## 1. 評価サマリ

| 評価項目 | スコア (5段階) | 概要 |
|----------|:---:|------|
| セットアップ容易性 | **4** | Maven Wrapper同梱、3ステップで起動可能。ただしRAG依存（pgvector/Embedding）の初期化手順が不足 |
| ドキュメント充実度 | **4** | 23設計書 + 11主要ドキュメント + INDEX.md。ただしユーザーガイドが「計画段階」記述のまま |
| エラーメッセージ品質 | **3** | Tool層は日本語で親切。ただし例外体系が薄く、構造化エラーコードなし |
| AI統合の使いやすさ | **4** | Tool/Resource/Prompt命名・descriptionは高品質。2ツール未登録の問題あり |
| 設定カスタマイズ性 | **3** | プロファイル2種、環境変数対応あり。ただしdev/prod分離なし、.env.exampleなし |
| **総合** | **3.6** | Phase 3完了として高水準。実運用に向けてはドキュメント同期とエラー体系整備が課題 |

---

## 2. セットアップ容易性分析

### 2.1 良い点

| 項目 | 詳細 |
|------|------|
| Maven Wrapper同梱 | `./mvnw package` でビルド可能。Maven未インストールでもOK |
| 最小構成3ステップ | clone → `./mvnw package` → MCP設定ファイル記述で完了 |
| STDIO デフォルト | DB不要のSTDIOモードがデフォルト。最速で動作確認可能 |
| docker-compose.yml | pgvector用のDocker Compose提供。`docker compose up`で起動 |
| DB初期化スクリプト | `db/init/01_create_extensions.sql`でpgvector拡張を自動セットアップ |
| Flyway | `V1__create_vector_tables.sql`でスキーマ自動マイグレーション |
| セットアップガイド | `docs/07-setup-guide.md`に手順・トラブルシューティング表あり |
| MCP設定例 | Claude Desktop / Claude Code / MCP Inspector の設定例完備 |

### 2.2 課題

| # | 課題 | 影響度 | 詳細 |
|---|------|:---:|------|
| S-1 | RAG依存の初期化手順が不足 | 高 | PostgreSQL+pgvectorのセットアップ後、Embeddingモデルのダウンロード・配置手順がない。`EMBEDDING_MODEL_PATH`で参照するONNXモデルの取得方法が未記載 |
| S-2 | `.env.example`未提供 | 中 | `JINA_API_KEY`、`VOYAGE_API_KEY`、`EMBEDDING_MODEL_PATH`等の環境変数一覧が設定ファイル内のコメントにしか存在しない |
| S-3 | docker-composeにアプリ本体がない | 中 | `docker-compose.yml`はpgvectorのみ。アプリ本体のDockerfileが存在せず、フルスタック起動ができない（Phase 4予定） |
| S-4 | テスト用H2セットアップ未案内 | 低 | `./mvnw test`でH2インメモリDBを使用するが、テスト実行手順がREADMEにない |

### 2.3 セットアップ手順数

| モード | 手順数 | 前提条件 |
|--------|:---:|------|
| STDIO（Phase 1機能のみ） | 3 | JDK 17 |
| RAGフル（Phase 2-3機能） | 6+ | JDK 17, Docker, PostgreSQL, ONNXモデル |

---

## 3. ドキュメント品質分析

### 3.1 ドキュメント構成

| カテゴリ | ファイル数 | 内容 |
|----------|:---:|------|
| 主要ドキュメント | 11 | overview, architecture, use-cases, RAGパイプライン仕様, DBスキーマ, API仕様, セットアップ, ユーザーガイド, 検索品質, WBS, 進捗 |
| 設計書（designs/） | 23 | Phase 1-3の各WBSタスクに対応する詳細設計書 |
| ADR（decisions/） | 1 | ADR-001: RAG-Enhanced Architecture |
| 調査レポート（research/） | 2 | RAG×MCP分析、Embeddingモデル移行 |
| テスト結果（test-results/） | 2 | Claude統合テスト、MCP Inspectorテスト |
| チェックリスト（checklists/） | 不明 | WBSタスク完了チェックリスト |
| INDEX.md | 1 | 読者別ナビゲーション付き索引 |
| **合計** | **40+** | |

### 3.2 良い点

| 項目 | 詳細 |
|------|------|
| 全文日本語 | README含め全ドキュメントが日本語。プロジェクト要件に完全準拠 |
| INDEX.md | 読者タイプ別（初めての方/開発者/利用者/管理者）のガイド付き |
| README.md | 概要、アーキテクチャ図（ASCII）、技術スタック表、クイックスタート、機能一覧、知識ソース表、ドキュメントリンク一覧を網羅 |
| 設計書23件 | Phase 1-3の各タスクに詳細設計書。WBS番号でナンバリング |
| ADR存在 | アーキテクチャ決定記録が正式プロセスで管理 |
| Javadocカバレッジ | 98ソースファイル中90ファイル（92%）にJavadocあり。584箇所のドキュメントコメント |
| `@param`/`@return`タグ | 主要メソッドにパラメータ・返却値の説明あり |

### 3.3 課題

| # | 課題 | 影響度 | 詳細 |
|---|------|:---:|------|
| D-1 | ユーザーガイドが「計画段階」のまま | 高 | `docs/08-user-guide.md`冒頭に「ステータス: 計画段階」「現時点では動作しない」と明記。Phase 3完了後も未更新 |
| D-2 | README.mdのPhase進捗が古い | 高 | Phase 1のチェックボックスが未完了（Tool実装、Resource実装等）になっているが、実際はPhase 3まで完了 |
| D-3 | セットアップガイドのTool一覧が古い | 中 | `docs/07-setup-guide.md`は「Tools（2種）」としてsearch_apiとvalidate_handler_queueのみ記載。実際は10ツール存在 |
| D-4 | CHANGELOG未作成 | 中 | リリースノート・変更履歴が存在しない。62件のPRがあるが変更履歴が追跡不能 |
| D-5 | GitHub URL不整合 | 低 | README.mdは`kumagoro1202`、ユーザーガイドは`kumanoGoro`のURLを記載 |
| D-6 | INDEX.md進捗表記が古い | 低 | 「Phase 1-2 完了 / Phase 3 進行中（76%）」と記載。実際はPhase 3完了 |
| D-7 | CONTRIBUTING.md未作成 | 低 | OSSとしてのコントリビューションガイドがない |
| D-8 | docs/14-architecture-for-beginners.md | 低 | ナンバリングが12-13を飛ばして14。記事系ドキュメントの番号体系が不明 |

### 3.4 ドキュメント充実度（概算）

| 領域 | カバー率 | 備考 |
|------|:---:|------|
| プロジェクト概要 | 95% | README + overview.md |
| アーキテクチャ | 90% | architecture.md + 23設計書 |
| セットアップ | 70% | Phase 1は十分、RAG依存部分が不足 |
| ユーザーガイド | 40% | 「計画段階」記述のまま未更新 |
| API仕様 | 85% | api-specification.md（ただし最新Tool未反映の可能性） |
| テスト結果 | 80% | 2件のテストレポートあり |
| Javadoc | 92% | 98ファイル中90ファイルにコメント |
| **総合** | **約78%** | |

---

## 4. エラーメッセージ品質分析

### 4.1 良い点

| 項目 | 具体例 |
|------|------|
| 日本語エラーメッセージ | `"検索クエリを指定してください。"` — 何が問題かが明確 |
| 有効値の提示 | `"有効な値: web, rest, batch, messaging"` — 正しい入力を案内 |
| ヒント提供 | semantic_searchの0件結果時に4項目のヒント（フィルタ緩和、別キーワード、モード変更、代替ツール） |
| 代替手段の案内 | `"search_apiツールをお試しください。"` — エラー時に別の手段を提示 |
| テストでの検証 | エラーメッセージ内容をテストコードで検証済み（SemanticSearchToolTest等） |

### 4.2 課題

| # | 課題 | 影響度 | 該当箇所 | 詳細 |
|---|------|:---:|------|------|
| E-1 | 例外クラスが1つのみ | 高 | `EmbeddingException` | アプリケーション全体で独自例外が`EmbeddingException`のみ。構造化されたエラー体系なし |
| E-2 | エラーコード体系なし | 高 | 全Tool | `ERR-001`のような構造化エラーコードがない。TroubleshootToolはNablarchのエラーコードを解析するが、MCP Server自身のエラーにはコード付与なし |
| E-3 | catch-all が曖昧 | 中 | `SemanticSearchTool:126` | `"検索中にエラーが発生しました。"` — DB接続エラーなのかEmbeddingエラーなのか不明 |
| E-4 | 生例外メッセージ露出 | 中 | `CodeGenerationTool:103` | `"コード生成中にエラーが発生しました: " + e.getMessage()` — 内部実装のメッセージがそのままユーザーに露出 |
| E-5 | TroubleshootTool出力にtypo | 低 | `TroubleshootTool:291` | `"\|------\|----\|n"` — `\n`が文字リテラル`n`になっている。Markdownテーブルが崩れる |
| E-6 | logback.xml未設定 | 低 | - | application.yamlでlogging設定のみ。logback.xmlによる詳細制御（ファイル出力、ローテーション等）なし |
| E-7 | デフォルトのconsoleパターンが空 | 低 | `application.yaml:83` | `logging.pattern.console:` が空値。デフォルトパターンに依存 |

### 4.3 エラーハンドリングパターン

```
Tool入力検証 → 日本語メッセージで応答（良好）
内部例外     → catch → ログ + 汎用エラーメッセージ（改善余地あり）
知識ベースなし → 「検索結果なし」+ヒント（良好）
```

---

## 5. AI統合の使いやすさ分析

### 5.1 良い点

| 項目 | 詳細 |
|------|------|
| Tool命名 | `semantic_search`, `design_handler_queue`, `generate_code`, `troubleshoot` — 動詞+名詞で機能が明確 |
| Tool description | 英語で詳細に記述。用途（"Use this when..."）を含む。AIモデルが適切な場面で呼び出しやすい |
| ToolParam description | 各パラメータに有効値を列挙（例: `"web, rest, batch, messaging"`） |
| Resource URI | `nablarch://handler/web`, `nablarch://guide/setup` — 直感的なURI設計 |
| Prompt引数 | 必須/任意が明示。descriptionに有効値を含む |
| Markdown出力 | 全Tool/Resource/Promptがstructured Markdownを出力。AIの後処理が容易 |
| 検索メタデータ | スコア、ソース、モジュール、URL、検索時間を含む豊富なメタデータ |

### 5.2 課題

| # | 課題 | 影響度 | 詳細 |
|---|------|:---:|------|
| A-1 | 2ツール未登録 | 高 | `McpServerConfig.nablarchTools()`に`TestGenerationTool`と`MigrationAnalysisTool`が未注入。ソースは存在するがMCPクライアントから呼び出し不能 |
| A-2 | description言語不統合 | 中 | `@Tool`/`@ToolParam`のdescriptionは英語、出力結果は日本語。AIモデルによっては混乱の原因 |
| A-3 | optional未指定 | 中 | `SemanticSearchTool`の`appType`, `module`等のオプションパラメータに`required = false`アノテーションなし。AIがすべてのパラメータを必須と誤認する可能性 |
| A-4 | 設定例ファイル不在 | 低 | `claude_desktop_config.json`や`.mcp.json`のサンプルファイルがリポジトリルートにない。ドキュメント内のコード片のみ |
| A-5 | Resource登録数の不整合 | 低 | コンテキスト情報では「8 URIパターン」だが、McpServerConfigは12リソース（6 handler + 6 guide）を登録。ResourceProviderは10クラスあるが、McpServerConfigで登録されているのはHandler/Guideの2プロバイダーのみ |

### 5.3 MCP登録状況

| コンポーネント | ソース数 | McpServerConfig登録数 | ギャップ |
|------|:---:|:---:|:---:|
| Tools | 10クラス | 8ツール | **-2**（TestGenerationTool, MigrationAnalysisTool未登録） |
| Resources | 10プロバイダー | 12リソース（2プロバイダー経由） | 8プロバイダー未登録 |
| Prompts | 6クラス | 6プロンプト | **0** |

---

## 6. 設定カスタマイズ性分析

### 6.1 良い点

| 項目 | 詳細 |
|------|------|
| YAMLベース設定 | `application.yaml`でSpring Boot標準の設定管理 |
| プロファイル切替 | `--spring.profiles.active=http`でHTTPモード切替可能 |
| 環境変数オーバーライド | `EMBEDDING_MODEL_PATH`, `JINA_API_KEY`, `VOYAGE_API_KEY`をデフォルト値付きで参照 |
| Embeddingプロバイダー切替 | `nablarch.mcp.embedding.provider: local/api`で無償/有償を切替 |
| CORS設定 | HTTPプロファイルにCORSオリジン設定あり |
| ログレベル変更 | コマンドライン引数で`--logging.level.com.tis.nablarch.mcp=DEBUG`指定可能 |
| HTTPセッション設定 | タイムアウト（30m）、最大セッション数（100）が設定可能 |

### 6.2 課題

| # | 課題 | 影響度 | 詳細 |
|---|------|:---:|------|
| C-1 | dev/prod プロファイル未分離 | 中 | default（STDIO）とhttp の2プロファイルのみ。開発環境/本番環境の分離がない |
| C-2 | DB認証情報がハードコード | 中 | `application.yaml`に`username: nablarch`, `password: nablarch_dev`がデフォルト値として記載。本番環境では環境変数で上書き必須だが、その案内なし |
| C-3 | `.env.example`未提供 | 中 | 設定可能な環境変数の一覧がない。`application.yaml`を読み解く必要あり |
| C-4 | ConfigurationProperties未活用 | 低 | `@ConfigurationProperties`クラスは`EmbeddingProperties`, `McpHttpProperties`, `RerankProperties`が存在するが、Spring Boot Configuration Processorによるメタデータ生成（IDE補完対応）がない |
| C-5 | 知識ベースパス固定 | 低 | `nablarch.mcp.knowledge.base-path: classpath:knowledge/`でクラスパス固定。外部ディレクトリから知識ファイルを読み込む設定方法がない |
| C-6 | Actuator未導入 | 低 | Spring Boot Actuatorがpom.xmlに未追加。ヘルスチェック、メトリクス、環境情報のエンドポイントなし |
| C-7 | test用YAML設定ミス | 低 | `application-test.yaml`の`nablarch.mcp.rerank`が本来のパス構造と一致していない可能性（`ingestion`ブロック内に配置されている） |

### 6.3 設定項目一覧

| 設定カテゴリ | 項目数 | 環境変数対応 |
|------|:---:|:---:|
| Spring基本設定 | 6 | - |
| MCP Server設定 | 4 | - |
| データソース | 4 | 未対応（ハードコード） |
| JPA/Flyway | 5 | - |
| Embedding（API） | 12 | `JINA_API_KEY`, `VOYAGE_API_KEY` |
| Embedding（Local） | 8 | `EMBEDDING_MODEL_PATH` |
| Reranking | 5 | - |
| HTTP Transport | 8 | - |
| ロギング | 3 | - |
| **合計** | **約55** | **3変数** |

---

## 7. 改善提案（優先度付き）

### 最優先（P1: 機能影響あり）

| # | 提案 | 対象 | 根拠 |
|---|------|------|------|
| P1-1 | TestGenerationTool/MigrationAnalysisToolをMcpServerConfigに登録 | `McpServerConfig.java` | A-1: 実装済みの2ツールがMCPクライアントから利用不能 |
| P1-2 | TroubleshootToolのMarkdownテーブルtypo修正 | `TroubleshootTool.java:291` | E-5: `"\|------\|----\|n"` → `"\|------\|----\|\n"` |

### 高（P2: ユーザー体験に直結）

| # | 提案 | 対象 | 根拠 |
|---|------|------|------|
| P2-1 | README.mdのPhaseチェックボックスを最新化 | `README.md` | D-2: Phase 3完了が反映されていない |
| P2-2 | ユーザーガイドを「計画段階」から「Phase 3完了」に更新 | `docs/08-user-guide.md` | D-1: 全12プロンプト例が「想定」記述のまま |
| P2-3 | セットアップガイドのTool一覧を10ツールに更新 | `docs/07-setup-guide.md` | D-3: 2ツールしか記載がない |
| P2-4 | ONNXモデルのダウンロード・配置手順を追加 | `docs/07-setup-guide.md` | S-1: RAG利用時の必須手順が欠落 |
| P2-5 | `.env.example`を作成 | リポジトリルート | S-2, C-3: 環境変数一覧を明示 |

### 中（P3: 品質向上）

| # | 提案 | 対象 | 根拠 |
|---|------|------|------|
| P3-1 | optional ToolParamに`required = false`を追加 | 全Toolクラス | A-3: AIがオプションパラメータを必須と誤認 |
| P3-2 | CHANGELOG.md作成 | リポジトリルート | D-4: 62PRの変更履歴が追跡不能 |
| P3-3 | エラーメッセージに内部情報を露出しない | `CodeGenerationTool.java:103` | E-4: `e.getMessage()`がそのままユーザーに返る |
| P3-4 | 未登録ResourceProvider(8クラス)の統合方針検討 | `McpServerConfig.java` | A-5: Api, Pattern, Config等のプロバイダーがMCPに未登録 |
| P3-5 | GitHub URL統一（kumagoro1202） | `docs/08-user-guide.md` | D-5: kumanoGoroは別プロジェクト用アカウント |
| P3-6 | INDEX.md進捗更新 | `docs/INDEX.md` | D-6: 「Phase 3 進行中 76%」→「Phase 3 完了 100%」 |

### 低（P4: 将来改善）

| # | 提案 | 対象 | 根拠 |
|---|------|------|------|
| P4-1 | application-dev.yaml / application-prod.yaml追加 | `src/main/resources/` | C-1: 環境別プロファイル |
| P4-2 | logback-spring.xml作成（ファイル出力、ローテーション） | `src/main/resources/` | E-6: 本番運用向けログ設定 |
| P4-3 | Spring Boot Actuator導入 | `pom.xml` | C-6: ヘルスチェック、メトリクス |
| P4-4 | CONTRIBUTING.md作成 | リポジトリルート | D-7: OSS公開時に必要 |
| P4-5 | 構造化例外体系の設計（MCPエラーコード） | 新規パッケージ | E-1, E-2: アプリ全体のエラーコード体系 |
| P4-6 | MCP設定サンプルファイル配置 | リポジトリルート | A-4: `examples/`ディレクトリに各ツール設定例 |

---

## 付録A: 検出問題サマリ

| カテゴリ | P1 | P2 | P3 | P4 | 合計 |
|----------|:---:|:---:|:---:|:---:|:---:|
| セットアップ | 0 | 2 | 0 | 0 | **2** |
| ドキュメント | 0 | 3 | 3 | 1 | **7** |
| エラーメッセージ | 1 | 0 | 1 | 3 | **5** |
| AI統合 | 1 | 0 | 2 | 1 | **4** |
| 設定 | 0 | 1 | 0 | 2 | **3** |
| **合計** | **2** | **6** | **6** | **7** | **21** |

## 付録B: 評価対象ファイル一覧

- `README.md` — プロジェクトREADME
- `docs/INDEX.md` — ドキュメント索引
- `docs/07-setup-guide.md` — セットアップガイド
- `docs/08-user-guide.md` — ユーザーガイド
- `src/main/resources/application.yaml` — メイン設定
- `src/main/resources/application-http.yaml` — HTTPプロファイル
- `src/test/resources/application-test.yaml` — テスト設定
- `src/main/java/com/tis/nablarch/mcp/config/McpServerConfig.java` — MCP登録
- `src/main/java/com/tis/nablarch/mcp/tools/*.java` — 10ツール実装
- `src/main/java/com/tis/nablarch/mcp/resources/*.java` — 10リソースプロバイダー
- `src/main/java/com/tis/nablarch/mcp/prompts/*.java` — 6プロンプト
- `src/main/java/com/tis/nablarch/mcp/embedding/EmbeddingException.java` — 例外
- `src/test/java/com/tis/nablarch/mcp/tools/*.java` — 13テストファイル
- `docker-compose.yml` — Docker Compose
- `pom.xml` — Mavenビルド定義

---

*本レポートはPhase 3完了時点（2026-02-10）のソースコードおよびドキュメントに基づく評価であり、Phase 4着手後に状況は変化する可能性がある。*
