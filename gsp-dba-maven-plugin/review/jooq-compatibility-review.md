# 互換性レビュー: Phase 2 jOOQ Code Generation PoC

**レビュアー**: Maven Pluginメンテナー / OSSコントリビューター
**レビュー日**: 2026-02-14
**対象**: Phase 2設計書 + PoC実装

---

## 1. 総合評価

**評価: B（良好 — 互換性面で注意が必要）**

PoCの設計は既存のMavenゴール名・パラメータの大部分を維持しており、プラグインとしてのインターフェースは概ね下位互換。ただし、生成されるEntityクラスのフォーマット差異と、一部パラメータの仕様変更が既存ユーザーに影響する。

---

## 2. 既存ユーザーのMaven設定への影響

### 2.1 Mavenゴール名

| ゴール | 影響 |
|--------|------|
| `generate-entity` | **変更なし** |
| `execute-ddl` | **変更なし** |
| `generate-ddl` | **変更なし** |
| `load-data` | **変更なし** |
| `export-schema` | **変更なし** |
| `import-schema` | **変更なし** |

**結論: ゴール名に影響なし（完全互換）**

### 2.2 Maven設定パラメータ

| パラメータ | 互換性 | 詳細 |
|-----------|--------|------|
| `rootPackage` | **完全互換** | そのまま使用可能 |
| `entityPackageName` | **完全互換** | そのまま使用可能 |
| `schema` | **完全互換** | そのまま使用可能 |
| `entityType` | **完全互換** | "jpa"/"doma" がそのまま使用可能 |
| `javaFileDestDir` | **完全互換** | そのまま使用可能 |
| `ignoreTableNamePattern` | **概ね互換** | 正規表現の解釈がjOOQのexcludesに変換される。大部分のパターンは動作するが、一部の複雑な正規表現で差異が出る可能性 |
| `useAccessor` | **Phase 3で対応** | Phase 2では未実装。Phase 3で対応 |
| `useJSR310` | **Phase 3で対応** | Phase 2の設定構築には含まれるが、Generator内での反映はPhase 3 |
| `allocationSize` | **Phase 3で対応** | Phase 2ではデフォルト値1でハードコード。Phase 3でMojoパラメータ連携 |
| `versionColumnNamePattern` | **Phase 3で対応** | Phase 2では@Version未対応 |
| `genDialectClassName` | **仕様変更** | S2JDBC-Gen GenDialectクラス名を指定するパラメータ。jOOQ版では不要化（JDBCDatabaseが自動判定）。廃止を推奨 |
| `dialectClassName` | **仕様変更** | S2JDBC Dialectクラス名を指定するパラメータ。jOOQ版では不要化。廃止を推奨 |
| `diconDir` | **不要化** | jOOQはDIコンテナ不使用。deprecated化を推奨 |
| `entityTemplate` | **仕様変更** | FreeMarkerテンプレートファイルを指定するパラメータ。jOOQ版ではJavaGenerator拡張で代替。**既存カスタマイズが使えなくなる** |
| `templateFilePrimaryDir` | **仕様変更** | カスタムテンプレートディレクトリ。entityTemplateと同様に影響あり |

### 2.3 影響サマリ

| カテゴリ | パラメータ数 | 影響 |
|---------|------------|------|
| 完全互換 | 5 | そのまま使用可能 |
| Phase 3で対応 | 4 | 機能として存続するが、Phase 2では未反映 |
| 仕様変更 | 4 | 設定変更が必要。移行ガイド必須 |
| 不要化 | 1 | deprecated→削除 |

---

## 3. 生成されるEntityクラスの形式互換性

### 3.1 出力形式の差異分析

**既存（S2JDBC-Gen版）の出力例** (`basic/h2/expected/output/TestTbl1.java`に相当):
```java
@Generated("GSP")
@Entity
@Table(name = "TEST_TBL1")
public class TestTbl1 implements Serializable {
    private static final long serialVersionUID = 1L;

    /** testTbl1Idプロパティ */
    @Id
    @GeneratedValue(generator = "TEST_TBL1_ID_SEQ", strategy = GenerationType.AUTO)
    @SequenceGenerator(name = "TEST_TBL1_ID_SEQ", sequenceName = "TEST_TBL1_ID_SEQ",
                       initialValue = 1, allocationSize = 1)
    @Column(name = "TEST_TBL1_ID", nullable = false, unique = true)
    public Long testTbl1Id;
    ...
}
```

**jOOQ版（Phase 2 PoC）の出力**:
```java
@Generated("GSP")
@Entity
@Table(name = "TEST_TBL1")
public class TestTbl1 implements Serializable {
    private static final long serialVersionUID = 1L;

    /** testTbl1Idプロパティ */
    @Id
    @GeneratedValue(generator = "TEST_TBL1_ID_SEQ", strategy = GenerationType.AUTO)
    @SequenceGenerator(name = "TEST_TBL1_ID_SEQ", sequenceName = "TEST_TBL1_ID_SEQ",
                       initialValue = 1, allocationSize = 1)
    @Column(name = "TEST_TBL1_ID", nullable = false, unique = true)
    public Long testTbl1Id;
    ...
}
```

### 3.2 確認された差異

