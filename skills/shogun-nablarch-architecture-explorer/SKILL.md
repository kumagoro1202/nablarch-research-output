---
name: nablarch-architecture-explorer
description: Nablarchフレームワークの特定のアーキテクチャ領域（ハンドラキュー、DB接続・トランザクション、インターセプタ、コンポーネント定義XML、リクエスト処理フロー、モジュール構成等）について、公式ドキュメント・GitHubソースコード・Javadocから技術情報を収集し、正確で詳細なナレッジベースドキュメントを生成する。「Nablarchのハンドラキュー構成を調べて」「Nablarchのトランザクション管理を調査して」「Nablarchのインターセプタの仕組みを調べて」「NablarchのXML設定を体系的にまとめて」「Nablarchのモジュール構成を調査して」といった要望に対応する。WebFetch + WebSearch + GitHub CLI を駆使した多段階アーキテクチャ調査スキル。
---

# Nablarch Architecture Explorer

## Overview

Nablarch（ナブラーク）フレームワークの特定のアーキテクチャ領域について、**公式ドキュメント**・**GitHubソースコード**・**Javadoc**・**サンプルプロジェクト**の4つの情報源から技術情報を収集し、FQCN（完全修飾クラス名）を含む正確で詳細なMarkdownナレッジベースドキュメントを生成するスキル。

Nablarchはハンドラキューアーキテクチャ（Chain of Responsibilityパターン）を核心とする独自設計のJavaフレームワークであり、一般的なSpring等とは大きく異なる構造を持つ。本スキルはNablarch固有のアーキテクチャ知識を正確に調査・体系化するために設計されている。

**主な用途:**
- Nablarchプロジェクトの技術調査・設計情報の収集
- ハンドラキュー構成パターンの調査・比較
- XML設定の構造・実例の収集
- MCP（Model Context Protocol）サーバ等のツール開発に必要な基礎情報の収集
- RAGナレッジベースのアーキテクチャ情報パート

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「Nablarchのハンドラキュー構成を調べて」
- 「Nablarchの○○（アーキテクチャ領域）を調査して」
- 「Nablarchのトランザクション管理の仕組みを詳しく調査して」
- 「NablarchのXML設定（コンポーネント定義）を体系的にまとめて」
- 「Nablarchのインターセプタの仕組みをソースコードから調べて」
- 「Nablarchのモジュール構成とMavenアーティファクトの依存関係を調べて」
- 「Nablarchの特定のアプリケーション種別（Web/REST/Batch等）のアーキテクチャを調査して」
- Nablarchのアーキテクチャ情報を正確なFQCN・XML設定例付きでドキュメント化する必要がある場合

**トリガーキーワード**: Nablarchアーキテクチャ, ハンドラキュー調査, Nablarch設定調査, Nablarchモジュール構成, NablarchDB管理, Nablarchインターセプタ, Nablarch XML設定

## Instructions

### Phase 1: 公式ドキュメント調査

対象領域に関するNablarch公式ドキュメントから情報を収集する。

#### Step 1.1: 対象領域のドキュメントページ特定

```
【実行手順】

1. 「Nablarch Knowledge Base」セクションの主要URLリストから対象領域のページを特定
   - ハンドラキュー → アーキテクチャ概要 + 各アプリ種別のアーキテクチャページ
   - DB/トランザクション → データベースアクセスページ
   - インターセプタ → Javadoc + ソースコード
   - XML設定 → システムリポジトリページ
   - モジュール構成 → Mavenモジュール構成ページ + GitHub

2. WebSearch で補完検索
   WebSearch: "site:nablarch.github.io {対象領域キーワード}"
   例: "site:nablarch.github.io handler queue"
   例: "site:nablarch.github.io transaction management"

3. アプリ種別フィルタが指定されている場合
   - 対象アプリ種別のアーキテクチャページに絞る
   - 「Nablarch Knowledge Base」セクションの6種アプリ種別一覧を参照
```

#### Step 1.2: WebFetchによるドキュメント内容取得

```
【実行手順 — 並列実行推奨】

1. 各ページをWebFetchで取得
   prompt例: "この技術ドキュメントの内容を全て抽出せよ。
   ハンドラ名、FQCN（完全修飾クラス名）、XML設定例、制約事項、
   処理フローを網羅的に記録。日本語で出力。"

2. 抽出すべき情報:
   - ハンドラ/クラスの完全修飾クラス名（FQCN）
   - ハンドラキューの構成例（順序付き）
   - XML設定の具体例
   - 各ハンドラの責務と制約事項
   - ハンドラ間の順序依存関係

3. 深度による取得方針:
   - overview: 対象領域のトップページのみ（最大5ページ）
   - detailed: 関連する全サブページ含む（最大15ページ）
   - exhaustive: 隣接領域のページも含む（ページ数制限なし）
```

