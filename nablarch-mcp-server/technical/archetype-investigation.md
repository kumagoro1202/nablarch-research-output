# アーキタイプ知識欠落調査レポート

**日時**: 2026-02-12
**対象**: nablarch-mcp-server（kumagoro1202）

---

## 1. 調査結果サマリ（原因の一言結論）

**複合原因: YAML知識にアーキタイプ情報が完全欠落 + BM25検索が日本語クエリで事実上機能せず + Claudeのtool選択がsearch_api優先**

pgvectorにはアーキタイプ情報が存在するが（20件以上のチャンク）、以下3層の障壁により到達困難：
1. search_api（YAML知識）にプロジェクト作成/アーキタイプ情報が一切ない
2. semantic_searchのBM25検索が日本語スペースなしクエリで0件を返す
3. ベクトル検索でもチャンク内HTMLノイズがembedding品質を低下させている

---

## 2. OfficialDocsIngester取込URL一覧

### 設定
- **baseUrl**: `https://nablarch.github.io/docs/LATEST/doc/`
- **方式**: インデックスページからhref属性を正規表現で抽出 → 各HTMLをクローリング
- **条件**: `@ConditionalOnProperty(name = "nablarch.mcp.ingestion.enabled", havingValue = "true")`

### 取込状況
- **全取込チャンク数**: 1,485件（全て`nablarch-official-docs`ソース）
- **Embedding付き**: 1,485件（100%完備）
- **code_chunks**: 0件（空）

### アーキタイプ関連ページの取込状況

| URL | 取込済み | チャンクID |
|-----|---------|-----------|
| `.../blank_project/index.html` | ✅ | 6434 |
| `.../blank_project/MavenModuleStructures/index.html` | ✅ | 6417-6424 |
| `.../blank_project/FirstStep.html` | ✅ | 6415 |
| `.../blank_project/setup_blankProject/setup_NablarchBatch.html` | ✅ | 5463-5466 |
| `.../blank_project/setup_blankProject/setup_NablarchBatch_Dbless.html` | ✅ | 5467 |
| `.../blank_project/setup_blankProject/setup_Jbatch.html` | ✅ | 5459 |
| `.../blank_project/setup_containerBlankProject/setup_ContainerBatch.html` | ✅ | 5469 |
| `.../blank_project/setup_containerBlankProject/setup_ContainerBatch_Dbless.html` | ✅ | 5472-5473 |

**結論**: アーキタイプ関連ページは全て取込済み。

### FintanIngester
- **baseUrl**: `https://fintan.jp/`
- **searchTags**: `["Nablarch"]`
- **取込済み件数**: **0件**（Fintanからの取込は一度も実行されていない）

---

## 3. pgvectorデータ状況

### テーブルスキーマ
```
document_chunks: 1,485件（embedding: vector(1024), 100%埋め込み済み）
code_chunks: 0件
```

### ソース分布
| ソース | 件数 |
|--------|------|
| nablarch-official-docs | 1,485 |
| fintan | 0 |

### アーキタイプ関連チャンク
`archetype`または`アーキタイプ`を含むチャンク: **20件以上**

代表例:
- ID 6417: 「Mavenアーキタイプの構成 — Nablarchの提供するMavenアーキタイプの構成と各ディレクトリ・ファイルの概要を...」(1,805文字)
- ID 6418: 「Mavenプロジェクト名 / 生成元のMaven archetype: pj-batch-dbless → nablarch-batch-dbless-archetype, pj-batch-ee → nablarch-batch-ee-archetype...」(1,996文字)
- ID 5463: 「Nablarchバッチプロジェクトの初期セットアップ — Nablarchバッチプロジェクトの生成、動作確認...」(1,996文字)
- ID 6434: 「ブランクプロジェクト — Nablarchのアーキタイプからプロジェクトのひな形（ブランクプロジェクト）を生成する方法...」(832文字)

**全てembedding付き（has_embedding = true）**

### チャンク品質の問題
チャンクにSphinx HTML由来のナビゲーションヘッダが残存:
```
Mavenアーキタイプの構成 — ∇Nablarch  6u3 ドキュメント Docs » Nablarchアプリケーションフレームワーク » アプリケーションフレームワーク » ブランクプロジェクト » Mavenアーキタイプの構成 GitHub English ...
```
このノイズがembeddingの意味的精度を低下させている。

