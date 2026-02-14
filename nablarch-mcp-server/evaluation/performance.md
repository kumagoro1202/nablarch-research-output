# Nablarch MCP Server - パフォーマンス・運用評価レポート

評価日: 2026-02-10
評価対象: nablarch-mcp-server (commit: 02943c1)

---

## 1. 評価サマリ

| 評価項目 | スコア | 状態 |
|---------|-------|------|
| **起動パフォーマンス** | ⭐⭐⭐☆☆ 3/5 | 要改善 |
| **レスポンス速度** | ⭐⭐⭐⭐☆ 4/5 | 良好（ただしテスト失敗あり） |
| **メモリ効率** | ⭐⭐⭐☆☆ 3/5 | 普通（最適化余地あり） |
| **スケーラビリティ** | ⭐⭐⭐☆☆ 3/5 | 基礎実装のみ |
| **ログ・監視** | ⭐⭐☆☆☆ 2/5 | 最低限のみ実装 |
| **運用容易性** | ⭐⭐⭐☆☆ 3/5 | Docker対応、CI/CD未実装 |
| **総合スコア** | ⭐⭐⭐☆☆ 3.0/5 | 機能実装優先、性能・運用は改善余地大 |

### 総括

- **Phase 3完了時点**: 機能実装（10 Tools + 8 Resources + 6 Prompts）に注力。性能・運用面は最低限の実装
- **テスト状況**: 実行時には大量のエラー（109 errors, 4 failures）が発生。ただし過去のレポート（2026-02-04）では805/810件成功と記載あり。何らかの環境問題またはリグレッションの可能性
- **本番展開前の課題**: Phase 4で運用面の強化が必須（モニタリング、CI/CD、性能最適化）

---

## 2. 起動・レスポンス分析

### 2.1 起動時間

**実測データ:**
- `mvn clean test` 実行時間: **25.894秒** (内訳: user 1m26.914s, sys 0m5.458s)
  - コンパイル時間含む総時間
  - 実際のアプリケーション起動時間はこれより短いと推定（10秒前後）

**起動時処理:**
- **@PostConstruct初期化**: 16クラスで初期化処理を実施
  - `NablarchKnowledgeBase`: 7つのYAMLファイル（合計156KB）をロード・インデックス構築
  - 8つの`ResourceProvider`: 各自YAMLファイルをロード・Markdown生成
  - 6つの`Prompt`: テンプレート構築
  - 1つの`AbstractOnnxEmbeddingClient`: 未使用だがBean登録

**起動時間への影響要因:**
1. **知識ファイル一括読み込み**: 起動時に全YAMLをメモリロード（遅延ロードなし）
2. **PostgreSQL接続確立**: Flyway migration実行（DB依存で起動失敗リスクあり）
3. **Beanスキャン・初期化**: Spring Boot標準の処理時間
4. **ONNX Embeddingモデル**: `provider: local`設定時、モデルファイル読み込みが発生する可能性（未確認）

**起動失敗リスク:**
- PostgreSQL未起動時: 起動失敗（`docker-compose up`前提だが、ローカル開発時に障害）
- DB接続設定なしでの起動は不可（STDIO modeでもDB必須）

**改善提案:**
- [ ] **優先度: 高** - DB接続をオプション化（STDIO modeではRAG機能OFFでも起動可能にする）
- [ ] **優先度: 中** - 知識ファイルの遅延ロード検討（使用頻度の低いファイルは初回アクセス時にロード）
- [ ] **優先度: 中** - 起動時間計測用のログ追加（各初期化処理の所要時間を記録）

### 2.2 レスポンス速度

**テスト実行時間:**
- 総テスト数: 153件（ただしエラー多数）
- 総実行時間: 22.912秒
- 1テストあたり平均: **約0.15秒**

**個別テストの所要時間（成功したもの）:**
- `JinaEmbeddingClientTest`: 5件, 4.291秒（1件あたり0.86秒）
- `CodeSageOnnxEmbeddingClientTest`: 5件（3件スキップ）, 0.056秒
- `BgeM3OnnxEmbeddingClientTest`: 4件（2件スキップ）, 0.027秒
- その他: 多くのテストが0.002〜0.020秒で完了（または即座にエラー）

