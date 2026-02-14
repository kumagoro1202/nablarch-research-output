# gsp-dba-maven-plugin Phase 2 設計書: jOOQ Code Generation PoC

**作成日**: 2026-02-14
**対象プロジェクト**: gsp-dba-maven-plugin v5.2.0
**ブランチ**: feature/phase2-jooq-poc（feature/remove-seasar-utilities から分岐）

---

## 1. 設計方針

### 1.1 現状のS2JDBC-Gen依存構造

```
GenerateEntity (Mojo)
  ├→ dicon設定ファイル生成（FreeMarker）
  └→ executeGenerateEntity()
      ├→ ExtendedGenerateEntityCommand extends GenerateEntityCommand  ← S2JDBC-Gen
      ├→ CommandInvokerImpl  ← S2JDBC-Gen
      └→ GspFactoryImpl / DomaGspFactoryImpl extends FactoryImpl  ← S2JDBC-Gen
          ├→ DbTableMetaReaderWithView extends DbTableMetaReaderImpl  ← S2JDBC-Gen
          ├→ EntitySetDescFactoryImpl  ← S2JDBC-Gen
          ├→ GspAttributeDescFactoryImpl extends AttributeDescFactoryImpl  ← S2JDBC-Gen
          ├→ JakartaEntityModelFactoryImpl extends EntityModelFactoryImpl  ← S2JDBC-Gen
          └→ JSR310AttributeModelFactoryImpl extends AttributeModelFactoryImpl  ← S2JDBC-Gen
```

S2JDBC-Genへの依存ポイント:
- **コマンド実行パイプライン**: `GenerateEntityCommand` → `CommandInvoker` → `FactoryImpl`
- **メタデータ読取**: `DbTableMetaReaderImpl`（`DbTableMetaReaderWithView`の親クラス）
- **型マッピング**: 6つの`ExtendedXxxGenDialect` + `GenDialectRegistry`
- **エンティティ記述**: `EntityDescFactory`, `AttributeDescFactory`
- **モデル生成**: `EntityModelFactory`, `AttributeModelFactory`
- **テンプレート出力**: S2JDBC-Gen内蔵のFreeMarkerテンプレートエンジン（`GeneratorImpl`）

### 1.2 jOOQ Code Generation 採用方針

**方針: jOOQの`JDBCDatabase`+カスタム`JavaGenerator`によるEntity生成**

| 観点 | 採用方式 | 理由 |
|------|---------|------|
| メタデータ読取 | `org.jooq.meta.jdbc.JDBCDatabase` | 標準JDBC `DatabaseMetaData`を使用。全6DB対応。OSS版で使用可能 |
| コード生成 | `JavaGenerator`拡張 + 既存FreeMarkerテンプレート併用 | jOOQ組込みのJPA annotation生成を基盤とし、gsp固有の出力形式は既存テンプレートで対応 |
| 型マッピング | `JDBCDatabase`のデフォルト型マッピング + `ForcedType`設定 | S2JDBC-GenのGenDialect columnTypeMapを`ForcedType`で代替 |
| DB方言 | `JDBCDatabase`のDB自動検出 | `JDBCDatabase`はJDBC URLからDB種別を自動判定。明示的なDialect指定不要 |
| 設定方式 | プログラマティック設定（`Configuration`オブジェクト） | Maven Mojoパラメータとの統合が容易。XML不要 |

### 1.3 jOOQ OSS版のDB対応制限と対策

| DB | jOOQ OSS対応 | JDBCDatabase対応 | 対策 |
|----|-------------|-----------------|------|
| H2 | **対応** | **対応** | そのまま使用 |
| PostgreSQL | **対応** | **対応** | そのまま使用 |
| MySQL | **対応** | **対応** | そのまま使用 |
| Oracle | **非対応（商用版）** | **対応** | `JDBCDatabase`経由なら動作可能 |
| SQL Server | **非対応（商用版）** | **対応** | `JDBCDatabase`経由なら動作可能 |
| DB2 | **非対応（商用版）** | **対応** | `JDBCDatabase`経由なら動作可能 |

