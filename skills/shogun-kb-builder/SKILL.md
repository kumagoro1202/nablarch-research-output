---
name: kb-builder
description: 特定の技術・フレームワーク・ツールに関するナレッジベースを体系的に構築する。複数の情報源（GitHub organization、公式ドキュメント、開発標準、技術ブログ等）を並列で調査し、領域別のMarkdownドキュメント群として統合する。「○○のナレッジベースを構築して」「○○を網羅的に調査してKBにまとめて」「○○の技術情報を体系化して」「○○のGitHub/ドキュメント/標準を並列調査して」といった要望に対応する。gh CLI + WebFetch + WebSearch を駆使した並列調査・統合文書化スキル。
---

# KB Builder — ナレッジベース構築パターン

## Overview

特定の技術・フレームワーク・ツールについて、複数の情報源を**並列に調査**し、領域別のMarkdownドキュメント群として体系的なナレッジベース（KB）を構築するスキル。

調査対象を独立した領域に分割し、各領域を別ファイルに出力することで、複数エージェントによる並列実行とファイル競合の回避（RACE-001対策）を同時に実現する。

**主な用途:**
- 新技術の採用検討に必要な情報の網羅的収集
- 既存技術の体系的ドキュメント化
- RAGシステム用のナレッジベース素材作成
- 技術デューデリジェンスの情報基盤構築

**このスキルの特徴:**
- 領域分割 → 並列調査 → 統合文書化の3段階アプローチ
- 各領域が独立ファイルに出力されるため、並列実行時のファイル競合なし
- gh CLI / WebFetch / WebSearch の3ツールを使い分ける調査パターン集
- 実績ベースのベストプラクティス（Nablarch KB構築で6並列調査を実証済み）

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「○○のナレッジベースを構築して」
- 「○○を網羅的に調査してまとめて」
- 「○○のGitHub/ドキュメント/開発標準/コミュニティを並列調査して」
- 「○○の技術情報を体系的にドキュメント化して」
- 「RAG用に○○のKBを作成して」
- 「○○について複数領域を分担して調査して」
- 複数の情報源（GitHub、公式サイト、ブログ等）を横断して1つの技術を調査する場合
- 複数エージェントで分担して並列に調査したい場合

**トリガーキーワード**: ナレッジベース構築, KB構築, 網羅的調査, 体系的調査, 並列調査, 技術調査, ドキュメント化, 情報収集

## Instructions

### Phase 1: 調査設計

対象技術の全体像を把握し、並列実行可能な調査領域に分割する。

#### Step 1.1: 情報源の特定

```
【実行手順】

1. 対象技術の基本情報を確認
   - 公式サイトURL
   - GitHub organization / repository
   - 公式ドキュメントサイト
   - 開発元組織

2. 利用可能な情報源を列挙
   - GitHub（リポジトリ、README、Issues、Releases）
   - 公式ドキュメント（ガイド、APIリファレンス、チュートリアル）
   - 開発標準（コーディング規約、設計テンプレート、テスト標準）
   - ナレッジサイト（Fintan、MDN、AWS Docs等）
   - コミュニティ（Qiita、Zenn、Stack Overflow、ブログ）
   - アーキテクチャ（設計思想、モジュール構成、依存関係）

3. 情報源の優先度を決定
   - 高: 一次情報（GitHub、公式ドキュメント）
   - 中: 二次情報（開発標準、ナレッジサイト）
   - 低: 三次情報（コミュニティ、ブログ）
```

#### Step 1.2: 調査領域の分割