**重いテスト:**
- `JinaEmbeddingClientTest`: API呼び出しリトライ処理により時間がかかる（WireMock使用、意図的な失敗テスト含む）

**レスポンス速度に影響する要素:**
1. **Embedding API呼び出し**: Jina/Voyage API利用時はネットワークレイテンシ（タイムアウト30秒設定）
2. **ONNX推論**: ローカルモデル使用時はCPU推論時間（現状未計測）
3. **知識ファイル検索**: メモリ内検索のため高速（インデックスあり）
4. **DB検索**: ベクトル検索・BM25検索の性能はDB負荷次第

**改善提案:**
- [ ] **優先度: 高** - Embedding処理の性能プロファイリング（API vs ONNX、バッチサイズ最適化）
- [ ] **優先度: 中** - DB検索のスロークエリ分析（pgvectorのインデックスチューニング）
- [ ] **優先度: 低** - テストのモック化徹底（外部API依存を削減）

---

## 3. メモリ・スケーラビリティ分析

### 3.1 メモリ使用量

**現状の設定:**
- **JVMメモリ設定**: pom.xmlに明示的な設定なし（JVMデフォルト値依存）
- **Javaバージョン**: OpenJDK 17.0.18
- **Spring Boot**: デフォルトメモリプロファイル使用

**メモリ使用要素:**
1. **知識ファイル**: 156KB（YAML 7本 + Resource用YAML複数）→ メモリ展開後は数MB程度
2. **Hibernate/JPA**: エンティティキャッシュ（設定未確認）
3. **HikariCP**: デフォルト接続プール設定（最大10接続と推定）
4. **ONNX Runtime**: モデルファイル使用時（BGE-M3: ~2GB, CodeSage-small: ~500MB）
   - **重要**: 現状のデフォルト設定（`provider: local`）では、ONNXモデルロードがメモリを大きく消費する可能性
   - モデルパス: `/opt/models/bge-m3/model.onnx` → 環境変数未設定時は起動失敗リスク
5. **ベクトルインデックス**: PostgreSQL側で保持（アプリメモリ影響小）

**推定メモリ使用量:**
- **API mode（`provider: api`）**: Heap 512MB〜1GB で動作可能
- **Local ONNX mode（`provider: local`）**: Heap 4GB以上推奨（モデルロード + 推論バッファ）

**メモリ不足リスク:**
- ONNX モデル使用時に `-Xmx` 未指定だとOOMエラーの可能性
- 複数クライアント同時接続時の推論処理でメモリ逼迫

**改善提案:**
- [ ] **優先度: 高** - pom.xmlまたはREADMEにJVMメモリ推奨値を記載（`-Xms512m -Xmx4g`）
- [ ] **優先度: 高** - ONNXモデルの遅延ロード・キャッシュ制御の実装
- [ ] **優先度: 中** - Spring Actuatorで `/actuator/metrics/jvm.memory.*` 公開
- [ ] **優先度: 低** - プロファイリング実施（VisualVM, JProfiler等で実測）

### 3.2 キャッシュ機構

**現状:**
- **キャッシュ未実装**: `@Cacheable`, `@EnableCaching`, `CacheManager` のいずれも使用なし
- 知識ファイルは起動時一度だけロードし、メモリに保持（実質的に永続キャッシュ）
- DB検索結果のキャッシュなし（毎回クエリ実行）

**キャッシュ候補:**
1. **Embedding結果**: 同一テキストの埋め込みベクトルをキャッシュ（APIコスト削減）
2. **検索結果**: 頻繁に実行されるクエリ結果を一定時間キャッシュ
3. **Markdown生成結果**: Resource提供時の動的Markdown生成をキャッシュ

**改善提案:**
- [ ] **優先度: 中** - Spring Cache + Caffeine導入（Embedding結果キャッシュ）
- [ ] **優先度: 低** - Redis導入検討（複数インスタンス間でキャッシュ共有）

### 3.3 スケーラビリティ

**現状の実装:**