#### Step 1.3: ハンドラキュー構成図の抽出（ハンドラキュー調査時）

```
【ハンドラキュー調査の場合の追加手順】

1. 各アプリケーション種別のアーキテクチャページから最小構成ハンドラキューを取得
   - Web: 13ハンドラ最小構成
   - REST: 7ハンドラ最小構成
   - Nablarchバッチ: 9ハンドラ最小構成
   - MOMメッセージング: 14ハンドラ最小構成
   - HTTPメッセージング: 11ハンドラ最小構成

2. 各ハンドラについてFQCN・責務・順序制約を記録
   - 順序が前後すると動作しないハンドラの組み合わせを特定
   - メインスレッド用/サブスレッド用の区別を記録（バッチの場合）
```

### Phase 2: GitHubソースコード調査

Nablarch organizationのGitHubリポジトリから、ソースコードレベルの正確な情報を収集する。

#### Step 2.1: 対象リポジトリの特定

```
【実行手順】

1. 対象領域に関連するリポジトリを特定
   - 「Nablarch Knowledge Base」セクションのモジュール一覧を参照
   - GitHub organizationの全リポジトリ: gh api orgs/nablarch/repos --paginate

2. 主要リポジトリのマッピング:
   | 領域 | リポジトリ |
   |------|----------|
   | Handler基盤 | nablarch-core, nablarch-fw |
   | Web | nablarch-fw-web |
   | REST | nablarch-fw-jaxrs |
   | バッチ | nablarch-fw-batch, nablarch-fw-standalone |
   | メッセージング | nablarch-fw-messaging, nablarch-fw-messaging-mom |
   | DB/トランザクション | nablarch-core-jdbc, nablarch-core-transaction, nablarch-common-jdbc |
   | DI/リポジトリ | nablarch-core-repository |
   | DAO | nablarch-common-dao |

3. gh CLI でソースコード取得
   gh api repos/nablarch/{repo}/contents/src/main/java/{path} --jq '.content' | base64 -d

   または WebFetch で GitHub UI から取得:
   WebFetch URL: "https://github.com/nablarch/{repo}/blob/master/src/main/java/{path}"
```

#### Step 2.2: インターフェース・クラス定義の確認

```
【確認すべきソースコード（領域に応じて選択）】

ハンドラキュー関連:
- nablarch.fw.Handler (nablarch-core)
- nablarch.fw.InboundHandleable (nablarch-core)
- nablarch.fw.OutboundHandleable (nablarch-core)
- nablarch.fw.HandlerQueueManager (nablarch-core)
- nablarch.fw.ExecutionContext (nablarch-core)

インターセプタ関連:
- nablarch.fw.Interceptor (nablarch-core)
- nablarch.fw.Interceptor.Factory (nablarch-core)
- nablarch.fw.Interceptor.Impl (nablarch-core)

DB/トランザクション関連:
- nablarch.core.db.connection.ConnectionFactorySupport (nablarch-core-jdbc)
- nablarch.core.transaction.TransactionFactory (nablarch-core-transaction)
- nablarch.common.handler.DbConnectionManagementHandler (nablarch-common-jdbc)
- nablarch.common.handler.TransactionManagementHandler (nablarch-common-jdbc)

XML設定関連:
- nablarch.core.repository.SystemRepository (nablarch-core-repository)
- nablarch.core.repository.di.DiContainer (nablarch-core-repository)

【確認項目】
- publicメソッドのシグネチャ
- 型パラメータ
- Javadocコメント
- アノテーション定義
```

#### Step 2.3: 実装クラスの調査

```
【実行手順 — 深度 detailed 以上】

1. インターフェースの主要実装クラスを列挙
   gh api search/code --method GET \
     -f q="implements Handler org:nablarch language:java" \
     --jq '.items[] | {name: .name, repo: .repository.full_name, path: .path}'

2. 各実装クラスについて以下を確認:
   - handleメソッドの実装内容
   - InboundHandleable/OutboundHandleableの実装有無
   - コンストラクタの引数（プロパティインジェクション対象）
   - 依存するコンポーネント
```

### Phase 3: サンプルプロジェクト分析

