---
name: nablarch-handler-queue-designer
description: Nablarchプロジェクトで要件定義からハンドラキュー構成を自動設計・生成する。「Nablarchのハンドラキューを設計したい」「Web/REST/バッチのハンドラ構成を教えて」「ハンドラキューのXMLを生成して」「Xenlon移行後のハンドラキューを設計して」と言われた時に使用。
---

# Nablarch Handler Queue Designer (NHQD)

## Overview

Nablarchフレームワークのハンドラキュー構成を、要件定義から自動設計するスキル。
アプリケーション種別と非機能要件に基づき、順序制約を厳守したハンドラキュー構成XML及び設計書を生成する。

**対応するNablarchバージョン**: 6u3（Jakarta EE 10, Java 17+）

## When to Use

以下の状況で使用する:

- 「Nablarchのハンドラキューを設計したい」
- 「Webアプリ / REST API / バッチのハンドラ構成を教えて」
- 「ハンドラキューの設定XMLを生成して」
- 「Xenlon移行後のハンドラキュー構成を設計して」
- 「ハンドラの順序制約を確認したい」
- 「既存のハンドラキュー構成をレビューして」
- Nablarchプロジェクトの新規構築時
- COBOL/PL/IからNablarchへの移行時（Xenlon利用時）

## Instructions

### STEP 1: 要件ヒアリング

ユーザーに以下の情報を確認せよ。不明な項目は推奨デフォルト値を提示して確認する。

#### 1.1 必須ヒアリング項目

| # | 項目 | 選択肢 | 補足 |
|---|------|--------|------|
| 1 | **アプリケーション種別** | `web` / `rest` / `batch` / `batch_resident` / `mom_messaging` / `http_messaging` / `db_queue` | 最初に確定させること |
| 2 | **データベース使用** | `true` / `false` | DB種別も確認（PostgreSQL, Oracle, DB2等） |
| 3 | **トランザクション** | `required` / `not_required` | DBを使う場合は原則required |

#### 1.2 アプリ種別依存ヒアリング項目

**Web / REST 共通:**

| # | 項目 | 選択肢 | デフォルト |
|---|------|--------|-----------|
| 4 | 認証方式 | `session` / `token` / `none` | web→session, rest→token |
| 5 | ログインチェック | `true` / `false` | 認証ありならtrue |
| 6 | CSRF対策 | `true` / `false` | web→true, rest→false |
| 7 | セキュアヘッダ | `true` / `false` | true |
| 8 | CORS | `true` / `false` | rest→true, web→false |
| 9 | アクセスログ | `true` / `false` | true |

**Web専用:**

| # | 項目 | 選択肢 | デフォルト |
|---|------|--------|-----------|
| 10 | セッション管理 | `db` / `http_session` | db |
| 11 | マルチパート（ファイルアップロード） | `true` / `false` | false |
| 12 | 入力正規化（トリム等） | `true` / `false` | true |
| 13 | 二重送信防止 | `true` / `false` | true |
| 14 | カスタムタグ使用 | `true` / `false` | true |
| 15 | ヘルスチェック | `true` / `false` | false |

**バッチ専用:**

| # | 項目 | 選択肢 | デフォルト |
|---|------|--------|-----------|
| 10 | マルチスレッド | `true` / `false` | true |
| 11 | 常駐バッチ | `true` / `false` | false |
| 12 | リトライ制御 | `true` / `false` | 常駐→true |
| 13 | 停止制御 | `true` / `false` | 常駐→true |

**メッセージング専用:**

| # | 項目 | 選択肢 | デフォルト |
|---|------|--------|-----------|
| 10 | メッセージング方式 | `mom` / `http` / `db_queue` | - |
| 11 | 再送制御 | `true` / `false` | mom→true |

#### 1.3 共通オプション

| # | 項目 | 選択肢 | デフォルト |
|---|------|--------|-----------|
| A | カスタムハンドラ | リスト（名前+挿入位置） | なし |
| B | Xenlon移行 | `true` / `false` | false |
| C | 移行元言語（Xenlon時） | `COBOL` / `PL/I` / `Easytrieve` | - |

### STEP 2: 要件YAML構築

ヒアリング結果を以下のYAMLフォーマットに構造化する。