**接続プール設定:**
```yaml
# application.yaml には明示的な設定なし
# Spring Boot デフォルト（HikariCP）:
#   - maximum-pool-size: 10
#   - minimum-idle: 10
```

**HTTPモード設定（application-http.yaml）:**
```yaml
server:
  port: 8080
  tomcat:
    threads:
      max: 200        # 最大スレッド数
      min-spare: 10   # 最小アイドルスレッド数

mcp:
  http:
    session:
      timeout: 30m
      max-sessions: 100  # 最大セッション数
```

**スケーラビリティ分析:**

| 項目 | 現状 | ボトルネック想定 | 改善案 |
|-----|------|----------------|--------|
| DB接続 | 最大10接続（推定） | 同時リクエスト10以上で待機発生 | HikariCP設定明示化（max: 50） |
| HTTPスレッド | 最大200 | CPU bound処理（ONNX推論）でスレッド占有 | 非同期処理 or スレッドプール分離 |
| セッション | 最大100 | 長時間接続クライアントで上限到達 | タイムアウト調整、セッション管理改善 |
| 知識ファイル | 起動時一括ロード | ファイル増加時にメモリ・起動時間増大 | ページング、遅延ロード |
| ベクトル検索 | pgvectorに依存 | 大量データ時にクエリ性能劣化 | インデックスチューニング、キャッシュ |

**水平スケーリング対応:**
- **現状**: ステートレス設計（セッション管理除く）のため、複数インスタンス起動は可能
- **課題**: セッション管理がメモリベースのため、ロードバランサ使用時はSticky Session必須

**改善提案:**
- [ ] **優先度: 高** - HikariCP設定を明示化し、環境変数で調整可能にする
- [ ] **優先度: 中** - ONNX推論を別スレッドプールで実行（HTTPスレッドを占有しない）
- [ ] **優先度: 低** - Redis Session導入（水平スケーリング対応）
- [ ] **優先度: 低** - 負荷テスト実施（JMeter, Gatling等）

---

## 4. ログ・監視分析

### 4.1 ログ設定

**現状の設定（application.yaml）:**
```yaml
logging:
  level:
    root: WARN
    com.tis.nablarch.mcp: INFO
```

**HTTPモード（application-http.yaml）:**
```yaml
logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  level:
    root: INFO
    com.tis.nablarch.mcp: DEBUG
```

**ログファイル出力:**
- **現状**: コンソール出力のみ（ファイルローテーション設定なし）
- **logback設定ファイル**: 存在せず（Spring Boot デフォルト）

**ログレベルの適切性:**
- ✅ **本番想定**: `root: WARN`, `mcp: INFO` は妥当
- ✅ **開発・デバッグ**: HTTPモードで `mcp: DEBUG` に切り替え可能
- ❌ **問題**: ファイル出力なし → ログ永続化・分析不可

**構造化ログ:**
- **現状**: 未対応（Plain textログのみ）
- **影響**: ELK Stack, Splunk等のログ集約ツールでの解析が困難

**改善提案:**
- [ ] **優先度: 高** - logback.xml追加（ファイル出力、ローテーション設定）
- [ ] **優先度: 中** - 構造化ログ対応（Logstash JSON Encoder使用）
- [ ] **優先度: 中** - リクエストID/セッションIDをMDCで自動記録
- [ ] **優先度: 低** - アクセスログ・監査ログの分離

### 4.2 メトリクス・監視

**Spring Boot Actuator:**
- **現状**: **未実装**（依存関係に `spring-boot-starter-actuator` なし）
- **影響**: JVMメトリクス、ヘルスチェック、スレッドダンプ等が取得不可

**ヘルスチェックエンドポイント:**
- **現状**: なし
- **docker-compose.yml**: PostgreSQLにはヘルスチェックあり（`pg_isready`）
- **アプリケーション側**: ヘルスチェックエンドポイント未実装
- **影響**: Kubernetes等のオーケストレーションツールでの死活監視不可

**メトリクス収集:**
- **現状**: なし（Micrometer未使用）
- **望ましいメトリクス**:
  - リクエスト数・レスポンスタイム（Tool別）
  - Embedding API呼び出し回数・レイテンシ
  - DB接続プール使用率
  - JVM Heap/GC時間
  - 例外発生率

