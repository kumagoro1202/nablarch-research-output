---
name: nablarch-knowledge-builder
description: Nablarchフレームワークの知識ファイル（YAMLベース）を構築・拡充するスキル。MCP Serverが参照する構造化知識データ（ハンドラカタログ、API パターン、設計パターン、エラーカタログ等）をYAML形式で体系的に構築し、既存知識ファイルの拡充・新規カテゴリの追加・他フレームワークへの展開を行う。「Nablarchの○○知識をYAMLにまとめて」「知識ベースに新しいカテゴリを追加して」「Spring向けの知識YAMLを構築して」といった要望に対応する。
---

# Nablarch Knowledge Builder

## Overview

Nablarchフレームワーク（および他のJavaフレームワーク）の技術知識を、MCP Serverが参照可能な構造化YAMLファイルとして構築・拡充するスキル。

nablarch-mcp-serverでは、以下の7種のYAML知識ファイルがMCP Resource/Promptの基盤データとして使用されている：

| ファイル名 | 行数 | 用途 |
|---|---|---|
| handler-catalog.yaml | ~516行 | Web/REST/Batch/Messaging各アプリタイプのハンドラ仕様・順序定義 |
| api-patterns.yaml | ~607行 | アクションクラスのAPIパターン（Web/REST/Batch/Messaging別） |
| handler-constraints.yaml | ~232行 | ハンドラ間の順序制約（must_before/must_after） |
| config-templates.yaml | ~399行 | アプリタイプ別XMLテンプレート |
| design-patterns.yaml | ~507行 | Nablarch固有設計パターン（architecture/action/validation/database/testing） |
| error-catalog.yaml | ~250行 | よくあるエラーと解決策（handler/config カテゴリ） |
| module-catalog.yaml | ~310行 | モジュール一覧・主要クラス・依存関係 |

本スキルは、これらの知識ファイルを**新規構築**する手法と、既存ファイルを**拡充**する手法の両方を体系化する。さらに、同じYAMLスキーマパターンをSpring Boot、Jakarta EE、Quarkus等の他フレームワークに**展開**するためのガイドラインも提供する。

**本スキルの特長:**
- 7種の実績あるYAMLスキーマ定義を再利用可能なテンプレートとして提供
- 公式ドキュメント・ソースコード・サンプルプロジェクトからのデータ収集パイプライン
- 構造化・正規化・検証の3段階品質保証プロセス
- MCP Server統合テストによる知識ファイルの動作検証
- 他フレームワークへの横展開パターン

**用途:**
- MCP Serverの新規知識カテゴリ追加
- 既存知識ファイルのデータ拡充（新ハンドラ、新パターン等の追加）
- Nablarchバージョンアップに伴う知識ファイル更新
- Spring Boot / Jakarta EE / Quarkus 等向けの知識YAML構築
- フレームワーク横断ナレッジベースの統一スキーマ設計

**実績:**
- nablarch-mcp-server Phase 1で7ファイル・計2,821行の知識ベースを構築
- MCP Resource 12種・Prompt 6種が全てこの知識ファイルを参照して動作

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「Nablarchの○○をYAML知識ファイルにまとめて」
- 「MCP Serverの知識ベースに新カテゴリを追加して」
- 「handler-catalog.yamlに新しいハンドラを追加して」
- 「Spring Boot版の知識YAMLを構築して」
- 「既存の知識ファイルをNablarch 6u3対応に更新して」
- 「フレームワークの設計パターンをYAML形式で構造化して」
- MCP ServerのResource/Promptが参照する知識データを新規作成・更新する必要がある場合
- 特定のフレームワークの技術知識を機械可読な構造化形式で管理したい場合

**トリガーキーワード**: 知識YAML, ナレッジベース構築, knowledge file, YAML知識, MCP知識データ, ハンドラカタログ, パターンカタログ

## Instructions

### Phase 1: スキーマ設計（YAML構造の定義）

構築する知識ファイルのYAMLスキーマを設計する。

#### Step 1.1: 対象カテゴリの特定

```
【カテゴリ分析】

1. 構築対象の知識カテゴリを特定する
   - 新規カテゴリの場合: 既存7カテゴリとの重複がないか確認
   - 既存カテゴリの拡充の場合: 現行スキーマを読み込む

2. 対象フレームワークを確認する
   - Nablarch: 既存スキーマを流用
   - Spring Boot: Nablarchスキーマを基に適応
   - Jakarta EE: サーブレット/EJB等のパラダイムに適応
   - Quarkus: CDI/ネイティブコンパイル等の特性を考慮

3. 知識ファイルの利用者を確認する
   - MCP Resource（読み取り専用、Markdown変換して提供）
   - MCP Prompt（引数に応じてフィルタリング・整形して提供）
   - MCP Tool（検索・バリデーション等で参照）
```