```
【分割原則】

1. 各領域は独立して調査可能であること
   - 領域Aの結果が領域Bの調査に必要、という依存関係がないこと
   - 例外: 統合フェーズでの相互参照は可

2. 各領域は別ファイルに出力すること（RACE-001回避）
   - 同一ファイルに複数エージェントが書き込む → 競合発生
   - 領域ごとに1ファイル → 競合なし

3. 領域数は利用可能なエージェント数に合わせる
   - 2-3エージェント → 3-4領域
   - 4-6エージェント → 5-7領域
   - 7-8エージェント → 6-8領域（8以上に分けても管理コストが増大）

4. 各領域の作業量が概ね均等になるよう調整
   - GitHub調査（リポジトリ数が多い場合）は重い
   - 公式ドキュメント調査（ページ数が多い場合）は重い
   - コミュニティ調査（記事が少ない技術）は軽い

【標準的な6領域分割（推奨）】
| 領域ID | 領域名 | 主要ツール | 出力ファイル |
|--------|--------|-----------|-------------|
| D1 | 公式ドキュメント | WebFetch, WebSearch | {tech}_kb_official_docs.md |
| D2 | GitHub | gh CLI, gh api | {tech}_kb_github.md |
| D3 | 開発標準・ガイド | gh CLI, WebFetch | {tech}_kb_dev_standards.md |
| D4 | ナレッジサイト | WebSearch, WebFetch | {tech}_kb_{site_name}.md |
| D5 | コミュニティ | WebSearch, WebFetch | {tech}_kb_community.md |
| D6 | アーキテクチャ | gh CLI, WebFetch | {tech}_kb_architecture.md |
```

#### Step 1.3: 出力ファイルパスの確定

```
【命名規則】
{output_dir}/{tech_lowercase}_kb_{domain}.md

例（Nablarchの場合）:
  output/nablarch_kb_official_docs.md
  output/nablarch_kb_github.md
  output/nablarch_kb_dev_standards.md
  output/nablarch_kb_fintan.md
  output/nablarch_kb_community.md
  output/nablarch_kb_architecture.md

例（Reactの場合）:
  output/react_kb_official_docs.md
  output/react_kb_github.md
  output/react_kb_ecosystem.md
  output/react_kb_community.md

【重要】
- ファイルパスは調査開始前に全て確定させる
- 各エージェントに「自分の出力ファイル」を明示して割り当てる
- 統合目次ファイルは全領域完了後に別途作成
```

### Phase 2: GitHub調査パターン

GitHub organization / repository の情報を網羅的に収集する。

#### Step 2.1: リポジトリ一覧の取得

```
【gh CLI コマンド】

# Organization の場合（全リポジトリ取得）
gh repo list {org} --limit 200 --json name,description,stargazerCount,updatedAt,language,isArchived

# 注意: フィールド名は stargazerCount（stargazersCount ではない）
# --limit 200 で十分な場合が多いが、大規模orgは --limit 500 に

# 単一リポジトリの場合
gh repo view {owner}/{repo} --json name,description,stargazerCount,forksCount,updatedAt

【結果の分類】
取得した全リポジトリを役割別にカテゴリ分けする:
- コアライブラリ（基盤機能）
- フレームワーク（実行制御、アプリケーション種別対応）
- 共通コンポーネント（業務機能）
- アダプタ（外部連携）
- テスト（テスティングFW、テストユーティリティ）
- ビルド・BOM管理（親POM、プロファイル）
- サンプル・Example
- ツール・ユーティリティ
- ドキュメント

【実践知見】
- アーカイブ済みリポジトリ（isArchived: true）も記録する（歴史的経緯の理解に有用）
- 説明（description）が空のリポジトリはREADMEで補完する
- 言語（language）の分布はプロジェクトの技術特性を示す指標
```

#### Step 2.2: 主要リポジトリのREADME取得

```
【gh CLI コマンド】

# READMEの取得（gh repo view）
gh repo view {owner}/{repo}

# READMEが長い場合や gh repo view で取得できない場合
gh api repos/{owner}/{repo}/readme --jq '.content' | base64 -d

【取得対象の選定基準】
- Stars数が多いリポジトリ（上位10件程度）
- フレームワークのコアリポジトリ
- ドキュメントリポジトリ
- ツールリポジトリ（特にAI関連）
- BOM/親POMリポジトリ（バージョン管理の理解）

【READMEから抽出する情報】
- プロジェクトの目的・概要
- 主要クラス・インターフェースの一覧
- モジュール構成
- ビルド方法
- 依存関係
- ライセンス
```

#### Step 2.3: メトリクス情報の取得