nablarch-example-* リポジトリのXML設定ファイルを分析し、実プロジェクトの構成パターンを抽出する。

#### Step 3.1: サンプルプロジェクトのXML取得

```
【対象サンプルプロジェクト】
- nablarch-example-web    — Webアプリケーション
- nablarch-example-rest   — RESTful Webサービス
- nablarch-example-batch  — バッチアプリケーション

【XML設定ファイルの取得 — 並列実行推奨】

1. メイン設定ファイルの取得
   WebFetch: "https://github.com/nablarch/nablarch-example-web/blob/main/src/main/resources/web-component-configuration.xml"
   WebFetch: "https://github.com/nablarch/nablarch-example-rest/blob/main/src/main/resources/rest-component-configuration.xml"
   WebFetch: "https://github.com/nablarch/nablarch-example-batch/blob/main/src/main/resources/batch-component-configuration.xml"

2. インポートされているXMLファイルも追跡取得
   - 各設定ファイルの<import>タグからインポート先を特定
   - ハンドラキュー定義、DB設定、インターセプタ順序定義を優先取得

【抽出項目】
- handlerQueue リストの全要素（順序付き）
- 各handlerの設定プロパティ
- import構造（ファイル間の依存関係）
- config-file参照（プロパティファイル）
```

#### Step 3.2: 設定パターンの比較分析

```
【実行手順】

1. アプリ種別間のハンドラキュー構成を比較
   - 共通ハンドラ: 全種別に存在するハンドラ
   - 種別固有ハンドラ: 特定種別にのみ存在するハンドラ
   - ハンドラ順序: 共通ハンドラの相対的な順序が種別間で一致するか

2. 最小構成（公式ドキュメント）と実プロジェクト構成の差分
   - 追加されているハンドラ（ログ、セキュリティ、カスタム等）
   - プロパティ設定の違い

3. カスタムハンドラのパターン
   - サンプルプロジェクトに含まれるカスタムハンドラの実装
   - handlerQueueへの挿入位置の傾向
```

### Phase 4: 体系化

収集した情報をカテゴリ別に整理し、構造化されたMarkdownドキュメントを生成する。

#### Step 4.1: カテゴリ分類

```
【調査領域別の標準カテゴリ】

ハンドラキュー調査の場合:
1. ハンドラキューの設計パターン（アプリ種別ごとの最小構成）
2. 全ハンドラ一覧と仕様（FQCN付きテーブル）
3. ハンドラ間の順序制約
4. カスタムハンドラの作成方法
5. 実プロジェクトの構成例

DB/トランザクション調査の場合:
1. DB接続プール設定（DataSource/JNDI）
2. トランザクション境界の制御
3. 複数DB接続の設定
4. 個別トランザクション
5. ユニバーサルDAO
6. SQLファイルベースDB操作

XML設定調査の場合:
1. XML基本構造（component-configuration）
2. 主要要素（component, property, list, map）
3. ファイル管理（import, config-file）
4. Autowire設定
5. 初期化と破棄
6. 実プロジェクトのインポート構造例

モジュール構成調査の場合:
1. Core モジュール群
2. Framework モジュール群
3. Common Component 群
4. Adaptor 群
5. モジュール間依存関係
6. 拡張ポイント
```

#### Step 4.2: テーブル形式での整理

```
【ハンドラ一覧テーブル — 必須フォーマット】
| ハンドラ名 | FQCN | カテゴリ | 責務 |
|-----------|------|---------|------|

【ハンドラキュー構成テーブル — アプリ種別ごと】
| 順序 | ハンドラ名 | FQCN | スレッド | 責務 |
|------|-----------|------|---------|------|

【モジュール一覧テーブル】
| モジュール | artifactId | groupId | 責務 |
|----------|-----------|---------|------|

全てのクラス名はFQCN（完全修飾クラス名）で記載すること。
略称のみの記載は禁止。
```

#### Step 4.3: Markdownドキュメント作成

「Output Format」セクションのテンプレートに従い、ドキュメントを作成する。

```
【品質基準】
- 全てのクラス名はFQCN（完全修飾クラス名）で記載
- XML設定例はコードブロックで囲む
- 各情報にソースURL/GitHubリンクを付記
- 目次を含める（リンク付き）
- 各セクションの冒頭に概要を記載
- ハンドラキュー構成図はASCIIアートで図示
```

### Phase 5: 検証

ドキュメントに記載したクラス名・パッケージ名の正確性をGitHubソースコードで検証する。

#### Step 5.1: FQCN検証

