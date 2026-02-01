# Fintan（TISナレッジサイト）Nablarch関連コンテンツ調査報告

> **調査日**: 2026-02-01
> **調査者**: 足軽4号（subtask_019 / cmd_007）
> **対象**: https://fintan.jp/ のNablarch関連コンテンツ

---

## 目次

1. [Fintanサイト概要](#1-fintanサイト概要)
2. [Nablarch関連コンテンツ一覧](#2-nablarch関連コンテンツ一覧)
3. [システム開発ガイド](#3-システム開発ガイド)
4. [SPA + REST APIリファレンス](#4-spa--rest-apiリファレンス)
5. [導入事例・ベストプラクティス](#5-導入事例ベストプラクティス)
6. [開発ツール・環境構築ガイド](#6-開発ツール環境構築ガイド)
7. [コーディング規約・静的解析](#7-コーディング規約静的解析)
8. [テスト関連コンテンツ](#8-テスト関連コンテンツ)
9. [セキュリティ関連](#9-セキュリティ関連)
10. [TIS関連・Nablarch以外のコンテンツ](#10-tis関連nablarch以外のコンテンツ)
11. [ライセンス・利用規約](#11-ライセンス利用規約)
12. [GitHub関連リポジトリ一覧](#12-github関連リポジトリ一覧)

---

## 1. Fintanサイト概要

### 1.1 Fintanとは

Fintanは2018年にTIS株式会社テクノロジー&イノベーション本部が立ち上げた技術ノウハウ共有サイトである。

- **URL**: https://fintan.jp/
- **運営**: TIS株式会社 テクノロジー&イノベーション本部
- **ミッション**: 「先進技術・ノウハウを駆使して、新しい社会の活力を創造し、人々の笑顔を増やしていく」
- **コンテンツ**: 研究成果、開発プラクティス、テンプレート/サンプルなど再利用可能なコンテンツを公開
- **料金**: 無料
- **分類体系**: SWEBOK V3.0に基づく

**ソース**: https://fintan.jp/about/

### 1.2 コンテンツカテゴリ

Fintanは以下のカテゴリでコンテンツを提供している：

| カテゴリ | 件数（概算） |
|---------|------------|
| アプリケーション開発事例 | 約100件 |
| ソフトウェアテスティング | 約16件 |
| エンジニア育成・学習 | 約15件 |
| アジャイル・スクラム | 約14件 |
| モバイルアプリケーション開発 | 約10件 |
| 開発プロセス | 約10件 |
| ITアーキテクチャ | 複数 |

**ソース**: https://fintan.jp/?page_id=182

### 1.3 ロール別コンテンツ

| ロール | URL |
|-------|-----|
| アーキテクト向け | https://fintan.jp/for-architect/ |
| バックエンドエンジニア向け | https://fintan.jp/for-backend-engineer/ |
| フロントエンドエンジニア向け | https://fintan.jp/for-frontend-engineer/ |
| モバイルエンジニア向け | https://fintan.jp/for-mobile-engineer/ |
| デザイナー向け | https://fintan.jp/for-designer/ |
| 新規事業関係者向け | https://fintan.jp/for-new-biz-starter/ |

---

## 2. Nablarch関連コンテンツ一覧

### 2.1 公式Nablarchページ群

| # | タイトル | URL | 言語 | 概要 |
|---|---------|-----|------|------|
| 1 | Nablarch（総合ページ） | https://fintan.jp/en/page/1954/ | EN/JA | Nablarch全コンテンツのポータル |
| 2 | Nablarch Application Framework | https://fintan.jp/en/page/1660/ | EN | フレームワーク本体の説明 |
| 3 | Nablarch System Development Guide | https://fintan.jp/en/page/1667/ | EN | システム開発ガイド |
| 4 | Nablarchシステム開発ガイド | https://fintan.jp/page/252/9/ | JA | 同上（日本語版、より詳細） |
| 5 | Nablarch Development Standard | https://fintan.jp/en/page/1658/ | EN | 開発標準 |
| 6 | Nablarch Testing Framework | https://fintan.jp/en/page/1662/ | EN | テスティングフレームワーク |
| 7 | Nablarchとは？特長や開発方法について解説 | https://fintan.jp/page/1868/4/ | JA | 特長・学習方法の解説 |
| 8 | Nablarchの開発がどのように行われているかのご紹介 | https://fintan.jp/page/1590/ | JA | Nablarch自体の開発プロセス |
| 9 | NablarchのUniversalDaoでできるDB操作 | https://fintan.jp/page/1587/ | JA | DB操作の詳細解説 |
| 10 | Nablarchトレーニングコンテンツ | https://fintan.jp/page/259/ | JA | 自習用トレーニング教材 |
| 11 | Nablarch Blog Category | https://fintan.jp/en/blog-category/nablarch-en/ | EN | Nablarch関連ブログ記事一覧 |
| 12 | Nablarch Team 著者ページ | https://fintan.jp/en/author/nablarch-team/ | EN | Nablarch Teamの全記事 |

### 2.2 SPA + REST API関連（Nablarchバックエンド）

| # | タイトル | URL | 概要 |
|---|---------|-----|------|
| 13 | SPA + REST API構成のサービス開発リファレンス | https://fintan.jp/page/1453/ | メインリファレンス |
| 14 | REST APIを1つ作成してみよう | https://fintan.jp/page/426/ | REST APIチュートリアル |
| 15 | 会津大生向けSPA(React)+API(Nablarch)ハンズオン | https://fintan.jp/page/392/ | 大学向けハンズオン |
| 16 | バックエンド×リファレンス、フロントエンド×自前 | https://fintan.jp/page/446/ | Vue.js連携事例 |
| 17 | CSRF対策をしてみよう | https://fintan.jp/page/428/ | CSRF対策チュートリアル |
| 18 | Webアプリケーション導入編 | https://fintan.jp/?p=6078 | 入門ガイド |
| 19 | バリデーションしてみよう | https://fintan.jp/page/421/ | バリデーションチュートリアル |
| 20 | 方式設計ガイドを使ってみよう | https://fintan.jp/page/432/ | 方式設計ガイド活用法 |

### 2.3 関連コンテンツ（フレームワーク非依存含む）

| # | タイトル | URL | 概要 |
|---|---------|-----|------|
| 21 | Coding Standards（コーディング規約） | https://fintan.jp/en/page/5986/ | Java等のコーディング規約 |
| 22 | Programmers Self Checklist | https://fintan.jp/en/page/1721/ | プログラマー自己チェックリスト |
| 23 | Java FW & 設計開発支援コンテンツについて | https://fintan.jp/en/page/6004/ | FW依存/非依存コンテンツ一覧 |
| 24 | アプリケーション方式設計書サンプル | https://fintan.jp/page/317/ | Nablarch前提の設計書サンプル |
| 25 | About Trademarks | https://fintan.jp/en/page/1891/ | 商標情報 |

---

## 3. システム開発ガイド

### 3.1 Nablarchシステム開発ガイド

**URL**: https://fintan.jp/en/page/1667/ （EN）/ https://fintan.jp/page/252/9/ （JA）

Nablarchを使ったシステム開発の全体を俯瞰するガイド。ウォーターフォール開発を前提とし、各工程で参照すべきNablarchコンテンツと活用方法を示す。

**対象者**: Nablarchに不慣れなエンジニア（特にITアーキテクト、プロジェクトマネージャー）

#### 開発フェーズと主要アクティビティ

| フェーズ | 主要アクティビティ | 参照コンテンツ |
|---------|-------------------|---------------|
| **要件定義** | 業務/システム要件確立、アーキテクチャ設計、テスト計画、開発ガイド作成 | 要件定義フレームワーク、非機能要求グレード（IPA） |
| **方式設計** | 開発標準カスタマイズ、DB設計標準、UI標準策定 | Nablarch開発標準、アプリケーション方式設計書サンプル |
| **設計** | UT標準確立（Heavy/Lightweight型）、パッケージ構成検討 | テスティングFW、開発標準 |
| **PGUT** | 初期プロジェクト構築、チーム環境構築、静的解析導入 | ブランクプロジェクト、Checkstyle/SpotBugs |
| **結合テスト** | 疎通確認、環境設定 | テスティングFW |

#### テスト方法論（2つのアプローチ）

- **Heavy型**: 包括的な自動テスト。大規模プロジェクトで変更が少ない場合に適切。テストデータをExcelで詳細管理。
- **Lightweight型**: テスト自動化を選択的に実施し手動テストを組み合わせる。小規模で頻繁に変更があるプロジェクトに適切。

#### セキュリティ設計

- IPAの「安全なウェブサイトの作り方」を参照
- 「セキュリティ実装チェックリスト」の全項目が「根本的解決」になるよう設計
- **Nablarch機能セキュリティマトリクス**: Nablarchの機能でカバーできるセキュリティ項目を確認するためのマトリクス
- OWASP Top 10も補助的に参照

### 3.2 Nablarch開発標準

**URL**: https://fintan.jp/en/page/1658/
**GitHub**: https://github.com/nablarch-development-standards/nablarch-development-standards/

大規模システムにおける開発者間の成果物品質のばらつきを抑制するためのガイドライン。

#### 4つの構成要素

| コンポーネント | 内容 | 言語 |
|-------------|------|------|
| 開発プロセス標準 | Nablarchでのシステム開発の作業プロセス | 日本語のみ |
| アプリケーション開発標準 | UI標準、コーディング規約、UT仕様 | 英語あり |
| 設計書フォーマット＆サンプル | 設計書のテンプレートと記載例 | 日本語のみ |
| 開発プロセス支援ツール | ソースコード・ドキュメント自動生成 | 英語あり |

### 3.3 アプリケーション方式設計書サンプル

**URL**: https://fintan.jp/page/317/

要件定義工程でアプリケーション方式に合意する際の成果物サンプル。Nablarch採用を前提。

- **オンプレミス環境向け**: nablarch_sample.zip
- **AWS環境向け**: nablarch_sample_aws.zip
- 方式設計書本紙のみ公開（セキュリティ設計書等は非公開）

---

## 4. SPA + REST APIリファレンス

### 4.1 メインリファレンス

**URL**: https://fintan.jp/page/1453/
**投稿**: 2020-09-30 / **更新**: 2025-05-30

TISの協業開発チームが提供する、SPA + REST API構成のWebアプリケーション開発リファレンス。

#### 技術スタック

| レイヤー | 技術 |
|---------|------|
| フロントエンド | React（TypeScript）、React Router、create-react-app |
| バックエンド | Nablarch（Java） |
| DB | PostgreSQL |
| キャッシュ | Redis |
| コンテナ | Docker Compose |

#### 3つの主要コンテンツ

1. **方式設計ガイド**: REST API設計、SPA・API通信、ファイルダウンロード/アップロード、CORS、XSS、CSRF対策
2. **コード例**: 小規模チャットアプリケーション（GitHubで公開）
3. **ハンズオンコンテンツ**: 初心者向け実践教材（Next.jsも採用）

#### 方式設計ガイドのカバー項目

- バリデーション設計
- API設計
- データ取得方法（Fetch）
- CORS対策
- ファイルダウンロード/アップロード
- XSS対策
- CSRF対策

### 4.2 チュートリアルシリーズ

#### 4.2.1 REST APIを1つ作成してみよう

**URL**: https://fintan.jp/page/426/

Todo一覧取得APIの実装チュートリアル。

- PostgreSQL + Flyway（マイグレーション）
- UniversalDao でDB操作
- `@Path("/todos")` + `@GET` でエンドポイント定義
- `SimpleRestTestSupport` でテスト自動化
- TypeScript側は `RestClient` で `/api/todos` に接続

**実行コマンド**:
```bash
docker-compose up -d    # DB起動
mvn test                # テスト実行
mvn jetty:run           # バックエンド起動
npm start               # フロントエンド起動
```

#### 4.2.2 CSRF対策

**URL**: https://fintan.jp/page/428/

- **バックエンド**: `CsrfTokenUtil` でトークン生成、`CsrfTokenVerificationHandler` で検証
- **フロントエンド**: `useEffect` で初期化時にトークン取得、以降のリクエストに埋め込み
- `rest-component-configuration.xml` でハンドラ登録

#### 4.2.3 バリデーション

**URL**: https://fintan.jp/page/421/

- **バックエンド**: Bean Validation（`@NotBlank`等）+ `ValidatorUtil.validate()`
- **フロントエンド**: `useValidation` フック + `stringField().required().maxLength(20)`
- フロントエンド・バックエンド両方での実装を推奨（セキュリティ上の理由）

#### 4.2.4 Vue.jsフロントエンド連携

**URL**: https://fintan.jp/page/446/

SPA + REST APIのフロントエンド・バックエンド疎結合を実証。

- バックエンド: Nablarch（既存のサービス開発リファレンスをそのまま使用）
- フロントエンド: Vue CLI + TypeScript（ポート3000）
- `RestClient.ts`: Fetch APIラップ、CORS対応、指数バックオフリトライ
- React版と同等のバックエンド通信コードで実装可能 → **フレームワーク非依存の設計が有効**

#### 4.2.5 方式設計ガイドの使い方

**URL**: https://fintan.jp/page/432/

- GitHubの `spa-restapi-guide` リポジトリをClone
- MkDocsでドキュメントサイト生成
- textlintで文章校正
- カスタマイズ: `mkdocs.yml` 編集、`docs/` にMarkdown追加

### 4.3 会津大学ハンズオン

**URL**: https://fintan.jp/page/392/

会津大学復興支援センターICT人材育成事業で実施。

- **SPA側**: React + OpenAPI + モックサーバ構築
- **API側**: Nablarch RESTful Webサービス + レイヤードアーキテクチャ
- テスト自動化: JUnit + OpenAPIドキュメント検証
- 参加者: 学部1年〜修士2年

---

## 5. 導入事例・ベストプラクティス

### 5.1 導入実績

Fintan上では**具体的な顧客名やプロジェクト詳細を記載した導入事例ページは存在しない**。

ただし以下の情報が確認できる：

- 「大小さまざまなシステムの開発・運用の実績がある」（https://fintan.jp/en/page/1954/）
- 「TISの金融業界のミッションクリティカルシステム構築の豊富な経験」（https://fintan.jp/en/page/1660/）
- 「多数の導入実績」があることは複数ページで言及

### 5.2 Nablarch自体の開発プロセス

**URL**: https://fintan.jp/page/1590/

Nablarchフレームワーク自体がどのように開発されているかの紹介。

#### ソースコード管理

- GitHub上で3ブランチ体系: `master`（リリース済み最新）→ `develop`（次リリース）→ `feature`（個別修正）
- プルリクエスト + レビュー → developへマージ

#### テスト体制

- **自動テスト**: JUnit + Concourse CI、失敗時はSlack通知
- **品質管理**: SonarQubeでカバレッジ・静的解析を集計
- **手動テスト**: Exampleアプリで機能検証
- **性能テスト**: Apache JMeterで実施（必要時）

#### ドキュメント管理

- reStructuredText形式で記述
- Sphinx（Python）でHTML変換
- GitHubでソースコードと同様に管理

#### 重要な設計方針

- **後方互換性の維持に特に重点**を置く
- 修正は既存の動作に影響を与えないよう細心の注意を払って検討
- サードパーティライブラリ使用時は有識者レビュー必須

### 5.3 UniversalDao（DB操作のベストプラクティス）

**URL**: https://fintan.jp/page/1587/

NablarchはDBアクセスにJDBCラッパーとUniversalDaoの2種類を提供。**UniversalDaoを推奨**。

#### 推奨方針

「UniversalDaoでできることはUniversalDaoを使い、できないことだけJDBCラッパーを使う」

#### UniversalDaoの主要機能

| 機能 | API例 |
|------|-------|
| 登録 | `UniversalDao.insert(entity)` |
| 一括登録 | `UniversalDao.batchInsert(entities)` |
| 主キー検索 | `UniversalDao.findById(Class, id)` |
| 全件取得 | `UniversalDao.findAll(Class)` |
| SQL文実行 | `UniversalDao.findAllBySqlFile(Class, sqlId, condition)` |
| 件数取得 | `UniversalDao.countBySqlFile(Class, sqlId, condition)` |
| 存在確認 | `UniversalDao.exists(Class, sqlId, condition)` |
| 遅延ロード | `UniversalDao.defer().findAll(Class)` |
| ページング | `UniversalDao.per(10).page(3).findAll(Class)` |

#### JDBCラッパーが必要な場面

- 主キーやバージョン番号以外の条件でのUPDATE/DELETE

---

## 6. 開発ツール・環境構築ガイド

### 6.1 ブランクプロジェクト

Nablarchのアーキタイプから生成する初期プロジェクト。疎通確認機能付き。

- ウェブアプリケーション用
- RESTful Webサービス用
- バッチアプリケーション用

**参照**: https://fintan.jp/page/1868/4/

### 6.2 Nablarchトレーニングコンテンツ

**URL**: https://fintan.jp/page/259/

テキストとハンズオンの2部構成。

#### テキスト構成

| トピック | 学習時間 |
|---------|---------|
| Nablarch概要 | 約20分 |
| ウェブアプリケーション | 約30分 |
| Nablarchバッチアプリケーション | 約30分 |
| RESTfulウェブサービス | 約15分 |
| JSR352準拠バッチアプリケーション | 約15分 |
| 設定・機能拡張編 | 約2時間 |

#### ハンズオン

- **15個のコンテンツ**
- 総学習時間: 約15時間（準備含む）
- レベル: 入門（30分〜1時間）、基本（1〜2時間）
- 実際にコードを記述しながら学習

### 6.3 Collaborage（チーム開発環境テンプレート）

クラウド上のチーム開発環境テンプレート。

- Redmine（課題管理）
- GitBucket（ソースコード管理）
- Jenkins（CI/CD）

### 6.4 DBA業務支援ツール

ルーチンDB作業を自動化するツール群。

### 6.5 開発プロセス支援ツール

ソースコードおよびドキュメントの自動生成ツール。

### 6.6 Nablarchの実行基盤

以下の実行基盤をサポート：

| 実行基盤 | 概要 |
|---------|------|
| ウェブアプリケーション | サーバーサイドHTML生成 |
| ウェブサービス（RESTful） | REST API |
| バッチアプリケーション | Nablarch独自バッチ |
| JSR352準拠バッチ | Java Batch標準準拠 |
| MOMメッセージング | Message Oriented Middleware方式 |
| テーブルキューメッセージング | テーブルをキューとして使用 |

---

## 7. コーディング規約・静的解析

### 7.1 Coding Standards（コーディング規約）

**URL**: https://fintan.jp/en/page/5986/
**GitHub**: https://github.com/Fintan-contents/coding-standards/tree/main/en

Java等のコーディング規約と静的解析ツール導入ガイドを提供。

- **対象言語**: Java（主）、その他
- **静的解析ツール**: Checkstyle、SpotBugs
- **著者**: Nablarch Team / TIS Development Platform Center
- **公開日**: 2022-11-02

### 7.2 Nablarchシステム開発ガイドでの記述

**URL**: https://fintan.jp/page/252/9/

#### Javaコーディング規約
- バグ防止と品質向上を目的
- GitHubで管理される標準資料

#### 静的検査ツール

| ツール | 目的 |
|-------|------|
| **Checkstyle** | 機械的に検出できる規約違反を解消。Checkstyleガイドとして実装手順を提供 |
| **SpotBugs** | 明らかに問題のあるコード、後々問題が発生しそうなコードを排除 |
| **使用不許可APIチェックツール** | 必要に応じてAPIを制限（セキュリティ向上） |

#### コードフォーマッター
- すぐに使えるフォーマッターを提供
- フォーマッター適用は規約遵守の前提条件

### 7.3 プログラマー自己チェックリスト

**URL**: https://fintan.jp/en/page/1721/

開発者がレビュー前に自己検証するためのチェックリスト。

- **チェック対象**: Javaコーディング規約、SQL、テスト手順、UI実装
- **構成**: 約30%がNablarch固有、約70%がJava汎用
- **カスタマイズ可能**: 他フレームワーク使用時はプロジェクトに合わせて調整
- **提供形式**: Excel(.xlsx) / PDF
- **ライセンス**: CC BY-SA 4.0

---

## 8. テスト関連コンテンツ

### 8.1 Nablarch Testing Framework

**URL**: https://fintan.jp/en/page/1662/
**ドキュメント**: https://nablarch.github.io/docs/LATEST/doc/en/

JUnitベースの自動テスティングフレームワーク。

- **特徴**: テストケースとテスト結果をExcelで定義可能
- **ライセンス**: Apache License 2.0

### 8.2 テスト種別＆テスト観点カタログ

**URL**: https://fintan.jp/page/1456/
**GitHub**: https://github.com/Fintan-contents/testing-viewpoint

Fintanのテスト関連コンテンツの**中核ドキュメント**。

#### テスト種別
- 機能テスト、性能テスト、セキュリティテスト等をアプリケーション検証の目的で分類

#### テスト観点
- 検出したい不具合や検証対象を整理し、テストケース検討のベースとなる観点を提供

#### 対応アプリケーション種別
- ウェブアプリケーション
- モバイルアプリケーション
- バッチアプリケーション
- メッセージング
- ウェブサービス

#### 提供ファイル

| ファイル | 版数 | 更新日 |
|---------|------|--------|
| テスト種別&観点カタログガイド | 第1.2版 | 2018/08/16 |
| テスト種別カタログ | 第1.3版 | 2022/03/31 |
| テスト観点カタログ | 第1.6版 | 2025/04/04 |

### 8.3 全体テスト計画ガイド

**URL**: https://fintan.jp/page/1458/6/ / https://fintan.jp/en/page/1683/

テスト計画の属人化軽減を目的とした包括ガイド。9つの検討領域をカバー：

1. テスト方針
2. テストフェーズの位置づけ
3. カバレッジ方針
4. テスト範囲
5. テスト環境
6. チーム構成
7. タスクと役割分担
8. スケジュール
9. 管理方針

**ライセンス**: CC BY-SA 4.0

### 8.4 性能テスト計画ガイド

**URL**: https://fintan.jp/page/2534/

性能テスト計画で検討すべきトピックを解説。全体テスト計画ガイド、テスト種別＆観点カタログ、非機能要求グレードを前提として利用。

### 8.5 障害テスト計画ガイド＆サンプル

- **ガイド**: https://fintan.jp/page/6394/
- **サンプル**: https://fintan.jp/page/6371/

障害テスト計画の属人化回避を目的。テスト種別＆観点カタログの定義に準拠。

### 8.6 生成AIを活用したテスト仕様書作成の自動化

**URL**: https://fintan.jp/page/15459/

#### 手法
1. 設計書をMarkdownに変換
2. 設計書の解析・関連情報収集
3. テスト観点の必要性判断
4. テストすべき要素の抽出
5. テストケース生成

#### 使用AI
- AWS Bedrock の Claude Sonnet / Haiku

#### 成果
- **判断精度**: 88〜91%
- **テスト仕様品質**: 51〜67%が高品質
- **工数削減**: 約40%（14.75時間 → 8.50時間）
- **レビュー工数**: 4時間 → 0.5時間
- **費用**: $11〜$20/機能

#### 特筆事項
- Fintanのテスト観点カタログ（100以上の観点）を入力として使用
- TIS社内のシステム開発で実際に活用

---

## 9. セキュリティ関連

### 9.1 Nablarchのセキュリティ対応

- JPCERT/CCと連携した脆弱性情報の収集・対応
- 脆弱性を埋め込まない品質確保

**参照**: https://fintan.jp/en/page/1954/

### 9.2 Nablarch機能セキュリティマトリクス

Nablarchシステム開発ガイド内で提供。

- IPAの「安全なウェブサイトの作り方」のセキュリティ実装チェックリスト項目
- Nablarchの機能でカバーできる項目を確認するためのマトリクス
- OWASP Top 10も補助的に参照

**参照**: https://fintan.jp/en/page/1667/

### 9.3 CSRF対策（SPA + REST API）

SPA + REST APIリファレンスで詳細なCSRF対策チュートリアルを提供。

- `CsrfTokenUtil` + `CsrfTokenVerificationHandler`（バックエンド）
- React `useEffect` でのトークン初期化（フロントエンド）

**参照**: https://fintan.jp/page/428/

### 9.4 Find Security Bugs

バックエンドエンジニア向けコンテンツとして「Find Security Bugs によるセキュリティ対策レポート」が提供されている。

**参照**: https://fintan.jp/for-backend-engineer/

---

## 10. TIS関連・Nablarch以外のコンテンツ

### 10.1 Java/Jakarta EE関連

| コンテンツ | URL | 概要 |
|----------|-----|------|
| Java FW & 設計開発支援コンテンツ | https://fintan.jp/en/page/6004/ | Nablarch/Spring FW依存と非依存コンテンツの整理 |
| アプリケーションアーキテクチャ学習コンテンツ | https://fintan.jp/for-architect/ 内 | エンタープライズシステム基礎知識 |
| クラウドネイティブ・アプリケーション開発 | https://fintan.jp/for-architect/ 内 | クラウドネイティブの基本概念 |

#### フレームワーク依存/非依存の区分

| 区分 | コンテンツ |
|------|----------|
| **Nablarch依存** | ランタイム、テスティングFW、機能拡張、実装例、マニュアル、プログラマーズガイド、TIPS、開発標準 |
| **Spring依存** | 機能拡張、サンプルプロジェクト、プログラマーズガイド、TIPS |
| **FW非依存** | 設計サンプル、コーディング規約、要件定義、テストカタログ、全体テスト計画 |

### 10.2 CI/CD関連

| コンテンツ | URL | 概要 |
|----------|-----|------|
| サービス開発の進め方 | https://fintan.jp/page/10602/ | スクラム+CI/CD、GitLab Flow、静的解析、脆弱性管理 |
| AWS CI/CD構成例 | https://fintan.jp/page/1550/ | CodeCommit→CodeBuild→CodeDeploy→CodePipeline |
| DevOps環境構築キット Epona | https://fintan.jp/for-backend-engineer/ 内 | サービス迅速立ち上げ用ツールキット |

#### サービス開発の進め方（詳細）

- **プロセス**: スクラム
- **ブランチ戦略**: GitLab Flow
- **CI/CD**: GitLab CI/CD（静的解析含む）
- **静的解析**: ソースコード・設定ファイルの早期エラー検出
- **脆弱性管理**: npm-audit、OWASP Dependency-Check
- **品質**: リリースごとに品質目標と担保戦略を策定

#### AWS CI/CD構成例（詳細）

| AWS サービス | 役割 |
|-------------|------|
| CodeCommit | Gitリポジトリ |
| CodeBuild | ビルド＋テスト（VPC内配置でRDS接続） |
| CodeDeploy | EC2へのローリングデプロイ |
| CodePipeline | パイプライン統合管理 |

- REST API: EC2 + ELB
- SPA: S3 + CloudFront

### 10.3 要件定義関連

| コンテンツ | URL | 概要 |
|----------|-----|------|
| 要件定義基礎研修テキスト | https://fintan.jp/page/1674/ | 要件定義の基礎知識を効率的に学べる研修テキスト |
| 要件定義フレームワーク | PDF | 非機能要件定義のメトリクス提供 |

#### 要件定義基礎研修テキスト構成

| セクション | 内容 |
|----------|------|
| 要件定義概論 | 一般的な知識を解説 |
| 要件定義計画 | 進め方と注意点 |
| 業務要件定義 | 進め方と注意点 |
| システム要件定義 | 進め方と注意点 |

### 10.4 その他注目コンテンツ

| コンテンツ | 概要 |
|----------|------|
| アジャイルとDevOpsの学習コンテンツ | アジリティをもったサービス提供 |
| Lerna 高可用ソフトウェアスタック | アクターモデルベースの高可用システム |
| Architecture Decision Records導入事例 | スクラム開発でのADR活用 |
| モバイルアプリケーション開発ノウハウ | SIerのモバイル開発知見 |

---

## 11. ライセンス・利用規約

### 11.1 Fintanコンテンツのライセンス体系

| コンテンツ種別 | ライセンス | 許可される利用 |
|-------------|----------|--------------|
| **ドキュメント（標準）** | Creative Commons 表示 4.0 国際 | 使用・複製・改変・再配布（表示義務あり） |
| **コードサンプル** | Apache License 2.0 | 使用・複製・改変・翻案・再配布 |
| **開発標準** | CC BY-SA 4.0（表示-継承） | 使用・複製・改変・再配布（継承義務あり） |
| **一部コンテンツ** | Fintan コンテンツ 使用許諾条項 | 個別条項に従う |

**注**: 「注があるものを除いて」の記述から、例外的なライセンス規定が存在する可能性あり。

### 11.2 Nablarchコンテンツのライセンス

| コンテンツ | ライセンス |
|----------|----------|
| アプリケーションフレームワーク | Apache License 2.0 |
| 実装サンプル集 | Apache License 2.0 |
| テスティングフレームワーク | Apache License 2.0 |
| 開発標準 | CC BY-SA 4.0 |

**参照**: https://fintan.jp/wp-content/uploads/2021/12/nablarch-license-pricing-structure.pdf
※ PDFの詳細内容はバイナリ解析不可のため完全抽出できず。

### 11.3 商標情報

**URL**: https://fintan.jp/en/page/1891/

- Nablarchは商標登録一覧には**含まれていない**
- ®マークやTMマークは省略される場合がある
- 第三者の商標は識別目的での参照のみ

### 11.4 RAG構築時の留意事項

- ドキュメント（CC BY 4.0 / CC BY-SA 4.0）: **RAGのナレッジベースとして利用可能**（表示義務あり、SA付きは派生物にも同ライセンス適用）
- コードサンプル（Apache 2.0）: **利用可能**（著作権表示・ライセンス表示義務あり）
- 「Fintan コンテンツ 使用許諾条項」適用コンテンツ: **個別確認が必要**
- 有償サポート対象コンテンツ: 非公開部分は利用不可

---

## 12. GitHub関連リポジトリ一覧

| リポジトリ | URL | 内容 |
|----------|-----|------|
| Nablarch本体 | https://github.com/nablarch/nablarch | フレームワーク本体 |
| Nablarch開発標準 | https://github.com/nablarch-development-standards/nablarch-development-standards/ | 開発標準ドキュメント |
| コーディング規約 | https://github.com/Fintan-contents/coding-standards | Java等のコーディング規約 |
| テスト観点カタログ | https://github.com/Fintan-contents/testing-viewpoint | テスト種別＆観点カタログ |
| Nablarchドキュメント | https://nablarch.github.io/docs/LATEST/doc/en/ | 公式ドキュメント |
| リリースノート | https://nablarch.github.io/docs/LATEST/doc/releases/index.html | リリースノート（日本語のみ） |

---

## 付録: 調査でアクセスできなかったリソース

| リソース | 理由 |
|---------|------|
| Nablarchライセンス・価格体系PDF | PDFバイナリのため詳細テキスト抽出不可 |
| Fintan コンテンツ 使用許諾条項（全文） | 直接URLが不明 |
| 有償サポートの詳細 | 非公開（TIS社への問い合わせが必要） |
