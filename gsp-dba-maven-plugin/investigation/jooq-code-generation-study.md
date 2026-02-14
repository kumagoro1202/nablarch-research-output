# jOOQ Code Generation 体系的学習レポート

**作成日**: 2026-02-14
**対象プロジェクト**: gsp-dba-maven-plugin

---

## 目次

1. [jOOQ全体像](#1-jooq全体像)
2. [jOOQ Code Generation 詳細](#2-jooq-code-generation-詳細)
3. [Maven Plugin設定](#3-maven-plugin設定)
4. [カスタマイズパターン](#4-カスタマイズパターン)
5. [gsp-dba-maven-pluginへの適用考察](#5-gsp-dba-maven-pluginへの適用考察)

---

## 1. jOOQ全体像

### 1.1 アーキテクチャ概要

jOOQ（Java Object Oriented Querying）は、Lukas Eder氏が開発するJavaのデータベースアクセスライブラリである。
3つの主要レイヤーで構成される。

```
┌──────────────────────────────────────────────┐
│           jOOQ Architecture                   │
│                                               │
│  ┌─────────────────────────────────────────┐  │
│  │  Layer 1: Code Generation               │  │
│  │  (jooq-codegen / jooq-codegen-maven)    │  │
│  │                                         │  │
│  │  DBスキーマ → Java型安全クラス生成       │  │
│  │  - Tables, Records, POJOs, DAOs         │  │
│  │  - Sequences, UDTs, Routines            │  │
│  └────────────────┬────────────────────────┘  │
│                   │ 生成されたクラスを使用       │
│  ┌────────────────▼────────────────────────┐  │
│  │  Layer 2: DSL (Domain Specific Language) │  │
│  │  (jooq)                                 │  │
│  │                                         │  │
│  │  型安全SQLクエリビルダー                  │  │
│  │  - SELECT, INSERT, UPDATE, DELETE       │  │
│  │  - JOIN, UNION, Subquery               │  │
│  │  - Window Functions, CTEs              │  │
│  └────────────────┬────────────────────────┘  │
│                   │ SQLを実行                   │
│  ┌────────────────▼────────────────────────┐  │
│  │  Layer 3: SQL Execution                 │  │
│  │  (jooq)                                 │  │
│  │                                         │  │
│  │  JDBC経由でSQL実行・結果マッピング        │  │
│  │  - Connection管理                       │  │
│  │  - ResultSet → POJO/Record変換          │  │
│  │  - トランザクション管理                  │  │
│  └─────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

**各レイヤーの責務と依存関係**:

| レイヤー | Mavenアーティファクト | 責務 | 依存関係 |
|---------|---------------------|------|---------|
| Code Generation | `jooq-codegen`, `jooq-codegen-maven` | DBスキーマからJavaソースコード生成 | `jooq-meta`（メタデータ読み取り）に依存 |
| DSL | `jooq` | 型安全SQLクエリの構築 | 独立（生成コードはオプション） |
| SQL Execution | `jooq` | SQL実行、結果マッピング | JDBC |

**重要**: gsp-dba-maven-pluginの文脈では**Layer 1（Code Generation）のみが関係する**。DSLやSQL Executionは不要。

### 1.2 Mavenモジュール構成

jOOQは以下のMavenモジュールで構成される:

| アーティファクト | 説明 | gspでの必要性 |
|----------------|------|-------------|
| `org.jooq:jooq` | コアライブラリ（DSL + Execution） | **不要**（コード生成のみなら） |
| `org.jooq:jooq-meta` | データベースメタデータリーダー | **必要**（コード生成の入力） |
| `org.jooq:jooq-codegen` | コード生成エンジン（ライブラリ） | **必要** |
| `org.jooq:jooq-codegen-maven` | Maven Pluginラッパー | 参考（gspは独自Mojoを使う） |

### 1.3 OSS版 vs 商用版の機能差異

jOOQはデュアルライセンスで、対応DBによってエディションが分かれる。

| エディション | ライセンス | 価格 | 主な対応DB |
|------------|-----------|------|----------|
| **Open Source** | Apache 2.0 | 無料 | Derby, Firebird, H2, HSQLDB, Ignite, MariaDB, MySQL, PostgreSQL, SQLite, YugabyteDB, CockroachDB, DuckDB, ClickHouse, Trino |
| **Express** | 商用 | €99/開発者/年 | + Oracle, SQL Server, MS Access |
| **Professional** | 商用 | €399/開発者/年 | + 高度なjOOQ機能、バージョン固定サポート |
| **Enterprise** | 商用 | €799/開発者/年 | + DB2, SAP HANA, Snowflake, Redshift, Teradata, Informix等 |

**gsp-dba-maven-pluginへの影響**:

| gsp対応DB | jOOQ OSS対応 | 必要エディション |
|----------|-------------|---------------|
| H2 | ✅ | OSS |
| PostgreSQL | ✅ | OSS |
| MySQL | ✅ | OSS |
| Oracle | ❌ | Express以上 |
| SQL Server | ❌ | Express以上 |
| DB2 | ❌ | Enterprise |

→ **6DB全対応にはEnterprise版が必要**。ただし、gspはコード生成部分のみjOOQを使用するため、jOOQ自体のDB接続機能を使わずに`java.sql.DatabaseMetaData`を直接読む独自Databaseを実装すれば、ライセンス制約を回避できる可能性がある（後述§5で考察）。

### 1.4 バージョンと歴史

| バージョン | リリース | 主な変更 |
|----------|---------|---------|
| 3.0 | 2013 | モジュール化、`org.jooq.meta`パッケージ導入 |
| 3.2 | 2014 | デュアルライセンス化（OSS + 商用） |
| 3.11 | 2018 | Java 8必須化 |
| 3.14 | 2020 | Java 11必須化 |
| 3.17 | 2022 | `jakarta.persistence`対応 |
| 3.19 | 2023 | Kotlin/Scala codegen強化 |
| 3.20 | 2024-2025 | ClickHouse, Databricks対応、Java Record POJO |
| **3.21** | **開発中** | 最新開発版 |

**最新安定版**: 3.20.x（2026年2月時点）

---

## 2. jOOQ Code Generation 詳細

### 2.1 処理パイプライン

jOOQのコード生成は以下のパイプラインで処理される:

```
入力                    処理                    出力
───────────         ──────────────         ──────────────
                    ┌──────────────┐
  DBスキーマ    ──→ │  Database     │
  DDLファイル   ──→ │  (メタデータ  │
  JPA Entity   ──→ │   読み取り)   │
  Liquibase    ──→ │              │
  XMLスキーマ  ──→ └──────┬───────┘
                         │
                         │ Definition群
                         │ (TableDefinition,
                         │  ColumnDefinition, ...)
                         │
                    ┌─────▼───────┐
                    │ Generator   │
                    │ (JavaWriter │
                    │  でコード   │
                    │  出力)      │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ Strategy    │      ┌─────────────┐
                    │ (命名規約   │─────→│ Records     │
                    │  解決)      │      │ POJOs       │
                    └─────────────┘      │ DAOs        │
                                         │ Tables      │
                                         │ Sequences   │
                                         │ UDTs        │
                                         │ Routines    │
                                         │ Interfaces  │
                                         └─────────────┘
```

### 2.2 入力ソース（Database実装）

jOOQは複数の入力ソースに対応する。

#### A. ライブデータベース接続（最も一般的）

JDBC接続してDatabaseMetaDataから情報を取得する方式。

| Database実装クラス | 対応DB |
|-------------------|--------|
| `org.jooq.meta.h2.H2Database` | H2 |
| `org.jooq.meta.postgres.PostgresDatabase` | PostgreSQL |
| `org.jooq.meta.mysql.MySQLDatabase` | MySQL |
| `org.jooq.meta.mariadb.MariaDBDatabase` | MariaDB |
| `org.jooq.meta.oracle.OracleDatabase` | Oracle（商用版） |
| `org.jooq.meta.sqlserver.SQLServerDatabase` | SQL Server（商用版） |
| `org.jooq.meta.db2.DB2Database` | DB2（Enterprise版） |
| `org.jooq.meta.sqlite.SQLiteDatabase` | SQLite |
| `org.jooq.meta.derby.DerbyDatabase` | Derby |
| `org.jooq.meta.hsqldb.HSQLDBDatabase` | HSQLDB |
| `org.jooq.meta.firebird.FirebirdDatabase` | Firebird |
| `org.jooq.meta.cockroachdb.CockroachDBDatabase` | CockroachDB |
| `org.jooq.meta.jdbc.JDBCDatabase` | 汎用JDBC（未対応DB用） |

#### B. DDLDatabase（SQLファイルから生成）

Flywayのマイグレーションスクリプト等、SQLファイルを入力として使用する。
実DBへの接続不要。内部でH2等にスキーマを構築して解析する。

```xml
<database>
    <name>org.jooq.meta.extensions.ddl.DDLDatabase</name>
    <properties>
        <property>
            <key>scripts</key>
            <value>src/main/resources/db/migration/*.sql</value>
        </property>
        <property>
            <key>sort</key>
            <value>flyway</value>
        </property>
    </properties>
</database>
```

#### C. JPADatabase（JPA Entityから生成）

既存のJPA Entityクラスを入力として、jOOQクラスを生成する。
Hibernateを内部で使い、インメモリH2にスキーマを構築→逆解析。

```xml
<database>
    <name>org.jooq.meta.extensions.jpa.JPADatabase</name>
    <properties>
        <property>
            <key>packages</key>
            <value>com.example.entity</value>
        </property>
    </properties>
</database>
```

#### D. LiquibaseDatabase

Liquibaseの変更セット（XML/YAML/JSON）を入力とする。

```xml
<database>
    <name>org.jooq.meta.extensions.liquibase.LiquibaseDatabase</name>
    <properties>
        <property>
            <key>scripts</key>
            <value>src/main/resources/db/changelog/db.changelog-master.xml</value>
        </property>
    </properties>
</database>
```

#### E. XMLDatabase

SQL標準INFORMATION_SCHEMA形式のXMLファイルを入力とする。

### 2.3 出力（生成物の種類）

`<generate>`要素で有効/無効を制御する。

| 出力タイプ | 設定キー | 説明 | デフォルト |
|----------|---------|------|----------|
| **Records** | `<records>` | テーブルに対応するRecordクラス（型安全な行表現） | `true` |
| **POJOs** | `<pojos>` | Plain Old Java Objects（getter/setter付き） | `false` |
| **Immutable POJOs** | `<immutablePojos>` | setterなしの不変POJO | `false` |
| **Java Records** | `<pojosAsJavaRecordClasses>` | Java 16+ recordクラスとしてPOJO生成 | `false` |
| **DAOs** | `<daos>` | CRUD操作を持つData Access Objectクラス | `false` |
| **Interfaces** | `<interfaces>` | RecordとPOJOの共通インターフェース | `false` |
| **Tables** | (常に生成) | テーブルメタデータクラス | 常時 |
| **Sequences** | (存在すれば生成) | シーケンスメタデータ | 自動 |
| **UDTs** | (存在すれば生成) | ユーザー定義型 | 自動 |
| **Routines** | (存在すれば生成) | ストアドプロシージャ/ファンクション | 自動 |
| **JPA Annotations** | `<jpaAnnotations>` | POJOにJPAアノテーション付与 | `false` |
| **Validation Annotations** | `<validationAnnotations>` | Bean Validationアノテーション付与 | `false` |
| **Fluent Setters** | `<fluentSetters>` | メソッドチェーン可能なsetter生成 | `false` |
| **Java Time Types** | `<javaTimeTypes>` | `java.time.*`型を使用 | `true`（3.17+） |

### 2.4 主要クラス群

#### エントリポイント

```java
// コード生成の起動
org.jooq.codegen.GenerationTool.generate(configuration);
```

`GenerationTool`はXMLまたはプログラマティック設定（`org.jooq.meta.jaxb.Configuration`）を受け取り、コード生成パイプライン全体を駆動する。

#### Generator（コード生成エンジン）

| クラス | 説明 |
|-------|------|
| `org.jooq.codegen.Generator` | ジェネレータインターフェース |
| `org.jooq.codegen.JavaGenerator` | **デフォルト実装**。Java/Kotlin/Scalaコードを生成 |
| `org.jooq.codegen.JavaWriter` | 生成コードの書き出しユーティリティ |

`JavaGenerator`が担う主な処理:
- `generate(Database)` → 全テーブル/シーケンス/UDT/ルーチンを走査
- `generateTable(TableDefinition)` → テーブルクラス生成
- `generateRecord(TableDefinition)` → Recordクラス生成
- `generatePojo(TableDefinition)` → POJOクラス生成
- `generateDao(TableDefinition)` → DAOクラス生成
- `generateInterface(TableDefinition)` → インターフェース生成

#### GeneratorStrategy（命名戦略）

| クラス | 説明 |
|-------|------|
| `org.jooq.codegen.GeneratorStrategy` | 命名戦略インターフェース |
| `org.jooq.codegen.DefaultGeneratorStrategy` | **デフォルト実装**。CamelCase変換等 |

主要メソッド（オーバーライド可能）:

| メソッド | 責務 | 例 |
|---------|------|-----|
| `getJavaClassName(Definition, Mode)` | クラス名決定 | `USER_TABLE` → `UserTable` |
| `getJavaIdentifier(Definition)` | Java識別子名 | フィールド名等 |
| `getJavaMemberName(Definition, Mode)` | メンバー変数名 | `user_name` → `userName` |
| `getJavaPackageName(Definition, Mode)` | パッケージ名 | `com.example.gen.tables` |
| `getTargetDirectory()` | 出力ディレクトリ | `target/generated-sources/jooq` |

`Mode`はコンテキストを表す enum:
- `DEFAULT` → Tableクラス
- `RECORD` → Recordクラス
- `POJO` → POJOクラス
- `DAO` → DAOクラス
- `INTERFACE` → インターフェース

#### Database（メタデータソース）

| クラス | 説明 |
|-------|------|
| `org.jooq.meta.Database` | メタデータソースインターフェース |
| `org.jooq.meta.AbstractDatabase` | 共通基盤実装 |
| `org.jooq.meta.h2.H2Database` | H2固有実装 |
| `org.jooq.meta.postgres.PostgresDatabase` | PostgreSQL固有実装 |
| (以下各DB実装) | ... |

#### Definition（メタデータモデル）

データベースオブジェクトを表現するモデルクラス群:

| クラス | 説明 |
|-------|------|
| `org.jooq.meta.SchemaDefinition` | スキーマ |
| `org.jooq.meta.TableDefinition` | テーブル/ビュー |
| `org.jooq.meta.ColumnDefinition` | カラム |
| `org.jooq.meta.UniqueKeyDefinition` | ユニークキー |
| `org.jooq.meta.ForeignKeyDefinition` | 外部キー |
| `org.jooq.meta.IndexDefinition` | インデックス |
| `org.jooq.meta.SequenceDefinition` | シーケンス |
| `org.jooq.meta.DataTypeDefinition` | データ型 |

### 2.5 テンプレートエンジンについて

jOOQのコード生成は**テンプレートエンジンを使用しない**。`JavaGenerator`がJava APIで直接コードを出力する（`JavaWriter`を使用）。

これはS2JDBC-GenがFreeMarkerテンプレートを使用するのと大きく異なる点である。
jOOQではテンプレートではなく、`JavaGenerator`のメソッドをオーバーライドしてコード出力をカスタマイズする。

---

## 3. Maven Plugin設定

### 3.1 基本設定（完全な設定例）

以下はPostgreSQLに接続してコード生成を行う完全な設定例:

```xml
<project>
    <dependencies>
        <!-- jOOQコアライブラリ（生成コードのコンパイルに必要） -->
        <dependency>
            <groupId>org.jooq</groupId>
            <artifactId>jooq</artifactId>
            <version>3.20.4</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.jooq</groupId>
                <artifactId>jooq-codegen-maven</artifactId>
                <version>3.20.4</version>

                <!-- プラグイン実行時のみ必要な依存 -->
                <dependencies>
                    <dependency>
                        <groupId>org.postgresql</groupId>
                        <artifactId>postgresql</artifactId>
                        <version>42.7.4</version>
                    </dependency>
                </dependencies>

                <executions>
                    <execution>
                        <id>jooq-codegen</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>

                <configuration>
                    <!-- JDBC接続設定 -->
                    <jdbc>
                        <driver>org.postgresql.Driver</driver>
                        <url>jdbc:postgresql://localhost:5432/mydb</url>
                        <user>myuser</user>
                        <password>mypassword</password>
                    </jdbc>

                    <generator>
                        <!-- ジェネレータ実装クラス -->
                        <name>org.jooq.codegen.JavaGenerator</name>

                        <!-- データベースメタデータ設定 -->
                        <database>
                            <!-- DB固有のメタデータリーダー -->
                            <name>org.jooq.meta.postgres.PostgresDatabase</name>

                            <!-- 対象スキーマ -->
                            <inputSchema>public</inputSchema>

                            <!-- テーブルフィルタ（Java正規表現） -->
                            <includes>.*</includes>
                            <excludes>
                                SCHEMA_INFO
                              | flyway_schema_history
                            </excludes>

                            <!-- 型マッピング（カスタム型変換） -->
                            <forcedTypes>
                                <forcedType>
                                    <userType>java.time.Instant</userType>
                                    <includeTypes>TIMESTAMP.*WITH TIME ZONE</includeTypes>
                                </forcedType>
                            </forcedTypes>
                        </database>

                        <!-- 生成オプション -->
                        <generate>
                            <!-- 生成物の種類 -->
                            <records>true</records>
                            <pojos>true</pojos>
                            <daos>false</daos>
                            <interfaces>false</interfaces>

                            <!-- POJOオプション -->
                            <immutablePojos>false</immutablePojos>
                            <pojosAsJavaRecordClasses>false</pojosAsJavaRecordClasses>
                            <fluentSetters>true</fluentSetters>

                            <!-- アノテーション -->
                            <jpaAnnotations>true</jpaAnnotations>
                            <jpaVersion>3.1</jpaVersion>
                            <validationAnnotations>false</validationAnnotations>

                            <!-- 型 -->
                            <javaTimeTypes>true</javaTimeTypes>

                            <!-- 非推奨マーカー -->
                            <deprecated>false</deprecated>
                        </generate>

                        <!-- 命名戦略 -->
                        <strategy>
                            <name>org.jooq.codegen.DefaultGeneratorStrategy</name>
                        </strategy>

                        <!-- 出力先 -->
                        <target>
                            <packageName>com.example.generated</packageName>
                            <directory>target/generated-sources/jooq</directory>
                        </target>
                    </generator>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3.2 主要パラメータ解説

#### `<jdbc>` — JDBC接続

| パラメータ | 説明 | 必須 |
|----------|------|------|
| `driver` | JDBCドライバクラス名 | ○ |
| `url` | JDBC接続URL | ○ |
| `user` | ユーザー名 | ○ |
| `password` | パスワード | ○ |
| `properties` | 追加接続プロパティ | × |

#### `<database>` — メタデータソース

| パラメータ | 説明 | デフォルト |
|----------|------|----------|
| `name` | Databaseクラス名（FQCN） | (必須) |
| `inputSchema` | 対象スキーマ名 | (必須) |
| `inputCatalog` | 対象カタログ名 | − |
| `includes` | 生成対象テーブル（正規表現） | `.*` |
| `excludes` | 除外テーブル（正規表現） | (空) |
| `includeTables` | テーブルを含めるか | `true` |
| `includeRoutines` | ルーチンを含めるか | `true` |
| `includeSequences` | シーケンスを含めるか | `true` |
| `includeUDTs` | UDTを含めるか | `true` |
| `recordVersionFields` | 楽観ロック用バージョンカラム（正規表現） | − |
| `recordTimestampFields` | タイムスタンプカラム（正規表現） | − |
| `forcedTypes` | カスタム型マッピング | − |
| `properties` | DB固有プロパティ | − |

#### `<generate>` — 生成オプション

| パラメータ | 説明 | デフォルト |
|----------|------|----------|
| `records` | Recordクラス生成 | `true` |
| `pojos` | POJOクラス生成 | `false` |
| `immutablePojos` | 不変POJO | `false` |
| `pojosEqualsAndHashCode` | equals/hashCode生成 | `false` |
| `pojosToString` | toString生成 | `true` |
| `pojosAsJavaRecordClasses` | Java Record形式 | `false` |
| `daos` | DAOクラス生成 | `false` |
| `interfaces` | 共通インターフェース | `false` |
| `fluentSetters` | メソッドチェーンsetter | `false` |
| `javaTimeTypes` | `java.time.*`使用 | `true` |
| `jpaAnnotations` | JPA注釈付与 | `false` |
| `jpaVersion` | JPAバージョン | `2.2` |
| `validationAnnotations` | Bean Validation付与 | `false` |
| `deprecated` | @Deprecated付与 | `true` |
| `generatedAnnotation` | @Generated付与 | `true` |
| `globalObjectNames` | グローバルオブジェクト参照 | `true` |

#### `<target>` — 出力先

| パラメータ | 説明 | デフォルト |
|----------|------|----------|
| `packageName` | 生成コードのパッケージ | (必須) |
| `directory` | 出力ディレクトリ | `target/generated-sources/jooq` |
| `encoding` | ファイルエンコーディング | `UTF-8` |
| `locale` | ロケール | システムデフォルト |

### 3.3 プロファイルを使った複数DB対応パターン

```xml
<profiles>
    <!-- PostgreSQL -->
    <profile>
        <id>postgres</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.jooq</groupId>
                    <artifactId>jooq-codegen-maven</artifactId>
                    <configuration>
                        <jdbc>
                            <driver>org.postgresql.Driver</driver>
                            <url>${postgres.url}</url>
                            <user>${postgres.user}</user>
                            <password>${postgres.password}</password>
                        </jdbc>
                        <generator>
                            <database>
                                <name>org.jooq.meta.postgres.PostgresDatabase</name>
                                <inputSchema>${postgres.schema}</inputSchema>
                            </database>
                        </generator>
                    </configuration>
                    <dependencies>
                        <dependency>
                            <groupId>org.postgresql</groupId>
                            <artifactId>postgresql</artifactId>
                            <version>42.7.4</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </build>
    </profile>

    <!-- H2 -->
    <profile>
        <id>h2</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.jooq</groupId>
                    <artifactId>jooq-codegen-maven</artifactId>
                    <configuration>
                        <jdbc>
                            <driver>org.h2.Driver</driver>
                            <url>${h2.url}</url>
                            <user>${h2.user}</user>
                            <password>${h2.password}</password>
                        </jdbc>
                        <generator>
                            <database>
                                <name>org.jooq.meta.h2.H2Database</name>
                                <inputSchema>${h2.schema}</inputSchema>
                            </database>
                        </generator>
                    </configuration>
                    <dependencies>
                        <dependency>
                            <groupId>com.h2database</groupId>
                            <artifactId>h2</artifactId>
                            <version>2.2.224</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </build>
    </profile>

    <!-- MySQL -->
    <profile>
        <id>mysql</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.jooq</groupId>
                    <artifactId>jooq-codegen-maven</artifactId>
                    <configuration>
                        <jdbc>
                            <driver>com.mysql.cj.jdbc.Driver</driver>
                            <url>${mysql.url}</url>
                            <user>${mysql.user}</user>
                            <password>${mysql.password}</password>
                        </jdbc>
                        <generator>
                            <database>
                                <name>org.jooq.meta.mysql.MySQLDatabase</name>
                                <inputSchema>${mysql.schema}</inputSchema>
                            </database>
                        </generator>
                    </configuration>
                    <dependencies>
                        <dependency>
                            <groupId>com.mysql</groupId>
                            <artifactId>mysql-connector-j</artifactId>
                            <version>8.3.0</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

### 3.4 CI/CD統合パターン

#### パターン1: DBマイグレーション後に実行

```xml
<build>
    <plugins>
        <!-- 1. Flyway migration -->
        <plugin>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-maven-plugin</artifactId>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals><goal>migrate</goal></goals>
                </execution>
            </executions>
        </plugin>

        <!-- 2. jOOQ code generation (Flyway後に実行) -->
        <plugin>
            <groupId>org.jooq</groupId>
            <artifactId>jooq-codegen-maven</artifactId>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals><goal>generate</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### パターン2: DDLファイルから直接（DB接続不要）

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <configuration>
        <!-- jdbc要素不要 -->
        <generator>
            <database>
                <name>org.jooq.meta.extensions.ddl.DDLDatabase</name>
                <properties>
                    <property>
                        <key>scripts</key>
                        <value>src/main/resources/db/migration/*.sql</value>
                    </property>
                    <property>
                        <key>sort</key>
                        <value>flyway</value>
                    </property>
                </properties>
            </database>
        </generator>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.jooq</groupId>
            <artifactId>jooq-meta-extensions</artifactId>
            <version>3.20.4</version>
        </dependency>
    </dependencies>
</plugin>
```

#### パターン3: Testcontainers連携

```xml
<plugin>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-jooq-codegen-maven-plugin</artifactId>
    <version>0.0.4</version>
    <dependencies>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>1.19.3</version>
        </dependency>
    </dependencies>
    <configuration>
        <database>
            <type>POSTGRES</type>
            <containerImage>postgres:16</containerImage>
        </database>
        <flyway>
            <locations>
                filesystem:src/main/resources/db/migration
            </locations>
        </flyway>
        <jooq>
            <generator>
                <target>
                    <packageName>com.example.generated</packageName>
                    <directory>target/generated-sources/jooq</directory>
                </target>
            </generator>
        </jooq>
    </configuration>
</plugin>
```

### 3.5 プログラマティック設定（Javaコードから実行）

Maven Pluginを使わず、Javaコードから直接コード生成を実行できる。
**gsp-dba-maven-pluginのMojoから呼び出す場合に最も重要な方式**。

```java
import org.jooq.codegen.GenerationTool;
import org.jooq.meta.jaxb.*;

// 設定をプログラマティックに構築
Configuration configuration = new Configuration()
    .withJdbc(new Jdbc()
        .withDriver("org.postgresql.Driver")
        .withUrl("jdbc:postgresql://localhost:5432/mydb")
        .withUser("user")
        .withPassword("password"))
    .withGenerator(new Generator()
        .withName("org.jooq.codegen.JavaGenerator")
        .withDatabase(new Database()
            .withName("org.jooq.meta.postgres.PostgresDatabase")
            .withInputSchema("public")
            .withIncludes(".*")
            .withExcludes("SCHEMA_INFO"))
        .withGenerate(new Generate()
            .withPojos(true)
            .withJpaAnnotations(true)
            .withJavaTimeTypes(true)
            .withFluentSetters(true))
        .withStrategy(new Strategy()
            .withName("com.example.MyCustomStrategy"))
        .withTarget(new Target()
            .withPackageName("com.example.generated")
            .withDirectory("target/generated-sources/jooq")));

// 実行
GenerationTool.generate(configuration);
```

---

## 4. カスタマイズパターン

### 4.1 カスタムGeneratorの作り方（JavaGeneratorの拡張）

`JavaGenerator`を継承し、特定のメソッドをオーバーライドすることで生成コードをカスタマイズできる。

#### オーバーライド可能な主要メソッド

| カテゴリ | メソッド | 用途 |
|---------|---------|------|
| テーブル | `generateTableClassFooter(TableDefinition, JavaWriter)` | テーブルクラスの末尾にコード追加 |
| テーブル | `generateTableClassJavadoc(TableDefinition, JavaWriter)` | テーブルクラスのJavadoc |
| レコード | `generateRecordClassFooter(TableDefinition, JavaWriter)` | Recordクラスの末尾にコード追加 |
| レコード | `generateRecordClassJavadoc(TableDefinition, JavaWriter)` | RecordクラスのJavadoc |
| POJO | `generatePojoClassFooter(TableDefinition, JavaWriter)` | POJOクラスの末尾にコード追加 |
| POJO | `generatePojoClassJavadoc(TableDefinition, JavaWriter)` | POJOクラスのJavadoc |
| DAO | `generateDaoClassFooter(TableDefinition, JavaWriter)` | DAOクラスの末尾にコード追加 |
| Enum | `generateEnumClassFooter(EnumDefinition, JavaWriter)` | Enumクラスの末尾にコード追加 |
| Interface | `generateInterfaceClassFooter(TableDefinition, JavaWriter)` | インターフェースの末尾にコード追加 |

#### 例1: POJOクラスにカスタムtoString()を追加

```java
package com.example.codegen;

import org.jooq.codegen.JavaGenerator;
import org.jooq.codegen.JavaWriter;
import org.jooq.meta.TableDefinition;

public class GspJavaGenerator extends JavaGenerator {

    @Override
    protected void generatePojoClassFooter(TableDefinition table, JavaWriter out) {
        out.println();
        out.tab(1).println("@Override");
        out.tab(1).println("public String toString() {");
        out.tab(2).println("return \"%s\";", table.getName());
        out.tab(1).println("}");
    }
}
```

#### 例2: Recordクラスにカスタムメソッドを追加

```java
public class CustomGenerator extends JavaGenerator {

    @Override
    protected void generateRecordClassFooter(TableDefinition table, JavaWriter out) {
        out.println();
        out.tab(1).println("/**");
        out.tab(1).println(" * Auto-generated custom method.");
        out.tab(1).println(" */");
        out.tab(1).println("public String toDisplayString() {");
        out.tab(2).println("return \"Record[\" + valuesRow() + \"]\";");
        out.tab(1).println("}");
    }
}
```

#### 設定方法

```xml
<generator>
    <!-- カスタムGeneratorクラスを指定 -->
    <name>com.example.codegen.GspJavaGenerator</name>
    <!-- ... 他の設定 ... -->
</generator>
```

**注意**: カスタムGeneratorクラスはMaven Pluginの`<dependencies>`に含める必要がある。
別モジュールとしてビルドするか、プラグインの`<dependencies>`にプロジェクト自身を含める。

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <dependencies>
        <!-- カスタムGeneratorを含むモジュール -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>my-codegen-extensions</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</plugin>
```

### 4.2 カスタムGeneratorStrategyの作り方

`DefaultGeneratorStrategy`を継承して命名規約を変更する。

#### 例1: テーブル名のプレフィックス除去

```java
package com.example.codegen;

import org.jooq.codegen.DefaultGeneratorStrategy;
import org.jooq.meta.Definition;

public class GspGeneratorStrategy extends DefaultGeneratorStrategy {

    @Override
    public String getJavaClassName(Definition definition, Mode mode) {
        String name = super.getJavaClassName(definition, mode);

        // テーブル名の "T_" プレフィックスを除去
        if (mode == Mode.DEFAULT || mode == Mode.RECORD || mode == Mode.POJO) {
            if (name.startsWith("T")) {
                name = name.substring(1);
            }
        }

        return name;
    }
}
```

#### 例2: パッケージ構成のカスタマイズ

```java
public class CustomPackageStrategy extends DefaultGeneratorStrategy {

    @Override
    public String getJavaPackageName(Definition definition, Mode mode) {
        String base = getTargetPackage();

        switch (mode) {
            case POJO:      return base + ".model";
            case DAO:       return base + ".dao";
            case RECORD:    return base + ".record";
            case INTERFACE: return base + ".api";
            default:        return base + ".table";
        }
    }
}
```

#### 設定方法

```xml
<generator>
    <strategy>
        <name>com.example.codegen.GspGeneratorStrategy</name>
    </strategy>
</generator>
```

### 4.3 カスタムDatabase実装

`org.jooq.meta.AbstractDatabase`を継承して、独自のメタデータソースを実装できる。

```java
package com.example.codegen;

import org.jooq.meta.*;
import java.sql.*;
import java.util.*;

public class CustomDatabase extends AbstractDatabase {

    @Override
    protected DSLContext create0() {
        // jOOQ DSLContextを初期化（メタデータクエリ用）
        return DSL.using(getConnection(), SQLDialect.DEFAULT);
    }

    @Override
    protected List<SchemaDefinition> getSchemata0() throws SQLException {
        // スキーマ一覧を返す
        List<SchemaDefinition> result = new ArrayList<>();
        result.add(new SchemaDefinition(this, "", ""));
        return result;
    }

    @Override
    protected List<TableDefinition> getTables0() throws SQLException {
        // テーブル一覧をDatabaseMetaDataから読み取る
        List<TableDefinition> result = new ArrayList<>();
        DatabaseMetaData meta = getConnection().getMetaData();
        ResultSet rs = meta.getTables(null, getInputSchema(), "%",
            new String[]{"TABLE", "VIEW"});

        while (rs.next()) {
            String tableName = rs.getString("TABLE_NAME");
            SchemaDefinition schema = getSchemata().get(0);
            result.add(new TableDefinition(schema, tableName, ""));
        }
        return result;
    }

    @Override
    protected List<SequenceDefinition> getSequences0() throws SQLException {
        return Collections.emptyList();
    }

    // ... 他の抽象メソッドも実装必要
}
```

**注意**: カスタムDatabaseの実装は複雑であり、`AbstractDatabase`の多数の抽象メソッドを実装する必要がある。
現実的には`JDBCDatabase`（汎用実装）をベースにカスタマイズするのが効率的。

### 4.4 JPA/Domaアノテーション付きPOJO生成

#### JPA Entityとしての生成

```xml
<generate>
    <pojos>true</pojos>
    <jpaAnnotations>true</jpaAnnotations>
    <jpaVersion>3.1</jpaVersion>
</generate>
```

生成されるコード例:

```java
@Entity
@Table(name = "USER_TABLE", schema = "PUBLIC",
    uniqueConstraints = @UniqueConstraint(columnNames = {"EMAIL"}))
public class UserTable implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "USER_ID", nullable = false, precision = 10)
    private Integer userId;

    @Column(name = "USER_NAME", nullable = false, length = 100)
    private String userName;

    @Column(name = "EMAIL", nullable = false, length = 255)
    private String email;

    // getter/setter ...
}
```

#### Domaアノテーション付きPOJOの生成

jOOQはDoma固有のアノテーションを直接サポートしない。
カスタムGeneratorでDoma `@Entity`, `@Column`, `@Id`等を付与する必要がある。

```java
public class DomaEntityGenerator extends JavaGenerator {

    @Override
    protected void generatePojoClassAnnotations(TableDefinition table, JavaWriter out) {
        // Doma @Entity アノテーション
        out.println("@org.seasar.doma.Entity");
        out.println("@org.seasar.doma.Table(name = \"%s\")", table.getName());
    }

    @Override
    protected void generatePojoGetter(TypedElementDefinition<?> column,
                                       int index, JavaWriter out) {
        // Domaは publicフィールドを使うため getter は不要
        // フィールドに @Column を付与
    }
}
```

**注**: Doma Entity生成にはjOOQ以外の選択肢（Doma CodeGenやgsp独自テンプレート）も検討が必要。

### 4.5 テーブルフィルタリング（include/exclude）

```xml
<database>
    <!-- 対象テーブル（Java正規表現） -->
    <includes>
        USER_.*        <!-- USER_ で始まるテーブルのみ -->
      | ORDER_.*       <!-- ORDER_ で始まるテーブルのみ -->
      | PRODUCT        <!-- PRODUCT テーブル -->
    </includes>

    <!-- 除外テーブル（includesより優先） -->
    <excludes>
        .*_BACKUP      <!-- _BACKUP で終わるテーブルを除外 -->
      | SCHEMA_INFO    <!-- SCHEMA_INFO を除外 -->
      | flyway_.*      <!-- Flyway管理テーブルを除外 -->
    </excludes>

    <!-- POJO生成のみのフィルタ -->
    <pojosIncludes>USER_.*</pojosIncludes>
    <pojosExcludes></pojosExcludes>

    <!-- Record生成のみのフィルタ -->
    <recordsIncludes>.*</recordsIncludes>
    <recordsExcludes></recordsExcludes>
</database>
```

### 4.6 型マッピングのカスタマイズ（forcedTypes）

#### 基本例: カラム型の変換

```xml
<database>
    <forcedTypes>
        <!-- TIMESTAMP WITH TIME ZONE → Instant -->
        <forcedType>
            <userType>java.time.Instant</userType>
            <includeTypes>TIMESTAMP.*WITH TIME ZONE</includeTypes>
        </forcedType>

        <!-- FLAGカラム → Boolean -->
        <forcedType>
            <userType>java.lang.Boolean</userType>
            <converter>org.jooq.impl.EnumConverter</converter>
            <includeExpression>.*\.FLAG_.*</includeExpression>
            <includeTypes>CHAR\(1\)</includeTypes>
        </forcedType>

        <!-- JSON型 → カスタムクラス -->
        <forcedType>
            <userType>com.example.JsonData</userType>
            <converter>com.example.converter.JsonDataConverter</converter>
            <includeTypes>JSON|JSONB</includeTypes>
        </forcedType>
    </forcedTypes>
</database>
```

#### カスタムConverterの実装

```java
package com.example.converter;

import org.jooq.Converter;

public class JsonDataConverter implements Converter<String, JsonData> {

    @Override
    public JsonData from(String databaseObject) {
        return databaseObject == null ? null : JsonData.parse(databaseObject);
    }

    @Override
    public String to(JsonData userObject) {
        return userObject == null ? null : userObject.toJson();
    }

    @Override
    public Class<String> fromType() { return String.class; }

    @Override
    public Class<JsonData> toType() { return JsonData.class; }
}
```

---

## 5. gsp-dba-maven-pluginへの適用考察

### 5.1 S2JDBC-Gen Entity生成パイプラインとjOOQの対応関係

```
S2JDBC-Gen パイプライン              jOOQ パイプライン
════════════════════                ════════════════════

GenerateEntityCommand               GenerationTool.generate()
  │                                   │
  ├→ FactoryImpl                      ├→ Configuration (JAXB)
  │   ├→ EntityDescFactory            │
  │   ├→ AttributeDescFactory         │
  │   └→ EntityModelFactory           │
  │                                   │
  ├→ DbTableMetaReader               ├→ Database (meta)
  │   └→ DbTableMetaReaderImpl       │   ├→ H2Database
  │       └→ DatabaseMetaData        │   ├→ PostgresDatabase
  │                                   │   └→ (各DB実装)
  │                                   │
  ├→ GenDialect                      ├→ (不要 — DB実装に内包)
  │   ├→ H2GenDialect                │
  │   ├→ PostgreGenDialect           │
  │   └→ (各DB方言)                  │
  │                                   │
  └→ FreeMarker Template             └→ JavaGenerator
      └→ gsp_entity.ftl                 └→ generatePojo() + overrides
```

**主要な対応関係**:

| S2JDBC-Gen コンポーネント | jOOQ 対応 | 移行方針 |
|--------------------------|-----------|---------|
| `GenerateEntityCommand` | `GenerationTool` + `Configuration` | プログラマティック設定で置換 |
| `CommandInvoker` | 不要（GenerationToolが直接実行） | 削除 |
| `FactoryImpl` / `GspFactoryImpl` | 不要（Generatorに統合） | 削除 |
| `EntityDescFactory` | `TableDefinition` + `ColumnDefinition` | jOOQ meta APIで代替 |
| `DbTableMetaReaderImpl` | `Database`実装（H2Database等） | jOOQ標準Databaseを使用 |
| `GenDialect` / `GenDialectRegistry` | jOOQ `SQLDialect` + `Database`実装 | jOOQ標準で代替 |
| `FreeMarker gsp_entity.ftl` | カスタム`JavaGenerator`のoverride | Java APIでコード出力 |

### 5.2 GenDialect体系とjOOQ SQLDialectの対応

S2JDBC-Genの`GenDialect`は型マッピングとSQL方言を担当していた。
jOOQでは各`Database`実装が型マッピングを内包し、`SQLDialect`が方言を管理する。

| gsp GenDialect | jOOQ Database | jOOQ SQLDialect |
|---------------|---------------|-----------------|
| `H2GenDialect` | `H2Database` | `SQLDialect.H2` |
| `PostgreGenDialect` | `PostgresDatabase` | `SQLDialect.POSTGRES` |
| `MysqlGenDialect` | `MySQLDatabase` | `SQLDialect.MYSQL` |
| `OracleGenDialect` | `OracleDatabase` ※商用版 | `SQLDialect.ORACLE` |
| `MssqlGenDialect` | `SQLServerDatabase` ※商用版 | `SQLDialect.SQLSERVER` |
| `Db2GenDialect` | `DB2Database` ※Enterprise版 | `SQLDialect.DB2` |

**gsp固有の`ExtendedXxxGenDialect`**: 現在のgspでは`ExtendedOracleGenDialect`等でOracleのDATE→`java.sql.Date`マッピング等をカスタマイズしている。jOOQでは`forcedTypes`設定で同等の型マッピングカスタマイズが可能。

```xml
<!-- Oracle DATE → java.sql.Date のマッピング例 -->
<forcedTypes>
    <forcedType>
        <userType>java.sql.Date</userType>
        <includeTypes>DATE</includeTypes>
    </forcedType>
</forcedTypes>
```

### 5.3 JPA Entity / Doma Entity 両方の生成

gsp-dba-maven-pluginは`entityType`パラメータで JPA（デフォルト）または Doma のEntity生成を切り替えている。

#### JPA Entity生成

jOOQ標準機能で直接対応可能:

```xml
<generate>
    <pojos>true</pojos>
    <jpaAnnotations>true</jpaAnnotations>
    <jpaVersion>3.1</jpaVersion>
    <fluentSetters>false</fluentSetters>
</generate>
```

S2JDBC-Genの`gsp_entity.ftl`テンプレートで生成していた`@Entity`, `@Table`, `@Column`, `@Id`, `@GeneratedValue`等はjOOQの`<jpaAnnotations>true</jpaAnnotations>`設定で自動付与される。

#### Doma Entity生成

jOOQはDoma固有のアノテーション（`@org.seasar.doma.Entity`, `@org.seasar.doma.Column`等）を標準サポートしない。

**方針A: カスタムJavaGeneratorで対応**

```java
public class DomaEntityGenerator extends JavaGenerator {

    @Override
    protected void printClassAnnotations(JavaWriter out, TableDefinition table, Mode mode) {
        if (mode == Mode.POJO) {
            out.println("@org.seasar.doma.Entity");
            out.println("@org.seasar.doma.Table(name = \"%s\")", table.getName());
        }
    }

    // カラムアノテーション等もオーバーライドで対応
}
```

**方針B: jOOQでPOJO生成 → 後処理でDomaアノテーション付与**

生成されたPOJOを後処理スクリプト（またはアノテーションプロセッサ）で変換する方式。
管理が複雑になるため推奨しない。

**方針C: Doma Entity生成はgsp独自FreeMarkerテンプレートを維持**

jOOQのメタデータ（`TableDefinition`, `ColumnDefinition`等）をFreeMarkerテンプレートのデータモデルに変換し、既存の`gsp_doma_entity.ftl`（もしくは新規テンプレート）で生成する方式。
S2JDBC-Genへの依存は排除しつつ、テンプレートベースの柔軟性を維持できる。

**推奨: 方針A（JPA）+ 方針C（Doma）の併用**

```
entityType=jpa  → jOOQの<jpaAnnotations>true</jpaAnnotations>で直接生成
entityType=doma → jOOQ metaでDB読み取り → FreeMarkerテンプレートでDoma Entity生成
```

### 5.4 FreeMarkerテンプレートとjOOQの統合方法

gsp-dba-maven-pluginは`gsp_entity.ftl`等のFreeMarkerテンプレートを使用してEntity生成をカスタマイズしている。
jOOQはテンプレートエンジンを使わないため、2つの統合方法がある。

#### 方法1: JavaGeneratorのオーバーライドに完全移行

FreeMarkerテンプレートをJava APIでの出力に置き換える。
テンプレートの変更にはJavaコードの修正が必要になるが、型安全性は向上する。

```java
public class GspEntityGenerator extends JavaGenerator {

    @Override
    protected void generatePojo(TableDefinition table, JavaWriter out) {
        // gsp_entity.ftl の内容をJava APIで再実装
        // テーブルコメントからJavadocを生成
        if (table.getComment() != null) {
            out.println("/**");
            out.println(" * %s", table.getComment());
            out.println(" */");
        }

        // Entity出力
        out.println("@Entity");
        out.println("@Table(name = \"%s\")", table.getName());
        out.println("public class %s {",
            getStrategy().getJavaClassName(table, Mode.POJO));

        for (ColumnDefinition column : table.getColumns()) {
            generatePojoField(column, out);
        }

        out.println("}");
    }
}
```

#### 方法2: jOOQ meta → FreeMarkerテンプレートのブリッジ

jOOQのメタデータを読み取り、FreeMarkerテンプレートのデータモデルに変換する。
既存テンプレートを活用できるため移行コストが低い。

```java
public class JooqFreeMarkerBridge {

    /**
     * jOOQのTableDefinitionからFreeMarkerのデータモデルを構築する。
     */
    public Map<String, Object> buildModel(TableDefinition table) {
        Map<String, Object> model = new HashMap<>();

        // テーブル情報
        model.put("tableName", table.getName());
        model.put("comment", table.getComment());
        model.put("className", toCamelCase(table.getName()));

        // カラム情報
        List<Map<String, Object>> columns = new ArrayList<>();
        for (ColumnDefinition col : table.getColumns()) {
            Map<String, Object> colModel = new HashMap<>();
            colModel.put("name", col.getName());
            colModel.put("javaType", resolveJavaType(col));
            colModel.put("nullable", col.isNullable());
            colModel.put("comment", col.getComment());
            colModel.put("primaryKey", isPrimaryKey(col));
            columns.add(colModel);
        }
        model.put("columns", columns);

        return model;
    }
}
```

**推奨: 方法1（JPA Entity）+ 方法2（Doma Entity）の併用**

### 5.5 ライセンス制約の回避策

Oracle/SQL Server/DB2はjOOQ OSS版で非対応だが、以下の回避策がある:

#### 回避策1: JDBCDatabase（汎用メタデータリーダー）

jOOQの`JDBCDatabase`は`java.sql.DatabaseMetaData`を直接使用するため、ライセンス制約なく全DBで動作する。

```xml
<database>
    <name>org.jooq.meta.jdbc.JDBCDatabase</name>
    <inputSchema>MY_SCHEMA</inputSchema>
</database>
```

ただし、DB固有のメタデータ（Oracle固有型、SQL Server固有型等）の精度はネイティブ実装に劣る。

#### 回避策2: gsp独自Databaseの実装

現行のgspの`Dialect`クラス群（H2Dialect, OracleDialect等）が持つDB固有知識を、jOOQの`Database`インターフェースに移植する。

```java
public class GspOracleDatabase extends AbstractDatabase {
    // 既存のOracleDialectの知識を活用してメタデータを提供
    // jOOQ商用版に依存せず、DatabaseMetaData直接アクセス
}
```

この方式はjOOQ OSS版のみで全6DB対応が可能だが、実装工数が大きい。

#### 回避策3: コード生成部分のみjOOQ meta依存を外す

jOOQ metaではなく、gsp独自のメタデータリーダー（現行の`DbTableMetaReaderWithView`を発展させたもの）でDBスキーマを読み取り、jOOQの`JavaGenerator`相当の機能を独自実装する。

この場合、jOOQへの依存は完全になくなるが、コード生成エンジン自体を自前で実装する必要があり、現実的ではない。

#### 推奨

**回避策1（JDBCDatabase）を第一選択とし、DB固有精度が不足する場合は回避策2を検討**。
Oracle/SQL ServerのDATE型マッピング等の精度問題は`forcedTypes`で補完可能。

### 5.6 移行後のGenerateEntity Mojoイメージ

```java
@Mojo(name = "generate-entity")
public class GenerateEntity extends AbstractDbaMojo {

    @Parameter(defaultValue = "entity")
    protected String entityPackageName;

    @Parameter(required = true)
    protected String rootPackage;

    @Parameter(defaultValue = "jpa")
    protected String entityType;

    @Parameter(defaultValue = "target/generated-sources/entity/")
    protected File javaFileDestDir;

    @Override
    protected void executeMojoSpec() throws MojoExecutionException {
        try {
            if ("doma".equals(entityType)) {
                generateDomaEntities();  // FreeMarkerテンプレート方式を維持
            } else {
                generateJpaEntities();   // jOOQ Code Generationで生成
            }
        } catch (Exception e) {
            throw new MojoExecutionException("Entity生成中にエラー", e);
        }
    }

    private void generateJpaEntities() throws Exception {
        Configuration config = new Configuration()
            .withJdbc(new Jdbc()
                .withDriver(driver)
                .withUrl(url)
                .withUser(adminUser)
                .withPassword(adminPassword))
            .withGenerator(new Generator()
                .withName(GspEntityGenerator.class.getName())
                .withDatabase(new Database()
                    .withName(resolveJooqDatabase())
                    .withInputSchema(schema)
                    .withExcludes("SCHEMA_INFO|.*\\$.*"))
                .withGenerate(new Generate()
                    .withPojos(true)
                    .withRecords(false)
                    .withDaos(false)
                    .withJpaAnnotations(true)
                    .withJavaTimeTypes(useJSR310))
                .withStrategy(new Strategy()
                    .withName(GspGeneratorStrategy.class.getName()))
                .withTarget(new Target()
                    .withPackageName(rootPackage + "." + entityPackageName)
                    .withDirectory(javaFileDestDir.getAbsolutePath())));

        GenerationTool.generate(config);
    }

    private String resolveJooqDatabase() {
        // URL/driverからjOOQ Database実装を決定
        // 商用DB（Oracle, SQL Server, DB2）はJDBCDatabaseを使用
        String[] urlTokens = url.split(":");
        String dbType = urlTokens[1];
        switch (dbType) {
            case "h2":         return "org.jooq.meta.h2.H2Database";
            case "postgresql": return "org.jooq.meta.postgres.PostgresDatabase";
            case "mysql":      return "org.jooq.meta.mysql.MySQLDatabase";
            default:           return "org.jooq.meta.jdbc.JDBCDatabase";
        }
    }
}
```

---

## 参考資料

### 公式ドキュメント
- [jOOQ公式マニュアル - Code Generation](https://www.jooq.org/doc/latest/manual/code-generation/)
- [jOOQ Configuration and Setup](https://www.jooq.org/doc/latest/manual/code-generation/codegen-configuration/)
- [jOOQ Maven Plugin](https://www.jooq.org/doc/latest/manual/code-generation/codegen-execution/codegen-maven/)
- [jOOQ Custom Code Sections](https://www.jooq.org/doc/latest/manual/code-generation/codegen-custom-code/)
- [jOOQ Custom Generator Strategies](https://www.jooq.org/doc/latest/manual/code-generation/codegen-object-naming/codegen-generatorstrategy/)
- [jOOQ Generator Configuration](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-generator/)
- [jOOQ Database Name and Properties](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-database/codegen-database-name/)
- [jOOQ Programmatic Configuration](https://www.jooq.org/doc/latest/manual/code-generation/codegen-execution/codegen-programmatic/)
- [jOOQ JPADatabase](https://www.jooq.org/doc/latest/manual/code-generation/codegen-meta-sources/codegen-jpa/)
- [jOOQ DDLDatabase](https://www.jooq.org/doc/latest/manual/code-generation/codegen-meta-sources/codegen-ddl/)
- [jOOQ LiquibaseDatabase](https://www.jooq.org/doc/latest/manual/code-generation/codegen-liquibase/)
- [jOOQ JPA Annotations](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-generate/codegen-generate-annotations/)
- [jOOQ Forced Types](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-database/codegen-database-forced-types/)
- [jOOQ Licensing](https://www.jooq.org/legal/licensing)
- [jOOQ Download & Pricing](https://www.jooq.org/download/)
- [jOOQ Supported RDBMS](https://www.jooq.org/doc/latest/manual/reference/supported-rdbms/)
- [jOOQ Version Support Matrix](https://www.jooq.org/download/support-matrix)

### GitHub
- [jOOQ GitHub Repository](https://github.com/jOOQ/jOOQ)
- [JavaGenerator.java ソースコード](https://github.com/jOOQ/jOOQ/blob/main/jOOQ-codegen/src/main/java/org/jooq/codegen/JavaGenerator.java)
- [jOOQ-codegen-maven pom.xml](https://github.com/jOOQ/jOOQ/blob/main/jOOQ-codegen-maven/pom.xml)
- [testcontainers-jooq-codegen-maven-plugin](https://github.com/testcontainers/testcontainers-jooq-codegen-maven-plugin)

### 参考記事
- [Getting Started with jOOQ - Baeldung](https://www.baeldung.com/jooq-intro)
- [jOOQ Facts: From JPA Annotations to jOOQ Table Mappings - Vlad Mihalcea](https://vladmihalcea.com/jooq-facts-from-jpa-annotations-to-jooq-table-mappings/)
- [Using jOOQ with Spring: Code Generation - Petri Kainulainen](https://www.petrikainulainen.net/programming/jooq/using-jooq-with-spring-code-generation/)
- [Flyway and jOOQ for Unbeatable SQL Development Productivity - jOOQ Blog](https://blog.jooq.org/flyway-and-jooq-for-unbeatable-sql-development-productivity/)
- [Using Testcontainers to Generate jOOQ Code - jOOQ Blog](https://blog.jooq.org/using-testcontainers-to-generate-jooq-code/)

### 関連レポート
- [gsp-dba-maven-plugin Seasar依存調査＋代替案レポート](gsp-seasar-dependency-analysis.md)
