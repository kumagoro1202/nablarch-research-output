# Nablarch 開発標準・ガイド・テスティングFW ナレッジベース

> **調査日**: 2026-02-01
> **調査者**: ashigaru6 (cmd_007 / subtask_021)
> **対象**: Nablarch開発標準、コーディングガイド、テスティングフレームワーク

---

## 目次

1. [開発標準（nablarch-development-standards）](#1-開発標準nablarch-development-standards)
2. [スタイルガイド（nablarch-style-guide）](#2-スタイルガイドnablarch-style-guide)
3. [テスティングフレームワーク（nablarch-testing）](#3-テスティングフレームワークnablarch-testing)
4. [公式ドキュメント・静的解析ツール](#4-公式ドキュメント静的解析ツール)
5. [サンプルプロジェクト構成パターン](#5-サンプルプロジェクト構成パターン)

---

## 1. 開発標準（nablarch-development-standards）

### 1.1 概要

- **GitHub Organization**: `nablarch-development-standards`（`nablarch` orgとは別）
- **リポジトリ**: https://github.com/nablarch-development-standards/nablarch-development-standards
- **ブランチ**: `main`
- **Stars**: 22 | **Forks**: 14
- **最新バージョン**: 2.3 (2024-09-30)
- **ライセンス**: CC BY-SA 4.0（ドキュメント）、Apache 2.0（ツール）

Nablarch開発標準は、設計ドキュメント・プログラムコードの品質を向上・標準化するためのガイドライン。TIS Inc.が開発するエンタープライズJavaフレームワーク「Nablarch」に最適化されている。

### 1.2 設計思想

- **設計書の情報を最小化**: 実装に必要な仕様のみ記録し、内部設計書（プログラム仕様書）を排除して生産性向上
- **仕様レビューと設計実装の両立**: 1つのドキュメントで仕様レビュー者・実装者の両方をカバー（印刷範囲内が仕様レビュー用、範囲外が実装者用）
- **擬似SQL**: SQLを理解できない関係者のため、日本語表現のSQL構造を設計書で使用

### 1.3 システム階層構造

```
システム
  > 業務大分類（例: 会員管理、債権管理）
    > サブシステム（例: 契約管理、請求、入金）
      > 機能（例: 新規契約、契約変更）
        > 取引（最小業務単位 / ビジネストランザクション）
```

### 1.4 4つのコンテンツ

| コンテンツ | 概要 | パス |
|-----------|------|------|
| **010 開発プロセス標準** | Nablarchシステム開発の標準WBS | `010_開発プロセス標準/` |
| **020 アプリケーション開発標準** | UI標準、コーディング規約、単体テスト標準 | `020_アプリケーション開発標準/` |
| **030 設計ドキュメント** | 設計書のフォーマット（テンプレート）とサンプル | `030_設計ドキュメント/` |
| **開発プロセス支援ツール** | ソースコード生成、設計書自動化ツール | 別リポジトリ |

### 1.5 アプリケーション開発標準の詳細

| カテゴリ | ドキュメント | 説明 |
|---------|-------------|------|
| 設計標準 | DB設計標準.docx | DB設計のガイドライン |
| 設計標準 | UI標準(画面).xlsx | 画面レイアウト標準仕様 |
| 設計標準 | UI標準(画面)別冊_UI部品カタログ.xlsx | 入出力UI部品の標準仕様 |
| 設計標準 | UI標準(帳票).xlsx | 帳票レイアウト標準仕様 |
| 設計標準 | 共通コンポーネント設計標準.docx | 共通ユーティリティの設計標準 |
| テスト標準 | 単体テスト標準.xlsx | 単体テストの設計方針・アプローチ |

### 1.6 設計ドキュメント（030）

11カテゴリの設計書テンプレートとサンプルを提供：

1. **標準書式** — Excel/Wordテンプレート
2. **システム機能設計** — 処理フロー、機能一覧、機能設計書（画面/バッチ/Webサービス/メッセージング）
3. **共通コンポーネント設計** — 一覧、設計書
4. **インタフェース設計** — WebサービスAPI一覧、外部IF設計書（HTTP/ファイル/JSON/XML等）
5. **画面設計** — 画面一覧、画面遷移図
6. **帳票設計** — 帳票一覧、帳票設計書
7. **メール設計** — メール一覧、メール設計書
8. **メッセージ設計** — メッセージ設計書
9. **コード設計** — コード設計書
10. **データモデル設計** — テーブル一覧/定義書、ドメイン定義書、採番一覧
11. **テスト仕様書** — 単体テスト仕様書（画面/バッチ/Webサービス/メッセージング/共通コンポーネント）

**ソース**: https://github.com/nablarch-development-standards/nablarch-development-standards/tree/main/030_%E8%A8%AD%E8%A8%88%E3%83%89%E3%82%AD%E3%83%A5%E3%83%A1%E3%83%B3%E3%83%88

### 1.7 開発プロセス支援ツール（nablarch-development-standards-tools）

- **リポジトリ**: https://github.com/nablarch-development-standards/nablarch-development-standards-tools
- **バージョン**: 1.3 (2024-09-30)
- **ライセンス**: Apache 2.0

#### ツール一覧

| カテゴリ | ツール | 説明 |
|---------|-------|------|
| バッチ開発補助 | シェルスクリプト自動生成ツール.xlsm | ジョブスケジューラが呼び出すシェルスクリプトを自動生成 |
| バッチ開発補助 | scripts/ | 事前構築済みスクリプトライブラリ（バッチ実行、メッセージング、ファイル操作等） |
| 登録用データ作成 | コードテーブル登録用データ出力ツール | コード設計書からCSVを生成 |
| 登録用データ作成 | メッセージテーブル登録用データ作成ツール | メッセージ設計書からCSVを生成 |
| 登録用データ作成 | リクエストテーブル登録用データ作成ツール | リクエストID一覧からCSVを生成 |
| 外部IF設計 | フォーマット定義ファイル自動生成ツール | 外部IF設計書からフォーマット定義を生成 |
| DB設計 | テーブル定義・ドメイン定義整合性チェックツール | テーブル定義のドメイン存在・型整合性検証 |
| DB設計 | 排他制御主キークラス自動生成ツール | テーブル定義から排他制御クラスを生成 |
| DB設計 | ドメインクラス作成ツール | ドメイン定義からJavaクラスを生成 |

#### 外部ツール（別リポジトリ）

- **Jakarta Server Pages 静的分析ツール**: nablarch-testingにバンドル
- **gsp-dba-maven-plugin**: DBA定型作業の自動化 ([coastland/gsp-dba-maven-plugin](https://github.com/coastland/gsp-dba-maven-plugin))
- **Nablarch sql-executor**: Nablarch特殊構文対応SQLインタラクティブ実行ツール ([nablarch/sql-executor](https://github.com/nablarch/sql-executor))

### 1.8 バージョン履歴

| バージョン | 日付 | 主な変更 |
|-----------|------|---------|
| 2.3 | 2024-09-30 | Jakarta Bean Validation向けドメイン定義更新、JakartaEE10対応、UI/テスト標準更新 |
| 2.2 | 2022-10-31 | UI標準更新（OS/ブラウザサポート、PRGパターン）、DBMSをPostgreSQLに変更 |
| 2.1 | 2021-10-31 | JSON/XML外部IF設計書フォーマット改善 |
| 2.0 | 2020-08-14 | 英語版追加（部分） |
| 1.0 | 2018-10-09 | 初版 |

---

## 2. スタイルガイド（nablarch-style-guide）

### 2.1 概要

- **リポジトリ（アーカイブ済み）**: https://github.com/nablarch-development-standards/nablarch-style-guide
- **移行先（現行）**: https://github.com/Fintan-contents/coding-standards
- **ブランチ**: `master`
- **言語**: 日本語（primary）、英語あり

2022年10月にFintan-contents/coding-standardsに移行。Nablarch依存が少なく、他フレームワークでも利用可能。

### 2.2 対象言語

| 言語 | ドキュメント |
|------|-------------|
| **Java** | コーディング規約（最も包括的）、静的解析設定 |
| JavaScript | コーディング規約、利用可能API一覧 |
| JSP | コーディング規約 |
| SQL | コーディング規約 |
| Shell Script | 開発標準 |

### 2.3 Java コーディング規約の前提条件

以下3つの作業が完了していることが前提：
1. **Eclipse コードフォーマッタ**の適用（`nablarch-code-formatter.xml`）
2. **Checkstyle** 違反の解消
3. **SpotBugs** 指摘の解消

### 2.4 設計指針（3原則）

1. **実行時エラーよりコンパイルエラー** — 可能な限りコンパイル時にバグを検出
2. **変更可能な状態の削減** — ミュータブルなフィールド・再代入可能な変数を最小化
3. **状態変更の局所化** — 状態変更をできるだけローカルに保つ

### 2.5 Java命名規則

#### クラス命名

| 役割 | 命名ルール |
|------|-----------|
| Action | 機能名 + `Action` |
| Entity | テーブル物理名をUpperCamelCase |
| 業務基底Form | 機能名 + `FormBase` |
| 業務Form | 機能名 + `Form` |
| Validator | 機能名 + `Validator` |
| DTO | データ型名 + `Dto` |
| 排他制御 | テーブル物理名 + `ExclusiveControl` |
| テスト | テスト対象クラス名 + `Test` |
| リクエスト単体テスト | Actionクラス名 + `RequestTest` |

- **Checkstyleルール**: `^[A-Z][a-zA-Z0-9]*$`
- **抽象クラス**: `^(Abstract.*|.*FormBase)$`

#### メソッド命名

| 機能 | パターン | 例 |
|------|---------|-----|
| インスタンス生成 | `create` + 対象 | `createItemCode` |
| 型変換 | `to` + 型 | `toString`, `toItemCode` |
| 型として扱う | `as` + 型 | `asReadLock`, `asList` |
| 包含チェック | `contains` + 対象 | `containsKey` |
| 能力チェック | `can` + 操作 | `canRead`, `canEncode` |
| 状態チェック | `is` + 状態 | `isClosed`, `isEmpty` |

#### 変数命名

- **Checkstyleルール（MemberName）**: `^[a-z][a-zA-Z0-9]*$`（lowerCamelCase）
- **定数（ConstantName）**: `^[A-Z](_?[A-Z0-9]+)*$`（UPPER_SNAKE_CASE）
- **ローカルfinal変数**: lowerCamelCase（UPPER_SNAKE_CASEではない）
- **型パラメータ**: 単一大文字（例: `T`, `E`）

**ソース**: https://github.com/nablarch-development-standards/nablarch-style-guide/blob/master/en/java/java-style-guide.md

### 2.6 パッケージ構成

システム設計のステレオタイプに従いパッケージを構成：

| 役割 | パッケージ例 |
|------|-------------|
| Action | `com.example.action` |
| Entity | `com.example.entity` |
| 業務Form | `com.example.form` |
| Validator | `com.example.validation` |
| DTO | `com.example.dto` |

- **PackageName Checkstyleルール**: `^[a-z]+(\.[a-z_][a-z0-9_]*)*$`（severity: error）
- **PackageDeclaration**: 全クラスにpackage宣言必須（デフォルトパッケージ禁止）

### 2.7 例外処理パターン

#### 禁止事項

| ルール | 理由 |
|--------|------|
| アプリケーションプログラマが独自に例外クラスを作成しない | アーキテクト相談必須 |
| `java.lang.Exception` をthrowしない | 具体的な例外クラスを使用 |
| try-catchを条件分岐に使わない | if文を使用（性能上の理由） |
| 汎用例外をcatchしない | `Exception`/`Throwable`/`RuntimeException`/`Error`のcatch禁止（Checkstyle `IllegalCatch`、severity: error） |
| 汎用例外をthrows宣言しない | 同上4型の宣言禁止（Checkstyle `IllegalThrows`、severity: error） |

#### 必須事項

- 例外処理はプロジェクトのシステム設計に従い**一貫して**コーディング
- クローズ可能リソースには `try-with-resources` を使用
- 空のcatchブロック禁止（理由をコメントに記述）
- Javadocの`@throws`は例外が発生する**条件**を記述

### 2.8 ロギング規約

- **機密情報のマスク**: パスワード等をログに出力しない
- シリアライズ時の機密情報排除: `transient`キーワード使用

```java
public class LoginForm implements Serializable {
    private String username;
    private transient String password;  // シリアライズから除外
}
```

### 2.9 禁止事項一覧（全16項目）

| # | 禁止事項 |
|---|---------|
| 1 | 式の途中でのインクリメント/デクリメント |
| 2 | フィールドを一時変数として使用 |
| 3 | サブクラスでのフィールドシャドウイング |
| 4 | コレクション/配列メソッドでnullを返す（空コレクションを返す） |
| 5 | コンストラクタ内での自クラスメソッド呼び出し |
| 6 | インスタンス経由でstaticメソッドを呼び出す |
| 7 | 独自例外クラスの作成（アーキテクト承認なし） |
| 8 | `java.lang.Exception` のthrow |
| 9 | try-catchでの条件分岐 |
| 10 | ログ/シリアライズでの機密情報出力 |
| 11 | レガシーJava API使用（StringBuffer, Vector, Hashtable等） |
| 12 | 直接リフレクション使用 |
| 13 | クラス状態の不整合 |
| 14 | 無断スレッド作成 |
| 15 | 名前=値の定数（意味のある名前を使用） |

### 2.10 推奨事項（全14項目）

- 最小限のアクセス修飾子
- インスタンス変数は `private`
- ローカル変数のスコープを狭く
- `final` を積極使用（再代入回避）
- 引数の状態を変更しない
- null可能な戻り値には `Optional` を検討
- コレクションには型パラメータを必ずバインド
- コレクション操作にStream APIを検討
- `@Override` を常に付与

**ソース**: https://github.com/nablarch-development-standards/nablarch-style-guide/blob/master/en/java/java-style-guide.md

---

## 3. テスティングフレームワーク（nablarch-testing）

### 3.1 概要

| リポジトリ | 目的 | Maven Artifact |
|-----------|------|---------------|
| [nablarch/nablarch-testing](https://github.com/nablarch/nablarch-testing) | コアテスティングフレームワーク | `nablarch-testing` (v2.2.0) |
| [nablarch/nablarch-testing-junit5](https://github.com/nablarch/nablarch-testing-junit5) | JUnit 5拡張モジュール | `nablarch-testing-junit5` |
| [nablarch/nablarch-testing-rest](https://github.com/nablarch/nablarch-testing-rest) | RESTful Webサービステスト | `nablarch-testing-rest` |

JUnit 4をベースに構築された自動テストフレームワーク。**Excelファイル(.xls/.xlsx)をテストデータの主要媒体**として使用する点が最大の特徴。DB setup/verification、HTTPリクエストパラメータ、期待レスポンス、ファイルI/O、メッセージングをカバー。

**主要依存**: Apache POI 3.8（Excel読込）、JUnit 4.13.1、H2 Database、ActiveMQ Artemis（メッセージング）

### 3.2 Excelベーステストデータ機構

#### ファイル配置規約

- Excelファイルはテストクラスと**同名**（例: `MyActionTest.xls` / `MyActionTest.java`）
- テストソースコードと**同一ディレクトリ**に配置
- 各**ワークシート**が1つの**テストメソッド**に対応（シート名 = メソッド名）
- ベースパスは `nablarch.test.resource-root` で設定可能（デフォルト: `test/java/`）

#### データ型（DataType列挙）

各データブロックの先頭行は `DataType=identifier` 形式のヘッダ行：

| DataType | Excelマーカー | 用途 |
|----------|--------------|------|
| `SETUP_TABLE_DATA` | `SETUP_TABLE` | テスト実行前にDBに挿入する行データ |
| `EXPECTED_TABLE_DATA` | `EXPECTED_TABLE` | テスト後の期待DB状態（指定カラムのみ比較） |
| `EXPECTED_COMPLETED` | `EXPECTED_COMPLETE_TABLE` | テスト後の期待DB状態（省略カラムはデフォルト値で補完） |
| `LIST_MAP` | `LIST_MAP` | キー値ペアコレクション（HTTPパラメータ、テストケース定義等） |
| `SETUP_FIXED` | `SETUP_FIXED` | 固定長ファイル準備データ |
| `EXPECTED_FIXED` | `EXPECTED_FIXED` | 期待固定長ファイル内容 |
| `SETUP_VARIABLE` | `SETUP_VARIABLE` | 可変長ファイル準備データ |
| `EXPECTED_VARIABLE` | `EXPECTED_VARIABLE` | 期待可変長ファイル内容 |
| `MESSAGE` | `MESSAGE` | メッセージングテストデータ |
| `EXPECTED_REQUEST_HEADER_MESSAGES` | `EXPECTED_REQUEST_HEADER_MESSAGES` | 期待リクエストメッセージヘッダ |
| `EXPECTED_REQUEST_BODY_MESSAGES` | `EXPECTED_REQUEST_BODY_MESSAGES` | 期待リクエストメッセージボディ |
| `RESPONSE_HEADER_MESSAGES` | `RESPONSE_HEADER_MESSAGES` | レスポンスヘッダメッセージデータ |
| `RESPONSE_BODY_MESSAGES` | `RESPONSE_BODY_MESSAGES` | レスポンスボディメッセージデータ |

#### Excel読込パイプライン

```
Excel File (.xls/.xlsx)
    │
    ▼
PoiXlsReader (TestDataReader実装)
    │  - Apache POI WorkbookFactoryでワークブックを開く
    │  - シート名（"ClassName/SheetName"形式）でシートを取得
    │  - 行単位で読込、空白行・コメント行（"//"開始）をスキップ
    │  - LRUキャッシュ（サイズ=1）でワークブックをキャッシュ
    │
    ▼
BasicTestDataParser (TestDataParser実装)
    │  - DataType別の専用パーサーに委譲:
    │    - TableDataParser (SETUP_TABLE, EXPECTED_TABLE)
    │    - ListMapParser (LIST_MAP)
    │    - FixedLengthFileParser, VariableLengthFileParser
    │    - MessageParser, SendSyncMessageParser, GroupMessageParser
    │  - TestDataInterpreters（特殊値処理）を適用
    │
    ▼
ドメインオブジェクト
    - TableData (DBデータ)
    - List<Map<String, String>> (LIST_MAP)
    - DataFile / FixedLengthFile / VariableLengthFile
    - MessagePool (メッセージング)
```

#### 特殊セル値

| 表記 | 意味 |
|------|------|
| `null`（大文字小文字問わず） | Null値 |
| `"quoted text"` | リテラル文字列（引用符は除去） |
| `${systemTime}` / `${updateTime}` | 現在のシステムタイムスタンプ |
| `${setUpTime}` | 設定から取得した固定タイムスタンプ |
| `${characterSet,length}` | 指定文字種・長さの自動生成文字列 |
| `${binaryFile:path}` | バイナリファイルデータ（BLOBカラム用） |
| `\\r` / `\\n` | キャリッジリターン / ラインフィード |
| `//comment` | コメント（その列以降無視） |
| `[columnName]` | マーカーカラム（テストデータから除外、視覚用のみ） |

**注意**: 全セルはテキスト（文字列）書式で記述必須。数値書式等は予期しない動作の原因になる。

#### テストデータインタプリタ

特殊値処理のチェーン：
- `BasicJapaneseCharacterInterpreter`: 日本語テスト文字列生成
- `BinaryFileInterpreter`: バイナリファイル参照処理
- `DateTimeInterpreter`: 日時プレースホルダ処理
- `LineSeparatorInterpreter`: 改行記法処理
- `NullInterpreter`: null値処理
- `QuotationTrimmer`: 引用符除去

### 3.3 テストクラス継承階層

```
TestEventDispatcher (JUnit @Before/@After/@BeforeClass/@AfterClass ライフサイクル)
├── TestSupport (基底: Excelデータアクセス、ThreadContext、パラメータマップ)
│
├── DbAccessTestSupport (DB setup、テーブルアサーション、トランザクション管理)
│
├── EntityTestSupport (Entity/Formバリデーションテスト)
│   ├── testValidateAndConvert() -- Nablarch Validation
│   ├── testBeanValidation() -- Bean Validation (JSR-380)
│   ├── testSingleValidation() -- 単一フィールドバリデーション
│   ├── testValidateCharsetAndLength() -- 文字種・文字長バリデーション
│   └── testSetterAndGetter() / testConstructorAndGetter()
│
├── HttpRequestTestSupport (HTTP リクエスト実行、組込サーバ使用)
│   └── AbstractHttpRequestTestTemplate<INF extends TestCaseInfo>
│       └── BasicHttpRequestTestTemplate (便利サブクラス)
│
├── StandaloneTestSupportTemplate (スタンドアロン/バッチ/メッセージングテストテンプレート)
│   ├── BatchRequestTestSupport (バッチプロセステスト)
│   └── MessagingRequestTestSupport (メッセージングプロセステスト)
│
├── IntegrationTestSupport (統合テストオーケストレータ)
│
└── SimpleRestTestSupport (RESTテスト、軽量)
    └── RestTestSupport (RESTテスト+DB)
```

### 3.4 リクエスト単体テスト（Web）

組込HTTPサーバ（Jetty）で実際のNablarchハンドラチェーンにリクエストを通してテスト。

**テストクラス構造**:
```java
public class MyActionRequestTest extends BasicHttpRequestTestTemplate {
    @Override
    protected String getBaseUri() {
        return "/action/myAction/";
    }

    @Test
    public void testNormalCase() {
        execute();  // メソッド名をシート名として使用
    }
}
```

**Excelシート構造**:
- `LIST_MAP=testShots`: テストケース一覧（no, description, expectedStatusCode, requestUri等）
- `LIST_MAP=requestParams`: テストケースごとのHTTPリクエストパラメータ
- `LIST_MAP=responseResult`: 期待レスポンス値
- `SETUP_TABLE` / `EXPECTED_TABLE`: ケースごとのDB操作

**主要機能**:
- トークン処理: `isValidToken`フラグでCSRFトークン自動設定
- Cookie対応: テストケースごとのCookie設定
- ファイルアップロード: `FileSupport`クラス
- HTML検証: `Html4HtmlChecker` / `Html5HtmlChecker`
- レスポンスダンプ: レスポンスボディをファイルに保存

**ソース**: https://github.com/nablarch/nablarch-testing/blob/main/src/main/java/nablarch/test/core/http/AbstractHttpRequestTestTemplate.java

### 3.5 DB テスト

`DbAccessTestSupport`が提供する機能：

- **トランザクション管理**: `@Before`でトランザクション開始、`@After`で終了
- **データセットアップ**: `setUpDb(sheetName)` — Excelの `SETUP_TABLE` ブロックを読み込み、既存データ削除（子テーブルから）→新データ挿入（親テーブルから）。外部キー依存関係を自動分析（`TableDataSorter`）
- **テーブルアサーション**: `assertTableEquals(sheetName)` — `EXPECTED_TABLE` / `EXPECTED_COMPLETE_TABLE` と実DBデータを行単位比較
- **SQL結果比較**: `assertSqlResultSetEquals()` — `SqlResultSet` と `LIST_MAP` 期待値を比較

**TableDataクラス機能**:
- 型対応バインディング（バイナリは16進文字列、数値はBigDecimal、日付はTimestamp）
- 省略カラムのデフォルト値自動補完（`fillDefaultValues()`）
- 主キー順データロード
- CLOB→String変換

### 3.6 バッチテスト

`BatchRequestTestSupport`（`StandaloneTestSupportTemplate`の継承）でコマンドラインバッチ実行をシミュレート。

**テストフロー**（テストショットごと）:
1. DB setup（`setUpDb`シート + ケースごとのグループID）
2. 入力ファイルsetup（`SETUP_FIXED`/`SETUP_VARIABLE`ブロック）
3. テストデータのカラムから`CommandLine`生成
4. `Main.handle(commandLine, executionContext)` 実行（バッチ起動）
5. アサーション: ステータスコード、DB状態、出力ファイル、ログメッセージ

**テストケースExcel必須カラム**:

| カラム | 説明 |
|--------|------|
| `no` | テストショット番号 |
| `description` | テストケース説明 |
| `expectedStatusCode` | 期待終了コード |
| `diConfig` | コンポーネント設定ファイルパス |
| `requestPath` | バッチのリクエストパス |
| `userId` | ThreadContext用ユーザーID |
| `setUpTable` | DB setupのグループID |
| `expectedTable` | DBアサーションのグループID |
| `setUpFile` | 入力ファイルsetupのグループID |
| `expectedFile` | 期待出力ファイルのグループID |
| `expectedLog` | 期待ログメッセージのID |
| `args[0]`, `args[1]`... | コマンドライン引数 |

### 3.7 メッセージングテスト

- `MessagingRequestTestSupport`: 組込ActiveMQ Artemisを使用した同期メッセージリクエスト/レスポンステスト
- `MessagingReceiveTestSupport`: メッセージ受信処理テスト
- `EmbeddedMessagingProvider`: テスト用組込JMSプロバイダ
- `MockMessagingContext` / `MockMessagingClient`: リクエスト単体テスト用モック

### 3.8 JUnit 5サポート（nablarch-testing-junit5）

JUnit 4ベースのテストサポートクラスをラップするJUnit 5 `Extension`実装を提供。

| アノテーション | Extension | ラップ対象 |
|-------------|-----------|-----------|
| `@NablarchTest` | `TestSupportExtension` | `TestSupport` |
| `@DbAccessTest` | `DbAccessTestExtension` | `DbAccessTestSupport` |
| `@EntityTest` | `EntityTestExtension` | `EntityTestSupport` |
| `@BatchRequestTest` | `BatchRequestTestExtension` | `BatchRequestTestSupport` |
| `@BasicHttpRequestTest` | `BasicHttpRequestTestExtension` | `BasicHttpRequestTestTemplate` |
| `@RestTest` | `RestTestExtension` | `RestTestSupport` |
| `@SimpleRestTest` | `SimpleRestTestExtension` | `SimpleRestTestSupport` |
| `@IntegrationTest` | `IntegrationTestExtension` | `IntegrationTestSupport` |
| `@MessagingRequestTest` | `MessagingRequestTestExtension` | `MessagingRequestTestSupport` |

**使用パターン**:
```java
@NablarchTest
class MyTest {
    TestSupport support;  // Extensionにより自動注入

    @Test
    void testSomething() {
        Map<String, String> map = support.getMap("sheetName", "id");
    }
}
```

### 3.9 RESTテストサポート（nablarch-testing-rest）

**SimpleRestTestSupport**（軽量）:
- 組込HTTPサーバ（Jetty）でアプリケーションのハンドラチェーンを起動
- Fluent リクエストビルダー: `get()`, `post()`, `put()`, `delete()`, `patch()`
- `HttpResponse`を返してアサーション

**RestTestSupport**（DB対応）:
- `SimpleRestTestSupport` + DB機能
- `assertTableEquals()`, `getListMap()`, `getParamMap()`

```java
public class MyRestActionTest extends RestTestSupport {
    @Test
    public void testGetUsers() {
        HttpResponse response = sendRequest(get("/api/users"));
        assertStatusCode("GET users", HttpResponse.Status.OK, response);
    }
}
```

**ソース**: https://github.com/nablarch/nablarch-testing-rest

### 3.10 その他コンポーネント

- **FileSupport**: 固定長/可変長ファイルのsetupとアサーション
- **LogVerifier**: テスト実行中の期待ログメッセージ検証
- **HtmlChecker**: HTML出力の構文検証

---

## 4. 公式ドキュメント・静的解析ツール

### 4.1 公式ドキュメントサイト

- **URL**: https://nablarch.github.io/docs/LATEST/doc/en/index.html
- **バージョン**: 6u3

| セクション | 内容 |
|-----------|------|
| Nablarchとは | コンセプト、モジュール一覧、ライセンス |
| アプリケーションフレームワーク | Web, WebService, Batch, Messaging、標準ハンドラ/ライブラリ、ブランクプロジェクト |
| 開発ツール | Java静的解析、テスティングフレームワーク、ユーティリティツール |
| 学習リソース | 実行可能サンプル |
| リファレンス | APIドキュメント、バージョンアップポリシー、移行ガイド |
| 外部コンテンツ | Fintanプラットフォームへのリンク |

### 4.2 Fintan（TIS技術ハブ）

- **URL**: https://fintan.jp/en/
- **Nablarch開発標準**: https://fintan.jp/en/page/1658/
- **Nablarchコンテンツ一覧**: https://fintan.jp/en/page/1954/
- **Nablarchシステム開発ガイド**: https://fintan.jp/en/page/1667/

Fintanが提供するNablarch関連リソース：

| カテゴリ | コンテンツ |
|---------|----------|
| アプリケーションFW | マニュアル、サンプル、トレーニング |
| 開発ガイダンス | システム開発ガイド、SPA+REST APIリファレンス |
| 開発標準 | 要件定義FW、テスト種別・観点カタログ、テスト計画ガイド、開発プロセス標準、アプリ開発標準、設計書フォーマット、スタイルガイド |
| 開発ツール | Collaborage、ブランクプロジェクト、テスティングFW、DBA支援ツール |

### 4.3 Checkstyle設定

#### リポジトリ

- **現行**: [Fintan-contents/coding-standards](https://github.com/Fintan-contents/coding-standards/tree/main/java/staticanalysis/checkstyle)
- **レガシー**: [nablarch-development-standards/nablarch-style-guide](https://github.com/nablarch-development-standards/nablarch-style-guide/tree/master/java/staticanalysis/checkstyle)

#### Error レベルルール（修正必須）

| ルール | 説明 |
|-------|------|
| `ConstantName` | `static final`フィールド: `^[A-Z](_?[A-Z0-9]+)*$` |
| `EqualsAvoidNull` | `equals()`のリテラルは左辺に |
| `HiddenField` | ローカル変数がフィールドをシャドウしない（コンストラクタ/セッタ除外） |
| `HideUtilityClassConstructor` | ユーティリティクラスはprivateコンストラクタ必須 |
| `IllegalCatch` | Exception/Throwable/RuntimeException/Errorのcatch禁止 |
| `IllegalThrows` | Exception/Throwable/RuntimeException/Errorのthrow禁止 |
| `IllegalType` | 実装クラス（ArrayList, HashMap等）を変数/パラメータ/戻り値型に使用禁止 |
| `MissingJavadocType` | 全クラスにJavadoc必須（scope: private） |
| `MissingJavadocMethod` | 全メソッドにJavadoc必須（scope: private） |
| `MethodLength` | メソッドは150行以下 |
| `NeedBraces` | if/forブロックに波括弧必須 |
| `NoFinalizer` | `finalize()`のオーバーライド禁止 |
| `PackageDeclaration` | package宣言必須 |
| `StringLiteralEquality` | 文字列比較は`equals()`を使用（`==`禁止） |
| `FileLength` | ファイルは2000行以下 |
| `LineLength` | 1行最大150文字（import文除外） |

#### 禁止型（IllegalType）

全java.utilコレクション実装クラスを変数型/パラメータ型/戻り値型として禁止。**インタフェースを使用**：

- `ArrayList` → `List`
- `HashMap` → `Map`
- `HashSet` → `Set`
- `LinkedList` → `List`
- `TreeMap` → `NavigableMap` / `SortedMap`
- その他20+の並行/特殊実装

#### 抑制メカニズム

1. **ファイルベース抑制**（`SuppressionFilter`）: `suppressions.xml`で除外ファイルをリスト
2. **コメントベース抑制**（`SuppressWithNearbyCommentFilter`）: `// SUPPRESS CHECKSTYLE #<issue_number>` をラインコメントとして追加

#### Maven統合

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.0.0</version>
  <dependencies>
    <dependency>
      <groupId>com.puppycrawl.tools</groupId>
      <artifactId>checkstyle</artifactId>
      <version>8.11</version>
    </dependency>
  </dependencies>
  <configuration>
    <configLocation>${basedir}/checkstyle/nablarch-checkstyle.xml</configLocation>
    <propertyExpansion>config_loc=${basedir}/checkstyle/</propertyExpansion>
  </configuration>
</plugin>
```

実行: `mvn checkstyle:check`

**ソース**: https://github.com/nablarch-development-standards/nablarch-style-guide/blob/master/en/java/staticanalysis/checkstyle/checkstyle-example/checkstyle/nablarch-checkstyle.xml

### 4.4 SpotBugs設定

#### 運用原則

1. **全SpotBugs警告を解消** — ゼロ警告ポリシー
2. 警告解消のためにコード品質を落としてはならない — アーキテクト相談
3. 自動生成コードは除外可能、ただし除外は最小範囲に

#### 除外フィルタ設定

```xml
<FindBugsFilter>
    <!-- Formクラスの未使用フィールドを許容 -->
    <Match>
        <Bug pattern="UUF_UNUSED_FIELD" />
        <Or><Class name="please.change.me.form.FormClassName"/></Or>
    </Match>
    <!-- 配列露出はチェックしない（商用アプリでは通常問題なし） -->
    <Match><Bug pattern="EI_EXPOSE_REP" /></Match>
    <Match><Bug pattern="EI_EXPOSE_REP2" /></Match>
    <!-- 特定の未公開API使用除外 -->
    <Match>
        <Bug pattern="UPU_UNPUBLISHED_API_USAGE"/>
        <Class name="please.change.me.ClassName"/>
        <Method name="bar" params="java.lang.String, int"/>
    </Match>
</FindBugsFilter>
```

#### Maven統合

```xml
<plugin>
  <groupId>com.github.spotbugs</groupId>
  <artifactId>spotbugs-maven-plugin</artifactId>
  <version>4.5.0.0</version>
  <configuration>
    <excludeFilterFile>spotbugs/spotbugs_exclude_for_production.xml</excludeFilterFile>
    <maxHeap>1024</maxHeap>
    <jvmArgs>-Dnablarch-findbugs-config=spotbugs/published-config/production</jvmArgs>
    <plugins>
      <plugin>
        <groupId>com.nablarch.framework</groupId>
        <artifactId>nablarch-unpublished-api-checker</artifactId>
        <version>1.0.0</version>
      </plugin>
    </plugins>
  </configuration>
</plugin>
```

#### Find Security Bugs統合

```xml
<plugin>
  <groupId>com.h3xstream.findsecbugs</groupId>
  <artifactId>findsecbugs-plugin</artifactId>
  <version>1.11.0</version>
</plugin>
```

### 4.5 未公開APIチェッカー（Unpublished API Checker）

#### リポジトリ

- **SpotBugs版**: [nablarch/nablarch-unpublished-api-checker](https://github.com/nablarch/nablarch-unpublished-api-checker)（v1.0.1）
- **FindBugs版（レガシー）**: [nablarch/nablarch-unpublished-api-checker-findbugs](https://github.com/nablarch/nablarch-unpublished-api-checker-findbugs)

#### アーキテクチャ

**ホワイトリスト方式**のSpotBugsプラグイン。許可されていないAPI使用を検出。

**検出ルール**:
- 未公開クラスの参照（インスタンス化、クラスメソッド呼び出し）
- 未公開メソッドの呼び出し
- 未公開例外のcatch/throw

**バグコード**: `UPU` / **バグタイプ**: `UPU_UNPUBLISHED_API_USAGE`

#### 設定ファイル

| ファイル | 用途 |
|---------|------|
| `JavaOpenApi.config` | Nablarchが許可するJava標準ライブラリAPI |
| `JakartaEEOpenApi.config` | Jakarta EE標準API |
| `NablarchApiForProgrammer.config` | プログラマ向けNablarch API |
| `NablarchTestingApiForProgrammer.config` | プログラマ向けテスティングAPI |
| `NablarchApiForArchitect.config` | アーキテクト向けNablarch API |
| `NablarchTestingApiForArchitect.config` | アーキテクト向けテスティングAPI |

#### 設定形式

```
# パッケージレベル（サブパッケージ含む）
java.lang
nablarch.fw.web

# クラス/インタフェースレベル
nablarch.common.code.CodeUtil

# コンストラクタ/メソッドレベル
java.lang.Boolean.Boolean(boolean)
java.lang.String.indexOf(int)
```

#### 許可Java標準API（主要）

- `java.lang`: Boolean, Byte, Character, Double, Enum, Float, Integer, Long, Math, Number, Object, Short, String, StringBuilder
- `java.math`: BigDecimal, BigInteger
- `java.sql`: Blob, Clob, Date, Time, Timestamp
- `java.util`: 全コレクションFW（Collection, List, Set, Map等）、Arrays, Collections, Objects, Optional, UUID
- `java.util.function`: 全パッケージ
- `java.util.stream`: 全パッケージ

**禁止（ホワイトリストに未記載）**:
- `java.lang.StringBuffer`（→ StringBuilder使用）
- `java.util.Vector`（→ ArrayList使用）
- `java.util.Hashtable`（→ HashMap使用）
- `java.util.Stack`（→ Deque使用）
- `java.util.Dictionary`（→ Map使用）
- `java.util.Enumeration`（→ Iterator使用）
- `java.util.StringTokenizer`（→ String.split使用）

#### 継承チェック動作

スーパークラス/インタフェース参照でメソッドを呼び出した場合、実行時型ではなく**宣言型**レベルでAPI検証：

```java
List<String> list = new ArrayList<>();
list.add(test); // List.addをチェック（ArrayList.addではない）
```

### 4.6 ArchUnit（アーキテクチャテスト）

アーキテクチャルールをテストコードで強制：

```java
// 宣言チェック
ArchRuleDefinition.fields().that().haveRawType(DaoContext.class)
    .should().bePrivate().andShould().beFinal().andShould().notBeStatic();

// 依存チェック
ArchRuleDefinition.noClasses().that().resideInAPackage("..service..")
    .should().dependOnClassesThat().resideInAPackage("..form..");

// レイヤーチェック
Architectures.layeredArchitecture()
    .layer("Action").definedBy("..action..")
    .layer("Service").definedBy("..service..")
    .layer("Form").definedBy("..form..")
    .whereLayer("Action").mayNotBeAccessedByAnyLayer()
    .whereLayer("Service").mayOnlyBeAccessedByLayers("Action")
    .whereLayer("Form").mayOnlyBeAccessedByLayers("Action");
```

**除外メカニズム**: `archunit_ignore_patterns.txt`（`src/test/resources`配下）にregexパターンを記述

**ソース**: https://github.com/nablarch-development-standards/nablarch-style-guide/blob/master/en/java/staticanalysis/archunit/docs/ArchUnit-commentary.md

---

## 5. サンプルプロジェクト構成パターン

### 5.1 分析対象

| プロジェクト | パッケージング | ブランチ |
|------------|-------------|---------|
| [nablarch-example-web](https://github.com/nablarch/nablarch-example-web) | war | main |
| [nablarch-example-batch](https://github.com/nablarch/nablarch-example-batch) | jar | main |
| [nablarch-example-rest](https://github.com/nablarch/nablarch-example-rest) | war | main |

全プロジェクト: **Nablarch 6u3**, **Java 17**, **Jakarta EE BOM 10.0.0**

### 5.2 標準Mavenディレクトリレイアウト

#### Web プロジェクト (war)

```
nablarch-example-web/
├── pom.xml
├── src/main/java/com/nablarch/example/app/
│   ├── entity/core/validation/validator/    # ドメインバリデーション
│   └── web/
│       ├── action/                          # Actionクラス（コントローラ）
│       ├── common/                          # 共通ユーティリティ
│       │   ├── authentication/              # 認証ロジック
│       │   ├── code/                        # コード列挙定義
│       │   └── file/                        # ファイル処理
│       ├── dto/                             # DTO
│       ├── form/                            # Formビーン（入力バリデーション）
│       └── handler/                         # カスタムハンドラ
├── src/main/resources/
│   ├── web-boot.xml                         # 起動設定
│   ├── web-component-configuration.xml      # メインコンポーネント定義
│   ├── routes.xml                           # URLルーティング
│   ├── db.xml                               # DataSource設定
│   ├── common.properties / env.properties   # プロパティ
│   └── com/nablarch/example/app/entity/*.sql # SQLファイル
├── src/main/webapp/
│   ├── WEB-INF/web.xml                      # サーブレット設定
│   └── WEB-INF/view/                        # JSPビュー
└── src/test/
    ├── java/.../action/                     # テスト + .xlsx
    └── resources/data/*.csv                 # テストデータ
```

#### Batch プロジェクト (jar)

```
nablarch-example-batch/
├── pom.xml
├── distribution.xml                         # アセンブリ記述子
├── src/main/
│   ├── format/                              # データフォーマット定義
│   ├── java/com/nablarch/example/app/batch/
│   │   ├── action/                          # バッチActionクラス
│   │   ├── form/                            # Formビーン
│   │   ├── interceptor/                     # カスタムインターセプタ
│   │   └── reader/                          # カスタムデータリーダー
│   └── resources/
│       ├── batch-component-configuration.xml # メインバッチ設定
│       ├── import-zip-code-file.xml          # ジョブ個別設定
│       └── batch-config/*.properties         # ジョブ個別プロパティ
└── work/                                    # 作業ディレクトリ
```

#### REST プロジェクト (war)

```
nablarch-example-rest/
├── pom.xml
├── src/main/java/com/nablarch/example/
│   ├── action/                              # RESTアクションクラス
│   ├── domain/                              # ドメイン定義
│   ├── dto/                                 # レスポンス/検索DTO
│   ├── form/                                # リクエストFormビーン
│   └── jackson/                             # JSONシリアライザ
├── src/main/resources/
│   ├── rest-boot.xml                        # 起動設定
│   ├── rest-component-configuration.xml     # メインコンポーネント設定
│   └── db.xml                               # DataSource設定
└── src/main/webapp/WEB-INF/web.xml          # 最小サーブレット設定
```

### 5.3 共通依存関係（目的別）

#### BOM / 依存管理（全プロジェクト共通）

| 依存 | バージョン | 用途 |
|------|----------|------|
| `com.nablarch.profile:nablarch-bom` | 6u3 | Nablarchバージョン管理 |
| `org.junit:junit-bom` | 5.11.0 | JUnit 5 BOM |
| `jakarta.platform:jakarta.jakartaee-bom` | 10.0.0 | Jakarta EE BOM |

#### コアNablarch依存

| 依存 | Web | Batch | REST | 用途 |
|------|-----|-------|------|------|
| `nablarch-main-default-configuration` | Y | Y | Y | デフォルト設定 |
| `nablarch-web`（プロファイル） | Y | - | - | Webプロファイル |
| `nablarch-batch`（プロファイル） | - | Y | - | バッチプロファイル |
| `nablarch-fw-jaxrs` | Y | - | Y | JAX-RSフレームワーク |
| `nablarch-core-validation-ee` | - | - | Y | Bean Validation |
| `nablarch-common-dao` | - | - | Y | Universal DAO |

#### テスト依存

| 依存 | Web | Batch | REST |
|------|-----|-------|------|
| `nablarch-testing-default-configuration` | Y | Y | Y |
| `nablarch-testing` | - | Y | - |
| `nablarch-testing-rest` | Y | - | Y |
| `nablarch-testing-jetty12` | Y | - | Y |
| `nablarch-testing-junit5` | Y | Y | Y |

### 5.4 設定ファイルパターン

#### Boot設定XML（エントリポイント）

| プロジェクト | ファイル | インポート先 |
|------------|---------|-------------|
| Web | `web-boot.xml` | `web-component-configuration.xml` |
| Batch | N/A（`nablarch.fw.launcher.Main`で起動） | `batch-component-configuration.xml` |
| REST | `rest-boot.xml` | `rest-component-configuration.xml` |

#### プロパティファイル（全プロジェクト共通パターン）

| ファイル | 用途 |
|---------|------|
| `common.properties` | アプリ全体設定（basePackage, timeout, fetch size等） |
| `env.properties` | 環境固有設定（DB接続、ファイルパス） |
| `app-log.properties` | アプリケーションログ設定 |
| `log.properties` | ログフレームワーク設定 |
| `messages.properties` | ユーザー向けメッセージ |

#### SQLファイル規約

SQLファイルは使用するJavaクラスの**隣に配置**（リソースディレクトリでパッケージ構造をミラー）：

```
src/main/java/com/nablarch/example/app/entity/Project.java
src/main/resources/com/nablarch/example/app/entity/Project.sql
```

### 5.5 ハンドラキュー構成

#### Web ハンドラキュー

```
webFrontController (for /action/* and /)
├── HttpCharacterEncodingHandler
├── ThreadContextClearHandler
├── GlobalErrorHandler
├── HttpResponseHandler
├── SecureHandler (CSP, X-Frame-Options等)
├── MultipartHandler
├── SessionStoreHandler
├── ThreadContextHandler
├── HttpAccessLogHandler
├── NormalizationHandler
├── ForwardingHandler
├── HttpErrorHandler (エラーページマッピング)
├── NablarchTagHandler
├── CsrfTokenVerificationHandler
├── DbConnectionManagementHandler
├── TransactionManagementHandler
├── LoginUserPrincipalCheckHandler (カスタム)
└── PackageMapping (Actionクラスへディスパッチ)

jaxrsController (for /api/*)
├── HttpCharacterEncodingHandler
├── ThreadContextClearHandler
├── GlobalErrorHandler
├── JaxRsResponseHandler
├── ThreadContextHandler
├── JaxRsAccessLogHandler
├── DbConnectionManagementHandler
├── TransactionManagementHandler
└── JaxRsPackageMapping
```

**重要**: Webプロジェクトは**2つのハンドラキュー**を定義 — ページリクエスト用とAPIエンドポイント用

#### Batch ハンドラキュー

```
├── StatusCodeConvertHandler
├── ThreadContextClearHandler
├── GlobalErrorHandler
├── ThreadContextHandler
├── DbConnectionManagementHandler (init/cleanup)
├── TransactionManagementHandler (init/cleanup)
├── RequestPathJavaPackageMapping (ディスパッチ)
├── MultiThreadExecutionHandler
├── DbConnectionManagementHandler (業務)
├── LoopHandler (トランザクションループ)
└── DataReadHandler
```

#### REST ハンドラキュー

```
├── HttpCharacterEncodingHandler
├── ThreadContextClearHandler
├── GlobalErrorHandler
├── JaxRsResponseHandler (SecureHandler付き)
├── MultipartHandler
├── ThreadContextHandler
├── JaxRsAccessLogHandler
├── DbConnectionManagementHandler
├── TransactionManagementHandler
└── PathOptionsProviderRoutesMapping (アノテーションベースルーティング)
```

### 5.6 プロジェクト種別間の主要差異

| 観点 | Web | Batch | REST |
|------|-----|-------|------|
| パッケージング | `war` | `jar` | `war` |
| エントリポイント | サーブレットフィルタ | `nablarch.fw.launcher.Main` | サーブレットフィルタ |
| ルーティング | `routes.xml`（XMLベース） | 規約ベース | アノテーションベース |
| ビュー層 | JSP | N/A | JSON（Jackson） |
| セッション管理 | DBセッションストア + CSRF | N/A | ステートレス |
| テストデータ形式 | `.xlsx`/`.xls`/`.csv` | `.xls`/`.csv` | `.xlsx`/`.xls`/`.csv`/`.json` |

### 5.7 Archetype（プロジェクトテンプレート）

- **リポジトリ**: [nablarch/nablarch-single-module-archetype](https://github.com/nablarch/nablarch-single-module-archetype)

| モジュール | 説明 |
|-----------|------|
| `nablarch-web` | Webアプリケーション |
| `nablarch-jaxrs` | RESTfulサービス |
| `nablarch-batch` | Nablarchバッチ |
| `nablarch-batch-ee` | Jakarta Batch (JSR-352) |
| `nablarch-batch-dbless` | DB接続なしバッチ |
| `nablarch-container-web` | Dockerコンテナ Web |
| `nablarch-container-jaxrs` | Dockerコンテナ REST |
| `nablarch-container-batch` | Dockerコンテナ バッチ |

**生成コマンド例**:
```bash
mvn archetype:generate -DarchetypeGroupId=com.nablarch.archetype \
  -DarchetypeArtifactId=nablarch-web-archetype -DarchetypeVersion=6u3
```

### 5.8 抽出パターンまとめ

1. **シングルモジュールMavenプロジェクト** — マルチモジュールは不使用
2. **XMLベースDIコンテナ** — Nablarch独自の`*-component-configuration.xml`で設定
3. **ハンドラキューアーキテクチャ** — 全リクエスト処理をXMLで定義されたハンドラチェーンで処理
4. **規約ベースSQL** — SQLファイルを対応Javaクラスの隣に配置
5. **Entity自動生成** — ERDモデルから`gsp-dba-maven-plugin`でビルド時に生成
6. **プロパティベース外部化** — `common.properties`（共通）+ `env.properties`（環境固有）
7. **Boot XML間接参照** — 薄いboot XMLがメインコンポーネント設定をインポート（テストオーバーライド対応）
8. **ExcelベーステストData** — `.xls`/`.xlsx`ファイルをテストクラスの隣に配置
9. **ジョブ個別設定** — バッチジョブごとに独自XMLと独自プロパティファイル
10. **Webデュアルハンドラキュー** — ページリクエストとAPIリクエストで別々のハンドラキュー

---

## ソースURL一覧

### 開発標準
- https://github.com/nablarch-development-standards/nablarch-development-standards
- https://github.com/nablarch-development-standards/nablarch-development-standards-tools
- https://github.com/nablarch-development-standards/nablarch-style-guide

### スタイルガイド（現行）
- https://github.com/Fintan-contents/coding-standards

### テスティングフレームワーク
- https://github.com/nablarch/nablarch-testing
- https://github.com/nablarch/nablarch-testing-junit5
- https://github.com/nablarch/nablarch-testing-rest

### 静的解析ツール
- https://github.com/nablarch/nablarch-unpublished-api-checker
- https://github.com/nablarch/nablarch-unpublished-api-checker-findbugs

### 公式ドキュメント
- https://nablarch.github.io/docs/LATEST/doc/en/index.html
- https://nablarch.github.io/docs/LATEST/doc/en/external_contents/index.html

### Fintan
- https://fintan.jp/en/page/1658/ （開発標準）
- https://fintan.jp/en/page/1667/ （システム開発ガイド）
- https://fintan.jp/en/page/1954/ （コンテンツ一覧）
- https://github.com/Fintan-contents/nablarch-system-development-guide

### サンプルプロジェクト
- https://github.com/nablarch/nablarch-example-web
- https://github.com/nablarch/nablarch-example-batch
- https://github.com/nablarch/nablarch-example-rest
- https://github.com/nablarch/nablarch-single-module-archetype
