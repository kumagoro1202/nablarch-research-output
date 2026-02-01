# Nablarchバッチエンジニア育成 学習計画

> **作成日**: 2026-02-02
> **タスクID**: subtask_033 (cmd_015)
> **プロジェクト**: nablarch_strategy
> **作成者**: ashigaru1
> **参照KB**: O-005（公式ドキュメント）、O-006（アーキテクチャ詳細）、O-007（開発標準・テスティングFW）

---

## 目次

1. [スキルロードマップ](#1-スキルロードマップ)
2. [学習ステップ](#2-学習ステップ)
3. [学習コンテンツ・教材](#3-学習コンテンツ教材)
4. [解説書・リファレンス](#4-解説書リファレンス)
5. [学習計画テンプレート](#5-学習計画テンプレート)

---

## 1. スキルロードマップ

### 1.1 スキルロードマップ ビジュアル図

```
                    Nablarchバッチエンジニア スキルロードマップ

  ┌─────────────────────────────────────────────────────────────────────┐
  │                                                                     │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌────────────────┐   │
  │  │  初級    │──▶│  中級    │──▶│  上級    │──▶│ エキスパート   │   │
  │  │ Beginner │   │ Inter-   │   │ Advanced │   │    Expert      │   │
  │  │          │   │ mediate  │   │          │   │                │   │
  │  └──────────┘   └──────────┘   └──────────┘   └────────────────┘   │
  │       │              │              │                │              │
  │       ▼              ▼              ▼                ▼              │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌────────────────┐  │
  │  │Java基礎  │   │ハンドラ  │   │マルチ    │   │アーキテクチャ  │  │
  │  │Maven基礎 │   │キュー理解│   │スレッド  │   │設計・レビュー  │  │
  │  │XML設定   │   │DB/TX管理 │   │常駐バッチ│   │性能チューニング│  │
  │  │Hello     │   │DataReader│   │エラー設計│   │FW拡張・カスタム│  │
  │  │  Batch   │   │テスト基礎│   │テスト応用│   │チーム指導      │  │
  │  └──────────┘   └──────────┘   └──────────┘   └────────────────┘  │
  │                                                                     │
  │  目安期間:                                                          │
  │  ├── 1-4週 ──┤├── 5-12週 ──┤├── 13-20週 ─┤├── 21週以降 ────┤    │
  │                                                                     │
  └─────────────────────────────────────────────────────────────────────┘
```

### 1.2 レベル別スキルマップ

#### 初級（Beginner）— Java基礎 + Nablarch入門

| # | スキル | 詳細 | 達成基準 |
|---|--------|------|----------|
| 1 | Java SE基礎 | Java 17構文、コレクション、ストリームAPI、例外処理 | FizzBuzzから簡単なCRUDアプリまで書ける |
| 2 | Maven基礎 | pom.xml、依存管理、ビルドライフサイクル、プロファイル | `mvn package -P prod` の意味を説明できる |
| 3 | JDBC基礎 | Connection, PreparedStatement, ResultSet | 素のJDBCでSELECT/INSERT/UPDATE/DELETEを書ける |
| 4 | Nablarchコンセプト | ハンドラキューアーキテクチャの概念理解 | パイプライン処理モデルを図で説明できる |
| 5 | XMLコンポーネント定義 | `<component>`, `<property>`, `<list>`, `<import>` | component-configuration.xmlの基本構造を読める |
| 6 | システムリポジトリ | `SystemRepository.get()`, DI概念 | DIコンテナの役割を説明できる |
| 7 | Hello Batch | nablarch-example-batchの実行 | サンプルバッチを手元で動かせる |
| 8 | バッチ起動方法 | `nablarch.fw.launcher.Main`, `-requestPath` | コマンドラインからバッチを起動できる |

#### 中級（Intermediate）— バッチ開発の実務スキル

| # | スキル | 詳細 | 達成基準 |
|---|--------|------|----------|
| 1 | ハンドラキュー構成 | 都度起動バッチの9ハンドラ構成、各ハンドラの役割 | ハンドラキューの順序と理由を説明できる |
| 2 | BatchAction実装 | `BatchAction<D>`の`handle()`と`createReader()` | 独力でバッチアクションクラスを実装できる |
| 3 | DataReader実装 | `DatabaseRecordReader`, `FileDataReader`, カスタムReader | 要件に応じて適切なDataReaderを選択・実装できる |
| 4 | DB接続・TX管理 | `DbConnectionManagementHandler` + `TransactionManagementHandler` | メインスレッド/サブスレッドのDB接続の違いを説明できる |
| 5 | ユニバーサルDAO | `UniversalDao.findAll()`, `findAllBySqlFile()`, バッチ操作 | SQLファイルを使ったDB操作を実装できる |
| 6 | SQLファイル | `$if`, `:param[]`, `$sort` の動的SQL構文 | 動的条件付きSQLを正しく書ける |
| 7 | ファイルI/O | データバインド（CSV, 固定長）、`@Csv`, `@CsvFormat` | CSV/固定長ファイルの読み書きをデータバインドで実装できる |
| 8 | Bean Validation | `@Domain`, `@Required`, カスタムバリデータ | バッチ入力データのバリデーションを実装できる |
| 9 | LoopHandler | コミット間隔設定、トランザクションループ制御 | 適切なコミット間隔を設計・設定できる |
| 10 | テスト基礎 | `BatchRequestTestSupport`, Excelテストデータ | Excelベースのバッチリクエスト単体テストを書ける |
| 11 | ログ出力 | 障害通知ログ、パフォーマンスログの設定 | バッチ用ログ出力を設定・カスタマイズできる |
| 12 | 設計書 | 機能設計書（バッチ用テンプレート）の読み書き | 開発標準に従った設計書を作成できる |

#### 上級（Advanced）— 複雑なバッチシステムの設計・実装

| # | スキル | 詳細 | 達成基準 |
|---|--------|------|----------|
| 1 | マルチスレッドバッチ | `MultiThreadExecutionHandler`, スレッド数設定、排他制御 | マルチスレッドバッチの設計・実装ができる |
| 2 | 常駐バッチ | `ProcessResidentHandler`, `BasicProcessStopHandler`, `RetryHandler` | 常駐バッチの設計・運用を含めて対応できる |
| 3 | 複数DB接続 | 複数`DbConnectionManagementHandler`配置、`dbConnectionName` | 複数DBを使うバッチを設計・実装できる |
| 4 | 個別トランザクション | `SimpleDbTransactionManager`, `UniversalDao.Transaction` | 業務失敗時もログ記録するパターンを実装できる |
| 5 | エラーハンドリング設計 | `ProcessAbnormalEnd`, リトライ戦略、再開可能設計 | バッチ異常時の復旧設計ができる |
| 6 | Jakarta Batch (JSR352) | Batchlet/Chunk、リスナー、jBeret | JSR352準拠バッチの設計・実装ができる |
| 7 | テスト応用 | 常駐バッチテスト（`OneShotLoopHandler`差替え）、ファイルアサーション | 複雑なバッチのテスト戦略を設計できる |
| 8 | JUnit 5拡張 | `@BatchRequestTest`, `BatchRequestTestExtension` | JUnit 5ベースのテストに移行できる |
| 9 | 排他制御 | 楽観ロック（`@Version`）、悲観ロック（`SELECT FOR UPDATE`） | 適切な排他制御方式を選択・実装できる |
| 10 | 性能設計 | フェッチサイズ、コミット間隔、スレッド数の最適化 | 性能要件に基づいたバッチ設計ができる |

#### エキスパート（Expert）— アーキテクチャ設計・チーム指導

| # | スキル | 詳細 | 達成基準 |
|---|--------|------|----------|
| 1 | カスタムハンドラ設計 | `Handler<TData, TResult>`実装、ハンドラキューへの配置設計 | プロジェクト固有のカスタムハンドラを設計・実装できる |
| 2 | カスタムインターセプタ | `@Interceptor`メタアノテーション、`Interceptor.Impl`継承 | カスタムインターセプタの設計・実装ができる |
| 3 | FW拡張 | カスタムDataReader、カスタムDialect、カスタムBodyConverter | フレームワークの拡張ポイントを活用した開発ができる |
| 4 | 性能チューニング | 遅延ロード、ページネーション、DB接続プール調整 | 大規模バッチの性能問題を特定・解決できる |
| 5 | 運用設計 | ジョブスケジューラ連携、監視設計、リカバリ設計 | バッチシステム全体の運用設計ができる |
| 6 | コードレビュー | ハンドラキュー構成レビュー、TX境界レビュー、テストレビュー | チームの成果物を品質面でレビューできる |
| 7 | テスト戦略 | テスト計画、テストデータ設計、自動化戦略 | プロジェクト全体のテスト戦略を策定できる |
| 8 | 技術選定 | Nablarchバッチ vs Jakarta Batch vs Spring Batch の選定根拠 | 要件に応じた適切な技術選定と根拠説明ができる |
| 9 | v5→v6移行 | javax→jakarta名前空間変更、依存更新 | 既存プロジェクトのNablarch 6移行をリードできる |

### 1.3 前提スキル（Java/Jakarta EE基礎）

| # | スキル | 必要レベル | Nablarchバッチとの関連 |
|---|--------|----------|---------------------|
| 1 | Java SE 17 | 中級 | Nablarch 6u3はJava 17/21対応 |
| 2 | JDBC API | 基礎 | DBアクセスの基盤、ユニバーサルDAOの内部 |
| 3 | Jakarta Persistence (JPA) アノテーション | 基礎 | `@Entity`, `@Table`, `@Column`, `@Id` をユニバーサルDAOで使用 |
| 4 | Jakarta Bean Validation | 基礎 | `@NotNull`, `@Size`, `@Domain` によるバリデーション |
| 5 | Maven | 中級 | ビルド、依存管理、プロファイル、アーキタイプ |
| 6 | SQL | 中級 | SQLファイル記述、動的SQL構文 |
| 7 | XML | 基礎 | component-configuration.xml の読み書き |
| 8 | JUnit 4/5 | 基礎 | テスティングFWの基盤 |

---

## 2. 学習ステップ

### 2.1 学習ステップ フロー図

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                   学習ステップ フロー図                           │
  └───────────────────────────────────────────────────────────────────┘

  ┌─────────────────────┐
  │ Step 0: 環境構築    │
  │ Java17, Maven, IDE  │
  └────────┬────────────┘
           │
           ▼
  ┌─────────────────────┐
  │ Step 1: Java基礎    │
  │ 復習（1-2週）       │
  └────────┬────────────┘
           │
           ▼
  ┌─────────────────────┐     ┌──────────────────────────┐
  │ Step 2: Nablarch    │────▶│ Step 3: サンプル実行     │
  │ コンセプト理解      │     │ nablarch-example-batch   │
  │ （1週）             │     │ （1週）                  │
  └─────────────────────┘     └────────┬─────────────────┘
                                       │
                                       ▼
                              ┌──────────────────────────┐
                              │ Step 4: 都度起動バッチ   │
                              │ 実装演習（2-3週）        │
                              └────────┬─────────────────┘
                                       │
                          ┌────────────┼────────────┐
                          ▼            ▼            ▼
                  ┌──────────┐ ┌──────────┐ ┌──────────┐
                  │ Step 5a  │ │ Step 5b  │ │ Step 5c  │
                  │ DB/TX    │ │ ファイル │ │ テスト   │
                  │ 深掘り   │ │ I/O演習  │ │ 実装演習 │
                  │（1-2週） │ │（1-2週） │ │（1-2週） │
                  └────┬─────┘ └────┬─────┘ └────┬─────┘
                       └────────────┼────────────┘
                                    ▼
                          ┌──────────────────────────┐
                          │ Step 6: マルチスレッド    │
                          │ ・常駐バッチ（2-3週）    │
                          └────────┬─────────────────┘
                                   │
                                   ▼
                          ┌──────────────────────────┐
                          │ Step 7: 総合演習         │
                          │ 実務相当のバッチ開発     │
                          │ （2-4週）                │
                          └────────┬─────────────────┘
                                   │
                                   ▼
                          ┌──────────────────────────┐
                          │ Step 8: 上級・応用       │
                          │ エラー設計, 性能, 運用   │
                          │ （継続）                 │
                          └──────────────────────────┘
```

### 2.2 各ステップの詳細

#### Step 0: 環境構築（0.5週）

**目標**: 開発環境を整え、Nablarchプロジェクトをビルドできる状態にする

| # | タスク | 具体的内容 | 達成基準 |
|---|--------|----------|----------|
| 1 | JDK 17インストール | OpenJDK 17 or Temurin 17 | `java -version` で17を確認 |
| 2 | Mavenインストール | Maven 3.9.9以上 | `mvn -version` で確認 |
| 3 | IDEセットアップ | IntelliJ IDEA or Eclipse + Nablarchフォーマッタ | プロジェクトをインポートしてビルド成功 |
| 4 | H2データベース確認 | nablarch-example-batchに同梱のH2 | サンプルのDB初期化が実行できる |
| 5 | Git基礎 | clone, branch, commit | nablarch-example-batchをcloneできる |

#### Step 1: Java基礎復習（1-2週）

**目標**: Nablarchバッチ開発に必要なJava SE 17の基礎を確認する

| # | タスク | 学習内容 | 達成基準 |
|---|--------|---------|----------|
| 1 | コレクション操作 | `List`, `Map`, `Set`, `Stream API` | ストリームAPIでfilter/map/collectが書ける |
| 2 | 例外処理 | try-catch-finally, カスタム例外, 非チェック例外 | Nablarchの「非チェック例外のみ」方針を理解 |
| 3 | アノテーション | `@Override`, `@Entity`, `@Column` 等の理解 | JPA/Bean Validationアノテーションが読める |
| 4 | ジェネリクス | `Handler<TData, TResult>` のような型パラメータ | ジェネリクスインターフェースの実装が書ける |
| 5 | JDBC基礎 | Connection, PreparedStatement, ResultSet | 素のJDBCでCRUD操作が書ける |

#### Step 2: Nablarchコンセプト理解（1週）

**目標**: ハンドラキューアーキテクチャを含むNablarchの設計思想を理解する

| # | タスク | 学習リソース | 達成基準 |
|---|--------|------------|----------|
| 1 | Nablarch概要を読む | 公式ドキュメント: アーキテクチャ概要ページ | 3原則（堅牢性、テスト容易性、即戦力）を説明できる |
| 2 | ハンドラキューを理解する | O-006 セクション1「ハンドラキューの設計パターン」 | パイプライン処理モデル（往路/復路）を図で説明できる |
| 3 | XML設定を読む | O-006 セクション4「コンポーネント定義XMLの構造」 | `<component>`, `<property>`, `<list>`, `<import>` の役割を説明できる |
| 4 | バッチアプリ種別を理解する | 公式ドキュメント: バッチアーキテクチャページ | 都度起動バッチと常駐バッチの違いを説明できる |

#### Step 3: サンプルプロジェクト実行（1週）

**目標**: nablarch-example-batchを動かして、バッチの実行フローを体感する

| # | タスク | 具体的手順 | 達成基準 |
|---|--------|----------|----------|
| 1 | サンプルclone | `git clone https://github.com/nablarch/nablarch-example-batch.git` | ビルド成功 |
| 2 | ImportZipCodeFileAction実行 | Getting Startedに従いCSVインポートバッチを実行 | ZIP_CODE_DATAテーブルにデータが登録される |
| 3 | FileDeleteAction実行 | PDFファイル削除バッチの実行 | 指定フォルダのPDFが削除される |
| 4 | RegistrationPdfFileAction実行 | 常駐バッチの起動と停止 | 常駐バッチの動作を観察できる |
| 5 | XML設定を読む | `batch-component-configuration.xml` のハンドラキューを追跡 | 各ハンドラの役割をコメントで書き込める |
| 6 | デバッグ実行 | ハンドラのhandle()にブレークポイント設置 | ハンドラが順序通りに実行されることを確認 |

#### Step 4: 都度起動バッチ実装演習（2-3週）

**目標**: 独力でDB読込→処理→DB書込みの都度起動バッチを実装できる

| # | 演習課題 | 内容 | 達成基準 |
|---|---------|------|----------|
| 1 | DB→DBバッチ | `DatabaseRecordReader`で読込、加工してINSERT | アクションクラスとDataReaderの実装が完成 |
| 2 | CSV→DBバッチ | `FileDataReader` + データバインドでCSV読込→DB登録 | `@Csv`, `@CsvFormat`を使ったForm定義ができる |
| 3 | DB→CSVバッチ | DBから読込→CSVファイル出力 | ファイル出力パターンの実装ができる |
| 4 | バリデーション付き | 入力データのBean Validation→エラーログ出力→正常データのみ処理 | `@Domain`バリデーションを設定できる |
| 5 | SQLファイル作成 | 動的条件付きSQLの作成（`$if`, `:param[]`） | 動的SQLの構文を正しく使える |

#### Step 5a: DB/トランザクション深掘り（1-2週）

**目標**: バッチにおけるDB接続とトランザクション管理を深く理解する

| # | タスク | 学習内容 | 達成基準 |
|---|--------|---------|----------|
| 1 | メイン/サブスレッドのDB接続 | ハンドラキューでDB接続が2回出現する理由 | メインスレッド用とサブスレッド用の違いを説明できる |
| 2 | コミット間隔 | `LoopHandler`のtransactionCommitInterval設定 | 適切なコミット間隔を設計できる |
| 3 | ユニバーサルDAO深掘り | バッチ操作(`batchInsert`等)、遅延ロード、ページネーション | 大量データ処理でのDAO使い方を実装できる |
| 4 | 個別トランザクション | `SimpleDbTransactionManager`の使い方 | ログ記録用の個別TXを実装できる |

#### Step 5b: ファイルI/O演習（1-2週）

**目標**: CSV・固定長ファイルの読み書きを実装できる

| # | タスク | 学習内容 | 達成基準 |
|---|--------|---------|----------|
| 1 | データバインド（CSV） | `@Csv`, `@CsvFormat`, `ObjectMapperIterator` | CSV読込バッチの実装ができる |
| 2 | データバインド（固定長） | `@FixedLength` 形式のForm定義 | 固定長ファイルの読み書きができる |
| 3 | 汎用データフォーマット | XML, JSON形式のファイル処理 | フォーマット定義ファイルを作成できる |
| 4 | ファイルパス管理 | `FilePathSetting`によるパス解決 | 環境別のファイルパス設定ができる |

#### Step 5c: テスト実装演習（1-2週）

**目標**: Excelベースのバッチリクエスト単体テストを書けるようになる

| # | タスク | 学習内容 | 達成基準 |
|---|--------|---------|----------|
| 1 | テストFW概要 | テストクラス継承階層、Excelデータ構造 | テストFWのアーキテクチャを説明できる |
| 2 | Excelテストデータ作成 | `SETUP_TABLE`, `EXPECTED_TABLE`, `LIST_MAP` の記述 | 正しいExcelテストデータを作成できる |
| 3 | バッチテスト実装 | `BatchRequestTestSupport`の使い方 | バッチアクションのリクエスト単体テストが書ける |
| 4 | ファイルアサーション | `SETUP_FIXED`/`EXPECTED_FIXED` の使い方 | ファイル入出力のテストが書ける |
| 5 | JUnit 5移行 | `@BatchRequestTest`アノテーション | JUnit 5スタイルでテストが書ける |

#### Step 6: マルチスレッド・常駐バッチ（2-3週）

**目標**: マルチスレッドバッチと常駐バッチの設計・実装ができる

| # | タスク | 学習内容 | 達成基準 |
|---|--------|---------|----------|
| 1 | MultiThreadExecutionHandler | スレッド数設定、メイン/サブスレッド分岐 | マルチスレッドバッチを実装できる |
| 2 | 排他制御 | `DatabaseRecordReader`の悲観ロック、`@Version`楽観ロック | 適切な排他制御方式を選択できる |
| 3 | 常駐バッチ設計 | `ProcessResidentHandler`, `BasicProcessStopHandler` | 常駐バッチのハンドラキューを設計できる |
| 4 | リトライ処理 | `RetryHandler`の設定と使い方 | リトライ可能例外の設計ができる |
| 5 | 常駐バッチテスト | `OneShotLoopHandler`への差替えテスト手法 | 常駐バッチのテストが書ける |

#### Step 7: 総合演習（2-4週）

**目標**: 実務相当の複合バッチシステムを設計・実装・テストする

| # | 演習課題 | 内容 |
|---|---------|------|
| 1 | **月次集計バッチ** | DB→集計処理→DB登録。マルチスレッド、コミット間隔設定、エラーログ、リカバリ対応 |
| 2 | **ファイル連携バッチ** | 外部ファイル取込→バリデーション→DB登録→結果ファイル出力。固定長入力、CSV出力 |
| 3 | **常駐監視バッチ** | テーブルポーリング→新規データ検出→外部API呼出→ステータス更新。常駐バッチ、リトライ |
| 4 | **テスト一式** | 上記3バッチ全てのExcelベーステスト作成。正常系・異常系・境界値 |
| 5 | **設計書作成** | 開発標準テンプレートに従った機能設計書の作成 |

#### Step 8: 上級・応用（継続）

**目標**: アーキテクチャ設計、性能チューニング、運用設計ができる

| # | タスク | 学習内容 |
|---|--------|---------|
| 1 | カスタムハンドラ開発 | プロジェクト固有のハンドラ（認証、監査ログ等）の設計・実装 |
| 2 | 性能チューニング | フェッチサイズ調整、DB接続プール設定、スレッド数最適化 |
| 3 | Jakarta Batch (JSR352) | Batchlet/Chunk、リスナーアーキテクチャ、jBeret |
| 4 | 運用設計 | ジョブスケジューラ連携、シェルスクリプト自動生成ツール活用 |
| 5 | v5→v6移行 | javax→jakarta名前空間変更、BOM更新 |

---

## 3. 学習コンテンツ・教材

### 3.1 Nablarch公式ドキュメント（バッチ関連・URL検証済み）

#### バッチアプリケーション コアドキュメント

| # | ドキュメント | URL | 内容 |
|---|------------|-----|------|
| 1 | バッチアーキテクチャ（日本語） | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html | 都度起動/常駐バッチのアーキテクチャ、ハンドラキュー構成、DataReader/Action概要 |
| 2 | バッチアーキテクチャ（英語） | https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/nablarch_batch/architecture.html | 同上の英語版 |
| 3 | バッチ機能詳細 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html | 11機能領域: 起動方法、リポジトリ初期化、バリデーション、DB、ファイルI/O、排他制御、実行制御等 |
| 4 | Getting Started（バッチ） | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html | CSVインポートバッチのチュートリアル（Form定義、DataReader実装、Action実装） |
| 5 | マルチスレッドバッチ | https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/nablarch_batch/feature_details/nablarch_batch_multiple_process.html | マルチスレッド実行制御、`DatabaseRecordReader`の悲観ロック |
| 6 | バッチエラーハンドリング | https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/nablarch_batch/feature_details/nablarch_batch_error_process.html | `ProcessAbnormalEnd`による異常終了制御 |

#### テスティングフレームワーク

| # | ドキュメント | URL | 内容 |
|---|------------|-----|------|
| 7 | テスティングFWインデックス | https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/index.html | テストFW全体の入口ページ |
| 8 | 単体テストガイド | https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/index.html | テスト種別一覧（クラス単体、リクエスト単体、バッチ、メッセージング等） |
| 9 | バッチリクエスト単体テスト実行方法 | https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/batch.html | `BatchRequestTestSupport`の使い方、Excelテストデータ構造 |
| 10 | バッチリクエスト単体テスト詳細 | https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/06_TestFWGuide/RequestUnitTest_batch.html | DBアサーション、ファイルアサーション、ログアサーション、常駐バッチテスト |
| 11 | JUnit 5拡張 | https://nablarch.github.io/docs/LATEST/doc/en/development_tools/testing_framework/guide/development_guide/06_TestFWGuide/JUnit5_Extension.html | JUnit 5対応、`@BatchRequestTest`等のアノテーション |

#### ライブラリ・設定

| # | ドキュメント | URL | 内容 |
|---|------------|-----|------|
| 12 | データベースアクセス概要 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database_management.html | DB接続管理の全体像 |
| 13 | ユニバーサルDAO | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html | CRUD API、バッチ操作、SQLファイル、ページネーション、遅延ロード |
| 14 | データベースアクセス(JDBCラッパー) | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html | 動的SQL構文(`$if`, `:param[]`, `$sort`) |
| 15 | システムリポジトリ | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html | XML DI コンテナ、Autowire、環境依存値 |
| 16 | Bean Validation | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation/bean_validation.html | ドメインバリデーション、文字種チェック |
| 17 | 標準ハンドラ一覧 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/handlers/index.html | 全ハンドラの一覧とリンク |

#### Jakarta Batch (JSR352)

| # | ドキュメント | URL | 内容 |
|---|------------|-----|------|
| 18 | Jakarta Batchアーキテクチャ | https://nablarch.github.io/docs/LATEST/doc/en/application_framework/application_framework/batch/jsr352/architecture.html | Batchlet/Chunk、リスナーアーキテクチャ、jBeret推奨 |

#### Mavenアーキタイプ

| # | ドキュメント | URL | 内容 |
|---|------------|-----|------|
| 19 | Mavenモジュール構成 | https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/blank_project/MavenModuleStructures/index.html | アーキタイプ一覧、BOM管理 |

### 3.2 GitHubサンプルプロジェクト

| # | リポジトリ | URL | 内容 |
|---|----------|-----|------|
| 1 | nablarch-example-batch | https://github.com/nablarch/nablarch-example-batch | オンデマンド3例 + 常駐1例のバッチ実装。Java 17, Maven 3.9.9+, H2 |
| 2 | nablarch-example-web | https://github.com/nablarch/nablarch-example-web | Web版サンプル（バッチとのDB/TX管理の比較用） |
| 3 | nablarch-testing | https://github.com/nablarch/nablarch-testing | テスティングFWのソースコード |
| 4 | nablarch-testing-junit5 | https://github.com/nablarch/nablarch-testing-junit5 | JUnit 5拡張モジュール |
| 5 | nablarch-development-standards | https://github.com/nablarch-development-standards/nablarch-development-standards | 開発標準（設計書テンプレート、単体テスト標準含む） |
| 6 | nablarch-development-standards-tools | https://github.com/nablarch-development-standards/nablarch-development-standards-tools | シェルスクリプト自動生成ツール等 |
| 7 | nablarch-single-module-archetype | https://github.com/nablarch/nablarch-single-module-archetype | Mavenアーキタイプ（バッチ用含む） |

### 3.3 既存KBからの教材抽出

| KB | セクション | 学習ステップ対応 | 用途 |
|----|----------|---------------|------|
| O-005 セクション3.3 | Nablarchバッチアプリケーション | Step 2-4 | バッチアーキテクチャ概要、ハンドラキュー、DataReader/Action一覧 |
| O-005 セクション3.4 | Jakarta Batch準拠バッチ | Step 8 | JSR352対応バッチの概要 |
| O-005 セクション5 | ライブラリ群の詳細 | Step 4-5 | DB, バリデーション, ファイルI/O, TX管理 |
| O-005 セクション9-10 | テスティングFW・アーキタイプ | Step 5c, 0 | テストカテゴリ、Maven生成コマンド |
| O-006 セクション1.4-1.5 | バッチ用ハンドラキュー | Step 2, 4, 6 | FQCN付きハンドラ一覧（都度起動・常駐）、実プロジェクト構成例 |
| O-006 セクション2.6-2.7 | バッチ/スタンドアローンハンドラ | Step 4, 6 | 全バッチハンドラのFQCNと責務 |
| O-006 セクション5 | DB接続・トランザクション管理 | Step 5a | 接続プール、TX境界、複数DB接続、個別TX、ユニバーサルDAO |
| O-007 セクション3 | テスティングFW詳細 | Step 5c | Excelデータ構造、テストクラス継承階層、バッチテスト手順 |
| O-007 セクション5.5 | バッチハンドラキュー構成 | Step 2, 4 | サンプルプロジェクトのハンドラキュー構成 |

### 3.4 Java/Jakarta EE推奨教材

| # | 教材 | 種別 | 対象 | 内容 |
|---|------|------|------|------|
| 1 | 「プロになるJava」（技術評論社） | 書籍 | 初級 | Java基礎からStream API、ラムダ式まで |
| 2 | Oracle Java Tutorial | Web | 初級 | https://docs.oracle.com/javase/tutorial/ |
| 3 | 「スッキリわかるSQL入門」（インプレス） | 書籍 | 初級 | SQL基礎 |
| 4 | Baeldung Java Tutorials | Web | 初級-中級 | https://www.baeldung.com/java-tutorial |
| 5 | 「Jakarta EE入門」 | 書籍 | 中級 | Jakarta Persistence, Bean Validation 等 |
| 6 | Maven公式ガイド | Web | 初級 | https://maven.apache.org/guides/index.html |
| 7 | JUnit 5 User Guide | Web | 初級 | https://junit.org/junit5/docs/current/user-guide/ |
| 8 | JSR 352: Batch Applications for the Java Platform | 仕様書 | 上級 | https://jcp.org/en/jsr/detail?id=352 |

---

## 4. 解説書・リファレンス

### 4.1 Nablarchバッチアーキテクチャ解説

#### バッチ用ハンドラキュー構成図（都度起動バッチ）

```
  ┌───────────────────────────────────────────────────────────────┐
  │              都度起動バッチ ハンドラキュー構成図               │
  │                                                               │
  │  ┌─────────────────── メインスレッド ───────────────────┐     │
  │  │                                                      │     │
  │  │  [1] StatusCodeConvertHandler                        │     │
  │  │   │  終了コード(例外種別) → プロセス終了コード変換   │     │
  │  │   ▼                                                  │     │
  │  │  [2] ThreadContextClearHandler                       │     │
  │  │   │  スレッドコンテキスト変数の全削除                │     │
  │  │   ▼                                                  │     │
  │  │  [3] GlobalErrorHandler                              │     │
  │  │   │  未捕捉例外のログ出力（最後の砦）               │     │
  │  │   ▼                                                  │     │
  │  │  [4] ThreadContextHandler                            │     │
  │  │   │  リクエストID、ユーザーID等の初期化              │     │
  │  │   ▼                                                  │     │
  │  │  [5] DbConnectionManagementHandler ← 初期化/終了用  │     │
  │  │   │  初期処理・終了処理で使うDB接続を取得            │     │
  │  │   ▼                                                  │     │
  │  │  [6] TransactionManagementHandler  ← 初期化/終了用  │     │
  │  │   │  初期処理・終了処理用トランザクション制御        │     │
  │  │   ▼                                                  │     │
  │  │  [7] RequestPathJavaPackageMapping                   │     │
  │  │   │  -requestPath引数からアクションクラスを特定      │     │
  │  │   ▼                                                  │     │
  │  │  [8] MultiThreadExecutionHandler                     │     │
  │  │   │  サブスレッド生成・並行実行                      │     │
  │  │   │                                                  │     │
  │  └───┼──────────────────────────────────────────────────┘     │
  │      │                                                        │
  │      ▼                                                        │
  │  ┌─────────────────── サブスレッド ────────────────────┐     │
  │  │                                                      │     │
  │  │  [9] DbConnectionManagementHandler ← 業務処理用     │     │
  │  │   │  業務処理で使うDB接続を取得                      │     │
  │  │   ▼                                                  │     │
  │  │  [10] LoopHandler                                    │     │
  │  │   │  トランザクションループ制御                      │     │
  │  │   │  commitInterval件ごとにコミット                  │     │
  │  │   ▼                                                  │     │
  │  │  [11] DataReadHandler                                │     │
  │  │   │  DataReaderから1件ずつデータを読み込み           │     │
  │  │   ▼                                                  │     │
  │  │  [Action.handle()]                                   │     │
  │  │      業務ロジック実行                                │     │
  │  │                                                      │     │
  │  └──────────────────────────────────────────────────────┘     │
  │                                                               │
  │  ポイント:                                                    │
  │  ・DB接続管理が2回出現（メインスレッド用 + サブスレッド用）   │
  │  ・MultiThreadExecutionHandlerがスレッド境界                  │
  │  ・LoopHandlerがコミット間隔を制御                            │
  └───────────────────────────────────────────────────────────────┘
```

#### 都度起動バッチの処理フロー図

```
  ┌───────────────────────────────────────────────────────────────┐
  │            都度起動バッチ 処理フロー図                         │
  └───────────────────────────────────────────────────────────────┘

  [コマンドライン起動]
   │  java -jar batch.jar
   │  -Dexec.mainClass=nablarch.fw.launcher.Main
   │  -Dexec.args="-diConfig batch-config.xml
   │               -requestPath com.example.MyBatchAction/REQ001
   │               -userId batch_user"
   │
   ▼
  [nablarch.fw.launcher.Main]
   │  1. XMLコンポーネント設定をロード
   │  2. SystemRepositoryを初期化
   │  3. ハンドラキューを構築
   │
   ▼
  [ハンドラキュー実行（メインスレッド）]
   │  StatusCodeConvert → GlobalError → ThreadContext
   │  → DbConnection(init) → Transaction(init)
   │  → RequestPathMapping → アクションクラス特定
   │
   ▼
  [Action.initialize()]  ← 初期処理（テーブルクリア等）
   │
   ▼
  [Action.createReader()]  ← DataReader生成
   │  DatabaseRecordReader / FileDataReader / カスタム
   │
   ▼
  [MultiThreadExecutionHandler]
   │  設定されたスレッド数分のサブスレッドを生成
   │
   ├──▶ [サブスレッド1]          [サブスレッド2]         ...
   │     │                        │
   │     ▼                        ▼
   │    [DbConnection(業務)]     [DbConnection(業務)]
   │     │                        │
   │     ▼                        ▼
   │    [LoopHandler]            [LoopHandler]
   │     │  ┌─────────────┐       │  ┌─────────────┐
   │     │  │ DataRead    │       │  │ DataRead    │
   │     │  │   ▼         │       │  │   ▼         │
   │     │  │ handle(data)│       │  │ handle(data)│
   │     │  │   ▼         │       │  │   ▼         │
   │     │  │ N件でcommit │       │  │ N件でcommit │
   │     │  └──▲──────────┘       │  └──▲──────────┘
   │     │     │  次のデータへ     │     │
   │     │     └──────────┘       │     └──────────┘
   │     │                        │
   │     ▼ (全データ処理完了)     ▼
   │
   ▼
  [Action.terminate()]  ← 終了処理
   │
   ▼
  [プロセス終了]  ← 終了コード返却（0:正常, 1:異常等）
```

#### 常駐バッチの処理フロー図

```
  ┌───────────────────────────────────────────────────────────────┐
  │            常駐バッチ 処理フロー図                             │
  └───────────────────────────────────────────────────────────────┘

  [コマンドライン起動]  ← 都度起動と同じMain
   │
   ▼
  [nablarch.fw.launcher.Main]
   │  SystemRepository初期化、ハンドラキュー構築
   │
   ▼
  [メインスレッド ハンドラキュー]
   │
   ▼
  [ProcessResidentHandler]  ← ★常駐バッチ固有
   │  ┌──────────────────────────────────┐
   │  │                                  │
   │  │  [BasicProcessStopHandler]       │
   │  │   │  停止フラグ(DBテーブル)確認  │
   │  │   │  → フラグONなら終了         │
   │  │   ▼                              │
   │  │  [RetryHandler]                  │
   │  │   │  リトライ可能例外の処理      │
   │  │   ▼                              │
   │  │  [MultiThreadExecutionHandler]   │
   │  │   │  サブスレッド生成            │
   │  │   │                              │
   │  │   ├──▶ [サブスレッド]            │
   │  │   │     │                        │
   │  │   │     ▼                        │
   │  │   │    DataRead                  │
   │  │   │     │  データあり → 処理     │
   │  │   │     │  データなし → 待機     │
   │  │   │     ▼                        │
   │  │   │    handle(data)              │
   │  │   │     │                        │
   │  │   │     ▼ commit                 │
   │  │   │                              │
   │  │   ▼  全サブスレッド完了          │
   │  │                                  │
   │  │  ★ dataWaitInterval 秒待機      │
   │  │                                  │
   │  └──▲───────────────────────────────┘
   │     │  停止フラグOFFなら繰り返し
   │     └────────────────────────┘
   │
   ▼
  [プロセス終了]  ← 停止フラグONで正常終了
```

### 4.2 都度起動バッチと常駐バッチの比較表

| 観点 | 都度起動バッチ | 常駐バッチ |
|------|-------------|-----------|
| **起動方式** | ジョブスケジューラ等からの定期起動 | デーモンプロセスとして常駐 |
| **起動クラス** | `nablarch.fw.launcher.Main` | 同左 |
| **処理対象** | 起動時点でのデータ全件 | 一定間隔で新規データを監視 |
| **終了条件** | 全データ処理完了 | 停止フラグ(DB)がON |
| **ハンドラキュー** | 9ハンドラ（DB有） / 6ハンドラ（DB無） | 9+5=14ハンドラ |
| **追加ハンドラ** | なし | ProcessResidentHandler, BasicProcessStopHandler, RetryHandler, ThreadContextHandler/ClearHandler |
| **データ待機** | なし | `dataWaitInterval`秒待機 |
| **リトライ** | なし（通常） | RetryHandlerで自動リトライ |
| **推奨度** | 推奨 | テーブルキューメッセージング推奨（新規の場合） |
| **テスト方法** | `BatchRequestTestSupport` | `OneShotLoopHandler`に差替えて1回実行 |
| **ユースケース** | 月次集計、データ移行、レポート生成 | メッセージ監視、ファイル監視 |

### 4.3 バッチ用ハンドラの全一覧

#### 共通ハンドラ（バッチで使用するもの）

| # | ハンドラ名 | FQCN | 役割 |
|---|-----------|------|------|
| 1 | ステータスコード変換 | `nablarch.fw.handler.StatusCodeConvertHandler` | 例外種別→プロセス終了コード変換 |
| 2 | グローバルエラー | `nablarch.fw.handler.GlobalErrorHandler` | 未捕捉例外のログ出力（最後の砦） |
| 3 | スレッドコンテキスト変数管理 | `nablarch.common.handler.threadcontext.ThreadContextHandler` | リクエストID、ユーザーID等の初期化 |
| 4 | スレッドコンテキスト変数削除 | `nablarch.common.handler.threadcontext.ThreadContextClearHandler` | スレッドローカル変数の全削除 |
| 5 | DB接続管理 | `nablarch.common.handler.DbConnectionManagementHandler` | DB接続の取得/解放 |
| 6 | トランザクション制御 | `nablarch.common.handler.TransactionManagementHandler` | TX開始/コミット/ロールバック |
| 7 | リクエストディスパッチ | `nablarch.fw.handler.RequestPathJavaPackageMapping` | `-requestPath`引数からアクションクラスを特定 |

#### バッチ/スタンドアローン専用ハンドラ

| # | ハンドラ名 | FQCN | 役割 | 使用場面 |
|---|-----------|------|------|---------|
| 8 | マルチスレッド実行制御 | `nablarch.fw.handler.MultiThreadExecutionHandler` | サブスレッド生成・並行実行 | 都度起動・常駐 |
| 9 | トランザクションループ制御 | `nablarch.fw.handler.LoopHandler` | コミット間隔制御、業務TX制御 | 都度起動・常駐（DB有） |
| 10 | ループ制御（DBなし） | `nablarch.fw.handler.DbLessLoopHandler` | DB不要のループ制御 | 都度起動（DB無） |
| 11 | データリード | `nablarch.fw.handler.DataReadHandler` | DataReaderから1件ずつデータ読込 | 都度起動・常駐 |
| 12 | プロセス常駐化 | `nablarch.fw.handler.ProcessResidentHandler` | 一定間隔での繰り返し実行 | 常駐バッチのみ |
| 13 | プロセス停止制御 | `nablarch.fw.handler.BasicProcessStopHandler` | 停止フラグ(DB)のチェック | 常駐バッチのみ |
| 14 | リトライ | `nablarch.fw.handler.RetryHandler` | リトライ可能例外の処理 | 常駐バッチのみ |
| 15 | プロセス多重起動防止 | `nablarch.fw.handler.DuplicateProcessCheckHandler` | 多重起動防止 | 必要時 |
| 16 | リクエストスレッド内ループ | `nablarch.fw.handler.RequestThreadLoopHandler` | ループ制御 | MOMメッセージング |
| 17 | 出力ファイル開放 | `nablarch.fw.handler.FileRecordWriterDisposeHandler` | 出力ファイルリソース解放 | ファイル出力時 |

#### DataReader一覧

| # | DataReader | FQCN | 用途 |
|---|-----------|------|------|
| 1 | DatabaseRecordReader | `nablarch.fw.reader.DatabaseRecordReader` | DBクエリ結果を1件ずつ読込 |
| 2 | FileDataReader | `nablarch.fw.reader.FileDataReader` | ファイル（CSV/固定長）を1行ずつ読込 |
| 3 | ValidatableFileDataReader | `nablarch.fw.reader.ValidatableFileDataReader` | バリデーション付きファイル読込 |
| 4 | ResumeDataReader | `nablarch.fw.reader.ResumeDataReader` | 再開可能なデータ読込（中断箇所からの再開） |

#### Action基底クラス一覧

| # | Action | FQCN | 用途 |
|---|--------|------|------|
| 1 | BatchAction | `nablarch.fw.action.BatchAction` | 汎用バッチ（`createReader()` + `handle()` + `initialize()`/`terminate()`） |
| 2 | FileBatchAction | `nablarch.fw.action.FileBatchAction` | ファイル入力バッチ（`FileDataReader`自動設定） |
| 3 | NoInputDataBatchAction | `nablarch.fw.action.NoInputDataBatchAction` | 入力データ不要バッチ |
| 4 | AsyncMessageSendAction | `nablarch.fw.action.AsyncMessageSendAction` | 非同期メッセージ送信バッチ |

### 4.4 DB接続・トランザクション管理の仕組み

#### バッチでのトランザクション境界

```
  メインスレッド:
  ┌──────────────────────────────────────────────────┐
  │ DbConnectionManagement(init)                     │
  │   │ ← DB接続取得                                 │
  │   ▼                                              │
  │ TransactionManagement(init)                      │
  │   │ ← TX開始                                     │
  │   ▼                                              │
  │ Action.initialize()  ← 初期処理（このTX内で実行）│
  │   │                                              │
  │ MultiThreadExecution → サブスレッドへ            │
  │   │                                              │
  │ Action.terminate()  ← 終了処理（このTX内で実行） │
  │   │                                              │
  │ TransactionManagement(cleanup) → コミット        │
  │ DbConnectionManagement(cleanup) → 接続解放       │
  └──────────────────────────────────────────────────┘

  サブスレッド:
  ┌──────────────────────────────────────────────────┐
  │ DbConnectionManagement(業務)                     │
  │   │ ← DB接続取得（メインとは別の接続）           │
  │   ▼                                              │
  │ LoopHandler                                      │
  │   │  ┌────────────────────────────────┐          │
  │   │  │ TX開始                         │          │
  │   │  │  ▼                             │          │
  │   │  │ DataRead → handle(data)        │ × N件   │
  │   │  │  ▼                             │          │
  │   │  │ コミット（commitInterval件ごと）│          │
  │   │  └──▲─────────────────────────────┘          │
  │   │     │ 次のデータへ                            │
  │   │     └──────────────┘                          │
  │   ▼                                              │
  │ DbConnectionManagement(cleanup) → 接続解放       │
  └──────────────────────────────────────────────────┘
```

#### マルチスレッド実行時の注意点

| # | 注意点 | 説明 |
|---|--------|------|
| 1 | DB接続は**スレッドごと** | メインスレッドとサブスレッドで別のDB接続を使用。ハンドラキューにDB接続管理が2回出現する理由 |
| 2 | DataReaderの**スレッドセーフティ** | `DatabaseRecordReader`は内部で`SELECT FOR UPDATE`による悲観ロックを使用。カスタムReaderは自分でスレッドセーフにする必要あり |
| 3 | コミット間隔の設計 | `LoopHandler`の`transactionCommitInterval`で設定。大きすぎるとロールバック範囲拡大、小さすぎるとオーバーヘッド増 |
| 4 | 排他制御 | マルチスレッドバッチではレコード排他が必要。`DatabaseRecordReader`使用時は自動で悲観ロック |
| 5 | 例外発生時 | サブスレッドで例外が発生してもメインスレッドの初期化/終了処理用TXには影響しない |

### 4.5 テスティングFWの使い方

#### Excelテストデータの書き方（バッチ用）

```
  Excelファイル: MyBatchActionRequestTest.xlsx

  ┌─────────── シート名: testNormalCase ──────────────────┐
  │                                                        │
  │  ■ LIST_MAP = testShots                               │
  │  ┌────┬────────────┬──────────────┬────────────┐      │
  │  │ no │description │expectedStat..│ diConfig   │      │
  │  ├────┼────────────┼──────────────┼────────────┤      │
  │  │ 1  │正常系テスト│ 0            │ batch-c... │      │
  │  │ 2  │異常データ  │ 1            │ batch-c... │      │
  │  └────┴────────────┴──────────────┴────────────┘      │
  │  (+ requestPath, userId, setUpTable, expectedTable,    │
  │     setUpFile, expectedFile, expectedLog, args[0]...)  │
  │                                                        │
  │  ■ SETUP_TABLE = SETUP_TABLE_DATA_1                   │
  │  ┌──────────┬───────┬───────┐                          │
  │  │ TABLE_A  │ col1  │ col2  │                          │
  │  ├──────────┼───────┼───────┤                          │
  │  │          │ val1  │ val2  │                          │
  │  └──────────┴───────┴───────┘                          │
  │                                                        │
  │  ■ EXPECTED_TABLE = EXPECTED_TABLE_DATA_1             │
  │  ┌──────────┬───────┬───────┐                          │
  │  │ TABLE_B  │ col1  │ col2  │                          │
  │  ├──────────┼───────┼───────┤                          │
  │  │          │ res1  │ res2  │                          │
  │  └──────────┴───────┴───────┘                          │
  │                                                        │
  │  ■ SETUP_FIXED = inputFile (入力ファイル)             │
  │  ┌───────────────────────────────┐                     │
  │  │ 00001TaroYamada  19900101... │                     │
  │  │ 00002HanakoSato  19850315... │                     │
  │  └───────────────────────────────┘                     │
  │                                                        │
  │  ■ EXPECTED_FIXED = outputFile (期待出力ファイル)     │
  │  ┌───────────────────────────────┐                     │
  │  │ 00001TaroYamada  OK ...      │                     │
  │  └───────────────────────────────┘                     │
  └────────────────────────────────────────────────────────┘
```

#### テストクラスの継承階層（バッチ関連）

```
  TestEventDispatcher
    └── TestSupport (Excelデータアクセス、ThreadContext)
        └── DbAccessTestSupport (DB setup/assert)
            └── StandaloneTestSupportTemplate
                └── BatchRequestTestSupport  ★バッチテスト用
```

**JUnit 4スタイル**:
```java
public class MyBatchActionRequestTest extends BatchRequestTestSupport {
    @Test
    public void testNormalCase() {
        execute();  // シート名"testNormalCase"のテストデータを使用
    }
}
```

**JUnit 5スタイル**:
```java
@BatchRequestTest
class MyBatchActionRequestTest {
    BatchRequestTestSupport support;  // Extensionにより自動注入

    @Test
    void testNormalCase() {
        support.execute();
    }
}
```

### 4.6 よくあるトラブルと対処法

| # | トラブル | 原因 | 対処法 |
|---|---------|------|--------|
| 1 | バッチが起動しない | `-requestPath` のFQCNが間違っている | パッケージ名+クラス名を正確に指定。`-requestPath=com.example.MyBatchAction/REQ001` |
| 2 | DB接続エラー | env.properties のDB設定が環境と不一致 | `${db.url}`, `${db.user}`, `${db.password}` を確認 |
| 3 | TX管理がおかしい | ハンドラキューでDB接続管理がTX管理より後に配置 | DB接続管理 → TX管理 の順序を確認（DB接続がTX開始に必要） |
| 4 | マルチスレッドで重複処理 | DataReaderがスレッドセーフでない | `DatabaseRecordReader`を使う（`SELECT FOR UPDATE`自動適用）かカスタムReaderで排他制御 |
| 5 | コミット間隔が効かない | `LoopHandler`のプロパティ名の誤り | `<property name="transactionCommitInterval" value="100" />` |
| 6 | 常駐バッチが停止しない | 停止フラグテーブルの設定不備 | `BasicProcessStopHandler`のテーブル名/カラム名設定を確認 |
| 7 | テストでExcelが読めない | ファイル名がテストクラス名と不一致 | `MyBatchActionRequestTest.java` → `MyBatchActionRequestTest.xlsx` |
| 8 | テストでDB assertが失敗 | `EXPECTED_TABLE`のカラム順・値の不一致 | Excelのセル書式がテキストであることを確認。null値は「null」と記載 |
| 9 | 性能が出ない | フェッチサイズが小さい / スレッド数が不適切 | `common.properties`の`nablarch.db.resultSet.fetchSize`を調整（デフォルト50） |
| 10 | OutOfMemoryError | 遅延ロードを使わず全件メモリに読込 | `UniversalDao.defer().findAllBySqlFile()` で遅延ロードを使用 |

---

## 5. 学習計画テンプレート

### 5.1 3ヶ月プラン（初級→中級到達）

**対象**: Javaの基礎はあるがNablarch未経験のエンジニア

| 週 | ステップ | 学習内容 | 達成目標 |
|----|---------|---------|----------|
| 1 | Step 0 | 環境構築、ツールセットアップ | 開発環境が整い、Mavenビルドが通る |
| 2 | Step 1 | Java SE 17基礎復習（コレクション、例外、アノテーション） | Nablarchで使うJava機能を一通り確認 |
| 3 | Step 2 | Nablarchコンセプト理解（ハンドラキュー、XML設定） | パイプライン処理モデルを説明できる |
| 4 | Step 3 | nablarch-example-batch実行、デバッグ | 3つのサンプルバッチを動かし、コードを追跡できる |
| 5-6 | Step 4 | 都度起動バッチ実装演習①: DB→DBバッチ | `BatchAction` + `DatabaseRecordReader`の実装ができる |
| 7 | Step 4 | 都度起動バッチ実装演習②: CSV→DBバッチ | データバインドによるCSV読込が実装できる |
| 8 | Step 5a | DB/TX管理の深掘り（メイン/サブスレッド、コミット間隔） | TX境界を正しく設計できる |
| 9 | Step 5b | ファイルI/O演習（CSV出力、固定長ファイル） | ファイル入出力を実装できる |
| 10-11 | Step 5c | テスト実装演習（Excel作成、BatchRequestTestSupport） | バッチのリクエスト単体テストが書ける |
| 12 | 確認 | 中級スキル到達度チェック、振り返り | スキルチェックリストの中級項目を全てクリア |

### 5.2 6ヶ月プラン（初級→上級到達）

**対象**: Javaの基礎はあるがNablarch未経験のエンジニア

| 月 | 週 | ステップ | 学習内容 | 達成目標 |
|----|-----|---------|---------|----------|
| 1月 | 1-2 | Step 0-1 | 環境構築、Java基礎復習 | 開発環境完成、Java基礎の確認 |
| 1月 | 3-4 | Step 2-3 | Nablarchコンセプト、サンプル実行 | ハンドラキューを理解し、サンプルを動かせる |
| 2月 | 5-7 | Step 4 | 都度起動バッチ実装演習（3課題） | 独力でバッチアクションを実装できる |
| 2月 | 8 | Step 5a | DB/TX管理深掘り | TX境界を正しく設計できる |
| 3月 | 9-10 | Step 5b-5c | ファイルI/O + テスト実装 | ファイル処理とテストの両方が書ける |
| 3月 | 11-12 | Step 6 | マルチスレッドバッチ、排他制御 | マルチスレッドバッチの実装ができる |
| 4月 | 13-14 | Step 6 | 常駐バッチ設計・実装、リトライ | 常駐バッチを設計・テストできる |
| 4月 | 15-16 | Step 7 | 総合演習①: 月次集計バッチ | 実務相当のバッチをフルスタックで実装 |
| 5月 | 17-18 | Step 7 | 総合演習②: ファイル連携バッチ | ファイル入出力を含む複合バッチ |
| 5月 | 19-20 | Step 7 | 総合演習③: 常駐監視バッチ + テスト一式 | 常駐バッチとテスト戦略 |
| 6月 | 21-22 | Step 8 | 上級応用: エラー設計、性能チューニング | 性能問題の特定・解決ができる |
| 6月 | 23-24 | Step 8 | 上級応用: 設計書作成、コードレビュー | 開発標準に従った設計書とレビューができる |

### 5.3 チーム研修カリキュラム案（5人チーム、4週間集中）

**前提**: Java中級レベルのエンジニア5名、1日8時間（講義2h + ハンズオン6h）

| 日 | 午前（講義 2h） | 午後（ハンズオン 6h） |
|----|-----------|--------------------|
| **Week 1: 基礎** | | |
| Day 1 | Nablarchアーキテクチャ概要、ハンドラキューの仕組み | 環境構築、nablarch-example-batch clone & ビルド |
| Day 2 | XML設定（component-configuration）、システムリポジトリ | サンプルバッチ3種の実行・コードリーディング |
| Day 3 | バッチアーキテクチャ（都度起動/常駐）、DataReader/Action | サンプルバッチにブレークポイント設置、ハンドラ実行追跡 |
| Day 4 | ユニバーサルDAO、SQLファイル | DB→DBバッチの実装演習 |
| Day 5 | データバインド（CSV/固定長）、Bean Validation | CSV→DBバッチの実装演習 |
| **Week 2: 実装** | | |
| Day 6 | DB接続・TX管理の詳細（メイン/サブスレッド） | DB→CSVバッチ + バリデーション付きバッチの実装演習 |
| Day 7 | テスティングFW概要、Excelテストデータ構造 | テストExcel作成演習 |
| Day 8 | BatchRequestTestSupport、JUnit 5拡張 | 実装したバッチのテスト作成 |
| Day 9 | マルチスレッドバッチ、排他制御 | マルチスレッドバッチの実装演習 |
| Day 10 | 常駐バッチ、リトライ処理 | 常駐バッチの実装演習 |
| **Week 3: 応用** | | |
| Day 11 | エラーハンドリング設計（ProcessAbnormalEnd、リカバリ） | エラー処理を組み込んだバッチの実装 |
| Day 12 | 性能設計（フェッチサイズ、コミット間隔、スレッド数） | 性能計測と最適化演習 |
| Day 13 | 開発標準（設計書テンプレート、コーディング規約） | 機能設計書の作成演習 |
| Day 14-15 | 総合演習キックオフ、要件説明 | 総合演習: チームで月次集計バッチシステムを設計・実装 |
| **Week 4: 総合演習** | | |
| Day 16-17 | 中間レビュー（設計・コードレビュー） | 総合演習: 実装・テスト作成 |
| Day 18-19 | テスト戦略、品質基準 | 総合演習: テスト・デバッグ・リファクタリング |
| Day 20 | 成果発表、振り返り、スキルチェック | 発表準備、スキルチェックリスト評価 |

### 5.4 スキル評価チェックリスト

#### 初級チェックリスト

| # | 評価項目 | 達成 |
|---|---------|------|
| 1 | Java SE 17の基本構文（コレクション、例外、アノテーション）を説明できる | [ ] |
| 2 | Mavenのpom.xml、ビルドライフサイクル、プロファイルを説明できる | [ ] |
| 3 | Nablarchのハンドラキューアーキテクチャ（パイプライン処理）を図で説明できる | [ ] |
| 4 | XMLコンポーネント定義の基本構造（component, property, list, import）を読める | [ ] |
| 5 | nablarch-example-batchのサンプルバッチを実行できる | [ ] |
| 6 | `-requestPath`引数の意味を説明できる | [ ] |

#### 中級チェックリスト

| # | 評価項目 | 達成 |
|---|---------|------|
| 1 | 都度起動バッチの9ハンドラ構成を順序付きで説明できる | [ ] |
| 2 | `BatchAction<D>`の`handle()`と`createReader()`を実装できる | [ ] |
| 3 | `DatabaseRecordReader`と`FileDataReader`を適切に使い分けられる | [ ] |
| 4 | メインスレッド用とサブスレッド用のDB接続の違いを説明できる | [ ] |
| 5 | ユニバーサルDAOを使ったCRUD操作とSQLファイルを実装できる | [ ] |
| 6 | 動的SQL構文（`$if`, `:param[]`）を正しく使える | [ ] |
| 7 | データバインドによるCSV/固定長ファイルの読み書きを実装できる | [ ] |
| 8 | Bean Validationによる入力バリデーションを設定できる | [ ] |
| 9 | `LoopHandler`のコミット間隔を適切に設定できる | [ ] |
| 10 | Excelベースのバッチリクエスト単体テストを作成できる | [ ] |
| 11 | 開発標準に従った設計書を作成できる | [ ] |

#### 上級チェックリスト

| # | 評価項目 | 達成 |
|---|---------|------|
| 1 | マルチスレッドバッチの設計・実装ができる | [ ] |
| 2 | 常駐バッチのハンドラキュー設計ができる | [ ] |
| 3 | 複数DB接続を使うバッチを設計・実装できる | [ ] |
| 4 | 個別トランザクション（`SimpleDbTransactionManager`）を実装できる | [ ] |
| 5 | `ProcessAbnormalEnd`を使ったエラーハンドリングを設計できる | [ ] |
| 6 | 常駐バッチのテスト（`OneShotLoopHandler`差替え）が書ける | [ ] |
| 7 | 排他制御方式（楽観/悲観）を要件に応じて選択できる | [ ] |
| 8 | 性能要件に基づいたフェッチサイズ・コミット間隔・スレッド数の設計ができる | [ ] |
| 9 | JUnit 5ベースのテストに移行できる | [ ] |

#### エキスパートチェックリスト

| # | 評価項目 | 達成 |
|---|---------|------|
| 1 | カスタムハンドラ（`Handler<TData, TResult>`実装）を設計・実装できる | [ ] |
| 2 | カスタムインターセプタ（`@Interceptor`メタアノテーション）を設計・実装できる | [ ] |
| 3 | 大規模バッチの性能問題を特定・解決できる | [ ] |
| 4 | ジョブスケジューラ連携を含む運用設計ができる | [ ] |
| 5 | チームのバッチコード・設計書をレビューできる | [ ] |
| 6 | Nablarchバッチ vs Jakarta Batch vs Spring Batch の選定根拠を説明できる | [ ] |

---

*本レポートはashigaru1がsubtask_033 (cmd_015)として作成。O-005/O-006/O-007の既存KB、公式ドキュメント（URL検証済み）、GitHubサンプルプロジェクトを参照。全URLは2026-02-02時点でWebFetch/WebSearchにて検証済み。Xenlon関連情報は含まない。*
