# リスクレビュー: Phase 2 jOOQ Code Generation PoC

**レビュアー**: テックリード / OSS戦略担当
**レビュー日**: 2026-02-14
**対象**: Phase 2設計書 + PoC実装

---

## 1. 総合評価

**評価: B（良好 — 戦略的リスクへの対策が鍵）**

PoCは技術的に健全であり、`JDBCDatabase`採用によってjOOQ OSS版の最大の制約（商用DB対応）を回避する賢明な設計判断が行われている。ただし、jOOQへの長期的依存に伴うリスクと、Phase 3での実装リスクに対して事前の緩和策を講じる必要がある。

---

## 2. jOOQ OSS版のDB対応制限

### 2.1 jOOQエディション別機能制限

| 機能 | OSS版（Apache 2.0） | 商用版（有償） |
|------|---------------------|---------------|
| OracleDatabase | **非対応** | 対応 |
| SQLServerDatabase | **非対応** | 対応 |
| DB2Database | **非対応** | 対応 |
| PostgresDatabase | 対応 | 対応 |
| MySQLDatabase | 対応 | 対応 |
| H2Database | 対応 | 対応 |
| **JDBCDatabase** | **対応** | 対応 |
| JavaGenerator | 対応 | 対応 |
| Code Generation API | 対応 | 対応 |

### 2.2 PoCでの対策: JDBCDatabase

Phase 2 PoCでは`org.jooq.meta.jdbc.JDBCDatabase`を採用している。これは`java.sql.DatabaseMetaData`（JDBC標準API）を使用してメタデータを取得するため、**OSS版でもOracle/SQL Server/DB2を含む全JDBC対応DBで動作する**。

**技術的根拠**:
- `JDBCDatabase`はjOOQ OSS版に含まれるクラス
- DB固有の`XxxDatabase`（OracleDatabase等）とは異なり、JDBC標準APIのみを使用
- gsp-dba-maven-pluginの6対象DB（H2, Oracle, PostgreSQL, MySQL, SQL Server, DB2）は全てJDBC対応

### 2.3 JDBCDatabase採用のトレードオフ

| 項目 | JDBCDatabase | DB固有Database |
|------|-------------|---------------|
| ライセンス | **OSS（Apache 2.0）** | 商用版必須（Oracle/SQL Server/DB2） |
| メタデータ精度 | JDBC標準APIに依存（DB固有の拡張情報は取得不可） | DB固有システムビューを直接参照（最大精度） |
| パフォーマンス | 標準的 | DB固有最適化あり |
| ストアドプロシージャ | 基本情報のみ | 完全なメタデータ |
| ユーザー定義型 | 限定的 | 完全対応 |

**影響評価**: gsp-dba-maven-pluginの用途（Entity生成）では、テーブル・カラム・PK・FK・インデックスのメタデータがあれば十分であり、これらは`DatabaseMetaData`で標準的に取得可能。**JDBCDatabaseで実用上の問題はない**。

### 2.4 リスク: JDBCDatabaseのDB固有の挙動差異

| DB | 既知の挙動差異 | 影響度 |
|----|--------------|--------|
| Oracle | `DatabaseMetaData.getColumns()`でNUMBER型のprecision/scaleが不正確な場合がある | **中** |
| SQL Server | スキーマ名のデフォルトが`dbo`（`PUBLIC`ではない） | **低** |
| DB2 | カラム名が大文字固定で返される（命名規約に影響） | **低** |
| MySQL | テーブルコメントの取得が`DatabaseMetaData`では不完全 | **低** |
| PostgreSQL | ユーザー定義型（ENUM等）が`VARCHAR`として返される | **低** |
| H2 | 概ね標準的（差異小） | **低** |

**緩和策**: Phase 3で6DB全てのテストを実行し、差異を検出・対応する。

---

## 3. ライセンスリスク

### 3.1 jOOQ OSS版のライセンス

