# Nablarch知識提供の最適手段調査レポート

**作成日**: 2026-02-10
**対象**: nablarch-mcp-server — 知識提供アーキテクチャの最適化

---

## 1. エグゼクティブサマリ

### 結論

Nablarch知識提供において、**現行のMCP Server方式は構造化知識の提供手段として妥当**だが、**単独では不十分**。Nablarch知識の特性（FQCN正確性、ハンドラキュー順序依存、API仕様の構造性）に最適な手段は、**MCP Server + Context Engineering（CLAUDE.md/Rules File）のハイブリッド方式**であり、Embedding RAGやAgentic Searchは補助的に活用すべきである。

Boris Cherny氏（Claude Code開発者）がEmbedding+ベクトル検索からAgentic Searchに移行した事例は**コードベース検索（Code Search）**に特化した判断であり、**フレームワーク知識提供**には直接適用できない。

### 推奨方針（1行）

**MCP Server（構造化知識）+ Context Engineering（CLAUDE.md）を基盤とし、大規模知識にはAgentic RAGを段階的に導入する三層ハイブリッドアーキテクチャ**

---

## 2. Boris Cherny氏の発言調査

### 2.1 原典の特定

Boris Cherny氏の発言は以下で確認された:

| ソース | 種別 | 確認状況 |
|--------|------|---------|
| [Latent Space Podcast "Claude Code: Anthropic's Agent in Your Terminal"](https://www.latent.space/p/claude-code) | ポッドキャスト | **原典確認済み** |
| [Threads投稿 by @jatinw21](https://www.threads.com/@jatinw21/post/DUPCuyBCbwM/) | SNS（要約） | 確認済み |
| [X(Twitter) @aakashgupta による引用](https://x.com/aakashgupta/status/2018933856460775597) | SNS（引用） | 確認済み |
| [SmartScope Blog 分析記事](https://smartscope.blog/en/ai-development/practices/rag-debate-agentic-search-code-exploration/) | ブログ分析 | 確認済み |

### 2.2 発言の正確な内容

Boris Cherny氏（Anthropic, Claude Code開発者）の主要発言:

> **RAGからの移行について:**
> "Early versions of Claude Code used RAG + a local vector db, but we found pretty quickly that agentic search generally works better."

> **Agentic Searchの定義:**
> "What does agentic search mean in this context? It means letting the agent look up information in however many search cycles it needs, just using regular code searching tools like glob and grep."

> **性能比較:**
> "It outperformed everything — by a lot."
> ベンチマークの根拠を問われた際: "just vibes — internal vibes. There's some internal benchmarks also, but mostly vibes. It just felt better."

> **セキュリティ:**
> Anthropic自身のコードベースすら第三者にアップロードすることへの懸念。RAGインデックスの保管先セキュリティリスク。

> **トレードオフ:**
> "At the cost of latency and tokens, you now have really awesome search without the security downsides."

### 2.3 重要な文脈: コード検索 vs 知識提供

Boris氏の判断には**明確なスコープ制限**がある:

- **対象**: コードベース検索（Code Search）— ファイル内の関数・クラス・変数の特定
- **ツール**: glob, grep, find — ファイルシステム上の文字列マッチング
- **前提**: コードは「正解」がファイルシステム上に存在する。grepで見つかる
- **非対象**: フレームワーク知識、API仕様、設計パターン、ベストプラクティス

**Nablarch知識提供はBoris氏のスコープ外**。Nablarchのハンドラカタログ・FQCN・設計パターンはファイルシステム上にgrep可能な形で存在しない（公式ドキュメントやGitHubソースコード上に散在する）。

### 2.4 コミュニティの反応・議論

Boris氏の発言に対する主要な反論:

| 出典 | 論点 |
|------|------|
| [Milvus Blog](https://milvus.io/blog/why-im-against-claude-codes-grep-only-retrieval-it-just-burns-too-many-tokens.md) | トークン消費量が過大。grepは構造理解がなく不要なマッチが大量に返る |
| [Relace.ai](https://www.relace.ai/blog/fast-agentic-search) | Agentic Searchは20ターン必要で遅い。並列化で4倍高速化した事例を紹介 |
| [Alberto Roura](https://albertoroura.com/vector-rag-agentic-search-why-not-both/) | 二項対立ではなくハイブリッドが最適。概念検索はVector RAG、精密検索はAgentic |
| [Mark Hendrickson](https://markmhendrickson.com/posts/agentic-search-and-the-truth-layer/) | Agentic Searchは推論であり保証ではない。信頼性層（Truth Layer）が別途必要 |

**2026年のコンセンサス**: 「Agentic Searchを基盤に、セマンティックインデックスを必要箇所に限定適用」が実務的な最適解。二項対立ではなく**ハイブリッド**。

---

## 3. 知識提供手段の網羅的比較

### 3.1 比較対象7手段

| # | 手段 | 概要 |
|---|------|------|
| A | **Embedding + ベクトル検索（RAG）** | 知識をベクトル化し類似検索で取得 |
| B | **Agentic Search** | エージェントがgrep/glob等で能動的に検索・推論 |
| C | **MCP Server（現行方式）** | 構造化知識をTools/Resources/Promptsで提供 |
| D | **Rules File / System Prompt** | CLAUDE.md等に知識を直接記述 |
| E | **Fine-tuning** | モデル自体にフレームワーク知識を学習 |
| F | **Agentic RAG** | エージェントがRAGを動的にオーケストレーション |
| G | **Context Engineering統合** | 上記を組み合わせた最適化アプローチ |

### 3.2 7軸比較表

| 比較軸 | A: Embedding RAG | B: Agentic Search | C: MCP Server | D: Rules File | E: Fine-tuning | F: Agentic RAG |
|--------|:-:|:-:|:-:|:-:|:-:|:-:|
| **正確性** | 3 | 4（コード） / 2（知識） | 5 | 4 | 3 | 4 |
| **コスト** | 3（インフラ） | 2（トークン大） | 4 | 5 | 1（学習コスト大） | 2 |
| **メンテナンス性** | 2（再インデックス） | 5（不要） | 3（YAML更新） | 4（ファイル編集） | 1（再学習） | 3 |
| **レイテンシ** | 4 | 2（多ターン） | 5 | 5（即時） | 5 | 2 |
| **スケーラビリティ** | 4 | 3 | 3 | 1（トークン制限） | 4 | 4 |
| **開発工数** | 3 | 5（ゼロ） | 2（実装必要） | 5（記述のみ） | 1 | 2 |
| **UX** | 3 | 4 | 4 | 3 | 4 | 4 |

**評価基準**: 5=最優秀、4=良好、3=普通、2=課題あり、1=不適

### 3.3 各手段の詳細分析

#### A: Embedding + ベクトル検索（RAG）

**仕組み**: ドキュメントをチャンク分割 → Embeddingモデルでベクトル化 → ベクトルDB格納 → クエリの類似検索でTop-K取得 → LLMに注入

**強み**:
- 大量ドキュメントのセマンティック検索に優位
- FSE 2025論文（Tencent/WXG）: EM 53.76%（BM25）でFine-tuning単独（44.20%）を上回る
- コードベースサイズに対するスケーラビリティが高い

**弱み**:
- 「最も類似」≠「最も関連」。`getUserById`で`findUserByEmail`が返る問題
- FQCNの正確性: `nablarch.fw.web.HttpRequest` と `nablarch.common.web.HttpRequest` はベクトル空間上で近接するが、完全に異なるクラス
- インフラ（ベクトルDB、Embeddingモデル）の維持コスト
- チャンキング戦略によって結果が大幅に変動
- インデックスの鮮度維持（re-indexing）が必要

**Nablarchとの相性**: **中〜低**。FQCNの正確な区別が困難。ハンドラキューの順序情報がチャンクで失われるリスク。

#### B: Agentic Search

**仕組み**: LLMがgrep/glob/read等のツールを反復的に使用し、必要な情報を能動的に検索

**強み**:
- インフラ不要。既存のファイルシステムツールのみ
- セキュリティ・プライバシーリスクがゼロ（外部送信なし）
- 鮮度問題なし（常にリアルタイムのファイルを参照）
- Boris Cherny氏: コード検索では「by a lot」でRAGを上回った

**弱み**:
- トークン消費: RAGの2.7〜3.9倍（Ferrazzi et al., 2026年1月）
- レイテンシ: 10〜20ターンの反復検索が必要
- **フレームワーク知識には不向き**: Nablarchの知識は`grep`可能な形で手元に存在しない
- 検索対象がローカルファイルシステムに限定

**Nablarchとの相性**: **低**。Nablarchの公式ドキュメントやGitHubソースはローカルにクローンしない限りgrepできない。クローンしてもJavadocやHTMLドキュメントの解析は非効率。

#### C: MCP Server（現行方式）

**仕組み**: 構造化知識（YAML）をMCPプロトコルのTools/Resources/Promptsとして提供。AIクライアントが標準プロトコルで知識にアクセス。

**強み**:
- **構造化データの正確な提供**: FQCN、ハンドラ順序、API仕様をYAMLで厳密に定義
- **MCPプロトコル標準**: OpenAI、Anthropic、Microsoft等が採用。2026年のデファクト標準
- **知識の粒度制御**: Tool（アクション）、Resource（参照情報）、Prompt（テンプレート）の3層で提供
- **セキュリティ**: ローカル実行（STDIO）で外部送信なし

**弱み**:
- **開発・運用コスト**: サーバー実装+YAML知識メンテナンスが必要
- **知識の網羅性**: 手動でYAMLを更新する必要がある（自動化されていない）
- **スケーラビリティ**: 知識量が増えるとYAMLファイルの管理が複雑化
- 品質評価評価: 情報精度3.0/5.0（FQCN誤り5件、version-info誤り8件）

**Nablarchとの相性**: **高**。FQCN・ハンドラキュー構成・API仕様は構造化データそのもの。MCP Resourceで正確に提供可能。ただし知識の正確性は手動メンテナンスに依存。

#### D: Rules File / System Prompt（CLAUDE.md等）

**仕組み**: プロジェクトのルールファイル（CLAUDE.md、.cursorrules、AGENTS.md等）にフレームワーク知識を直接記述。セッション開始時にコンテキストに自動ロード。

**強み**:
- **即時利用**: ファイル記述のみで追加インフラ不要
- **高い正確性**: 記述内容がそのままコンテキストに入るため、変換ロスなし
- **レイテンシゼロ**: 検索不要。セッション開始時に既にロード済み
- **メンテナンス容易**: テキストファイルの編集のみ

**弱み**:
- **トークン制限**: コンテキストウィンドウの容量制限。Nablarchの全知識は入りきらない
- **Faros AI報告**: AGENTS.mdの効果は「modest」。エージェントの変動のほうが大きい要因
- **知識の優先順位制御が困難**: 全情報が等価にロードされる
- **バージョン管理の複雑さ**: 複数プロジェクトで異なるルールファイルが必要

**Nablarchとの相性**: **中〜高**。最重要知識（FQCNマッピング、ハンドラキューの基本パターン）はCLAUDE.mdに記述可能。ただし全知識の収容は不可能。**核となる知識のクイックアクセス層**として有効。

#### E: Fine-tuning

**仕組み**: LLMモデル自体にNablarch知識を学習させる。推論時に追加コンテキストなしで回答可能。

**強み**:
- 推論時のオーバーヘッドゼロ
- FSE 2025: Fine-tuning+RAG（BM25）でEM 57.43%を達成（最高精度）
- 内部化された知識によるスタイル・パターンの自然な適用

**弱み**:
- **壊滅的忘却**: 汎用タスク性能が24.1%低下（FSE 2025）
- **学習コスト**: GPU、データ準備、学習パイプライン構築が必要
- **鮮度問題**: Nablarchのバージョン更新ごとに再学習が必要
- **実現可能性**: Claude/GPT等の商用モデルのFine-tuningは制限あり
- **Nablarch知識の学習データ不足**: 公開されたNablarchコードの絶対量が少ない

**Nablarchとの相性**: **低**。学習データ不足、コスト高、壊滅的忘却リスクが致命的。商用モデルのFine-tuningアクセスも限定的。現実的ではない。

#### F: Agentic RAG

**仕組み**: AIエージェントがRAGパイプラインを動的にオーケストレーション。複数データソースへの多段階検索、クエリ書き換え、結果の検証・再検索を自律的に実行。

**強み**:
- 複雑なクエリに対する適応的検索
- 複数知識ソースの統合（YAML + ドキュメント + ソースコード）
- 自己検証・修正能力

**弱み**:
- トークンコスト: 通常RAGの2.7〜3.9倍
- レイテンシ: 多段階推論で応答遅延
- 実装複雑度: エージェントオーケストレーション層の構築が必要

**Nablarchとの相性**: **中〜高**。知識量が増えた場合の拡張手段として有望。MCP Toolとして実装すれば、MCP Serverの枠組み内で統合可能。

### 3.4 比較サマリ図

```
正確性        ████████████████████░ MCP Server (5)
              ████████████████░░░░ Rules File (4)
              ████████████████░░░░ Agentic RAG (4)
              ████████████░░░░░░░░ Embedding RAG (3)
              ████████████░░░░░░░░ Fine-tuning (3)
              ████████░░░░░░░░░░░░ Agentic Search (2)*
              * コード検索なら4、フレームワーク知識なら2

コスト効率     ████████████████████░ Rules File (5)
              ████████████████░░░░ MCP Server (4)
              ████████████░░░░░░░░ Embedding RAG (3)
              ████████░░░░░░░░░░░░ Agentic Search (2)
              ████████░░░░░░░░░░░░ Agentic RAG (2)
              ████░░░░░░░░░░░░░░░░ Fine-tuning (1)

Nablarch適性   ████████████████████░ MCP Server (5)
              ████████████████░░░░ Rules File (4)
              ████████████████░░░░ Agentic RAG (4)
              ████████████░░░░░░░░ Embedding RAG (3)
              ████████░░░░░░░░░░░░ Agentic Search (2)
              ████░░░░░░░░░░░░░░░░ Fine-tuning (1)
```

---

## 4. Nablarch知識の特性分析

### 4.1 知識の種類と各手段の相性

| 知識タイプ | 特性 | 最適手段 | 理由 |
|-----------|------|---------|------|
| **FQCN（完全修飾クラス名）** | 1文字の差異が致命的。`fw.web` vs `common.web` | MCP Resource | 構造化データとして正確に格納・提供可能 |
| **ハンドラキュー構成** | 順序が意味を持つ。組み合わせパターンが多い | MCP Tool + Prompt | 動的に構成を設計するToolが最適 |
| **API仕様** | パラメータ名・型・戻り値が厳密 | MCP Resource | 構造化仕様データの正確な提供 |
| **設計パターン** | 文脈依存。ユースケースで適用が変わる | MCP Prompt + Rules File | テンプレート＋コンテキスト注入が最適 |
| **バージョン差異** | バージョンごとのAPIの変更・非推奨 | MCP Resource | version-info.yamlで構造的に管理 |
| **トラブルシューティング** | エラーメッセージ→原因→対処の3段階 | MCP Tool | 検索＋推論が必要。Toolで実装済み |
| **ベストプラクティス** | 開発経験に基づく暗黙知 | Rules File | CLAUDE.mdに直接記述が最も効果的 |
| **コード生成テンプレート** | 定型的なコードパターン | MCP Prompt | パラメータ化されたテンプレートが最適 |

### 4.2 Nablarch知識の固有の課題

1. **パッケージ構造の非直感性**: `nablarch.fw.web.*` と `nablarch.common.web.*` の区別。Embedding RAGでは近接ベクトルになり混同される
2. **ハンドラキューの順序依存性**: ハンドラの実行順序が動作を決定。単純な類似検索では順序情報が失われる
3. **公式ドキュメントの日本語中心**: Embeddingモデルの日本語性能が品質を左右
4. **知識量**: handler-catalog, api-patterns, design-patterns, error-catalog等、YAML 10ファイル。CLAUDE.mdに全収容は不可能だが、MCPで提供可能な範囲
5. **GitHub Stars 42**: コミュニティ規模が小さく、公開知識が限定的。Fine-tuningの学習データが不足

### 4.3 Boris氏のアプローチがNablarchに合わない理由

| Boris氏の前提 | Nablarchの現実 | 差異 |
|--------------|---------------|------|
| コードはローカルに存在 | Nablarch知識は公式ドキュメント+GitHubに散在 | grepで到達不可 |
| grepで正確にマッチ可能 | FQCN・ハンドラ構成は文書内に埋もれている | 文字列マッチングでは不足 |
| セキュリティが最重要 | 公開フレームワーク情報（機密性なし） | セキュリティ懸念は不要 |
| 「vibes」で判断可能 | FQCN 1文字の誤りが致命的 | 正確性への要求水準が異なる |

---

## 5. 現行方式（nablarch-mcp-server）の評価

### 5.1 品質評価評価結果の要約

6視点から評価された現行MCP Serverの総合スコア: **3.31/5.0**

| 視点 | スコア | 主要課題 |
|------|:------:|---------|
| 情報精度 | 3.0 | FQCN誤り5件、version-info誤り8件 |
| コード品質 | 3.4 | テスト劣化（805→34〜121件成功） |
| MCP仕様準拠 | 3.0 | Tool/Resource登録漏れ8件、isError未対応 |
| UX/DX | 3.6 | セットアップ容易。ドキュメント充実 |
| パフォーマンス | 3.0 | 監視・CI/CD未整備 |
| 市場・競合 | 4.0 | Java/Spring製で競合稀少。タイミング良好 |

### 5.2 現行方式の強み

1. **構造化知識の正確な提供**: YAML知識ファイル → ResourceProvider → MCPクライアントの流れで、変換ロスなく知識を提供
2. **MCPプロトコル標準準拠**: 2026年のデファクト標準。OpenAI/Anthropic/Microsoft採用
3. **3層知識提供**: Tools（アクション）、Resources（参照）、Prompts（テンプレート）の使い分け
4. **24機能実装済み**: 10 Tools + 8 Resources + 6 Prompts（フレームワーク特化MCPサーバーとして最大級）
5. **YAML知識 → 応答の直接性**: YAMLを修正すれば応答が即座に正確になる構造

### 5.3 現行方式の弱み

1. **知識の正確性が手動依存**: YAML更新は人手。自動検証機構がない
2. **知識量の上限**: 1回のMCP応答に含められる知識量には実質的な上限がある
3. **検索能力の限界**: 現行のBM25検索+セマンティック検索は「最も類似」を返すが「最も適切」とは限らない
4. **テスト劣化**: 品質保証の機能停止状態
5. **登録漏れ**: 実装済み機能の38%がMCPクライアントに未公開

### 5.4 知識提供アーキテクチャとしての位置づけ

現行MCP Serverは**Boris氏が放棄したEmbedding RAGとも、推奨したAgentic Searchとも異なる第三の道**:

- **Embedding RAGとの違い**: ベクトル化による近似検索ではなく、構造化データの正確な配信
- **Agentic Searchとの違い**: grep/globによるファイル探索ではなく、事前整理された知識の提供
- **本質**: フレームワーク知識の**キュレーション+構造化配信**プラットフォーム

---

## 6. 推奨方針: 三層ハイブリッドアーキテクチャ

### 6.1 アーキテクチャ概要

```
┌─────────────────────────────────────────────────────────┐
│                   AIコーディングアシスタント                │
│              (Claude Code / Cursor / Copilot)            │
└────────────────────────┬────────────────────────────────┘
                         │ コンテキスト構築
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────────┐
│  Layer 1    │  │  Layer 2     │  │  Layer 3         │
│  即時層     │  │  構造化層    │  │  拡張層          │
│             │  │              │  │                  │
│ CLAUDE.md   │  │ MCP Server   │  │ Agentic RAG      │
│ Rules File  │  │ (現行方式)   │  │ (将来拡張)       │
│             │  │              │  │                  │
│ 核心知識    │  │ Tools        │  │ 公式ドキュメント │
│ - 主要FQCN  │  │ Resources    │  │ GitHubソース     │
│ - 基本パターン│  │ Prompts      │  │ コミュニティ情報 │
│ - 禁止事項  │  │              │  │                  │
│             │  │ YAML知識     │  │ 動的検索+推論    │
│ トークン:低 │  │ トークン:中  │  │ トークン:高      │
│ レイテンシ:0│  │ レイテンシ:低│  │ レイテンシ:中〜高│
│ 正確性:最高 │  │ 正確性:高    │  │ 正確性:中        │
└─────────────┘  └──────────────┘  └──────────────────┘
```

### 6.2 各層の役割

#### Layer 1: 即時層（Context Engineering）

**実装**: CLAUDE.md / .cursorrules にNablarch核心知識を記述

**配置する知識**:
- 最重要FQCNマッピング（頻出する10〜20件）
- ハンドラキューの基本3パターン（Web/REST/バッチ）
- Nablarch固有の禁止事項（例: `nablarch.fw.web.HttpRequest` を `nablarch.common.web.HttpRequest` と混同しないこと）
- コード生成時の基本ルール

**効果**: セッション開始時にコンテキストに自動ロード。トークンコスト最小。レイテンシゼロ。

**工数**: 小（テキストファイル作成のみ）

#### Layer 2: 構造化層（MCP Server — 現行方式の改善）

**実装**: 現行nablarch-mcp-serverのP0/P1改善を実施

**必要な改善**:
1. FQCN誤り5件の修正（B-006）
2. version-info.yaml全面更新（B-007）
3. Tool/Resource登録漏れの修正（B-002, B-003）
4. FQCN自動検証テストの導入（P1-7）
5. DesignHandlerQueueTool二重管理の解消（P1-3）

**効果**: 構造化知識の正確な提供。MCPプロトコル標準による相互運用性。

**工数**: 中（P0+P1で既に計画済み）

#### Layer 3: 拡張層（Agentic RAG — 将来拡張）

**実装**: MCP Toolとして実装。公式ドキュメント・GitHubソースに対するAgentic検索

**ユースケース**:
- YAML知識にない質問（カバレッジ外の知識）
- バージョン間の差異調査
- 複数ドキュメントにまたがる複合質問
- 最新のNablarch情報の動的取得

**効果**: 知識量の制限を超えた回答能力。

**工数**: 大（Phase 5以降で段階的に実装）

### 6.3 各層の連携フロー

```
ユーザー質問: 「NablarchのWeb処理方式でハンドラキューを設計したい」

Step 1: Layer 1（即時層）をチェック
  → CLAUDE.mdにWeb処理の基本パターンあり → 基本情報を即座に活用

Step 2: Layer 2（MCP Server）に問い合わせ
  → DesignHandlerQueueTool で詳細設計
  → HandlerResourceProvider でハンドラカタログ参照
  → SetupHandlerQueuePrompt で構成テンプレート取得

Step 3: Layer 3（必要な場合のみ）
  → 特殊なハンドラの仕様詳細が不足 → 公式ドキュメントをAgentic検索
```

### 6.4 実装ロードマップ

```
Phase A（即座）: Layer 1 構築 ─────────────────
  │  CLAUDE.md にNablarch核心知識セクションを追加
  │  工数: 小（1日）
  │
Phase B（P0/P1と並行）: Layer 2 改善 ──────────
  │  FQCN修正、登録漏れ修正、自動検証テスト
  │  工数: 中（既存計画に含まれる）
  │
Phase C（Phase 5以降）: Layer 3 実装 ──────────
  │  Agentic RAG ToolをMCP Serverに追加
  │  公式ドキュメントクローラー構築
  │  工数: 大
  │
Phase D（将来検討）: 最適化 ────────────────────
     Layer間の自動ルーティング
     知識の鮮度自動更新
     トークン使用量の最適化
```

---

## 7. 主要発見事項

### 7.1 5つの主要発見

1. **Boris氏の判断は「コード検索」特化**: Agentic Search（grep/glob）が有効なのはコードベース検索であり、フレームワーク知識提供には直接適用できない。Nablarch知識はファイルシステム上にgrep可能な形で存在しない。

2. **MCP Serverは第三の道**: Boris氏が放棄したEmbedding RAGでも、推奨したAgentic Searchでもない、**構造化知識のキュレーション配信**という独自のアプローチ。フレームワーク知識提供には最も適している。

3. **Rules File（CLAUDE.md）の即効性**: 最重要知識のクイックアクセス層として、追加インフラなしで即座に効果を発揮。ただし容量制限があるため核心知識に限定すべき。

4. **2026年のコンセンサスはハイブリッド**: 単一手段ではなく、複数手段の組み合わせが最適。MCP + Context Engineering + （将来）Agentic RAGの三層構造が現実的。

5. **現行方式の最大課題は正確性**: 知識提供手段としてのアーキテクチャは妥当だが、知識データ自体の正確性（FQCN誤り5件、version-info誤り8件）がボトルネック。手段を変えるより**知識データの品質向上が最優先**。

### 7.2 追加の知見

- **トークンコスト**: Agentic SearchはRAGの2.7〜3.9倍のトークンを消費する（Ferrazzi et al., 2026）
- **Fine-tuning**: Nablarchは学習データが圧倒的に不足（GitHub Stars 42）。壊滅的忘却のリスクも高い。現実的選択肢ではない
- **Embedding RAGの限界**: FQCN（`nablarch.fw.web.HttpRequest` vs `nablarch.common.web.HttpRequest`）のようにベクトル空間上で近接するが意味が異なるデータには不向き
- **FSE 2025の示唆**: RAG + Fine-tuningの組み合わせがEM 57.43%で最高精度だが、Nablarchでは学習データ不足でこのパスは困難

---

## 8. 参考文献

### 原典・一次情報

- [Latent Space Podcast: "Claude Code: Anthropic's Agent in Your Terminal"](https://www.latent.space/p/claude-code) — Boris Cherny氏のAgentic Search発言の原典
- [X(Twitter) @aakashgupta: Boris Cherny引用スレッド](https://x.com/aakashgupta/status/2018933856460775597)
- [Threads @jatinw21: Boris Cherny発言の解説](https://www.threads.com/@jatinw21/post/DUPCuyBCbwM/)

### 技術比較・分析

- [SmartScope Blog: Settling the RAG Debate](https://smartscope.blog/en/ai-development/practices/rag-debate-agentic-search-code-exploration/) — RAG vs Agentic Searchの包括的分析
- [Relace.ai: Exploiting parallel tool calls for 4x faster agentic search](https://www.relace.ai/blog/fast-agentic-search)
- [Alberto Roura: Vector RAG? Agentic Search? Why Not Both?](https://albertoroura.com/vector-rag-agentic-search-why-not-both/) — ハイブリッドアプローチの提案
- [Mark Hendrickson: Agentic retrieval infers. It doesn't guarantee.](https://markmhendrickson.com/posts/agentic-search-and-the-truth-layer/)
- [Tiger Data: Why Cursor is About to Ditch Vector Search](https://www.tigerdata.com/blog/why-cursor-is-about-to-ditch-vector-search-and-you-should-too)

### 学術論文・調査レポート

- [FSE 2025: RAG or Fine-tuning? A Comparative Study on LCMs-based Code Completion in Industry](https://arxiv.org/abs/2505.15179) — Tencent/WXG, EM比較データ
- [NVIDIA Blog: Traditional RAG vs Agentic RAG](https://developer.nvidia.com/blog/traditional-rag-vs-agentic-rag-why-ai-agents-need-dynamic-knowledge-to-get-smarter/)
- [LlamaIndex: Agentic Retrieval Guide](https://www.llamaindex.ai/blog/rag-is-dead-long-live-agentic-retrieval)
- [DigitalOcean: RAG, AI Agents, and Agentic RAG](https://www.digitalocean.com/community/conceptual-articles/rag-ai-agents-agentic-rag-comparative-analysis)

### MCP・Context Engineering

- [MCP公式仕様](https://modelcontextprotocol.io/specification/2025-11-25)
- [Kanerika: MCP vs RAG in 2026](https://kanerika.com/blogs/mcp-vs-rag/)
- [Omar Santos: Integrating Agentic RAG with MCP Servers](https://becomingahacker.org/integrating-agentic-rag-with-mcp-servers-technical-implementation-guide-1aba8fd4e442)
- [Faros AI: Context Engineering for Developers](https://www.faros.ai/blog/context-engineering-for-developers) — CLAUDE.md/AGENTS.mdの効果分析
- [Kubiya: Context Engineering Best Practices 2025](https://www.kubiya.ai/blog/context-engineering-best-practices)
- [Qodo: Context Engineering: The New Backbone of Scalable AI Systems](https://www.qodo.ai/blog/context-engineering/)

### 内部評価レポート

- `final-report.md` — nablarch-mcp-server多角的評価統合レポート

---

