---
name: framework-knowledge-builder
description: 任意のフレームワーク・技術について、公式ドキュメント・GitHubリポジトリ・ソースコード・コミュニティ情報を並列に調査し、体系的なナレッジベースを構築する包括的スキル。「○○のナレッジベースを構築して」「○○のアーキテクチャを詳細に調査して」「○○の公式ドキュメントを網羅的にKBにまとめて」「○○のGitHub/ドキュメント/標準を並列調査して」「○○のハンドラキュー構成を調べて」「○○のFW全体像を体系的にドキュメント化して」といった要望に対応する。WebFetch + WebSearch + GitHub CLI を駆使した6フェーズ並列調査・統合文書化スキル。
---

# Framework Knowledge Builder — 包括的ナレッジベース構築

## Overview

任意のフレームワーク・技術について、複数の情報源を**並列に調査**し、体系的なナレッジベース（KB）を構築する包括的スキル。公式ドキュメントからソースコードレベルの正確な技術情報まで、アーキテクチャ詳細を含む網羅的なドキュメントを生成する。

**統合元スキル:**
- S-006 nablarch-architecture-explorer: FW固有のアーキテクチャ詳細調査（FQCN付き）
- S-007 kb-builder: 並列調査設計・RACE-001回避・統合文書化
- S-008 framework-documentation-surveyor: 公式ドキュメント網羅的調査
- S-009 framework-architecture-explorer: FW非依存のアーキテクチャ調査

**主な用途:**
- 新技術の採用検討に必要な情報の網羅的収集
- フレームワークのアーキテクチャ詳細調査（FQCN、設定例、処理フロー付き）
- RAGシステム用のナレッジベース素材作成
- MCP サーバー開発に必要な技術基盤情報の調査
- 技術デューデリジェンスの情報基盤構築
- 既存技術の体系的ドキュメント化

**このスキルの特徴:**
- 6フェーズのシステマティックな調査プロセス
- 領域分割 → 並列調査 → 統合文書化（RACE-001完全回避）
- アーキテクチャ詳細（FQCN、XML/YAML設定例、処理フロー図）対応
- gh CLI / WebFetch / WebSearch の3ツール使い分け
- 実績ベースのベストプラクティス（Nablarch KB構築で6並列調査を実証済み）

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「○○のナレッジベースを構築して」
- 「○○のアーキテクチャを詳細に調査して」
- 「○○の公式ドキュメントを網羅的にまとめて」
- 「○○のGitHub/ドキュメント/標準を並列調査して」
- 「○○の設定パターンを体系的に調べて」
- 「○○のモジュール構成と依存関係を調査して」
- 「○○の技術情報を正確なクラス名付きでドキュメント化して」
- 「RAG用に○○のKBを作成して」
- 「○○について複数領域を分担して調査して」
- フレームワークのアーキテクチャ情報を正確なFQCN・設定例付きでドキュメント化する必要がある場合
- 複数エージェントで分担して並列に技術調査したい場合

**トリガーキーワード**: ナレッジベース構築, KB構築, 網羅的調査, アーキテクチャ調査, 体系的調査, 並列調査, 技術調査, ドキュメント化, FQCN, 設定パターン

## Instructions

### Phase 1: 調査設計（領域分割・ファイルパス確定・並列度決定）

対象技術の全体像を把握し、並列実行可能な調査領域に分割する。

#### Step 1.1: 情報源の特定

```
【実行手順】

1. 対象技術の基本情報を確認
   - 公式サイトURL
   - GitHub organization / repository
   - 公式ドキュメントサイト
   - 開発元組織
   - ドキュメント言語（日本語/英語/多言語）

2. 利用可能な情報源を列挙し優先度を決定
   | 優先度 | 情報源 | 例 |
   |--------|--------|-----|
   | 高 | GitHub（ソースコード、README、Releases） | github.com/{org} |
   | 高 | 公式ドキュメント（ガイド、API、チュートリアル） | {framework}.dev/docs |
   | 中 | 開発標準（コーディング規約、設計テンプレート） | {org}-development-standards |
   | 中 | ナレッジサイト（Fintan、MDN、AWS Docs等） | fintan.jp 等 |
   | 低 | コミュニティ（Qiita、Zenn、Stack Overflow） | qiita.com 等 |
   | 高 | アーキテクチャ（設計思想、モジュール構成） | ソースコード分析 |

3. 対象技術のドキュメント構造を事前確認
   WebFetch(url: "{公式URL}", prompt: "サイト全体の構造、カテゴリ一覧、ページ数を把握せよ")
```

#### Step 1.2: 調査領域の分割

