---
name: fintan-content-scraper
description: Fintan（fintan.jp）および類似のナレッジサイト（developer.mozilla.org, docs.aws.amazon.com等）から、特定カテゴリやキーワードに関連するコンテンツを網羅的に収集し、Markdown形式の体系的なレポートとして出力する。「○○サイトの△△関連コンテンツを調査して」「Fintanから□□の情報を集めて」「ナレッジサイトの技術情報をスクレイピングして」「○○の技術資料を網羅的に収集して」といった要望に対応する。WebSearch + WebFetch を駆使した多段階調査スキル。
---

# Fintan Content Scraper

## Overview

Fintan（https://fintan.jp/）を主要ターゲットとし、類似のナレッジサイトからも特定カテゴリ/キーワードのコンテンツを網羅的に収集し、Markdown形式の体系的なレポートとして出力するスキル。

WebSearch による `site:` 指定検索と WebFetch によるページ内容取得を組み合わせ、対象サイトのコンテンツを漏れなく発見・分類・要約する。出力は RAG ナレッジベースの素材としても利用可能なよう、ソースURL・ライセンス情報を付記する。

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「Fintanから○○関連のコンテンツを全て調べてほしい」
- 「○○サイトの△△に関する技術情報を網羅的に収集して」
- 「ナレッジサイトの特定カテゴリを調査してレポートにまとめて」
- 「RAG用のナレッジベース素材として○○サイトの情報を収集して」
- 「○○技術の公式ドキュメントサイトを体系的に整理して」
- 特定のナレッジサイトから技術情報を収集し、構造化されたMarkdownとして出力する必要がある場合

**トリガーキーワード**: Fintan調査, サイト調査, コンテンツ収集, ナレッジ収集, 技術情報スクレイピング, site:調査, 網羅的調査

**主要ターゲットサイト**:
- https://fintan.jp/ （TIS ナレッジサイト — 主要ユースケース）
- その他のナレッジサイト（developer.mozilla.org, docs.aws.amazon.com, learn.microsoft.com 等）

## Instructions

### Phase 1: 全体把握（サイト構造の理解）

対象サイトの全体構造を把握し、調査の土台を作る。

#### Step 1.1: トップページの構造把握

```
【実行手順】
1. WebFetch で対象サイトのトップページにアクセス
   - URL例: https://fintan.jp/en/ （英語版があれば英語版も確認）
   - 抽出項目: カテゴリ一覧、メニュー構造、コンテンツ数

2. WebSearch で「site:{対象サイト}」を検索
   - 目的: インデックスされているページの全体像を把握
   - リンク一覧から主要セクションを特定
```

#### Step 1.2: カテゴリ/ロール別ページの特定

```
【Fintanの場合の既知構造】
- コンテンツ一覧: https://fintan.jp/?page_id=182
- アーキテクト向け: https://fintan.jp/for-architect/
- バックエンドエンジニア向け: https://fintan.jp/for-backend-engineer/
- フロントエンドエンジニア向け: https://fintan.jp/for-frontend-engineer/
- モバイルエンジニア向け: https://fintan.jp/for-mobile-engineer/
- デザイナー向け: https://fintan.jp/for-designer/
- 新規事業関係者向け: https://fintan.jp/for-new-biz-starter/
- 分類体系: SWEBOK V3.0 に基づく

【他サイトの場合】
- サイトマップ（/sitemap.xml）の確認
- フッター/ヘッダーのナビゲーション構造の確認
- カテゴリページ・タグページの発見
```

#### Step 1.3: About ページ・利用規約の確認

```
【確認項目】
1. サイトの運営組織
2. コンテンツのライセンス（CC BY, Apache 2.0 等）
3. 利用規約・再利用ポリシー
4. 商標情報

【Fintanの場合の既知情報】
- 運営: TIS株式会社テクノロジー&イノベーション本部
- ドキュメント: CC BY 4.0（標準）/ CC BY-SA 4.0（開発標準）
- コードサンプル: Apache License 2.0
- 一部: Fintan コンテンツ 使用許諾条項（独自）
- 参照: https://fintan.jp/about/
```

### Phase 2: 対象コンテンツ探索（多段階検索）

WebSearch の `site:` 演算子を活用して、対象キーワードに関連するページを網羅的に発見する。

#### Step 2.1: 基本検索（必須）

