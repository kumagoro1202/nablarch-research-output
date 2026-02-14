# アーキテクチャレビュー: Phase 2 jOOQ Code Generation PoC

**レビュアー**: Javaアーキテクト（15年経験）
**レビュー日**: 2026-02-14
**対象**: Phase 2設計書 + PoC実装

---

## 1. 総合評価

**評価: B+（良好 — 一部改善が必要）**

Phase 2 PoCは「jOOQのJDBCDatabaseとカスタムJavaGeneratorでS2JDBC-Genを代替できる」ことを実証した。設計方針は合理的であり、S2JDBC-Genの複雑なコマンドパイプラインを排除できる点は大きな改善である。

ただし、以下の点でPhase 3に向けた改善が必要。

---

## 2. 設計の妥当性

### 2.1 良い点

| 項目 | 評価 |
|------|------|
| `JDBCDatabase`の選択 | **優** — OSS版で全6DB対応。DB固有メタデータ実装への依存を回避 |
| カスタムJavaGenerator | **優** — S2JDBC-Genの12クラス体系を4クラスに集約。複雑度の大幅削減 |
| GspGeneratorStrategy | **良** — 命名規約の集約化。PersistenceConventionの過剰な抽象化を排除 |
| GspEntityGenerationConfig | **優** — dicon設定の完全排除。プログラマティック設定で可読性向上 |
| テスト設計 | **良** — H2インメモリDBを使った高速テスト。CI環境での実行が容易 |

### 2.2 懸念点

| 項目 | 評価 | 詳細 |
|------|------|------|
| 型マッピングのハードコード | **要改善** | `GspJpaEntityGenerator.getJavaType()`でDB型→Java型をswitch文でハードコード。拡張性に欠ける |
| VIEW対応の未実装 | **要対応** | Phase 2スコープ外だが、Phase 3では`DbTableMetaReaderWithView`の機能を移植する必要あり |
| FreeMarkerテンプレート非対応 | **要検討** | 既存ユーザーのカスタムテンプレートが使えなくなる。移行パスの提供が必要 |
| 生成コードの差異 | **要検討** | 既存S2JDBC-Gen生成コードとの差異が移行障壁になる可能性 |

---

## 3. SOLID原則への準拠

### 3.1 単一責任原則 (SRP)

| クラス | 評価 |
|--------|------|
| `GspJpaEntityGenerator` | **要改善** — 型マッピング（`getJavaType`）と生成ロジック（`generatePojo`）が同一クラス。型マッピングは分離すべき |
| `GspGeneratorStrategy` | **良好** — 命名変換に責務が限定されている |
| `GspEntityGenerationConfig` | **良好** — 設定構築に責務が限定されている |
| `GspColumnTypeMapper` | **良好** — 型マッピング設定に責務が限定。ただし現状はJSR310のみでスコープが狭い |

**推奨**: `GspJpaEntityGenerator`から型マッピングロジックを`GspColumnTypeMapper`に移動し、Generator自体は生成ロジックのみに集中させる。

### 3.2 オープン・クローズド原則 (OCP)

- **良好**: `GspJpaEntityGenerator`と`GspDomaEntityGenerator`（未実装）でJPA/Doma切替可能
- **改善余地**: 型マッピングがswitch文で書かれているため、新DBの型追加時にソースコード修正が必要。Strategy PatternまたはMap<String, Function>で外部化を検討

### 3.3 リスコフの置換原則 (LSP)

- **良好**: `JavaGenerator`を正しく拡張している。`generatePojo()`のオーバーライドはjOOQの設計意図に沿っている

### 3.4 インターフェース分離原則 (ISP)

- **良好**: jOOQのGenerator/GeneratorStrategy/Database SPIを適切に使い分けている

### 3.5 依存性逆転原則 (DIP)

- **要改善**: `GspJpaEntityGenerator`が`DataTypeDefinition`の具象クラスに依存。型マッピングを抽象化し、テスト容易性を向上させるべき

---

## 4. S2JDBC-Genとの機能カバレッジ

### 4.1 Phase 2で実現された機能