```
【gh API コマンド — 並列実行可能】

# リポジトリ基本統計
gh api repos/{owner}/{repo} --jq '{
  stars: .stargazers_count,
  forks: .forks_count,
  watchers: .watchers_count,
  open_issues: .open_issues_count,
  created: .created_at,
  updated: .pushed_at
}'

# コントリビューター数（主要リポジトリのみ）
gh api repos/{owner}/{repo}/contributors --jq 'length'

# リリースタグ
gh api repos/{owner}/{repo}/tags --jq '.[].name'

# Issue/PR 統計（深度 thorough のみ）
gh api repos/{owner}/{repo}/issues?state=all --jq 'length'

【レート制限への対策】
- gh api は1時間5000リクエストの制限あり
- 主要リポジトリ（10-15件）に絞って詳細取得
- 残りはリスト取得時の基本情報で代替
- レート制限に達した場合: 取得済みデータで中間レポートを作成
```

#### Step 2.4: リポジトリ間の依存関係の把握

```
【依存関係の調査方法】

1. 親POM / BOM の確認
   - pom.xml の <parent> 要素
   - <dependencyManagement> セクション

2. README / ドキュメントからの推測
   - 「requires {module}」等の記述
   - モジュール一覧表の依存列

3. 依存関係の図式化（テキスト表現）
   {parent} → {core} → {fw} → {fw-web/batch/jaxrs}

   例（Nablarch）:
   nablarch-parent → nablarch-core → nablarch-fw → nablarch-fw-web
                                                  → nablarch-fw-batch
                                                  → nablarch-fw-jaxrs
```

### Phase 3: 公式ドキュメント調査パターン

公式ドキュメントサイトの構造を把握し、主要コンテンツを収集する。

#### Step 3.1: サイト構造の把握

```
【実行手順】

1. トップページをWebFetchで取得
   WebFetch(url: "{公式ドキュメントURL}", prompt: "サイト全体の構造を把握せよ。メインメニュー、カテゴリ一覧、ページ数を抽出。")

2. サイトマップの確認
   WebFetch(url: "{公式ドキュメントURL}/sitemap.xml", prompt: "全ページのURLを一覧化せよ。")
   ※ sitemap.xml がない場合はナビゲーションから推測

3. WebSearchで補完
   WebSearch: "site:{ドキュメントドメイン} {技術名}"
   WebSearch: "site:{ドキュメントドメイン} guide tutorial reference"
```

#### Step 3.2: セクション別コンテンツ取得

```
【取得戦略】

1. ガイド系ドキュメント（概念説明、アーキテクチャ）
   - 優先度: 高
   - WebFetch prompt: "このガイドの全内容を抽出せよ。概念、用語定義、アーキテクチャ図の説明を含む。"

2. APIリファレンス（クラス、メソッド一覧）
   - 優先度: 中（量が多いため取捨選択）
   - WebFetch prompt: "主要クラス・インターフェースの名前、役割、主要メソッドを抽出。"

3. チュートリアル / Getting Started
   - 優先度: 高
   - WebFetch prompt: "手順、必要な前提条件、サンプルコードを全て抽出。"

4. FAQ / トラブルシューティング
   - 優先度: 中
   - WebFetch prompt: "全FAQ項目のQ&Aを抽出。"

5. リリースノート / 変更履歴
   - 優先度: 低（最新2-3バージョンのみ）
   - WebFetch prompt: "最新バージョンの変更内容を抽出。破壊的変更に注目。"

【並列実行】
- 独立したページは4件まで並列でWebFetchを実行可能
- 依存関係のないページは常に並列で取得せよ
```

#### Step 3.3: 多言語対応

```
【日本語技術の場合】
- 日本語ドキュメントを優先取得
- 英語版が存在する場合は差分を確認
- 英語版の方が情報が多い場合がある（Nablarchでは日英でほぼ同等）

【グローバル技術の場合】
- 英語ドキュメントを優先取得
- 日本語訳が存在する場合は翻訳品質を確認
- 未翻訳セクションの有無を記録
```

### Phase 4: 開発標準・ガイド調査パターン

コーディング規約、設計テンプレート、テスト標準などの開発標準を調査する。

#### Step 4.1: 開発標準リポジトリの特定