```
【分割原則】

1. 各領域は独立して調査可能であること
   - 領域Aの結果が領域Bの調査に必要、という依存関係がないこと
   - 例外: 統合フェーズでの相互参照は可

2. 各領域は別ファイルに出力すること（RACE-001回避）
   - 1ファイル = 1エージェントの原則を厳守
   - 統合目次ファイルは全領域完了後に別途作成

3. 領域数は利用可能なエージェント数に合わせる
   - 2-3エージェント → 3-4領域（領域統合）
   - 4-6エージェント → 5-7領域（標準分割）
   - 7-8エージェント → 6-8領域（細分割、8以上は管理コスト過大）

4. 各領域の作業量が概ね均等になるよう調整

【標準6領域分割（推奨）】
| 領域ID | 領域名 | 主要ツール | 出力ファイル |
|--------|--------|-----------|-------------|
| D1 | 公式ドキュメント | WebFetch, WebSearch | {tech}_kb_official_docs.md |
| D2 | GitHubリポジトリ | gh CLI, gh api | {tech}_kb_github.md |
| D3 | 開発標準・ガイド | gh CLI, WebFetch | {tech}_kb_dev_standards.md |
| D4 | ナレッジサイト | WebSearch, WebFetch | {tech}_kb_{site_name}.md |
| D5 | コミュニティ | WebSearch, WebFetch | {tech}_kb_community.md |
| D6 | アーキテクチャ | gh CLI, WebFetch, ソース分析 | {tech}_kb_architecture.md |

【単独エージェント実行時の縮約分割（3領域）】
| 領域ID | 領域名 | 内容 |
|--------|--------|------|
| D1+D4 | ドキュメント・ナレッジ | 公式ドキュメント + ナレッジサイト |
| D2+D3 | GitHub・開発標準 | リポジトリ + コーディング規約 + テスト標準 |
| D5+D6 | コミュニティ・アーキテクチャ | 開発者評価 + 設計思想 + モジュール構成 |
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

例（Spring Bootの場合）:
  output/springboot_kb_official_docs.md
  output/springboot_kb_github.md
  output/springboot_kb_ecosystem.md
  output/springboot_kb_community.md

【重要】
- ファイルパスは調査開始前に全て確定させる
- 各エージェントに「自分の出力ファイル」を明示して割り当てる
- 統合目次ファイルは全領域完了後に別途作成
```

### Phase 2: 公式ドキュメント調査（WebFetch + 構造抽出）

公式ドキュメントサイトの構造を把握し、主要コンテンツを体系的に収集する。

#### Step 2.1: サイト構造の把握

```
【実行手順】

1. トップページをWebFetchで取得
   WebFetch(url: "{公式URL}", prompt: "サイト全体の構造を把握せよ。
   メインメニュー、カテゴリ一覧、ページ数を抽出。")

2. サイトマップの確認（存在する場合）
   WebFetch(url: "{公式URL}/sitemap.xml", prompt: "全ページのURLを一覧化せよ")

3. WebSearchで補完
   WebSearch: "site:{ドキュメントドメイン} {技術名}"
   WebSearch: "site:{ドキュメントドメイン} architecture guide reference"

4. ドキュメント構造をカテゴリに分類
   - アーキテクチャ概要
   - 各機能/モジュールのガイド
   - APIリファレンス
   - チュートリアル / Getting Started
   - FAQ / トラブルシューティング
   - リリースノート
```

#### Step 2.2: セクション別コンテンツ取得

```
【取得戦略 — 並列実行推奨】

1. アーキテクチャ・設計ガイド（優先度: 高）
   prompt: "このページの全内容を抽出せよ。アーキテクチャ概念、設計思想、
   主要コンポーネント名、設定例、処理フローを網羅的に記録。"

2. 機能別ガイド（優先度: 高）
   prompt: "このページの全内容を抽出せよ。クラス名（FQCN）、
   設定例（XML/YAML/Java Config）、制約事項、パフォーマンス注意点を含む。"

3. APIリファレンス（優先度: 中）
   prompt: "主要クラス・インターフェースの名前、責務、主要メソッドを抽出。"

4. チュートリアル（優先度: 高）
   prompt: "手順、前提条件、サンプルコードを全て抽出。"

5. FAQ / トラブルシューティング（優先度: 中）
   prompt: "全項目のQ&Aを抽出。"

6. リリースノート（優先度: 低 — 最新2-3バージョンのみ）
   prompt: "最新バージョンの変更内容を抽出。破壊的変更に注目。"

【並列実行ルール】
- 独立したページは4件まで並列でWebFetch実行可能
- 依存関係のないページは常に並列で取得せよ
```

#### Step 2.3: 多言語対応

```
【日本語技術の場合】（Nablarch等）
- 日本語ドキュメントを優先取得
- 日本語プロンプトでWebFetchするとより正確に情報抽出できる
- 英語版の有無を確認し、差分がある場合は注記

【グローバル技術の場合】（Spring, React等）
- 英語ドキュメントを優先取得
- 日本語訳の存在確認と翻訳品質チェック
- 未翻訳セクションの有無を記録
```

### Phase 3: GitHubリポジトリ調査（gh CLI + 分類 + メトリクス）

GitHub organization / repository の情報を網羅的に収集・分類する。

#### Step 3.1: リポジトリ一覧の取得と分類

```
【gh CLI コマンド】

# Organization の場合
gh repo list {org} --limit 200 --json name,description,stargazerCount,updatedAt,language,isArchived

# 注意: フィールド名は stargazerCount（stargazersCount ではない）

# 単一リポジトリの場合
gh repo view {owner}/{repo} --json name,description,stargazerCount,forksCount,updatedAt

【分類カテゴリ（フレームワーク共通）】
- Core ライブラリ（基盤機能）
- Framework モジュール（実行制御、アプリケーション種別対応）
- Common Component（業務機能、共通部品）
- Adaptor / Integration（外部連携）
- Testing（テスティングFW、ユーティリティ）
- Build / BOM（親POM、バージョン管理）
- Sample / Example（サンプルプロジェクト）
- Tools / Utilities
- Documentation

【結果テーブル形式】
| リポジトリ | 説明 | Stars | 最終更新 | 言語 | カテゴリ |
|-----------|------|-------|---------|------|---------|
```