```
【実行手順】

1. ドキュメントに記載した全FQCNについて、GitHubソースコードでの存在を確認
   gh api search/code --method GET \
     -f q="{ClassName} org:nablarch language:java" \
     --jq '.total_count'

2. 存在しないクラスが見つかった場合:
   - 正しいクラス名をGitHub検索で特定
   - ドキュメントを修正
   - 推測で補完した場合は「推測」と明記

3. パッケージ名の検証:
   - ソースコードのpackage文と一致するか確認
   - 特にnablarch-core vs nablarch-fw のパッケージ配置に注意
```

#### Step 5.2: 設定値の検証

```
【確認項目】

1. XMLの属性名・要素名が正しいか
   - class属性のFQCNが存在するか
   - property name がsetterメソッドと一致するか

2. ハンドラキューの順序制約が公式ドキュメントと一致するか
   - 最小構成の順序が公式のものと同じか
   - 順序制約の記述が正確か

3. 推測箇所の明記
   - ソースコードで直接確認できなかった情報に「推測」タグを付与
   - 「【推測】この情報はドキュメントからの推測であり、ソースコードでの確認が必要」
```

#### Step 5.3: 最終チェック

```
【チェックリスト】
□ 全FQCNが正確（推測箇所は明記済み）
□ XML設定例が実際のサンプルプロジェクトと整合
□ ハンドラキュー順序が公式ドキュメントと一致
□ 全情報にソースURL/GitHubリンクが付記されている
□ 目次のリンクが正しい
□ Xenlon関連の情報が含まれていない
```

## Input Format

スキル実行時に以下のパラメータを指定する。

```yaml
# 必須パラメータ
investigation_area: "ハンドラキュー"              # 調査領域
output_path: "output/nablarch_kb_architecture.md"  # 出力ファイルパス

# オプションパラメータ
depth: "detailed"                    # overview / detailed / exhaustive（デフォルト: detailed）
app_type_filter:                     # アプリ種別フィルタ（省略時は全種別）
  - "Web"
  - "REST"
additional_areas:                    # 追加調査領域
  - "トランザクション管理"
exclude_keywords:                    # 除外キーワード
  - "Xenlon"
```

### パラメータ説明

| パラメータ | 必須 | デフォルト | 説明 |
|----------|------|----------|------|
| `investigation_area` | yes | -- | 調査領域。例: ハンドラキュー, DB接続・トランザクション, インターセプタ, コンポーネント定義XML, リクエスト処理フロー, モジュール構成 |
| `output_path` | yes | -- | 出力Markdownファイルのパス |
| `depth` | -- | `detailed` | 調査深度。`overview`=公式ドキュメントのみ、`detailed`=公式+GitHub+サンプル、`exhaustive`=全情報源+隣接領域 |
| `app_type_filter` | -- | 全種別 | アプリケーション種別フィルタ。Web, REST, Batch, MOM Messaging, HTTP Messaging, Table Queue Messaging |
| `additional_areas` | -- | `[]` | 追加調査領域 |
| `exclude_keywords` | -- | `[]` | 除外キーワード |

### 深度別の処理範囲

| Phase | overview | detailed | exhaustive |
|-------|----------|----------|------------|
| 1. 公式ドキュメント | 対象ページのみ（5ページ以内） | 関連サブページ含む（15ページ以内） | 隣接領域も含む（制限なし） |
| 2. GitHubソース | なし | 主要インターフェース・クラス | 全実装クラス+テストコード |
| 3. サンプル分析 | なし | メイン設定XMLのみ | 全インポートXML+プロパティ |
| 4. 体系化 | 概要テーブルのみ | 標準カテゴリ構成 | 全カテゴリ+比較分析 |
| 5. 検証 | なし | 主要FQCNのみ | 全FQCNを検証 |

## Output Format

### ファイル構造テンプレート

```markdown
# Nablarch {調査領域} ナレッジベース

> **調査実施**: {task_id}
> **調査日**: YYYY-MM-DD
> **対象バージョン**: Nablarch 6u3
> **ソース**: 公式ドキュメント + GitHubソースコード
> **調査深度**: {depth}

---

## 目次

1. [{セクション1}](#1-セクション1)
2. [{セクション2}](#2-セクション2)
...
N. [参考リンク一覧](#n-参考リンク一覧)

---

## 1. {メインセクション}

> **ソース**: [{ページ名}]({URL})

### 1.1 {サブセクション}

{内容 — FQCN付きテーブル、XML設定例、処理フロー図を含む}

| ハンドラ名 | FQCN | 責務 |
|-----------|------|------|
| {名前} | `{完全修飾クラス名}` | {責務の説明} |

```xml
<!-- XML設定例 -->
<component name="..." class="...">
  <property name="..." value="..." />