```
【検索パターン — 並列実行可能】

1. GitHub orgから直接検索
   gh repo list {org} --limit 200 --json name,description | (--jq で development-standards, style-guide, coding 等を含むものを抽出)

2. 別Organization の可能性
   WebSearch: "{技術名} development standards GitHub"
   WebSearch: "{技術名} coding standards style guide"

   ※ Nablarchでは nablarch-development-standards が nablarch orgとは別orgで存在

3. Fintanや公式サイトでの公開
   WebSearch: "site:fintan.jp {技術名} 開発標準"
   WebSearch: "{技術名} coding conventions best practices"
```

#### Step 4.2: コーディング規約の調査

```
【調査項目】
- 命名規則（クラス、メソッド、変数、パッケージ）
- フォーマットルール（インデント、改行、最大行長）
- 禁止事項リスト（使ってはいけないAPI、パターン）
- 推奨事項リスト（推奨されるパターン、イディオム）
- 静的解析ツール設定
  - Checkstyle ルール数・設定ファイル
  - SpotBugs / FindBugs 設定
  - ArchUnit ルール
  - その他カスタムチェッカー

【ドキュメント形式の確認】
- Markdown / HTML → WebFetch で取得可能
- Excel (.xlsx) / Word (.docx) → URLのみ記録（バイナリは詳細取得不可）
- PDF → URLのみ記録
```

#### Step 4.3: 設計テンプレートの調査

```
【調査項目】
- テンプレート一覧（種類と用途）
- テンプレートの形式（Excel、Word、Markdown等）
- 設計書カテゴリ（機能設計、DB設計、画面設計、IF設計等）
- サンプル（記入例）の有無
- バージョン履歴

【実践知見 — Nablarch開発標準の場合】
- 030_設計ドキュメント/ に11カテゴリの設計書テンプレートとサンプルを配置
- Excel/Word形式が中心（日本のSIer文化に合わせた選択）
- 擬似SQL（日本語表現のSQL構造）という独自概念あり
```

#### Step 4.4: テスト標準の調査

```
【調査項目】
- テスティングFWの構成（クラス階層、主要クラス）
- テストデータの管理方式（Excel、CSV、YAML等）
- テスト種別（単体、結合、E2E）ごとの標準
- テスト記述パターン（アサーション、セットアップ）
- CI/CDとの統合方法

【実践知見 — Nablarch Testingの場合】
- ExcelベースのテストデータがNablarch最大の特徴
- DataType列挙12種、特殊セル値、読込パイプラインの詳細な仕組み
- JUnit5拡張が11アノテーション対応
```

### Phase 5: 統合・文書化

各領域の調査結果を統一フォーマットでMarkdownドキュメントに出力する。

#### Step 5.1: 各領域ファイルの構成

```
【統一テンプレート — 各領域共通】

# {技術名} {領域名} ナレッジベース

> **調査日**: YYYY-MM-DD
> **調査者**: {エージェント名} ({タスクID})
> **対象**: {調査対象の説明}

---

## 目次

1. [概要](#1-概要)
2. [{セクション名}](#2-セクション名)
...
N. [ソースURL一覧](#n-ソースurl一覧)

---

## 1. 概要
{領域の概要、調査範囲、主要な発見}

## 2-N. 詳細セクション
{調査結果をテーブル、リスト、コードブロックで構造化}

## N. ソースURL一覧
{全ての参照元URL}

---

*本ドキュメントは{エージェント名}が調査・作成した。*
```

#### Step 5.2: テーブル形式の標準化

```
【リポジトリ一覧テーブル】
| リポジトリ | 説明 | Stars | 最終更新 | 言語 |
|-----------|------|-------|---------|------|

【コンテンツ一覧テーブル】
| # | タイトル | URL | カテゴリ | 概要 |
|---|---------|-----|---------|------|

【メトリクス比較テーブル】
| 指標 | 値 | 備考 |
|------|-----|------|

【全ての情報にソースURLを付記する】
- テーブル内: URLカラムにリンクを記載
- 本文内: **ソース**: URL の形式
- 引用: > 引用文 — [出典](URL)
```

#### Step 5.3: 統合目次ファイルの作成（オプション）

