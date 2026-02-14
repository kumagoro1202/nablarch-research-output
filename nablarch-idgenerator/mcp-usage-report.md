# Nablarch MCPサーバー活用評価レポート — IdGenerator調査での有用性

**作成日**: 2026-02-14
**対象調査**: IdGenerator調査（IdGenerator重複リスク）, SequenceIdGeneratorSupport調査（SequenceIdGeneratorSupport重複リスク）

---

## 1. クエリ一覧と結果概要

### 1.1 semantic_search クエリ結果

| # | クエリ | 結果件数 | 最高スコア | 内容概要 | 有用性 |
|---|-------|---------|-----------|---------|--------|
| 1 | `IdGenerator` | 5件 | 0.032 | コンポーネント設定XML例（TableIdGenerator/SequenceIdGenerator/BasicDaoContextFactory）。公式ドキュメントの採番設定ページ | ★★★☆☆ |
| 2 | `SequenceIdGenerator` | 5件 | 0.032 | #1と同一の結果。SequenceIdGenerator固有の内部動作情報なし | ★★☆☆☆ |
| 3 | `SequenceIdGeneratorSupport` | 5件 | 0.482 | UniversalDAO/@GeneratedValue/@SequenceGenerator の使い方。SequenceIdGeneratorSupport自体の情報はなし | ★★☆☆☆ |
| 4 | `採番 重複` | 5件 | 0.462 | **Doma Adaptor（無関係）**のDB設定例が最上位。採番重複に関する情報は皆無 | ★☆☆☆☆ |
| 5 | `ID生成 シーケンス` | 5件 | 0.481 | ログ機能の「実行時ID」に関する説明（ID体系の話で、シーケンス採番とは無関係） | ★☆☆☆☆ |

### 1.2 search_api クエリ結果

| # | クエリ | 結果件数 | 内容概要 | 有用性 |
|---|-------|---------|---------|--------|
| 6 | `keyword="IdGenerator"` | 2件 | (1) numbering-util パターン — IdGenerator FQCN記載 (2) nablarch-common-idgenerator モジュール情報 | ★★☆☆☆ |
| 7 | `keyword="SequenceIdGeneratorSupport"` | **0件** | 該当なし。知識ベースに含まれていない | ☆☆☆☆☆ |
| 8 | `keyword="generateId"` | **0件** | 該当なし。メソッドレベルの情報は未登録 | ☆☆☆☆☆ |

---

## 2. 有用性評価（総合）

### 2.1 総合評価: ★★☆☆☆（限定的な有用性）

IdGenerator採番重複リスク調査において、Nablarch MCPサーバーは**ほぼ活用できなかった**。

| 評価軸 | 評価 | 根拠 |
|-------|------|------|
| **調査の核心情報の提供** | ☆ | SequenceIdGeneratorSupport、generateId()メソッド、内部動作に関する情報は皆無 |
| **補助情報の提供** | ★★ | コンポーネント設定XMLの基本例は提供。コンテキスト理解の出発点にはなりうる |
| **関連クラスの発見** | ★ | IdGeneratorインターフェースとモジュール名は確認可。ただし実装クラスの詳細はなし |
| **誤解の防止** | ★ | 採番方式の選択肢（テーブル/シーケンス）は示されるが、リスク分析には不十分 |
| **一次ソースへの誘導** | ★★★ | 結果にURL付きで公式ドキュメントへのリンクが提供される点は有用 |

### 2.2 クエリ種別ごとの評価

| ツール | 評価 | コメント |
|-------|------|---------|
| `semantic_search` | ★★☆☆☆ | 結果は返るが、関連度が低い。「採番 重複」でDoma Adaptorが返るなど、検索精度に課題 |
| `search_api` | ★☆☆☆☆ | IdGeneratorのFQCNは取得できるが、実装クラスやメソッドレベルの情報が欠落 |

---

## 3. MCPサーバー情報 vs GitHub直接読解の比較

### 3.1 MCPサーバーだけで解決できた部分

| 情報 | MCPの提供内容 | 十分性 |
|------|-------------|--------|
| コンポーネント設定の基本例 | SequenceIdGenerator/TableIdGeneratorのXML設定例 | △ 基本レベルのみ |
| 公式ドキュメントURL | 採番機能の公式ページURL | ○ 一次ソースへの到達に有用 |
| FQCNの確認 | `nablarch.common.idgenerator.IdGenerator` | ○ |
| モジュール名の確認 | `nablarch-common-idgenerator` | ○ |

### 3.2 GitHubソース読解が必須だった部分（MCPでは不可能）