```yaml
# NHQD 要件定義
project:
  name: "{プロジェクト名}"
  type: web  # web | rest | batch | batch_resident | mom_messaging | http_messaging | db_queue

requirements:
  database:
    enabled: true          # true | false
    type: PostgreSQL       # PostgreSQL | Oracle | DB2 | MySQL | SQLServer
    transaction: required  # required | not_required

  authentication:
    enabled: true          # true | false
    type: session          # session | token | none
    login_check: true      # true | false

  security:
    csrf_protection: true  # true | false
    secure_headers: true   # true | false
    cors: false            # true | false

  session:
    enabled: true          # true | false
    store: db              # db | http_session

  logging:
    access_log: true       # true | false
    sql_log: false         # true | false

  validation:
    bean_validation: true         # true | false
    double_submit_check: false    # true | false

  file_handling:
    multipart: false       # true | false

  normalization:
    trim: true             # true | false
    date_format: true      # true | false

  health_check:
    enabled: false         # true | false

  # バッチ専用
  batch:
    multi_thread: true     # true | false
    resident: false        # true | false
    retry: false           # true | false
    stop_control: false    # true | false

  # メッセージング専用
  messaging:
    type: mom              # mom | http | db_queue
    resend_control: true   # true | false

  # カスタムハンドラ
  custom_handlers:
    - name: "CustomHandler"
      position: "after:TransactionManagementHandler"
      description: "説明"

  # Xenlon移行
  migration:
    from_xenlon: false
    source_language: null   # COBOL | PL/I | Easytrieve
    original_platform: null # IBM | Fujitsu | NEC | Hitachi
```

### STEP 3: ベースパターン選択

アプリケーション種別に基づき、以下の6つのベースパターンから選択する。

---

#### パターン1: Web アプリケーション

**起動クラス**: `WebFrontController`
**エントリポイント**: `web.xml` のサーブレットフィルタ

```
WebFrontController (handlerQueue)
├──  1. HttpCharacterEncodingHandler        [必須] 文字エンコーディング設定
├──  2. ThreadContextClearHandler           [必須] スレッドコンテキスト初期化
├──  3. GlobalErrorHandler                  [必須] グローバルエラー処理
├──  4. HttpResponseHandler                 [必須] HTTPレスポンス処理
├──  5. SecureHandler                       [条件] セキュアヘッダ設定（CSP, X-Frame-Options等）
├──  6. MultipartHandler                    [条件] マルチパート処理（ファイルアップロード時）
├──  7. SessionStoreHandler                 [条件] セッション管理
├──  8. ThreadContextHandler                [必須] スレッドコンテキスト変数設定
├──  9. HttpAccessLogHandler                [条件] アクセスログ出力
├── 10. NormalizationHandler                [条件] 入力正規化（トリム、日付フォーマット等）
├── 11. ForwardingHandler                   [推奨] 内部フォワード処理
├── 12. HttpErrorHandler                    [推奨] HTTPエラーページマッピング
├── 13. NablarchTagHandler                  [条件] Nablarchカスタムタグ処理
├── 14. CsrfTokenVerificationHandler        [条件] CSRF対策トークン検証
├── 15. DbConnectionManagementHandler       [条件] DB接続管理
├── 16. TransactionManagementHandler        [条件] トランザクション制御
├── 17. LoginUserPrincipalCheckHandler      [条件] ログインユーザー認証チェック
├── 18. HealthCheckEndpointHandler          [条件] ヘルスチェック
├── 19. {カスタムハンドラ挿入ポイント}
└── 20. HttpRequestJavaPackageMapping       [必須] ディスパッチ（末尾固定）
```

**要件→ハンドラ マッピングルール:**

| 要件フラグ | 追加するハンドラ |
|-----------|----------------|
| `security.secure_headers: true` | SecureHandler |
| `file_handling.multipart: true` | MultipartHandler |
| `session.enabled: true` | SessionStoreHandler |
| `logging.access_log: true` | HttpAccessLogHandler |
| `normalization.trim: true` or `normalization.date_format: true` | NormalizationHandler |
| `validation.double_submit_check: true` | NablarchTagHandler (必要), CsrfTokenVerificationHandler |
| `security.csrf_protection: true` | CsrfTokenVerificationHandler |
| `database.enabled: true` | DbConnectionManagementHandler |
| `database.transaction: required` | TransactionManagementHandler |
| `authentication.login_check: true` | LoginUserPrincipalCheckHandler |
| `health_check.enabled: true` | HealthCheckEndpointHandler |

---

#### パターン2: RESTful ウェブサービス

**起動クラス**: `WebFrontController`
**エントリポイント**: `web.xml` のサーブレットフィルタ

```
WebFrontController (handlerQueue)
├── 1. HttpCharacterEncodingHandler         [推奨] 文字エンコーディング
├── 2. GlobalErrorHandler                   [必須] グローバルエラー処理
├── 3. JaxRsResponseHandler                 [必須] JAX-RSレスポンス処理
├── 4. DbConnectionManagementHandler        [条件] DB接続管理
├── 5. TransactionManagementHandler         [条件] トランザクション制御
├── 6. CorsPreflightRequestHandler          [条件] CORSプリフライト処理
├── 7. HealthCheckEndpointHandler           [条件] ヘルスチェック
└── 8. RoutesMapping                        [必須] ルーティング（末尾）
       ├── a. BodyConvertHandler             [必須] リクエスト/レスポンスボディ変換
       └── b. JaxRsBeanValidationHandler     [条件] BeanValidation
```

**要件→ハンドラ マッピングルール:**