| 項目 | 詳細 |
|------|------|
| ライセンス | Apache License 2.0 |
| Maven Artifact | `org.jooq:jooq-codegen` (Maven Central公開) |
| 再配布 | 自由（Apache 2.0準拠） |
| 商用利用 | 制限なし |
| ソース変更 | 自由（変更の明記が必要） |
| 特許条項 | Apache 2.0に含まれる |

### 3.2 ライセンスの境界

**安全な領域（OSS版で利用可能）**:
- `org.jooq.meta.jdbc.JDBCDatabase` — メタデータ取得
- `org.jooq.codegen.JavaGenerator` — コード生成基盤
- `org.jooq.codegen.GeneratorStrategy` — 命名戦略
- `org.jooq.meta.jaxb.Configuration` — 設定API
- `org.jooq.meta.jaxb.ForcedType` — 型マッピング設定

**商用版が必要な領域（PoCでは使用していない）**:
- `org.jooq.meta.oracle.OracleDatabase`
- `org.jooq.meta.sqlserver.SQLServerDatabase`
- `org.jooq.meta.db2.DB2Database`
- `org.jooq.meta.xml.XMLDatabase`のDB固有拡張
- jOOQ DSL APIの商用DB方言（SQLDialect.ORACLE等）

### 3.3 PoC実装のライセンス適合性

PoCで使用しているjOOQクラスを全て検証した:

| クラス | パッケージ | ライセンス |
|--------|-----------|-----------|
| `JavaGenerator` | `org.jooq.codegen` | OSS |
| `DefaultGeneratorStrategy` | `org.jooq.codegen` | OSS |
| `JDBCDatabase` | `org.jooq.meta.jdbc` | OSS |
| `Configuration` | `org.jooq.meta.jaxb` | OSS |
| `ForcedType` | `org.jooq.meta.jaxb` | OSS |
| `TableDefinition` | `org.jooq.meta` | OSS |
| `ColumnDefinition` | `org.jooq.meta` | OSS |
| `DataTypeDefinition` | `org.jooq.meta` | OSS |

**結論: PoC実装は全てOSS版の範囲内。ライセンスリスクなし。**

### 3.4 将来のライセンスリスク

| リスク | 可能性 | 影響 | 緩和策 |
|--------|--------|------|--------|
| jOOQがOSS版を廃止 | **極低** | 高 | Apache 2.0で公開済みバージョンは永続的に利用可能。最悪の場合、最終OSS版をforkして使用可能 |
| JDBCDatabaseがOSS版から除外 | **極低** | 高 | 同上。代替として独自のDatabaseMetaData読取りクラスを実装 |
| OSS版のCode Generation APIが制限 | **低** | 中 | JavaGenerator拡張はOSS版の中核機能であり、除外の可能性は低い |
| jOOQのバージョンアップで非互換変更 | **中** | 中 | PoCの依存APIは安定したSPI。メジャーバージョンアップ時の動作確認テストをCIに組み込む |

---

## 4. 長期メンテナンス性

### 4.1 jOOQへの依存度分析

**依存しているjOOQ API層**:

| API層 | 依存度 | 安定性 | 代替困難度 |
|-------|--------|--------|-----------|
| Code Generation SPI（`JavaGenerator`, `GeneratorStrategy`） | **高** | 高（メジャーバージョン間で安定） | 高（代替ライブラリがない） |
| メタデータAPI（`TableDefinition`, `ColumnDefinition`） | **高** | 高 | 中（独自実装で代替可能だが工数大） |
| 設定API（`Configuration`, `Database`, `Generate`） | **中** | 高 | 低（設定構造の変更のみ） |
| 型マッピング（`ForcedType`） | **低** | 高 | 低（独自型マッピングで代替可能） |

### 4.2 jOOQバージョンアップ対応

**jOOQのリリースサイクル**:
- メジャーバージョン: 約2-3年ごと（3.14→3.15→...→3.19が現在最新）
- マイナーバージョン: 数ヶ月ごと
- パッチバージョン: 1-2ヶ月ごと

**バージョンアップ影響度**:

