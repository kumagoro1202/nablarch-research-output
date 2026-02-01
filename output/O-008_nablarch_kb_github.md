# Nablarch GitHub Organization 網羅的調査レポート

> **調査日**: 2026-02-01
> **タスクID**: subtask_018 / parent: cmd_007
> **データソース**: gh CLI + GitHub REST API + Web検索

---

## 目次

1. [Organization 概要](#1-organization-概要)
2. [全リポジトリ一覧（カテゴリ別）](#2-全リポジトリ一覧カテゴリ別)
3. [主要リポジトリ詳細](#3-主要リポジトリ詳細)
4. [nablarch-tools-for-ai の現状](#4-nablarch-tools-for-ai-の現状)
5. [メトリクス情報](#5-メトリクス情報)
6. [リリース履歴](#6-リリース履歴)
7. [リポジトリ間の依存関係](#7-リポジトリ間の依存関係)

---

## 1. Organization 概要

| 項目 | 値 |
|---|---|
| **Organization名** | [nablarch](https://github.com/nablarch) |
| **リポジトリ総数** | 106 |
| **アクティブ（非アーカイブ）** | 88 |
| **アーカイブ済み** | 18 |
| **主要言語** | Java |
| **ライセンス** | Apache License 2.0（全リポジトリ共通） |
| **Organization作成日** | 2016-08-19 |
| **メインリポジトリ** | [nablarch/nablarch](https://github.com/nablarch/nablarch)（42 stars, 11 forks） |
| **Issue Tracking** | [Nablarch JIRA](https://nablarch.atlassian.net) |

---

## 2. 全リポジトリ一覧（カテゴリ別）

### 2.1 Nablarch Core（10リポジトリ）

フレームワークの基盤ライブラリ群。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-core](https://github.com/nablarch/nablarch-core) | Nablarch Core | 2 | 5 | 2025-03 | Java |
| [nablarch-core-beans](https://github.com/nablarch/nablarch-core-beans) | JavaBeans Utilities | 1 | 1 | 2025-03 | Java |
| [nablarch-core-repository](https://github.com/nablarch/nablarch-core-repository) | Settings manager（DIコンテナ） | 0 | 2 | 2024-09 | Java |
| [nablarch-core-transaction](https://github.com/nablarch/nablarch-core-transaction) | Transaction managers | 0 | 0 | 2024-09 | Java |
| [nablarch-core-jdbc](https://github.com/nablarch/nablarch-core-jdbc) | JDBC Utilities | 0 | 2 | 2024-09 | Java |
| [nablarch-core-message](https://github.com/nablarch/nablarch-core-message) | Messages（エラー、警告等） | 0 | 0 | 2024-09 | Java |
| [nablarch-core-validation](https://github.com/nablarch/nablarch-core-validation) | Input validation | 0 | 0 | 2020-10 | Java |
| [nablarch-core-validation-ee](https://github.com/nablarch/nablarch-core-validation-ee) | Jakarta Bean Validation準拠 | 0 | 0 | 2023-09 | Java |
| [nablarch-core-dataformat](https://github.com/nablarch/nablarch-core-dataformat) | File manager | 0 | 1 | 2025-03 | Java |
| [nablarch-core-applog](https://github.com/nablarch/nablarch-core-applog) | Logging | 0 | 0 | 2024-09 | Java |

### 2.2 Nablarch Framework（14リポジトリ）

実行制御・ハンドラキュー・各アプリケーション種別対応。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-fw](https://github.com/nablarch/nablarch-fw) | Nablarch Handlers（コアハンドラ） | 0 | 1 | 2021-03 | Java |
| [nablarch-fw-web](https://github.com/nablarch/nablarch-fw-web) | Web統合基盤 | 0 | 0 | 2025-03 | Java |
| [nablarch-fw-web-tag](https://github.com/nablarch/nablarch-fw-web-tag) | Jakarta Server Pages Custom Tag Library | 0 | 0 | 2024-09 | Java |
| [nablarch-fw-web-dbstore](https://github.com/nablarch/nablarch-fw-web-dbstore) | JDBC利用セッションストア | 0 | 0 | 2020-10 | Java |
| [nablarch-fw-web-hotdeploy](https://github.com/nablarch/nablarch-fw-web-hotdeploy) | Hot deploy | 0 | 0 | 2020-10 | Java |
| [nablarch-fw-web-extension](https://github.com/nablarch/nablarch-fw-web-extension) | ファイルアップロード/ダウンロード | 0 | 0 | 2020-10 | Java |
| [nablarch-fw-web-doublesubmit-jdbc](https://github.com/nablarch/nablarch-fw-web-doublesubmit-jdbc) | JDBC利用トークンマネージャ | 0 | 0 | 2020-04 | Java |
| [nablarch-fw-jaxrs](https://github.com/nablarch/nablarch-fw-jaxrs) | RESTful Web Services | 0 | 1 | 2025-03 | Java |
| [nablarch-fw-batch](https://github.com/nablarch/nablarch-fw-batch) | バッチアプリケーション基盤 | 0 | 1 | 2025-03 | Java |
| [nablarch-fw-batch-ee](https://github.com/nablarch/nablarch-fw-batch-ee) | Jakarta Batch準拠バッチ | 0 | 1 | 2020-10 | Java |
| [nablarch-fw-standalone](https://github.com/nablarch/nablarch-fw-standalone) | スタンドアロンアプリケーション | 0 | 0 | 2022-03 | Java |
| [nablarch-fw-messaging](https://github.com/nablarch/nablarch-fw-messaging) | メッセージングアーキテクチャ | 0 | 0 | 2024-09 | Java |
| [nablarch-fw-messaging-http](https://github.com/nablarch/nablarch-fw-messaging-http) | HTTPメッセージング | 0 | 0 | 2020-10 | Java |
| [nablarch-fw-messaging-mom](https://github.com/nablarch/nablarch-fw-messaging-mom) | MOMメッセージング | 0 | 0 | 2024-03 | Java |

### 2.3 Nablarch Common Component（15リポジトリ）

共通業務コンポーネント群。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-common-dao](https://github.com/nablarch/nablarch-common-dao) | エンティティクラスからSQL生成（ユニバーサルDAO） | 0 | 1 | 2025-03 | Java |
| [nablarch-common-jdbc](https://github.com/nablarch/nablarch-common-jdbc) | DB接続ハンドラ | 0 | 0 | 2023-09 | Java |
| [nablarch-common-databind](https://github.com/nablarch/nablarch-common-databind) | データバインド（Map, List等） | 0 | 1 | 2025-03 | Java |
| [nablarch-common-auth](https://github.com/nablarch/nablarch-common-auth) | 認可 | 0 | 0 | 2025-03 | Java |
| [nablarch-common-auth-jdbc](https://github.com/nablarch/nablarch-common-auth-jdbc) | JDBC認可実装 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-auth-session](https://github.com/nablarch/nablarch-common-auth-session) | SessionStore認可実装 | 0 | 0 | 2023-03 | Java |
| [nablarch-common-idgenerator](https://github.com/nablarch/nablarch-common-idgenerator) | ID生成 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-idgenerator-jdbc](https://github.com/nablarch/nablarch-common-idgenerator-jdbc) | JDBC ID生成実装 | 0 | 1 | 2025-03 | Java |
| [nablarch-common-code](https://github.com/nablarch/nablarch-common-code) | コード管理 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-code-jdbc](https://github.com/nablarch/nablarch-common-code-jdbc) | OTLT利用コード管理 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-exclusivecontrol](https://github.com/nablarch/nablarch-common-exclusivecontrol) | 排他制御 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-exclusivecontrol-jdbc](https://github.com/nablarch/nablarch-common-exclusivecontrol-jdbc) | JDBC排他制御実装 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-encryption](https://github.com/nablarch/nablarch-common-encryption) | 暗号化 | 0 | 0 | 2020-10 | Java |
| [nablarch-common-date](https://github.com/nablarch/nablarch-common-date) | システム日付・業務日付 | 0 | 0 | 2020-10 | Java |
| [nablarch-mail-sender](https://github.com/nablarch/nablarch-mail-sender) | メール送信 | 0 | 0 | 2022-03 | Java |

### 2.4 Adaptor（14リポジトリ）

外部ライブラリ/フレームワークとの統合アダプタ。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-router-adaptor](https://github.com/nablarch/nablarch-router-adaptor) | http-request-routerアダプタ | 0 | 0 | 2025-03 | Java |
| [nablarch-jaxrs-adaptor](https://github.com/nablarch/nablarch-jaxrs-adaptor) | Jakarta RESTful Web Servicesアダプタ | 0 | 0 | 2025-03 | Java |
| [nablarch-wmq-adaptor](https://github.com/nablarch/nablarch-wmq-adaptor) | IBM MQアダプタ | 0 | 1 | 2024-03 | Java |
| [nablarch-doma-adaptor](https://github.com/nablarch/nablarch-doma-adaptor) | Domaアダプタ（DB） | 0 | 0 | 2020-10 | Java |
| [nablarch-slf4j-adaptor](https://github.com/nablarch/nablarch-slf4j-adaptor) | SLF4Jアダプタ | 0 | 1 | 2020-10 | Java |
| [slf4j-nablarch-adaptor](https://github.com/nablarch/slf4j-nablarch-adaptor) | SLF4J→Nablarchログブリッジ | 0 | 1 | 2021-09 | Java |
| [nablarch-jboss-logging-adaptor](https://github.com/nablarch/nablarch-jboss-logging-adaptor) | JBoss Loggingアダプタ | 0 | 0 | 2020-10 | Java |
| [nablarch-jsr310-adaptor](https://github.com/nablarch/nablarch-jsr310-adaptor) | JSR310日時アダプタ | 0 | 0 | 2020-10 | Java |
| [nablarch-lettuce-adaptor](https://github.com/nablarch/nablarch-lettuce-adaptor) | Lettuce（Redis）キャッシュ | 0 | 0 | 2024-09 | Java |
| [nablarch-micrometer-adaptor](https://github.com/nablarch/nablarch-micrometer-adaptor) | Micrometerメトリクス | 0 | 2 | 2024-09 | Java |
| [nablarch-web-thymeleaf-adaptor](https://github.com/nablarch/nablarch-web-thymeleaf-adaptor) | Thymeleafテンプレート | 0 | 0 | 2018-04 | Java |
| [nablarch-mail-sender-freemarker-adaptor](https://github.com/nablarch/nablarch-mail-sender-freemarker-adaptor) | FreeMarkerメールテンプレート | 0 | 0 | 2018-04 | Java |
| [nablarch-mail-sender-thymeleaf-adaptor](https://github.com/nablarch/nablarch-mail-sender-thymeleaf-adaptor) | Thymeleafメールテンプレート | 0 | 0 | 2018-04 | Java |
| [nablarch-mail-sender-velocity-adaptor](https://github.com/nablarch/nablarch-mail-sender-velocity-adaptor) | Velocityメールテンプレート | 0 | 0 | 2020-09 | Java |

### 2.5 Examples / Samples（15リポジトリ）

サンプルアプリケーション・ビジネスサンプル。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-example-web](https://github.com/nablarch/nablarch-example-web) | Webアプリサンプル | 4 | 28 | 2025-08 | Java |
| [nablarch-example-thymeleaf-web](https://github.com/nablarch/nablarch-example-thymeleaf-web) | Thymeleaf Webサンプル | 0 | 2 | 2025-03 | HTML |
| [nablarch-example-rest](https://github.com/nablarch/nablarch-example-rest) | REST APIサンプル | 0 | 4 | 2025-03 | Java |
| [nablarch-example-batch](https://github.com/nablarch/nablarch-example-batch) | バッチサンプル | 2 | 3 | 2025-10 | Java |
| [nablarch-example-batch-ee](https://github.com/nablarch/nablarch-example-batch-ee) | Jakarta Batchサンプル | 0 | 4 | 2025-03 | Java |
| [nablarch-example-db-queue](https://github.com/nablarch/nablarch-example-db-queue) | DBキューサンプル | 0 | 1 | 2025-03 | Java |
| [nablarch-example-http-messaging](https://github.com/nablarch/nablarch-example-http-messaging) | HTTPメッセージングサンプル | 0 | 2 | 2025-03 | Java |
| [nablarch-example-http-messaging-send](https://github.com/nablarch/nablarch-example-http-messaging-send) | HTTP送信サンプル | 0 | 1 | 2025-03 | Java |
| [nablarch-example-mom-delayed-receive](https://github.com/nablarch/nablarch-example-mom-delayed-receive) | MOM遅延受信サンプル | 0 | 2 | 2025-03 | Java |
| [nablarch-example-mom-delayed-send](https://github.com/nablarch/nablarch-example-mom-delayed-send) | MOM遅延送信サンプル | 0 | 2 | 2025-03 | Java |
| [nablarch-example-mom-sync-receive](https://github.com/nablarch/nablarch-example-mom-sync-receive) | MOM同期受信サンプル | 0 | 1 | 2025-03 | Java |
| [nablarch-example-mom-sync-send-batch](https://github.com/nablarch/nablarch-example-mom-sync-send-batch) | MOM同期送信バッチサンプル | 0 | 1 | 2025-03 | Java |
| [nablarch-example-mom-testing-common](https://github.com/nablarch/nablarch-example-mom-testing-common) | MOMテスト共通 | 0 | 1 | 2025-03 | Java |
| [nablarch-example-workflow](https://github.com/nablarch/nablarch-example-workflow) | ワークフローサンプル | 0 | 2 | 2025-05 | Java |
| [nablarch-biz-sample-all](https://github.com/nablarch/nablarch-biz-sample-all) | 業務サンプル全集 | 1 | 2 | 2025-03 | Java |

### 2.6 Testing（9リポジトリ）

テスティングフレームワークとサポートツール。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | アーカイブ | 言語 |
|---|---|---|---|---|---|---|
| [nablarch-testing](https://github.com/nablarch/nablarch-testing) | テスティングFW | 0 | 1 | 2024-09 | - | Java |
| [nablarch-testing-junit5](https://github.com/nablarch/nablarch-testing-junit5) | JUnit 5拡張 | 0 | 0 | 2022-03 | - | Java |
| [nablarch-testing-rest](https://github.com/nablarch/nablarch-testing-rest) | RESTテスト | 0 | 0 | 2024-09 | - | Java |
| [nablarch-testing-jetty12](https://github.com/nablarch/nablarch-testing-jetty12) | Jetty 12テスト | 0 | 0 | 2024-09 | - | Java |
| [nablarch-test-support](https://github.com/nablarch/nablarch-test-support) | テストサポート | 0 | 0 | 2022-03 | - | Java |
| [nablarch-test-support-hereis](https://github.com/nablarch/nablarch-test-support-hereis) | Here-isテストサポート | 0 | 0 | 2017-02 | - | Java |
| [nablarch-integration-test](https://github.com/nablarch/nablarch-integration-test) | 統合テスト | 0 | 1 | 2020-10 | - | Java |
| [nablarch-testing-jetty9](https://github.com/nablarch/nablarch-testing-jetty9) | Jetty 9テスト | 0 | 1 | 2024-10 | **archived** | Java |
| [nablarch-testing-jetty6](https://github.com/nablarch/nablarch-testing-jetty6) | Jetty 6テスト | 0 | 1 | 2024-10 | **archived** | Java |

### 2.7 Build / Configuration / Tools（15リポジトリ）

ビルド設定、プロジェクトテンプレート、開発ツール。

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-parent](https://github.com/nablarch/nablarch-parent) | 親POM（BOM） | 0 | 0 | 2025-03 | - |
| [nablarch-profiles](https://github.com/nablarch/nablarch-profiles) | Mavenプロファイル | 0 | 0 | 2025-03 | Java |
| [nablarch-default-configuration](https://github.com/nablarch/nablarch-default-configuration) | デフォルト設定 | 0 | 0 | 2025-03 | Java |
| [nablarch-single-module-archetype](https://github.com/nablarch/nablarch-single-module-archetype) | Mavenアーキタイプ | 0 | 1 | 2025-03 | Java |
| [nablarch-toolbox](https://github.com/nablarch/nablarch-toolbox) | ツールボックス | 0 | 0 | 2020-10 | Java |
| [nablarch-unpublished-api-checker](https://github.com/nablarch/nablarch-unpublished-api-checker) | 非公開API検出 | 0 | 0 | 2025-03 | Java |
| [nablarch-openapi-generator](https://github.com/nablarch/nablarch-openapi-generator) | OpenAPI生成 | 0 | 0 | 2025-03 | Java |
| [sql-executor](https://github.com/nablarch/sql-executor) | SQL実行ツール | 1 | 0 | 2025-03 | JavaScript |
| [nablarch-plugins-bundle](https://github.com/nablarch/nablarch-plugins-bundle) | プラグインバンドル | 0 | 0 | 2025-05 | XSLT |
| [nablarch-plugins-bundle-test](https://github.com/nablarch/nablarch-plugins-bundle-test) | プラグインバンドルテスト | 0 | 0 | 2025-02 | Java |
| [nablarch-ui-development-template](https://github.com/nablarch/nablarch-ui-development-template) | UI開発テンプレート | 0 | 0 | 2025-05 | Java |
| [nablarch-ui-build-test-pj](https://github.com/nablarch/nablarch-ui-build-test-pj) | UIビルドテスト | 0 | 0 | 2025-02 | JavaScript |
| [nablarch-gradle-plugin](https://github.com/nablarch/nablarch-gradle-plugin) | Gradleプラグイン | 1 | 0 | 2022-03 | Groovy |
| [nablarch-intellij-plugin](https://github.com/nablarch/nablarch-intellij-plugin) | IntelliJ IDEAプラグイン | 1 | 1 | 2020-10 | Kotlin |
| [nablarch-tools-for-ai](https://github.com/nablarch/nablarch-tools-for-ai) | AI開発支援ツール | 0 | 1 | 2025-09 | - |

### 2.8 Documentation / Website（3リポジトリ）

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch](https://github.com/nablarch/nablarch) | Nablarch Information（メインリポジトリ） | 42 | 11 | 2025-08 | - |
| [nablarch-document](https://github.com/nablarch/nablarch-document) | 公式ドキュメント（HTML） | 3 | 9 | 2025-03 | HTML |
| [nablarch.github.io](https://github.com/nablarch/nablarch.github.io) | GitHub Pages公開サイト | 0 | 1 | 2025-05 | HTML |

### 2.9 その他（8リポジトリ）

| リポジトリ | 説明 | Stars | Forks | 最終更新 | 言語 |
|---|---|---|---|---|---|
| [nablarch-fw-scoped-dicontainer](https://github.com/nablarch/nablarch-fw-scoped-dicontainer) | スコープ付きDIコンテナ | 0 | 0 | 2020-09 | Java |
| [nablarch-backward-compatibility](https://github.com/nablarch/nablarch-backward-compatibility) | 後方互換性 | 0 | 0 | 2018-04 | Java |
| [dependabot-alert-test](https://github.com/nablarch/dependabot-alert-test) | Dependabotアラートテスト | 0 | 0 | 2025-05 | - |

### 2.10 アーカイブ済みリポジトリ（18リポジトリ）

| リポジトリ | 説明 | Stars | Forks | アーカイブ理由（推定） |
|---|---|---|---|---|
| [nablarch-etl](https://github.com/nablarch/nablarch-etl) | ETLフレームワーク | 2 | 1 | 機能廃止 |
| [nablarch-etl-maven-plugin](https://github.com/nablarch/nablarch-etl-maven-plugin) | ETL Mavenプラグイン | 0 | 0 | ETL廃止に伴い |
| [nablarch-etl-designer](https://github.com/nablarch/nablarch-etl-designer) | ETLデザイナー | 0 | 2 | ETL廃止に伴い |
| [nablarch-workflow](https://github.com/nablarch/nablarch-workflow) | ワークフローエンジン | 0 | 0 | 機能刷新 |
| [nablarch-workflow-tool](https://github.com/nablarch/nablarch-workflow-tool) | ワークフローツール | 0 | 0 | workflow廃止に伴い |
| [nablarch-messaging-simulator](https://github.com/nablarch/nablarch-messaging-simulator) | メッセージングシミュレータ | 0 | 0 | 機能廃止 |
| [nablarch-testing-jetty9](https://github.com/nablarch/nablarch-testing-jetty9) | Jetty 9テスト | 0 | 1 | Jetty 12へ移行 |
| [nablarch-testing-jetty6](https://github.com/nablarch/nablarch-testing-jetty6) | Jetty 6テスト | 0 | 1 | Jetty 12へ移行 |
| [nablarch-statistics-report](https://github.com/nablarch/nablarch-statistics-report) | 統計レポート | 0 | 0 | 機能廃止 |
| [nablarch-smime-integration](https://github.com/nablarch/nablarch-smime-integration) | S/MIME統合 | 0 | 0 | 機能廃止 |
| [nablarch-unpublished-api-checker-findbugs](https://github.com/nablarch/nablarch-unpublished-api-checker-findbugs) | FindBugs版API検出 | 0 | 0 | 新版に置換 |
| [nablarch-report](https://github.com/nablarch/nablarch-report) | レポート | 0 | 0 | 機能廃止 |
| [nablarch-report-sample](https://github.com/nablarch/nablarch-report-sample) | レポートサンプル | 0 | 1 | report廃止に伴い |
| [master](https://github.com/nablarch/master) | マスターデータ（XSLT） | 1 | 0 | 旧管理方式 |
| [nablarch-git-sync-script](https://github.com/nablarch/nablarch-git-sync-script) | Git同期スクリプト | 3 | 0 | 不要化 |
| [nablarch-ci-script](https://github.com/nablarch/nablarch-ci-script) | CIスクリプト | 0 | 0 | CI刷新 |
| [nablarch-log4j-adaptor](https://github.com/nablarch/nablarch-log4j-adaptor) | Log4jアダプタ | 0 | 0 | SLF4Jに移行 |
| [nablarch-module-version](https://github.com/nablarch/nablarch-module-version) | モジュールバージョン管理 | 0 | 0 | parent/BOMに統合 |

---

## 3. 主要リポジトリ詳細

### 3.1 nablarch（メインリポジトリ）

- **URL**: https://github.com/nablarch/nablarch
- **説明**: "Nablarch Information" — フレームワーク全体の説明・モジュール一覧を掲載
- **Stars**: 42 | **Forks**: 11
- **作成日**: 2016-08-19 | **最終更新**: 2025-08-13
- **内容**: README.mdにモジュール一覧テーブル（7カテゴリ）を掲載。LICENSE.txt。
- **Issue Tracking**: [Nablarch JIRA](https://nablarch.atlassian.net) を使用（GitHub Issuesではない）

README内のモジュール分類:
1. **Nablarch Core** — 10モジュール（基盤ライブラリ）
2. **Nablarch Framework** — 14モジュール（実行制御・ハンドラ）
3. **Nablarch Common Component** — 15モジュール（業務共通）
4. **Nablarch Adaptor** — 8+ モジュール（外部連携）
5. **Nablarch Framework Examples** — サンプルアプリケーション
6. **Nablarch Example Implementations** — 業務ユースケース実装例
7. **Misc** — その他ツール

### 3.2 nablarch-fw（コアフレームワーク）

- **URL**: https://github.com/nablarch/nablarch-fw
- **説明**: Nablarch Handlers — ハンドラキューパターンの実行制御基盤
- **Stars**: 0 | **Forks**: 1
- **最終更新**: 2021-03-30
- **役割**: リクエスト処理のハンドラキューを管理。全アプリケーション種別（Web、バッチ、メッセージング）共通の基盤

### 3.3 nablarch-fw-web（Webフレームワーク）

- **URL**: https://github.com/nablarch/nablarch-fw-web
- **説明**: Web統合基盤
- **Stars**: 0 | **Forks**: 0
- **最終更新**: 2025-03-27
- **役割**: Webアプリケーション（オンライン画面）のHTTPリクエスト/レスポンス処理

### 3.4 nablarch-fw-batch（バッチフレームワーク）

- **URL**: https://github.com/nablarch/nablarch-fw-batch
- **説明**: バッチアプリケーション基盤
- **Stars**: 0 | **Forks**: 1
- **最終更新**: 2025-03-27
- **役割**: Nablarch独自のバッチ実行制御。nablarch-fw-batch-ee（Jakarta Batch準拠）とは別

### 3.5 nablarch-fw-jaxrs（JAX-RSフレームワーク）

- **URL**: https://github.com/nablarch/nablarch-fw-jaxrs
- **説明**: RESTful Web Services
- **Stars**: 0 | **Forks**: 1
- **最終更新**: 2025-03-27
- **役割**: Jakarta RESTful Web Services（JAX-RS）準拠のREST API基盤

### 3.6 nablarch-core（コアライブラリ）

- **URL**: https://github.com/nablarch/nablarch-core
- **説明**: Nablarch Core
- **Stars**: 2 | **Forks**: 5
- **最終更新**: 2025-03-27
- **役割**: フレームワーク全体の基盤クラス。ハンドラインターフェース、リクエスト/レスポンス等

### 3.7 nablarch-common-dao（ユニバーサルDAO）

- **URL**: https://github.com/nablarch/nablarch-common-dao
- **説明**: エンティティクラスからSQL自動生成
- **Stars**: 0 | **Forks**: 1
- **最終更新**: 2025-03-27
- **役割**: アノテーションベースでエンティティからCRUD SQLを自動生成

### 3.8 nablarch-testing（テスティングFW）

- **URL**: https://github.com/nablarch/nablarch-testing
- **説明**: テスティングフレームワーク
- **Stars**: 0 | **Forks**: 1
- **最終更新**: 2024-09-27
- **役割**: Excel連携JUnitテスト。テストデータをExcelで定義しJUnitで自動実行

### 3.9 nablarch-example-web（Webサンプル）

- **URL**: https://github.com/nablarch/nablarch-example-web
- **説明**: Webアプリケーションサンプル
- **Stars**: 4 | **Forks**: 28（最多フォーク数）
- **最終更新**: 2025-08-13
- **役割**: Nablarch Webアプリケーションの参照実装。新規開発者の学習起点

### 3.10 nablarch-parent（BOM / 親POM）

- **URL**: https://github.com/nablarch/nablarch-parent
- **Stars**: 0 | **Forks**: 0
- **最終更新**: 2025-03-27
- **役割**: 全モジュールのバージョンを一元管理するBOM（Bill of Materials）

---

## 4. nablarch-tools-for-ai の現状

| 項目 | 値 |
|---|---|
| **URL** | https://github.com/nablarch/nablarch-tools-for-ai |
| **作成日** | 2025-09-01 |
| **最終更新** | 2025-09-01（master）/ 2026-01-29（PR活動） |
| **最終push** | 2026-01-15 |
| **ライセンス** | Apache License 2.0 |
| **デフォルトブランチ** | master |
| **Stars** | 0 |
| **Forks** | 1（kiyotisによるフォーク） |
| **空か** | 否（LICENSE + README.md） |
| **Issues有効** | はい |

### ファイル内容（master）

- `LICENSE` — Apache-2.0
- `README.md` — 「Nablarchを使用した開発において、生成AIを活用して生産性を向上させることを目的としたツールを格納する場所」

### オープンPR（活発な開発の兆候）

| 項目 | 値 |
|---|---|
| **PR #1** | [NTF（Nablarch Testing Framework）のコンテキスト情報を作成](https://github.com/nablarch/nablarch-tools-for-ai/pull/1) |
| **作成者** | [kiyotis](https://github.com/kiyotis) |
| **作成日** | 2026-01-21 |
| **最終更新** | 2026-01-29 |
| **ブランチ** | `feature/add-ntf-context` → `develop` |
| **状態** | Open（未マージ） |

**重要な発見**: TISのコントリビューター（kiyotis）が2026年1月に**Nablarch Testing Frameworkのコンテキスト情報**をAIツール用に作成するPRを提出している。これは**TISが実際にNablarchのAI開発支援を進めている**直接的な証拠である。PR内容はNTFのコンテキスト情報の作成であり、RAGシステムや生成AIによるNablarch開発支援を想定していると推測される。

---

## 5. メトリクス情報

### 5.1 スター数ランキング（Top 10）

| 順位 | リポジトリ | Stars | Forks |
|---|---|---|---|
| 1 | [nablarch](https://github.com/nablarch/nablarch) | 42 | 11 |
| 2 | [nablarch-example-web](https://github.com/nablarch/nablarch-example-web) | 4 | 28 |
| 3 | [nablarch-document](https://github.com/nablarch/nablarch-document) | 3 | 9 |
| 4 | [nablarch-git-sync-script](https://github.com/nablarch/nablarch-git-sync-script) | 3 | 0 |
| 5 | [nablarch-core](https://github.com/nablarch/nablarch-core) | 2 | 5 |
| 6 | [nablarch-example-batch](https://github.com/nablarch/nablarch-example-batch) | 2 | 3 |
| 7 | [nablarch-etl](https://github.com/nablarch/nablarch-etl) | 2 | 1 |
| 8 | [nablarch-core-beans](https://github.com/nablarch/nablarch-core-beans) | 1 | 1 |
| 9 | [nablarch-biz-sample-all](https://github.com/nablarch/nablarch-biz-sample-all) | 1 | 2 |
| 10 | [nablarch-intellij-plugin](https://github.com/nablarch/nablarch-intellij-plugin) | 1 | 1 |

### 5.2 フォーク数ランキング（Top 5）

| 順位 | リポジトリ | Forks | 考察 |
|---|---|---|---|
| 1 | [nablarch-example-web](https://github.com/nablarch/nablarch-example-web) | 28 | 最も利用されるサンプル |
| 2 | [nablarch](https://github.com/nablarch/nablarch) | 11 | メインリポジトリ |
| 3 | [nablarch-document](https://github.com/nablarch/nablarch-document) | 9 | ドキュメント貢献 |
| 4 | [nablarch-core](https://github.com/nablarch/nablarch-core) | 5 | コア開発参加 |
| 5 | [nablarch-example-batch-ee](https://github.com/nablarch/nablarch-example-batch-ee) / [nablarch-example-rest](https://github.com/nablarch/nablarch-example-rest) | 4 | サンプル利用 |

### 5.3 コントリビューター情報（主要リポジトリ）

| リポジトリ | コントリビューター数 | 主要コントリビューター |
|---|---|---|
| nablarch-core | 20+ | siosio, opengl-8080, mseko, tamutamu, kankichi00 |
| nablarch-fw | 13 | mseko, tamutamu, siosio, backpaper0, opengl-8080 |
| nablarch-document | 20+ | siosio, kumagoro1202, tisuchida, yama-gen, opengl-8080 |
| nablarch（メイン） | 15 | kumagoro1202, opengl-8080, koyi2016, sambatriste, mseko |

**頻出コントリビューター**（複数リポジトリに跨がるアクティブ開発者）:
- **siosio** — Core, Document で最多コントリビューション
- **opengl-8080** — Core, FW, Document, メインリポジトリ全てに参加
- **mseko** — Core, FW で主要コントリビューター
- **kumagoro1202** — Document, メインリポジトリで最多
- **tisuchida** — Core, Document
- **kiyotis** — Core, FW, Document + **nablarch-tools-for-ai のPR作成者**

### 5.4 使用言語の分布

| 言語 | リポジトリ数 | 割合 |
|---|---|---|
| Java | 91 | 86% |
| HTML | 3 | 3% |
| JavaScript | 3 | 3% |
| XSLT | 2 | 2% |
| Kotlin | 1 | 1% |
| Groovy | 1 | 1% |
| Shell | 2 | 2% |
| なし（データのみ） | 3 | 3% |

### 5.5 活発さの指標

| 期間 | 更新されたリポジトリ数 | 状態 |
|---|---|---|
| 2025年以降 | 47 | アクティブ |
| 2024年 | 10 | メンテナンス |
| 2023年以前 | 31 | 低活動 |
| アーカイブ | 18 | 廃止 |

**最新更新リポジトリ（2025年後半〜2026年）**:
- nablarch-etl — 2025-12-27（アーカイブだが最近更新）
- nablarch-example-batch — 2025-10-25
- nablarch-tools-for-ai — 2025-09-01（PR活動は2026-01まで継続）
- nablarch-example-web — 2025-08-13
- nablarch — 2025-08-13

---

## 6. リリース履歴

### 6.1 nablarch-parent タグ一覧

BOMバージョン管理は `nablarch-parent` リポジトリのタグで管理されている。

| バージョン | 系統 | 備考 |
|---|---|---|
| **6u3** | Nablarch 6 | 最新安定版 |
| 6u2 | Nablarch 6 | 正式リリース後の最初のバージョン |
| 6u1 | Nablarch 6 | 先行リリース |
| 6 | Nablarch 6 | 先行リリース |
| 5u26 | Nablarch 5 | Nablarch 5系最終 |
| 5u25 | Nablarch 5 | |
| 5u24 | Nablarch 5 | |
| 5u23 | Nablarch 5 | |
| 5u22 | Nablarch 5 | |
| 5u21 | Nablarch 5 | |
| 5u20 | Nablarch 5 | |
| 5u19 | Nablarch 5 | |
| 5u18 | Nablarch 5 | |
| 5u17 | Nablarch 5 | |
| 5u16 | Nablarch 5 | |
| 5u15 | Nablarch 5 | |
| 5u14 | Nablarch 5 | |
| 5u13 | Nablarch 5 | |
| 5u12 | Nablarch 5 | 取得できた最古のタグ |

### 6.2 Nablarch 6 の技術要件

| 項目 | 値 |
|---|---|
| **最新バージョン** | 6u3 |
| **Java要件** | Java 17以上（モジュールはJava 17でコンパイル）|
| **Java 21** | サポート（セットアップ手順あり） |
| **Jakarta EE** | Jakarta EE 10準拠 |
| **名前空間** | javax.* → jakarta.* に移行 |

### 6.3 Nablarch 5 → 6 の主要変更点

| 変更点 | 詳細 |
|---|---|
| **Java EE → Jakarta EE 10** | 名前空間を javax.* から jakarta.* に全面移行 |
| **Java 17必須** | Java 17以上が必要（Nablarch 5はJava 8対応） |
| **Jetty更新** | nablarch-testing-jetty6 → nablarch-testing-jetty12 |
| **Hibernate Validator** | 5.3.6.Final → 8.0.0.Final |
| **waitt-maven-plugin廃止** | jetty-ee10-maven-pluginに置換 |
| **gsp-dba-maven-plugin** | 5.1.0でJakarta EE / Nablarch 6u2対応 |
| **アーキタイプバージョン** | `-DarchetypeVersion=6u3` で生成 |

### 6.4 バージョン方針

- Nablarch 6のバージョン6/6u1は**先行リリース**（pre-release）
- **6u2**が正式リリース後の最初の安定バージョン
- 6u3以降はインクリメンタルアップデート
- Nablarch 5系は5u12〜5u26まで長期メンテナンスが行われた（15バージョン）

---

## 7. リポジトリ間の依存関係

### 7.1 レイヤー構造

公式READMEのモジュール分類と命名規則から推定される依存関係:

```
┌─────────────────────────────────────────────┐
│               Examples / Samples             │
│  nablarch-example-web, -batch, -rest, etc.  │
├─────────────────────────────────────────────┤
│              Nablarch Adaptors               │
│  nablarch-*-adaptor, slf4j-nablarch-adaptor │
├─────────────────────────────────────────────┤
│          Nablarch Common Components          │
│  nablarch-common-dao, -auth, -code, etc.    │
├─────────────────────────────────────────────┤
│            Nablarch Framework                │
│  nablarch-fw, -fw-web, -fw-batch, etc.      │
├─────────────────────────────────────────────┤
│              Nablarch Core                   │
│  nablarch-core, -core-jdbc, -core-beans...  │
├─────────────────────────────────────────────┤
│         Build / Configuration                │
│  nablarch-parent, nablarch-profiles          │
└─────────────────────────────────────────────┘
```

### 7.2 依存関係の詳細

```
nablarch-parent（BOM）
  └─→ 全モジュールのバージョンを管理

nablarch-core
  └─→ nablarch-core-beans, nablarch-core-repository 等の基盤

nablarch-fw（ハンドラキュー）
  └─→ nablarch-core に依存
  └─→ nablarch-fw-web, -fw-batch, -fw-jaxrs 等が継承

nablarch-fw-web
  └─→ nablarch-fw に依存
  └─→ nablarch-fw-web-tag, -web-dbstore, -web-extension 等が拡張

nablarch-common-dao
  └─→ nablarch-core-jdbc に依存

nablarch-testing
  └─→ nablarch-core, nablarch-fw に依存
  └─→ nablarch-testing-jetty12 がWebテスト用に拡張

nablarch-example-web
  └─→ nablarch-fw-web, nablarch-common-dao 等を使用
```

### 7.3 命名規則

| プレフィックス | 意味 |
|---|---|
| `nablarch-core-*` | コア基盤ライブラリ |
| `nablarch-fw-*` | フレームワーク（実行制御） |
| `nablarch-fw-web-*` | Web固有の拡張 |
| `nablarch-fw-batch-*` | バッチ固有の拡張 |
| `nablarch-fw-messaging-*` | メッセージング固有の拡張 |
| `nablarch-common-*` | 共通業務コンポーネント |
| `nablarch-common-*-jdbc` | JDBCによる実装 |
| `nablarch-*-adaptor` | 外部ライブラリアダプタ |
| `nablarch-example-*` | サンプルアプリケーション |
| `nablarch-testing-*` | テスティングフレームワーク拡張 |
| `nablarch-mail-sender-*-adaptor` | メールテンプレートエンジンアダプタ |

---

## 参考リンク

- [Nablarch GitHub Organization](https://github.com/nablarch)
- [Nablarch 公式ドキュメント (6u3)](https://nablarch.github.io/docs/LATEST/doc/en/)
- [Nablarch 5→6 移行ガイド](https://nablarch.github.io/docs/LATEST/doc/en/migration/index.html)
- [Jakarta EE仕様名について](https://nablarch.github.io/docs/LATEST/doc/en/jakarta_ee/index.html)
- [Java 21セットアップ](https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/blank_project/setup_blankProject/setup_Java21.html)
- [nablarch-tools-for-ai PR #1](https://github.com/nablarch/nablarch-tools-for-ai/pull/1)