#### Step 1.2: YAMLスキーマの定義

```
【スキーマ設計テンプレート】

既存7ファイルの共通パターン:

1. ファイルヘッダ（コメント形式）:
   # {Title}
   # {Description}
   # Version: {version} ({phase} Knowledge Base)
   # Reference: {url}

2. トップレベルキー構造パターン:

   パターンA: アプリタイプ別分類（handler-catalog, config-templates）
   ---
   web:
     description: "..."
     handlers:  # or templates
       - name: XxxHandler
         fqcn: "full.qualified.ClassName"
         description: "..."
         # カテゴリ固有フィールド

   パターンB: フラットリスト（api-patterns, design-patterns, error-catalog）
   ---
   patterns:  # or errors
     - name: pattern-name
       category: category-name
       description: "..."
       # カテゴリ固有フィールド

   パターンC: 制約定義（handler-constraints）
   ---
   constraints:
     - handler: HandlerName
       must_before:
         - OtherHandler
       must_after:
         - AnotherHandler
       reason: "..."

   パターンD: モジュール定義（module-catalog）
   ---
   modules:
     - name: module-name
       artifact_id: "nablarch-xxx"
       description: "..."
       key_classes:
         - "full.qualified.ClassName"
       dependencies:
         - "other-module"

3. フィールド設計原則:
   - name: 一意識別子（kebab-case or PascalCase）
   - fqcn: 完全修飾クラス名（ソースコードで検証済みのもののみ）
   - description: 日本語の簡潔な説明
   - category: 分類用カテゴリ（フィルタリングに使用）
   - code_example: 実際のコード例（|ブロックスカラー形式）
```

#### Step 1.3: スキーマの検証基準定義

```
【検証チェックリスト】

□ 全エントリにname フィールドがあること
□ FQCNが実在するクラスであること（GitHubソースで確認）
□ description が日本語で記述されていること
□ category値が事前定義されたリストに含まれること
□ code_example がコンパイル可能な構文であること
□ YAMLとしてパース可能であること（Jackson ObjectMapper）
□ 既存の知識ファイルとキー名・構造が整合していること
```

### Phase 2: データ収集（情報源の調査と抽出）

YAMLに格納するデータを複数の情報源から収集する。

#### Step 2.1: 公式ドキュメントからの収集

```
【公式ドキュメント調査】

1. Nablarch公式ドキュメントサイト
   WebFetch: https://nablarch.github.io/docs/LATEST/doc/
   - ハンドラ一覧、設定例、API仕様を収集
   - 日本語ドキュメントを優先

2. Javadoc
   WebFetch: https://nablarch.github.io/docs/LATEST/javadoc/
   - FQCN、メソッドシグネチャ、パラメータ説明を収集

3. リリースノート
   - バージョン別の変更点、非推奨API、新機能を収集
```

#### Step 2.2: ソースコードからの収集

```
【GitHubソースコード調査】

1. Nablarch本体リポジトリ
   Bash: gh api repos/nablarch/{module}/contents/src/main/java/...
   - Handlerインターフェース実装クラスの列挙
   - @Published アノテーション付きクラスの特定
   - 設定パラメータの抽出（getter/setter分析）

2. サンプルプロジェクト
   - nablarch-example-web, nablarch-example-rest 等
   - 実際のXML設定ファイルからハンドラキュー構成を抽出
   - テストコードからAPIの使用パターンを抽出

3. FQCN検証
   - 全てのFQCNがソースコードに存在することを確認
   - 推測によるFQCNは絶対に記載しない
```

#### Step 2.3: コミュニティ知識からの収集

```
【コミュニティ調査】

1. Fintan（NTTデータ技術ブログ）
   WebSearch: "site:fintan.jp Nablarch {対象トピック}"
   - ベストプラクティス、Tips、トラブルシューティングを収集

2. Stack Overflow / Qiita / Zenn
   WebSearch: "Nablarch {対象トピック} site:qiita.com OR site:zenn.dev"
   - 実務での利用パターン、よくある問題を収集

3. GitHub Issues
   Bash: gh issue list -R nablarch/{module} --label bug
   - よくある問題とワークアラウンドを収集
```

### Phase 3: 構造化（データの正規化とYAML化）

収集したデータをYAMLスキーマに従って構造化する。

#### Step 3.1: データの正規化

