# ドキュメント乖離チェック＋修正計画

> **作成日**: 2026-02-12
> **担当**: 担当者A
> **対象リポジトリ**: ~/nablarch-mcp-server（mainブランチ）
> **基準コミット**: 77610cb (feat: Embeddingデータ手動取込スクリプト追加)

---

## 棚卸し結果

テストデータ（src/test/resources, target/）を除く全Markdownファイル一覧。

| # | ファイル | カテゴリ | 行数 |
|---|---------|---------|------|
| 1 | README.md | ルート | 267 |
| 2 | docs/INDEX.md | ドキュメントガイド | 199 |
| 3 | docs/articles/INDEX.md | 連載記事INDEX | 207 |
| 4 | docs/articles/01-what-is-mcp.md | 連載記事 | 332 |
| 5 | docs/articles/02-project-overview.md | 連載記事 | 516 |
| 6 | docs/articles/03-setup-guide.md | 連載記事 | 919 |
| 7 | docs/articles/03A-nablarch-introduction.md | 連載記事 | 803 |
| 8 | docs/articles/04A-hands-on-basic.md | 連載記事 | 343 |
| 9 | docs/articles/04B-hands-on-advanced.md | 連載記事 | 750 |
| 10 | docs/articles/05-architecture-overview.md | 連載記事 | 783 |
| 11 | docs/articles/05A-rag-introduction.md | 連載記事 | 672 |
| 12 | docs/articles/06-knowledge-structure.md | 連載記事 | 623 |
| 13 | docs/articles/07-rag-pipeline-deep-dive.md | 連載記事 | 569 |
| 14 | docs/articles/08-spring-ai-mcp-integration.md | 連載記事 | 770 |
| 15 | docs/articles/09-configuration-guide.md | 連載記事 | 621 |
| 16 | docs/articles/10-tool-design-patterns.md | 連載記事 | 1308 |
| 17 | docs/articles/11-resource-prompt-patterns.md | 連載記事 | 733 |
| 18 | docs/articles/12-extension-guide.md | 連載記事 | 1278 |
| 19 | docs/articles/13-testing-strategy.md | 連載記事 | 968 |
| 20 | docs/articles/14-troubleshooting-and-roadmap.md | 連載記事 | 1474 |
| 21 | docs/articles/nablarch-mcp-server-for-beginners.md | 補足記事 | 335 |
| 22 | docs/articles/nablarch-mcp-server-for-junior-engineers.md | 補足記事 | 599 |
| 23 | docs/guides/01-setup.md | ガイド | 326 |
| 24 | docs/guides/02-user-guide.md | ガイド | 609 |
| 25 | docs/guides/03-streamable-http.md | ガイド | 659 |
| 26 | docs/reference/01-overview.md | リファレンス | 347 |
| 27 | docs/reference/02-architecture.md | リファレンス | 1125 |
| 28 | docs/reference/03-use-cases.md | リファレンス | 1124 |
| 29 | docs/reference/04-rag-pipeline-spec.md | リファレンス | 1354 |
| 30 | docs/reference/05-database-schema.md | リファレンス | 607 |
| 31 | docs/reference/06-api-specification.md | リファレンス | 599 |
| 32 | docs/reference/07-architecture-beginners.md | リファレンス | 659 |
| 33 | docs/reference/api/resource-uri-specification.md | API仕様 | 715 |
| 34 | docs/reference/api/tool-api-specification.md | API仕様 | 975 |
| 35 | docs/project/progress.md | プロジェクト管理 | 353 |
| 36 | docs/project/search-quality-report.md | プロジェクト管理 | 737 |
| 37 | docs/project/wbs.md | プロジェクト管理 | 652 |
| 38 | docs/decisions/ADR-001_rag-enhanced-architecture.md | ADR | 194 |
| 39 | docs/research/O-023_nablarch_rag_mcp_analysis.md | 調査レポート | 1190 |
| 40 | docs/research/O-024_embedding-model-migration.md | 調査レポート | 564 |
| 41-44 | docs/test-results/*.md (4ファイル) | テスト結果 | 1559 |
| 45-50 | .claude/skills/*.md (6ファイル) | Agent Skills | 1595 |
| 51-95 | docs/checklists/WBS-*.md (45ファイル) | WBSチェックリスト | ~1500 |
| 96-118 | docs/designs/*.md (23ファイル) | 設計書 | ~10000 |

**合計**: 118ファイル（テストデータ除く）

---

## 乖離チェック結果

### 乖離カテゴリ一覧

| カテゴリ | 影響度 | 該当ファイル数 | 概要 |
|---------|--------|-------------|------|
| **A. Embeddingモデル参照の古さ** | 🔴高 | 5 | Jina/Voyageを主表記のまま（実際のデフォルトはONNX local） |
| **B. Tool名の不一致** | 🔴高 | 3 | search_nablarch_api, validate_config, optimize, recommend等の旧名 |
| **C. Resource URI仕様の乖離** | 🔴高 | 5 | パラメトリックURI記載だが実装はリスト形式 |
| **D. 設定値のズレ** | 🟡中 | 2 | model-path、JARバージョン、ログ設定の乖離 |
| **E. 存在しないクラス参照** | 🟡中 | 1 | KnowledgeBaseLoaderクラスは実在しない |
| **F. インフラ改善の未反映** | 🟡中 | 10+ | init-knowledge.sh、日本語FTS、DLスクリプト等の未記載 |
| **G. 統計値の不整合** | 🟢低 | 5 | テスト数、PR数の食い違い |
| **H. GitHubアカウント誤り** | 🔴高 | 1 | kumanoGoroと誤記（正: kumagoro1202） |
| **I. Prompt名の不正確** | 🟢低 | 2 | READMEのPrompt説明が実装と不一致 |

---

### 詳細乖離チェック

#### A. Embeddingモデル参照の古さ（🔴高優先度）

ONNX移行タスクでONNX Embedding移行済み。デフォルト`provider: local`でBGE-M3 / CodeSageを使用。しかし複数ドキュメントがJina v4 / Voyage-code-3を主表記のまま。

| # | ファイル | 乖離箇所 | 修正方針 |
|---|---------|---------|---------|
| A1 | README.md L46-47 | アーキテクチャ図: "Doc Embedder (Jina v4) \| Code Embedder (Voyage-code-3)" | ONNX bge-m3 / CodeSageをデフォルト表記に変更 |
| A2 | README.md L130-131 | 技術スタック表: "ドキュメントEmbedding \| Jina embeddings-v4" | "BAAI/bge-m3（ONNX）/ Jina v4（API fallback）"に変更 |
| A3 | README.md L178-179 | Phase 2: "デュアルEmbedding（Jina v4 + Voyage-code-3）" | "デュアルEmbedding（ローカルONNX / API切替可能）"に変更 |
| A4 | docs/reference/02-architecture.md L91-92 | 図: "Docエンベッダ (Jina v4)" "Codeエンベッダ (Voyage-c3)" | ONNX bge-m3 / CodeSageをデフォルト表記に変更 |
| A5 | docs/project/progress.md | Phase 2 Wave 2: "Embedding統合: Jina v4" "Embedding統合: Voyage-code-3" | 歴史的記録なので注釈追加のみ（「※現在はONNXローカルモデルがデフォルト」） |

#### B. Tool名の不一致（🔴高優先度）

実装の@Tool(name=...)と記事上のTool名が一致しない箇所。

| # | ファイル | 旧名（記事上） | 正名（@Tool） | 修正方針 |
|---|---------|-------------|-------------|---------|
| B1 | docs/articles/10-tool-design-patterns.md L72 | `search_nablarch_api` | `search_api` | 全箇所置換 |
| B2 | docs/articles/10-tool-design-patterns.md L77 | `validate_config` | `validate_handler_queue` | 全箇所置換 |
| B3 | docs/articles/04A-hands-on-basic.md L15,28 | `design`（省略形） | `design_handler_queue` | フルネームに変更 |
| B4 | docs/articles/04B-hands-on-advanced.md L30,207,253,407-408 | `optimize`, `recommend` | `optimize_handler_queue`, `recommend_pattern` | フルネームに変更 |

#### C. Resource URI仕様の乖離（🔴高優先度）

記事がパラメトリックURIを記載しているが、実装はリスト形式URI。

**実装のResource URI一覧**（McpServerConfig.javaより）:
- `nablarch://handler/{type}` → web, rest, batch, messaging, http-messaging, jakarta-batch（6種）
- `nablarch://guide/{topic}` → setup, testing, validation, database, handler-queue, error-handling（6種）
- `nablarch://api/modules`（リスト1本）
- `nablarch://pattern/list`（リスト1本）
- `nablarch://example/list`（リスト1本）
- `nablarch://config/list`（リスト1本）
- `nablarch://antipattern/list`（リスト1本）
- `nablarch://version/info`（1本）

| # | ファイル | 乖離内容 | 修正方針 |
|---|---------|---------|---------|
| C1 | docs/articles/02-project-overview.md L336 | `nablarch://api/{module}/{class}` → 実装は `nablarch://api/modules` | 実装に合わせて修正 |
| C2 | docs/articles/02-project-overview.md L342 | `nablarch://version` → 実装は `nablarch://version/info` | 修正 |
| C3 | docs/articles/11-resource-prompt-patterns.md L57,94-96 | `nablarch://api/{module}/{class}` 階層URI → 実装は `nablarch://api/modules` | 実装に合わせて修正 |
| C4 | docs/articles/11-resource-prompt-patterns.md L76 | Antipattern MIMEタイプ `text/markdown` → 実装は `application/json` | 修正 |
| C5 | docs/articles/11-resource-prompt-patterns.md L77 | Config MIMEタイプ `application/xml` → 実装は `text/plain` | 修正 |
| C6 | docs/articles/11-resource-prompt-patterns.md L78 | Version URI `nablarch://version` → 実装は `nablarch://version/info` | 修正 |
| C7 | docs/articles/04B-hands-on-advanced.md L642 | `nablarch://pattern/form-validation-pattern` → 実装は `nablarch://pattern/list` | 修正 |
| C8 | docs/articles/03-setup-guide.md L529-535 | Handler Resources 4種のみ記載 → 実装は6種（http-messaging, jakarta-batch欠落） | 6種全て記載 |
| C9 | docs/reference/api/resource-uri-specification.md | URI仕様書全体 → パラメトリックURI前提の記載 | 実装に合わせて全面修正 |
| C10 | docs/reference/06-api-specification.md | 同上 | 実装に合わせて修正 |

#### D. 設定値のズレ（🟡中優先度）

| # | ファイル | 乖離内容 | 修正方針 |
|---|---------|---------|---------|
| D1 | docs/articles/09-configuration-guide.md L143-144 | model-path デフォルト `/opt/models/bge-m3` → 実装は `${user.home}/models/bge-m3` | `${user.home}/models/bge-m3` に修正 |
| D2 | docs/articles/09-configuration-guide.md L154-155 | model-path デフォルト `/opt/models/codesage-small-v2` → 実装は `${user.home}/models/codesage-small-v2` | 同上修正 |
| D3 | docs/articles/09-configuration-guide.md L196-201 | ログ設定をapplication.yamlに記載 → 実際はlogback-spring.xmlで管理 | logback-spring.xmlの説明に修正 |
| D4 | docs/articles/09-configuration-guide.md L239-245 | HTTPモードのログ設定（logging.pattern.console等）→ 実際のapplication-http.yamlに存在しない | 実際の設定に合わせて修正 |
| D5 | docs/articles/09-configuration-guide.md L265-267 | JARファイル名 `nablarch-mcp-server-0.2.0-SNAPSHOT.jar` → 実際は `0.1.0-SNAPSHOT` | `0.1.0-SNAPSHOT` に修正 |

#### E. 存在しないクラス参照（🟡中優先度）

| # | ファイル | 乖離内容 | 修正方針 |
|---|---------|---------|---------|
| E1 | docs/articles/03-setup-guide.md L330-331 | ログ例に `KnowledgeBaseLoader` クラス → 実在しない（実際は `NablarchKnowledgeBase`） | クラス名を修正、ログ出力例を実際のものに更新 |

#### F. インフラ改善の未反映（🟡中優先度）

以下の新機能・変更がドキュメントに未反映:

| cmd | 内容 | 影響ドキュメント | 修正方針 |
|-----|------|---------------|---------|
| インフラ改善タスク | Docker自動起動/DLスクリプト/ENV分離/BM25修正 | README, guides/01-setup, articles/03 | 運用改善内容を反映 |
| データ取込タスク | pgvectorベクトル検索データ取込（467ページ・1485チャンク） | README, articles/07 | RAG取込実績値を更新 |
| 日本語FTSタスク | 日本語FTS対応（pg_trgm、ILIKE+similarity()方式） | articles/07, reference/04-rag-pipeline-spec | BM25検索の実装変更を反映 |
| 手動取込タスク | 手動取込スクリプト（init-knowledge.sh + IngestionRunner.java） | guides/01-setup, articles/03-setup-guide | セットアップ手順にナレッジ初期化を追加 |
| — | download-models.sh の存在 | guides/01-setup, articles/03-setup-guide, articles/09 | ONNXモデルDL手順を追加 |

#### G. 統計値の不整合（🟢低優先度）

| # | ファイル | 記載値 | 正しい値 | 修正方針 |
|---|---------|-------|---------|---------|
| G1 | docs/articles/02-project-overview.md L434 | テスト: 1,019件 | 最新値に更新 | mvn test実行結果で更新 |
| G2 | README.md L158 | テスト: 1,027テスト以上 | 同上 | 同上 |
| G3 | docs/articles/13-testing-strategy.md L1,27,69 | テスト: 1,019件 | 同上 | 同上 |
| G4 | docs/articles/02-project-overview.md L444-445 | PR: 80件、最新PR #80 | #85以上 | PR履歴を最新化 |
| G5 | docs/project/progress.md L348 | テスト: 1,027件以上 | 同上 | mvn test実行結果で更新 |
| G6 | docs/project/progress.md L239-311 | PR履歴 #78まで | #83まで | #79-#83のPR追記 |
| G7 | docs/INDEX.md L196 | "Phase 1-3 完了 / Phase 4 未着手" | Phase 4-1完了 | 修正 |

#### H. GitHubアカウント誤り（🔴高優先度）

| # | ファイル | 乖離内容 | 修正方針 |
|---|---------|---------|---------|
| H1 | docs/guides/02-user-guide.md L35 | GitHub URL `kumanoGoro/nablarch-mcp-server` → 正しくは `kumagoro1202` | URLを `kumagoro1202` に修正 |
| H2 | docs/guides/02-user-guide.md L48 | 同上 | 同上 |
| H3 | docs/guides/02-user-guide.md L64-89 | docker-compose例が「将来提供予定」だが実在する。DB名 `nablarch_kb`（正: `nablarch_mcp`）、環境変数 `TRANSPORT_MODE`（正: `SPRING_PROFILES_ACTIVE`） | 実際のdocker-compose.ymlに合わせて全面修正 |

#### I. Prompt名の不正確（🟢低優先度）

| # | ファイル | 乖離内容 | 修正方針 |
|---|---------|---------|---------|
| I1 | README.md L20 | Prompts説明: "Webアプリ作成、REST API作成、バッチ作成、ハンドラキュー設計、コードレビュー、トラブルシューティング" | 実際のPrompt名（setup-handler-queue, create-action, review-config, explain-handler, migration-guide, best-practices）に合わせて修正 |
| I2 | docs/reference/06-api-specification.md L14,29 | "バージョン 0.1.0" "Phase 1では" → MCP server version 0.2.0、Phase表記を削除 | 修正 |

---

### チェック不要と判定したファイル群

| カテゴリ | ファイル数 | 理由 |
|---------|----------|------|
| docs/checklists/WBS-*.md | 45 | タスク完了チェックリスト（歴史的記録、修正不要） |
| docs/designs/*.md | 23 | 設計時点の仕様書（歴史的記録、実装と異なっていても設計時の意図として保持） |
| docs/decisions/ADR-001*.md | 1 | ADR（決定記録、変更不要） |
| docs/research/O-*.md | 2 | 調査レポート（歴史的記録、変更不要） |
| docs/test-results/*.md | 4 | テスト実行時点の記録（変更不要） |
| .claude/skills/*.md | 6 | Agent Skills（Nablarch FW自体のガイドであり、MCPサーバー実装とは独立） |
| docs/project/search-quality-report.md | 1 | 検索品質評価結果（測定時点の記録） |

---

## 修正計画

### 修正対象ファイル一覧

| 優先度 | 修正ファイル | 修正カテゴリ | 推定修正量 |
|--------|-----------|------------|----------|
| P1 | README.md | A1-A3, I1 | 中（5箇所） |
| P1 | docs/articles/10-tool-design-patterns.md | B1-B2 | 大（Tool名全箇所置換） |
| P1 | docs/articles/04A-hands-on-basic.md | B3 | 小（2箇所） |
| P1 | docs/articles/04B-hands-on-advanced.md | B4, C7 | 中（10箇所） |
| P1 | docs/articles/02-project-overview.md | C1-C2, G4 | 中（5箇所） |
| P1 | docs/articles/11-resource-prompt-patterns.md | C3-C6 | 大（URI仕様全面） |
| P1 | docs/guides/02-user-guide.md | H1-H3 | 大（GitHub URL + docker-compose例） |
| P1 | docs/reference/02-architecture.md | A4 | 中（図の修正） |
| P2 | docs/articles/09-configuration-guide.md | D1-D5 | 中（5箇所） |
| P2 | docs/articles/03-setup-guide.md | E1, C8, F | 中（3箇所 + init-knowledge追加） |
| P2 | docs/reference/api/resource-uri-specification.md | C9 | 大（URI仕様全面） |
| P2 | docs/reference/06-api-specification.md | C10, I2 | 中（URI + バージョン修正） |
| P2 | docs/guides/01-setup.md | F | 小（init-knowledge/download-models追記） |
| P3 | docs/articles/13-testing-strategy.md | G3 | 小（数値更新） |
| P3 | docs/project/progress.md | A5, G5-G6 | 中（PR履歴追記 + 数値更新） |
| P3 | docs/INDEX.md | G7 | 小（1箇所） |
| P3 | docs/articles/INDEX.md | — | 小（最終更新日のみ） |

### 修正ファイル総数: 17ファイル

---

### 推奨修正順序と担当者割当案

RACE-001（同一ファイルの競合）を防止するため、同一ファイルを複数担当者に割り当てない。

#### Wave 1: P1修正（並列度4）

| 担当者 | 担当ファイル | 修正カテゴリ | 備考 |
|------|-----------|------------|------|
| 担当者A | README.md, docs/INDEX.md | A1-A3, I1, G7 | ルート+INDEXの整合修正 |
| 担当者B | docs/articles/10-tool-design-patterns.md, docs/articles/04A-hands-on-basic.md, docs/articles/04B-hands-on-advanced.md | B1-B4, C7 | Tool名修正（3記事） |
| 担当者C | docs/articles/02-project-overview.md, docs/articles/11-resource-prompt-patterns.md | C1-C6, G4 | Resource URI修正（2記事） |
| 担当者D | docs/guides/02-user-guide.md, docs/reference/02-architecture.md | H1-H3, A4 | GitHub URL修正 + 図修正 |

#### Wave 2: P2修正（並列度3）

| 担当者 | 担当ファイル | 修正カテゴリ | 備考 |
|------|-----------|------------|------|
| 担当者E | docs/articles/09-configuration-guide.md | D1-D5 | 設定値修正（単一ファイル） |
| 担当者F | docs/articles/03-setup-guide.md, docs/guides/01-setup.md | E1, C8, F | セットアップ手順修正 + init-knowledge追記 |
| 担当者G | docs/reference/api/resource-uri-specification.md, docs/reference/06-api-specification.md | C9, C10, I2 | API仕様書修正（2ファイル） |

#### Wave 3: P3修正（並列度1-2）

| 担当者 | 担当ファイル | 修正カテゴリ | 備考 |
|------|-----------|------------|------|
| 担当者H | docs/articles/13-testing-strategy.md, docs/project/progress.md | G3, A5, G5-G6 | 統計値更新 + PR履歴追記 |

### 追記: init-knowledge.sh内のバグ発見

ドキュメント乖離チェック中に、スクリプト自体のバグも発見:

| ファイル | 行 | バグ内容 |
|---------|-----|---------|
| scripts/init-knowledge.sh L68 | `grep -q "nablarch-mcp-postgres"` → docker-compose.ymlのcontainer_nameは `nablarch-mcp-pgvector` | コンテナ存在判定が常に失敗 |
| scripts/init-knowledge.sh L85,127-128 | `docker compose exec -T postgres` → docker-compose.ymlのサービス名は `pgvector` | execコマンドが失敗する |

これらはドキュメント修正とは別タスクだが、修正が必要。

---

## サマリ

| 項目 | 値 |
|------|-----|
| **棚卸しファイル数** | 118 |
| **修正要ファイル数** | 17 |
| **修正不要ファイル数** | 101 |
| **乖離カテゴリ数** | 9（A〜I） |
| **P1（高優先度）修正** | 8ファイル |
| **P2（中優先度）修正** | 5ファイル |
| **P3（低優先度）修正** | 4ファイル |
| **推奨並列度** | Wave1: 4名, Wave2: 3名, Wave3: 1-2名 |
| **最大乖離カテゴリ** | A（Embeddingモデル）、B（Tool名）、C（Resource URI） |
| **主要な発見** | ONNX移行後もJina/Voyageが主表記のまま、Tool名の旧名残存、Resource URIが設計書の記載と実装で乖離 |