**改善提案:**
- [ ] **優先度: 高** - Spring Boot Actuator導入（最低限 `/actuator/health`, `/actuator/info`）
- [ ] **優先度: 高** - ヘルスチェックエンドポイント実装（DB接続確認含む）
- [ ] **優先度: 中** - Micrometer導入（Prometheus形式でメトリクス公開）
- [ ] **優先度: 低** - 分散トレーシング（OpenTelemetry, Zipkin等）

### 4.3 エラーハンドリング・アラート

**現状:**
- エラーはログ出力のみ（SLF4J経由）
- アラート機構なし

**改善提案:**
- [ ] **優先度: 中** - エラーレート監視・閾値アラート（Prometheus Alertmanager等）
- [ ] **優先度: 低** - Sentry等のエラートラッキングサービス連携

---

## 5. 運用容易性分析

### 5.1 コンテナ化対応

**Docker対応:**
- ✅ `docker-compose.yml` 存在（PostgreSQL + pgvector）
- ❌ アプリケーション用 `Dockerfile` なし
- ❌ マルチステージビルド未実装

**docker-compose.yml の内容:**
```yaml
services:
  pgvector:
    image: pgvector/pgvector:pg16
    container_name: nablarch-mcp-pgvector
    environment:
      POSTGRES_DB: nablarch_mcp
      POSTGRES_USER: nablarch
      POSTGRES_PASSWORD: nablarch_dev
    ports:
      - "5432:5432"
    volumes:
      - pgvector-data:/var/lib/postgresql/data
      - ./db/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U nablarch -d nablarch_mcp"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
```

**課題:**
- アプリケーション本体のDockerイメージなし → 本番デプロイ手順が不明確
- README記載: "Docker で実行（Phase 4）" → Phase 4未着手

**改善提案:**
- [ ] **優先度: 高** - Dockerfile作成（OpenJDK 17ベース、マルチステージビルド）
- [ ] **優先度: 中** - docker-compose.yml にアプリケーションサービス追加
- [ ] **優先度: 中** - `.dockerignore` 作成

### 5.2 CI/CD パイプライン

**現状:**
- ❌ GitHub Actions ワークフローなし（`.github/workflows/` ディレクトリ不在）
- ❌ 自動テスト実行なし
- ❌ 自動デプロイなし

**望ましいCI/CDフロー:**
1. **CI**: PR作成時 → `mvn clean test` 実行 → 結果をPRにコメント
2. **CD**: mainブランチマージ時 → Dockerイメージビルド → コンテナレジストリにプッシュ
3. **品質ゲート**: テストカバレッジ計測（JaCoCo）、静的解析（SpotBugs, Checkstyle）

**改善提案:**
- [ ] **優先度: 高** - GitHub Actions ワークフロー作成（`.github/workflows/test.yml`）
- [ ] **優先度: 中** - Dockerイメージビルド・プッシュの自動化
- [ ] **優先度: 低** - CodeQL/Dependabot導入（セキュリティスキャン）

### 5.3 設定の環境別切り替え

**現状の設定ファイル:**
- `application.yaml`: デフォルト設定（STDIOモード）
- `application-http.yaml`: HTTPモード用
- `application-test.yaml`: テスト用

**環境変数対応:**
- ✅ `${EMBEDDING_MODEL_PATH:/opt/models/...}`: モデルパスを環境変数で上書き可能
- ✅ `${JINA_API_KEY:}`, `${VOYAGE_API_KEY:}`: APIキーを環境変数から読み込み
- ✅ `${db.pool.maxSize}`: 知識ファイル内でDB設定を参照（実際の動作は未検証）

**課題:**
- Spring Profilesは使用しているが、本番環境用の `application-prod.yaml` が存在しない
- DB接続情報がハードコード（`username: nablarch`, `password: nablarch_dev`）→ Secretsが平文