#### Step 3.2: 主要リポジトリの詳細調査

```
【README取得 — 並列実行可能】

gh repo view {owner}/{repo}
# or
gh api repos/{owner}/{repo}/readme --jq '.content' | base64 -d

【取得対象の選定基準】
- Stars数が多いリポジトリ（上位10-15件）
- フレームワークのコアリポジトリ
- ドキュメント / サンプルリポジトリ
- 最近更新されたリポジトリ

【READMEから抽出する情報】
- プロジェクトの目的・概要
- 主要クラス・インターフェースの一覧
- モジュール構成
- ビルド方法
- 依存関係
```

#### Step 3.3: メトリクス収集

```
【gh API コマンド — 並列実行可能】

# リポジトリ基本統計
gh api repos/{owner}/{repo} --jq '{
  stars: .stargazers_count,
  forks: .forks_count,
  open_issues: .open_issues_count,
  created: .created_at,
  updated: .pushed_at
}'

# コントリビューター数（主要リポジトリのみ）
gh api repos/{owner}/{repo}/contributors --jq 'length'

# リリースタグ
gh api repos/{owner}/{repo}/tags --jq '.[].name'

【レート制限対策】
- gh api は1時間5000リクエスト
- 主要リポジトリ（10-15件）に絞って詳細取得
- 制限到達時は取得済みデータで中間レポートを作成
```

#### Step 3.4: リポジトリ間依存関係

```
【依存関係の調査方法】

1. 親POM / BOM の確認
   - pom.xml: <parent> 要素, <dependencyManagement>
   - build.gradle: dependencies ブロック
   - package.json: dependencies/peerDependencies

2. 依存関係の図式化（テキスト表現）
   {parent} → {core} → {fw} → {fw-web/batch/jaxrs}

3. モジュール一覧テーブル
   | カテゴリ | artifactId / package | groupId / scope | 責務 |
   |---------|---------------------|----------------|------|
```

### Phase 4: アーキテクチャ詳細調査（ソースコード分析・設定パターン抽出）

ソースコードレベルの正確なアーキテクチャ情報を収集する。

#### Step 4.1: コアインターフェース・クラスの特定

```
【実行手順】

1. フレームワークの核心となるインターフェース/クラスを特定
   - 公式ドキュメントやREADMEで言及されるクラス
   - パイプライン/ミドルウェア/ハンドラ等のパターンの基底型

2. ソースコード取得
   gh api repos/{org}/{repo}/contents/src/main/java/{path} --jq '.content' | base64 -d
   # または
   WebFetch: "https://github.com/{org}/{repo}/blob/{branch}/src/main/java/{path}"

3. 確認すべき情報
   - publicメソッドのシグネチャ
   - 型パラメータ
   - Javadoc / JSDoc コメント
   - アノテーション定義
   - 継承・実装関係
```

#### Step 4.2: 設定パターンの抽出

```
【サンプルプロジェクトのXML/YAML/Config取得 — 並列実行推奨】

1. メイン設定ファイルの取得
   WebFetch: "https://github.com/{org}/{example-repo}/blob/{branch}/src/main/resources/{config-file}"

2. 設定ファイル構造の分析
   - コンポーネント定義（DI設定）
   - パイプライン/ミドルウェア構成
   - DB接続設定
   - ロギング設定
   - セキュリティ設定

3. アプリケーション種別間の比較（フレームワークが複数種別をサポートする場合）
   - 共通設定 vs 種別固有設定
   - 設定のカスタマイズポイント
```

#### Step 4.3: 処理フローの分析

```
【実行手順】

1. リクエスト処理フローの特定
   - エントリーポイント → ミドルウェア/ハンドラ → ビジネスロジック → レスポンス
   - スレッドモデル（シングルスレッド/マルチスレッド）

2. フロー図の作成（ASCII / Mermaid）
   リクエスト → [Handler1] → [Handler2] → ... → [Action]
                  ↑                                    |
                  ← [Handler1] ← [Handler2] ← ... ← 戻り

3. アプリケーション種別ごとのフロー差分
   - Web vs API vs Batch vs Messaging
   - 同期 vs 非同期
```

#### Step 4.4: FQCN/クラス名の検証

```
【検証手順 — 深度 detailed 以上】

1. ドキュメントに記載した全FQCN/クラス名について存在を確認
   gh api search/code --method GET \
     -f q="{ClassName} org:{org} language:java" \
     --jq '.total_count'

2. 存在しないクラスが見つかった場合:
   - 正しいクラス名をGitHub検索で特定
   - ドキュメントを修正
   - 推測で補完した場合は「【推測】」と明記

3. パッケージ名の検証:
   - ソースコードのpackage文と一致するか確認
   - 特にモジュール間のパッケージ配置の差異に注意

【レート制限】
- gh api search/code は1分間に10リクエストまで
- 大量のFQCN検証時は間隔を空けて実行
```

### Phase 5: コミュニティ・外部情報調査（WebSearch）

技術ブログ、Q&Aサイト、カンファレンス資料等からコミュニティ情報を収集する。