| 情報 | IdGenerator調査での必要性 | MCPの対応状況 |
|------|-------------------------|-------------|
| **SequenceIdGeneratorSupportのソースコード全文** | 必須（主対象） | ❌ 知識ベースに未登録 |
| **SequenceIdGeneratorのソースコード全文** | 必須（比較対象） | ❌ 未登録 |
| **generateId()メソッドの内部処理フロー** | 必須（安全性分析の核心） | ❌ 未登録 |
| **DbConnectionContextのThreadLocal実装詳細** | 必須（スレッド安全性分析） | ❌ 未登録 |
| **BasicDbConnectionのstatementReuse機構** | 重要（キャッシュ挙動分析） | ❌ 未登録 |
| **BasicSqlPStatement.retrieve()の内部動作** | 重要（executeQuery比較） | ❌ 未登録 |
| **TableIdGeneratorの内部動作とロールバック挙動** | 必須（重複メカニズム特定） | ❌ 未登録 |
| **Dialect.buildSequenceGeneratorSql()の各DB実装** | 重要（SQL生成確認） | ❌ 未登録 |
| **テスト用サブクラス（TestSequenceIdGenerator）** | 重要（設計意図の理解） | ❌ 未登録 |
| **@Deprecated情報とSequenceIdGeneratorへの移行理由** | 重要（推奨方針） | ❌ 未登録 |

### 3.3 MCPサーバーがミスリードした部分

| クエリ | 返された内容 | 問題点 |
|-------|-------------|--------|
| `"採番 重複"` | Doma Adaptorのデータベース設定 | 採番・重複とは無関係。ベクトル検索のスコアリングが不適切 |
| `"ID生成 シーケンス"` | ログ機能の「実行時ID」体系 | 「ID」「シーケンス」のキーワード一致だが、採番とは別の概念 |
| `"SequenceIdGeneratorSupport"` | UniversalDAOの@GeneratedValue説明 | SequenceIdGeneratorSupport自体の説明ではなく、関連はあるが遠い |

---

## 4. IdGenerator関連の知識カバレッジ評価

### 4.1 カバレッジマトリクス

| 知識領域 | カバー状況 | 詳細 |
|---------|----------|------|
| **コンポーネント設定方法** | ○ 部分カバー | 基本的なXML設定例は含まれる |
| **IdGeneratorインターフェース** | △ FQCN のみ | メソッドシグネチャ・Javadocなし |
| **SequenceIdGenerator** | △ 名前のみ | 内部動作・安全性特性の情報なし |
| **SequenceIdGeneratorSupport** | ❌ 未カバー | 知識ベースに存在しない |
| **TableIdGenerator** | △ 名前のみ | 重複リスクの情報なし |
| **FastTableIdGenerator** | ❌ 未カバー | 知識ベースに存在しない |
| **DbConnectionContext（ThreadLocal）** | ❌ 未カバー | DB接続管理の内部動作なし |
| **Dialect（SQL生成）** | ❌ 未カバー | buildSequenceGeneratorSql()の情報なし |
| **採番時のスレッド安全性** | ❌ 未カバー | synchronizedの有無、排他制御の情報なし |
| **ロールバック時の挙動** | ❌ 未カバー | SEQUENCE vs テーブルの違い情報なし |
| **デフォルト設定（TableIdGenerator）** | ❌ 未カバー | デフォルト設定情報なし |
| **@Deprecated情報** | ❌ 未カバー | 移行ガイダンスなし |

### 4.2 カバレッジ数値

- **IdGenerator関連の知識領域**: 12領域
- **カバーされている領域**: 1領域（コンポーネント設定方法の基本）
- **部分的にカバー**: 3領域（FQCN/名前のみ）
- **未カバー**: 8領域
- **カバレッジ率**: **約8%**（完全カバー基準）/ **約33%**（部分含む）

---

## 5. 知識拡充提案

### 5.1 追加すべき知識領域（優先度順）