**重要**: `JDBCDatabase`は標準JDBC `DatabaseMetaData` APIを使用するため、jOOQ OSS版でも全DBで動作する。DB固有の`org.jooq.meta.oracle.OracleDatabase`等は不要。

---

## 2. jOOQカスタムGeneratorの設計

### 2.1 クラス構成

```
jp.co.tis.gsp.tools.dba.entitygen/
├── GspEntityGenerationConfig.java    ← jOOQ設定構築（Mojoパラメータ→jOOQ Configuration変換）
├── GspJpaEntityGenerator.java        ← JPA Entity生成（JavaGenerator拡張）
├── GspDomaEntityGenerator.java       ← Doma Entity生成（JavaGenerator拡張）
├── GspGeneratorStrategy.java         ← 命名戦略（GeneratorStrategy実装）
├── GspColumnTypeMapper.java          ← DB型→Java型マッピング（ForcedType設定生成）
└── GspViewSupport.java               ← VIEW対応（既存ViewAnalyzerを統合）
```

### 2.2 GspJpaEntityGenerator（JPA Entity生成モード）

```java
/**
 * jOOQ JavaGeneratorを拡張し、gsp-dba-maven-plugin互換のJPA Entityを生成する。
 *
 * 生成されるEntityの特徴:
 * - Jakarta Persistence API（jakarta.persistence.*）使用
 * - @Generated("GSP") アノテーション付与
 * - @Entity, @Table, @Column, @Id, @GeneratedValue, @SequenceGenerator 等
 * - Serializable実装
 * - publicフィールド（useAccessor=falseの場合）
 * - JSR310対応（LocalDate/LocalDateTime/LocalTime）
 * - VIEW対応（VIEWの基底テーブルからPK/FK取得）
 */
public class GspJpaEntityGenerator extends JavaGenerator {

    @Override
    protected void generatePojo(TableDefinition table, JavaWriter out) {
        // gsp互換のJPA Entity生成ロジック
        // 1. import文の構築（Jakarta Persistence）
        // 2. @Generated, @Entity, @Table アノテーション
        // 3. フィールド定義（@Id, @Column, @GeneratedValue等）
        // 4. アクセサ（useAccessor設定に応じて）
    }

    @Override
    protected void generatePojoClassAnnotations(TableDefinition table, JavaWriter out) {
        // @Generated("GSP")
        // @Entity
        // @Table(schema=..., name=...)
    }
}
```

**生成出力例（JPA Entity）**:
```java
@Generated("GSP")
@Entity
@Table(name = "TEST_TBL1")
public class TestTbl1 implements Serializable {
    private static final long serialVersionUID = 1L;

    /** testTbl1Id */
    @Id
    @GeneratedValue(generator = "TEST_TBL1_ID_SEQ", strategy = GenerationType.AUTO)
    @SequenceGenerator(name = "TEST_TBL1_ID_SEQ", sequenceName = "TEST_TBL1_ID_SEQ",
                       initialValue = 1, allocationSize = 1)
    @Column(name = "TEST_TBL1_ID", nullable = false, unique = true)
    public Long testTbl1Id;

    /** testName */
    @Column(name = "TEST_NAME", length = 30, nullable = true, unique = false)
    public String testName;
}
```

### 2.3 GspDomaEntityGenerator（Doma Entity生成モード）

```java
/**
 * Doma Entity生成用のカスタムGenerator。
 * org.seasar.doma.* アノテーションを使用したEntityを生成する。
 */
public class GspDomaEntityGenerator extends JavaGenerator {

    @Override
    protected void generatePojo(TableDefinition table, JavaWriter out) {
        // Doma互換のEntity生成ロジック
        // 1. import文の構築（org.seasar.doma.*）
        // 2. @Entity, @Table アノテーション
        // 3. フィールド定義（@Id, @Column, @GeneratedValue, @Version等）
        // 4. getter/setter（Domaではアクセサ必須）
    }
}
```

