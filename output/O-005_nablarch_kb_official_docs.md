# Nablarch公式ドキュメント・仕様 網羅的調査レポート

> **調査日**: 2026-02-01
> **対象バージョン**: Nablarch 6u3（最新）
> **情報源**: https://nablarch.github.io/docs/LATEST/doc/en/

---

## 目次

1. [公式ドキュメント全体構成](#1-公式ドキュメント全体構成)
2. [Nablarchフレームワーク概要](#2-nablarchフレームワーク概要)
3. [全対応アプリケーション種別の詳細仕様](#3-全対応アプリケーション種別の詳細仕様)
   - 3.1 [Webアプリケーション](#31-webアプリケーション)
   - 3.2 [RESTful Webサービス](#32-restful-webサービス)
   - 3.3 [Nablarchバッチアプリケーション](#33-nablarchバッチアプリケーション)
   - 3.4 [Jakarta Batch準拠バッチ（JSR352）](#34-jakarta-batch準拠バッチjsr352)
   - 3.5 [MOMメッセージング](#35-momメッセージング)
   - 3.6 [HTTPメッセージング（テーブルキューイング）](#36-httpメッセージングテーブルキューイング)
   - 3.7 [テーブルキューメッセージング](#37-テーブルキューメッセージング)
4. [標準ハンドラ一覧](#4-標準ハンドラ一覧)
5. [ライブラリ群の詳細](#5-ライブラリ群の詳細)
   - 5.1 [データベースアクセス](#51-データベースアクセス)
   - 5.2 [バリデーション](#52-バリデーション)
   - 5.3 [ログ出力](#53-ログ出力)
   - 5.4 [ファイル操作・データ変換](#54-ファイル操作データ変換)
   - 5.5 [メッセージ管理](#55-メッセージ管理)
   - 5.6 [コード管理](#56-コード管理)
   - 5.7 [日付管理](#57-日付管理)
   - 5.8 [トランザクション管理](#58-トランザクション管理)
   - 5.9 [排他制御](#59-排他制御)
   - 5.10 [メール送信](#510-メール送信)
   - 5.11 [セッションストア](#511-セッションストア)
   - 5.12 [ファイルパス管理](#512-ファイルパス管理)
   - 5.13 [認可・権限チェック](#513-認可権限チェック)
   - 5.14 [サービス閉塞チェック](#514-サービス閉塞チェック)
   - 5.15 [静的データキャッシュ](#515-静的データキャッシュ)
   - 5.16 [システム間メッセージング](#516-システム間メッセージング)
   - 5.17 [その他ライブラリ](#517-その他ライブラリ)
6. [設定方式](#6-設定方式)
7. [アダプタ一覧](#7-アダプタ一覧)
8. [対応プラットフォーム・バージョン情報](#8-対応プラットフォームバージョン情報)
9. [開発ツール・テスティングフレームワーク](#9-開発ツールテスティングフレームワーク)
10. [ブランクプロジェクト・Mavenアーキタイプ](#10-ブランクプロジェクトmavenアーキタイプ)
11. [クラウドネイティブサポート](#11-クラウドネイティブサポート)
12. [バージョン5→6移行ガイド](#12-バージョン56移行ガイド)

---

## 1. 公式ドキュメント全体構成

**トップページ**: https://nablarch.github.io/docs/LATEST/doc/en/index.html

ドキュメントは日本語と英語の二言語で提供される。主要セクションは以下の通り。

### 主要セクション一覧

| セクション | 内容 | URL |
|---|---|---|
| Nablarchとは | コンセプト、モジュール一覧、ライセンス | [index.html](https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/nablarch/index.html) |
| アプリケーションフレームワーク | Web/REST/バッチ/メッセージング、ハンドラ、ライブラリ | [application_framework/index.html](https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/index.html) |
| 開発ツール | 静的解析、テスティングフレームワーク、ツールボックス | [development_tools/](https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/index.html) |
| サンプル・例題 | Web/REST/バッチ/メッセージングの実装例、業務ユースケース | [examples/index.html](https://nablarch.github.io/docs/LATEST/doc/en/examples/index.html) |
| 移行ガイド | v5→v6移行手順 | [migration/index.html](https://nablarch.github.io/docs/LATEST/doc/en/migration/index.html) |
| 外部コンテンツ | Fintan掲載の開発ガイド等 | [external_contents/index.html](https://nablarch.github.io/docs/LATEST/doc/en/external_contents/index.html) |

---

## 2. Nablarchフレームワーク概要

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/nablarch/index.html

### 2.1 コンセプト

Nablarchは**堅牢性（Robustness）**、**テスト容易性（Testability）**、**即戦力（Ready-to-Use）**の3原則を掲げるJavaアプリケーションフレームワーク。TISの基幹システム構築経験から生まれた。

### 2.2 アーキテクチャ — ハンドラキュー

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/nablarch/architecture.html

Nablarchの根幹は**ハンドラキューアーキテクチャ**（パイプライン型処理モデル）である。

- **Chain of Responsibilityパターン**に類似（Servletフィルタと同じ考え方）
- **リクエスト処理（インバウンド）**: ハンドラが上から順に実行される
- **レスポンス処理（アウトバウンド）**: 「レスポンスが返ると、それまで実行されたハンドラが**逆順で**実行される」
- **インターセプタ**: 実行時にハンドラキューへ動的に追加されるハンドラ
- ハンドラの順序は重要：「一部のハンドラはコンテキストを考慮した順序で配置しないと正常に機能しない」

フレームワークは3つの主要要素で構成：
1. **ハンドラキュー** — リクエスト/レスポンスのパイプライン処理
2. **インターセプタ** — 動的にキューに追加されるハンドラ
3. **ライブラリ** — 再利用可能な共通機能群

### 2.3 基本方針

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/nablarch/policy.html

| 方針 | 内容 |
|---|---|
| Null処理 | 非入力値はnullに統一。コレクション/配列のAPIはnullを返さない |
| 例外 | **非チェック例外のみ**使用。原因チェーンあり |
| 国際化 | ログと例外メッセージは**英語に統一** |
| スレッド安全性 | フレームワーク全体がスレッドセーフ |
| OSS方針 | プロダクションコードは**OSSを使用しない**。脆弱性対応の迅速化のため。OSSはアダプタ経由で任意使用 |
| Java互換性 | Java 17準拠。Java 18以降のAPIは使用しない |
| BigDecimal | 指数表現の変換を制限（スケール: -9999～9999）。ヒープ枯渇防止 |
| コンポーネント | データベースアクセスコンポーネントはインターフェース定義。SQL修正が設定変更のみで可能 |
| Public API | `@Published`アノテーション付きクラス/メソッドが公開API。後方互換性を保証 |

### 2.4 対応アプリケーション種別

| 種別 | 説明 |
|---|---|
| Webアプリケーション | Servlet APIベースの従来型Web |
| RESTful Webサービス | Jakarta RESTful Web Services準拠 |
| Nablarchバッチ | オンデマンド/常駐バッチ |
| Jakarta Batch準拠バッチ | JSR352準拠（jBeret推奨） |
| MOMメッセージング | Message Oriented Middleware経由 |
| HTTPメッセージング | HTTP経由のメッセージング |
| テーブルキューメッセージング | DBテーブルをキューとして使用 |

---

## 3. 全対応アプリケーション種別の詳細仕様

### 3.1 Webアプリケーション

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/web/architecture.html

#### アーキテクチャ

3つの主要コンポーネントで構成：
1. **Nablarch Servlet Context Initialization Listener** — システムリポジトリとロギング基盤を初期化
2. **Web Front Controller** — サーブレットフィルタ。リクエスト処理をハンドラキューに委譲
3. **ハンドラキュー** — Chain of Responsibilityパターンによるリクエスト/レスポンス処理

#### 処理フロー

1. Web Front Controllerがリクエストを受信
2. ハンドラキューに委譲し順次処理
3. DispatchHandlerがURIからアクションクラスを特定
4. アクションがForm/Entityを使ってビジネスロジック実行
5. HttpResponseオブジェクトを返却
6. HTTPレスポンスハンドラがレスポンスを変換（JSPフォワード、リダイレクト等）

#### 最小ハンドラ構成（13ハンドラ）

ハンドラカテゴリ：
- **リクエスト/レスポンス変換**: HTTPキャラクタエンコーディング、HTTPレスポンス、内部フォワード、マルチパートリクエスト、セッション変数ストア、正規化、セキュアハンドラ
- **フィルタリング**: サービス閉塞チェック、権限チェック
- **データアクセス**: DB接続管理、トランザクション制御
- **セキュリティ**: CSRFトークン検証、HTTPエラー制御
- **インフラ**: スレッドコンテキスト、HTTPアクセスログ、ヘルスチェックエンドポイント

#### 主要機能

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/web/feature_details.html

| 機能 | 説明 |
|---|---|
| 入力バリデーション | Bean Validation統合、エラーメッセージ表示 |
| データベースアクセス | Universal DAO推奨 |
| 排他制御 | 楽観ロック/悲観ロック（Universal DAO推奨） |
| ファイル操作 | アップロード（マルチパート）、ダウンロード（データバインド推奨） |
| ルーティング | URI→アクションマッピング（Routing Adapter推奨） |
| 二重送信防止 | POST再送信防止、CSRFトークン検証 |
| セッション管理 | セッションストアによる入力データ保持 |
| ページネーション | 範囲指定DBクエリ |
| UI開発 | JSPタグライブラリ、Thymeleaf統合 |
| 国際化 | メッセージ/コード名の多言語化 |
| エラーハンドリング | ステータスコード設定、例外別遷移先 |
| ステートレス対応 | HTTPセッション不使用設計パターン |
| CSP | Content Security Policy対応 |

---

### 3.2 RESTful Webサービス

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/web_service/rest/architecture.html

#### アーキテクチャ

Webアプリケーションと同じ基盤上で、Jakarta RESTful Web Servicesのリソースクラスに類似したビジネスアクションを使用。

#### 処理フロー

1. **Web Front Controller**（jakarta.servlet.Filter実装）がリクエスト受信
2. **ハンドラキュー**がリクエストを処理
3. **DispatchHandler**がURIからアクションクラスを決定
4. **アクションクラス**がForm/Entityでビジネスロジック実行
5. レスポンス変換でメディアタイプに応じた出力

#### 最小ハンドラ構成（7ハンドラ）

| 順序 | ハンドラ | リクエスト処理 | レスポンス処理 | 例外処理 |
|---|---|---|---|---|
| 1 | Global Error Handler | — | — | ランタイム例外をログ出力 |
| 2 | Jakarta RESTful Web Services Response Handler | — | レスポンス書き込み | 例外レスポンス生成 |
| 3 | Database Connection Management | DB接続取得 | 接続解放 | — |
| 4 | Transaction Control | トランザクション開始 | コミット | ロールバック |
| 5 | Router Adapter | リクエストパスからアクション決定 | — | — |
| 6 | Request Body Conversion Handler | ボディをFormクラスに変換 | Formをレスポンスボディに変換 | — |
| 7 | Jakarta RESTful Web Services Bean Validation | Formクラスをバリデーション | — | — |

**注意**: ハンドラ6-7はRouter Adapter内にネストされ、メインキューには直接設定しない。

#### サポートアノテーション

- `@Produces` — レスポンスメディアタイプ指定
- `@Consumes` — リクエストメディアタイプ指定
- `@Valid` — Bean Validation実行

#### 制約

- Servletリソースインジェクション非対応
- `@Context`アノテーション非対応

#### 主要機能

入力バリデーション、DB連携、排他制御、URI→リソースマッピング、パス/クエリパラメータ、レスポンスヘッダ、国際化、認証、認可、エラーレスポンス、CORS、OpenAPIコード生成

---

### 3.3 Nablarchバッチアプリケーション

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/nablarch_batch/architecture.html

#### 処理タイプ

| タイプ | 説明 |
|---|---|
| **オンデマンドバッチ** | 定期起動。DB/ファイルからデータを読み取り処理 |
| **常駐バッチ** | 継続実行。一定間隔でデータを監視・処理。マルチプロセス対応 |

#### ハンドラキュー構成

**オンデマンドバッチ（DB使用） — 9ハンドラ**

| 順序 | ハンドラ | スレッド |
|---|---|---|
| 1 | Status Code → Process End Code Conversion | メイン |
| 2 | Global Error Handler | メイン |
| 3 | Database Connection Management（初期化/終了） | メイン |
| 4 | Transaction Control（初期化/終了） | メイン |
| 5 | Request Dispatch Handler | メイン |
| 6 | Multi-thread Execution Control | メイン |
| 7 | Database Connection Management（業務処理） | サブ |
| 8 | Transaction Loop Control | サブ |
| 9 | Data Read Handler | サブ |

**オンデマンドバッチ（DB不使用）**: DB関連ハンドラ除去で6ハンドラに簡略化。Loop Control Handlerを使用。

**常駐バッチ**: オンデマンドDB版に以下を追加:
- Thread Context Variable Management Handler
- Retry Handler
- Process Resident Handler
- Process Stop Control Handler
- Thread Context Variable Delete Handler

#### データリーダー

| リーダー | 用途 |
|---|---|
| DatabaseRecordReader | DBクエリ |
| FileDataReader | ファイル入力 |
| ValidatableFileDataReader | バリデーション付きファイル入力 |
| ResumeDataReader | 再開可能処理 |

#### アクションクラス

| クラス | 用途 |
|---|---|
| BatchAction | 汎用バッチ |
| FileBatchAction | ファイル入力バッチ |
| NoInputDataBatchAction | 入力データ不要バッチ |
| AsyncMessageSendAction | 非同期メッセージ送信 |

#### コマンドライン引数

`-requestPath=com.sample.Action/REQUEST_ID` 形式でアクションとリクエストIDを指定。

---

### 3.4 Jakarta Batch準拠バッチ（JSR352）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/jsr352/architecture.html

#### 概要

Jakarta Batch仕様に準拠したバッチアプリケーション。実装にはjBatch よりも**jBeret推奨**（ドキュメントの充実度とMaven Centralからの入手容易性のため）。

#### ステップタイプ

| タイプ | 説明 |
|---|---|
| **Batchlet** | タスク指向（ファイル取得、単一SQL実行等） |
| **Chunk** | レコード処理（Read-Process-Writeサイクル） |

#### リスナーアーキテクチャ

Jakarta Batch仕様では複数リスナーの実行順序が保証されないが、Nablarchはシステムリポジトリ設定により**順序保証**を実現。

**標準リスナー**:

| レベル | リスナー | 目的 |
|---|---|---|
| Job | JobProgressLogListener | 起動/終了ログ |
| Job | DuplicateJobRunningCheckListener | 多重起動防止 |
| Step | StepProgressLogListener | ステップ進捗ログ |
| Step | DbConnectionManagementListener | DB接続管理 |
| Step | StepTransactionManagementListener | トランザクション管理 |
| ItemWriter | ItemWriteTransactionManagementListener | 書き込みトランザクション |

#### リスナー設定

Job定義XML（`job.xml`）と コンポーネント設定の2箇所で定義:
- デフォルト: `jobListeners`, `stepListeners`, `itemWriteListeners`
- ジョブ固有: `{jobName}.jobListeners`
- ステップ固有: `{jobName}.{stepName}.stepListeners`

#### 制約

- Nablarchは例外をキャッチしない（Jakarta Batchランタイムに委譲）
- TransientUserData（JobContext/StepContextの一時領域）使用禁止
- StepContext一時領域は`@StepScoped`インジェクション専用

---

### 3.5 MOMメッセージング

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/messaging/mom/architecture.html

#### 種別

| 種別 | 説明 |
|---|---|
| **同期応答メッセージング** | レスポンスメッセージを生成・送信 |
| **非同期応答メッセージング** | メッセージ内容をDBに格納し、バッチ処理 |

#### 処理フロー

1. Common startup launcherがハンドラキューを実行
2. データリーダー（FwHeaderReader/MessageReader）がMQを監視・受信
3. Request dispatch handlerがメッセージのリクエストIDからアクション決定
4. アクションがForm/Entityでビジネスロジック実行
5. ResponseMessageを返却
6. プロセス停止まで2-5を繰り返し
7. StatusCodeConvertHandlerが終了コードに変換

#### 同期応答メッセージング — 最小ハンドラ構成（14ハンドラ）

| 順序 | ハンドラ |
|---|---|
| 1 | StatusCodeConvertHandler |
| 2 | GlobalErrorHandler |
| 3 | MultiThreadExecutionHandler |
| 4 | RetryHandler |
| 5 | MessagingContextHandler（MQ接続取得/解放） |
| 6 | DatabaseConnectionHandler |
| 7 | RequestThreadLoopHandler |
| 8 | ThreadContextClearHandler |
| 9 | ThreadContextHandler（リクエストID、ユーザID初期化） |
| 10 | ProcessStopHandler |
| 11 | MessageReplyHandler（レスポンスメッセージ作成・MQ送信） |
| 12 | DataReadHandler |
| 13 | RequestDispatchHandler |
| 14 | TransactionControlHandler |

#### 非同期応答メッセージング

同期版と同じ構成だが以下が異なる：
- **MessageReplyHandler** → **ResendMessageControlHandler** に置換
- TransactionControlHandlerは**2フェーズコミット**対応が必要（特にIBM MQ環境でDB+キュー操作のアトミック性確保）

#### データリーダー

- **FwHeaderReader** — メッセージからフレームワーク制御ヘッダを抽出
- **MessageReader** — MQからメッセージ読み取り

#### アクションクラス

- **MessagingAction** — 同期応答メッセージング用テンプレート
- **AsyncMessageReceiveAction** — 非同期応答メッセージング用テンプレート

#### 制約

**「MOMメッセージングでは汎用データフォーマットの固定長データのみ扱える」**

---

### 3.6 HTTPメッセージング

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/web_service/http_messaging/architecture.html

#### 処理フロー

1. **WebFrontController**（Jakarta Servlet Filter実装）がリクエスト受信
2. ハンドラキューで処理
3. DispatchHandlerがURIからアクション特定
4. アクションがビジネスロジック実行
5. ResponseMessage作成・返却
6. HTTP Messaging Response Conversion HandlerがJSON/XMLに変換

#### 最小ハンドラ構成（11ハンドラ）

| 順序 | ハンドラ | 役割 |
|---|---|---|
| 1 | Thread Context Clear | スレッドローカル値削除 |
| 2 | Global Error Handler | ランタイム例外ログ |
| 3 | HTTP Response Handler | Servlet操作・エラーページ |
| 4 | Thread Context Management | リクエストID・コンテキスト変数初期化 |
| 5 | HTTP Messaging Error Control | デフォルトエラーボディ生成 |
| 6 | Request Dispatch Handler | アクションへルーティング |
| 7 | HTTP Messaging Request Parsing | HTTPボディをRequestMessageに変換 |
| 8 | Database Connection Management | DB接続取得/解放 |
| 9 | HTTP Messaging Response Conversion | エラーレスポンス生成 |
| 10 | Transaction Control | トランザクション管理 |
| 11 | HTTP Messaging Response Conversion | 業務メッセージのHTTPレスポンス変換 |

#### 主要機能

初期化、入力値チェック、DBアクセス、排他制御、URI→アクションマッピング、国際化、認証、権限チェック、エラーレスポンス

---

### 3.7 テーブルキューメッセージング

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/messaging/db/architecture.html

#### 概要

データベーステーブルをキューとして使用し、「テーブルを定期的に監視して未処理レコードを順次処理する」方式。

#### 最小ハンドラ構成（14ハンドラ）

**メインスレッド（1-8）**:
1. Status Code → Process End Code Conversion
2. Thread Context Variable Delete
3. Global Error Handler
4. Thread Context Variable Management
5. Retry Handler
6. Database Connection Management（初期化）
7. Transaction Control（初期化）
8. Request Dispatch Handler

**サブスレッド（9-14）**:
9. Multi-thread Execution Control
10. Database Connection Management（業務処理）
11. Loop Control Handler in Request Thread
12. Process Stop Control Handler
13. Data Read Handler
14. Transaction Control（業務処理）

#### データリーダー

**DatabaseTableQueueReader** を使用（通常のDatabaseRecordReaderではない）。未処理データがない場合も検索SQLを繰り返し実行して継続監視。同一レコードの同時処理を防止する識別子追跡機能付き。

---

## 4. 標準ハンドラ一覧

### 4.1 共通ハンドラ（10個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/common/index.html

| ハンドラ | 目的 |
|---|---|
| Global Error Handler | アプリケーション全体のエラーハンドリング |
| Output File Release Handler | ファイルレコードライターの解放 |
| Request Dispatch Handler | リクエストをJavaパッケージにマッピング |
| Request Handler Entry | リクエスト処理のエントリポイント |
| Thread Context Variable Management Handler | スレッドローカルコンテキスト変数管理 |
| Thread Context Variable Delete Handler | スレッドコンテキスト変数のクリア |
| Database Connection Management Handler | DB接続の管理 |
| Transaction Control Handler | トランザクション境界の制御 |
| Permission Check Handler | ユーザ権限の検証 |
| Service Availability Check Handler | サービス閉塞状態の確認 |

### 4.2 Webアプリケーション専用ハンドラ（19個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/web/index.html

| ハンドラ | 目的 |
|---|---|
| HTTP Character Encoding Control | 文字エンコーディング管理 |
| HTTP Response Handler | HTTPレスポンス処理 |
| Secure Handler | セキュリティ関連機能 |
| HTTP Error Control Handler | HTTPエラーハンドリング |
| Internal Forward Handler | 内部リクエスト転送 |
| Resource Mapping Handler | リソースマッピング |
| Nablarch Custom Tag Control Handler | カスタムタグ処理制御 |
| HTTP Access Log Handler | HTTPアクセスログ記録 |
| Multipart Request Handler | マルチパートフォーム処理 |
| Session Variable Store Handler | セッション変数ストレージ管理 |
| HTTP Request Dispatch Handler | HTTPリクエストディスパッチ |
| HTTP Rewrite Handler | HTTPリクエスト書き換え |
| Mobile Terminal Access Handler | モバイルデバイスリクエスト処理 |
| POST Resubmit Prevention Handler | POST二重送信防止 |
| Normalize Handler | リクエスト/レスポンス正規化 |
| Session Concurrent Access Handler | セッション同時アクセス制御 |
| Hot Deploy Handler | ホットデプロイ機能 |
| CSRF Token Verification Handler | CSRFトークン検証 |
| Health Check Endpoint Handler | ヘルスチェックエンドポイント |

### 4.3 RESTful Webサービス専用ハンドラ（5個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/rest/index.html

| ハンドラ | 目的 |
|---|---|
| Request Body Conversion Handler | リクエストボディ変換 |
| Jakarta RESTful Web Services Bean Validation Handler | Bean Validation |
| Jakarta RESTful Web Services Response Handler | レスポンス処理・フォーマット |
| CORS Preflight Request Handler | CORSプリフライトリクエスト管理 |
| HTTP Access Log (for RESTful) Handler | RESTful用HTTPアクセスログ |

### 4.4 HTTPメッセージング専用ハンドラ（3個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/http_messaging/index.html

| ハンドラ | 目的 |
|---|---|
| HTTP Messaging Error Control Handler | エラーハンドリング |
| HTTP Messaging Request Conversion Handler | リクエスト変換 |
| HTTP Messaging Response Conversion Handler | レスポンス変換 |

### 4.5 スタンドアロンアプリケーション共通ハンドラ（8個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/standalone/index.html

| ハンドラ | 目的 |
|---|---|
| Common Launcher | スタンドアロンアプリの起動 |
| Data Read Handler | データ読み取り管理 |
| Retry Handler | リトライ制御 |
| Status Code → Process End Code Conversion | 終了コード変換 |
| Process Stop Control Handler | プロセス停止管理 |
| Process Multiple Launch Prevention Handler | 多重起動防止 |
| Multi-thread Execution Control Handler | マルチスレッド実行制御 |
| Loop Control Handler in Request Thread | リクエストスレッドのループ制御 |

### 4.6 バッチアプリケーション専用ハンドラ（3個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/batch/index.html

| ハンドラ | 目的 |
|---|---|
| Process Resident Handler | プロセスの常駐化管理 |
| Transaction Loop Control Handler | トランザクションループ制御 |
| Loop Control Handler | DB不使用バッチのループ制御 |

### 4.7 MOMメッセージング専用ハンドラ（3個）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/handlers/mom_messaging/index.html

| ハンドラ | 目的 |
|---|---|
| Messaging Context Management Handler | メッセージングコンテキスト管理 |
| Resent Message Control Handler | 再送メッセージ制御 |
| Message Response Control Handler | メッセージレスポンス管理 |

---

## 5. ライブラリ群の詳細

### 5.1 データベースアクセス

#### 5.1.1 Universal DAO（推奨）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/database/universal_dao.html

シンプルなO/Rマッパー。Entity定義からCRUD SQLを自動生成し、検索結果をBeanオブジェクトとして返す。

**Maven依存**:
```xml
<groupId>com.nablarch.framework</groupId>
<artifactId>nablarch-common-dao</artifactId>
```

**サポートするJakarta Persistenceアノテーション**:

| アノテーション | 用途 |
|---|---|
| `@Entity` | エンティティクラスの指定（テーブル名はPascal→snake_case変換） |
| `@Table` | テーブル名の明示指定、スキーマ指定 |
| `@Column` | カラム名指定（省略時はプロパティ名から変換） |
| `@Id` | 主キー指定（複合キーは複数宣言） |
| `@Version` | 楽観ロック用バージョン番号（数値型のみ） |
| `@Temporal` | java.util.Date/Calendar → DB型変換 |
| `@GeneratedValue` | 自動採番（AUTO/IDENTITY/SEQUENCE/TABLE） |
| `@SequenceGenerator` | シーケンスオブジェクト設定 |
| `@TableGenerator` | テーブル採番設定 |
| `@Access` | アノテーション配置位置（フィールド/プロパティ） |

**CRUD API**:
```java
// 登録
UniversalDao.insert(entity);
UniversalDao.batchInsert(entityList);
// 更新（主キー指定）
UniversalDao.update(entity);        // 楽観ロック適用
UniversalDao.batchUpdate(entityList); // 楽観ロック非適用
// 削除（主キー指定）
UniversalDao.delete(entity);
UniversalDao.batchDelete(entityList);
// 検索
Entity result = UniversalDao.findById(EntityClass.class, primaryKeyValue);
```

**SQLファイル検索**:
```java
List<Entity> results = UniversalDao.findAllBySqlFile(Entity.class, "SQL_ID", conditionBean);
```

**ページネーション**:
```java
EntityList<Entity> results = UniversalDao.per(pageSize).page(pageNumber)
    .findAllBySqlFile(Entity.class, "SQL_ID");
Pagination pagination = results.getPagination();
```

**遅延ロード**:
```java
try (DeferredEntityList<Entity> results =
        (DeferredEntityList<Entity>) UniversalDao.defer()
        .findAllBySqlFile(Entity.class, "SQL_ID")) {
    for (Entity item : results) { /* レコード単位処理 */ }
}
```

**楽観ロック**:
```java
@Version
private Long version;
// 更新時にバージョン不一致で OptimisticLockException
```

**サポートデータ型**: String, Short, Integer, Long, BigDecimal, Boolean, java.util.Date, java.sql.Date, java.sql.Timestamp, java.time.LocalDate, java.time.LocalDateTime, byte[]

**制約**: 「Universal DAOでは主キー以外の条件での更新/削除はできない」→ Database Access (JDBC Wrapper) を直接使用

#### 5.1.2 Database Access（JDBCラッパー）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/database/database.html

Universal DAOの基盤。PreparedStatementによるSQLインジェクション防止。

**コアAPI**:
- **DbConnectionContext**: DB接続管理ハンドラが登録したアクティブ接続を取得
- **SqlPStatement**: パラメータ化SQL実行（`connection.prepareStatementBySqlId()`）
- **SqlResultSet**: クエリ結果コレクション
- **ParameterizedSqlPStatement**: Beanベースの名前付きパラメータ

**SQLファイル形式**:
- クラスパス配下に配置
- 複数SQLは一意のSQLIDで区別
- 空行でSQL分離
- フォーマット: `SQLID = SQL文`
- コメント: `--` プレフィックス

**名前付きパラメータ**: `:propertyName` 構文。Beanオブジェクトのプロパティに自動マッピング。

**動的クエリ**:
- **条件分岐**: `$if(propertyName){condition}` — プロパティがnull/空/サイズ0の場合に条件除外
- **IN句**: `:userKbn[]` — 配列/Collectionプロパティを複数バインドマーカーに展開
- **ORDER BY**: `$sort(propertyId){(sortId1 clause1)(sortId2 clause2)}` — ランタイムでソート切替

**LIKE検索**: `name like :userName%`（前方）、`:%userName`（後方）、`:%userName%`（部分）。自動エスケープ付き。

**ダイアレクト対応**: Oracle, PostgreSQL, MySQL, SQL Server, DB2, H2

**キャッシュ**: InMemoryResultSetCache。SQLID+バインドパラメータ単位。TTL設定可能。

---

### 5.2 バリデーション

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/validation/bean_validation.html

#### Bean Validation（推奨）

Jakarta Bean Validation準拠。バリデーションエンジンは自前提供せず、Hibernate Validator等が必要。

**標準Jakarta Beanアノテーション**: `@AssertTrue`, `@Valid`, `@Size` 等

**Nablarch固有アノテーション**（`nablarch.core.validation.ee` パッケージ）:

| アノテーション | 用途 |
|---|---|
| `@Required` | 必須チェック |
| `@Length` | 文字数制約 |
| `@SystemChar` | 文字種バリデーション |
| `@Domain` | ドメインレベルバリデーション |

**ドメインバリデーション**: DomainBeanクラスにバリデーションルールを集約し、`@Domain("name")`で適用。

**文字種バリデーション定義**:
- RangedCharsetDef — Unicode範囲指定
- LiteralCharsetDef — 文字列挙
- CompositeCharsetDef — 複合定義

**エラーメッセージ**: `NablarchMessageInterpolator`がメッセージ管理と統合。`{messageId}`形式。

**重要な制約**: 「Beanクラスの全プロパティはStringで定義すべき」（外部入力はバリデーション前にBeanに変換されるため）

#### Nablarch Validation

Nablarch独自のバリデーション機能。Bean Validationが要件に合わない場合の代替。

---

### 5.3 ログ出力

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/log.html

#### アーキテクチャ

3つの置換可能コンポーネント：
- **Logger/LoggerFactory** — ログ開始
- **LogWriter** — 出力先（ファイル、コンソール等）
- **LogFormatter** — メッセージフォーマット

#### ログレベル（6段階）

| レベル | 用途 |
|---|---|
| FATAL | 即時対応が必要な致命的障害 |
| ERROR | アプリ継続を妨げる問題 |
| WARN | 将来の問題可能性 |
| INFO | 本番運用情報（アクセス/統計ログ） |
| DEBUG | 開発デバッグ情報 |
| TRACE | フレームワーク開発用詳細情報 |

#### ログカテゴリ（7種）

| カテゴリ | 用途 |
|---|---|
| 障害通知ログ | インシデント時の一次対応者特定 |
| 障害解析ログ | 障害原因の詳細 |
| SQLログ | SQL実行時間・ステートメント |
| 性能ログ | プロセス実行メトリクス・メモリ使用量 |
| HTTPアクセスログ | Webアプリの実行状況 |
| HTTPアクセスログ（RESTful） | RESTサービスの実行情報 |
| メッセージングログ | メッセージ送受信状況 |

#### 設定ファイル

- **log.properties**: LoggerFactory、LogWriter定義、Logger設定
- **app-log.properties**: カテゴリ別フォーマッタ設定

#### LogWriter実装

| 実装 | 用途 |
|---|---|
| FileLogWriter | ファイル出力（ローテーション対応） |
| SynchronousFileLogWriter | マルチプロセスファイル書き込み（低頻度ログ限定） |
| StandardOutputLogWriter | コンソール出力 |
| LogPublisher | リスナーにLogContext送信 |

#### JSONログ

全ログ種別にJSONフォーマッタを提供（JsonLogFormatter, FailureJsonLogFormatter, SqlJsonLogFormatter, HttpAccessJsonLogFormatter, JaxRsAccessJsonLogFormatter, MessagingJsonLogFormatter）

#### ローテーション

- **FileSizeRotatePolicy**（デフォルト）: ファイルサイズ閾値
- **DateRotatePolicy**: 日時スケジュール

---

### 5.4 ファイル操作・データ変換

#### 5.4.1 データバインド

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/data_io/data_bind.html

CSV/TSV/固定長ファイルをJava BeanまたはMapオブジェクトとして扱う。

**CSVアノテーション**: `@Csv`, `@CsvFormat`
**固定長アノテーション**: `@FixedLength`, `@Field`, `@Lpad`, `@Rpad`

**プリセットCSVフォーマット**:
| フォーマット | 説明 |
|---|---|
| DEFAULT | カンマ区切り、空行無視、ヘッダ付き |
| RFC4180 | RFC準拠、ヘッダなし |
| EXCEL | 標準カンマフォーマット |
| TSV | タブ区切り |

**マルチレイアウト対応**: 複数レコード形式の固定長ファイルに対応。`MultiLayout`継承+`RecordIdentifier`実装。

#### 5.4.2 汎用データフォーマット

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/data_io/data_format/format_definition.html

**対応フォーマット**: 固定長、可変長（CSV/TSV）、JSON、XML

**フォーマット定義ファイル**:
```
file-type: "Variable"
text-encoding: "MS932"
record-separator: "\\r\\n"
field-separator: ","
[RecordName]
1 fieldName fieldType
```

**フィールド型**: String(X), Numeric(N, N9), パック10進数, ゾーン10進数, 日付/時刻
**文字セット**: UTF-8, Shift_JIS, EBCDIC
**特徴**: パディング/トリミング対応、階層データ（ドット記法）、XMLネームスペース、マルチレイアウトレコード

#### 5.4.3 フォーマッタ

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/format.html

日付・数値等のフォーマット機能。フォーマット設定を一元管理し、画面/ファイル/メール間で個別設定不要。

---

### 5.5 メッセージ管理

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/message.html

#### 概要

画面固定テキストとエラーメッセージの管理。「メッセージは個別に定義すべき。共通化は仕様変更時の意図しない副作用を招く」。

#### メッセージ定義

デフォルト: `classpath:messages.properties`（UTF-8、native2ascii不要）

#### フォーマット

**MessageFormat方式**: `{0}`, `{1}` プレースホルダ
**Map方式**: `{propertyName}` プレースホルダ

#### 多言語対応

- `messages.properties`（デフォルト言語）
- `messages_en.properties`（英語）
- `messages_zh.properties`（中国語）
- ThreadContext#getLanguage() で言語選択

#### メッセージレベル

| レベル | CSSクラス |
|---|---|
| INFO | nablarch_info |
| WARN | nablarch_warn |
| ERROR | nablarch_error |

#### DB格納代替

`BasicStringResourceLoader` でプロパティファイルをDB管理に置換可能。

---

### 5.6 コード管理

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/code.html

静的コード情報（値↔名前のマッピング）を管理。動的な「商品コード」等は対象外。

**DBテーブル構成**:
- **コードパターンテーブル**: ID, VALUE, PATTERN（パターンフラグ 0/1）
- **コード名称テーブル**: ID, VALUE, LANG, SORT_ORDER, NAME, SHORT_NAME, OPTIONAL_NAME

**主要機能**:
- パターン切替: `CodeUtil.getValues("GENDER", "PATTERN1")`
- 多言語対応: `CodeUtil.getName("GENDER", "MALE", Locale.ENGLISH)`
- バリデーション: `@CodeValue(codeId = "GENDER")`

---

### 5.7 日付管理

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/date.html

| 種別 | 説明 | 設定 |
|---|---|---|
| **システム日時** | OS日時 | `BasicSystemTimeProvider`（コンポーネント名: systemTimeProvider） |
| **業務日付** | DB管理の業務日付（yyyyMMdd） | `BasicBusinessDateProvider`（テーブル名、カテゴリ列、日付列指定） |

**日付上書き**: システムプロパティ `BasicBusinessDateProvider.<Category>=yyyyMMdd` で一時的に上書き可能（バッチ再実行時等）。

---

### 5.8 トランザクション管理

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/transaction.html

DB・メッセージキュー等のトランザクション制御。

**JDBCトランザクション**: `JdbcTransactionFactory` でアイソレーションレベル・タイムアウト設定。

**タイムアウトメカニズム**: SQL実行前後およびクエリタイムアウト例外時にチェック。`Transaction#begin`呼び出しでリセット（commit/rollbackではリセットされない）。

**制約**: タイムアウトはDB アクセス時のみ機能。DB アクセスなしのロジック（無限ループ等）は検知不可。

---

### 5.9 排他制御

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/exclusive_control.html

**注意**: この機能は**非推奨**。Universal DAOの排他制御機能の使用を推奨。

楽観ロック（バージョン番号チェック）と悲観ロック（更新前にバージョン番号更新）を提供。

---

### 5.10 メール送信

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/mail.html

#### アーキテクチャ

**遅延オンラインプロセス**パターン：メール送信リクエストをDBに格納し、常駐バッチで非同期送信。ビジネストランザクションとメール送信を分離。

#### DBテーブル構成

| テーブル | 目的 |
|---|---|
| Mail Request | メールデータ（件名、送信者、ステータス） |
| Mail Recipient | 宛先（TO/CC/BCC） |
| Mail Attached Files | 添付ファイルメタデータ+バイナリ |
| Mail Template | 再利用テンプレート（言語別） |

#### テンプレートエンジン

FreeMarker、Thymeleaf、Velocityアダプタ対応

#### セキュリティ

ヘッダインジェクション攻撃防止（件名、Return-Pathの改行文字検出→InvalidCharacterException）

#### マルチプロセス対応

悲観ロック（プロセスID列）による重複処理防止

---

### 5.11 セッションストア

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/session_store.html

HTTPセッションを抽象化。セッションID（デフォルト: `NABLARCH_SID`）をCookieで管理。

#### ストアタイプ

| タイプ | 説明 | 用途 |
|---|---|---|
| **DB Store** | USER_SESSIONテーブルに永続化 | サーバ再起動/ローリングメンテ対応 |
| **HIDDEN Store** | クライアント側hidden フィールド（AES暗号化） | マルチタブ操作対応の入力→確認→完了フロー |
| **HTTP Session Store** | APサーバメモリ | 頻繁アクセスデータ（認証情報等） |

#### シリアライズ

JavaSerializeStateEncoder（デフォルト）、JavaSerializeEncryptStateEncoder（暗号化）、JaxbStateEncoder（XML）

#### セッション操作

```java
SessionUtil.changeId(ctx);      // セッションID再生成
SessionUtil.put(ctx, "user", user, "db"); // DB Storeに保存
SessionUtil.invalidate(ctx);    // セッション破棄
```

#### Redis対応

Lettuce Adapterによるセッションストレージ

---

### 5.12 ファイルパス管理

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/file_path_management.html

論理名でファイルI/Oディレクトリと拡張子を管理。`FilePathSetting`コンポーネント（名前: `filePathSetting`）で設定。

**スキーム**: `file:` （推奨）と `classpath:`

**制約**: パスにスペース不可。classpathスキームはJBoss/WildFlyの仮想ファイルシステムと非互換。

---

### 5.13 認可・権限チェック

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/permission_check.html

**2つのアプローチ**:
1. **ハンドラベース**: 動的な認可条件、データ駆動型権限管理向け
2. **アノテーションベース**: 安定した認可要件、簡易モデル向け

---

### 5.14 サービス閉塞チェック

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/service_availability.html

リクエスト単位でサービス閉塞状態をDBで管理。Webアプリは503エラー、常駐バッチはアイドル処理を返す。

**DBテーブル**: リクエストID（PK）、サービス利用可否ステータス

---

### 5.15 静的データキャッシュ

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/static_data_cache.html

DB/ファイルの静的データをヒープにキャッシュ。`StaticDataLoader`インターフェース実装。

**ロード方式**:
- **バッチロード**: システム起動時に全データロード
- **オンデマンドロード**: 初回アクセス時にキャッシュ

---

### 5.16 システム間メッセージング

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/system_messaging.html

外部システムとのメッセージ送受信。2方式:
1. **MOMメッセージング** — Message Oriented Middleware経由
2. **HTTPメッセージング** — HTTPプロトコル経由

---

### 5.17 その他ライブラリ

| ライブラリ | 説明 | ソース |
|---|---|---|
| **BeanUtil** | Beanプロパティコピー、型変換 | ドキュメント内参照 |
| **JSPカスタムタグ** | Nablarch提供のJSPタグライブラリ | ドキュメント内参照 |
| **DB二重送信防止** | データベースによる二重送信防止 | ドキュメント内参照 |
| **ステートレスWeb** | HTTPセッション検出機能（セッション作成試行時に例外） | [stateless_web_app.html](https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/stateless_web_app.html) |

---

## 6. 設定方式

### 6.1 システムリポジトリ（XMLコンポーネント定義）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/libraries/repository.html

NablarchのDIコンテナ。オブジェクトと設定値を管理。

#### XML構文

```xml
<component-configuration
  xmlns="http://tis.co.jp/nablarch/component-configuration"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <component name="sample" class="com.example.SampleComponent">
    <property name="prop1" value="value1" />
    <property name="prop2" ref="otherComponent" />
    <property name="list1">
      <list>
        <value>item1</value>
        <value>item2</value>
      </list>
    </property>
  </component>
</component-configuration>
```

#### プロパティインジェクション

- **直接参照**: `<property name="sample" ref="sample" />`
- **ネスト定義**: property内にcomponent定義
- **リテラル値**: String, Integer, Long, Boolean, 配列
- **コレクション**: `<list>`, `<map>`

#### オートワイヤリング

| モード | 説明 |
|---|---|
| **ByType**（デフォルト） | 一致する型が1つの場合にインジェクト |
| **ByName** | プロパティ名とコンポーネント名のマッチング |
| **None** | 自動インジェクション無効 |

#### ファクトリパターン

`ComponentFactory<T>`インターフェース実装。**ネスト不可**。

#### 環境依存値

- **設定ファイル**: key=value形式（Nablarch固有）
- **Propertiesファイル**: 標準Java Properties
- **参照構文**: `${key.name}`

**上書き優先度**（高→低）:
1. OS環境変数（有効時）
2. システムプロパティ（-D）
3. 設定ファイル

環境変数名: ドットとハイフンをアンダースコア+大文字に変換（例: `example.error-message` → `EXAMPLE_ERROR_MESSAGE`）

#### アノテーションベース設定

`@SystemRepositoryComponent`アノテーションでXMLなしのコンポーネント管理:
- `@ConfigValue` — 設定値/環境依存値のインジェクト
- `@ComponentRef` — 名前付きコンポーネントのインジェクト

#### 初期化順序

`BasicApplicationInitializer`（名前: `initializer`）の`<component-ref>`リストで順序指定。

#### コンポーネント取得

```java
SampleComponent sample = SystemRepository.get("sampleComponent");
Component2 nested = SystemRepository.get("component.component2"); // ドット記法
```

---

## 7. アダプタ一覧

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/adaptors/index.html

| アダプタ | 用途 |
|---|---|
| **Log Adapter** | ログ機能拡張 |
| **Routing Adapter** | リクエストルーティング |
| **IBM MQ Adapter** | IBM MQメッセージング統合 |
| **Jakarta RESTful Web Services Adapter** | RESTful Webサービス対応 |
| **Doma Adapter** | Doma ORM統合（エンティティリスナー機能） |
| **JSR310 Adapter** | Java Date and Time API対応 |
| **E-mail FreeMarker Adapter** | FreeMarkerメールテンプレート |
| **E-mail Thymeleaf Adapter** | Thymeleafメールテンプレート |
| **E-mail Velocity Adapter** | Velocityメールテンプレート |
| **Web Application Thymeleaf Adapter** | ThymeleafによるWebビュー |
| **Lettuce Adapter** | Redisクライアント統合 |
| **SLF4J Adapter** | SLF4Jロギング統合 |
| **Micrometer Adapter** | メトリクス収集（Micrometer） |

---

## 8. 対応プラットフォーム・バージョン情報

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/nablarch/platform.html

### 必須環境

| 項目 | バージョン |
|---|---|
| **Java** | Java SE 17以上 |
| **JDBC** | 3.0以上 |

### Jakarta EE仕様（使用機能に応じて）

| 仕様 | バージョン |
|---|---|
| Jakarta Standard Tag Library | 3.0+ |
| Jakarta Activation | 2.1+ |
| Jakarta Server Pages | 3.1+ |
| Jakarta Servlet | 6.0+ |
| Jakarta Mail | 2.1+ |
| Jakarta Messaging | 3.1+ |
| Jakarta Persistence | 3.1+ |
| Jakarta Batch | 2.1+ |
| Jakarta Bean Validation | 3.0+ |
| Jakarta RESTful Web Services | 3.1+ |

### 検証済みJavaバージョン

- **Java SE 17**
- **Java SE 21**（別途設定が必要）

### 検証済みデータベース

| DB | バージョン |
|---|---|
| Oracle Database | 19c, 21c, 23ai |
| IBM Db2 | 11.5, 12.1 |
| SQL Server | 2017, 2019, 2022 |
| PostgreSQL | 12.2, 13.2, 14.0, 15.2, 16.2, 17.4 |

### 検証済みアプリケーションサーバー

| APサーバー | バージョン |
|---|---|
| WebSphere Application Server Liberty | 25.0.0.2 |
| Open Liberty | 25.0.0.2 |
| Red Hat JBoss EAP | 8.0.0 |
| WildFly | 35.0.1.Final |
| Apache Tomcat | 10.1.17 |

### その他検証済み環境

| コンポーネント | バージョン |
|---|---|
| Hibernate Validator | 8.0.0.Final |
| JBeret | 2.1.1.Final |
| IBM MQ | 9.3 |

### 対応ブラウザ（PC）

Microsoft Edge, Mozilla Firefox, Google Chrome, Safari

---

## 9. 開発ツール・テスティングフレームワーク

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/index.html

### 開発ツール

| カテゴリ | ツール |
|---|---|
| **Java静的解析** | インスペクション、フォーマット統一、APIチェック |
| **テスティングフレームワーク** | 単体テスト、自動化テストフレームワーク、テストツール |
| **ツールボックス** | JSP静的解析、SQLエグゼキュータ、OpenAPIジェネレータ |

### テスティングフレームワーク — テストカテゴリ

| カテゴリ | 対象 |
|---|---|
| **クラス単体テスト** | 全アプリケーション種別共通 |
| **Webアプリリクエスト単体テスト** | リクエスト単体、ファイルアップロード、サブ機能 |
| **RESTfulリクエスト単体テスト** | リクエスト単体、サブ機能 |
| **バッチリクエスト単体テスト** | バッチ処理、サブ機能 |
| **メッセージングテスト** | 同期/非同期受信、HTTP送受信、非同期送信 |
| **メール送信テスト** | メール送信処理 |

### JUnit 5対応

`nablarch-testing-junit5` モジュール。`TestSupport`クラス提供（JUnit 4では基底クラス継承方式だった）。

---

## 10. ブランクプロジェクト・Mavenアーキタイプ

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/blank_project/MavenModuleStructures/index.html

### アーキタイプ一覧

全アーキタイプのグループID: `com.nablarch.archetype`

| アーティファクトID | 用途 | パッケージ形式 |
|---|---|---|
| nablarch-web-archetype | Webアプリケーション | WAR |
| nablarch-jaxrs-archetype | RESTful Webサービス | WAR |
| nablarch-batch-ee-archetype | Jakarta Batch準拠バッチ | JAR |
| nablarch-batch-archetype | Nablarchバッチ | JAR |
| nablarch-batch-dbless-archetype | DB不使用バッチ | JAR |
| nablarch-container-web-archetype | Docker版Webアプリ | Container |
| nablarch-container-jaxrs-archetype | Docker版REST | Container |
| nablarch-container-batch-archetype | Docker版バッチ | Container |
| nablarch-container-batch-dbless-archetype | Docker版DB不使用バッチ | Container |

### ビルドプロファイル

- **dev**（デフォルト）: `src/env/dev/resources`
- **prod**: `src/env/prod/resources`

プロダクションビルド: `mvn package -P prod -DskipTests=true`

### BOMによるバージョン管理

Nablarchの各モジュールバージョンはBOMで管理。`pom.xml`のBOMバージョンを変更するだけでアップグレード可能。

---

## 11. クラウドネイティブサポート

### 11.1 Docker コンテナ化

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/cloud_native/containerize/index.html

Twelve-Factor App方法論に基づくコンテナ化ガイド。

**Webアプリの必須変更**:
1. **ステートレス化**: HTTPセッション状態を保持しない
2. **ログ出力**: 全ログを標準出力に（ファイル出力ではなく）
3. **環境設定**: OS環境変数から（Mavenプロファイル埋め込みではなく）

**コンテナ用アーキタイプ**: Jib Maven pluginを統合。Dockerfileなしでコンテナイメージ生成。

### 11.2 分散トレーシング（AWS X-Ray）

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/cloud_native/distributed_tracing/aws_distributed_tracing.html

AWS X-Ray SDKによる分散トレーシング。自動インストルメンテーションエージェントは使用不可（フレームワーク構造上）、手動SDK統合が必要。

**設定対象**:
1. **受信リクエスト**: X-Rayサーブレットフィルタを`web.xml`に追加
2. **送信HTTPコール**: Jersey + Apache HttpComponents + AWS SDK
3. **SQLクエリ**: `TracingDataSource.decorate()`でデータソースをラップ

---

## 12. バージョン5→6移行ガイド

**ソース**: https://nablarch.github.io/docs/LATEST/doc/en/migration/index.html

### 主要変更点

| 項目 | Nablarch 5 | Nablarch 6 |
|---|---|---|
| Java | 11+ | **17+** |
| EE仕様 | Java EE | **Jakarta EE 10** |
| 名前空間 | `javax.*` | **`jakarta.*`** |

### 名前空間移行

| パッケージ | 旧 | 新 |
|---|---|---|
| Servlet | javax.servlet | jakarta.servlet |
| JSP | javax.servlet.jsp | jakarta.servlet.jsp |
| JPA | javax.persistence | jakarta.persistence |
| JAX-RS | javax.ws.rs | jakarta.ws.rs |
| Bean Validation | javax.validation | jakarta.validation |

### 主要ライブラリ更新

| ライブラリ | 旧バージョン | 新バージョン |
|---|---|---|
| Hibernate Validator | 5.3.6.Final | 8.0.0.Final |
| Jersey BOM | — | 3.1.8+ |
| ActiveMQ → Artemis | — | 2.37.0+（Jakarta版） |
| waitt-maven-plugin | — | jetty-ee10-maven-plugin v12.0.12 に置換 |
| gsp-dba-maven-plugin | — | 5.1.0 |

### 設定ファイル変更

- **web.xml スキーマ**: `http://xmlns.jcp.org/xml/ns/javaee` → `https://jakarta.ee/xml/ns/jakartaee`
- **JSP taglib**: `http://java.sun.com/jsp/jstl/core` → `jakarta.tags.core`

### 移行手順概要

1. 最新Nablarch 5にアップグレード（前提条件）
2. BOMバージョンを6u2以降に更新
3. Java EE依存をJakarta EEに置換
4. `import`文と設定ファイルを更新
5. ビルドツールを更新

**注意**: 後方互換性なし。Jakarta EE 10対応のAPサーバーが必須。

---

## 補足: Nablarchバージョン履歴

| バージョン | 主要特徴 |
|---|---|
| Nablarch 5（5u26が最終） | Java EE、Java 11+、javax.* 名前空間 |
| Nablarch 6u2 | Jakarta EE 10正式対応の初版 |
| **Nablarch 6u3**（最新） | 最新の検証済み環境（Java 21、PostgreSQL 17.4等） |

**GitHub**: https://github.com/nablarch
**公式サイト**: https://nablarch.github.io/

---

*本レポートは https://nablarch.github.io/docs/LATEST/doc/en/ 配下の公式ドキュメントを網羅的に調査した結果である。推測は含まず、ドキュメントに記載された事実のみを記録している。*