#### Step 5.1: 技術ブログ・記事検索

```
【検索パターン — 並列実行可能】

1. 日本語ブログプラットフォーム
   WebSearch: "site:qiita.com {技術名}"
   WebSearch: "site:zenn.dev {技術名}"
   WebSearch: "{技術名} 使ってみた OR 所感 OR 入門"

2. 英語圏
   WebSearch: "{技術名} tutorial OR getting-started OR review"
   WebSearch: "site:stackoverflow.com {技術名}"
   WebSearch: "site:dev.to {技術名}"

3. 別名・カタカナ検索（日本語技術の場合）
   WebSearch: "site:qiita.com {カタカナ名}"
```

#### Step 5.2: ベストプラクティス・Tips収集

```
【検索パターン】

WebSearch: "{技術名} best practices"
WebSearch: "{技術名} tips tricks"
WebSearch: "{技術名} common mistakes OR pitfalls"
WebSearch: "{技術名} performance optimization"

【主要記事は WebFetch で深掘り】
- いいね数が多い記事
- 具体的なコード例を含む記事
- 設定のカスタマイズ事例
```

#### Step 5.3: ナレッジサイト調査

```
【技術に応じたナレッジサイト】

Java系:
  WebSearch: "site:fintan.jp {技術名}"
  WebFetch: Fintan記事を個別取得

Web系:
  WebSearch: "site:web.dev {技術名}"
  WebSearch: "site:developer.mozilla.org {技術名}"

クラウド系:
  WebSearch: "site:aws.amazon.com {技術名}"
  WebSearch: "site:cloud.google.com {技術名}"
```

### Phase 6: 体系化・統合文書化（テンプレート適用・品質チェック）

収集した情報をカテゴリ別に整理し、構造化されたMarkdownドキュメントを生成する。

#### Step 6.1: カテゴリ別構成

```
【調査領域別の標準カテゴリ】

アーキテクチャKBの場合:
1. アーキテクチャ概要（設計思想、パターン）
2. コアコンポーネント一覧（FQCN/クラス名付きテーブル）
3. パイプライン/ミドルウェア構成（種別ごと）
4. 設定パターン（XML/YAML/Config例）
5. DB/トランザクション管理
6. リクエスト処理フロー
7. 内部アーキテクチャ（モジュール構成、拡張ポイント）

公式ドキュメントKBの場合:
1. ドキュメント構造概要
2. アプリケーション種別（Web/API/Batch等）
3. 主要機能ガイド
4. 設定リファレンス
5. チュートリアル要約
6. FAQ / トラブルシューティング

開発標準KBの場合:
1. コーディング規約
2. 設計テンプレート
3. テスティングFW
4. CI/CDパイプライン
5. サンプルプロジェクト構成
```

#### Step 6.2: テーブル形式の標準化

```
【コンポーネント一覧テーブル — 必須フォーマット】
| コンポーネント名 | FQCN / クラス名 | カテゴリ | 責務 |
|---------------|----------------|---------|------|

【パイプライン構成テーブル — 種別ごと】
| 順序 | コンポーネント名 | FQCN / クラス名 | スレッド | 責務 |
|------|---------------|----------------|---------|------|

【モジュール一覧テーブル】
| モジュール | artifactId / package | groupId / scope | 責務 |
|----------|---------------------|----------------|------|

【リポジトリ一覧テーブル】
| リポジトリ | 説明 | Stars | 最終更新 | 言語 |
|-----------|------|-------|---------|------|

全てのクラス名はFQCN（完全修飾クラス名）で記載すること。
略称のみの記載は禁止。確認できない場合は「未確認」と記載。
```

#### Step 6.3: Markdownドキュメント作成

```
【品質基準】
□ 全クラス名がFQCN（推測箇所は「【推測】」付記）
□ 設定例はコードブロックで囲む（実プロジェクトから引用）
□ 各情報にソースURL / GitHubリンクを付記
□ 目次を含める（リンク付き）
□ 各セクション冒頭に概要を記載
□ 処理フロー図はASCIIアートまたはMermaidで図示
□ 500行以上のファイルには必ず目次をつける
□ 除外キーワード（Xenlon等）が含まれていないこと
```

#### Step 6.4: 統合目次ファイルの作成（全領域完了後）

```
【統合目次テンプレート】

# {技術名} ナレッジベース — 統合目次

> **構築日**: YYYY-MM-DD
> **領域数**: {N}
> **総ファイルサイズ**: 約{X}KB

## 構成ファイル一覧

| # | 領域 | ファイル | 行数 | 主要トピック |
|---|------|---------|------|------------|
| 1 | 公式ドキュメント | [{ファイル名}]({相対パス}) | {行数} | {概要} |
...

## 主要な発見
{全領域を通じた重要な発見事項}

## 横断的な関連情報
{複数領域にまたがる重要な情報}
```

## Input Format

スキル実行時に以下のパラメータを指定する。

