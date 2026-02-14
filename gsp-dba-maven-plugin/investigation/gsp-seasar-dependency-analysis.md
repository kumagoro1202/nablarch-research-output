# gsp-dba-maven-plugin Seasar依存調査＋代替案レポート

**作成日**: 2026-02-10
**対象プロジェクト**: gsp-dba-maven-plugin v5.2.0

---

## 1. エグゼクティブサマリ

### 結論

gsp-dba-maven-pluginは**Seasar2に深く依存**しており、特に**S2JDBC-Gen（Entity生成エンジン）**への依存が最も深刻である。Seasar2は**2016年9月26日にEOL**を迎えて既に約10年が経過しており、セキュリティパッチや新機能追加は一切行われていない。

**推奨する移行方針**:

| 優先度 | 対象 | 推奨代替 | 工数 |
|--------|------|----------|------|
| 1（最優先） | S2 Frameworkユーティリティ | Apache Commons / 標準Java API | **小** |
| 2（重要） | S2JDBC-Gen Entity生成 | **jOOQ Code Generation** or **Hibernate Tools** | **大** |
| 3（中） | dicon設定ファイル生成 | FreeMarkerテンプレート書き換え（dicon廃止） | **中** |
| 4（低） | Doma連携（`org.seasar.doma`） | そのまま維持（Domaは**活発に開発中**、EOLではない） | **不要** |

**総合移行工数見積: 大**（S2JDBC-Genの置換がボトルネック）

### 主要な発見

1. **pom.xml上のSeasar依存は4アーティファクト**（s2-framework, s2-extension, s2-tiger, s2jdbc-gen）、全てバージョン2.4.46
2. **mainソース60ファイル中27ファイル（45%）がSeasarをインポート**しており、依存が広範
3. **最深部はS2JDBC-Genのコマンド実行パイプライン**。`GenerateEntityCommand`を継承し、`CommandInvoker`で実行する構造が根幹
4. **S2 Frameworkユーティリティ**（StringUtil, StatementUtil, ConnectionUtil等）は薄いラッパーであり、標準Java APIやApache Commonsで容易に置換可能
5. **Doma（`org.seasar.doma`）は別プロジェクト**として活発に開発中（最新 Doma 3.11.1）。EOLではなく、移行不要

---

## 2. Seasar依存一覧

### 2.1 pom.xml上の依存アーティファクト

| # | groupId | artifactId | version | scope | EOL | 用途 |
|---|---------|-----------|---------|-------|-----|------|
| 1 | org.seasar.container | **s2-framework** | 2.4.46 | compile | **2016/9/26 EOL** | ユーティリティクラス群（StringUtil, StatementUtil等） |
| 2 | org.seasar.container | **s2-extension** | 2.4.46 | compile | **2016/9/26 EOL** | JDBC拡張（ConnectionUtil, GenDialect等） |
| 3 | org.seasar.container | **s2-tiger** | 2.4.46 | compile | **2016/9/26 EOL** | Java5+アノテーション対応（CollectionsUtil, Maps, ReflectionUtil） |
| 4 | org.seasar.container | **s2jdbc-gen** | 2.4.46 | compile | **2016/9/26 EOL** | Entity生成エンジン（Command, Factory, Model, Meta, Dialect） |
| 5 | org.seasar.doma | **doma-core** | 2.62.0 | provided | **活発開発中** | Domaアノテーション（Entity, Column, Id等）※Doma対応Entity生成用 |
| 6 | net.unit8.solr | **solr-jdbc-s2jdbc** | 0.2.0 | optional(solrプロファイル) | 不明 | Solr連携（オプション機能） |

**Seasar Maven Repository**: `https://maven.seasar.org/maven2` が`<repositories>`に定義されている。

### 2.2 使用しているSeasar APIのカテゴリ分類

#### カテゴリA: S2JDBC-Gen（Entity生成エンジン）— 影響度: 最大