**生成出力例（Doma Entity）**:
```java
@Generated("GSP")
@Entity
@Table(name = "TEST_TBL1")
public class TestTbl1 implements Serializable {
    private static final long serialVersionUID = 1L;

    /** testTbl1Id */
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    @SequenceGenerator(sequence = "TEST_TBL1_ID_SEQ", initialValue = 1, allocationSize = 1)
    @Column(name = "TEST_TBL1_ID")
    public Long testTbl1Id;

    // getter/setter...
}
```

### 2.4 GspGeneratorStrategy（命名戦略）

```java
/**
 * gsp-dba-maven-plugin互換の命名戦略。
 * テーブル名→クラス名、カラム名→フィールド名の変換ルールを定義。
 *
 * S2JDBC-GenのPersistenceConventionを代替する。
 */
public class GspGeneratorStrategy extends DefaultGeneratorStrategy {

    @Override
    public String getJavaClassName(Definition definition, Mode mode) {
        // テーブル名 → PascalCase（既存の命名規約を維持）
        // 例: TEST_TBL1 → TestTbl1, ORDER_DETAIL → OrderDetail
    }

    @Override
    public String getJavaMemberName(Definition definition, Mode mode) {
        // カラム名 → camelCase（既存の命名規約を維持）
        // 例: TEST_TBL1_ID → testTbl1Id, TEST_NAME → testName
    }

    @Override
    public String getJavaPackageName(Definition definition, Mode mode) {
        // rootPackage + "." + entityPackageName
        // 例: jp.co.tis.gsptest.entity.entity
    }
}
```

### 2.5 モード切替方式

```java
// GenerateEntity Mojo 内での切替
if ("doma".equals(entityType)) {
    generator = new GspDomaEntityGenerator();
} else {
    generator = new GspJpaEntityGenerator();
}
```

既存のMojoパラメータ `entityType` をそのまま使用。`"jpa"`（デフォルト）と `"doma"` の切替。

---

## 3. GenDialect体系の再設計

### 3.1 現状のS2JDBC-Gen GenDialect構造

```
S2JDBC-Gen GenDialect（org.seasar.extension.jdbc.gen.dialect）
├── H2GenDialect        ← ExtendedH2GenDialect が拡張
├── OracleGenDialect    ← ExtendedOracleGenDialect が拡張
├── PostgreGenDialect   ← ExtendedPostgreGenDialect が拡張
├── MysqlGenDialect     ← ExtendedMysqlGenDialect が拡張
├── MssqlGenDialect     ← ExtendedMssqlGenDialect が拡張
└── Db2GenDialect       ← ExtendedDb2GenDialect が拡張
```

各ExtendedGenDialectの役割:
- **columnTypeMap のカスタマイズ**: DB固有のデータ型→Java型マッピングの修正
  - `date` → `java.sql.Date`（TemporalType.DATE）
  - `timestamp` → `java.sql.Timestamp`（TemporalType.TIMESTAMP）
  - `boolean`/`bool`/`bit` → `boolean`
  - Oracle `number` → BigDecimal/Short/Integer/Long/BigInteger（精度依存）
  - DB2 `decimal` → boolean / BigDecimal（精度依存）

### 3.2 jOOQ ForcedTypeによる代替

jOOQの`ForcedType`設定で、S2JDBC-GenのGenDialect columnTypeMapと同等のカスタマイズが可能。

```java
public class GspColumnTypeMapper {

    /**
     * DB種別に応じたForcedType設定を生成する。
     * S2JDBC-Gen ExtendedXxxGenDialect の columnTypeMap をjOOQ ForcedTypeに変換。
     */
    public static List<ForcedType> createForcedTypes(String databaseProduct) {
        List<ForcedType> types = new ArrayList<>();

        // 全DB共通: date → java.sql.Date
        types.add(new ForcedType()
            .withName("DATE")
            .withIncludeTypes("date")
            .withUserType("java.sql.Date"));

        // 全DB共通: timestamp → java.sql.Timestamp
        types.add(new ForcedType()
            .withName("TIMESTAMP")
            .withIncludeTypes("timestamp.*|datetime|smalldatetime")
            .withUserType("java.sql.Timestamp"));

        // DB固有設定
        switch (databaseProduct) {
            case "Oracle":
                // Oracle number(p,s) → 精度依存の型変換
                // → jOOQのデフォルト型マッピングで概ね対応可能
                break;
            case "Db2":
                // DB2 decimal(1,0) → boolean
                break;
            // ...
        }
        return types;
    }
}
```