```
【全領域完了後に作成】

# {技術名} ナレッジベース — 統合目次

> **構築日**: YYYY-MM-DD
> **領域数**: {N}
> **総ファイルサイズ**: 約{X}KB

## ナレッジベース構成

| # | 領域 | ファイル | サイズ | 主要トピック |
|---|------|---------|--------|------------|
| 1 | 公式ドキュメント | {tech}_kb_official_docs.md | XX KB | ... |
| 2 | GitHub | {tech}_kb_github.md | XX KB | ... |
| ... | ... | ... | ... | ... |

## 主要な発見
{全領域を通じた重要な発見事項}

## 推奨される活用方法
{KBの活用方法のガイダンス}
```

## Input Format

スキル実行時に以下のパラメータを指定する。

```yaml
# 必須パラメータ
technology_name: "Nablarch"                     # 対象技術名
domains:                                        # 調査領域リスト
  - name: "GitHub"
    type: "github"                              # github / docs / standards / site / community / architecture
    output_file: "output/nablarch_kb_github.md"
    params:
      org: "nablarch"
  - name: "公式ドキュメント"
    type: "docs"
    output_file: "output/nablarch_kb_official_docs.md"
    params:
      url: "https://nablarch.github.io/docs/LATEST/doc/"
  - name: "開発標準"
    type: "standards"
    output_file: "output/nablarch_kb_dev_standards.md"
    params:
      org: "nablarch-development-standards"
      repo: "nablarch-development-standards"
output_dir: "output/"                           # 出力ディレクトリ

# オプションパラメータ
file_naming_pattern: "{tech}_kb_{domain}.md"    # ファイル命名パターン（デフォルト）
parallelism: 6                                  # 並列度（利用可能なエージェント数）
depth: "standard"                               # quick / standard / thorough
exclude_keywords:                               # 除外キーワード
  - "Xenlon"
language: "ja"                                  # 出力言語（デフォルト: ja）
generate_index: true                            # 統合目次ファイル生成（デフォルト: true）
```

### パラメータ説明

| パラメータ | 必須 | デフォルト | 説明 |
|----------|------|----------|------|
| `technology_name` | yes | — | 対象技術の名称 |
| `domains` | yes | — | 調査領域リスト（各領域に name, type, output_file, params を指定） |
| `output_dir` | yes | — | 出力ディレクトリ |
| `file_naming_pattern` | — | `{tech}_kb_{domain}.md` | ファイル命名パターン |
| `parallelism` | — | `4` | 並列度（利用可能なエージェント数） |
| `depth` | — | `standard` | 調査深度 |
| `exclude_keywords` | — | `[]` | 除外キーワード |
| `language` | — | `ja` | 出力言語 |
| `generate_index` | — | `true` | 統合目次ファイルの生成 |

### 領域タイプ（type）

| type | 説明 | 主要ツール |
|------|------|-----------|
| `github` | GitHub org/repo 調査 | gh CLI, gh api |
| `docs` | 公式ドキュメント調査 | WebFetch, WebSearch |
| `standards` | 開発標準・ガイド調査 | gh CLI, WebFetch |
| `site` | ナレッジサイト調査 | WebSearch (site:), WebFetch |
| `community` | コミュニティ調査 | WebSearch, WebFetch |
| `architecture` | アーキテクチャ調査 | gh CLI, WebFetch |

## Output Format

### 領域別ファイルテンプレート

```markdown
# {技術名} {領域名} ナレッジベース

> **調査日**: YYYY-MM-DD
> **調査者**: {エージェント名} ({タスクID} / {親コマンドID})
> **データソース**: {gh CLI + GitHub REST API + Web検索 等}

---

## 目次

1. [セクション1](#1-セクション1)
2. [セクション2](#2-セクション2)
...

---

## 1. セクション1

### 1.1 サブセクション

| カラム1 | カラム2 | カラム3 |
|--------|--------|--------|
| 値1 | 値2 | 値3 |

**ソース**: https://example.com/...

---

## N. ソースURL一覧

- https://github.com/...
- https://example.com/docs/...

---

*本ドキュメントは{エージェント名}が{タスクID}として調査・作成した。*
```