| API | 使用ファイル | 用途 |
|-----|-------------|------|
| `o.s.e.jdbc.gen.internal.command.GenerateEntityCommand` | ExtendedGenerateEntityCommand | Entity生成コマンド（**継承**） |
| `o.s.e.jdbc.gen.command.CommandInvoker` | GenerateEntity, GenerateService | コマンド実行 |
| `o.s.e.jdbc.gen.internal.command.CommandInvokerImpl` | GenerateEntity, GenerateService | コマンド実行実装 |
| `o.s.e.jdbc.gen.internal.factory.FactoryImpl` | GspFactoryImpl, DomaGspFactoryImpl | ファクトリ（**継承**） |
| `o.s.e.jdbc.gen.desc.*` | GspFactoryImpl, GspAttributeDescFactoryImpl等 | Entityの属性記述子 |
| `o.s.e.jdbc.gen.model.*` | GspFactoryImpl, DomaGspFactoryImpl等 | Entityモデル生成 |
| `o.s.e.jdbc.gen.meta.*` | DbTableMetaReaderWithView, GspAttributeDescFactoryImpl, Dialect等 | DBメタデータ読取 |
| `o.s.e.jdbc.gen.dialect.*` | 各ExtendedGenDialect, 各Dialect | DB方言定義（**継承**） |
| `o.s.e.jdbc.gen.internal.command.GenerateNamesCommand` | GenerateService | Names生成 |
| `o.s.e.jdbc.gen.internal.command.GenerateServiceCommand` | GenerateService | Service生成 |
| `o.s.e.jdbc.gen.internal.util.ReflectUtil` | GenerateEntity, GenerateService | リフレクション |
| `o.s.framework.convention.PersistenceConvention` | GspFactoryImpl, GspAttributeDescFactoryImpl等 | 永続化命名規約 |

**影響範囲**: `s2jdbc/gen/`パッケージ7ファイル + `mojo/`3ファイル + `dialect/`8ファイル = **計18ファイル**

#### カテゴリB: S2 Frameworkユーティリティ — 影響度: 小

| API | 使用ファイル数 | 用途 | 代替 |
|-----|-------------|------|------|
| `o.s.framework.util.StringUtil` | 7 | isBlank, equals等 | Apache Commons Lang `StringUtils` / `String.isBlank()` (Java 11+) |
| `o.s.framework.util.StatementUtil` | 8 | Statement.close() | try-with-resources |
| `o.s.framework.util.ResultSetUtil` | 4 | ResultSet.close() | try-with-resources |
| `o.s.framework.util.DriverManagerUtil` | 4 | Driver登録 | `Class.forName()` / JDBC 4.0自動登録 |
| `o.s.framework.util.FileOutputStreamUtil` | 2 | ファイル出力 | `java.nio.file.Files` |
| `o.s.framework.util.FileInputStreamUtil` | 1 | ファイル入力 | `java.nio.file.Files` |
| `o.s.framework.util.OutputStreamUtil` | 1 | ストリーム出力 | 標準API |
| `o.s.framework.util.JarFileUtil` | 1 | Jar操作 | `java.util.jar.JarFile` |
| `o.s.framework.util.ClassUtil` | 1 | クラスロード | `Class.forName()` |
| `o.s.framework.util.MethodUtil` | 1 | メソッド呼出 | `java.lang.reflect.Method` |
| `o.s.framework.util.ArrayMap` | 1 | 順序付きMap | `java.util.LinkedHashMap` |
| `o.s.framework.exception.SQLRuntimeException` | 1 | SQL例外ラッパー | 自前実装 or `UncheckedSQLException` |

#### カテゴリC: S2 Extension JDBC — 影響度: 中

| API | 使用ファイル数 | 用途 | 代替 |
|-----|-------------|------|------|
| `o.s.extension.jdbc.util.ConnectionUtil` | 7 | Connection.close() | try-with-resources |
| `o.s.extension.jdbc.gen.dialect.GenDialectRegistry` | 6 | GenDialect登録 | S2JDBC-Gen代替に含む |

#### カテゴリD: S2 Tiger拡張 — 影響度: 小