| S2JDBC-Gen機能 | PoC実装状態 | 備考 |
|----------------|-----------|------|
| テーブルメタデータ読取 | **実装済** | JDBCDatabaseが担当 |
| JPA Entity生成（@Entity, @Table, @Column, @Id） | **実装済** | GspJpaEntityGeneratorが担当 |
| @GeneratedValue（IDENTITY/SEQUENCE） | **実装済** | PKカラムに自動付与 |
| @SequenceGenerator | **実装済** | カラム名_SEQ形式 |
| PascalCase/camelCase命名規約 | **実装済** | GspGeneratorStrategyが担当 |
| ignoreTableNamePattern | **実装済** | jOOQ excludesに変換 |
| Serializable実装 | **実装済** | |
| publicフィールド | **実装済** | |
| @Generated("GSP") | **実装済** | |
| 基本型マッピング | **実装済** | VARCHAR→String, INTEGER→Integer等 |

### 4.2 Phase 3で対応が必要な機能

| S2JDBC-Gen機能 | 重要度 | 対応方針 |
|----------------|--------|---------|
| Doma Entity生成 | **高** | GspDomaEntityGenerator実装 |
| VIEW対応（基底テーブルからPK/FK推定） | **高** | GspViewSupport実装（ViewAnalyzer流用） |
| @Version（バージョンカラム） | **高** | versionColumnNamePatternで判定 |
| useAccessor（getter/setter） | **中** | Generator内で切替 |
| JSR310対応（LocalDate/LocalDateTime） | **中** | ForcedTypeで対応（設計済み） |
| 外部キー→@JoinColumn, @ManyToOne等 | **中** | jOOQのFK情報を利用 |
| DBコメント→Javadoc | **低** | jOOQのDatabaseMetaData.REMARKS利用 |
| 複合ユニーク制約→@UniqueConstraint | **低** | jOOQのUniqueKey情報を利用 |
| カスタムFreeMarkerテンプレート | **低** | 移行ガイド提供、代替手段としてGenerator拡張 |

---

## 5. 拡張性評価

### 5.1 将来のDB追加

**評価: 良好**

`JDBCDatabase`は標準JDBC `DatabaseMetaData`を使用するため、JDBC対応のDBであれば追加設定なしで動作する。DB固有の型マッピングのみ`GspColumnTypeMapper`に追加すればよい。

S2JDBC-Gen時代は新DBの追加に以下が必要だった:
1. `ExtendedXxxGenDialect`クラスの新規作成
2. `XxxDialect`のコンストラクタで`GenDialectRegistry.register()`
3. テスト用Dialectクラス

jOOQ版では型マッピング設定の追加のみで対応可能。

### 5.2 新しいEntity形式対応

**評価: 良好**

新しいEntity形式（例: Records Pattern対応Entity、Kotlin Data Class）を追加する場合:
1. `JavaGenerator`を拡張した新しいGeneratorクラスを作成
2. `GspEntityGenerationConfig`でジェネレータを切替

S2JDBC-Gen時代は`FactoryImpl`の拡張が必要で、`EntityModelFactory`, `AttributeModelFactory`, `EntityDescFactory`等の複数インターフェースの実装が必要だった。jOOQ版では単一の`JavaGenerator`拡張で済む。

---

## 6. 推奨事項

### Phase 3に向けた優先順位

1. **[高] 型マッピングの外部化**: `GspJpaEntityGenerator.getJavaType()`をMapベースに変更し、DB固有設定を`GspColumnTypeMapper`に集約
2. **[高] VIEW対応**: `GspViewSupport`を実装し、`ViewAnalyzer`の機能を移植
3. **[高] Doma Entity生成**: `GspDomaEntityGenerator`を実装
4. **[中] バージョンカラム対応**: `versionColumnNamePattern`による`@Version`付与
5. **[中] 関連モデル対応**: FK→`@JoinColumn`, `@ManyToOne`, `@OneToMany`
6. **[低] テンプレート移行ガイド**: 既存のFreeMarkerテンプレートカスタマイズからの移行方法をドキュメント化

---

## 7. 結論

Phase 2 PoCは「jOOQでS2JDBC-Genを代替できる」ことを十分に実証した。アーキテクチャ面では、S2JDBC-Genの12クラスに跨る複雑なパイプラインを4クラスの簡潔な構造に置き換えることに成功している。

Phase 3では型マッピングの外部化、VIEW対応、Doma Entity生成の3つが最優先課題。これらが完了すれば、S2JDBC-Genの完全除去が実現可能である。