```
【検索パターン — 全て並列実行可能】

1. メインキーワード検索
   WebSearch: "site:{対象サイト} {キーワード}"
   例: "site:fintan.jp Nablarch"

2. 日本語バリエーション検索
   WebSearch: "site:{対象サイト} {キーワード日本語}"
   例: "site:fintan.jp ナブラーク"（カタカナ表記がある場合）

3. 英語バリエーション検索（日本語サイトの場合）
   WebSearch: "site:{対象サイト}/en/ {keyword}"
   例: "site:fintan.jp/en/ Nablarch"
```

#### Step 2.2: 関連キーワード展開検索（深度 standard 以上）

対象キーワードから派生する関連キーワードで追加検索を行う。

```
【展開パターン — 実践で有効だった検索例】

技術フレームワークの場合:
- "{キーワード} + 開発ガイド/Development Guide"
- "{キーワード} + テスト/Testing"
- "{キーワード} + コーディング規約/Coding Standards"
- "{キーワード} + セキュリティ/Security"
- "{キーワード} + 導入事例/Case Study"
- "{キーワード} + ハンズオン/Tutorial"
- "{キーワード} + 環境構築/Setup"
- "{キーワード} + SPA REST API"（Webフレームワークの場合）
- "{キーワード} + バッチ/Batch"（エンタープライズFWの場合）

汎用技術の場合:
- "{キーワード} + ベストプラクティス"
- "{キーワード} + CI/CD"
- "{キーワード} + 品質/Quality"
- "{キーワード} + 学習/Learning"
```

#### Step 2.3: 著者・カテゴリ起点の検索（深度 thorough のみ）

```
【追加検索パターン】
- "site:{対象サイト} author:{著者名}"
  例: "site:fintan.jp author:Nablarch Team"
- "site:{対象サイト} {カテゴリ名}"
  例: "site:fintan.jp ソフトウェアテスティング"
- "site:{対象サイト} {タグ名}"
  例: "site:fintan.jp tag:react"
```

#### Step 2.4: 重複排除と一覧作成

```
【実行手順】
1. 全検索結果のURLを収集
2. 重複URLを排除（同一ページの英語版/日本語版は両方記録）
3. URLごとにタイトル・概要を記録
4. 以下の優先度でフェッチ対象を決定:
   - 高: キーワード直接関連ページ
   - 中: 関連技術ページ
   - 低: 間接的に関連するページ（カテゴリ/タグページ等）
```

### Phase 3: ページ内容取得（WebFetch）

#### Step 3.1: フェッチ戦略

```
【深度別フェッチ方針】

quick（概要のみ）:
- 優先度「高」のページのみフェッチ（最大10ページ）
- prompt: "Extract the title, key topics, and main takeaways"

standard（主要ページ詳細）:
- 優先度「高」「中」のページをフェッチ（最大25ページ）
- prompt: "Extract all information comprehensively"

thorough（全ページ詳細）:
- 全ページをフェッチ（ページ数制限なし）
- prompt: "Extract ALL information. Be comprehensive and exhaustive"
```

#### Step 3.2: WebFetch 実行のコツ（実践知見）

```
【並列実行】
- 独立した4ページまでは並列でWebFetchを実行可能
- 依存関係のないページは常に並列で取得せよ

【プロンプトの言語】
- 日本語サイト → 日本語でpromptを書くとより正確な抽出が可能
  例: "この記事の内容を全て抽出せよ。技術スタック、手順、成果物を日本語で出力。"
- 英語サイト → 英語でpromptを書く

【レートリミット対策】
- WebFetchでレートリミットに遭遇した場合:
  1. エラーメッセージのリセット時間を記録
  2. 取得済みの情報で中間レポートを作成
  3. リセット後に残りのページを取得

【アクセス不可ページの記録】
- 403/404/タイムアウト → 「アクセスできなかったリソース」に記録
- PDF → WebFetchで解析不可の場合あり → URLのみ記録
- リダイレクト → リダイレクト先URLで再フェッチ
```

#### Step 3.3: 情報抽出テンプレート

各ページから以下の情報を抽出する：

```
【抽出項目】
- タイトル
- URL
- 公開日 / 更新日
- 著者 / 組織
- 概要（2-3文）
- 主要コンテンツ（箇条書き）
- 技術スタック（該当する場合）
- GitHub リポジトリURL（該当する場合）
- ライセンス（該当する場合）
- 関連ページへのリンク
```