| API | 使用ファイル数 | 用途 | 代替 |
|-----|-------------|------|------|
| `o.s.framework.util.tiger.CollectionsUtil` | 2 | `newArrayList()` | `new ArrayList<>()` |
| `o.s.framework.util.tiger.Maps` | 2 | Map初期化 | `Map.of()` (Java 9+) |
| `o.s.framework.util.tiger.ReflectionUtil` | 1 | リフレクション | `java.lang.reflect.*` |

#### カテゴリE: Doma（活発開発中）— 移行不要

| API | 使用ファイル数 | 用途 |
|-----|-------------|------|
| `o.s.doma.Column`, `Entity`, `Id`, `Table`等 | 1 (DomaGspFactoryImpl) | Doma Entity生成用アノテーション参照 |

Domaは`org.seasar.doma`のグループIDだが、**Seasar2とは独立したプロジェクト**。最新版はDoma 3.11.1で活発に開発中。

### 2.3 dicon設定ファイル（S2Container DI設定）

| ファイル | 用途 |
|---------|------|
| `template/dicon/convention.dicon.ftl` | S2Container命名規約設定テンプレート |
| `template/dicon/jdbc.dicon.ftl` | S2Container JDBC設定テンプレート |
| `template/dicon/s2jdbc.dicon.ftl` | S2JDBC設定テンプレート |
| `src/test/resources/s2junit4.dicon` | テスト用S2JUnit4設定 |

これらはFreeMarkerテンプレートから実行時に生成され、S2JDBC-Genの`GenerateEntityCommand`が内部的に読み込む。

---

## 3. 代替ライブラリ調査結果

### 3.1 Entity生成エンジン（S2JDBC-Gen代替）

| 候補 | 最新バージョン | ライセンス | GitHub Stars | 特徴 | 評価 |
|------|-------------|-----------|-------------|------|------|
| **jOOQ Code Generation** | 3.20.11 (OSS Ed.) | Apache 2.0 (OSS) | 6.3k+ | DB→型安全Javaコード生成。Maven Plugin完備。6DB対応。非常に活発 | **★★★★★** |
| **Hibernate Tools (hbm2java)** | 7.2.3.Final | LGPL 2.1 | (Hibernate本体に含む) | DB→JPA Entity生成。Maven Plugin(`hibernate-tools-maven`)完備 | **★★★★☆** |
| **MyBatis Generator** | 1.4.2 | Apache 2.0 | 5.4k+ | DB→MyBatis Mapper+Model生成。Maven Plugin完備 | **★★★☆☆** |
| **Doma CodeGen** | (Doma 3.x) | Apache 2.0 | 400+ | アノテーションプロセッサベース。コンパイル時コード生成 | **★★★☆☆** |
| **jpa-entity-generator** | 0.99.8 | MIT | 300+ | DB→JPA Entity (Lombok対応)。Maven/Gradle | **★★☆☆☆** |
| **Gentity** | - | Apache 2.0 | 少数 | DBSchema(.dbs)→JPA Entity | **★☆☆☆☆** |

### 3.2 ユーティリティ置換マッピング

| Seasar API | 推奨代替 | 備考 |
|-----------|---------|------|
| `StringUtil.isBlank()` | `String.isBlank()` (Java 11+) / `StringUtils.isBlank()` (Commons Lang) | プロジェクトは既にCommons Lang 2.6を使用 |
| `StringUtil.equals()` | `Objects.equals()` | 標準API |
| `StatementUtil.close()` | try-with-resources | Java 7+ |
| `ResultSetUtil.close()` | try-with-resources | Java 7+ |
| `ConnectionUtil.close()` | try-with-resources | Java 7+ |
| `DriverManagerUtil.registerDriver()` | JDBC 4.0 auto-loading / `Class.forName()` | Java 6+ |
| `FileOutputStreamUtil` / `FileInputStreamUtil` | `java.nio.file.Files` | Java 7+ |
| `CollectionsUtil.newArrayList()` | `new ArrayList<>()` | Diamond operator |
| `Maps.map()` | `Map.of()` / `Map.ofEntries()` | Java 9+ |
| `ArrayMap` | `LinkedHashMap` | 標準API |
| `JarFileUtil` | `java.util.jar.JarFile` | 標準API |
| `ClassUtil.forName()` | `Class.forName()` | 標準API |
| `ReflectUtil.newInstance()` | `Constructor.newInstance()` | 標準API |
| `SQLRuntimeException` | カスタム `UncheckedSQLException` | 自前実装（数行） |