---

## 4. semantic_searchテスト結果

### 4.1 BM25検索テスト（SQL直接実行）

| クエリ | 方式 | 結果件数 | 備考 |
|--------|------|---------|------|
| `バッチプロジェクト作成計画して` | ILIKE完全一致 | **0件** | クエリ全体がそのまま部分文字列として存在しない |
| `バッチ AND プロジェクト AND 作成` | ILIKE AND結合 | **34件** | 個別キーワードに分割すればヒットする |
| `archetype generate nablarch` | ts_rank | **10件** | 英語キーワードは問題なくヒット |
| `バッチ プロジェクト 作成` | plainto_tsquery | **0件** | PostgreSQL simple configでは日本語トークナイズ不可 |

### 4.2 BM25SearchServiceの動作分析

`extractKeywords()`メソッドはスペース区切りでトークン化:
- 入力: `バッチプロジェクト作成計画して`（スペースなし）
- 出力: `["バッチプロジェクト作成計画して"]`（1キーワード）
- ILIKE: `content ILIKE '%バッチプロジェクト作成計画して%'` → **0件**

**日本語の自然文（スペースなし）が入力された場合、BM25検索は事実上機能しない。**

### 4.3 ベクトル検索の推定

ベクトル検索はEmbedding類似度で動作するため、日本語クエリでも理論上はヒットする。
ただし以下の品質低下要因がある:
- チャンク内のHTMLナビゲーションノイズがembedding品質を薄めている
- 「バッチプロジェクト作成計画して」というタスク指向クエリと、「初期セットアップ手順」というドキュメントタイトルのセマンティックギャップ

### 4.4 MCP Server実際の起動テスト

※provider=local起動にはONNXモデルの配置が必要であり、今回は直接起動テストは実施せず、SQL直接検索で代替。

---

## 5. YAML知識確認結果

### 知識ファイル一覧（17ファイル）
```
基本7ファイル:
  handler-catalog.yaml, handler-constraints.yaml, api-patterns.yaml,
  module-catalog.yaml, error-catalog.yaml, config-templates.yaml,
  design-patterns.yaml

カタログ7ファイル:
  data-io-catalog.yaml, validation-catalog.yaml, mail-catalog.yaml,
  message-catalog.yaml, utility-catalog.yaml, log-catalog.yaml,
  security-catalog.yaml

その他3ファイル:
  example-catalog.yaml, antipattern-catalog.yaml, version-info.yaml
```

### アーキタイプ関連検索

| 検索キーワード | 結果 |
|---------------|------|
| `archetype` | **0件** |
| `blank_project` | **0件** |
| `プロジェクト作成` | **0件** |
| `初期セットアップ` | **0件** |

**結論**: YAML知識にプロジェクト作成/アーキタイプ/ブランクプロジェクトに関する情報は一切存在しない。

### MCP Promptsのアーキタイプ誘導

6つのPrompt（setup-handler-queue, create-action, review-config, explain-handler, migration-guide, best-practices）のいずれにも、プロジェクトセットアップへの誘導なし。

---

## 6. 原因分析

### 根本原因（3層の障壁）

```
┌─────────────────────────────────────────────────────────┐
│ 障壁1: search_api（YAML知識）にアーキタイプ情報なし      │
│   → Claudeがsearch_apiを使う場合、到達不可能             │
├─────────────────────────────────────────────────────────┤
│ 障壁2: BM25検索が日本語で機能しない                      │
│   → 「バッチプロジェクト作成計画して」で0件              │
│   → hybrid searchのBM25側が死んでいる状態                │
├─────────────────────────────────────────────────────────┤
│ 障壁3: チャンク品質（HTMLナビゲーションノイズ）          │
│   → ベクトル検索のembedding精度低下                      │
│   → アーキタイプ情報がランク上位に出にくい               │
└─────────────────────────────────────────────────────────┘
```

### Claudeのtool選択パターン（推定）

「バッチプロジェクト作成計画して」という質問に対して:
1. Claude は`search_api`（YAML知識検索）を最初に使う可能性が高い → archetype情報に到達不可
2. `semantic_search`を使った場合でも、BM25が0件でベクトル検索のみに依存
3. ベクトル検索がアーキタイプ関連チャンクを上位に返すかは不確実（HTMLノイズによる精度低下）