### 統合目次ファイルテンプレート

```markdown
# {技術名} ナレッジベース — 統合目次

> **構築日**: YYYY-MM-DD
> **プロジェクト**: {プロジェクト名}
> **領域数**: {N}

---

## 構成ファイル一覧

| # | 領域 | ファイル | 行数 | 主要トピック |
|---|------|---------|------|------------|
| 1 | {領域名} | [{ファイル名}]({相対パス}) | {行数} | {概要} |

## 主要な発見

- {発見1}
- {発見2}
- ...

## 横断的な関連情報

{複数領域にまたがる重要な情報}
```

## Parallelization Strategy

cmd_007で実際に6足軽が並列実行したナレッジベース構築のパターンをベストプラクティスとして記載する。

### 実績: Nablarch KB構築（6並列調査）

```
【プロジェクト】nablarch_strategy / cmd_007
【対象技術】Nablarch（TIS社のJavaフレームワーク）
【並列度】6エージェント（足軽1〜6号）
【成果物】6つの領域別KBファイル + 統合利用

領域分割:
  足軽1号: 公式ドキュメント → output/nablarch_kb_official_docs.md
  足軽2号: GitHub調査       → output/nablarch_kb_github.md （564行）
  足軽3号: Fintanコンテンツ  → output/nablarch_kb_fintan.md
  足軽4号: コミュニティ      → output/nablarch_kb_community.md
  足軽5号: アーキテクチャ    → output/nablarch_kb_architecture.md
  足軽6号: 開発標準         → output/nablarch_kb_dev_standards.md （1500行）

結果:
  - 全6領域が並列に調査完了（ファイル競合ゼロ）
  - 合計数千行のナレッジベースを構築
  - 各領域は独立して作成され、相互参照の必要なし
```

### 並列化の原則

#### 1. ファイル分割戦略（RACE-001回避）

```
【絶対ルール】
- 1つのファイルに書き込むエージェントは常に1つだけ
- 複数エージェントが同じファイルに書き込む → 後の書き込みが先の内容を破壊

【ファイル割り当て】
タスク割り当て時に出力ファイルパスを明示:
  task:
    worker: ashigaru2
    output_file: "output/nablarch_kb_github.md"  # ← 専用ファイル

【統合は最後に】
- 各領域の調査が全て完了してから統合目次を作成
- 統合は1エージェントが担当（通常は家老またはまとめ担当の足軽）
```

#### 2. 領域間の独立性の確認方法

```
【確認チェックリスト】
各領域ペア(A, B)について以下を確認:
□ 領域Aの調査に領域Bの結果は不要か？
□ 領域Bの調査に領域Aの結果は不要か？
□ 領域Aと領域Bで同じファイルに書き込まないか？
□ 領域Aと領域Bで同じAPIエンドポイントを大量に叩かないか？
  （レート制限の競合回避）

【独立でない場合の対処】
- 依存関係がある場合: 依存元を先に実行し、完了後に依存先を開始
- APIレート制限の競合: 片方をWebSearchベース、もう片方をgh CLIベースに分離
```

#### 3. タスク割り当てのベストプラクティス

```
【割り当て時に伝えるべき情報】
1. 調査対象の領域名と範囲
2. 使用すべき主要ツール（gh CLI / WebFetch / WebSearch）
3. 出力ファイルパス（絶対パス）
4. 出力フォーマット（テンプレートまたは参考ファイル）
5. 除外キーワード（あれば）
6. 調査の深度（quick / standard / thorough）

【割り当て例（YAMLタスク）】
task:
  task_id: subtask_018
  description: |
    【Nablarch情報収集: 領域2 — GitHub情報の網羅的調査】
    nablarch organization のGitHub情報を網羅的に調査し、体系的にまとめよ。

    ## 調査対象
    - nablarch organization の全リポジトリ一覧
    - 主要リポジトリのREADMEと構成
    - メトリクス情報（Stars, Forks, Contributors）
    - リリース履歴

    ## 出力先
    output/nablarch_kb_github.md
```

#### 4. 統合フェーズの進め方