```
【正規化ルール】

1. ハンドラ名:
   - PascalCase（例: HttpCharacterEncodingHandler）
   - "Handler" サフィックスを含める
   - 省略形は使用しない

2. FQCN:
   - ピリオド区切りの完全修飾名
   - パッケージ名は全て小文字
   - ソースコードで検証済みのもののみ

3. 説明文:
   - 日本語で記述
   - 1文で簡潔に（50文字以内推奨）
   - 「○○を△△する」の形式

4. コード例:
   - インポート文は省略可（FQCNで特定可能なため）
   - コンパイル可能な構文であること
   - 実プロジェクトから抽出した例を優先

5. 順序制約:
   - must_before: このハンドラが前に来るべき対象
   - must_after: このハンドラが後に来るべき対象
   - reason: 制約の理由（日本語）
```

#### Step 3.2: YAML生成

```
【YAML生成手順】

1. ヘッダコメントを記述
   # {Title}
   # {Description}
   # Version: {version} ({phase} Knowledge Base)
   # Reference: {reference_url}

2. スキーマに従ってデータを配置
   - パターンA/B/C/Dのいずれかを選択
   - ブロックスカラー（|）はコード例・長い説明に使用
   - フロースカラーは短い値に使用

3. 空行でセクションを区切り、可読性を確保
   - トップレベルキーの間に1空行
   - リスト要素の間に1空行（要素が複数フィールドを持つ場合）

4. YAMLリントチェック
   - インデント: スペース2文字
   - 文字列値: ダブルクォートで囲む
   - ブーリアン: true/false（yes/noは使用しない）
```

### Phase 4: 検証（品質保証とMCP統合テスト）

生成したYAMLファイルの品質を検証する。

#### Step 4.1: 構文検証

```
【構文チェック】

1. YAMLパース検証
   - Jackson ObjectMapper でパース可能であること
   - エンコーディング: UTF-8
   - BOMなし

2. スキーマ整合性
   - 必須フィールドが全エントリに存在すること
   - フィールド名が既存ファイルと統一されていること
   - データ型が想定通りであること（文字列、数値、リスト等）

3. 参照整合性
   - handler-constraints.yaml のハンドラ名が handler-catalog.yaml に存在すること
   - module-catalog.yaml の dependencies が実在するモジュールを参照すること
   - config-templates.yaml のアプリタイプが handler-catalog.yaml のキーと一致すること
```

#### Step 4.2: 内容検証

```
【内容チェック】

1. FQCN検証
   - 全FQCNがGitHubリポジトリのソースコードに存在すること
   - deprecated クラスにはその旨を annotation フィールドに記載

2. 重複チェック
   - 同一ファイル内でname値が重複していないこと
   - 異なるファイル間で矛盾する情報がないこと

3. 網羅性チェック
   - 公式ドキュメントに記載されたハンドラが漏れなくカタログに含まれること
   - 主要な設計パターンが design-patterns.yaml に網羅されていること
```

#### Step 4.3: MCP統合テスト

```
【統合テスト】

1. Resource提供テスト
   - HandlerResourceProvider / GuideResourceProvider が知識ファイルを正しく読み込めること
   - Markdown変換後の出力が期待形式であること

2. Prompt実行テスト
   - 各Promptクラスが知識ファイルを参照して正しい結果を返すこと
   - 引数フィルタリング（app_type, handler_name等）が正しく動作すること

3. Tool実行テスト
   - SearchApiToolが知識ファイルを検索できること
   - ValidateHandlerQueueToolが制約検証に知識ファイルを活用できること

4. ビルドテスト
   ./gradlew clean build
   - 知識ファイルの変更がビルドエラーを引き起こさないこと
```

### Phase 5: 他フレームワークへの展開

同じスキーマパターンをNablarch以外のフレームワークに適用する。

#### Step 5.1: スキーマ適応

```
【フレームワーク別適応ガイド】

1. Spring Boot 展開:
   handler-catalog.yaml → filter-catalog.yaml（Servlet Filter相当）
   - HandlerQueue → FilterChain / Interceptor
   - XML設定 → @Configuration / application.yml
   - handler_name → filter_name / interceptor_name

   api-patterns.yaml → controller-patterns.yaml
   - Action → @Controller / @RestController
   - HttpRequest → HttpServletRequest / @RequestParam

   design-patterns.yaml → spring-patterns.yaml
   - handler-queue-pattern → filter-chain-pattern
   - 追加: DI, AOP, Boot Auto-configuration パターン

2. Jakarta EE 展開:
   handler-catalog.yaml → servlet-filter-catalog.yaml
   - HandlerQueue → FilterChain + Servlet
   - EJBインターセプタ も含める

   design-patterns.yaml → jakartaee-patterns.yaml
   - CDI, JPA, Bean Validation パターン

3. Quarkus 展開:
   handler-catalog.yaml → extension-catalog.yaml
   - Quarkus Extension / CDI Interceptor
   - ネイティブコンパイル対応情報を追加

4. 共通注意点:
   - FQCN は対象フレームワークの実クラスで検証すること
   - Nablarch固有の概念は適切に翻訳すること
   - 対象フレームワークの慣例（命名規則等）に従うこと
```