```yaml
# 必須パラメータ
technology_name: "Nablarch"                     # 対象技術名
output_dir: "output/"                           # 出力ディレクトリ

# 調査モード選択（2つのモードから選択）
mode: "comprehensive"                           # comprehensive / focused

# comprehensive モード用パラメータ（KB全体を構築）
domains:                                        # 調査領域リスト
  - name: "公式ドキュメント"
    type: "docs"                                # docs / github / standards / site / community / architecture
    output_file: "output/nablarch_kb_official_docs.md"
    params:
      url: "https://nablarch.github.io/docs/LATEST/doc/"
  - name: "GitHub"
    type: "github"
    output_file: "output/nablarch_kb_github.md"
    params:
      org: "nablarch"
  - name: "アーキテクチャ"
    type: "architecture"
    output_file: "output/nablarch_kb_architecture.md"

# focused モード用パラメータ（特定領域を深掘り）
investigation_area: "ハンドラキュー"              # 調査領域
output_path: "output/nablarch_kb_architecture.md"  # 出力ファイルパス
app_type_filter:                                # アプリ種別フィルタ（省略時は全種別）
  - "Web"
  - "Batch"

# 共通オプションパラメータ
depth: "standard"                               # quick / standard / thorough（デフォルト: standard）
parallelism: 6                                  # 並列度（利用可能なエージェント数）
file_naming_pattern: "{tech}_kb_{domain}.md"    # ファイル命名パターン
exclude_keywords:                               # 除外キーワード
  - "Xenlon"
language: "ja"                                  # 出力言語（デフォルト: ja）
generate_index: true                            # 統合目次ファイル生成（デフォルト: true）
```

### パラメータ説明

| パラメータ | 必須 | デフォルト | 説明 |
|----------|------|----------|------|
| `technology_name` | yes | -- | 対象技術の名称 |
| `output_dir` | yes | -- | 出力ディレクトリ |
| `mode` | -- | `comprehensive` | 調査モード。`comprehensive`=KB全体構築、`focused`=特定領域深掘り |
| `domains` | comprehensive時 | -- | 調査領域リスト |
| `investigation_area` | focused時 | -- | 深掘り対象の領域名 |
| `depth` | -- | `standard` | 調査深度 |
| `parallelism` | -- | `4` | 並列度 |
| `exclude_keywords` | -- | `[]` | 除外キーワード |

### 深度別の処理範囲

| Phase | quick | standard | thorough |
|-------|-------|----------|----------|
| 1. 調査設計 | 簡易分割 | 標準6領域分割 | 細分割+追加領域 |
| 2. 公式ドキュメント | トップ+主要5ページ | 関連サブページ含む(15ページ) | 隣接領域含む(制限なし) |
| 3. GitHub | リポジトリ一覧のみ | 一覧+主要リポジトリ詳細 | 全リポジトリ+メトリクス |
| 4. アーキテクチャ | なし | 主要インターフェース・設定 | 全実装クラス+テストコード |
| 5. コミュニティ | 主要プラットフォーム1つ | 全プラットフォーム基本検索 | 全+関連KW展開 |
| 6. 統合文書化 | 概要テーブルのみ | 標準カテゴリ構成 | 全カテゴリ+比較分析 |

### 領域タイプ（type）

| type | 説明 | 主要ツール | 出力内容 |
|------|------|-----------|---------|
| `docs` | 公式ドキュメント調査 | WebFetch, WebSearch | 機能ガイド、APIリファレンス要約 |
| `github` | GitHub org/repo 調査 | gh CLI, gh api | リポジトリ一覧・分類、メトリクス |
| `standards` | 開発標準・ガイド調査 | gh CLI, WebFetch | コーディング規約、テスト標準 |
| `site` | ナレッジサイト調査 | WebSearch (site:), WebFetch | 記事一覧、ベストプラクティス |
| `community` | コミュニティ調査 | WebSearch, WebFetch | 記事一覧、開発者評価 |
| `architecture` | アーキテクチャ調査 | gh CLI, WebFetch, ソース分析 | FQCN付き構成図、設定例 |

## Output Format

### 領域別ファイルテンプレート

```markdown
# {技術名} {領域名} ナレッジベース

> **調査日**: YYYY-MM-DD
> **調査者**: {エージェント名} ({タスクID})
> **対象バージョン**: {version}
> **データソース**: {gh CLI + WebFetch + WebSearch 等}

---

## 目次

1. [セクション1](#1-セクション1)
2. [セクション2](#2-セクション2)
...

---

## 1. セクション1

> **ソース**: [{ページ名}]({URL})

### 1.1 サブセクション

{内容 — FQCN付きテーブル、設定例、処理フロー図を含む}

| コンポーネント | FQCN | 責務 |
|-------------|------|------|
| {名前} | `{完全修飾クラス名}` | {説明} |

```xml
<!-- 設定例（実プロジェクトからの引用） -->
<component name="..." class="...">
  <property name="..." value="..." />
</component>
```

---

## N. ソースURL一覧

### 公式ドキュメント
- [{ページ名}]({URL})

### GitHub リポジトリ
- [{リポジトリ名}]({URL})

---

*本ドキュメントは{エージェント名}が調査・作成した。*
```

## Parallelization Strategy

cmd_007で実際に6足軽が並列実行したナレッジベース構築の実績をベストプラクティスとして記載する。

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
- 統合は1エージェントが担当
```

#### 2. 領域間の独立性チェック

```
【チェックリスト — 各領域ペア(A, B)で確認】
□ 領域Aの調査に領域Bの結果は不要か？
□ 領域Bの調査に領域Aの結果は不要か？
□ 領域Aと領域Bで同じファイルに書き込まないか？
□ 領域Aと領域Bで同じAPIエンドポイントを大量に叩かないか？