| 変更種別 | 影響 | 対応コスト |
|---------|------|-----------|
| パッチ（3.19.x → 3.19.y） | 通常影響なし | 低（テスト実行のみ） |
| マイナー（3.19 → 3.20） | APIの非推奨化が発生する可能性 | 低〜中 |
| メジャー（3.x → 4.x） | APIの変更あり。`JavaGenerator`のSPIが変わる可能性 | 中〜高 |

**推奨**: jOOQ 3.19.x系でバージョンを固定し、メジャーアップグレードは計画的に行う。

### 4.3 S2JDBC-Genとの比較: メンテナンスコスト

| 項目 | S2JDBC-Gen（現行） | jOOQ Code Generation（移行後） |
|------|-------------------|-------------------------------|
| 依存ライブラリの更新頻度 | **更新なし**（EOL） | 定期的（活発にメンテナンス） |
| セキュリティ修正 | **なし**（EOL） | あり（CVE対応） |
| Java新バージョン対応 | **なし**（Java 8で停止） | あり（Java 17+対応） |
| Jakarta EE対応 | **なし**（javax.persistence固定） | あり（jakarta.persistence対応） |
| バグ修正 | **自前で対応** | コミュニティ + 開発者による修正 |
| Maven Central公開 | **なし**（Seasar独自リポジトリ） | あり |
| ドキュメント | 限定的（日本語のみ） | 充実（英語、公式サイト） |

**結論**: jOOQへの依存は、EOLのS2JDBC-Genに依存し続けるリスクと比較して明確に優位。メンテナンスコストの「不確実性」はあるが、「確実にメンテナンスされない」状態よりも健全。

---

## 5. Phase 3で想定されるリスク

### 5.1 技術的リスク

| # | リスク | 影響度 | 発生確率 | 緩和策 |
|---|--------|--------|---------|--------|
| R1 | **Oracle NUMBER型のprecision推定精度** — JDBCDatabaseでのNUMBER(p,s)のメタデータ取得精度がDB固有DatabaseクラスよりS2JDBC-GenのExtendedOracleGenDialectと比較して劣る可能性 | **高** | **中** | Phase 3で Oracle DB実機テスト を行い、既存GenDialectの型マッピングとの差異を検証。差異がある場合、`GspColumnTypeMapper`にOracle固有のForcedType設定を追加 |
| R2 | **VIEW対応の複雑性** — `DbTableMetaReaderWithView`はVIEWの基底テーブルからPK/FK情報を推定する独自ロジックを持つ。jOOQのJDBCDatabaseではVIEWのPK情報が取得できない | **高** | **高** | `GspViewSupport`クラスを新設し、`ViewAnalyzer`の推定ロジックを移植。jOOQ Databaseのメタデータをpost-processで補完 |
| R3 | **Doma Entity生成の複雑性** — DomaのアノテーションはJPAと大きく異なる（`@org.seasar.doma.Entity`, `@Column`, `@Id`, `@GeneratedValue(strategy = GenerationType.SEQUENCE)`等）。既存の`DomaGspFactoryImpl`は`DomaEntityModelFactory`内部クラスで複雑な処理を行っている | **中** | **中** | `GspDomaEntityGenerator`を`GspJpaEntityGenerator`のパターンに従い実装。Domaアノテーション仕様を事前に整理し設計書に反映 |
| R4 | **FreeMarkerテンプレートカスタマイズの互換性断絶** — 既存ユーザーが`entityTemplate`パラメータでカスタムFreeMarkerテンプレートを使用している場合、jOOQ版では動作しない | **高** | **低**（カスタマイズ利用者は少数と推定） | 移行ガイド作成。`JavaGenerator`拡張による代替手段の提供。可能であればFreeMarkerテンプレート→Generator変換ツールの提供 |
| R5 | **6DB全対応のテスト負荷** — Phase 3では6DB全てでEntity生成の正確性を検証する必要がある。Oracle, SQL Server, DB2は環境構築が複雑 | **中** | **高** | Docker化されたDB環境をCI/CDに組み込む。最低限H2 + PostgreSQL + MySQLでのCI実行、Oracle/SQL Server/DB2は手動テストまたはTestcontainers使用 |

### 5.2 スケジュールリスク

