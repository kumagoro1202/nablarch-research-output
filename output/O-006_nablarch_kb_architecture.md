# Nablarch アーキテクチャ詳細 ナレッジベース

> **調査実施**: subtask_022 (cmd_007)
> **調査日**: 2026-02-01
> **対象バージョン**: Nablarch 6u3 (最新安定版)
> **ソース**: 公式ドキュメント + GitHubソースコード

---

## 目次

1. [ハンドラキューの設計パターン（アプリケーション種別ごと）](#1-ハンドラキューの設計パターン)
2. [全ハンドラ一覧と仕様](#2-全ハンドラ一覧と仕様)
3. [インターセプタの仕組み](#3-インターセプタの仕組み)
4. [コンポーネント定義（XML）の構造](#4-コンポーネント定義xmlの構造)
5. [DB接続・トランザクション管理の詳細](#5-db接続トランザクション管理の詳細)
6. [リクエスト処理フロー](#6-リクエスト処理フロー)
7. [Nablarchの内部アーキテクチャ](#7-nablarchの内部アーキテクチャ)

---

## 1. ハンドラキューの設計パターン

> **ソース**: [アーキテクチャ — Nablarch 6u3](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html)

### 1.1 ハンドラキューとは

ハンドラキューは、Nablarchアーキテクチャの核心である。リクエストやレスポンスに対する**横断的な処理（cross-cutting concerns）**を行うハンドラ群を、予め定められた順序に沿って定義したキューである。

**処理モデル**: サーブレットフィルタのチェーン実行と同様のパイプライン型処理。

```
リクエスト → [Handler1] → [Handler2] → ... → [HandlerN] → アクション
                ↑                                              |
                |        レスポンス（逆順で戻る）               |
                ← [Handler1] ← [Handler2] ← ... ← [HandlerN] ←
```

**重要な制約**: ハンドラには前後関係の依存性があり、順序を誤ると正常に動作しない。各ハンドラのドキュメントで制約を確認する必要がある。

### 1.2 Webアプリケーション用ハンドラキュー

> **ソース**: [Webアプリケーション アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html)
> **実例XML**: [nablarch-example-web/web-component-configuration.xml](https://github.com/nablarch/nablarch-example-web)

起動方式: `WebFrontController`（`javax.servlet.Filter` 実装）がリクエストを受信し、ハンドラキューに委譲。

#### 最小構成ハンドラキュー（13ハンドラ）

| 順序 | ハンドラ | クラス名 | 責務 |
|------|--------|---------|------|
| 1 | HTTP文字エンコード制御 | `nablarch.fw.web.handler.HttpCharacterEncodingHandler` | リクエスト/レスポンスの文字エンコーディング設定 |
| 2 | グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` | 実行時例外のログ出力 |
| 3 | HTTPレスポンス | `nablarch.fw.web.handler.HttpResponseHandler` | フォワード/リダイレクト/レスポンス書き込み実行 |
| 4 | セキュア | `nablarch.fw.web.handler.SecureHandler` | セキュリティレスポンスヘッダ付与 |
| 5 | マルチパートリクエスト | `nablarch.fw.web.handler.MultipartHandler` | ファイルアップロード処理 |
| 6 | セッション変数保存 | `nablarch.common.web.session.SessionStoreHandler` | セッションデータの読み書き |
| 7 | ノーマライズ | `nablarch.fw.web.handler.NormalizationHandler` | リクエストパラメータの正規化 |
| 8 | 内部フォーワード | `nablarch.fw.web.handler.ForwardingHandler` | 内部フォーワード時のハンドラ再実行 |
| 9 | HTTPエラー制御 | `nablarch.fw.web.handler.HttpErrorHandler` | エラーレスポンス生成 |
| 10 | Nablarchカスタムタグ制御 | `nablarch.common.web.handler.NablarchTagHandler` | カスタムタグの前処理 |
| 11 | データベース接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | DB接続の取得/解放 |
| 12 | トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` | トランザクション開始/コミット/ロールバック |
| 13 | ルーティングアダプタ | `nablarch.integration.router.RoutesMapping` | リクエストパスをアクションにマッピング |

#### 実プロジェクトでの構成例（nablarch-example-web）

```
1. HttpCharacterEncodingHandler
2. ThreadContextClearHandler          ← スレッドコンテキスト変数削除
3. GlobalErrorHandler
4. HttpResponseHandler
5. SecureHandler                      ← CSP, X-Frame-Options等
6. MultipartHandler
7. SessionStoreHandler
8. ThreadContextHandler               ← リクエストID等の初期化
9. HttpAccessLogHandler               ← アクセスログ出力
10. NormalizationHandler
11. ForwardingHandler
12. HttpErrorHandler                   ← ステータスコード別エラーページ
13. NablarchTagHandler
14. CsrfTokenVerificationHandler       ← CSRF対策
15. DbConnectionManagementHandler
16. TransactionManagementHandler
17. LoginUserPrincipalCheckHandler     ← カスタム: ログインチェック
18. ExampleErrorForwardHandler         ← カスタム: エラーフォワード
19. PackageMapping                     ← ディスパッチ
```

### 1.3 RESTful Webサービス用ハンドラキュー

> **ソース**: [RESTful Webサービス アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html)
> **実例XML**: [nablarch-example-rest/rest-component-configuration.xml](https://github.com/nablarch/nablarch-example-rest)

起動方式: `WebFrontController`（Webと同じServlet Filter）

#### 最小構成ハンドラキュー（7ハンドラ）

| 順序 | ハンドラ | クラス名 | 責務 |
|------|--------|---------|------|
| 1 | グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` | 実行時例外のログ出力 |
| 2 | JAX-RSレスポンス | `nablarch.fw.jaxrs.JaxRsResponseHandler` | レスポンス書き込み、エラーレスポンス処理 |
| 3 | データベース接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | DB接続取得/解放 |
| 4 | トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` | TX制御 |
| 5 | ルーターハンドラ | `nablarch.integration.router.RoutesMapping` | URIをアクションメソッドにマッピング |
| 6 | ボディ変換 | `nablarch.fw.jaxrs.BodyConvertHandler` | リクエスト/レスポンスボディ変換 |
| 7 | BeanValidation | `nablarch.fw.jaxrs.JaxRsBeanValidationHandler` | 変換済みフォームのバリデーション |

#### 実プロジェクトでの構成例（nablarch-example-rest）

```
1. HttpCharacterEncodingHandler
2. ThreadContextClearHandler
3. GlobalErrorHandler
4. JaxRsResponseHandler               ← SecureHandler内蔵（responseFinishers経由）
5. MultipartHandler
6. ThreadContextHandler
7. JaxRsAccessLogHandler
8. DbConnectionManagementHandler
9. TransactionManagementHandler
10. PathOptionsProviderRoutesMapping    ← JaxRsMethodBinderFactory使用
```

### 1.4 バッチアプリケーション用ハンドラキュー

> **ソース**: [バッチアプリケーション アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html)
> **実例XML**: [nablarch-example-batch/batch-component-configuration.xml](https://github.com/nablarch/nablarch-example-batch)

起動方式: `nablarch.fw.launcher.Main`（javaコマンドから直接起動）
コマンドライン引数: `-requestPath=fully.qualified.ClassName/REQUEST_ID`

#### 都度起動バッチ（DB接続あり）最小構成

| 順序 | ハンドラ | クラス名 | スレッド | 責務 |
|------|--------|---------|---------|------|
| 1 | ステータスコード変換 | `nablarch.fw.handler.StatusCodeConvertHandler` | メイン | 終了コード変換 |
| 2 | グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` | メイン | 例外処理・ログ |
| 3 | DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | メイン | 初期処理/終了処理用DB接続 |
| 4 | トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` | メイン | 初期処理/終了処理用TX |
| 5 | リクエストディスパッチ | `nablarch.fw.handler.RequestPathJavaPackageMapping` | メイン | アクションクラス特定 |
| 6 | マルチスレッド実行制御 | `nablarch.fw.handler.MultiThreadExecutionHandler` | メイン | サブスレッド生成・並行実行 |
| 7 | DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | サブ | 業務処理用DB接続 |
| 8 | トランザクションループ制御 | `nablarch.fw.handler.LoopHandler` | サブ | 業務TX制御 |
| 9 | データリード | `nablarch.fw.handler.DataReadHandler` | サブ | 個別レコード読込 |

**注意**: DB接続管理ハンドラとトランザクション制御ハンドラがメインスレッド用とサブスレッド用で**2回**出現する。

#### 都度起動バッチ（DB接続なし）最小構成

DB関連ハンドラを除去し、`LoopHandler`の代わりに`nablarch.fw.handler.DbLessLoopHandler`を使用。

#### 実プロジェクトでの構成例（nablarch-example-batch）

```
1. StatusCodeConvertHandler
2. ThreadContextClearHandler
3. GlobalErrorHandler
4. ThreadContextHandler
5. DbConnectionManagementHandler        ← メインスレッド用
6. TransactionManagementHandler          ← メインスレッド用
7. RequestPathJavaPackageMapping         ← ディスパッチ
8. MultiThreadExecutionHandler           ← サブスレッド分岐点
9. DbConnectionManagementHandler         ← サブスレッド用
10. LoopHandler                           ← トランザクションループ制御
11. DataReadHandler                       ← データ読込
```

### 1.5 常駐バッチ用ハンドラキュー

> **ソース**: [バッチアプリケーション アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html)

都度起動バッチに加えて、メインスレッドに以下のハンドラを追加:

| 追加ハンドラ | クラス名 | 責務 |
|------------|---------|------|
| スレッドコンテキスト変数管理 | `nablarch.common.handler.threadcontext.ThreadContextHandler` | リクエストIDなどの初期化 |
| スレッドコンテキスト変数削除 | `nablarch.common.handler.threadcontext.ThreadContextClearHandler` | 後処理でローカル変数削除 |
| リトライ | `nablarch.fw.handler.RetryHandler` | リトライ可能例外の処理 |
| プロセス常駐化 | `nablarch.fw.handler.ProcessResidentHandler` | 一定間隔での繰り返し実行 |
| プロセス停止制御 | `nablarch.fw.handler.BasicProcessStopHandler` | 停止フラグのチェック |

### 1.6 MOMメッセージング用ハンドラキュー

> **ソース**: [MOMメッセージング アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/messaging/mom/architecture.html)

起動方式: `nablarch.fw.launcher.Main`（バッチと同じ）

#### 同期応答型 最小構成（14ハンドラ）

| 順序 | ハンドラ | クラス名 | 責務 |
|------|--------|---------|------|
| 1 | ステータスコード変換 | `StatusCodeConvertHandler` | 終了コード変換 |
| 2 | グローバルエラー | `GlobalErrorHandler` | 例外処理・ログ |
| 3 | マルチスレッド実行制御 | `MultiThreadExecutionHandler` | サブスレッド並行実行 |
| 4 | リトライ | `RetryHandler` | リトライ可能例外 |
| 5 | メッセージングコンテキスト管理 | `nablarch.fw.messaging.handler.MessagingContextHandler` | MQ接続取得/解放 |
| 6 | DB接続管理 | `DbConnectionManagementHandler` | DB接続取得/解放 |
| 7 | リクエストスレッド内ループ制御 | `nablarch.fw.handler.RequestThreadLoopHandler` | 繰り返し実行制御 |
| 8 | スレッドコンテキスト変数削除 | `ThreadContextClearHandler` | ローカル変数削除 |
| 9 | スレッドコンテキスト変数管理 | `ThreadContextHandler` | リクエストIDなど初期化 |
| 10 | プロセス停止制御 | `BasicProcessStopHandler` | 停止フラグチェック |
| 11 | 電文応答制御 | `nablarch.fw.messaging.handler.MessageReplyHandler` | 応答電文をMQに送信 |
| 12 | データリード | `DataReadHandler` | 要求電文1件読込 |
| 13 | リクエストディスパッチ | `RequestPathJavaPackageMapping` | アクション決定 |
| 14 | トランザクション制御 | `TransactionManagementHandler` | TX開始/コミット/ロールバック |

#### 応答不要型

同期応答型から以下を除外:
- 電文応答制御ハンドラ（`MessageReplyHandler`）
- 再送電文制御ハンドラ（`MessageResendHandler`）

**重要**: 応答不要型ではDBとキューを1トランザクションで扱う「2相コミット制御」が必須。

### 1.7 HTTPメッセージング用ハンドラキュー

> **ソース**: [HTTPメッセージング アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/http_messaging/architecture.html)
> **注意**: RESTful Webサービスの使用が推奨されている。

起動方式: `WebFrontController`（Webと同じ）

#### 最小構成（11ハンドラ）

| 順序 | ハンドラ | クラス名 | 責務 |
|------|--------|---------|------|
| 1 | スレッドコンテキスト変数削除 | `ThreadContextClearHandler` | スレッドローカル値削除 |
| 2 | グローバルエラー | `GlobalErrorHandler` | 例外ログ出力 |
| 3 | HTTPレスポンス | `HttpResponseHandler` | レスポンス書き込み |
| 4 | スレッドコンテキスト変数管理 | `ThreadContextHandler` | リクエストID初期化 |
| 5 | HTTPメッセージングエラー制御 | `nablarch.fw.messaging.handler.HttpMessagingErrorHandler` | デフォルトボディ設定・例外対応 |
| 6 | リクエストディスパッチ | `RequestPathJavaPackageMapping` | アクション特定 |
| 7 | HTTPメッセージングリクエスト変換 | `nablarch.fw.messaging.handler.HttpMessagingRequestParsingHandler` | HTTPボディ→RequestMessage変換 |
| 8 | DB接続管理 | `DbConnectionManagementHandler` | DB接続取得/解放 |
| 9 | HTTPメッセージングレスポンス変換 | `nablarch.fw.messaging.handler.HttpMessagingResponseBuildingHandler` | エラー用レスポンス生成 |
| 10 | トランザクション制御 | `TransactionManagementHandler` | TX制御 |
| 11 | HTTPメッセージングレスポンス変換 | `HttpMessagingResponseBuildingHandler` | 最終レスポンス生成 |

### 1.8 カスタムハンドラの作成方法

`Handler<TData, TResult>` インターフェースを実装する。

```java
public class MyCustomHandler implements Handler<Object, Object> {
    @Override
    public Object handle(Object data, ExecutionContext context) {
        // 往路処理（リクエスト方向）
        doBeforeProcess();

        // 後続ハンドラに委譲
        Object result = context.handleNext(data);

        // 復路処理（レスポンス方向）
        doAfterProcess();

        return result;
    }
}
```

XMLコンポーネント定義でハンドラキューのリストに追加:

```xml
<list name="handlerQueue">
  <!-- 既存ハンドラ -->
  <component class="com.example.MyCustomHandler" />
  <!-- 既存ハンドラ -->
</list>
```

---

## 2. 全ハンドラ一覧と仕様

> **ソース**: [Nablarchの提供する標準ハンドラ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html)
> **Javadoc**: [Handler インターフェース](https://nablarch.github.io/docs/5u8/javadoc/nablarch/fw/Handler.html)

### 2.1 ハンドラインターフェース階層

**ソースコード**: [nablarch-core/Handler.java](https://github.com/nablarch/nablarch-core/blob/master/src/main/java/nablarch/fw/Handler.java)

```java
// メインインターフェース
public interface Handler<TData, TResult> {
    TResult handle(TData data, ExecutionContext context);
}

// 前処理用マーカインターフェース
public interface InboundHandleable {
    Result handleInbound(ExecutionContext context);
}

// 後処理用マーカインターフェース
public interface OutboundHandleable {
    Result handleOutbound(ExecutionContext context);
}
```

- **Handler<TData, TResult>**: パイプライン処理の各ステージで実装する基本インターフェース
- **InboundHandleable**: 前処理（往路）を実行可能なマーカインターフェース
- **OutboundHandleable**: 後処理（復路）を実行可能なマーカインターフェース

`InboundHandleable` / `OutboundHandleable` を実装するハンドラ（例: `TransactionManagementHandler`, `DbConnectionManagementHandler`）は、往路でリソース取得、復路でリソース解放を分離して実行できる。

### 2.2 共通ハンドラ

| ハンドラ名 | クラス（パッケージ省略） | 責務 |
|-----------|----------------------|------|
| グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` | 未捕捉例外のログ出力 |
| 出力ファイル開放 | `nablarch.fw.handler.FileRecordWriterDisposeHandler` | 出力ファイルリソース解放 |
| リクエストディスパッチ | `nablarch.fw.handler.RequestPathJavaPackageMapping` | リクエストパスからアクション特定 |
| リクエストハンドラエントリ | `nablarch.fw.handler.RequestHandlerEntry` | requestPatternに基づく条件分岐 |
| スレッドコンテキスト変数管理 | `nablarch.common.handler.threadcontext.ThreadContextHandler` | リクエストID等の初期化 |
| スレッドコンテキスト変数削除 | `nablarch.common.handler.threadcontext.ThreadContextClearHandler` | スレッドローカル変数の全削除 |
| DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | DB接続の取得/解放 |
| トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` | TX開始/コミット/ロールバック |
| 認可チェック | `nablarch.common.handler.PermissionCheckHandler` | アクセス権限検証 |
| サービス提供可否チェック | `nablarch.common.handler.ServiceAvailabilityCheckHandler` | サービス利用可否判定 |

### 2.3 Webアプリケーション専用ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| HTTP文字エンコード制御 | `nablarch.fw.web.handler.HttpCharacterEncodingHandler` | 文字エンコーディング設定 |
| HTTPレスポンス | `nablarch.fw.web.handler.HttpResponseHandler` | フォワード/リダイレクト/書き込み |
| セキュア | `nablarch.fw.web.handler.SecureHandler` | セキュリティヘッダ付与 |
| HTTPエラー制御 | `nablarch.fw.web.handler.HttpErrorHandler` | エラーページ生成 |
| 内部フォーワード | `nablarch.fw.web.handler.ForwardingHandler` | 内部フォーワード処理 |
| リソースマッピング | `nablarch.fw.web.handler.ResourceMapping` | 静的リソースマッピング |
| Nablarchカスタムタグ制御 | `nablarch.common.web.handler.NablarchTagHandler` | JSPカスタムタグ前処理 |
| HTTPアクセスログ | `nablarch.common.web.handler.HttpAccessLogHandler` | アクセスログ出力 |
| マルチパートリクエスト | `nablarch.fw.web.handler.MultipartHandler` | ファイルアップロード |
| セッション変数保存 | `nablarch.common.web.session.SessionStoreHandler` | セッション読み書き |
| HTTPリクエストディスパッチ | `nablarch.fw.web.handler.HttpRequestJavaPackageMapping` | HTTPリクエスト用ディスパッチ |
| HTTPリライト | `nablarch.fw.web.handler.HttpRewriteHandler` | URLリライト |
| POST再送信防止 | `nablarch.fw.web.handler.PostResubmitPreventHandler` | POST再送信防止 |
| ノーマライズ | `nablarch.fw.web.handler.NormalizationHandler` | パラメータ正規化 |
| セッション並行アクセス | `nablarch.fw.web.handler.SessionConcurrentAccessHandler` | セッション並行アクセス制御 |
| ホットデプロイ | `nablarch.fw.web.handler.HotDeployHandler` | 開発時ホットデプロイ |
| CSRFトークン検証 | `nablarch.fw.web.handler.CsrfTokenVerificationHandler` | CSRF対策 |
| ヘルスチェックエンドポイント | `nablarch.fw.web.handler.HealthCheckEndpointHandler` | ヘルスチェック |

### 2.4 RESTful Webサービス専用ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| リクエストボディ変換 | `nablarch.fw.jaxrs.BodyConvertHandler` | リクエスト/レスポンスボディ変換 |
| BeanValidation | `nablarch.fw.jaxrs.JaxRsBeanValidationHandler` | フォームバリデーション |
| JAX-RSレスポンス | `nablarch.fw.jaxrs.JaxRsResponseHandler` | レスポンス書き込み/エラーハンドリング |
| CORSプリフライト | `nablarch.fw.jaxrs.CorsPreflightRequestHandler` | CORSプリフライトリクエスト対応 |
| HTTPアクセスログ(REST用) | `nablarch.fw.jaxrs.JaxRsAccessLogHandler` | RESTアクセスログ |

### 2.5 HTTPメッセージング専用ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| エラー制御 | `nablarch.fw.messaging.handler.HttpMessagingErrorHandler` | エラー時デフォルトボディ設定 |
| リクエスト変換 | `nablarch.fw.messaging.handler.HttpMessagingRequestParsingHandler` | HTTPボディ→RequestMessage |
| レスポンス変換 | `nablarch.fw.messaging.handler.HttpMessagingResponseBuildingHandler` | ResponseMessage→HTTPレスポンス |

### 2.6 スタンドアローン型共通ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| 共通起動ランチャ | `nablarch.fw.launcher.Main` | アプリケーション起動 |
| データリード | `nablarch.fw.handler.DataReadHandler` | データ1件ずつ読込 |
| リトライ | `nablarch.fw.handler.RetryHandler` | リトライ可能例外の処理 |
| ステータスコード変換 | `nablarch.fw.handler.StatusCodeConvertHandler` | 終了コード変換 |
| プロセス停止制御 | `nablarch.fw.handler.BasicProcessStopHandler` | 停止フラグチェック |
| プロセス多重起動防止 | `nablarch.fw.handler.DuplicateProcessCheckHandler` | 多重起動防止 |
| マルチスレッド実行制御 | `nablarch.fw.handler.MultiThreadExecutionHandler` | サブスレッド生成 |
| リクエストスレッド内ループ制御 | `nablarch.fw.handler.RequestThreadLoopHandler` | ループ制御 |

### 2.7 バッチアプリケーション専用ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| プロセス常駐化 | `nablarch.fw.handler.ProcessResidentHandler` | 一定間隔繰り返し実行 |
| トランザクションループ制御 | `nablarch.fw.handler.LoopHandler` | 業務トランザクションループ |
| ループ制御（DBなし） | `nablarch.fw.handler.DbLessLoopHandler` | DB不要のループ制御 |

### 2.8 MOMメッセージング専用ハンドラ

| ハンドラ名 | クラス | 責務 |
|-----------|------|------|
| メッセージングコンテキスト管理 | `nablarch.fw.messaging.handler.MessagingContextHandler` | MQ接続取得/解放 |
| 再送電文制御 | `nablarch.fw.messaging.handler.MessageResendHandler` | 再送電文の制御 |
| 電文応答制御 | `nablarch.fw.messaging.handler.MessageReplyHandler` | 応答電文のMQ送信 |

---

## 3. インターセプタの仕組み

> **ソースコード**: [nablarch-core/Interceptor.java](https://github.com/nablarch/nablarch-core/blob/v5-main/src/main/java/nablarch/fw/Interceptor.java)
> **Javadoc**: [Interceptor](https://nablarch.github.io/docs/5u9/javadoc/nablarch/fw/Interceptor.html)

### 3.1 インターセプタとは

インターセプタは、**実行時に動的にハンドラキューに追加されるハンドラ**である。ハンドラキューに静的に定義するハンドラとは異なり、特定のリクエストにのみ処理を追加したり、リクエストごとに設定値を切り替える場合に使用する。

**設計思想**: 親クラスで共通処理を実装するのではなく、個別のハンドラ/インターセプタとして実装することを推奨。親クラスの肥大化を防ぎ、テスト容易性を向上させるため。

### 3.2 インターセプタの仕組み（ソースコード分析）

```
@Interceptor メタアノテーション
      │
      ▼
カスタムアノテーション（@AroundAdvice等）
      │
      ▼ Interceptor.Factory.wrap(handler)
      │
   handleメソッドのアノテーションを収集
      │
      ▼ アノテーションをソート（SystemRepository "interceptorsOrder"）
      │
      ▼ 各アノテーションに対応するInterceptor.Implでラップ
      │
   ラップされたHandler（チェーン構造）
```

#### 実行の流れ

1. `Interceptor.Factory.wrap(handler)` でhandlerの`handle`メソッドに付与されたアノテーションを収集
2. `SystemRepository`の`interceptorsOrder`キーで定義された順序にソート
3. 各インターセプタに対応する`Interceptor.Impl`サブクラスでラップ
4. `wrapped.handle()` 呼び出し時に各インターセプタが順次実行
5. 最後に元のハンドラの`handle()`が呼ばれる

### 3.3 カスタムインターセプタの作成方法

```java
// 1. @Interceptorメタアノテーションを付与したアノテーションを作成
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Interceptor(AroundAdvice.Impl.class)  // 実装クラスを指定
public @interface AroundAdvice {

    // 2. Interceptor.Implを継承した実装クラスを内部クラスとして定義
    public static class Impl
            extends Interceptor.Impl<HttpRequest, HttpResponse, AroundAdvice> {

        @Override
        public HttpResponse handle(HttpRequest req, ExecutionContext ctx) {
            // 前処理
            doBeforeAdvice(req, ctx);

            // 元のハンドラに委譲
            HttpResponse res = getOriginalHandler().handle(req, ctx);

            // 後処理
            doAfterAdvice(req, ctx);
            return res;
        }
    }
}
```

**Interceptor.Impl<TData, TResult, T extends Annotation> の主要メソッド**:
- `getOriginalHandler()` — ラップ対象のハンドラを取得
- `getInterceptor()` — 処理対象のアノテーションインスタンスを取得
- `setOriginalHandler()` — ラップ対象を設定（Factoryが使用）
- `setInterceptor()` — アノテーションを設定（Factoryが使用）

### 3.4 標準インターセプタ一覧

| インターセプタ | クラス | 責務 |
|-------------|------|------|
| InjectForm | `nablarch.common.web.interceptor.InjectForm` | バリデーション成功後のフォームをリクエストスコープに格納。エラー時は`ApplicationException`送出 |
| OnError | `nablarch.fw.web.interceptor.OnError` | 特定の例外発生時のレスポンスを指定 |
| OnErrors | `nablarch.fw.web.interceptor.OnErrors` | 複数の例外に対するレスポンス指定 |
| OnDoubleSubmission | `nablarch.common.web.interceptor.OnDoubleSubmission` | 二重サブミットチェック |
| UseToken | `nablarch.common.web.token.UseToken` | トークン発行 |

### 3.5 インターセプタの実行順序設定

実行順序は`SystemRepository`で明示的に定義する必要がある。未定義の場合、JVMバージョンによって順序が変わる可能性がある。

```xml
<!-- XMLコンポーネント定義での実行順定義 -->
<list name="interceptorsOrder">
  <value>nablarch.common.web.interceptor.OnDoubleSubmission</value>
  <value>nablarch.common.web.token.UseToken</value>
  <value>nablarch.fw.web.interceptor.OnErrors</value>
  <value>nablarch.fw.web.interceptor.OnError</value>
  <value>nablarch.common.web.interceptor.InjectForm</value>
</list>
```

**重要**: `interceptorsOrder`に定義されていないインターセプタを使用すると、`IllegalArgumentException`が送出される。

### 3.6 ハンドラとインターセプタの関係

| 観点 | ハンドラ | インターセプタ |
|------|---------|-------------|
| 定義場所 | XMLコンポーネント定義（静的） | アクションメソッドのアノテーション（動的） |
| 適用範囲 | 全リクエスト | 特定のアクションメソッドのみ |
| 追加タイミング | 起動時に固定 | 実行時にFactory.wrap()で動的追加 |
| 用途 | 横断的処理（認証、TX、ログ等） | 特定処理の前後処理（バリデーション、トークン等） |

---

## 4. コンポーネント定義（XML）の構造

> **ソース**: [システムリポジトリ — Nablarch 6u3](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html)
> **Javadoc**: [ComponentConfiguration](https://nablarch.github.io/docs/6/javadoc/nablarch/core/repository/di/config/xml/schema/ComponentConfiguration.html)

### 4.1 XML基本構造

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component-configuration
    xmlns="http://tis.co.jp/nablarch/component-configuration"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://tis.co.jp/nablarch/component-configuration
        https://nablarch.github.io/schema/component-configuration.xsd">

  <!-- ここにコンポーネント定義を記述 -->

</component-configuration>
```

### 4.2 主要要素

#### `<component>` — コンポーネント定義

```xml
<component name="myComponent" class="com.example.MyClass">
  <property name="propertyName" value="literalValue" />
  <property name="refProperty" ref="anotherComponent" />
</component>
```

| 属性 | 必須 | 説明 |
|------|------|------|
| `name` | 任意 | コンポーネント名（`SystemRepository.get()`で取得する際のキー） |
| `class` | 必須 | 対象クラスのFQCN |
| `autowireType` | 任意 | `ByType`(デフォルト), `ByName`, `None` |

**スコープ**: 構築されるオブジェクトは**シングルトン**。アプリケーションライフサイクルで1回だけ生成される。

#### `<property>` — プロパティインジェクション（setter DI）

```xml
<property name="propertyName" value="literalValue" />   <!-- リテラル値 -->
<property name="propertyName" ref="componentName" />     <!-- 参照 -->
<property name="propertyName">                           <!-- インライン定義 -->
  <component class="com.example.InnerClass" />
</property>
```

**対応リテラル型**: `String`, `String[]`, `Integer`/`int`, `Integer[]`/`int[]`, `Long`/`long`, `Boolean`/`boolean`

#### `<list>` — リスト定義

```xml
<list name="handlers">
  <value>item1</value>                                <!-- 文字列値 -->
  <component class="com.example.Handler1" />          <!-- インラインコンポーネント -->
  <component-ref name="existingComponent" />          <!-- 既存コンポーネント参照 -->
</list>
```

#### `<map>` — マップ定義

```xml
<map name="settings">
  <entry key="name" value="literalValue" />
  <entry key="bean">
    <value-component class="com.example.Bean" />
  </entry>
</map>
```

### 4.3 ファイル管理

#### `<import>` — 他XMLファイルのインポート

```xml
<import file="nablarch/core/db-base.xml" />
<import file="com/example/app/web/common/authentication/authenticator.xml" />
```

#### `<config-file>` — プロパティファイルの読み込み

```xml
<config-file file="env.properties" />
<config-file file="common.properties" />
```

### 4.4 環境ごとの設定切り替え

プロパティファイルの値を`${key}`記法で参照:

```xml
<!-- env.properties: db.url=jdbc:h2:./testdb -->
<component name="dataSource" class="...BasicDataSource">
  <property name="url" value="${db.url}" />
  <property name="user" value="${db.user}" />
  <property name="password" value="${db.password}" />
</component>
```

Mavenプロファイル（`dev`, `prod`）により、環境別のプロパティファイルを切り替える。

### 4.5 Autowire設定

| モード | 動作 |
|-------|------|
| `ByType`（デフォルト） | 型が一致する唯一のコンポーネントを自動注入 |
| `ByName` | プロパティ名とコンポーネント名が一致するものを注入 |
| `None` | 明示的なインジェクションのみ（推奨） |

### 4.6 初期化と破棄

```xml
<!-- 初期化（Initializableインターフェース実装クラス） -->
<component name="initializer"
    class="nablarch.core.repository.initialization.BasicApplicationInitializer">
  <property name="initializeList">
    <list>
      <component-ref name="dbStore" />
      <component-ref name="expiration" />
      <component-ref name="packageMapping" />
    </list>
  </property>
</component>

<!-- 破棄（Disposableインターフェース実装クラス） -->
<component name="disposer"
    class="nablarch.core.repository.disposal.BasicApplicationDisposer">
  <property name="disposableList">
    <list>
      <component-ref name="dataSource" />
    </list>
  </property>
</component>
```

### 4.7 アノテーションベースのコンポーネント定義

```java
@SystemRepositoryComponent
public class MyService {
    @ConfigValue("${app.timeout}")
    private int timeout;

    @ComponentRef
    private MyRepository repository;
}
```

### 4.8 システムリポジトリへのロードとアクセス

```java
// ロード（フレームワークが自動実行。通常はユーザが実装する必要はない）
XmlComponentDefinitionLoader loader =
    new XmlComponentDefinitionLoader("web-boot.xml");
SystemRepository.load(new DiContainer(loader));

// アクセス
MyComponent comp = SystemRepository.get("myComponent");
```

### 4.9 実プロジェクトのXML構成例

**Webアプリケーション（nablarch-example-web）のインポート構造**:

```
web-component-configuration.xml  ← ルートファイル
  ├── nablarch/schema-config.xml            ← テーブル定義
  ├── JSR310.xml                            ← JSR310対応
  ├── env.properties                        ← 環境設定
  ├── common.properties                     ← 共通設定
  ├── validation.xml                        ← バリデーション設定
  ├── passwordEncryptor.xml                 ← パスワード暗号化
  ├── authenticator.xml                     ← 認証設定
  ├── authorization-session.xml             ← 認可設定
  ├── nablarch/common/dao.xml               ← ユニバーサルDAO
  ├── nablarch/core/db-base.xml             ← DB基本設定
  ├── db.xml                                ← DB接続設定
  ├── nablarch/webui/interceptors.xml       ← インターセプタ実行順
  ├── error-page-for-webui.xml              ← エラーページ設定
  ├── filepath-for-webui.xml                ← ファイルパス設定
  ├── nablarch/webui/multipart.xml          ← マルチパート設定
  ├── nablarch-tag.xml                      ← カスタムタグ設定
  ├── double-submission.xml                 ← 二重サブミット防止
  ├── request-mapper-for-webui.xml          ← リクエストマッピング
  ├── nablarch/webui/session-store.xml      ← セッションストア設定
  └── threadcontext-for-webui-in-sessionstore.xml ← スレッドコンテキスト
```

---

## 5. DB接続・トランザクション管理の詳細

> **ソース**:
> - [データベースアクセス](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database_management.html)
> - [ユニバーサルDAO](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html)
> - [データベースアクセス(JDBCラッパー)](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html)

### 5.1 DB接続プール設定

2種類の接続方法:

**1. DataSourceベース**

```xml
<component name="connectionFactory"
    class="nablarch.core.db.connection.BasicDbConnectionFactoryForDataSource">
  <property name="dataSource" ref="dataSource" />
  <property name="transactionFactory" ref="transactionFactory" />
</component>
```

**2. JNDIベース**

```xml
<component name="connectionFactory"
    class="nablarch.core.db.connection.BasicDbConnectionFactoryForJndi">
  <property name="jndiResourceName" value="java:comp/env/jdbc/myDB" />
  <property name="transactionFactory" ref="transactionFactory" />
</component>
```

独自の接続方法は`ConnectionFactorySupport`を継承して実装可能。

### 5.2 トランザクション境界の制御

**ハンドラベースのトランザクション制御**:

`DbConnectionManagementHandler` + `TransactionManagementHandler` をセットで使用。

```xml
<!-- DB接続管理ハンドラ -->
<component name="dbConnectionManagementHandler"
    class="nablarch.common.handler.DbConnectionManagementHandler">
  <property name="connectionFactory" ref="connectionFactory" />
</component>

<!-- トランザクション制御ハンドラ -->
<component name="transactionManagementHandler"
    class="nablarch.common.handler.TransactionManagementHandler">
  <property name="transactionFactory" ref="transactionFactory" />
</component>
```

**動作**: `TransactionManagementHandler` は `InboundHandleable` と `OutboundHandleable` を実装しており:
- **Inbound（往路）**: トランザクション開始
- **Outbound（復路）**: `isProcessSucceeded`の結果に基づきコミット or ロールバック

### 5.3 複数DB接続の設定

ハンドラキュー上に`DbConnectionManagementHandler`を複数配置:

```xml
<list name="handlerQueue">
  <!-- DB1の接続管理 -->
  <component class="nablarch.common.handler.DbConnectionManagementHandler">
    <property name="connectionFactory" ref="connectionFactory1" />
  </component>
  <!-- DB2の接続管理 -->
  <component class="nablarch.common.handler.DbConnectionManagementHandler">
    <property name="connectionFactory" ref="connectionFactory2" />
    <property name="dbConnectionName" value="db2" />
  </component>
  ...
</list>
```

### 5.4 個別トランザクション

業務処理が失敗しても確定させたいDB操作（ログ記録等）向け:

```xml
<component name="find-persons-transaction"
    class="nablarch.core.db.transaction.SimpleDbTransactionManager">
  <property name="connectionFactory" ref="connectionFactory" />
  <property name="transactionFactory" ref="transactionFactory" />
  <property name="dbTransactionName" value="find-persons-transaction" />
</component>
```

```java
private static final class FindPersonsTransaction
        extends UniversalDao.Transaction {
    private EntityList<Person> persons;

    FindPersonsTransaction() {
        super("find-persons-transaction");
    }

    @Override
    protected void execute() {
        persons = UniversalDao.findAllBySqlFile(
            Person.class, "FIND_PERSONS");
    }
}
```

### 5.5 ユニバーサルDAOの内部アーキテクチャ

内部で`データベースアクセス(JDBCラッパー)`を使用する簡易O/Rマッパー。

**コンポーネント設定**:
```xml
<component name="daoContextFactory"
    class="nablarch.common.dao.BasicDaoContextFactory" />
```

**依存**: `com.nablarch.framework:nablarch-common-dao`

#### Entity定義

Jakarta Persistenceアノテーションを使用:

| アノテーション | 用途 |
|-------------|------|
| `@Entity` | テーブル対応クラス |
| `@Table` | スキーマ・テーブル名指定 |
| `@Column` | カラム名指定 |
| `@Id` | 主キー（複合キーは複数指定） |
| `@Version` | 楽観的ロック用バージョンカラム |
| `@GeneratedValue` | 自動採番（AUTO, IDENTITY, SEQUENCE, TABLE） |
| `@SequenceGenerator` | シーケンス採番設定 |
| `@TableGenerator` | テーブル採番設定 |
| `@Temporal` | Date/Calendarマッピング |

#### 主要機能

```java
// CRUD（SQL自動生成）
UniversalDao.insert(entity);
UniversalDao.update(entity);
UniversalDao.delete(entity);
UniversalDao.findById(User.class, id);
UniversalDao.findAll(User.class);

// バッチ実行
UniversalDao.batchInsert(entityList);
UniversalDao.batchUpdate(entityList);
UniversalDao.batchDelete(entityList);

// SQLファイルでの検索
UniversalDao.findAllBySqlFile(User.class, "FIND_BY_NAME");
UniversalDao.findAllBySqlFile(User.class, "sample.entity.Member#FIND_BY_NAME");

// ページング
EntityList<User> users = UniversalDao.per(20).page(1)
    .findAllBySqlFile(User.class, "FIND_ALL");
Pagination pagination = users.getPagination();

// 遅延ロード
try (DeferredEntityList<User> users =
        (DeferredEntityList<User>) UniversalDao.defer()
            .findAllBySqlFile(User.class, "FIND_BY_NAME")) {
    for (User user : users) { /* 処理 */ }
}

// 楽観的ロック（@Version付きEntityで自動実行）
UniversalDao.update(user);  // OptimisticLockException発生の可能性
```

### 5.6 SQLファイルベースのDB操作

#### SQLファイルのルール

- クラスパス上に配置
- ファイル名: `ClassName.sql`（対応するクラスと同じ名前）
- 1ファイルに複数SQL定義可能
- SQL IDで識別: `SQL_ID = ClassName#sqlId`
- 名前付きパラメータ: `:propertyName`
- コメント: `--`

#### 動的SQL構文

**1. 可変条件（$if）**
```sql
SELECT * FROM USER
WHERE
  $if(userName) {USER_NAME = :userName}
  AND $if(age) {AGE = :age}
```
→ プロパティがnull/空の場合、その条件が除外される

**2. IN句の配列展開**
```sql
SELECT * FROM USER WHERE STATUS IN (:status[])
```
→ Collection/配列プロパティを自動展開

**3. 動的ORDER BY（$sort）**
```sql
SELECT * FROM USER
$sort(sortKey) {
  (userName USER_NAME)
  (age AGE DESC)
}
```
→ 実行時のパラメータに基づきソート条件を切替

### 5.7 Dialect（データベース方言）

データベース固有の動作を抽象化:

| 機能 | 説明 |
|------|------|
| IDENTITY/シーケンスサポート | DB固有の自動採番方式 |
| ページネーションSQL生成 | DB固有のLIMIT/OFFSET構文 |
| 例外分類 | 重複制約エラー、タイムアウト等の判定 |
| 接続検証クエリ | DB固有のpingクエリ |

主なDialect実装: `DefaultDialect`, `H2Dialect`, `OracleDialect`, `PostgreSQLDialect` など。

---

## 6. リクエスト処理フロー

### 6.1 Webアプリケーションの処理フロー

```
[クライアント]
    │ HTTP Request
    ▼
[WebFrontController] ← javax.servlet.Filter実装
    │ NablarchServletContextListenerが初期化済み
    ▼
[ハンドラキュー]
    │
    ├── HttpCharacterEncodingHandler → エンコーディング設定
    ├── GlobalErrorHandler → 例外キャッチ準備
    ├── HttpResponseHandler → レスポンス変換準備
    ├── SecureHandler → セキュリティヘッダ設定
    ├── SessionStoreHandler → セッション変数ロード
    ├── NormalizationHandler → パラメータ正規化
    ├── DbConnectionManagementHandler → DB接続取得
    ├── TransactionManagementHandler → TX開始
    ├── RoutesMapping(DispatchHandler) → URIからアクション特定
    │       ▼
    │   [アクションクラス]
    │       │ @InjectForm → バリデーション＆フォームインジェクション
    │       │ @OnError/@OnErrors → エラーハンドリング
    │       │ @OnDoubleSubmission → 二重サブミットチェック
    │       ▼
    │   業務ロジック実行（Form, Entity使用）
    │       │
    │       ▼
    │   HttpResponse返却（JSPパス、リダイレクト先等）
    │
    ├── TransactionManagementHandler → コミット/ロールバック
    ├── DbConnectionManagementHandler → DB接続解放
    ├── SessionStoreHandler → セッション変数保存
    ├── HttpResponseHandler → JSPフォワード/リダイレクト実行
    │
    ▼
[クライアント] ← HTTP Response
```

### 6.2 ディスパッチの仕組み

**Webアプリケーション**:
- `RoutesMapping`（`nablarch-router-adaptor`）がURIパターンからアクションクラスを特定
- アクションクラスをハンドラキューの末尾に追加

**バッチアプリケーション**:
- `RequestPathJavaPackageMapping` がコマンドライン引数の`-requestPath`からアクション特定
- 形式: `-requestPath=com.example.batch.MyBatchAction/REQUEST_ID`

**MOMメッセージング**:
- 受信電文のリクエストIDからアクション特定

### 6.3 セッション管理

> **ソース**: [セッション変数保存](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html)

3種類のセッションストア:

| ストア | 保存先 | 用途 | クラス |
|-------|-------|------|-------|
| DBストア | データベース（USER_SESSIONテーブル） | 復元可能、ヒープ圧迫なし | `nablarch.common.web.session.store.DbStore` |
| HIDDENストア | クライアント（hiddenタグ） | 複数タブ対応 | `nablarch.common.web.session.store.HiddenStore` |
| HTTPセッションストア | アプリサーバメモリ | 認証情報等 | `nablarch.common.web.session.store.HttpSessionStore` |

```xml
<component name="sessionManager" class="nablarch.common.web.session.SessionManager">
  <property name="defaultStoreName" value="db"/>
  <property name="availableStores">
    <list>
      <component class="nablarch.common.web.session.store.HiddenStore"/>
      <component-ref name="dbStore"/>
      <component class="nablarch.common.web.session.store.HttpSessionStore"/>
    </list>
  </property>
</component>
```

処理フロー:
1. **往路**: `SessionStoreHandler`がクッキー(`NABLARCH_SID`)からセッションIDを取得、セッション変数をロード
2. **業務処理**: `SessionUtil.put(ctx, "key", value, "db")` でセッション変数を操作
3. **復路**: `SessionStoreHandler`がセッション変数を保存

### 6.4 バリデーションの実行タイミング

**Webアプリケーション**: `@InjectForm`インターセプタがアクションメソッド実行前にバリデーション実行

```java
@InjectForm(form = UserForm.class, prefix = "form")
@OnError(type = ApplicationException.class,
    path = "/WEB-INF/view/user/input.jsp")
public HttpResponse confirm(HttpRequest request, ExecutionContext context) {
    // バリデーション済みのフォームがリクエストスコープに格納済み
    UserForm form = context.getRequestScopedVar("form");
    ...
}
```

**RESTful Webサービス**: `JaxRsBeanValidationHandler`がリソースメソッド実行前にバリデーション

```java
@Produces(MediaType.APPLICATION_JSON)
@Valid  // ← この@Validでバリデーション有効化
public HttpResponse post(UserForm form) { ... }
```

**重要な注意点**:
- Bean Validationのプロパティ型は原則`String`にすべき（String以外だと変換エラーが先に発生）
- 相関バリデーションの実行順は保証されない
- DB相関チェックはバリデーション後に業務アクション内で実施すべき（SQLインジェクション防止）

### 6.5 レスポンス生成

**Webアプリケーション**:
```java
// JSPフォーワード
return new HttpResponse("/WEB-INF/view/user/list.jsp");
// リダイレクト
return new HttpResponse(303, "redirect://user/list");
```

`HttpResponseHandler`がHttpResponseのステータスコードとコンテンツパスに基づき:
- `2xx` + パスあり → JSPフォーワード/静的コンテンツ書き込み
- `3xx` → リダイレクト
- `4xx`/`5xx` → `HttpErrorHandler`でエラーページ表示

**RESTful Webサービス**:
- `BodyConvertHandler`がDTO/Entityを JSON/XML に変換
- `JaxRsResponseHandler`がHTTPレスポンスを生成

---

## 7. Nablarchの内部アーキテクチャ

> **ソース**: [GitHub - nablarch/nablarch](https://github.com/nablarch/nablarch)
> **ソース**: [Mavenアーキタイプ構成](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/blank_project/MavenModuleStructures/index.html)

### 7.1 フレームワークのモジュール構成

Nablarchは**高度にモジュール化された設計**で、各モジュールが独立したMavenアーティファクトとして管理されている。グループID: `com.nablarch.framework`

#### Nablarch Core（基盤モジュール）

| モジュール | artifactId | 責務 |
|----------|-----------|------|
| コア | `nablarch-core` | Handler, ExecutionContext, Interceptor, SystemRepository等の基盤 |
| JavaBeans | `nablarch-core-beans` | Beanユーティリティ |
| リポジトリ | `nablarch-core-repository` | DIコンテナ、XMLコンポーネント定義 |
| トランザクション | `nablarch-core-transaction` | トランザクション管理 |
| JDBC | `nablarch-core-jdbc` | JDBCユーティリティ |
| メッセージ | `nablarch-core-message` | エラーメッセージ、警告等 |
| バリデーション | `nablarch-core-validation` | 入力バリデーション |
| Bean Validation | `nablarch-core-validation-ee` | Jakarta Bean Validation準拠バリデーション |
| データフォーマット | `nablarch-core-dataformat` | ファイルフォーマット管理 |
| ログ | `nablarch-core-applog` | ログ出力 |

#### Nablarch Framework（実行制御基盤）

| モジュール | artifactId | 責務 |
|----------|-----------|------|
| ハンドラ | `nablarch-fw` | 共通ハンドラ実装 |
| Web | `nablarch-fw-web` | Web基盤、HTTPハンドラ |
| Web DBストア | `nablarch-fw-web-dbstore` | DBベースセッションストア |
| Web カスタムタグ | `nablarch-fw-web-tag` | JSPカスタムタグライブラリ |
| Web ホットデプロイ | `nablarch-fw-web-hotdeploy` | 開発時ホットデプロイ |
| Web 拡張 | `nablarch-fw-web-extension` | ファイルアップ/ダウンロード |
| Web 二重サブミット | `nablarch-fw-web-doublesubmit-jdbc` | JDBC二重サブミット防止 |
| Jakarta Batch | `nablarch-fw-batch-ee` | Jakarta Batch準拠バッチ |
| JAX-RS | `nablarch-fw-jaxrs` | RESTful Webサービス |
| スタンドアローン | `nablarch-fw-standalone` | スタンドアローンアプリ起動点 |
| バッチ | `nablarch-fw-batch` | バッチ用ハンドラ/アクション |
| メッセージング | `nablarch-fw-messaging` | メッセージングアーキテクチャ |
| HTTPメッセージング | `nablarch-fw-messaging-http` | HTTPメッセージング |
| MOMメッセージング | `nablarch-fw-messaging-mom` | MOMメッセージング |

#### Nablarch Common Component（共通部品）

| モジュール | artifactId | 責務 |
|----------|-----------|------|
| DAO | `nablarch-common-dao` | ユニバーサルDAO |
| ID生成 | `nablarch-common-idgenerator` | ID採番 |
| ID生成(JDBC) | `nablarch-common-idgenerator-jdbc` | JDBC ID採番 |
| JDBC共通 | `nablarch-common-jdbc` | DB接続ハンドラ |
| コード管理 | `nablarch-common-code` | 分類コード管理 |
| コード管理(JDBC) | `nablarch-common-code-jdbc` | OTLT方式コード管理 |
| 認証 | `nablarch-common-auth` | 認証基盤 |
| 認証(JDBC) | `nablarch-common-auth-jdbc` | JDBC認証 |
| 認証(セッション) | `nablarch-common-auth-session` | セッションベース認証 |
| 排他制御 | `nablarch-common-exclusivecontrol` | ロック機構 |
| 排他制御(JDBC) | `nablarch-common-exclusivecontrol-jdbc` | JDBCロック |
| 暗号化 | `nablarch-common-encryption` | 暗号化 |
| 日付管理 | `nablarch-common-date` | システム日付/業務日付 |
| データバインド | `nablarch-common-databind` | Bean/Collection変換 |
| メール送信 | `nablarch-mail-sender` | メール送信 |

#### Nablarch Adaptor（アダプタ）

| モジュール | artifactId | 対象 |
|----------|-----------|------|
| ルーター | `nablarch-router-adaptor` | http-request-router |
| JAX-RS | `nablarch-jaxrs-adaptor` | Jakarta RESTful WS |
| IBM MQ | `nablarch-wmq-adaptor` | IBM MQ |
| SLF4J | `nablarch-slf4j-adaptor` | SLF4J |
| Doma | `nablarch-doma-adaptor` | Domaフレームワーク |
| Thymeleaf(Web) | `nablarch-web-thymeleaf-adaptor` | Thymeleafテンプレート |
| Lettuce | `nablarch-lettuce-adaptor` | Redis(Lettuce)セッション |
| Micrometer | `nablarch-micrometer-adaptor` | Micrometer |

### 7.2 コアモジュール間の依存関係

```
nablarch-core（基盤: Handler, ExecutionContext, Interceptor）
    │
    ├── nablarch-core-repository（DIコンテナ、SystemRepository）
    │       └── nablarch-core に依存
    │
    ├── nablarch-core-transaction（トランザクション管理）
    │       └── nablarch-core に依存
    │
    ├── nablarch-core-jdbc（JDBCユーティリティ）
    │       ├── nablarch-core に依存
    │       └── nablarch-core-transaction に依存
    │
    ├── nablarch-fw（共通ハンドラ実装）
    │       └── nablarch-core に依存
    │
    ├── nablarch-fw-web（Web基盤）
    │       ├── nablarch-core に依存
    │       ├── nablarch-fw に依存
    │       └── javax.servlet-api に依存
    │
    ├── nablarch-fw-standalone（スタンドアローン起動）
    │       ├── nablarch-core に依存
    │       └── nablarch-fw に依存
    │
    └── nablarch-common-jdbc（DB接続ハンドラ）
            ├── nablarch-core に依存
            ├── nablarch-core-jdbc に依存
            └── nablarch-core-transaction に依存
```

### 7.3 拡張ポイント（Extension Point）の設計

Nablarchは以下の拡張ポイントを提供:

| 拡張ポイント | 方法 | 例 |
|------------|------|-----|
| **カスタムハンドラ** | `Handler<TData, TResult>`実装 | 独自認証ハンドラ |
| **カスタムインターセプタ** | `@Interceptor`メタアノテーション + `Interceptor.Impl`継承 | 独自前処理 |
| **カスタムDataReader** | `DataReader`インターフェース実装 | 独自データソース読込 |
| **カスタムDialect** | `DefaultDialect`継承 | 未サポートDB対応 |
| **カスタムBodyConverter** | `BodyConverter`インターフェース実装 | 独自シリアライズ |
| **カスタムConnectionFactory** | `ConnectionFactorySupport`継承 | 独自DB接続方式 |
| **コンポーネント差し替え** | XMLでcomponent再定義 | フレームワーク動作変更 |
| **アダプタ** | 各アダプタモジュール | Doma, Thymeleaf, Redis等 |

### 7.4 カスタマイズの仕組み

**XMLコンポーネント定義による差し替え**:
同じ`name`で再定義すると、後から読み込まれた定義が優先される。フレームワークのデフォルト動作を変更可能。

**ハンドラキューの構成変更**:
XMLの`handlerQueue`リストを編集することで、処理パイプラインを自由にカスタマイズ可能。

**Mavenアーキタイプによるプロジェクト生成**:

| アーキタイプ | artifactId | 用途 |
|-----------|-----------|------|
| Web | `nablarch-web-archetype` | Webアプリ |
| JAX-RS | `nablarch-jaxrs-archetype` | RESTfulサービス |
| バッチ | `nablarch-batch-archetype` | Nablarchバッチ |
| Jakarta Batch | `nablarch-batch-ee-archetype` | Jakarta Batch準拠 |
| バッチ(DBなし) | `nablarch-batch-dbless-archetype` | DB不要バッチ |
| Web(コンテナ) | `nablarch-container-web-archetype` | Docker Web |
| JAX-RS(コンテナ) | `nablarch-container-jaxrs-archetype` | Docker REST |
| バッチ(コンテナ) | `nablarch-container-batch-archetype` | Docker バッチ |
| バッチDBなし(コンテナ) | `nablarch-container-batch-dbless-archetype` | Docker DBなしバッチ |

全アーキタイプのグループID: `com.nablarch.archetype`

---

## 参考リンク一覧

### 公式ドキュメント
- [Nablarch 6u3 トップ](https://nablarch.github.io/)
- [アーキテクチャ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html)
- [アプリケーションフレームワーク](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/index.html)
- [標準ハンドラ一覧](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html)
- [システムリポジトリ](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html)
- [データベースアクセス](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database_management.html)
- [ユニバーサルDAO](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html)
- [データベースアクセス(JDBCラッパー)](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html)
- [セッション変数保存](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html)
- [Bean Validation](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation/bean_validation.html)
- [Mavenモジュール構成](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/blank_project/MavenModuleStructures/index.html)

### アーキテクチャ概要（処理方式別）
- [Webアプリケーション](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html)
- [RESTful Webサービス](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html)
- [バッチアプリケーション](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html)
- [MOMメッセージング](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/messaging/mom/architecture.html)
- [HTTPメッセージング](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/http_messaging/architecture.html)

### GitHub リポジトリ
- [nablarch/nablarch（メインREADME）](https://github.com/nablarch/nablarch)
- [nablarch-core（Handler, Interceptor等）](https://github.com/nablarch/nablarch-core)
- [nablarch-fw（共通ハンドラ）](https://github.com/nablarch/nablarch-fw)
- [nablarch-fw-web（Webハンドラ）](https://github.com/nablarch/nablarch-fw-web)
- [nablarch-fw-standalone（スタンドアローン）](https://github.com/nablarch/nablarch-fw-standalone)
- [nablarch-example-web（Webサンプル）](https://github.com/nablarch/nablarch-example-web)
- [nablarch-example-rest（RESTサンプル）](https://github.com/nablarch/nablarch-example-rest)
- [nablarch-example-batch（バッチサンプル）](https://github.com/nablarch/nablarch-example-batch)

### Javadoc
- [Handler](https://nablarch.github.io/docs/5u8/javadoc/nablarch/fw/Handler.html)
- [Interceptor](https://nablarch.github.io/docs/5u9/javadoc/nablarch/fw/Interceptor.html)
- [Interceptor.Factory](https://nablarch.github.io/docs/5u9/javadoc/nablarch/fw/Interceptor.Factory.html)
- [HandlerQueueManager](https://nablarch.github.io/docs/5u22/javadoc/nablarch/fw/HandlerQueueManager.html)
- [ComponentConfiguration](https://nablarch.github.io/docs/6/javadoc/nablarch/core/repository/di/config/xml/schema/ComponentConfiguration.html)