【独立でない場合の対処】
- 依存関係あり → 依存元を先に実行、完了後に依存先を開始
- APIレート制限の競合 → 片方をWebSearchベース、もう片方をgh CLIベースに分離
```

#### 3. タスク割り当てテンプレート

```yaml
task:
  task_id: subtask_xxx
  description: |
    【{技術名}情報収集: 領域{N} — {領域名}の調査】
    {技術名}の{領域名}を{調査方法}で調査し、体系的にまとめよ。

    ## 調査対象
    - {調査項目1}
    - {調査項目2}

    ## 使用ツール
    - {gh CLI / WebFetch / WebSearch}

    ## 出力先
    {output_file}（このファイルのみに出力せよ）

    ## 参照すべき既存情報
    - {既存KBファイル等}

    ## 除外キーワード
    - {除外対象}
  target_path: "{output_file}"
```

## Nablarch Knowledge Base

本スキルの調査効率化のため、Nablarchの基礎的なアーキテクチャ情報をインライン化する。Nablarch以外の技術を調査する場合はこのセクションをスキップせよ。

### 6種のアプリケーション種別

| # | アプリケーション種別 | 起動方式 | 最小ハンドラ数 | 主要用途 |
|---|-------------------|---------|-------------|---------|
| 1 | Webアプリケーション | `WebFrontController`（Servlet Filter） | 13 | サーバサイドレンダリングWebアプリ |
| 2 | RESTful Webサービス | `WebFrontController`（Servlet Filter） | 7 | REST API |
| 3 | Nablarchバッチ（都度起動） | `nablarch.fw.launcher.Main` | 9 | バッチ処理（オンデマンド） |
| 4 | Nablarchバッチ（常駐） | `nablarch.fw.launcher.Main` | 9+5 | バッチ処理（常駐デーモン） |
| 5 | MOMメッセージング | `nablarch.fw.launcher.Main` | 14 | MQ連携（同期応答/応答不要） |
| 6 | HTTPメッセージング | `WebFrontController`（Servlet Filter） | 11 | HTTP電文処理（非推奨、REST推奨） |

**補足**: Jakarta Batch（JSR352）準拠バッチ（jBeret使用）とテーブルキューメッセージングも存在するが、上記6種が主要アーキテクチャパターン。

### 主要ハンドラの分類

#### 共通ハンドラ

| ハンドラ名 | FQCN |
|-----------|------|
| グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` |
| スレッドコンテキスト変数管理 | `nablarch.common.handler.threadcontext.ThreadContextHandler` |
| スレッドコンテキスト変数削除 | `nablarch.common.handler.threadcontext.ThreadContextClearHandler` |
| DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` |
| トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` |
| リクエストディスパッチ | `nablarch.fw.handler.RequestPathJavaPackageMapping` |

#### バッチ/スタンドアローン専用

| ハンドラ名 | FQCN |
|-----------|------|
| ステータスコード変換 | `nablarch.fw.handler.StatusCodeConvertHandler` |
| マルチスレッド実行制御 | `nablarch.fw.handler.MultiThreadExecutionHandler` |
| トランザクションループ制御 | `nablarch.fw.handler.LoopHandler` |
| データリード | `nablarch.fw.handler.DataReadHandler` |
| プロセス常駐化 | `nablarch.fw.handler.ProcessResidentHandler` |
| プロセス停止制御 | `nablarch.fw.handler.BasicProcessStopHandler` |

#### Web専用（主要）

| ハンドラ名 | FQCN |
|-----------|------|
| HTTP文字エンコード制御 | `nablarch.fw.web.handler.HttpCharacterEncodingHandler` |
| HTTPレスポンス | `nablarch.fw.web.handler.HttpResponseHandler` |
| セキュア | `nablarch.fw.web.handler.SecureHandler` |
| セッション変数保存 | `nablarch.common.web.session.SessionStoreHandler` |
| CSRFトークン検証 | `nablarch.fw.web.handler.CsrfTokenVerificationHandler` |

#### REST専用

| ハンドラ名 | FQCN |
|-----------|------|
| JAX-RSレスポンス | `nablarch.fw.jaxrs.JaxRsResponseHandler` |
| ボディ変換 | `nablarch.fw.jaxrs.BodyConvertHandler` |
| BeanValidation | `nablarch.fw.jaxrs.JaxRsBeanValidationHandler` |

### ハンドラキュー順序制約

- **GlobalErrorHandler** は常にキュー先頭近くに配置
- **DbConnectionManagementHandler** は **TransactionManagementHandler** より前
- **DispatchHandler** はキュー末尾近く
- バッチでは **MultiThreadExecutionHandler** がメイン/サブスレッドの境界
- **ThreadContextClearHandler** は **ThreadContextHandler** より前

### 公式ドキュメントの主要URLリスト

#### アーキテクチャ
- トップ: https://nablarch.github.io/
- アーキテクチャ概要: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html
- 標準ハンドラ一覧: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html

#### アプリケーション種別
- Web: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html
- REST: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
- バッチ: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html
- MOMメッセージング: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/messaging/mom/architecture.html
- HTTPメッセージング: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/http_messaging/architecture.html

#### ライブラリ
- データベースアクセス: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database_management.html
- ユニバーサルDAO: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html