### 3.3 S2JDBC-Gen GenDialect → jOOQ マッピング一覧

| gsp ExtendedGenDialect | S2JDBC-Gen基底 | jOOQ対応 | 備考 |
|------------------------|---------------|---------|------|
| ExtendedH2GenDialect | H2GenDialect | `JDBCDatabase` (自動) | date,timestamp,boolean型のカスタマイズ |
| ExtendedOracleGenDialect | OracleGenDialect | `JDBCDatabase` (自動) | number精度→Java型マッピング |
| ExtendedPostgreGenDialect | PostgreGenDialect | `JDBCDatabase` (自動) | timestamptz,bool型のカスタマイズ |
| ExtendedMysqlGenDialect | MysqlGenDialect | `JDBCDatabase` (自動) | datetime,timestamp型のカスタマイズ |
| ExtendedMssqlGenDialect | MssqlGenDialect | `JDBCDatabase` (自動) | smalldatetime,datetime,bit型のカスタマイズ |
| ExtendedDb2GenDialect | Db2GenDialect | `JDBCDatabase` (自動) | decimal精度→boolean変換 |

`JDBCDatabase`はJDBC標準の`DatabaseMetaData`を使用するため、DB固有のGenDialect実装は不要。型マッピングのカスタマイズは`ForcedType`で対応する。

### 3.4 GenDialectRegistry の廃止

現状、各gsp Dialectクラス（H2Dialect, OracleDialect等）のコンストラクタで`GenDialectRegistry.register()`を呼んでいる:

```java
// H2Dialect.java (現状)
public H2Dialect() {
    GenDialectRegistry.deregister(org.seasar.extension.jdbc.dialect.H2Dialect.class);
    GenDialectRegistry.register(org.seasar.extension.jdbc.dialect.H2Dialect.class,
                                new ExtendedH2GenDialect());
}
```

jOOQ移行後はGenDialectRegistry自体が不要になり、各Dialectコンストラクタからこの呼び出しを除去する。

---

## 4. dicon設定廃止に伴う新しい設定方式の設計

### 4.1 現状のdicon設定

`GenerateEntity.executeMojoSpec()`が3つのdiconファイルを自動生成:
- `convention.dicon` — S2Container命名規約設定
- `jdbc.dicon` — S2Container JDBC接続設定
- `s2jdbc.dicon` — S2JDBC設定

これらはS2JDBC-Genの`GenerateEntityCommand`が内部的にS2Containerを起動してDI設定として読み込む。

### 4.2 jOOQでの代替

jOOQはDIコンテナを使用しない。プログラマティック設定（`Configuration`オブジェクト）で全設定を渡す。

```java
public class GspEntityGenerationConfig {

    /**
     * MojoパラメータからjOOQ Code Generation設定を構築する。
     * dicon設定ファイルは不要。
     */
    public static org.jooq.meta.jaxb.Configuration create(
            String jdbcUrl, String jdbcUser, String jdbcPassword, String jdbcDriver,
            String schemaName, String rootPackage, String entityPackageName,
            File javaFileDestDir, String entityType, boolean useAccessor,
            boolean useJSR310, String ignoreTableNamePattern,
            String genDialectClassName, int allocationSize) {

        return new org.jooq.meta.jaxb.Configuration()
            .withJdbc(new Jdbc()
                .withUrl(jdbcUrl)
                .withUser(jdbcUser)
                .withPassword(jdbcPassword)
                .withDriver(jdbcDriver))
            .withGenerator(new Generator()
                .withName("doma".equals(entityType)
                    ? GspDomaEntityGenerator.class.getName()
                    : GspJpaEntityGenerator.class.getName())
                .withStrategy(new Strategy()
                    .withName(GspGeneratorStrategy.class.getName()))
                .withDatabase(new Database()
                    .withName("org.jooq.meta.jdbc.JDBCDatabase")
                    .withInputSchema(schemaName)
                    .withExcludes(convertIgnorePattern(ignoreTableNamePattern))
                    .withForcedTypes(GspColumnTypeMapper.createForcedTypes(
                        detectDatabaseProduct(jdbcUrl))))
                .withGenerate(new Generate()
                    .withPojos(true)
                    .withDaos(false)
                    .withRecords(false)
                    .withFluentSetters(false)
                    .withJpaAnnotations("jpa".equals(entityType))
                    .withJpaVersion("3.0"))
                .withTarget(new Target()
                    .withPackageName(rootPackage + "." + entityPackageName)
                    .withDirectory(javaFileDestDir.getAbsolutePath())));
    }
}
```