```
【タイミング】
- 全領域の足軽が報告完了してから統合を開始
- 一部領域が未完了の場合、完了分だけで暫定版を作成可能

【統合作業の内容】
1. 各領域ファイルの行数・品質を確認
2. 統合目次ファイル（{tech}_kb_index.md）を作成
3. 領域間の矛盾・重複がないか確認
4. 横断的な発見事項をまとめる

【統合目次の作成者】
- 通常: 家老（タスク管理者）
- または: 最初に完了した足軽に追加タスクとして割り当て
```

## Examples

### Example 1: Nablarch ナレッジベース構築（6並列 — 実績ベース）

```
入力:
  technology_name: "Nablarch"
  domains:
    - name: "公式ドキュメント"
      type: "docs"
      output_file: "output/nablarch_kb_official_docs.md"
      params:
        url: "https://nablarch.github.io/docs/LATEST/doc/"
    - name: "GitHub"
      type: "github"
      output_file: "output/nablarch_kb_github.md"
      params:
        org: "nablarch"
    - name: "開発標準"
      type: "standards"
      output_file: "output/nablarch_kb_dev_standards.md"
      params:
        org: "nablarch-development-standards"
    - name: "Fintan"
      type: "site"
      output_file: "output/nablarch_kb_fintan.md"
      params:
        url: "https://fintan.jp/"
    - name: "コミュニティ"
      type: "community"
      output_file: "output/nablarch_kb_community.md"
    - name: "アーキテクチャ"
      type: "architecture"
      output_file: "output/nablarch_kb_architecture.md"
  output_dir: "output/"
  parallelism: 6
  depth: "standard"
  exclude_keywords: ["Xenlon"]

GitHub調査の詳細（足軽2号が実行）:
  1. gh repo list nablarch --limit 200 → 106リポジトリ取得
  2. 10カテゴリに分類（Core, Framework, Common, Adapter, Testing, Build, Messaging, Sample, Tools, Docs）
  3. 主要12リポジトリのREADMEを個別取得（gh repo view / gh api）
  4. nablarch-tools-for-ai の詳細調査 → PR #1発見（TISのAI支援開発の証左）
  5. nablarch-parent のタグからリリース履歴を取得（6, 6u3, 6u2, 5u26...5u12）
  6. コントリビューター数を主要4リポジトリから取得

開発標準調査の詳細（足軽6号が実行）:
  1. nablarch-development-standards orgを発見（別org）
  2. 5領域を並列調査:
     - 開発プロセス標準、アプリ開発標準、設計ドキュメント（11カテゴリ）
     - コーディング規約（Checkstyle 57ルール、SpotBugs、ArchUnit）
     - テスティングFW（ExcelベースDataType 12種、JUnit5拡張11アノテーション）
     - サンプルプロジェクト（3プロジェクト、10共通パターン、9 Archetype）
  3. 1500行のKBファイルを作成

出力:
  - 6つの領域別KBファイル（合計数千行）
  - 全ファイルにソースURL付記
  - ファイル競合ゼロ（RACE-001回避成功）
```

### Example 2: React ナレッジベース構築（4並列）

```
入力:
  technology_name: "React"
  domains:
    - name: "公式ドキュメント"
      type: "docs"
      output_file: "output/react_kb_official_docs.md"
      params:
        url: "https://react.dev/"
    - name: "GitHub"
      type: "github"
      output_file: "output/react_kb_github.md"
      params:
        owner: "facebook"
        repo: "react"
    - name: "エコシステム"
      type: "architecture"
      output_file: "output/react_kb_ecosystem.md"
    - name: "コミュニティ"
      type: "community"
      output_file: "output/react_kb_community.md"
  output_dir: "output/"
  parallelism: 4
  depth: "standard"

領域分割の考え方:
  - Reactは開発標準が公式ドキュメント内にあるため、独立領域にしない
  - エコシステム（Next.js, Remix, 状態管理等）を独立領域として調査
  - コミュニティは英語圏が圧倒的に多いため英語検索を重視

出力:
  - 4つの領域別KBファイル
  - 統合目次ファイル
```

### Example 3: Kubernetes ナレッジベース構築（3並列 — 小規模チーム）