</component>
```

---

## 2-N. {その他のセクション}

{調査領域に応じた内容}

---

## N. 参考リンク一覧

### 公式ドキュメント
- [{ページ名}]({URL})

### GitHub リポジトリ
- [{リポジトリ名}]({URL})

### Javadoc
- [{クラス名}]({URL})
```

## Nablarch Knowledge Base

本スキルの調査を効率化するために、Nablarchの基礎的なアーキテクチャ情報をインライン化する。

### 6種のアプリケーション種別

| # | アプリケーション種別 | 起動方式 | 最小ハンドラ数 | 主要用途 |
|---|-------------------|---------|-------------|---------|
| 1 | Webアプリケーション | `WebFrontController`（Servlet Filter） | 13 | サーバサイドレンダリングWebアプリ |
| 2 | RESTful Webサービス | `WebFrontController`（Servlet Filter） | 7 | REST API |
| 3 | Nablarchバッチ（都度起動） | `nablarch.fw.launcher.Main` | 9 | バッチ処理（オンデマンド） |
| 4 | Nablarchバッチ（常駐） | `nablarch.fw.launcher.Main` | 9+5 | バッチ処理（常駐デーモン） |
| 5 | MOMメッセージング | `nablarch.fw.launcher.Main` | 14 | MQ連携（同期応答/応答不要） |
| 6 | HTTPメッセージング | `WebFrontController`（Servlet Filter） | 11 | HTTP電文処理（非推奨、REST推奨） |

**補足**: Jakarta Batch（JSR352）準拠バッチ（jBeret使用）とテーブルキューメッセージングも存在するが、上記6種が主要なアーキテクチャパターンである。

### 主要ハンドラの分類

#### 共通ハンドラ（全アプリ種別で使用）