### 4.3 dicon関連の変更点

| 現状 | Phase 2以降 |
|------|----------|
| `convention.dicon.ftl` テンプレート | 不要（削除） |
| `jdbc.dicon.ftl` テンプレート | 不要（削除） |
| `s2jdbc.dicon.ftl` テンプレート | 不要（削除） |
| `diconDir` Mojoパラメータ | 不要（deprecated化→Phase 3で削除） |
| FreeMarkerでdicon生成する処理 | 不要（削除） |

---

## 5. 統合設計: GenerateEntity Mojo改修

### 5.1 改修後のGenerateEntity.executeMojoSpec()

```java
@Override
protected void executeMojoSpec() throws MojoExecutionException, MojoFailureException {
    // dicon生成は不要（jOOQはDIコンテナを使わない）

    // jOOQ設定を構築
    org.jooq.meta.jaxb.Configuration jooqConfig =
        GspEntityGenerationConfig.create(
            url, adminUser, adminPassword, driver,
            schema, rootPackage, entityPackageName,
            javaFileDestDir, entityType, useAccessor,
            useJSR310, ignoreTableNamePattern,
            genDialectClassName, allocationSize);

    // jOOQ Code Generationを実行
    try {
        GenerationTool.generate(jooqConfig);
    } catch (Exception e) {
        throw new MojoExecutionException("Entity生成に失敗しました", e);
    }
}
```

### 5.2 削除されるS2JDBC-Gen依存クラス（Phase 3で実施）

| クラス | 役割 | 代替 |
|--------|------|------|
| `ExtendedGenerateEntityCommand` | GenerateEntityCommandの拡張 | `GspJpaEntityGenerator` / `GspDomaEntityGenerator` |
| `GspFactoryImpl` | Entity生成ファクトリ | jOOQ `Generator` + `GeneratorStrategy` |
| `DomaGspFactoryImpl` | Doma Entity生成ファクトリ | `GspDomaEntityGenerator` |
| `GspAttributeDescFactoryImpl` | 属性記述ファクトリ | jOOQ内蔵型マッピング + `ForcedType` |
| `JakartaEntityModelFactoryImpl` | Jakarta EEモデルファクトリ | jOOQ内蔵JPA annotation生成 |
| `JSR310AttributeModelFactoryImpl` | JSR310対応ファクトリ | jOOQ `ForcedType`設定 |
| `DbTableMetaReaderWithView` | VIEW対応メタデータリーダー | `JDBCDatabase` + `GspViewSupport` |
| `ExtendedH2GenDialect` | H2型マッピング | `ForcedType`設定 |
| `ExtendedOracleGenDialect` | Oracle型マッピング | `ForcedType`設定 |
| `ExtendedPostgreGenDialect` | PostgreSQL型マッピング | `ForcedType`設定 |
| `ExtendedMysqlGenDialect` | MySQL型マッピング | `ForcedType`設定 |
| `ExtendedMssqlGenDialect` | SQL Server型マッピング | `ForcedType`設定 |
| `ExtendedDb2GenDialect` | DB2型マッピング | `ForcedType`設定 |

### 5.3 既存Mojoパラメータとの互換性