#### GitHub
- メイン: https://github.com/nablarch/nablarch
- Core: https://github.com/nablarch/nablarch-core
- Example-Web: https://github.com/nablarch/nablarch-example-web
- Example-REST: https://github.com/nablarch/nablarch-example-rest
- Example-Batch: https://github.com/nablarch/nablarch-example-batch

### 主要モジュール一覧（groupId: com.nablarch.framework）

| カテゴリ | artifactId | 責務 |
|---------|-----------|------|
| Core | `nablarch-core` | Handler, ExecutionContext, Interceptor, SystemRepository基盤 |
| Core | `nablarch-core-repository` | DIコンテナ、XMLコンポーネント定義 |
| Core | `nablarch-core-transaction` | トランザクション管理 |
| Core | `nablarch-core-jdbc` | JDBCユーティリティ |
| FW | `nablarch-fw` | 共通ハンドラ実装 |
| FW | `nablarch-fw-web` | Web基盤、HTTPハンドラ |
| FW | `nablarch-fw-jaxrs` | RESTful Webサービス |
| FW | `nablarch-fw-batch` | バッチ用ハンドラ/アクション |
| FW | `nablarch-fw-standalone` | スタンドアローンアプリ起動点 |
| Common | `nablarch-common-dao` | ユニバーサルDAO |
| Common | `nablarch-common-jdbc` | DB接続ハンドラ |

## Examples

### Example 1: Nablarch KB全体構築（6並列 — 実績ベース）

```
入力:
  technology_name: "Nablarch"
  mode: "comprehensive"
  domains:
    - { name: "公式ドキュメント", type: "docs", output_file: "output/nablarch_kb_official_docs.md", params: { url: "https://nablarch.github.io/docs/LATEST/doc/" } }
    - { name: "GitHub", type: "github", output_file: "output/nablarch_kb_github.md", params: { org: "nablarch" } }
    - { name: "開発標準", type: "standards", output_file: "output/nablarch_kb_dev_standards.md", params: { org: "nablarch-development-standards" } }
    - { name: "Fintan", type: "site", output_file: "output/nablarch_kb_fintan.md", params: { url: "https://fintan.jp/" } }
    - { name: "コミュニティ", type: "community", output_file: "output/nablarch_kb_community.md" }
    - { name: "アーキテクチャ", type: "architecture", output_file: "output/nablarch_kb_architecture.md" }
  parallelism: 6
  depth: "standard"
  exclude_keywords: ["Xenlon"]

実行結果:
  - 6エージェントが並列に6領域を調査
  - 合計数千行のKB構築（ファイル競合ゼロ）
  - アーキテクチャKBは全ハンドラのFQCN・XML設定例・処理フロー図を含む
```

### Example 2: Nablarchハンドラキュー深掘り（focused モード）

```
入力:
  technology_name: "Nablarch"
  mode: "focused"
  investigation_area: "ハンドラキュー"
  output_path: "output/nablarch_kb_handler_queue.md"
  depth: "thorough"
  exclude_keywords: ["Xenlon"]

Phase 2 — 公式ドキュメント調査:
  WebFetch（並列4件ずつ）:
    1. アーキテクチャ概要 → パイプライン処理モデル抽出
    2. Webアーキテクチャ → 13ハンドラ最小構成
    3. RESTアーキテクチャ → 7ハンドラ最小構成
    4. バッチアーキテクチャ → 9ハンドラ最小構成（都度起動+常駐）
    5. MOMメッセージング → 14ハンドラ最小構成
    6. 標準ハンドラ一覧 → 全FQCN・責務

Phase 3 — GitHub調査:
  サンプルプロジェクト3つのXML設定を並列取得:
    - nablarch-example-web/web-component-configuration.xml
    - nablarch-example-rest/rest-component-configuration.xml
    - nablarch-example-batch/batch-component-configuration.xml

Phase 4 — アーキテクチャ詳細:
  Handler.java, Interceptor.java, ExecutionContext.java のソース確認
  主要ハンドラ実装のGitHub検索・検証

出力: 約1200行のMarkdownドキュメント（7セクション構成）
```

### Example 3: Spring Boot KB構築（4並列）

```
入力:
  technology_name: "Spring Boot"
  mode: "comprehensive"
  domains:
    - { name: "公式ドキュメント", type: "docs", output_file: "output/springboot_kb_docs.md", params: { url: "https://docs.spring.io/spring-boot/reference/" } }
    - { name: "GitHub", type: "github", output_file: "output/springboot_kb_github.md", params: { org: "spring-projects", repo: "spring-boot" } }
    - { name: "エコシステム", type: "architecture", output_file: "output/springboot_kb_ecosystem.md" }
    - { name: "コミュニティ", type: "community", output_file: "output/springboot_kb_community.md" }
  parallelism: 4
  depth: "standard"

領域分割の考え方:
  - Spring Bootは開発標準が公式内にあるため独立領域にしない
  - エコシステム（Spring Cloud, Spring Security, Spring Data等）を独立領域として調査
  - 英語圏のコミュニティが圧倒的に大きいため英語検索重視
```

### Example 4: React KB構築（単独エージェント — 縮約分割）