| 要件フラグ | 追加するハンドラ |
|-----------|----------------|
| `database.enabled: true` | DbConnectionManagementHandler |
| `database.transaction: required` | TransactionManagementHandler |
| `security.cors: true` | CorsPreflightRequestHandler |
| `health_check.enabled: true` | HealthCheckEndpointHandler |
| `validation.bean_validation: true` | JaxRsBeanValidationHandler |

**RoutesMapping内ハンドラの設定方法:**
RoutesMapping（ルーティングハンドラ）より後に配置するハンドラは、ハンドラキューに直接設定せず、RoutesMapping内の `handlerList` プロパティに設定する。

---

#### パターン3: バッチアプリケーション（都度起動）

**起動クラス**: `Main`
**エントリポイント**: `java` コマンド直接起動

```
Main (handlerQueue)
├── 1. StatusCodeConvertHandler             [必須] 終了コード変換
├── 2. ThreadContextClearHandler            [必須] スレッドコンテキスト初期化
├── 3. GlobalErrorHandler                   [必須] グローバルエラー処理
├── 4. ThreadContextHandler                 [必須] スレッドコンテキスト設定
├── 5. DbConnectionManagementHandler        [条件] DB接続管理（初期処理用）
├── 6. TransactionManagementHandler         [条件] トランザクション制御（初期処理用）
├── 7. RequestPathJavaPackageMapping        [必須] ディスパッチ
├── 8. MultiThreadExecutionHandler          [条件] マルチスレッド実行制御
├── 9. DbConnectionManagementHandler        [条件] DB接続管理（業務処理用）
├── 10. LoopHandler (or DblessLoopHandler)  [必須] ループ制御
└── 11. DataReadHandler                     [必須] データ読み込み
```

**注意**: DB未使用の場合は `LoopHandler` の代わりに `DblessLoopHandler` を使用する。

---

#### パターン4: バッチアプリケーション（常駐）

**起動クラス**: `Main`

```
Main (handlerQueue)
├──  1. StatusCodeConvertHandler            [必須] 終了コード変換
├──  2. ThreadContextClearHandler           [必須] スレッドコンテキスト初期化
├──  3. GlobalErrorHandler                  [必須] グローバルエラー処理
├──  4. ThreadContextHandler                [必須] スレッドコンテキスト設定
├──  5. RetryHandler                        [推奨] リトライ制御
├──  6. ProcessResidentHandler              [必須] プロセス常駐化
├──  7. ProcessStopHandler                  [推奨] 停止制御
├──  8. DbConnectionManagementHandler       [条件] DB接続管理（初期処理用）
├──  9. TransactionManagementHandler        [条件] トランザクション制御（初期処理用）
├── 10. RequestPathJavaPackageMapping       [必須] ディスパッチ
├── 11. MultiThreadExecutionHandler         [条件] マルチスレッド実行制御
├── 12. DbConnectionManagementHandler       [条件] DB接続管理（業務処理用）
├── 13. LoopHandler                         [必須] ループ制御
└── 14. DataReadHandler                     [必須] データ読み込み
```

---

#### パターン5: MOM メッセージング

**起動クラス**: `Main`

```
Main (handlerQueue)
├──  1. StatusCodeConvertHandler            [必須] 終了コード変換
├──  2. ThreadContextClearHandler           [必須] スレッドコンテキスト初期化
├──  3. GlobalErrorHandler                  [必須] グローバルエラー処理
├──  4. ThreadContextHandler                [必須] スレッドコンテキスト設定
├──  5. RetryHandler                        [推奨] リトライ制御
├──  6. ProcessResidentHandler              [必須] プロセス常駐化
├──  7. ProcessStopHandler                  [推奨] 停止制御
├──  8. DbConnectionManagementHandler       [条件] DB接続管理
├──  9. TransactionManagementHandler        [条件] トランザクション制御
├── 10. RequestPathJavaPackageMapping       [必須] ディスパッチ
├── 11. MultiThreadExecutionHandler         [条件] マルチスレッド実行制御
├── 12. DbConnectionManagementHandler       [条件] DB接続管理（業務処理用）
├── 13. LoopHandler                         [必須] ループ制御
├── 14. MessageResendHandler                [条件] 再送メッセージ制御
└── 15. DataReadHandler                     [必須] データ読み込み
```

---

#### パターン6: HTTPメッセージング / DBキューメッセージング

**HTTPメッセージング** は Webアプリケーションベースで、以下のハンドラを追加:

```
WebFrontController (handlerQueue)
├── 1. HttpCharacterEncodingHandler         [必須]
├── 2. GlobalErrorHandler                   [必須]
├── 3. HttpResponseHandler                  [必須]
├── 4. ThreadContextHandler                 [必須]
├── 5. HttpMessagingRequestParsingHandler   [必須] HTTPメッセージング変換
├── 6. DbConnectionManagementHandler        [条件]
├── 7. TransactionManagementHandler         [条件]
├── 8. HttpMessagingResponseBuildingHandler  [必須] レスポンス変換
└── 9. RequestPathJavaPackageMapping        [必須] ディスパッチ
```