### Phase 4: 体系化（Markdown レポート作成）

#### Step 4.1: カテゴリ分類

収集した情報を以下のカテゴリに分類する。対象キーワードに応じてカテゴリを調整する。

```
【標準カテゴリ（技術フレームワーク調査の場合）】
1. サイト概要（運営組織、構造、カテゴリ）
2. コンテンツ一覧（全ページのタイトル・URL・概要テーブル）
3. 開発ガイド / ドキュメント
4. チュートリアル / ハンズオン
5. コーディング規約 / 静的解析
6. テスト関連
7. セキュリティ関連
8. CI/CD / DevOps
9. 導入事例 / ベストプラクティス
10. 開発ツール / 環境構築
11. 関連コンテンツ（対象技術以外）
12. ライセンス・利用規約
13. GitHub リポジトリ一覧
14. 付録: アクセスできなかったリソース
```

#### Step 4.2: レポート作成

「Output Format」セクションのテンプレートに従い、Markdown レポートを作成する。

```
【レポート品質基準】
- 全ての情報にソースURLを付記する
- テーブルは Markdown テーブルで記述する
- コード例は適切なコードブロックで囲む
- 目次を含める（リンク付き）
- 各セクションの冒頭に概要を記載する
- 日本語コンテンツは日本語で記載してよい
```

### Phase 5: メタ情報付加

#### Step 5.1: ライセンス情報の整理

```
【記載項目】
1. サイト全体のデフォルトライセンス
2. コンテンツ種別ごとのライセンス
3. RAG ナレッジベースとしての利用可否判定
   - CC BY / CC BY-SA / Apache 2.0 / MIT → 利用可能（条件あり）
   - 独自ライセンス → 要確認
   - 利用規約で再利用禁止 → 利用不可
4. 注意事項（商標、帰属表示の要件等）
```

#### Step 5.2: アクセス不可リソースの記録

```
【記録形式】
| リソース | URL（わかる場合） | 理由 |
|---------|------------------|------|
| XXX PDF | https://... | PDFバイナリのため詳細抽出不可 |
| YYY ページ | https://... | 403 Forbidden |
| ZZZ ページ | https://... | レートリミットにより未取得 |
```

#### Step 5.3: レポートヘッダーの付加

```markdown
# {対象サイト名} {キーワード}関連コンテンツ調査報告

> **調査日**: {YYYY-MM-DD}
> **調査者**: {エージェント名}
> **対象**: {サイトURL}
> **キーワード**: {検索キーワード}
> **深度**: {quick / standard / thorough}

---
```

## Input Format

スキル実行時に以下のパラメータを指定する。

```yaml
# 必須パラメータ
site_url: "https://fintan.jp/"        # 対象サイトのURL
keywords:                              # 検索キーワード（1つ以上）
  - "Nablarch"
output_path: "output/nablarch_kb_fintan.md"  # 出力ファイルパス

# オプションパラメータ
depth: "standard"                      # quick / standard / thorough（デフォルト: standard）
language: "ja"                         # 出力言語（デフォルト: ja）
exclude_keywords:                      # 除外キーワード（該当ページをスキップ）
  - "Xenlon"
additional_search_terms:               # 追加検索キーワード（展開検索に追加）
  - "SPA REST API"
  - "テスト自動化"
```

### パラメータ説明

| パラメータ | 必須 | デフォルト | 説明 |
|----------|------|----------|------|
| `site_url` | ✅ | — | 対象サイトのURL。`site:` 演算子で使用 |
| `keywords` | ✅ | — | メイン検索キーワード。1つ以上指定 |
| `output_path` | ✅ | — | 出力Markdownファイルのパス |
| `depth` | — | `standard` | 調査深度。`quick`=概要のみ、`standard`=主要ページ詳細、`thorough`=全ページ詳細 |
| `language` | — | `ja` | 出力言語 |
| `exclude_keywords` | — | `[]` | 除外キーワード |
| `additional_search_terms` | — | `[]` | 追加検索キーワード |

## Output Format

### ファイル構造テンプレート

