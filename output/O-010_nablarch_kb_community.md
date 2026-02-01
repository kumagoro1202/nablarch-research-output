# Nablarch コミュニティ・ユーザー情報調査レポート

> **作成日**: 2026-02-01
> **タスクID**: subtask_020 (parent: cmd_007)
> **プロジェクト**: nablarch_strategy
> **作成者**: 足軽5号（ペルソナ: マーケットリサーチャー）

---

## 目次

1. [Qiita上のNablarch記事](#1-qiita上のnablarch記事)
2. [Zenn上のNablarch記事](#2-zenn上のnablarch記事)
3. [その他技術ブログ](#3-その他技術ブログ)
4. [開発者の評価・フィードバック](#4-開発者の評価フィードバック)
5. [導入企業・業界の傾向](#5-導入企業業界の傾向)
6. [求人情報からの傾向分析](#6-求人情報からの傾向分析)
7. [GitHub・OSSコミュニティ活動](#7-githubossコミュニティ活動)
8. [セキュリティ・脆弱性情報](#8-セキュリティ脆弱性情報)
9. [Fintan（TIS技術共有サイト）のコンテンツ](#9-fintantis技術共有サイトのコンテンツ)
10. [良い点/課題点サマリ](#10-良い点課題点サマリ)

---

## 1. Qiita上のNablarch記事

Qiitaは日本のNablarchコミュニティにおける主要な情報発信プラットフォームである。確認できた記事を時系列順にリストアップする。

### 記事一覧

| # | タイトル | 投稿日 | カテゴリ | URL |
|---|---------|--------|---------|-----|
| 1 | github上のnablarch-exampleプロジェクトをEclipseに取り込む | 2017-03 | 入門 | [Qiita](https://qiita.com/KO_YAmajun/items/1014d4ea899a1164463c) |
| 2 | Javaのフレームワーク Nablarch を使ってみる [ウェブアプリケーション編] | 2017-01 | 入門 | [Qiita](https://qiita.com/heiwa/items/0a09322e93cb3bfe32cb) |
| 3 | Nablarch（RESTfulウェブサービス）で、データベースを使わないプロジェクトを作る | 2021-01 | Tips | [Qiita](https://qiita.com/charon/items/967f878bde5096693ffc) |
| 4 | Nablarchを学ぶ前に知っておくべき知識について | 2022-04 | 入門 | [Qiita](https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9) |
| 5 | Nablarchを使ったWebアプリケーションの作成 | 2022-04 | 入門 | [Qiita](https://qiita.com/kirin1218/items/c537cb8a444ee59763cd) |
| 6 | Nablarchを使ったバッチアプリケーションの作成 | 2022-04 | 入門 | [Qiita](https://qiita.com/kirin1218/items/eb3033ecb1520b497cc2) |
| 7 | Nablarchを使ったバッチアプリケーションの作成-その２- | 2022-04 | 解説 | [Qiita](https://qiita.com/kirin1218/items/ba6df84e07e96759715f) |
| 8 | Nablarchを使ってみよう | 2022-05 | レビュー | [Qiita](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 9 | Nablarch: writeタグとrawWriteタグの違いは？ | 2024-01 | Tips | [Qiita](https://qiita.com/KO_YAmajun/items/51773c6a42875dc51135) |
| 10 | Nablarch 6のRESTfulウェブサービスのExampleをOpen Liberty＋PostgreSQLを使うように変更 | 2024-06 | Tips | [Qiita](https://qiita.com/charon/items/2c14ab849d3edfd9fe61) |
| 11 | Nablarch: MySQL＆TiDB用のDialectを作成する | 2024-09 (更新2025-02) | Tips | [Qiita](https://qiita.com/KO_YAmajun/items/3433275e930be9a56270) |
| 12 | [5u25リリース記念] nablarch-micrometer-otlpでexample-restアプリの性能測定 | 2024-09 (更新2025-03) | 新機能 | [Qiita](https://qiita.com/KO_YAmajun/items/191ba0b148a61d6e9525) |
| 13 | Nablarchのブランクプロジェクトを作成した後に、使用するデータベースを切り替えるスクリプトを書く | 2024-10 | Tips/自動化 | [Qiita](https://qiita.com/charon/items/43b986e9614aaecd0ac7) |
| 14 | Fintan・Nablarchコンテンツリスト | 2025-01 | まとめ | [Qiita](https://qiita.com/KO_YAmajun/items/3f071e02444124f7ef07) |

### 記事の傾向分析

- **入門・チュートリアル系**: 最も多い（約50%）。Webアプリ、バッチ、REST各方式の入門記事
- **Tips・トラブルシューティング系**: DB切替、ハンドラ設定変更、タグ使い分けなど実務的な知見（約30%）
- **レビュー・所感系**: メリット/デメリットを整理した記事（約10%）
- **新機能紹介**: OpenTelemetry対応など最新機能の紹介（約10%）

### 主要な投稿者

- **KO_YAmajun**: TIS関係者と推測。Fintan/Nablarch公式コンテンツのキュレーション、最新機能紹介を担当。最も活発（6記事）
- **kirin1218**: Nablarch入門シリーズを体系的に執筆（5記事）。率直な評価で参考価値が高い
- **charon**: REST/DB周りの実践的なTips記事（3記事）
- **heiwa**: 初期のNablarch入門記事（1記事、2017年）

### いいね数

全体的にいいね数は少なめ（0〜4程度）。最も評価の高い記事は「Nablarchを使ってみよう」（いいね4）。Nablarchのニッチな性質を反映している。

---

## 2. Zenn上のNablarch記事

### 調査結果

**Zenn上にはNablarch専門の記事はほぼ存在しない。**

`site:zenn.dev Nablarch` で検索した結果、Nablarchを主題とした記事は見つからなかった。唯一、JUnitのテストに関する記事内でNablarch公式ドキュメントが参照として引用されている程度である。

- [Repositoryクラス、ServiceクラスのJUnitを使った単体テストについて](https://zenn.dev/monaka0309/articles/4cc135396a7ae2) — Nablarchテストフレームワークのドキュメントを参照

### 考察

Zennは比較的新しい技術プラットフォーム（2020年〜）であり、モダンな技術スタックの記事が多い。Nablarchのようなエンタープライズ向けフレームワークの記事がほぼ存在しないことは、Nablarchユーザー層とZennユーザー層の乖離を示している。

---

## 3. その他技術ブログ

### 3.1 個人技術ブログ

#### 「Nablarch 所感」 — 発火後忘失（はてなブログ系）【アクセス不可】
- **URL**: ~~https://himeji-cs.jp/blog/2018/02/02/nablarch/~~ （2026-02時点でドメイン消失、アクセス不可）
- **投稿日**: 2018-02-02
- **内容**: Nablarchを実際に試用した技術的に深い所感（調査時点で取得した要約を以下に記載）

**主な指摘事項:**（以下はNablarch公式ドキュメント・アーキテクチャKBでも裏付け可能な技術的事実）
- 設定がアノテーションではなくXMLファイルの外部読み込みで行われる（※公式ドキュメントで確認可: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/setting_guide/CustomizeComponentDefinition.html）
- DIを使いたい場合にフレームワークの修正が必要になる可能性（※Nablarchのコンポーネント定義はsetter injectionのみ対応）
- `ExecutionContext`型のコンテキストについて「FWを使う側からすると誰がどのタイミングで設定したかよくわからない値が入ったグローバル変数的な側面も持つ」と指摘
- Ease of Development（開発容易性）の達成について懐疑的

### 3.2 企業技術ブログ

#### 「Nablarchとは何か：エンタープライズシステム向けフレームワークの概要と特長」 — 株式会社一創
- **URL**: https://www.issoh.co.jp/tech/details/3384/
- **投稿日**: 2024-08頃
- **内容**: Nablarchの包括的な解説記事

**主な内容:**
- SpringやStrutsとの比較において、日本市場に特化している点を評価
- 金融業界、公共機関、製造業・流通業界での導入事例に言及
- 国内企業の要件に合わせた標準機能・テンプレートが豊富
- 導入時のカスタマイズ作業が最小限に抑えられる

#### CodeZine: TISのアプリケーション開発ノウハウを提供するWebサイト「Fintan」が公開
- **URL**: https://codezine.jp/article/detail/11139
- **投稿日**: 2018-10
- **内容**: Fintan公開のニュース記事。Nablarch開発標準や学習コンテンツの提供開始を報道

### 3.3 英語圏の情報

**Stack Overflow上にNablarchの質問・回答はほぼ存在しない。**

Nablarchは日本市場特化のフレームワークであり、英語圏のコミュニティ活動は事実上ない。GitHub上のドキュメントの一部が英語化されている（Nablarch 6u2以降）が、コミュニティの議論は日本語に限定されている。

### 3.4 note

note上にもNablarch専門の記事は確認できなかった。

---

## 4. 開発者の評価・フィードバック

### 4.1 良い点（ポジティブな評価）

| # | 評価内容 | 出典 |
|---|---------|------|
| 1 | サードパーティ利用を前提としていない開発者的な粋を感じる | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 2 | ライブラリを使い分けることで要件に合わせた柔軟な設計（DB、セッション管理等）が可能 | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 3 | 動くもの（ブランクプロジェクトやExample）がすぐ用意できるのは触ってみやすい | [Qiita heiwa](https://qiita.com/heiwa/items/0a09322e93cb3bfe32cb) |
| 4 | セキュリティ機能が標準装備。JPCERT/CC連携による脆弱性対応 | [TIS公式](https://www.tis.jp/service_solution/nablarch/) |
| 5 | 開発標準・ガイド・テストフレームワークまで一式提供 | [Fintan](https://fintan.jp/en/page/1954/) |
| 6 | 後方互換性が可能な限り維持される安定性 | [TIS公式](https://www.tis.jp/service_solution/nablarch/) |
| 7 | 有償契約継続でEOSL（サポート終了）が設定されない長期サポート | [TIS公式](https://www.tis.jp/service_solution/nablarch/) |
| 8 | 日本市場に特化しており国内の業務フローに適合しやすい | [一創](https://www.issoh.co.jp/tech/details/3384/) |

### 4.2 課題点・不満点（ネガティブな評価）

| # | 評価内容 | 出典 |
|---|---------|------|
| 1 | 「圧倒的、初心者殺しな情報のなさ。リファレンスがあるが使い方がわからない」 | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 2 | AWSやAzureとの親和性が弱い（対応ライブラリがなく自作必要） | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 3 | 最新の開発トレンドに乗り遅れている感（AWS、コンテナ、通信プロトコル等） | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 4 | 各ライブラリ仕様がガチガチでリッチな見た目のサイトには向かない | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 5 | 「お世辞にもわかりやすいフレームワークではない」 | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 6 | 公式の「未経験者でもすぐに開発を始められる」は「嘘だと思っている」 | [Qiita kirin1218](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17) |
| 7 | XML設定中心でアノテーションベースではない | 発火後忘失（アクセス不可）/ [Nablarch公式](https://nablarch.github.io/) で裏付け可 |
| 8 | ExecutionContextがグローバル変数的で誰がいつ設定したかわからない | 発火後忘失（アクセス不可）|
| 9 | DIを使いたい場合にFW修正が必要になる可能性 | 発火後忘失（アクセス不可）/ [Nablarch公式](https://nablarch.github.io/) で裏付け可 |
| 10 | 技術ドキュメントに難しい単語・技術が多く前提知識が必要 | [Qiita kirin1218](https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9) |

### 4.3 Springとの比較コメント

| 観点 | Nablarch | Spring Framework |
|------|----------|-----------------|
| 対象市場 | 日本国内エンタープライズ | グローバル |
| 開発標準 | 標準で豊富に提供 | 自前で整備が必要 |
| クラウド対応 | 弱い（ライブラリ不足） | 非常に強い（Spring Cloud等） |
| コミュニティ・情報量 | 少ない（Qiita中心で十数記事） | 非常に多い（世界規模） |
| セキュリティ | 標準装備・JPCERT連携 | Spring Security等で対応 |
| 最新トレンド追随 | 遅め | 積極的 |
| 長期保守サポート | 有償で手厚い（EOSL無し） | コミュニティ＋商用サポート |
| 開発プロセス | ウォーターフォール向き | アジャイル含め柔軟 |
| DIの利用 | XML設定中心 | アノテーション中心 |
| 求人数 | 極めて少ない | 非常に多い（レバテック: 1,130件） |

**注目点**: TIS自身もSpringとNablarchの両方を活用する立場を取っている（Fintan上にSpring向けコンテンツも提供）。

### 4.4 学習コストに関するコメント

- ドキュメントに専門用語が多く、DI等の前提知識が必要（[Qiita](https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9)）
- 独自のハンドラキューアーキテクチャの理解が必要
- ブランクプロジェクトは用意されているが、実際の開発ではカスタマイズが多くサクサクとはいかない（[Qiita](https://qiita.com/heiwa/items/0a09322e93cb3bfe32cb)）
- TIS公式は学習コンテンツ（15個のハンズオン教材）を整備しているが、コミュニティの自発的な情報発信は限定的

### 4.5 ドキュメントの品質に関するコメント

- 公式ドキュメントは体系的だが、難しい用語が多い
- 「リファレンスがあるが使い方がわからない」という声
- TISは開発ガイド、コーディング規約、UIスタンダードなど包括的なドキュメントを提供
- Nablarch 6u2以降、英語版ドキュメントも一部提供開始

---

## 5. 導入企業・業界の傾向

### 5.1 TISグループの基盤

Nablarchの母体であるTIS株式会社は、3,000社以上の顧客基盤を持つ大手SIerである。

**TISの主要実績:**
- クレジット取扱高主要25社のうち10社と取引
- 国内クレジットカードショッピング信用供与額: 年間54兆円
- カード会員数: 約1.7億人（取引先10社合計）
- クレジット取扱高: 全体の約50%

### 5.2 業界別の導入傾向

| 業界 | 利用傾向 | 具体的用途 |
|------|----------|-----------|
| **金融・クレジット** | 最も多い | 取引システム、カード基幹、ローン管理 |
| **公共・官公庁** | 多い | 危機管理情報システム、地方税電子申告 |
| **製造業** | 中程度 | 業務システム全般 |
| **流通業** | 中程度 | 業務システム全般 |
| **エネルギー（電気・ガス・水道）** | あり | 基幹系システム |

### 5.3 TISグループ以外での採用

**公開情報からは、TISグループ外の具体的な企業名による採用事例は確認できなかった。**

一創の記事では「金融業界、公共機関、製造業・流通業界での導入事例がある」と言及されているが、具体的な社名は非公開。Nablarchの利用はTISグループの受託開発案件が中心であると推測される。

### 5.4 案件の特徴

- 大規模基幹システム（数十〜数百人月規模）
- ウォーターフォール型開発が前提
- 長期保守が必要なシステム
- セキュリティ要件の高いシステム（金融、公共）
- 日本国内向けシステム（グローバル展開案件ではSpring等を利用）

---

## 6. 求人情報からの傾向分析

### 6.1 求人市場の現状

**「Nablarch」を指定した公開求人はほぼ存在しない。**

以下のプラットフォームで検索したが、Nablarch名指しの求人は見つからなかった:
- Indeed Japan
- レバテックフリーランス
- フリーランススタート
- Green
- Forkwell Jobs

### 6.2 推測される理由

1. **ニッチな市場**: Nablarchは主にTISグループ内の案件で使用されるため、一般の求人サイトに案件が公開されにくい
2. **非公開案件**: TISのパートナー企業やBP（ビジネスパートナー）向けに非公開で流通している可能性
3. **スキル表記**: 求人票では「Java」「SIer経験」として募集され、Nablarchは案件詳細でのみ言及されるケース

### 6.3 Java × SIer市場との関連

| 指標 | 値 | 出典 |
|------|-----|------|
| レバテックのJava案件数 | 3,496件 | フリーランススタート (2026-01) |
| レバテックのSpring案件数 | 1,130件 | フリーランススタート (2026-01) |
| Java平均月額単価 | 69万円 | レバテック |
| Java最高月額単価 | 145万円 | レバテック |
| SIer案件平均月収 | 67万円 | プロエンジニア |
| Nablarch単体の案件数 | 確認不可（ほぼ0） | 各求人サイト |

### 6.4 Nablarchエンジニアに求められるスキル

公式のシステム開発ガイドから推測されるスキルセット:
- Java開発経験（3年以上が望ましい）
- XML設定ファイルの理解
- ウォーターフォール型開発の経験
- DI（Dependency Injection）の理解
- RDBMSの知識
- Struts/SpringなどJavaフレームワーク経験（移行時に有利）

### 6.5 ライセンス・課金情報

Nablarchのフレームワーク自体はOSS（Apache-2.0）で無償だが、**有償サポート契約**がある:
- 課金対象: 本番用機器のCPUコア数ベース
- 開発用・テスト用・待機系は課金対象外
- 基本サービス: バグ修正提供、技術サポート
- オプションサービス: カスタム料金で強化サポート

---

## 7. GitHub・OSSコミュニティ活動

### 7.1 GitHub統計

| 指標 | 値 |
|------|-----|
| GitHubオーガニゼーション | [github.com/nablarch](https://github.com/nablarch) |
| 公開リポジトリ数 | 114 |
| メインリポジトリ Stars | 42 |
| メインリポジトリ Forks | 11 |
| オーガニゼーション作成日 | 2015-05-01 |
| 最終更新 | 2025-08-13 |
| 公開メンバー | 非公開（組織メンバーのみ閲覧可） |

### 7.2 関連オーガニゼーション

- **[nablarch](https://github.com/nablarch)**: フレームワーク本体（114リポジトリ）
- **[nablarch-development-standards](https://github.com/nablarch-development-standards)**: 開発標準
- **[Fintan-contents](https://github.com/Fintan-contents)**: 学習コンテンツ・ガイド

### 7.3 コミュニティの特徴

- **完全に企業主導**: TIS社の開発チーム（東京・大阪拠点）が開発・運営
- **外部コントリビューターの参加はほぼない**: オーガニゼーションメンバーが非公開、Issue/PR活動も限定的
- **Stars 42はJavaフレームワークとしては非常に少ない**（参考: Spring Boot ≈ 76k、Quarkus ≈ 14k）
- OSS化はしているが、コミュニティドリブンではなく企業ドリブンのOSS

---

## 8. セキュリティ・脆弱性情報

### 過去の脆弱性

| CVE ID | 種類 | CVSSv2 | 影響 | 対策版 | 発見日 |
|--------|------|--------|------|--------|--------|
| CVE-2019-5918 | XML外部実体参照（XXE）| 8.5（危険） | 情報漏洩・システム停止 | 5u14 | 2019-02 |
| CVE-2019-5919 | 暗号化の不備 | 中（Medium） | HIDDENストア情報の取得・改ざん | 5u14 | 2019-02 |

- **出典**: [Security NEXT](https://www.security-next.com/102898)、JVN（Japan Vulnerability Notes）
- IPAはCVE-2019-5918を3段階中最も深刻な「危険」に分類
- TIS開発チームがJPCERT/CCへ報告し、適切な情報開示を実施
- いずれも5u14で修正済み

### セキュリティへの取り組み

- 脆弱性を埋め込まない品質確保プロセス
- JPCERT/CCとの連携による脆弱性情報の収集・対応
- セキュリティハンドラ（SecureHandler、CsrfTokenVerificationHandler等）を標準搭載

---

## 9. Fintan（TIS技術共有サイト）のコンテンツ

### 概要

[Fintan](https://fintan.jp/)はTISが2018年10月に公開した技術共有サイト。Nablarchだけでなく、幅広い開発ノウハウを無償提供。

### Nablarch関連コンテンツ

| カテゴリ | 内容 |
|----------|------|
| **初心者向け** | ITアーキテクト概論、Webアプリケーション特有の仕組み、Nablarch概要 |
| **研修テキスト** | 要件定義基礎研修（概論、計画、業務・システム要件定義） |
| **学習コンテンツ** | アプリケーションアーキテクチャ（セッション管理、排他制御、システム間連携）、クラウドネイティブ、アジャイル/DevOps |
| **トレーニング** | Nablarchハンズオン15個（Web、バッチ、REST API）、Apache-2.0ライセンス |
| **ガイド** | Javaコーディング規約、ArchUnit静的解析ガイド、開発環境構築ガイド |
| **開発標準** | Nablarch開発標準（設計書テンプレート、UIスタンダード、コーディング規約） |
| **事例紹介** | 25件以上の実践事例（テスト自動化、パフォーマンスチューニング、マイグレーション等） |

### 特筆事項

- TIS社はNablarchだけでなくSpring向けのコンテンツも提供しており、フレームワーク中立的な立場
- Fintanのコンテンツは継続的に更新されている
- [Fintan・Nablarchコンテンツリスト（Qiita）](https://qiita.com/KO_YAmajun/items/3f071e02444124f7ef07) に網羅的なリンク集あり

---

## 10. 良い点/課題点サマリ

### 良い点サマリテーブル

| # | カテゴリ | 良い点 | 信頼度 |
|---|---------|--------|--------|
| 1 | 安定性 | 後方互換性を維持、EOSL無し（有償契約時） | 高（公式情報） |
| 2 | セキュリティ | 標準装備のセキュリティ機能、JPCERT/CC連携 | 高（公式情報） |
| 3 | 開発標準 | FW + 開発標準 + ガイド + テストFW一式提供 | 高（公式情報） |
| 4 | 品質統制 | 開発者ごとの成果物品質のばらつきを抑制 | 高（公式情報） |
| 5 | 導入容易性 | ブランクプロジェクトとExampleですぐ試せる | 中（ユーザー評価） |
| 6 | 柔軟性 | ライブラリ選択による要件対応 | 中（ユーザー評価） |
| 7 | 日本市場適合 | 国内業務フローに合致した標準機能 | 中（企業ブログ） |
| 8 | 学習教材 | 15のハンズオン教材をOSSで公開 | 高（公式情報） |

### 課題点サマリテーブル

| # | カテゴリ | 課題点 | 深刻度 |
|---|---------|--------|--------|
| 1 | 情報量 | コミュニティ情報が極端に少ない（Qiita十数記事、Zennほぼ0） | 高 |
| 2 | 学習コスト | ドキュメントが難解、前提知識が多い | 高 |
| 3 | クラウド対応 | AWS/Azure/GCPライブラリ不足 | 高 |
| 4 | トレンド追随 | コンテナ、マイクロサービス、モダンWeb対応が遅い | 高 |
| 5 | 人材市場 | 求人がほぼ存在せず、キャリアリスクが高い | 高 |
| 6 | 設定方式 | XML中心でアノテーションベースではない | 中 |
| 7 | UI制約 | リッチなUI構築に不向き | 中 |
| 8 | コミュニティ | 企業主導のOSSでコミュニティドリブンではない（GitHub Stars: 42） | 中 |
| 9 | 開発プロセス | ウォーターフォール前提でアジャイルに不向き | 中 |
| 10 | 英語対応 | 英語圏の情報・コミュニティがほぼない | 低（国内利用前提なら） |

---

## 付録: ソースURL一覧

### Qiita
- https://qiita.com/KO_YAmajun/items/1014d4ea899a1164463c
- https://qiita.com/heiwa/items/0a09322e93cb3bfe32cb
- https://qiita.com/charon/items/967f878bde5096693ffc
- https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9
- https://qiita.com/kirin1218/items/c537cb8a444ee59763cd
- https://qiita.com/kirin1218/items/eb3033ecb1520b497cc2
- https://qiita.com/kirin1218/items/ba6df84e07e96759715f
- https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17
- https://qiita.com/KO_YAmajun/items/51773c6a42875dc51135
- https://qiita.com/charon/items/2c14ab849d3edfd9fe61
- https://qiita.com/KO_YAmajun/items/3433275e930be9a56270
- https://qiita.com/KO_YAmajun/items/191ba0b148a61d6e9525
- https://qiita.com/charon/items/43b986e9614aaecd0ac7
- https://qiita.com/KO_YAmajun/items/3f071e02444124f7ef07

### ブログ・メディア
- ~~https://himeji-cs.jp/blog/2018/02/02/nablarch/~~ （2026-02時点でアクセス不可・ドメイン消失）
- https://www.issoh.co.jp/tech/details/3384/
- https://codezine.jp/article/detail/11139
- https://www.security-next.com/102898

### 公式
- https://www.tis.jp/service_solution/nablarch/
- https://nablarch.github.io/
- https://fintan.jp/en/page/1954/
- https://fintan.jp/en/page/1660/
- https://fintan.jp/en/page/1667/
- https://github.com/nablarch
- https://github.com/nablarch-development-standards
- https://github.com/Fintan-contents

---

*本レポートは足軽5号がマーケットリサーチャーとして調査・作成したものである。*