## Input Format

```yaml
# 知識ファイル構築リクエスト
target_framework: "nablarch"          # nablarch | spring-boot | jakarta-ee | quarkus
category: "handler-catalog"           # 構築するカテゴリ
mode: "create"                        # create（新規）| extend（拡充）| migrate（他FW展開）
app_types:                            # 対象アプリタイプ（オプション）
  - web
  - rest
  - batch
  - messaging
source_version: "6u2"                 # 対象フレームワークバージョン
output_dir: "src/main/resources/knowledge/"  # 出力先ディレクトリ
reference_files:                      # 参照する既存知識ファイル（拡充時）
  - "handler-catalog.yaml"
  - "handler-constraints.yaml"
```

## Output Format

```yaml
# 出力ファイル: {output_dir}/{category}.yaml
# ファイル形式: UTF-8, BOMなし, インデント2スペース

# {Title}
# {Description}
# Version: {version} ({phase} Knowledge Base)
# Reference: {reference_url}

# --- 以下、カテゴリに応じたスキーマのYAMLデータ ---
```

**出力される成果物:**
1. **YAMLファイル**: 1カテゴリにつき1ファイル（既存の7ファイルパターンに従う）
2. **検証レポート**: 構文検証・内容検証・参照整合性の結果サマリ
3. **変更ログ**: 拡充モードの場合、追加・変更・削除されたエントリの一覧

## Examples

### Example 1: Nablarch handler-catalog.yaml の拡充

```
【入力】
target_framework: nablarch
category: handler-catalog
mode: extend
app_types: [web]
source_version: "6u2"

【実行フロー】
Phase 1: 既存handler-catalog.yamlを読み込み、webセクションのスキーマを把握
Phase 2: Nablarch 6u2の公式ドキュメントから新規追加ハンドラを調査
         GitHubソースから新ハンドラのFQCN・パラメータを取得
Phase 3: 既存エントリのフォーマットに合わせてYAMLエントリを追記
Phase 4: FQCNの実在確認、handler-constraints.yamlとの整合性チェック
         ./gradlew clean build でビルド通過を確認

【出力】
- handler-catalog.yaml（webセクションに新エントリ追加済み）
- handler-constraints.yaml（新ハンドラの順序制約を追加）
- 検証レポート: 全チェックPASS
```

### Example 2: Spring Boot向けfilter-catalog.yaml の新規構築

```
【入力】
target_framework: spring-boot
category: filter-catalog
mode: migrate
source_version: "3.4"

【実行フロー】
Phase 1: Nablarchのhandler-catalog.yamlスキーマをベースにSpring Boot用スキーマを設計
         HandlerQueue概念 → FilterChain + Interceptor に適応
Phase 2: Spring Boot公式ドキュメントからFilter/Interceptor一覧を収集
         spring-boot-starter-webのソースからFQCNを取得
Phase 3: Spring Boot命名規則に合わせてYAMLを生成
         @Order アノテーション情報を order フィールドにマッピング
Phase 4: FQCNの検証、YAMLパース検証
Phase 5: NablarchとSpring Bootのマッピング表を作成

【出力】
- filter-catalog.yaml（Spring Boot用、新規作成）
- interceptor-catalog.yaml（Spring MVC Interceptor用、新規作成）
- framework-mapping.md（Nablarch ↔ Spring Bootの概念対応表）
```

### Example 3: エラーカタログの新規カテゴリ追加

```
【入力】
target_framework: nablarch
category: error-catalog
mode: extend
app_types: [batch, messaging]

【実行フロー】
Phase 1: 既存error-catalog.yamlを読み込み（handler/configカテゴリが存在）
         新カテゴリ "batch" と "messaging" を追加するスキーマ設計
Phase 2: Nablarchバッチ・メッセージング関連のGitHub Issuesを調査
         Fintanのトラブルシューティング記事を収集
         サンプルプロジェクトのログから典型的エラーを抽出
Phase 3: 既存カテゴリのフォーマットに合わせて新エントリを生成
Phase 4: エラーコード・メッセージの一意性チェック、解決策の整合性検証

【出力】
- error-catalog.yaml（batch/messagingカテゴリ追加済み、約100行追加）
- 検証レポート: 全チェックPASS
```