| ハンドラ名 | FQCN |
|-----------|------|
| グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` |
| スレッドコンテキスト変数管理 | `nablarch.common.handler.threadcontext.ThreadContextHandler` |
| スレッドコンテキスト変数削除 | `nablarch.common.handler.threadcontext.ThreadContextClearHandler` |
| DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` |
| トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` |
| リクエストディスパッチ | `nablarch.fw.handler.RequestPathJavaPackageMapping` |
| 認可チェック | `nablarch.common.handler.PermissionCheckHandler` |
| サービス提供可否チェック | `nablarch.common.handler.ServiceAvailabilityCheckHandler` |

#### Webアプリケーション専用ハンドラ（主要なもの）

| ハンドラ名 | FQCN |
|-----------|------|
| HTTP文字エンコード制御 | `nablarch.fw.web.handler.HttpCharacterEncodingHandler` |
| HTTPレスポンス | `nablarch.fw.web.handler.HttpResponseHandler` |
| セキュア | `nablarch.fw.web.handler.SecureHandler` |
| セッション変数保存 | `nablarch.common.web.session.SessionStoreHandler` |
| ノーマライズ | `nablarch.fw.web.handler.NormalizationHandler` |
| CSRFトークン検証 | `nablarch.fw.web.handler.CsrfTokenVerificationHandler` |

#### RESTful Webサービス専用ハンドラ

| ハンドラ名 | FQCN |
|-----------|------|
| JAX-RSレスポンス | `nablarch.fw.jaxrs.JaxRsResponseHandler` |
| ボディ変換 | `nablarch.fw.jaxrs.BodyConvertHandler` |
| BeanValidation | `nablarch.fw.jaxrs.JaxRsBeanValidationHandler` |
| CORSプリフライト | `nablarch.fw.jaxrs.CorsPreflightRequestHandler` |

#### バッチ/スタンドアローン専用ハンドラ

| ハンドラ名 | FQCN |
|-----------|------|
| ステータスコード変換 | `nablarch.fw.handler.StatusCodeConvertHandler` |
| マルチスレッド実行制御 | `nablarch.fw.handler.MultiThreadExecutionHandler` |
| トランザクションループ制御 | `nablarch.fw.handler.LoopHandler` |
| データリード | `nablarch.fw.handler.DataReadHandler` |
| プロセス常駐化 | `nablarch.fw.handler.ProcessResidentHandler` |
| プロセス停止制御 | `nablarch.fw.handler.BasicProcessStopHandler` |

#### MOMメッセージング専用ハンドラ

| ハンドラ名 | FQCN |
|-----------|------|
| メッセージングコンテキスト管理 | `nablarch.fw.messaging.handler.MessagingContextHandler` |
| 電文応答制御 | `nablarch.fw.messaging.handler.MessageReplyHandler` |
| 再送電文制御 | `nablarch.fw.messaging.handler.MessageResendHandler` |

### ハンドラキュー順序制約の概要

- **GlobalErrorHandler** は常にキューの先頭近くに配置（全例外を捕捉するため）
- **DbConnectionManagementHandler** は **TransactionManagementHandler** より前に配置（DB接続がトランザクション開始に必要）
- **DispatchHandler**（RoutesMapping等）はキューの末尾近くに配置（最終的にアクションクラスを実行）
- バッチでは **MultiThreadExecutionHandler** がメインスレッドとサブスレッドの境界となる
- **ThreadContextClearHandler** は **ThreadContextHandler** より前に配置（先にクリアしてから初期化）
- セッション関連ハンドラはDB接続より前に配置（セッションストアがDBを使用しない場合）
- セキュリティ関連ハンドラ（SecureHandler, CsrfTokenVerificationHandler）はビジネスロジックより前に配置

### 公式ドキュメントの主要URLリスト

#### アーキテクチャ概要
- トップ: https://nablarch.github.io/
- アーキテクチャ: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html
- 標準ハンドラ一覧: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html

#### アプリケーション種別アーキテクチャ
- Web: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html
- REST: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
- バッチ: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html
- MOMメッセージング: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/messaging/mom/architecture.html
- HTTPメッセージング: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/http_messaging/architecture.html

#### ライブラリ・設定
- システムリポジトリ: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html
- データベースアクセス: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database_management.html
- ユニバーサルDAO: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html
- Bean Validation: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation/bean_validation.html
- セッション変数保存: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html

#### Mavenモジュール構成
- https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/blank_project/MavenModuleStructures/index.html

#### GitHub リポジトリ
- メイン: https://github.com/nablarch/nablarch
- Core: https://github.com/nablarch/nablarch-core
- FW: https://github.com/nablarch/nablarch-fw
- FW-Web: https://github.com/nablarch/nablarch-fw-web
- Example-Web: https://github.com/nablarch/nablarch-example-web
- Example-REST: https://github.com/nablarch/nablarch-example-rest
- Example-Batch: https://github.com/nablarch/nablarch-example-batch

#### Javadoc
- Handler: https://nablarch.github.io/docs/5u8/javadoc/nablarch/fw/Handler.html
- Interceptor: https://nablarch.github.io/docs/5u9/javadoc/nablarch/fw/Interceptor.html
- Interceptor.Factory: https://nablarch.github.io/docs/5u9/javadoc/nablarch/fw/Interceptor.Factory.html

### 主要モジュール一覧（groupId: com.nablarch.framework）

| カテゴリ | artifactId | 責務 |
|---------|-----------|------|
| Core | `nablarch-core` | Handler, ExecutionContext, Interceptor, SystemRepository基盤 |
| Core | `nablarch-core-repository` | DIコンテナ、XMLコンポーネント定義 |
| Core | `nablarch-core-transaction` | トランザクション管理 |
| Core | `nablarch-core-jdbc` | JDBCユーティリティ |
| Core | `nablarch-core-validation-ee` | Jakarta Bean Validation準拠バリデーション |
| Core | `nablarch-core-applog` | ログ出力 |
| FW | `nablarch-fw` | 共通ハンドラ実装 |
| FW | `nablarch-fw-web` | Web基盤、HTTPハンドラ |
| FW | `nablarch-fw-jaxrs` | RESTful Webサービス |
| FW | `nablarch-fw-batch` | バッチ用ハンドラ/アクション |
| FW | `nablarch-fw-standalone` | スタンドアローンアプリ起動点 |
| FW | `nablarch-fw-messaging` | メッセージングアーキテクチャ |
| Common | `nablarch-common-dao` | ユニバーサルDAO |
| Common | `nablarch-common-jdbc` | DB接続ハンドラ |
| Adaptor | `nablarch-router-adaptor` | http-request-router |
| Adaptor | `nablarch-jaxrs-adaptor` | Jakarta RESTful WS |
| Adaptor | `nablarch-lettuce-adaptor` | Redis(Lettuce)セッション |
| Adaptor | `nablarch-micrometer-adaptor` | Micrometer |

## Examples

### Example 1: ハンドラキュー全体調査（detailed）

```
入力:
  investigation_area: "ハンドラキュー"
  output_path: "output/nablarch_kb_handler_queue.md"
  depth: "detailed"
  exclude_keywords: ["Xenlon"]

