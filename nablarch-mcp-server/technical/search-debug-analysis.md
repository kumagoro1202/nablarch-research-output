# 検索0件デバッグ調査レポート

**日時**: 2026-02-12
**対象**: nablarch-mcp-server（kumagoro1202）

---

## 1. 結論（0件の根本原因を一言で）

**全1,485件のdocument_chunksのapp_type/moduleカラムがNULLのため、`appType=batch`フィルタで全チャンクが排除される。** これがsemantic_searchの0件の主因。search_apiはキーワード全文をsingle-string containsで検索するためトークン化されたキーワードにヒットしない。

---

## 2. ユーザーの5テストケースの根本原因分析

### テスト1: semantic_search(hybrid, appType=batch) → 0件

**クエリ**: `"Nablarch batch application project setup Maven archetype"`, appType=batch

**原因**: `WHERE app_type = 'batch'` フィルタが全チャンクを排除

```sql
-- 証拠
SELECT count(*) FROM document_chunks WHERE app_type IS NOT NULL;
-- → 0件（全1,485件がNULL）

SELECT count(*) FROM document_chunks WHERE app_type = 'batch';
-- → 0件
```

**処理フロー**:
1. SemanticSearchTool → HybridSearchService.search(query, filters={appType=batch}, 50, HYBRID)
2. BM25SearchService.searchTable(): `content ILIKE '%Nablarch%' AND ... AND app_type = :app_type` → 0件
3. VectorSearchService.searchTable(): `embedding <=> vector ... AND app_type = :app_type` → 0件
4. 両方0件 → HybridSearchService返却0件
5. SemanticSearchTool: candidates=0 → rerankerスキップ → "検索結果なし"

**補足**: appTypeフィルタがなければBM25で2件、ベクトル検索は多数ヒットする:
```sql
-- appTypeフィルタなしで7キーワードAND ILIKE
SELECT count(*) FROM document_chunks WHERE
  content ILIKE '%Nablarch%' AND content ILIKE '%batch%' AND
  content ILIKE '%application%' AND content ILIKE '%project%' AND
  content ILIKE '%setup%' AND content ILIKE '%Maven%' AND content ILIKE '%archetype%';
-- → 2件

-- ベクトル検索: アーキタイプチャンク同士のコサイン類似度
SELECT 1 - (embedding <=> (SELECT embedding FROM document_chunks WHERE id = 6418)) AS sim
FROM document_chunks WHERE id = 5463;
-- → 0.795（高い類似度）
```

### テスト2: design_handler_queue(batch) → 成功 ✅

YAML知識ベース（handler-catalog.yaml）のbatchエントリから直接取得。pgvectorを使わないため影響なし。

### テスト3: semantic_search(keyword) → 0件

**クエリ**: `"nablarch-example-batch バッチアプリケーション構成 コンポーネント定義"`

**原因**: 3キーワードのAND ILIKE結合が厳しすぎて0件

```
extractKeywords() → ["nablarch-example-batch", "バッチアプリケーション構成", "コンポーネント定義"]
SQL: content ILIKE '%nablarch-example-batch%' AND
     content ILIKE '%バッチアプリケーション構成%' AND
     content ILIKE '%コンポーネント定義%'
```

```sql
-- "nablarch-example-batch" は3チャンクにしか存在しない
SELECT count(*) FROM document_chunks WHERE content ILIKE '%nablarch-example-batch%';
-- → 3件

-- 3件のうち他2キーワードも含むものは
SELECT count(*) FROM document_chunks WHERE
  content ILIKE '%nablarch-example-batch%' AND
  content ILIKE '%バッチアプリケーション構成%' AND
  content ILIKE '%コンポーネント定義%';
-- → 0件
```

**問題**: BM25SearchServiceのextractKeywords()はスペースで分割するが、日本語部分はスペースなしの塊として扱われる。「バッチアプリケーション構成」全体が1つのILIKEキーワードになり、この完全部分文字列が含まれるチャンクがない。

### テスト4: generate_code(batch) → 成功 ✅

YAML知識ベース（api-patterns.yaml）のbatch-actionパターンからコード生成。pgvectorを使わないため影響なし。

### テスト5: search_api(keyword="batch action DataReader", category=batch) → 0件