### 3.3 DB方言レジストリ（GenDialectRegistry代替）

| 候補 | 対応DB | 特徴 |
|------|--------|------|
| jOOQ SQLDialect | 30+ DB | 最も網羅的 |
| Hibernate Dialect | 20+ DB | JPA標準準拠 |
| 自前実装（現行Dialectクラス拡張） | 6 DB | 既存コードベース活用 |

---

## 4. 移行影響分析

### 4.1 コード変更量の推定

| カテゴリ | 対象ファイル数 | 変更規模 | 難易度 |
|---------|-------------|---------|--------|
| **A: S2JDBC-Gen（Entity生成）** | 18ファイル（main） + 10ファイル（test） | 全面書き換え | **高** |
| **B: S2 Frameworkユーティリティ** | 15ファイル | 機械的置換 | **低** |
| **C: S2 Extension JDBC** | 7ファイル | try-with-resources化 | **低** |
| **D: S2 Tiger拡張** | 4ファイル | 機械的置換 | **低** |
| **E: dicon設定ファイル** | 4テンプレート | テンプレート廃止+代替設定 | **中** |
| **合計** | main 27ファイル + test 10ファイル + リソース4ファイル | - | - |

### 4.2 S2JDBC-Gen置換の最重要ポイント

**S2JDBC-Genは単なるライブラリ依存ではなく、gsp-dba-maven-pluginのEntity生成機能の根幹**である。以下の構造がS2JDBC-Genに密結合している:

```
GenerateEntity (Mojo)
  └→ ExtendedGenerateEntityCommand extends GenerateEntityCommand  ← S2JDBC-Gen
      └→ CommandInvoker / CommandInvokerImpl  ← S2JDBC-Gen
          └→ FactoryImpl → GspFactoryImpl / DomaGspFactoryImpl  ← S2JDBC-Gen継承
              ├→ EntityDescFactory  ← S2JDBC-Gen
              ├→ AttributeDescFactory  ← S2JDBC-Gen
              ├→ EntityModelFactory  ← S2JDBC-Gen
              ├→ DbTableMetaReader  ← S2JDBC-Gen
              └→ GenDialect (DB方言)  ← S2JDBC-Gen
```

この階層構造全体をS2JDBC-Genから切り離す必要がある。

### 4.3 2Way-SQLについて

gsp-dba-maven-pluginは**2Way-SQLを使用していない**。Seasar2の2Way-SQL機能（S2Dao/S2JDBC）はこのプロジェクトでは使われておらず、この点は移行の障害にならない。

### 4.4 テストへの影響

- **テストファイル29件**のうち、Seasarインポートを持つのは**10ファイル**
- `s2junit4.dicon`（S2JUnit4テスト設定）が存在するが、テストコードの大半はMaven Plugin Testing Harnessベース
- テストリソースの`expected/output/`配下にDoma Entity出力期待値があるが、これは生成コードのフォーマット変更に伴い更新が必要

### 4.5 ビルドシステム（Maven Plugin構成）への影響

- Maven Plugin自体の構造（Mojo, Parameter等）は変更不要
- pom.xmlからSeasar依存を除去し、代替ライブラリの依存を追加
- Seasar Maven Repository（`maven.seasar.org`）をリポジトリから除去可能（Domaは`mavenCentral`に存在）
- ただし、Doma 2.62.0が`provided`スコープで残るため、Domaを使うプロジェクトへの影響はない

### 4.6 下位互換性（既存ユーザーへの影響）