Phase 1 — 公式ドキュメント調査:
  WebFetch:
    1. アーキテクチャ概要ページ → ハンドラキューの設計思想、パイプライン処理モデルを抽出
    2. Webアーキテクチャページ → 13ハンドラ最小構成を抽出
    3. RESTアーキテクチャページ → 7ハンドラ最小構成を抽出
    4. バッチアーキテクチャページ → 9ハンドラ最小構成を抽出（都度起動+常駐）
    5. MOMメッセージングページ → 14ハンドラ最小構成を抽出
    6. HTTPメッセージングページ → 11ハンドラ最小構成を抽出
    7. 標準ハンドラ一覧ページ → 全ハンドラのFQCN・責務を抽出
  並列実行: 4ページずつ並列WebFetch → 2バッチで完了

Phase 2 — GitHubソースコード調査:
  gh CLI:
    1. nablarch-core/Handler.java → Handlerインターフェース定義
    2. nablarch-core/Interceptor.java → Interceptor/Factory定義
    3. nablarch-core/ExecutionContext.java → handleNext()の実装
  WebFetch:
    4. nablarch-fw の主要ハンドラ実装（GlobalErrorHandler等）

Phase 3 — サンプルプロジェクト分析:
  WebFetch（並列）:
    1. nablarch-example-web の web-component-configuration.xml
    2. nablarch-example-rest の rest-component-configuration.xml
    3. nablarch-example-batch の batch-component-configuration.xml
  分析: 最小構成との差分、追加ハンドラ、カスタムハンドラを特定

Phase 4 — 体系化:
  7セクション構成のMarkdownドキュメント作成:
    1. ハンドラキューの設計パターン（6種別の最小構成+実プロジェクト構成）
    2. 全ハンドラ一覧と仕様（FQCN付きテーブル、50+ハンドラ）
    3. インターセプタの仕組み（Factory.wrap、カスタム作成法）
    4. コンポーネント定義XML（基本構造、主要要素）
    5. DB接続・トランザクション管理（接続プール、TX境界、複数DB）
    6. リクエスト処理フロー（Web/バッチ/メッセージング）
    7. 内部アーキテクチャ（モジュール構成、拡張ポイント）

Phase 5 — 検証:
  主要FQCN 30件をgh api search/codeで検証 → 全件正確

出力: 約1200行のMarkdownドキュメント
```

### Example 2: Web/RESTアプリケーション限定調査（overview）

```
入力:
  investigation_area: "ハンドラキュー"
  output_path: "output/nablarch_kb_web_rest.md"
  depth: "overview"
  app_type_filter: ["Web", "REST"]

Phase 1 — 公式ドキュメント調査のみ:
  WebFetch:
    1. Webアーキテクチャページ → 13ハンドラ構成
    2. RESTアーキテクチャページ → 7ハンドラ構成
    3. 標準ハンドラ一覧ページ → FQCN抽出

Phase 2-3: スキップ（overview）
Phase 4: 概要テーブルのみ作成
Phase 5: スキップ（overview）

出力: 約300行のMarkdownドキュメント
```

### Example 3: DB/トランザクション管理の徹底調査（exhaustive）

```
入力:
  investigation_area: "DB接続・トランザクション"
  output_path: "output/nablarch_kb_db_transaction.md"
  depth: "exhaustive"
  additional_areas: ["ユニバーサルDAO", "SQLファイル"]

Phase 1 — 公式ドキュメント調査:
  WebFetch:
    1. データベースアクセス概要
    2. ユニバーサルDAO
    3. JDBCラッパー
    4. トランザクション管理
    5. 排他制御
    6. 各アプリ種別のDB関連設定

Phase 2 — GitHubソースコード調査:
  ソースコード取得:
    - ConnectionFactorySupport
    - BasicDbConnectionFactoryForDataSource / ForJndi
    - DbConnectionManagementHandler
    - TransactionManagementHandler
    - UniversalDao
    - BasicDaoContextFactory
    - SqlStatement, SqlResultSet

Phase 3 — サンプルプロジェクト分析:
  全XML + プロパティファイル:
    - db.xml, dao.xml
    - env.properties (DB接続設定)
    - SQLファイル (.sql)

Phase 4 — 体系化:
  6セクション構成のドキュメント

Phase 5 — 検証:
  全FQCN + setter名を検証