**原因**: NablarchKnowledgeBase.search()がキーワード全文をsingle-string containsで検索

```java
// NablarchKnowledgeBase.java L151
String lowerKeyword = keyword.toLowerCase();
// → "batch action datareader"

// matchesKeyword for ApiPatternEntry (L667-669)
private boolean matchesKeyword(ApiPatternEntry p, String kw) {
    return ci(p.name, kw) || ci(p.description, kw) || ci(p.fqcn, kw) || ci(p.category, kw);
}
// ci() = text.toLowerCase().contains(kw)
```

batch-action パターンの各フィールド:
| フィールド | 値 | contains("batch action datareader") |
|-----------|-----|------|
| name | `batch-action` | ❌ (ハイフン ≠ スペース) |
| description | `標準バッチアクションパターン。DataReaderでデータを読み取り...` | ❌ |
| fqcn | `nablarch.fw.action.BatchAction` | ❌ |
| category | `batch` | ❌ |

**さらに**: category=batchフィルタにより検索対象が限定:
- APIパターン: category=batchのもののみ（batch-action, resident-batchの2件のみ）
- モジュール検索: **スキップ**（category≠"library"/"module"/blank）
- ハンドラ検索: **スキップ**（category≠"handler"/blank）
- 設計パターン: **スキップ**（category≠blank）
- エラー検索: **スキップ**（category≠"error"/blank）
- カタログ知識: matchesCategory("batch") → 7カタログファイルのどのセクションにも"batch"カテゴリなし → 0件

---

## 3. VectorSearchServiceの閾値・スコアリングロジック分析

### 閾値
**閾値は存在しない。** VectorSearchService.searchTable()のSQL:
```sql
SELECT id, content, 1 - (embedding <=> CAST(:query_vec AS vector)) AS vector_score, ...
FROM document_chunks
WHERE embedding IS NOT NULL
  [AND app_type = :app_type]  -- ← これが問題
ORDER BY embedding <=> CAST(:query_vec AS vector)
LIMIT :top_k
```

スコアの下限フィルタ（例: `WHERE vector_score > 0.5`）はない。topK件を無条件に返す設計。
→ **フィルタさえなければ結果は必ず返る。**

### HybridSearchServiceのRRFロジック
- BM25=0件、Vector=0件の場合: 空リストを返す（L140-143）
- 一方のみ0件の場合: もう一方をそのままtopKに切り詰めて返す（L146-151）
- **BM25=0でもVector側に結果があれば返される設計。問題はフィルタで両方0件になること。**

---

## 4. BM25SearchServiceの問題分析

### extractKeywords()のロジック
```java
String[] tokens = query.trim().split("\\s+");  // スペース分割のみ
```

- 英語: `"Nablarch batch ... archetype"` → 7トークン → 7キーワードのAND ILIKE
- 日本語混在: `"nablarch-example-batch バッチアプリケーション構成"` → 2トークン（日本語部分がスペースなしの塊）

### AND結合の過剰な厳格性
全キーワードがAND結合: `ILIKE '%kw1%' AND ILIKE '%kw2%' AND ...`

7キーワードのAND結合では1チャンクに全キーワードが含まれる必要があり、非常に制限的。
```sql
-- 7キーワードAND: 2件
-- 3キーワードAND (Nablarch, batch, archetype): 11件
```

---

## 5. NablarchKnowledgeBase（search_api）の問題分析

### Single-string contains問題
`search(keyword, category)` は `keyword` 全体を1つの部分文字列として各フィールドに対してcontains検索する。
トークン化しない。

| 検索方法 | 期待結果 | 実際 |
|---------|---------|------|
| `"batch"` (1語) | batch-action等にヒット | ✅ ヒットする |
| `"DataReader"` (1語) | description内にヒット | ✅ ヒットする |
| `"batch action DataReader"` (複合) | OR結合でヒット | ❌ 全文substring検索で0件 |

### categoryフィルタの厳格性
category=batchを指定すると:
- APIパターンのみ検索（category=batchの2件のみ）
- モジュール/ハンドラ/設計パターン/エラー/カタログ知識は**全てスキップ**
- BatchAction関連のモジュール情報（module-catalog.yaml）やエラー情報（error-catalog.yaml）にも到達できない

---