**改善提案:**
- [ ] **優先度: 高** - `application-prod.yaml` 作成（本番用最適設定）
- [ ] **優先度: 高** - DB接続情報を環境変数から読み込むよう変更
- [ ] **優先度: 中** - 設定ファイルのテンプレート化（`.env.example` 提供）
- [ ] **優先度: 低** - Kubernetes ConfigMap/Secret対応

### 5.4 バックアップ・リストア手順

**現状:**
- ❌ ドキュメント化されていない
- ❌ 自動バックアップ機構なし

**バックアップ対象:**
1. **PostgreSQL データベース**: ベクトルインデックス、ドキュメントメタデータ
2. **知識YAMLファイル**: Git管理下にあるため、リポジトリがバックアップ
3. **ONNXモデルファイル**: 外部ダウンロード or マウント → バックアップ手順不明

**改善提案:**
- [ ] **優先度: 中** - DBバックアップスクリプト作成（`pg_dump`）
- [ ] **優先度: 中** - docs/ にバックアップ・リストア手順を記載
- [ ] **優先度: 低** - 定期バックアップの自動化（cron or Kubernetes CronJob）

---

## 6. テスト実行結果の詳細分析

### 6.1 テスト実行結果サマリ

**実行日時**: 2026-02-10 07:49:06
**実行コマンド**: `mvn clean test`
**総実行時間**: 22.912秒（25.894秒 wall time）

```
[ERROR] Tests run: 153, Failures: 4, Errors: 109, Skipped: 6
[INFO] BUILD FAILURE
```

**内訳:**
- ✅ **成功**: 34件（153 - 4 - 109 - 6）
- ❌ **失敗（Failure）**: 4件
- ❌ **エラー（Error）**: 109件（大半が `NoClassDefFoundError`）
- ⏭️ **スキップ**: 6件

### 6.2 エラー詳細

**主要エラー: `NoClassDefFoundError`**

109件のエラーの大半が以下のパターン:
```
java.lang.NoClassDefFoundError: com/tis/nablarch/mcp/rag/search/BM25SearchService
java.lang.NoClassDefFoundError: com/tis/nablarch/mcp/resources/HandlerResourceProvider
java.lang.NoClassDefFoundError: com/tis/nablarch/mcp/tools/SemanticSearchTool
```

**原因分析:**
1. **コンパイルは成功**: `mvn clean compile` は `BUILD SUCCESS`
2. **クラスファイルは存在**: `target/classes/` 配下に全クラスが存在確認済み
3. **テスト実行時のみ発生**: クラスパス問題またはテスト設定の問題

**推定原因:**
- Surefire Plugin のクラスパス設定問題
- Java Module System（JPMS）との競合
- 依存関係の不整合（テスト実行時の依存解決失敗）

**コンテキスト文書との矛盾:**
- `context/nablarch-mcp-server.md` には「810件テスト（805件成功、5件スキップ）。最終実行 2026-02-04」と記載
- 4日間で **805件成功 → 34件成功** に大幅劣化
- **リグレッションが発生している可能性が高い**

### 6.3 スキップされたテスト

```
[WARNING] Tests run: 5, Failures: 0, Errors: 0, Skipped: 3
  -- in CodeSageOnnxEmbeddingClientTest
[WARNING] Tests run: 4, Failures: 0, Errors: 0, Skipped: 2
  -- in BgeM3OnnxEmbeddingClientTest
[WARNING] Tests run: 1, Failures: 0, Errors: 0, Skipped: 1
  -- in NablarchMcpServerApplicationTests
```

**スキップ理由（推定）:**
- ONNXモデルファイルが存在しない環境での条件付きスキップ
- `@EnabledIf` / `@DisabledIf` による環境依存テストの無効化
- Spring Context起動失敗による統合テストのスキップ

**評価:**
- スキップ数6件は許容範囲内（全体の4%）
- ただし、ONNXモデルのテストがスキップされるのは好ましくない（CI環境でもテスト実行すべき）

### 6.4 成功したテスト