| 項目 | 既存（S2JDBC-Gen） | jOOQ版 | 影響度 |
|------|-------------------|--------|--------|
| import文の順序 | S2JDBC-Gen依存の順序 | jOOQ JavaWriter依存の順序 | **低** — コンパイルに影響なし |
| @Column属性の順序 | name, length, nullable, unique | 同一順序を維持 | **低** |
| Javadocコメント形式 | `/** xxxプロパティ */` | `/** xxxプロパティ */` | **なし** — 同一形式 |
| @Version | versionColumnNamePatternで自動判定 | Phase 2未対応 | **中** — Phase 3で対応 |
| 外部キー関連 | @JoinColumn, @ManyToOne等 | Phase 2未対応 | **中** — Phase 3で対応 |
| 複合ユニーク制約 | @UniqueConstraint | Phase 2未対応 | **低** — Phase 3で対応 |
| useAccessor | getter/setter生成 | Phase 2未対応 | **中** — Phase 3で対応 |

### 3.3 コンパイル互換性

生成されたEntityクラスは既存プロジェクトで**そのままコンパイル可能**。以下の条件を満たす限り、既存コードへの影響はない:

1. Jakarta Persistence API（`jakarta.persistence.*`）を使用していること（既にPhase 1で移行済み）
2. publicフィールドへの直接アクセスを使用していること（デフォルト）
3. `@Generated("GSP")`の存在に依存するコードがないこと

---

## 4. 移行ガイドの必要性評価

### 4.1 必須の移行ガイド項目

1. **genDialectClassName パラメータの廃止**
   - 現在: `<genDialectClassName>jp.co.tis.gsp.tools.dba.dialect.ExtendedH2GenDialect</genDialectClassName>`
   - 移行後: パラメータ自体が不要。DB固有の型マッピングはjOOQが自動処理
   - 対応: `<genDialectClassName>`を指定してもwarning出力し無視する互換性レイヤーの提供を推奨

2. **entityTemplate / templateFilePrimaryDir パラメータの代替**
   - 現在: FreeMarkerテンプレートでEntity出力をカスタマイズ
   - 移行後: `JavaGenerator`を拡張した独自Generatorクラスの作成
   - 対応: カスタムテンプレートを使用しているユーザー向けの移行手順書を作成

3. **diconDir パラメータの廃止**
   - 現在: `<diconDir>target/classes</diconDir>`
   - 移行後: 不要
   - 対応: 指定してもwarning出力し無視する互換性レイヤー

### 4.2 移行ガイド不要な項目

- `rootPackage`, `entityPackageName`, `schema`, `entityType`, `javaFileDestDir` → 変更なし
- `ignoreTableNamePattern` → 大部分のパターンがそのまま動作

---

## 5. pom.xml依存関係への影響

### 5.1 追加される依存

```xml
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen</artifactId>
    <version>3.19.22</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
    <version>3.19.22</version>
</dependency>
```

### 5.2 将来的に除去される依存（Phase 3以降）

```xml
<!-- S2JDBC-Gen完全置換後に除去 -->
<dependency>
    <groupId>org.seasar.container</groupId>
    <artifactId>s2-framework</artifactId>
</dependency>
<dependency>
    <groupId>org.seasar.container</groupId>
    <artifactId>s2-extension</artifactId>
</dependency>
<dependency>
    <groupId>org.seasar.container</groupId>
    <artifactId>s2-tiger</artifactId>
</dependency>
<dependency>
    <groupId>org.seasar.container</groupId>
    <artifactId>s2jdbc-gen</artifactId>
</dependency>
```

### 5.3 Seasar Maven Repository除去

`<repository>` から `maven.seasar.org` を除去可能。jOOQはMaven Centralに公開されており、追加リポジトリ不要。

### 5.4 依存サイズの変化

| ライブラリ | サイズ（概算） | 備考 |
|-----------|-------------|------|
| **追加**: jooq-codegen | ~1.5 MB | |
| **追加**: jooq-meta | ~1.2 MB | |
| **追加**: jooq (core) | ~6 MB | jooq-codegenの推移的依存 |
| **除去**: s2-framework | ~1.8 MB | Phase 3で除去 |
| **除去**: s2-extension | ~0.6 MB | Phase 3で除去 |
| **除去**: s2-tiger | ~0.2 MB | Phase 3で除去 |
| **除去**: s2jdbc-gen | ~0.8 MB | Phase 3で除去 |
| **差分** | +5.3 MB | jOOQ coreが大きい |

PluginのJARサイズは約5MB増加するが、実行時の機能性向上（活発にメンテナンスされているOSS）を考慮すると許容範囲。

---

## 6. バージョニング推奨

### メジャーバージョンアップとしてリリース

S2JDBC-Genの除去は**破壊的変更**を含むため、`6.0.0` としてリリースすることを推奨。

- **5.x → 6.0.0** の変更点:
  - S2JDBC-Gen（Seasar2）依存の完全除去
  - Entity生成エンジンのjOOQ Code Generationへの移行
  - `genDialectClassName`, `dialectClassName`, `diconDir` パラメータの廃止
  - `entityTemplate`, `templateFilePrimaryDir` パラメータの仕様変更
  - Seasar Maven Repository (`maven.seasar.org`) の不要化

---

## 7. 結論

Phase 2 PoCは既存ユーザーへの影響を最小限に抑えながらS2JDBC-Gen脱却の道筋を示した。主要なMavenパラメータの互換性は維持されており、生成されるEntityクラスの基本構造も同等である。

**最大のリスクは`entityTemplate`カスタマイズの互換性**。FreeMarkerテンプレートを独自カスタマイズしているユーザーには移行コストが発生する。移行ガイドの作成と、可能であればFreeMarkerテンプレートへのブリッジレイヤーの提供を推奨する。

メジャーバージョンアップ（6.0.0）としてリリースし、5.x系はセキュリティ修正のみのメンテナンスモードとすることで、段階的な移行をサポートできる。