### 補足: init-knowledge.shのバグ（既知）
- L68, L85, L127-128: サービス名が`postgres`になっているが、実際は`pgvector`
- ただし取込自体はmvn spring-boot:run経由で実行されるため、このバグは取込結果に影響しない

---

## 7. 対策案（優先度付き）

### P1（最優先・即効性高）: YAML知識にアーキタイプ情報を追加

**効果**: search_apiでアーキタイプ情報に確実に到達可能になる
**工数**: 小（新規YAMLファイル1本追加）

```yaml
# knowledge/project-setup-catalog.yaml（新規）
project_setup:
  archetype:
    description: "Nablarch公式Mavenアーキタイプによるプロジェクト生成"
    source_url: "https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/blank_project/index.html"
    archetypes:
      - name: nablarch-batch-archetype
        project_name: pj-batch
        description: "Nablarchバッチアプリケーション"
        command: |
          mvn archetype:generate -DarchetypeGroupId=com.nablarch.archetype -DarchetypeArtifactId=nablarch-batch-archetype -DarchetypeVersion=6u3 ...
      - name: nablarch-batch-dbless-archetype
        project_name: pj-batch-dbless
        description: "Nablarchバッチアプリケーション（DB接続無し）"
      # ... 他のアーキタイプ
```

NablarchKnowledgeBase.javaのCATALOG_FILESに`"project-setup-catalog.yaml"`を追加。

### P2（重要・BM25検索改善）: 日本語クエリのキーワード分割改善

**効果**: BM25検索が日本語で機能するようになる
**工数**: 中（BM25SearchService.extractKeywords()の改修）

現状: `バッチプロジェクト作成計画して` → `["バッチプロジェクト作成計画して"]`（1キーワード）
改善案:
1. N-gram分割（3-gram等）で部分一致を改善
2. または、semantic_searchのPromptで「キーワードはスペース区切りで入力」とガイド
3. または、LLMにクエリ分解を依頼するQueryAnalyzer統合（TODO: 担当者DのWBS 2.2.14）

### P3（重要・チャンク品質改善）: HTMLノイズ除去

**効果**: embedding精度向上、ベクトル検索の検索精度向上
**工数**: 中（HtmlDocumentParser改修 or 取込後のクリーニング）

現状のチャンク:
```
Mavenアーキタイプの構成 — ∇Nablarch  6u3 ドキュメント Docs » ... GitHub English ...
```

除去すべきノイズ:
- Sphinxナビゲーション（「Docs » ... » ...」）
- GitHub/Englishリンク
- 「∇Nablarch 6u3 ドキュメント」ヘッダ

### P4（中期・MCP Prompt改善）: プロジェクトセットアップPromptの追加

**効果**: Claudeがプロジェクト作成の質問を受けた際にアーキタイプ使用をガイドできる
**工数**: 中（新規Promptクラス + McpServerConfig登録）

例: `setup-project` Promptの追加
```
引数: project_type (web, rest, batch, messaging)
応答: そのタイプのアーキタイプコマンドと初期セットアップ手順
```

### P5（低優先・Fintan取込）: FintanIngesterの実行

**効果**: Fintan記事のNablarch知識が追加される
**工数**: 小（実行のみ）
**備考**: 現在0件。Fintan記事にアーキタイプ関連があれば追加の知識源になる

---

## 付録: 検証に使用したSQLクエリ

```sql
-- アーキタイプ関連チャンク検索
SELECT id, source, url, substring(content, 1, 80) FROM document_chunks
WHERE content ILIKE '%archetype%' OR content ILIKE '%アーキタイプ%' LIMIT 20;

-- BM25テスト（日本語）
SELECT count(*) FROM document_chunks
WHERE content ILIKE '%バッチプロジェクト作成計画して%';  -- → 0件

-- BM25テスト（キーワード分割）
SELECT count(*) FROM document_chunks
WHERE content ILIKE '%バッチ%' AND content ILIKE '%プロジェクト%' AND content ILIKE '%作成%';  -- → 34件

-- Embedding有無確認
SELECT count(*) as total, count(embedding) as with_embedding FROM document_chunks;
-- → total=1485, with_embedding=1485
```