```
入力:
  technology_name: "React"
  mode: "comprehensive"
  domains:
    - { name: "ドキュメント・ナレッジ", type: "docs", output_file: "output/react_kb_docs.md", params: { url: "https://react.dev/" } }
    - { name: "GitHub・エコシステム", type: "github", output_file: "output/react_kb_github.md", params: { owner: "facebook", repo: "react" } }
    - { name: "コミュニティ", type: "community", output_file: "output/react_kb_community.md" }
  parallelism: 1
  depth: "quick"

単独エージェント:
  - 3領域を順次調査
  - 各領域は別ファイルに出力（将来の並列化に備える）
```

## Guidelines

### 必須ルール

1. **全てのクラス名はFQCN（完全修飾クラス名）で記載すること**
   - `GlobalErrorHandler` (NG) → `nablarch.fw.handler.GlobalErrorHandler` (OK)
   - パッケージ名不明の場合はGitHub検索で確認
   - 確認できない場合は「【推測】」プレフィックスまたは「未確認」と記載
   - **推測でクラス名を記載してはならない**

2. **RACE-001を必ず回避すること**
   - 1ファイル = 1エージェント の原則を厳守
   - タスク割り当て時に出力ファイルパスを明示
   - 統合作業は全領域完了後に1エージェントが実施

3. **全ての情報にソースURL/GitHubリンクを付記すること**
   - 公式ドキュメント → `> **ソース**: [{ページ名}]({URL})`
   - GitHubソースコード → `> **ソースコード**: [{ファイル名}]({GitHub URL})`
   - テーブル内: URLカラムにリンクを記載
   - URLが不明な情報は「ソース: 不明」と記載

4. **推測と事実を明確に区別すること**
   - ソースコードまたは公式ドキュメントで確認した情報 → そのまま記載
   - 推測した情報 → 「【推測】」プレフィックスを付記

5. **設定例は実プロジェクトから引用すること**
   - 自作の設定例ではなく、example-* リポジトリの実ファイルを引用
   - 引用元のGitHubリンクを付記

6. **並列実行を最大限活用すること**
   - 独立したWebFetchは4件まで並列実行
   - 独立したgh api呼び出しは並列実行可能
   - WebSearchも並列実行可能

7. **除外キーワードを遵守すること**
   - 指定された除外対象の情報は収集・記載しない
   - 収集済みデータに含まれていた場合は出力時に除外

8. **アーカイブ済みリポジトリも記録すること**
   - `isArchived: true` のリポジトリもリストに含める（歴史的経緯の理解に有用）

### 検索・調査のコツ（実践知見）

1. **公式ドキュメントが日本語のみの場合がある**
   - 日本語プロンプトでWebFetchすると正確に情報抽出できる
   - 英語版がある場合でも全ページ揃っていないことがある

2. **GitHubのブランチ名に注意**
   - 古いリポジトリは `master`、新しいリポジトリは `main`
   - `gh api repos/{org}/{repo} --jq '.default_branch'` で確認

3. **gh api search/code はレート制限がある**
   - 1分間に10リクエストまで
   - 大量のFQCN検証時は間隔を空けて実行

4. **パイプライン/ミドルウェアの順序制約は分散記載されている**
   - 一括で確認できるページがないことが多い
   - 各コンポーネントの「制約事項」セクションを個別確認する必要がある

5. **「見つからなかった」も重要な情報として記録する**
   - 「英語ドキュメントが存在しない」→ グローバル展開の限界
   - 「コミュニティ記事が少ない」→ ニッチ技術の指標

6. **gh CLI のフィールド名に注意**
   - `stargazerCount`（正）/ `stargazersCount`（誤）
   - 実績あるエラーなので注意

### ツール使い分けの指針

```
【gh CLI を使う場合】
- リポジトリ一覧の取得（gh repo list）
- リポジトリ詳細の取得（gh repo view）
- API経由のメトリクス取得（gh api）
- ソースコード取得（gh api repos/.../contents）
- リリースタグの取得
- コントリビューター情報の取得

【WebFetch を使う場合】
- 公式ドキュメントページの内容取得
- ナレッジサイト（Fintan等）のページ取得
- GitHubのファイル内容表示（UIページ）
- ブログ記事の詳細取得

【WebSearch を使う場合】
- site: 演算子での特定サイト検索
- 技術名 + キーワードでの一般検索
- コミュニティ記事の発見
- 開発標準リポジトリの発見（別orgの場合）
```

### アンチパターン（避けるべきこと）

- **FQCNの推測**: パッケージ名不明のクラスを推測で記載 → GitHub検索で確認必須
- **同一ファイルへの並列書き込み**: RACE-001の原因。1ファイル=1エージェント厳守
- **公式ドキュメントのみへの依存**: ドキュメントは最新でない場合がある → GitHubソースで補完
- **設定例の自作**: 存在しない設定を作成 → 実プロジェクトから引用
- **コンポーネント順序の推測**: 順序制約はドキュメント/ソースから確認 → 推測で並べない
- **URLの推測**: パターンからの推測は404リスク高 → 存在確認してから記載
- **逐次的なWebFetch**: 独立ページを1つずつ取得 → 4件まで並列実行
- **レート制限の無視**: エラーを無視して連続リクエスト → アカウント制限のリスク
- **過剰な分割**: 8領域以上に分割すると管理コストが成果を上回る
- **目次なしの長大ファイル**: 500行以上のファイルには必ず目次をつける