**DBキューメッセージング** はバッチベースで、テーブルをキューとして使用する構成。パターン4（常駐バッチ）とほぼ同一だが、DataReaderとして `DatabaseTableQueueReader` を使用する。

### STEP 4: 順序制約の検証

生成されたハンドラキューに対し、以下の**10件の順序制約**を全て検証する。1つでも違反があれば修正すること。

#### 順序制約一覧 (C-01 ~ C-10)

| 制約ID | 制約内容 | 理由 | 対象パターン |
|--------|---------|------|-------------|
| **C-01** | `TransactionManagementHandler` は `DbConnectionManagementHandler` より後に配置 | DB接続が確立されていないとトランザクション開始不可 | 全て |
| **C-02** | ディスパッチハンドラ（`HttpRequestJavaPackageMapping`, `RequestPathJavaPackageMapping`）はハンドラキュー末尾に配置 | 後続ハンドラを呼び出さないため | Web, Batch |
| **C-03** | `RoutesMapping` 内のハンドラ（`BodyConvertHandler`, `JaxRsBeanValidationHandler`）は `RoutesMapping` の `handlerList` プロパティに設定 | ルーティング後のみ実行されるべき | REST |
| **C-04** | `HttpMessagingRequestParsingHandler` は `HttpResponseHandler` より後に配置 | レスポンスハンドラがクライアントへの返却を担当 | HTTPメッセージング |
| **C-05** | `HttpMessagingRequestParsingHandler` は `ThreadContextHandler` より後に配置 | スレッドコンテキストが必須 | HTTPメッセージング |
| **C-06** | `HealthCheckEndpointHandler` は `HttpResponseHandler` or `JaxRsResponseHandler` より後に配置 | レスポンスハンドラに依存 | Web, REST |
| **C-07** | `LoopHandler` は `MultiThreadExecutionHandler` より後に配置 | サブスレッド内で動作するため | Batch |
| **C-08** | `DataReadHandler` は `LoopHandler` より後に配置 | ループ制御配下で動作 | Batch |
| **C-09** | `GlobalErrorHandler` は先頭付近に配置 | 全体のエラーをキャッチするため | 全て |
| **C-10** | インターセプタ実行順序は設定ファイルで明示的に指定 | JVM依存で非決定的になるため | Web |

#### デフォルトインターセプタ実行順序（C-10用）

Webアプリケーションでインターセプタを使用する場合、以下の順序を設定ファイルに明示すること:

1. `OnDoubleSubmission`
2. `UseToken`
3. `OnErrors`
4. `OnError`
5. `InjectForm`

#### 制約検証ロジック（DAG + トポロジカルソート）

順序制約は**有向非巡回グラフ（DAG）**としてモデル化し、トポロジカルソートで検証する。

```
制約グラフ（エッジ = "先に配置すべき" → "後に配置すべき"）:

GlobalErrorHandler ──→ (先頭付近: position <= 3)
DbConnectionManagementHandler ──→ TransactionManagementHandler     [C-01]
TransactionManagementHandler ──→ DispatchHandler
HttpResponseHandler ──→ HttpMessagingRequestParsingHandler          [C-04]
ThreadContextHandler ──→ HttpMessagingRequestParsingHandler         [C-05]
HttpResponseHandler/JaxRsResponseHandler ──→ HealthCheckEndpointHandler [C-06]
MultiThreadExecutionHandler ──→ LoopHandler                         [C-07]
LoopHandler ──→ DataReadHandler                                     [C-08]
DispatchHandler ──→ (末尾: position = last)                         [C-02]
```

**検証手順:**
1. 生成されたハンドラリストのインデックスを取得
2. 各制約について、先行ハンドラのインデックス < 後続ハンドラのインデックスを確認
3. 違反があれば、制約を満たすように並べ替え
4. 並べ替え後、再度全制約を検証（循環依存チェック）

### STEP 5: XML設定ファイル生成

検証済みのハンドラキューからXML設定ファイルを生成する。

#### Webアプリケーション用テンプレート

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Nablarch Handler Queue Configuration
  Generated by NHQD (Nablarch Handler Queue Designer)
  Project: {project.name}
  Type: {project.type}