| # | リスク | 影響 | 緩和策 |
|---|--------|------|--------|
| S1 | VIEW対応の見積もり超過 | Phase 3全体の遅延 | VIEW対応を独立サブタスクとし、Entity生成の基本機能と分離して開発 |
| S2 | DB固有テスト環境の構築遅延 | テスト不十分のままマージ | H2テストを最優先で完成させ、他DBのテストは段階的に追加 |
| S3 | 既存テストの予期しない失敗 | 回帰バグの発生 | Phase 2→Phase 3の段階移行。既存テストスイートを維持しながら新機能を追加 |

### 5.3 戦略的リスク

| # | リスク | 影響 | 緩和策 |
|---|--------|------|--------|
| P1 | **upstreamへのPR受け入れ拒否** — coastland/gsp-dba-maven-pluginの現メンテナーがjOOQ依存の追加を拒否する可能性 | 高 | fork先（kumagoro1202）で独立メンテナンス。upstreamへのPRはPhase 3完了後に提案し、受け入れを求める形で対話 |
| P2 | **jOOQ 4.xでの破壊的変更** — jOOQの次期メジャーバージョンでCode Generation SPIが大幅変更される可能性 | 中 | jOOQ 3.19.x系に固定。メジャーバージョンアップは計画的に対応。変更範囲を限定的にするため、jOOQ APIへの依存を4クラスに集約した現設計を維持 |
| P3 | **コミュニティの関心度低下** — gsp-dba-maven-plugin自体のユーザー数が減少し、メンテナンスの正当性が低下する可能性 | 低〜中 | Seasar2依存を完全除去することで、新規ユーザーの参入障壁を下げる。Maven Central単独での依存解決を実現し、利便性を向上させる |

---

## 6. 総合リスク評価マトリクス

```
          高  │  R4         R1,R2
              │             S1
    影響度    │  P1,P2      R3,R5
              │             S2,S3
          低  │  P3
              └──────────────────
                低    中    高
                   発生確率
```

### リリースブロッカー

以下はPhase 3完了前に必ず解決すべきリスク:

1. **R2（VIEW対応）**: 既存プロジェクトでVIEWベースのEntityを使用している場合、これが欠落すると致命的
2. **R1（Oracle NUMBER型）**: Oracle使用プロジェクトが多いため、型マッピングの精度保証は必須
3. **R5（6DB全テスト）**: 最低限、既存テストケースと同等のDB範囲でのテストパスが必要

---

## 7. リスク緩和策の優先順位

| 優先度 | 対策 | 対象リスク | 実装時期 |
|--------|------|----------|---------|
| **1（最高）** | Oracle DB実機テスト環境構築 + 型マッピング検証 | R1 | Phase 3序盤 |
| **2（高）** | GspViewSupport実装 + VIEW対応テスト | R2 | Phase 3序盤 |
| **3（高）** | Docker化DB環境 + CI/CDテストパイプライン | R5 | Phase 3並行 |
| **4（中）** | GspDomaEntityGenerator実装 | R3 | Phase 3中盤 |
| **5（中）** | FreeMarkerテンプレート移行ガイド作成 | R4 | Phase 3終盤 |
| **6（低）** | upstream PR戦略策定 | P1 | Phase 3完了後 |
| **7（低）** | jOOQバージョン固定 + アップグレード計画 | P2 | 継続的 |

---

## 8. 結論

Phase 2 PoCは`JDBCDatabase`採用によってjOOQ OSS版の最大のリスク（商用DB非対応）を巧みに回避しており、ライセンス面でのリスクは皆無と判断できる。

**最大のリスクはPhase 3のVIEW対応（R2）とOracle型マッピング精度（R1）**。これらはEntity生成の正確性に直結するため、Phase 3の最序盤で対応し、早期にリスクを解消することを推奨する。

長期的には、EOLのS2JDBC-Genへの依存を活発にメンテナンスされるjOOQに移行することで、セキュリティ・Java新バージョン対応・Jakarta EE対応の各面でプロジェクトの持続可能性が大幅に向上する。移行に伴うリスクは管理可能な範囲であり、Phase 3への進行を推奨する。