**成功例:**
- `JinaEmbeddingClientTest`: 5件（WireMockを使用したAPI呼び出しテスト）
- `CodeSageOnnxEmbeddingClientTest`: 2件（スキップ3件除く）
- `BgeM3OnnxEmbeddingClientTest`: 2件（スキップ2件除く）
- `DefaultCodeGeneratorTest`: 複数件
- `NamingConventionHelperTest`: 複数件
- その他ユーティリティクラスのテスト

**特徴:**
- 外部依存が少ない単体テストは成功している
- Spring Context起動を伴うテストが軒並み失敗

### 6.5 テスト品質に関する問題点

**1. テストが本番コードの品質を保証していない**
- 109件エラーでも `mvn clean compile` は成功 → テストが機能していない
- Phase 3完了宣言（context文書）とテスト結果の乖離

**2. 環境依存が大きい**
- PostgreSQL起動前提（docker-compose必須）
- ONNXモデルファイル存在前提
- 環境構築手順の不備でテスト実行不可になるリスク

**3. CI/CD未導入によるリグレッション検知の遅れ**
- 2026-02-04時点では805件成功していたが、その後のコミットでリグレッション発生
- GitHub Actionsでの自動テストがあれば即座に検知できた

### 6.6 テストに関する改善提案

- [ ] **優先度: 最高** - テスト失敗原因の究明・修正（NoClassDefFoundError解決）
- [ ] **優先度: 最高** - CI/CD導入（PR時の自動テスト実行）
- [ ] **優先度: 高** - テスト環境構築手順の文書化（README.mdまたはdocs/）
- [ ] **優先度: 高** - テストコンテナ使用検討（Testcontainers for PostgreSQL）
- [ ] **優先度: 中** - ONNXモデルのモックファイル用意（テスト用軽量モデル）
- [ ] **優先度: 中** - テストカバレッジ計測（JaCoCo）

---

## 7. 改善提案一覧（優先度順）

### 🔴 優先度: 最高（即座に対応すべき）

| # | カテゴリ | 改善内容 | 期待効果 |
|---|---------|---------|---------|
| 1 | テスト | テスト失敗原因の究明・修正（NoClassDefFoundError） | 品質保証の回復 |
| 2 | テスト | CI/CD導入（GitHub Actions） | リグレッション検知 |

### 🟠 優先度: 高（Phase 4で必須）

| # | カテゴリ | 改善内容 | 期待効果 |
|---|---------|---------|---------|
| 3 | 起動 | DB接続をオプション化（STDIO modeでRAG機能OFFでも起動可能） | 開発体験向上 |
| 4 | メモリ | JVMメモリ推奨値をREADME記載 | OOM防止 |
| 5 | メモリ | ONNXモデルの遅延ロード・メモリ管理 | リソース効率化 |
| 6 | 監視 | Spring Boot Actuator導入（health, info, metrics） | 運用可視化 |
| 7 | 監視 | ヘルスチェックエンドポイント実装 | 死活監視対応 |
| 8 | ログ | logback.xml追加（ファイル出力、ローテーション） | ログ永続化 |
| 9 | Docker | Dockerfile作成（マルチステージビルド） | デプロイ自動化 |
| 10 | 設定 | DB接続情報を環境変数から読み込み | セキュリティ向上 |
| 11 | 設定 | application-prod.yaml作成 | 本番環境対応 |
| 12 | スケーラビリティ | HikariCP設定明示化（環境変数で調整可能） | 負荷対応 |

### 🟡 優先度: 中（Phase 4後半〜Phase 5）

| # | カテゴリ | 改善内容 | 期待効果 |
|---|---------|---------|---------|
| 13 | 起動 | 知識ファイルの遅延ロード | 起動時間短縮 |
| 14 | 起動 | 起動時間計測用ログ追加 | 性能分析 |
| 15 | レスポンス | Embedding処理の性能プロファイリング | ボトルネック特定 |
| 16 | レスポンス | DB検索のスロークエリ分析 | 検索性能向上 |
| 17 | メモリ | Spring Actuatorでメモリメトリクス公開 | モニタリング |
| 18 | メモリ | Spring Cache + Caffeine導入 | API コスト削減 |
| 19 | スケーラビリティ | ONNX推論を別スレッドプールで実行 | 並行処理性能向上 |
| 20 | ログ | 構造化ログ対応（JSON出力） | ログ分析効率化 |
| 21 | ログ | リクエストID/セッションIDをMDC記録 | トレーサビリティ |
| 22 | 監視 | Micrometer導入（Prometheus対応） | メトリクス集約 |
| 23 | 監視 | エラーレート監視・閾値アラート | 障害早期検知 |
| 24 | Docker | docker-compose.ymlにアプリサービス追加 | ローカル開発効率化 |
| 25 | CI/CD | Dockerイメージビルド・プッシュ自動化 | デプロイ効率化 |
| 26 | バックアップ | DBバックアップスクリプト作成 | データ保護 |
| 27 | バックアップ | バックアップ・リストア手順文書化 | 運用標準化 |