## 6. デバッグテスト結果

### ベクトル検索の動作確認（フィルタなし）

pgvector内のアーキタイプチャンク間のコサイン類似度:
```sql
SELECT id, 1 - (embedding <=> (SELECT embedding FROM document_chunks WHERE id = 6418))
  AS cosine_sim, substring(content, 1, 60)
FROM document_chunks
WHERE content ILIKE '%archetype%'
ORDER BY cosine_sim DESC LIMIT 5;
```

| チャンクID | コサイン類似度 | 内容 |
|-----------|--------------|------|
| 6418 | 1.000 | Mavenプロジェクト名/生成元のMaven archetype一覧 |
| 6417 | 0.832 | Mavenアーキタイプの構成 |
| 6432 | 0.820 | nablarch-bom内の定義 |
| 6747 | 0.815 | Blank Project (English) |
| 5463 | 0.795 | Nablarchバッチプロジェクトの初期セットアップ |

→ **ベクトル検索は正常。フィルタなしなら高い類似度で結果が返る。**

### pgvectorメタデータの全貌

| カラム | 全1,485件の値 |
|--------|-------------|
| app_type | **全てNULL** |
| module | **全てNULL** |
| source | `nablarch-official-docs` (全件) |
| source_type | `documentation` (全件) |
| language | `ja` (推定全件) |
| fqcn | **全てNULL** |

→ OfficialDocsIngester.toEntity()がapp_type, module, fqcnを設定しないため。

---

## 7. 修正方針（具体的に）

### 修正A（最優先・即効性最高）: メタデータフィルタをNULL許容にする

**対象**: VectorSearchService.appendFilters() + BM25SearchService.appendFilters()

**現状**:
```java
if (filters.appType() != null) {
    sql.append(" AND app_type = :app_type");
}
```

**修正案**:
```java
if (filters.appType() != null) {
    sql.append(" AND (app_type = :app_type OR app_type IS NULL)");
}
```

**効果**: app_type=NULLのチャンクもフィルタを通過する。全チャンクが排除される問題を即座に解決。
**リスク**: 将来Fintanやコードチャンクでapp_typeが設定された場合に、無関係なチャンクも含まれる可能性。ただし現状の全排除よりは遥かに良い。

### 修正B（重要・OfficialDocsIngester改善）: メタデータ自動付与

**対象**: OfficialDocsIngester.toEntity() + HtmlDocumentParser

URLパスからapp_typeを推定して設定:
```
/batch/ → app_type = "batch"
/web/ → app_type = "web"
/handlers/web/ → app_type = "web"
/blank_project/ → app_type = NULL (全タイプ共通)
```

### 修正C（重要・search_api改善）: キーワードトークン化

**対象**: NablarchKnowledgeBase.search()

**修正案**: keywordをスペース分割し、各トークンのOR結合でマッチング:
```java
String[] tokens = keyword.toLowerCase().split("\\s+");
// 各フィールドに対して任意のトークンがcontainsすればマッチ
```

### 修正D（中・BM25改善）: AND結合の緩和

**対象**: BM25SearchService.searchTable()

全キーワードANDではなく、一定数以上のキーワードが含まれればヒット（OR + 重み付け）:
```sql
WHERE (content ILIKE '%kw1%')::int + (content ILIKE '%kw2%')::int + ... >= N
```

### 修正E（中・categoryフィルタ改善）: search_apiのカテゴリスコーピング緩和

**対象**: NablarchKnowledgeBase.search()

category=batchでもモジュール/エラー/カタログ知識を検索対象に含める。
現状はcategory=batchだとAPIパターンのみ、ハンドラ/モジュール/エラーはスキップされる。

### 優先順位

| 優先度 | 修正 | 効果 | 工数 |
|-------|------|------|------|
| P1 | A: NULL許容フィルタ | semantic_searchで即結果が返る | 極小（2行変更） |
| P1 | C: キーワードトークン化 | search_apiで複合キーワードがヒット | 小 |
| P2 | B: メタデータ自動付与 | フィルタが意味を持つようになる | 中（URL解析ロジック追加） |
| P2 | D: AND結合緩和 | BM25の検索再現率向上 | 中 |
| P3 | E: カテゴリスコーピング | search_apiの検索範囲拡大 | 小 |