| 優先度 | 知識領域 | 理由 | 拡充方法 |
|--------|---------|------|---------|
| **P1** | IdGenerator実装クラスの内部動作と安全性特性 | 採番は全Nablarchプロジェクトに共通の関心事。重複リスクの理解は業務システムの信頼性に直結 | YAML知識追加（パターン定義） |
| **P1** | TableIdGenerator vs SequenceIdGenerator vs FastTableIdGeneratorの使い分けガイド | 誤った選択が重複リスクに直結する。IdGenerator調査で明らかになった | YAML知識追加（ベストプラクティス） |
| **P2** | DbConnectionContextのThreadLocal設計とスレッド安全性 | Nablarchの並行処理理解の基盤。採番だけでなくDB操作全般に関わる | YAML知識追加 + pgvectorへのソースコード取込 |
| **P2** | @Deprecated API一覧と移行ガイド | SequenceIdGeneratorSupport以外にも旧APIが存在する可能性。移行支援は高需要 | YAML知識追加 |
| **P3** | Dialect パターンとDB差異吸収の仕組み | カスタムDialect作成や対応DB追加時に必要 | pgvectorへのソースコード取込 |
| **P3** | Nablarchデフォルト設定一覧 | 設定変更ガイドの情報をMCPから即座に引けるようにする | YAML知識追加 |

### 5.2 具体的な拡充方法

#### 方法A: YAML知識ファイル追加（P1項目向け）

```yaml
# 提案: knowledge/idgenerator-patterns.yaml
category: idgenerator
patterns:
  - id: safe-sequence-pattern
    name: "安全なシーケンス採番パターン"
    description: |
      SequenceIdGeneratorを使用した採番。DB SEQUENCEのNEXTVALによる
      アトミック排他制御で、マルチスレッド・マルチプロセスでも重複しない。
      ロールバック時も欠番のみで重複は発生しない。
    fqcn: nablarch.common.idgenerator.SequenceIdGenerator
    recommendation: "推奨"
    risk_level: "安全"
    source_url: "https://nablarch.github.io/docs/LATEST/doc/.../generator.html"

  - id: dangerous-table-pattern
    name: "危険なテーブル採番パターン"
    description: |
      TableIdGeneratorは業務トランザクション内で採番するため、
      ロールバック時に値が戻り、重複が発生する。
      FastTableIdGenerator（独立トランザクション）を使用すべき。
    fqcn: nablarch.common.idgenerator.TableIdGenerator
    recommendation: "非推奨（FastTableIdGeneratorを使用）"
    risk_level: "危険 — ロールバック時に重複発生"
```

#### 方法B: pgvectorへのソースコード取込（P2-P3項目向け）

`nablarch-common-idgenerator-jdbc`と`nablarch-core-jdbc`の主要クラスのソースコードをpgvectorに取り込み、semantic_searchで「採番 内部動作」「ThreadLocal スレッド安全」等のクエリに対してソースコードレベルの情報を返せるようにする。

対象クラス:
- `SequenceIdGenerator.java` — generateId()の全コード
- `SequenceIdGeneratorSupport.java` — @Deprecated旧版の全コード
- `TableIdGenerator.java` — ロールバック重複の根拠
- `FastTableIdGenerator.java` — 独立トランザクション設計
- `DbConnectionContext.java` — ThreadLocal設計
- `BasicDbConnection.java` — statementReuse機構

### 5.3 期待される改善効果

| 改善前（現状） | 改善後（知識拡充後） |
|-------------|-----------------|
| `search_api:"SequenceIdGeneratorSupport"` → 0件 | 実装クラス情報・@Deprecated警告・移行ガイドが返る |
| `semantic_search:"採番 重複"` → Doma Adaptor（無関係） | TableIdGeneratorの重複リスク警告 + 安全な代替パターンが返る |
| IdGenerator調査にGitHub読解100%必要 | 基本的な安全性判断の80%をMCPで完結可能に |

---

## 6. 総括

### 6.1 IdGenerator調査調査でのMCPサーバー活用度

| 調査項目 | MCP活用度 | GitHubソース読解 |
|---------|----------|----------------|
| クラス存在確認・FQCN取得 | 10% | 90% |
| 内部動作分析（generateId()） | 0% | 100% |
| スレッド安全性分析 | 0% | 100% |
| ロールバック挙動分析 | 0% | 100% |
| コンポーネント設定方法 | 30% | 70% |
| 安全/危険パターン特定 | 0% | 100% |
| **総合** | **約5%** | **約95%** |

### 6.2 結論

IdGenerator調査レベルの深い技術調査では、現在のNablarch MCPサーバーは**ほぼ役に立たなかった**。これはMCPサーバーの設計が悪いのではなく、**知識ベースにIdGenerator関連の詳細情報が未登録**であることが原因。

公式ドキュメントの採番ページの情報は含まれているが、それはコンポーネント設定の基本例に留まり、内部動作・安全性特性・リスク分析に必要な情報レベルには遠く及ばない。

**知識拡充により劇的な改善が見込める**分野であり、P1項目（IdGenerator安全性パターン + 使い分けガイド）の追加を強く推奨する。