| パラメータ | 現状 | Phase 2以降 | 互換性 |
|-----------|------|----------|--------|
| `rootPackage` | S2JDBC-Genに渡す | jOOQ Target.packageNameに渡す | 完全互換 |
| `entityPackageName` | S2JDBC-Genに渡す | jOOQ Target.packageNameに結合 | 完全互換 |
| `schema` | S2JDBC-Genに渡す | jOOQ Database.inputSchemaに渡す | 完全互換 |
| `entityType` | GspFactoryImpl/DomaGspFactoryImpl切替 | Generator切替 | 完全互換 |
| `useAccessor` | S2JDBC-Genに渡す | Generator内で制御 | 完全互換 |
| `useJSR310` | ExtendedGenerateEntityCommand | ForcedType設定で制御 | 完全互換 |
| `javaFileDestDir` | S2JDBC-Genに渡す | jOOQ Target.directoryに渡す | 完全互換 |
| `ignoreTableNamePattern` | S2JDBC-Genに渡す | jOOQ Database.excludesに変換 | 完全互換 |
| `entityTemplate` | FreeMarkerテンプレート指定 | Phase 2では未使用（jOOQ内蔵生成）。Phase 3で対応 | Phase 3で対応 |
| `templateFilePrimaryDir` | FreeMarkerテンプレートディレクトリ | Phase 2では未使用。Phase 3で対応 | Phase 3で対応 |
| `genDialectClassName` | GenDialect切替 | ForcedType設定で代替 | 仕様変更（Phase 3で移行ガイド提供） |
| `dialectClassName` | S2JDBC Dialect切替 | 不要（jOOQはJDBC URLから自動判定） | 不要化 |
| `diconDir` | dicon出力先 | 不要（jOOQはDI不使用） | deprecated化 |
| `allocationSize` | SequenceGenerator設定 | Generator内で@SequenceGeneratorに反映 | 完全互換 |
| `versionColumnNamePattern` | バージョンカラム判定 | Generator内で@Version判定に使用 | 完全互換 |

---

## 6. PoC実装計画

### 6.1 PoCスコープ

| 項目 | 対象 |
|------|------|
| DB | H2のみ（PoC段階） |
| Entity種別 | JPA Entity（`entityType=jpa`） |
| テストケース | 既存の`basic/h2`テストと同等のEntity生成 |
| 検証ポイント | 生成されたEntityが既存出力と同等であること |

### 6.2 pom.xml変更

```xml
<!-- jOOQ Code Generation（OSS版） -->
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen</artifactId>
    <version>3.19.16</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
    <version>3.19.16</version>
</dependency>
```

**注意**: jOOQ 3.19.x は Java 17+対応。プロジェクトはJava 17を使用しているため問題なし。

### 6.3 PoCで作成するファイル

```
src/main/java/jp/co/tis/gsp/tools/dba/entitygen/
├── GspEntityGenerationConfig.java    ← 設定構築
├── GspJpaEntityGenerator.java        ← JPA Entity生成
├── GspGeneratorStrategy.java         ← 命名戦略
└── GspColumnTypeMapper.java          ← 型マッピング

src/test/java/jp/co/tis/gsp/tools/dba/entitygen/
└── GspJpaEntityGeneratorTest.java    ← H2でのPoC検証テスト
```

### 6.4 PoCテスト内容

```java
@Test
public void testBasicH2EntityGeneration() {
    // 1. H2インメモリDBにテストテーブル作成
    // 2. jOOQ Code Generationでエンティティ生成
    // 3. 生成されたJavaファイルが期待通りの内容であることを検証
    //    - @Entity, @Table アノテーション
    //    - @Id, @Column, @GeneratedValue アノテーション
    //    - 正しいフィールド名・型
    //    - Serializable実装
}
```

---

## 7. 既存FreeMarkerテンプレートとの関係

### 7.1 Phase 2（PoC）でのアプローチ

Phase 2ではjOOQ内蔵のコード生成（`JavaGenerator.generatePojo()`）を使用する。既存のFreeMarkerテンプレート（`gsp_entity.ftl`, `gsp_doma_entity.ftl`）は使用しない。