出力: 約800行のMarkdownドキュメント
```

### Example 4: インターセプタ機構の調査（detailed）

```
入力:
  investigation_area: "インターセプタ"
  output_path: "output/nablarch_kb_interceptor.md"
  depth: "detailed"

Phase 1 — 公式ドキュメント:
  標準ハンドラ一覧ページからインターセプタ関連を抽出

Phase 2 — GitHubソースコード:
  重点的に調査:
    - nablarch.fw.Interceptor（メタアノテーション定義）
    - nablarch.fw.Interceptor.Factory（wrap()メソッド）
    - nablarch.fw.Interceptor.Impl（基底クラス）
    - 標準インターセプタ5種の実装

Phase 3 — サンプルプロジェクト:
    - interceptors.xml（実行順定義）
    - アクションクラスでの@InjectForm/@OnError使用例

Phase 4 — 体系化:
  インターセプタの仕組み、カスタム作成法、実行順定義、標準一覧

Phase 5 — 検証:
  全インターセプタクラスのFQCN検証

出力: 約500行のMarkdownドキュメント
```

## Guidelines

### 必須ルール

1. **全てのクラス名はFQCN（完全修飾クラス名）で記載すること**
   - `GlobalErrorHandler` (NG) → `nablarch.fw.handler.GlobalErrorHandler` (OK)
   - パッケージ名が不明な場合はGitHub検索で確認
   - 確認できない場合は「推測」と明記

2. **推測と事実を明確に区別すること**
   - ソースコードまたは公式ドキュメントで確認した情報 → そのまま記載
   - 推測した情報 → 「【推測】」プレフィックスを付記
   - 推測でクラス名を記載してはならない。確認できなければ「未確認」と記載

3. **全情報にソースURL/GitHubリンクを付記すること**
   - 公式ドキュメント → `> **ソース**: [{ページ名}]({URL})`
   - GitHubソースコード → `> **ソースコード**: [{ファイル名}]({GitHub URL})`
   - Javadoc → `> **Javadoc**: [{クラス名}]({Javadoc URL})`

4. **並列実行を最大限活用すること**
   - 独立したWebFetchは4件まで並列実行
   - 独立したgh api呼び出しは並列実行
   - WebSearch も並列実行可能

5. **Xenlon関連の情報は含めないこと**
   - Xenlon（Nablarchの拡張プラットフォーム）に関する情報はスコープ外
   - 検索結果にXenlonが含まれる場合はフィルタリングする

6. **XML設定例は実プロジェクトから引用すること**
   - 自作のXML例ではなく、nablarch-example-* の実XMLを引用
   - 引用元のGitHubリンクを付記

### 検索・調査のコツ（実践知見）

1. **公式ドキュメントは英語版が存在しない場合がある**
   - https://nablarch.github.io/docs/LATEST/doc/ 配下は基本的に日本語
   - `/en/` パスが存在するページもあるが、全ページではない
   - 日本語プロンプトでWebFetchするとより正確に情報抽出できる

2. **GitHubのブランチ名に注意**
   - nablarch-core等の古いリポジトリは`master`ブランチ
   - 一部のリポジトリは`main`ブランチ
   - `gh api repos/nablarch/{repo} --jq '.default_branch'` で確認

3. **Javadocのバージョンに注意**
   - 5u8, 5u9, 5u22 等のバージョンごとにURLが異なる
   - 最新のJavadocが存在しない場合がある
   - URLが404の場合はWebSearchで代替URLを探す

4. **ハンドラキューの順序制約は各ハンドラのドキュメントに記載されている**
   - 一括で順序制約を確認できるページはない
   - 各ハンドラの「制約事項」セクションを個別に確認する必要がある

5. **gh api search/code はレート制限がある**
   - 1分間に10リクエストまで
   - 大量のFQCN検証時は間隔を空けて実行

### アンチパターン（避けるべきこと）

- **FQCNの推測**: パッケージ名が不明なクラスを推測で記載する → 必ずGitHubソースで確認
- **公式ドキュメントのみへの依存**: ドキュメントは最新でない場合がある → GitHubソースで補完
- **XML設定例の自作**: 存在しないXML設定を作成する → 実プロジェクトから引用
- **ハンドラ順序の推測**: 順序制約はドキュメント/ソースから確認 → 推測で並べない
- **Javadocリンクの推測**: URLパターンからの推測は404リスク高 → 存在確認してから記載
- **逐次的なWebFetch**: 独立ページを1つずつ取得する → 4件まで並列実行