### 🟢 優先度: 低（Phase 5以降）

| # | カテゴリ | 改善内容 | 期待効果 |
|---|---------|---------|---------|
| 28 | レスポンス | テストのモック化徹底 | テスト高速化 |
| 29 | メモリ | プロファイリング実施（VisualVM等） | 詳細分析 |
| 30 | メモリ | Redis導入（キャッシュ共有） | スケールアウト対応 |
| 31 | スケーラビリティ | Redis Session導入 | 水平スケーリング |
| 32 | スケーラビリティ | 負荷テスト実施（JMeter等） | 限界値把握 |
| 33 | ログ | アクセスログ・監査ログ分離 | コンプライアンス対応 |
| 34 | 監視 | 分散トレーシング（OpenTelemetry） | マイクロサービス対応 |
| 35 | 監視 | Sentry等エラートラッキング連携 | エラー可視化 |
| 36 | CI/CD | CodeQL/Dependabot導入 | セキュリティ強化 |
| 37 | 設定 | Kubernetes ConfigMap/Secret対応 | クラウドネイティブ化 |
| 38 | バックアップ | 定期バックアップ自動化 | 運用自動化 |

---

## 8. 結論

### 8.1 現状評価

nablarch-mcp-server は **Phase 3完了時点で機能実装に注力** しており、10 Tools + 8 Resources + 6 Prompts を実装完了している。一方で、**性能・運用面は最低限の実装に留まっている**。

### 8.2 主要な問題点

1. **テスト品質の劣化**: 805件成功 → 34件成功（109件エラー）のリグレッション発生
2. **起動時DB依存**: PostgreSQL未起動だとアプリケーション起動不可
3. **監視機能の不在**: Actuator, メトリクス, ヘルスチェックなし
4. **CI/CD未整備**: 自動テスト・デプロイなし
5. **メモリ設定未文書化**: ONNXモデル使用時の推奨メモリ不明

### 8.3 推奨アクション

**Phase 4着手前に対応すべき事項:**
1. ✅ **テスト修正**: 109件のNoClassDefFoundErrorを解決し、テスト成功率を回復
2. ✅ **CI/CD導入**: GitHub Actionsで自動テスト実行
3. ✅ **監視基盤**: Spring Boot Actuator + ヘルスチェック実装

**Phase 4で取り組むべき事項:**
- Docker化（Dockerfile, docker-compose統合）
- 本番環境設定（application-prod.yaml, 環境変数化）
- ログ永続化（logback.xml, ファイル出力）
- 性能プロファイリング（起動時間、メモリ、レスポンスタイム）

### 8.4 本番稼働に向けた推奨スコアライン

| カテゴリ | 現状 | Phase 4目標 | 本番推奨 |
|---------|------|------------|---------|
| 起動パフォーマンス | 3/5 | 4/5 | 4/5 |
| レスポンス速度 | 4/5 | 4/5 | 5/5 |
| メモリ効率 | 3/5 | 4/5 | 4/5 |
| スケーラビリティ | 3/5 | 4/5 | 5/5 |
| ログ・監視 | 2/5 | 5/5 | 5/5 |
| 運用容易性 | 3/5 | 5/5 | 5/5 |
| **総合** | **3.0/5** | **4.3/5** | **4.7/5** |

---

**評価完了日**: 2026-02-10
**次回評価推奨**: Phase 4完了時（または3ヶ月後）