## Guidelines

### 必須ルール

1. **FQCNは実在確認必須**
   - GitHubリポジトリのソースコードでクラスの存在を確認すること
   - 推測によるFQCNは絶対に記載してはならない
   - パッケージ名が不明な場合は `gh api search/code?q={ClassName}+repo:nablarch/{module}` で検索

2. **既存スキーマとの整合性を保つこと**
   - 既存7ファイルのフィールド名・命名規則・データ型を踏襲する
   - 独自のフィールド追加は最小限にし、追加する場合はコメントで理由を記載
   - インデント・空行・クォーティングスタイルを統一する

3. **日本語で記述すること**
   - description、reason 等の説明フィールドは日本語で記述
   - ファイルヘッダのコメントは英語（既存ファイルの慣例に従う）
   - code_example はJavaコード（自然言語部分は日本語コメント）

4. **バージョン管理を行うこと**
   - ファイルヘッダのVersion フィールドを更新
   - 拡充時は変更内容をコミットメッセージに明記
   - 破壊的変更（フィールド削除・名称変更）は別ブランチで行う

5. **参照整合性を維持すること**
   - handler-constraints.yaml のハンドラ名は handler-catalog.yaml に存在すること
   - module-catalog.yaml の dependencies は実在するモジュールを指すこと
   - config-templates.yaml のアプリタイプは handler-catalog.yaml のキーと一致すること

6. **YAMLの構文規則を厳守すること**
   - インデント: スペース2文字（タブ禁止）
   - 文字列値: ダブルクォートで囲む
   - ブーリアン: true/false（yes/no禁止）
   - エンコーディング: UTF-8、BOMなし
   - 末尾の改行: 1つ

### 効率化のコツ

1. **既存ファイルのコピー&修正**
   - 新規カテゴリ作成時は、最も構造が近い既存ファイルをコピーして修正する
   - スキーマ設計を一から行うより、実績あるパターンの再利用が効率的

2. **GitHubソースの一括取得**
   - `gh api repos/nablarch/{module}/git/trees/master?recursive=1` でファイル一覧を取得
   - FQCNの一括検証に活用

3. **段階的な構築**
   - まずコアエントリ（必須ハンドラ等）を記述し、ビルド通過を確認
   - その後、オプショナルなエントリを段階的に追加
   - 一度に大量のエントリを追加しない（レビュー困難になるため）

4. **他フレームワーク展開時のマッピング表**
   - 最初にNablarch概念→対象FW概念のマッピング表を作成
   - マッピング表に基づいて機械的にスキーマを変換

5. **並列データ収集**
   - 公式ドキュメント・ソースコード・コミュニティの3系統を並列に調査
   - 独立した情報源からのデータ収集はTask toolで並列実行

### アンチパターン（避けるべきこと）

1. **FQCNの推測**
   - パッケージ名が不明なクラスを推測で記載する
   - 対策: 必ずGitHubソースで存在を確認してから記載

2. **独自スキーマの乱立**
   - 既存7ファイルと異なる独自のフィールド名・構造を採用する
   - 対策: 既存ファイルのパターン（A/B/C/D）のいずれかに合わせる

3. **説明文の英語記述**
   - descriptionやreasonを英語で記述する（既存ファイルは日本語）
   - 対策: 説明文は全て日本語。ヘッダコメントのみ英語

4. **検証なしの大量追加**
   - 100エントリ以上を一度に追加し、検証を省略する
   - 対策: 10-20エントリごとにビルドテストを実行

5. **参照整合性の無視**
   - handler-constraints.yamlに存在しないハンドラ名を記載する
   - 対策: 知識ファイル間の相互参照を必ずチェック

6. **バージョン情報の未更新**
   - ファイルを更新してもVersionフィールドを変更しない
   - 対策: 変更時は必ずVersionを更新し、コミットメッセージに含める

7. **コード例の自作**
   - 動作確認していないコード例を記載する
   - 対策: サンプルプロジェクトや公式ドキュメントから引用。自作する場合はコンパイル確認必須

8. **単一情報源への依存**
   - 公式ドキュメントのみから情報を収集する（ドキュメントは最新でない場合がある）
   - 対策: ソースコード、サンプルプロジェクト、コミュニティの3系統で交差検証