jOOQのJavaGeneratorは独自のJavaWriterを使用してコードを出力する仕組みであり、S2JDBC-GenのFreeMarkerベースの生成とは異なるアプローチとなる。

### 7.2 Phase 3での対応

既存のFreeMarkerテンプレートを使いたいユーザー向けに、jOOQのメタデータ→既存FreeMarkerモデルへの変換レイヤーを提供する可能性がある。ただしこれはPhase 3で判断する。

### 7.3 テンプレートカスタマイズへの影響

| 現状 | Phase 2以降 |
|------|-----------|
| `entityTemplate`パラメータでFreeMarkerテンプレート指定 | jOOQのJavaGenerator内でコード生成（テンプレート不使用） |
| `templateFilePrimaryDir`でカスタムテンプレート指定 | Generator拡張でカスタマイズ |
| 既存テンプレートのマクロ（`printAttrAnnotations`等） | Generator内のJavaWriterで同等出力 |

---

## 8. VIEW対応設計

### 8.1 現状のVIEW対応

`DbTableMetaReaderWithView`が以下の機能を提供:
- TABLE + VIEW の両方をメタデータ読取対象にする
- VIEWの場合、`ViewAnalyzer`で基底テーブルを特定しPK/FKを取得
- 単純VIEW（単一テーブルからのSELECT）のみ対応

### 8.2 jOOQでのVIEW対応

`JDBCDatabase`はデフォルトでVIEWも読み取る。ただし、VIEWからPK/FKを推定する機能は独自実装が必要。

```java
public class GspViewSupport {
    /**
     * 既存のViewAnalyzerを活用し、VIEWの基底テーブルからPK/FK情報を取得する。
     * jOOQ生成時にVIEWのEntityにも@Id等を付与するための情報を提供する。
     */
    public static ViewMetadata analyzeView(Connection conn, String viewName,
                                            String schemaName, Dialect dialect) {
        // ViewAnalyzer.parse()で基底テーブルを特定
        // 基底テーブルのPK/FKをVIEWカラムにマッピング
    }
}
```

---

## 9. リスクと緩和策

| リスク | 影響度 | 確率 | 緩和策 |
|--------|--------|------|--------|
| jOOQ OSS版でOracle/SQLServer/DB2のJDBCDatabase動作不良 | 高 | 低 | Phase 2はH2のみ。Phase 3で個別検証 |
| 生成Entityのフォーマットが既存と差異 | 中 | 高 | PoC段階で差異を洗い出し、Generator内で調整 |
| jOOQ JavaGeneratorの拡張性不足 | 中 | 低 | 最悪の場合、FreeMarkerテンプレートベースの独自生成に切替 |
| VIEW対応の複雑化 | 中 | 中 | 既存ViewAnalyzerを流用し、jOOQの型変換のみ利用 |
| jOOQバージョンアップによるAPI変更 | 低 | 中 | 安定版（3.19.x LTS）を使用 |
| `entityTemplate`カスタマイズ機能の互換性 | 中 | 確定 | Phase 3で移行ガイド提供。Generator拡張による代替手段を用意 |

---

## 10. 参照

- [jOOQ Code Generation](https://www.jooq.org/doc/latest/manual/code-generation/)
- [jOOQ Generated POJOs](https://www.jooq.org/doc/latest/manual/code-generation/codegen-pojos/)
- [jOOQ JPA Annotations](https://www.jooq.org/doc/latest/manual/code-generation/codegen-advanced/codegen-config-generate/codegen-generate-annotations/)
- [jOOQ Custom Generator Strategies](https://www.jooq.org/doc/latest/manual/code-generation/codegen-generatorstrategy/)
- [jOOQ Custom Code Sections](https://www.jooq.org/doc/latest/manual/code-generation/codegen-custom-code/)
- [jOOQ JDBCDatabase](https://www.jooq.org/doc/latest/manual/code-generation/codegen-configuration/)
- Seasar依存調査レポート（gsp-dba-maven-plugin Seasar依存調査）