| 観点 | 影響 |
|------|------|
| **Mavenゴール名** | 変更なし（generate-entity, execute-ddl等） |
| **設定パラメータ** | 一部変更の可能性あり（genDialectClassName等） |
| **生成されるEntityクラス** | JPA/Doma両対応を維持する場合、出力フォーマット調整が必要 |
| **dicon設定ファイル** | 完全廃止（ユーザーが直接diconを使うケースは想定されない） |
| **対応DB** | 6DB対応の維持が必要 |

---

## 5. 移行方針案

### 5.1 推奨する代替ライブラリの組み合わせ

#### 方針A（推奨）: jOOQ Code Generation + 標準Java API

| 置換対象 | 代替 | 理由 |
|---------|------|------|
| S2JDBC-Gen Entity生成 | **jOOQ Code Generation** | Maven Plugin完備、6DB+対応、活発な開発、OSS版Apache 2.0 |
| S2 Frameworkユーティリティ | 標準Java API + Apache Commons | 既存依存を活用、追加依存最小化 |
| dicon設定 | 不要（jOOQ設定で代替） | S2Container DI自体が不要に |
| Doma連携 | **Doma 3.x へアップグレード**（任意） | 活発開発中。Doma 2.62.0のまま維持も可 |

**メリット**:
- jOOQはEntity生成だけでなく型安全SQLビルダーとしても使える（将来的な拡張性）
- 非常に活発なOSSコミュニティ（Lukas Eder氏が積極的にメンテナンス）
- Maven Central配布、追加リポジトリ不要

**デメリット**:
- jOOQ OSS版はMySQL/PostgreSQL/H2/SQLite等のみ。Oracle/SQL Server/DB2は**商用版（年間ライセンス）**が必要
- 現行の6DB全対応を維持するには商用ライセンスまたは別アプローチが必要

#### 方針B: Hibernate Tools + 標準Java API

| 置換対象 | 代替 | 理由 |
|---------|------|------|
| S2JDBC-Gen Entity生成 | **Hibernate Tools (hbm2java)** | JPA標準準拠、全DB対応、Maven Plugin完備 |
| S2 Frameworkユーティリティ | 標準Java API + Apache Commons | 同上 |
| dicon設定 | `hibernate.properties` or `persistence.xml` | JPA標準設定 |

**メリット**:
- 全6DB対応（追加ライセンス不要）
- JPA標準に完全準拠
- Hibernate本体の一部として長期サポート

**デメリット**:
- Hibernate依存が増大（本体が巨大）
- 生成コードのカスタマイズ性がjOOQに劣る
- Doma Entity生成との両立は追加作業が必要

#### 方針C（最小工数）: ユーティリティのみ先行置換

| 置換対象 | 代替 | 理由 |
|---------|------|------|
| S2 Frameworkユーティリティ | 標準Java API + Apache Commons | リスク最小 |
| S2JDBC-Genは**そのまま維持** | - | EOLだが動作はする |

**メリット**: 工数最小、既存動作を維持
**デメリット**: 根本的な問題（S2JDBC-Gen EOL）を先送り。Java新バージョンでの動作保証なし

### 5.2 段階的移行ロードマップ

```
Phase 1（工数: 小、期間: 1-2週間）
├── S2 Frameworkユーティリティの置換（StringUtil, StatementUtil等）
├── S2 Tiger拡張の置換（CollectionsUtil, Maps等）
├── try-with-resources化（ConnectionUtil, StatementUtil, ResultSetUtil）
└── テスト実行・動作確認

Phase 2（工数: 中、期間: 2-4週間）
├── Entity生成エンジンの設計（jOOQ or Hibernate Toolsの選定）
├── GenDialect体系の再設計
├── dicon設定ファイル生成の廃止
└── PoC（Proof of Concept）実装（H2 DBで検証）

Phase 3（工数: 大、期間: 4-8週間）
├── S2JDBC-Gen完全置換実装
├── 6DB全ての動作確認
├── テストの全面改修
├── 生成Entityの出力フォーマット確認（JPA/Doma両対応）
└── パフォーマンステスト

Phase 4（工数: 小、期間: 1週間）
├── pom.xmlからSeasar依存の完全除去
├── Seasar Maven Repositoryの削除
├── ドキュメント更新
└── リリース準備
```