```
入力:
  technology_name: "Kubernetes"
  domains:
    - name: "公式ドキュメント + GitHub"
      type: "docs"
      output_file: "output/k8s_kb_docs_github.md"
      params:
        url: "https://kubernetes.io/docs/"
        org: "kubernetes"
    - name: "エコシステム・ツール"
      type: "architecture"
      output_file: "output/k8s_kb_ecosystem.md"
    - name: "コミュニティ・運用ノウハウ"
      type: "community"
      output_file: "output/k8s_kb_community_ops.md"
  output_dir: "output/"
  parallelism: 3
  depth: "quick"

領域分割の考え方:
  - 3エージェントしかないため、公式ドキュメントとGitHubを1領域に統合
  - エコシステムが非常に大きい（Helm, Istio, Prometheus等）ため独立領域
  - 運用ノウハウとコミュニティを統合して効率化

出力:
  - 3つの領域別KBファイル
  - 統合目次ファイル
```

## Guidelines

### 必須ルール

1. **RACE-001を必ず回避すること**
   - 1ファイル = 1エージェント の原則を厳守
   - タスク割り当て時に出力ファイルパスを明示
   - 統合作業は全領域完了後に1エージェントが実施

2. **全ての情報にソースURLを付記すること**
   - テーブル内、本文内、引用文の全てにソースを明記
   - URLが不明な情報は「ソース: 不明」と記載
   - GitHub情報はリポジトリURLをリンクとして記載

3. **アーカイブ済みリポジトリも記録すること**
   - `isArchived: true` のリポジトリもリストに含める
   - アーカイブ状態を明記する（歴史的経緯の理解に重要）

4. **レート制限に対処すること**
   - gh api: 1時間5000リクエスト
   - WebSearch: 連続使用でレート制限の可能性
   - WebFetch: サイトによりレート制限あり
   - 制限に達した場合: 取得済みデータで中間レポートを作成し、制限解除後に続行

5. **除外キーワードを遵守すること**
   - 除外対象の技術・製品に関する情報は収集しない
   - 収集済みデータに含まれていた場合は出力時に除外

6. **調査の深度を遵守すること**

| 深度 | GitHub | ドキュメント | 開発標準 | コミュニティ |
|------|--------|------------|---------|------------|
| quick | リポジトリ一覧のみ | トップページ+主要5ページ | README のみ | 主要プラットフォーム1つ |
| standard | 一覧+主要リポジトリ詳細 | 全セクション概要+主要ページ詳細 | 規約+テンプレート一覧 | 全プラットフォーム基本検索 |
| thorough | 全リポジトリ詳細+メトリクス | 全ページフェッチ | 全ドキュメント詳細 | 全プラットフォーム+関連KW展開 |

### ツール使い分けの指針

```
【gh CLI を使う場合】
- リポジトリ一覧の取得（gh repo list）
- リポジトリ詳細の取得（gh repo view）
- API経由のメトリクス取得（gh api）
- リリースタグの取得
- コントリビューター情報の取得

【WebFetch を使う場合】
- 公式ドキュメントページの内容取得
- ナレッジサイト（Fintan等）のページ取得
- ブログ記事の詳細取得
- サイトマップの取得

【WebSearch を使う場合】
- site: 演算子での特定サイト検索
- 技術名 + キーワードでの一般検索
- コミュニティ記事の発見
- 開発標準リポジトリの発見
```

### アンチパターン（避けるべきこと）

- **同一ファイルへの並列書き込み**: RACE-001の原因。絶対に避ける
- **領域間の暗黙的依存**: 「領域Aの結果を見て領域Bの範囲を決める」のような設計
- **レート制限の無視**: エラーを無視して連続リクエスト → アカウント制限のリスク
- **ソースURL未記載**: 情報の信頼性検証ができなくなる
- **バイナリファイルのWebFetch**: Excel/Word/PDFはURLのみ記録し、内容取得は諦める
- **過剰な分割**: 8領域以上に分割すると管理コストが成果を上回る
- **gh CLI のフィールド名ミス**: `stargazersCount` ではなく `stargazerCount`（実績あるエラー）
- **目次なしの長大ファイル**: 500行以上のファイルには必ず目次をつける