-->
<component-configuration
    xmlns="http://tis.co.jp/nablarch/component-configuration"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://tis.co.jp/nablarch/component-configuration">

  <!-- ハンドラキュー定義 -->
  <component name="webFrontController"
             class="nablarch.fw.web.servlet.WebFrontController">
    <property name="handlerQueue">
      <list>
        <!-- 1. 文字エンコーディング -->
        <component class="nablarch.fw.web.handler.HttpCharacterEncodingHandler"/>

        <!-- 2. スレッドコンテキスト初期化 -->
        <component class="nablarch.fw.handler.ThreadContextClearHandler"/>

        <!-- 3. グローバルエラー処理 -->
        <component class="nablarch.fw.handler.GlobalErrorHandler"/>

        <!-- 4. HTTPレスポンス処理 -->
        <component class="nablarch.fw.web.handler.HttpResponseHandler">
          <property name="customResponseWriter">
            <component class="nablarch.fw.web.handler.responsewriter.CustomResponseWriter"/>
          </property>
        </component>

        <!-- 5. セキュアヘッダ設定 -->
        <!-- {secure_headers: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.SecureHandler"/>

        <!-- 6. マルチパート処理 -->
        <!-- {multipart: true の場合のみ} -->
        <component-ref name="multipartHandler"/>

        <!-- 7. セッション管理 -->
        <!-- {session.enabled: true の場合のみ} -->
        <component-ref name="sessionStoreHandler"/>

        <!-- 8. スレッドコンテキスト設定 -->
        <component-ref name="threadContextHandler"/>

        <!-- 9. アクセスログ -->
        <!-- {access_log: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.HttpAccessLogHandler"/>

        <!-- 10. 入力正規化 -->
        <!-- {normalization: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.NormalizationHandler"/>

        <!-- 11. フォワード処理 -->
        <component class="nablarch.fw.web.handler.ForwardingHandler"/>

        <!-- 12. HTTPエラーページ -->
        <component class="nablarch.fw.web.handler.HttpErrorHandler"/>

        <!-- 13. カスタムタグ処理 -->
        <!-- {nablarch_tag: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.NablarchTagHandler"/>

        <!-- 14. CSRF対策 -->
        <!-- {csrf_protection: true の場合のみ} -->
        <component-ref name="csrfTokenVerificationHandler"/>

        <!-- 15. DB接続管理 -->
        <!-- {database.enabled: true の場合のみ} -->
        <component-ref name="dbConnectionManagementHandler"/>

        <!-- 16. トランザクション制御 -->
        <!-- {transaction: required の場合のみ} -->
        <component-ref name="transactionManagementHandler"/>

        <!-- 17. ログインチェック -->
        <!-- {login_check: true の場合のみ} -->
        <component class="nablarch.common.web.handler.LoginUserPrincipalCheckHandler"/>

        <!-- 18. ヘルスチェック -->
        <!-- {health_check: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.HealthCheckEndpointHandler"/>

        <!-- {カスタムハンドラ挿入ポイント} -->

        <!-- 末尾: ディスパッチ -->
        <component class="nablarch.fw.web.handler.HttpRequestJavaPackageMapping">
          <property name="basePackage" value="{base_package}.action"/>
        </component>
      </list>
    </property>
  </component>

</component-configuration>
```

#### RESTfulウェブサービス用テンプレート

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component-configuration
    xmlns="http://tis.co.jp/nablarch/component-configuration"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://tis.co.jp/nablarch/component-configuration">

  <component name="webFrontController"
             class="nablarch.fw.web.servlet.WebFrontController">
    <property name="handlerQueue">
      <list>
        <!-- 1. 文字エンコーディング -->
        <component class="nablarch.fw.web.handler.HttpCharacterEncodingHandler"/>

        <!-- 2. グローバルエラー処理 -->
        <component class="nablarch.fw.handler.GlobalErrorHandler"/>

        <!-- 3. JAX-RSレスポンス処理 -->
        <component class="nablarch.fw.jaxrs.JaxRsResponseHandler"/>

        <!-- 4. DB接続管理 -->
        <!-- {database.enabled: true の場合のみ} -->
        <component-ref name="dbConnectionManagementHandler"/>

        <!-- 5. トランザクション制御 -->
        <!-- {transaction: required の場合のみ} -->
        <component-ref name="transactionManagementHandler"/>

        <!-- 6. CORSプリフライト -->
        <!-- {cors: true の場合のみ。JaxRsResponseHandlerより後に配置} -->
        <component class="nablarch.fw.jaxrs.CorsPreflightRequestHandler"/>

        <!-- 7. ヘルスチェック -->
        <!-- {health_check: true の場合のみ} -->
        <component class="nablarch.fw.web.handler.HealthCheckEndpointHandler"/>

        <!-- 末尾: ルーティング -->
        <component class="nablarch.integration.router.RoutesMapping">
          <property name="methodBinderFactory">
            <component class="nablarch.fw.jaxrs.JaxRsMethodBinderFactory">
              <property name="handlerList">
                <list>
                  <!-- a. ボディ変換 -->
                  <component class="nablarch.fw.jaxrs.BodyConvertHandler">
                    <property name="bodyConverters">
                      <list>
                        <component class="nablarch.fw.jaxrs.JacksonBodyConverter"/>
                      </list>
                    </property>
                  </component>
                  <!-- b. BeanValidation -->
                  <!-- {bean_validation: true の場合のみ} -->
                  <component class="nablarch.fw.jaxrs.JaxRsBeanValidationHandler"/>
                </list>
              </property>
            </component>
          </property>
          <property name="basePackage" value="{base_package}.action"/>
        </component>
      </list>
    </property>
  </component>

</component-configuration>
```

#### バッチアプリケーション用テンプレート（都度起動）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component-configuration
    xmlns="http://tis.co.jp/nablarch/component-configuration"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://tis.co.jp/nablarch/component-configuration">

  <component name="main" class="nablarch.fw.launcher.Main">
    <property name="handlerQueue">
      <list>
        <!-- 1. 終了コード変換 -->
        <component class="nablarch.fw.handler.StatusCodeConvertHandler"/>

        <!-- 2. スレッドコンテキスト初期化 -->
        <component class="nablarch.fw.handler.ThreadContextClearHandler"/>

        <!-- 3. グローバルエラー処理 -->
        <component class="nablarch.fw.handler.GlobalErrorHandler"/>

        <!-- 4. スレッドコンテキスト設定 -->
        <component-ref name="threadContextHandler"/>

        <!-- 5. DB接続管理（初期処理用） -->
        <!-- {database.enabled: true の場合のみ} -->
        <component-ref name="dbConnectionManagementHandler"/>

        <!-- 6. トランザクション制御（初期処理用） -->
        <!-- {transaction: required の場合のみ} -->
        <component-ref name="transactionManagementHandler"/>

        <!-- 7. ディスパッチ -->
        <component class="nablarch.fw.handler.RequestPathJavaPackageMapping">
          <property name="basePackage" value="{base_package}.action"/>
        </component>

        <!-- 8. マルチスレッド制御 -->
        <!-- {multi_thread: true の場合のみ} -->
        <component class="nablarch.fw.handler.MultiThreadExecutionHandler">
          <property name="concurrentNumber" value="8"/>
        </component>

        <!-- 9. DB接続管理（業務処理用） -->
        <!-- {database.enabled: true の場合のみ} -->
        <component-ref name="dbConnectionManagementHandler"/>

        <!-- 10. ループ制御 -->
        <!-- DB有: LoopHandler / DB無: DblessLoopHandler -->
        <component class="nablarch.fw.handler.LoopHandler">
          <property name="transactionFactory" ref="jdbcTransactionFactory"/>
        </component>

        <!-- 11. データ読み込み -->
        <component class="nablarch.fw.handler.DataReadHandler"/>
      </list>
    </property>
  </component>

</component-configuration>
```

### STEP 6: 設計書（Markdown）生成

以下のフォーマットで設計書を生成する。

```markdown
# ハンドラキュー設計書

## 基本情報

| 項目 | 値 |
|------|-----|
| プロジェクト名 | {project.name} |
| アプリケーション種別 | {project.type} |
| ハンドラ数 | {count} |
| データベース | {database.type} |
| 認証方式 | {authentication.type} |
| 生成日時 | {timestamp} |

## ハンドラキュー構成

| # | ハンドラ | 分類 | 役割 | 往路処理 | 復路処理 | 例外処理 |
|---|---------|------|------|---------|---------|---------|
| 1 | {handler_name} | {必須/条件/推奨} | {role} | {inbound} | {outbound} | {error} |
| ... | ... | ... | ... | ... | ... | ... |

## 順序制約チェック結果

| 制約ID | 制約内容 | 結果 |
|--------|---------|------|
| C-01 | Transaction は DB接続より後 | PASS |
| C-02 | Dispatch は末尾 | PASS |
| ... | ... | ... |

## 要件カバレッジ

| 要件 | 対応ハンドラ | ステータス |
|------|------------|----------|
| DB接続 | DbConnectionManagementHandler | 対応済 |
| トランザクション | TransactionManagementHandler | 対応済 |
| ... | ... | ... |

## 設計根拠

{各ハンドラの採用理由、順序の根拠を記述}

## Xenlon移行考慮事項

{Xenlon移行の場合のみ記述}
```

## Examples

### Example 1: Webアプリケーション（フル構成19ハンドラ）

**入力YAML:**

```yaml
project:
  name: "顧客管理システム"
  type: web

requirements:
  database:
    enabled: true
    type: PostgreSQL
    transaction: required
  authentication:
    enabled: true
    type: session
    login_check: true
  security:
    csrf_protection: true
    secure_headers: true
    cors: false
  session:
    enabled: true
    store: db
  logging:
    access_log: true
  validation:
    bean_validation: true
    double_submit_check: true
  file_handling:
    multipart: true
  normalization:
    trim: true
    date_format: true
  health_check:
    enabled: false
```

**出力XML:**

```xml
<component name="webFrontController"
           class="nablarch.fw.web.servlet.WebFrontController">
  <property name="handlerQueue">
    <list>
      <component class="nablarch.fw.web.handler.HttpCharacterEncodingHandler"/>
      <component class="nablarch.fw.handler.ThreadContextClearHandler"/>
      <component class="nablarch.fw.handler.GlobalErrorHandler"/>
      <component class="nablarch.fw.web.handler.HttpResponseHandler"/>
      <component class="nablarch.fw.web.handler.SecureHandler"/>
      <component-ref name="multipartHandler"/>
      <component-ref name="sessionStoreHandler"/>
      <component-ref name="threadContextHandler"/>
      <component class="nablarch.fw.web.handler.HttpAccessLogHandler"/>
      <component class="nablarch.fw.web.handler.NormalizationHandler"/>
      <component class="nablarch.fw.web.handler.ForwardingHandler"/>
      <component class="nablarch.fw.web.handler.HttpErrorHandler"/>
      <component class="nablarch.fw.web.handler.NablarchTagHandler"/>
      <component-ref name="csrfTokenVerificationHandler"/>
      <component-ref name="dbConnectionManagementHandler"/>
      <component-ref name="transactionManagementHandler"/>
      <component class="nablarch.common.web.handler.LoginUserPrincipalCheckHandler"/>
      <component class="nablarch.fw.web.handler.HttpErrorForwardHandler"/>
      <component class="nablarch.fw.web.handler.HttpRequestJavaPackageMapping">
        <property name="basePackage" value="com.example.app.action"/>
      </component>
    </list>
  </property>
</component>
```

**制約チェック結果:**
- C-01: PASS (DbConnectionManagement[15] < TransactionManagement[16])
- C-02: PASS (HttpRequestJavaPackageMapping[19] = 末尾)
- C-06: N/A (HealthCheck未使用)
- C-09: PASS (GlobalErrorHandler[3] = 先頭付近)
- C-10: PASS (インターセプタ順序は別途設定ファイルで定義)

---

### Example 2: RESTful Webサービス（最小構成7ハンドラ）

**入力YAML:**

```yaml
project:
  name: "商品API"
  type: rest

requirements:
  database:
    enabled: true
    type: PostgreSQL
    transaction: required
  authentication:
    enabled: false
  security:
    cors: true
  validation:
    bean_validation: true
  health_check:
    enabled: true
```

**出力XML:**

```xml
<component name="webFrontController"
           class="nablarch.fw.web.servlet.WebFrontController">
  <property name="handlerQueue">
    <list>
      <component class="nablarch.fw.web.handler.HttpCharacterEncodingHandler"/>
      <component class="nablarch.fw.handler.GlobalErrorHandler"/>
      <component class="nablarch.fw.jaxrs.JaxRsResponseHandler"/>
      <component-ref name="dbConnectionManagementHandler"/>
      <component-ref name="transactionManagementHandler"/>
      <component class="nablarch.fw.jaxrs.CorsPreflightRequestHandler"/>
      <component class="nablarch.fw.web.handler.HealthCheckEndpointHandler"/>
      <component class="nablarch.integration.router.RoutesMapping">
        <property name="methodBinderFactory">
          <component class="nablarch.fw.jaxrs.JaxRsMethodBinderFactory">
            <property name="handlerList">
              <list>
                <component class="nablarch.fw.jaxrs.BodyConvertHandler">
                  <property name="bodyConverters">
                    <list>
                      <component class="nablarch.fw.jaxrs.JacksonBodyConverter"/>
                    </list>
                  </property>
                </component>
                <component class="nablarch.fw.jaxrs.JaxRsBeanValidationHandler"/>
              </list>
            </property>
          </component>
        </property>
        <property name="basePackage" value="com.example.api.action"/>
      </component>
    </list>
  </property>
</component>
```

**制約チェック結果:**
- C-01: PASS (DbConnectionManagement[4] < TransactionManagement[5])
- C-03: PASS (BodyConvertHandler, BeanValidationHandler は RoutesMapping内)
- C-06: PASS (HealthCheckEndpoint[7] > JaxRsResponseHandler[3])
- C-09: PASS (GlobalErrorHandler[2] = 先頭付近)

---

### Example 3: バッチアプリケーション（都度起動・9ハンドラ）

**入力YAML:**

```yaml
project:
  name: "月次集計バッチ"
  type: batch

requirements:
  database:
    enabled: true
    type: Oracle
    transaction: required
  batch:
    multi_thread: true
    resident: false
```

**出力XML:**

```xml
<component name="main" class="nablarch.fw.launcher.Main">
  <property name="handlerQueue">
    <list>
      <component class="nablarch.fw.handler.StatusCodeConvertHandler"/>
      <component class="nablarch.fw.handler.ThreadContextClearHandler"/>
      <component class="nablarch.fw.handler.GlobalErrorHandler"/>
      <component-ref name="threadContextHandler"/>
      <component-ref name="dbConnectionManagementHandler"/>
      <component-ref name="transactionManagementHandler"/>
      <component class="nablarch.fw.handler.RequestPathJavaPackageMapping">
        <property name="basePackage" value="com.example.batch.action"/>
      </component>
      <component class="nablarch.fw.handler.MultiThreadExecutionHandler">
        <property name="concurrentNumber" value="8"/>
      </component>
      <component-ref name="dbConnectionManagementHandler"/>
      <component class="nablarch.fw.handler.LoopHandler">
        <property name="transactionFactory" ref="jdbcTransactionFactory"/>
      </component>
      <component class="nablarch.fw.handler.DataReadHandler"/>
    </list>
  </property>
</component>
```

**制約チェック結果:**
- C-01: PASS (DbConnectionManagement[5] < TransactionManagement[6])
- C-07: PASS (MultiThreadExecution[8] < LoopHandler[10])
- C-08: PASS (LoopHandler[10] < DataReadHandler[11])
- C-09: PASS (GlobalErrorHandler[3] = 先頭付近)

---

### Example 4: 自然言語入力からの設計

**入力（自然言語）:**

```
社内向けの勤怠管理Webアプリを作りたい。
PostgreSQLを使って、ログインが必要。
ファイルアップロードは不要。
```

**解析結果（YAML）:**

```yaml
project:
  name: "勤怠管理システム"
  type: web
requirements:
  database:
    enabled: true
    type: PostgreSQL
    transaction: required
  authentication:
    enabled: true
    type: session
    login_check: true
  security:
    csrf_protection: true   # Web推奨デフォルト
    secure_headers: true    # Web推奨デフォルト
  session:
    enabled: true
    store: db               # デフォルト
  logging:
    access_log: true        # デフォルト
  file_handling:
    multipart: false        # 明示的に不要
  normalization:
    trim: true              # デフォルト
```

**生成されるハンドラ数**: 17（マルチパートなし、ヘルスチェックなし）

## Guidelines

### 絶対遵守ルール

1. **順序制約は絶対に違反してはならない**
   - C-01〜C-10 を全て検証してからXMLを出力すること
   - 1つでも違反があれば、修正してから出力する

2. **DispatchHandler は必ず末尾に配置**
   - `HttpRequestJavaPackageMapping`（Web）→ 末尾
   - `RequestPathJavaPackageMapping`（Batch/Messaging）→ マルチスレッドの前（ただしメインスレッド側の末尾）
   - `RoutesMapping`（REST）→ 末尾

3. **DB接続 → トランザクションの順序を厳守**（C-01）
   - `DbConnectionManagementHandler` を先に配置
   - その後に `TransactionManagementHandler` を配置
   - バッチでは「初期処理用」と「業務処理用」の2セットが必要

4. **RoutesMapping内ハンドラの配置方法**（C-03）
   - `BodyConvertHandler`, `JaxRsBeanValidationHandler` は `RoutesMapping` の `handlerList` に設定
   - ハンドラキューの `<list>` に直接書かない

5. **カスタムハンドラの挿入位置は推奨として提示**
   - 「この位置に挿入することを推奨しますが、要件に応じて調整してください」と明記
   - 一般的な推奨位置: `TransactionManagementHandler` の後、`DispatchHandler` の前

### Xenlon移行時の考慮事項

1. **COBOL→Nablarch移行**
   - Xenlon Migratorが業務ロジックを自動変換する
   - ハンドラキューは移行プロジェクト側で設計が必要
   - 元のCOBOLシステムのトランザクション境界を考慮してハンドラ配置を決定

2. **バッチ処理の移行**
   - COBOLバッチ → Nablarchバッチへの移行が多い
   - ファイルI/Oパターン: `FileDataReader` / `ValidatableFileDataReader` を使用
   - オンラインとバッチの連携: DBキューメッセージング（パターン6）の検討

3. **性能考慮**
   - Xenlon変換後のJavaコードは実行時ライブラリに依存
   - トランザクション制御ハンドラの配置がパフォーマンスに影響
   - マルチスレッドのスレッド数（`concurrentNumber`）は元システムの処理量を考慮

4. **段階的移行パターン**
   - 画面系（Web）とバッチ系を別々に移行する場合がある
   - 各フェーズで異なるハンドラキュー構成が必要
   - 共通コンポーネント（DB接続、トランザクション）は共有可能

### 出力時の注意

- XMLコメントでハンドラの役割を記述すること
- 条件付きハンドラはコメントで条件を明示すること
- `component-ref` と `component` の使い分け: 他の設定ファイルで定義済みのものは `component-ref` を使用
- `basePackage` はプロジェクト固有の値をプレースホルダとして出力し、ユーザーに確認を促すこと

### 参考情報

- [Nablarch公式ドキュメント](https://nablarch.github.io/)
- [Nablarchアーキテクチャ概要](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html)
- [RESTful WSアーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html)
- [バッチアプリケーションアーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html)
- [nablarch-example-web (GitHub)](https://github.com/nablarch/nablarch-example-web)
- [nablarch-example-batch (GitHub)](https://github.com/nablarch/nablarch-example-batch)