### 5.3 移行の優先順位

1. **Phase 1を即座に実行**: ユーティリティ置換はリスクが低く、Seasar依存を50%以上削減可能
2. **Phase 2のPoC**: Entity生成エンジンの代替選定には実際に動かして検証が必要
3. **Phase 3は最もリスクが高い**: S2JDBC-Genの完全置換。十分なテストカバレッジの確保が前提

### 5.4 リスクと緩和策

| リスク | 影響度 | 確率 | 緩和策 |
|--------|--------|------|--------|
| S2JDBC-Genの代替で生成Entityのフォーマットが変わる | 高 | 高 | FreeMarkerテンプレートで出力制御。移行ガイド作成 |
| jOOQ OSS版が全DB非対応 | 中 | 確定 | Hibernate Toolsを代替案として用意 / 商用ライセンス検討 |
| Java新バージョンでSeasar2が動作しなくなる | 高 | 中 | Phase 1の早期実行でリスク低減 |
| テストリソース（expected output）の大量更新 | 中 | 高 | 自動生成スクリプトで対応 |
| 下位互換性の破壊 | 中 | 中 | メジャーバージョンアップ（6.0.0）として実施 |

### 5.5 概算工数

| フェーズ | 工数 | 補足 |
|---------|------|------|
| Phase 1（ユーティリティ置換） | **小**（1-2人週） | 機械的置換。リスク低 |
| Phase 2（PoC） | **中**（2-4人週） | 設計+検証。技術選定含む |
| Phase 3（S2JDBC-Gen置換） | **大**（4-8人週） | 全面実装+6DB検証 |
| Phase 4（クリーンアップ） | **小**（0.5-1人週） | 依存除去+リリース |
| **合計** | **大（8-15人週）** | |

---

## 6. 参照URL一覧

### Seasar2 EOL関連
- [Seasar2公式サイト](https://www.seasar.org/)
- [Seasar2 GitHub](https://github.com/seasarorg/seasar2)
- [Seasar2 EOL通知（intra-mart）](https://product.intra-mart.support/hc/ja/articles/360019726453)
- [Seasar Conference最終回レポート](https://gihyo.jp/news/report/2015/10/2701)
- [Seasar2→Spring移行サービス（ウェイン）](https://www.wain.co.jp/migration/javamigration.html)

### 代替ライブラリ
- [jOOQ公式 - Code Generation](https://www.jooq.org/doc/latest/manual/code-generation/)
- [jOOQ Maven Plugin](https://www.jooq.org/doc/latest/manual/code-generation/codegen-maven/)
- [jOOQ - JPADatabase](https://www.jooq.org/doc/latest/manual/code-generation/codegen-jpa/)
- [Hibernate Tools公式](https://hibernate.org/tools/)
- [Hibernate Tools Maven Plugin (GitHub)](https://github.com/hibernate/hibernate-tools/blob/main/maven/README.md)
- [Hibernate Tools hbm2java Mojo](https://stadler.github.io/hibernate-tools/hbm2java-mojo.html)
- [MyBatis Generator公式](https://mybatis.org/generator/)
- [MyBatis Generator GitHub Releases](https://github.com/mybatis/generator/releases)

### Doma（活発開発中）
- [Doma GitHub](https://github.com/domaframework/doma)
- [Doma公式ドキュメント](https://docs.domaframework.org/en/latest/)
- [Doma Releases（最新3.11.1）](https://github.com/domaframework/doma/releases)
- [Nablarch Doma Adapter](https://nablarch.github.io/docs/LATEST/doc/en/application_framework/adaptors/doma_adaptor.html)

### その他
- [jpa-entity-generator (SmartNews)](https://github.com/smartnews/jpa-entity-generator)
- [Gentity - JPA Entity Generator](https://github.com/gentity/gentity)
- [Jeddict - Jakarta EE Modeler](https://jeddict.github.io/)