```markdown
# {サイト名} {キーワード}関連コンテンツ調査報告

> **調査日**: YYYY-MM-DD
> **調査者**: {agent_name}
> **対象**: {site_url}
> **キーワード**: {keywords}
> **深度**: {depth}

---

## 目次

1. [サイト概要](#1-サイト概要)
2. [{キーワード}関連コンテンツ一覧](#2-キーワード関連コンテンツ一覧)
3. [{カテゴリ1}](#3-カテゴリ1)
...
N-2. [ライセンス・利用規約](#n-2-ライセンス利用規約)
N-1. [GitHub関連リポジトリ一覧](#n-1-github関連リポジトリ一覧)
N. [付録: アクセスできなかったリソース](#n-付録)

---

## 1. サイト概要

### 1.1 サイトについて
- URL: {site_url}
- 運営: {organization}
- 目的: {purpose}

### 1.2 コンテンツカテゴリ
| カテゴリ | 件数 |
|---------|------|
| ... | ... |

---

## 2. {キーワード}関連コンテンツ一覧

### 2.1 直接関連コンテンツ
| # | タイトル | URL | 言語 | 概要 |
|---|---------|-----|------|------|
| 1 | ... | ... | ... | ... |

### 2.2 間接関連コンテンツ
| # | タイトル | URL | 概要 |
|---|---------|-----|------|
| ... | ... | ... | ... |

---

## 3-N. カテゴリ別詳細

### {カテゴリ名}

#### {サブトピック}
**URL**: {url}
{詳細内容}

---

## N-2. ライセンス・利用規約

### デフォルトライセンス
| コンテンツ種別 | ライセンス | RAG利用可否 |
|-------------|----------|-----------|
| ... | ... | ... |

### RAG構築時の留意事項
- ...

---

## N-1. GitHub関連リポジトリ一覧
| リポジトリ | URL | 内容 |
|----------|-----|------|
| ... | ... | ... |

---

## N. 付録: アクセスできなかったリソース
| リソース | URL | 理由 |
|---------|-----|------|
| ... | ... | ... |
```

## Examples

### Example 1: Fintan Nablarch 調査（standard）

```
入力:
  site_url: "https://fintan.jp/"
  keywords: ["Nablarch"]
  output_path: "output/nablarch_kb_fintan.md"
  depth: "standard"
  exclude_keywords: ["Xenlon"]

検索パターン（実際に実行した例）:
  1. "site:fintan.jp Nablarch"                     → 12件発見
  2. "site:fintan.jp Nablarch システム開発ガイド"     → 10件発見
  3. "site:fintan.jp Nablarch SPA REST API"         → 10件発見
  4. "site:fintan.jp Nablarch 開発環境 セットアップ"  → 10件発見
  5. "site:fintan.jp Nablarch コーディング規約"       → 10件発見
  6. "site:fintan.jp Nablarch 導入事例"              → 10件発見
  7. "site:fintan.jp Nablarch テスト Excel JUnit"    → 10件発見
  8. "site:fintan.jp Nablarch 学習 ハンズオン"        → 10件発見
  9. "site:fintan.jp Nablarch セキュリティ 脆弱性"    → 10件発見
  10. "site:fintan.jp Java CI/CD 品質"               → 10件発見
  11. "site:fintan.jp 利用規約 ライセンス"             → 10件発見
  12. "site:fintan.jp 要件定義 テスト観点 カタログ"     → 10件発見
  13. "site:fintan.jp Fintan コンテンツ一覧 カテゴリ"  → 10件発見
  14. "site:fintan.jp Nablarch バッチ メッセージング"   → 10件発見

WebFetch実行: 20ページ以上をフェッチ

出力: 12セクション構成のMarkdownレポート（約800行）
  - Nablarch公式ページ群: 12件
  - SPA + REST APIシリーズ: 8件
  - 関連コンテンツ: 5件+
  - ライセンス情報: 4種別のライセンスを特定
  - GitHub リポジトリ: 6件
```

### Example 2: Fintan テスト自動化 調査（quick）

```
入力:
  site_url: "https://fintan.jp/"
  keywords: ["テスト自動化", "テスト観点"]
  output_path: "output/fintan_testing.md"
  depth: "quick"

検索パターン:
  1. "site:fintan.jp テスト自動化"
  2. "site:fintan.jp テスト観点"
  3. "site:fintan.jp テスト種別"

WebFetch実行: 主要5-10ページのみ

出力: 簡潔なMarkdownレポート
```

### Example 3: MDN Web Docs CSS Grid 調査（standard）

```
入力:
  site_url: "https://developer.mozilla.org/"
  keywords: ["CSS Grid"]
  output_path: "output/mdn_css_grid.md"
  depth: "standard"
  language: "en"

検索パターン:
  1. "site:developer.mozilla.org CSS Grid"
  2. "site:developer.mozilla.org CSS Grid Layout"
  3. "site:developer.mozilla.org grid-template"
  4. "site:developer.mozilla.org CSS Grid examples"

出力: 英語のMarkdownレポート
```

### Example 4: AWS ドキュメント Lambda 調査（thorough）

```
入力:
  site_url: "https://docs.aws.amazon.com/"
  keywords: ["Lambda", "serverless"]
  output_path: "output/aws_lambda_docs.md"
  depth: "thorough"
  language: "ja"
  additional_search_terms: ["SAM", "API Gateway integration"]

検索パターン:
  1. "site:docs.aws.amazon.com Lambda"
  2. "site:docs.aws.amazon.com Lambda best practices"
  3. "site:docs.aws.amazon.com Lambda SAM"
  4. "site:docs.aws.amazon.com Lambda API Gateway"
  5. "site:docs.aws.amazon.com serverless"
  6. "site:docs.aws.amazon.com Lambda security"
  7. "site:docs.aws.amazon.com Lambda monitoring"
  ...

出力: 包括的なMarkdownレポート（全ページフェッチ）
```

## Guidelines

### 必須ルール

1. **`site:` 演算子を必ず使用すること**
   - WebSearch の `site:` 演算子が最も効率的な発見手段
   - 対象サイト以外の情報を混入させないために必須

2. **並列実行を最大限活用すること**
   - 独立した WebSearch は全て並列実行
   - 独立した WebFetch は4件まで並列実行
   - これにより調査時間を大幅に短縮できる

3. **全ての情報にソースURLを付記すること**
   - Markdownレポート内の各情報にはソースページのURLを必ず記載
   - `**URL**: https://...` または `**ソース**: https://...` の形式

4. **ライセンス情報を必ず調査・記載すること**
   - サイト全体のライセンス
   - コンテンツ種別ごとのライセンス
   - RAG ナレッジベースとしての利用可否判定

5. **アクセスできなかったリソースを記録すること**
   - 理由（404、レートリミット、PDF解析不可等）を明記
   - 後からリトライ可能なようURLを記録

### 検索のコツ（実践知見）

1. **キーワードの展開が発見率を大幅に向上させる**
   - メインキーワードだけでは全体の60-70%しか発見できない
   - 関連キーワード5-10個で追加検索すると90%以上をカバー可能

2. **日本語サイトは英語版も確認せよ**
   - Fintanのように `/en/` パスで英語版を持つサイトがある
   - 英語版の方が情報が多い場合もある

3. **カテゴリ/タグページは宝庫**
   - `site:fintan.jp/blog-category/` のようなカテゴリページ
   - `site:fintan.jp/blog-tag/` のようなタグページ
   - これらから関連コンテンツを芋づる式に発見できる

4. **著者ページも有効**
   - 特定チーム・著者の全記事を一覧できる
   - 例: `site:fintan.jp/author/nablarch-team/`

5. **WebFetch のプロンプトは具体的に**
   - 悪い例: "ページの内容を教えて"
   - 良い例: "この記事の内容を全て抽出せよ。技術スタック、手順、成果物、GitHub リンクを網羅的に記録。日本語で出力。"

### アンチパターン（避けるべきこと）

- **検索パターンの不足**: メインキーワード1つだけで検索を終わらせる（コンテンツの30%以上を見逃す）
- **逐次フェッチ**: WebFetch を1件ずつ実行する（並列実行で4倍速になる）
- **ライセンス調査の省略**: RAG利用時に法的リスクが生じる
- **アクセス不可リソースの無視**: 記録しないと後からリトライできない
- **レートリミット時の強制リトライ**: リセット時間を待つか、中間レポートを作成する
- **目次なしのレポート**: 長大なレポートは目次がないと読めない

### 深度別の目安

| 深度 | WebSearch回数 | WebFetch回数 | 所要API呼び出し | レポート行数 |
|------|-------------|-------------|----------------|------------|
| quick | 3-5回 | 5-10ページ | 10-20回 | 100-300行 |
| standard | 10-15回 | 15-25ページ | 30-50回 | 400-800行 |
| thorough | 15-25回 | 25ページ以上 | 50-80回 | 800行以上 |
