---
name: yaml-resource-provider-generator
description: YAMLナレッジファイルからMCP Resource用のMarkdown生成Javaクラス（ResourceProvider）を自動生成する。「YAMLからResourceProviderを生成して」「MCP Resourceの実装を自動化して」「ナレッジYAMLをMCPリソースとして公開するコードを生成して」「新しいリソースURIパターンを追加して」「HandlerResourceProviderのようなクラスを自動生成して」といった要望に対応する。@Component + @PostConstruct + ObjectMapper(YAMLFactory) パターンに基づくSpring Beanの自動生成と、対応するユニットテスト・McpServerConfig統合コードを含む。
---

# YAML Resource Provider Generator — MCP Resource自動生成

## Overview

YAMLナレッジファイルのスキーマ定義から、MCP ResourceとしてMarkdownを提供するJavaクラス（ResourceProvider）を自動生成するスキル。生成されるコードは `@Component` + `@PostConstruct` + `ObjectMapper(YAMLFactory)` のパターンに従い、Spring Boot環境で即座に動作する。

**参考実装（実績）:**
- `HandlerResourceProvider.java`（162行）: handler-catalog.yaml + handler-constraints.yaml → 6アプリタイプのハンドラキューMarkdown生成
- `GuideResourceProvider.java`（308行）: 6 YAML → 6トピック別の開発ガイドMarkdown生成
- `McpServerConfig.java`（216行）: ResourceProvider → SyncResourceSpecification へのMCP登録

**主な用途:**
- 新しいMCPリソースURIパターンの追加（nablarch://pattern/*, nablarch://config/* 等）
- YAMLナレッジファイルをMCPクライアントに公開するためのプロバイダ実装
- 既存ResourceProviderの追加トピック拡張
- テストクラスの自動生成

**このスキルの特徴:**
- 実績あるパターン（HandlerResourceProvider / GuideResourceProvider）のテンプレート化
- YAMLスキーマ解析 → Javaクラス生成 → テスト生成 → Config統合の一気通貫
- FQCN・YAML構造の正確性を保証する検証ステップ
- Spring Boot + MCP Java SDK 0.17.x の規約に準拠

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「YAMLからResourceProviderを生成して」
- 「MCPリソースの実装を自動化して」
- 「nablarch://pattern/* のResourceProviderを作って」
- 「新しいリソースURIパターンを追加して」
- 「HandlerResourceProviderと同じパターンでクラスを生成して」
- 「ナレッジYAMLをMCPクライアントに公開したい」
- 「GuideResourceProviderに新しいトピックを追加して」
- YAMLナレッジファイルが既に存在し、それをMCP Resourceとして公開する必要がある場合
- 複数のYAMLファイルを読み込んでMarkdownに変換するResourceProviderが必要な場合

**トリガーキーワード**: ResourceProvider, MCP Resource, YAML, ナレッジファイル, リソースプロバイダ, Markdown生成, SyncResourceSpecification

## Input Format

```yaml
# 必須パラメータ
resource_name: "pattern"                    # リソース名（URI: nablarch://{resource_name}/{key}）
provider_class_name: "PatternResourceProvider"  # 生成するクラス名

# YAML定義
yaml_files:                                 # 読み込むYAMLファイル（1件以上）
  - path: "knowledge/design-patterns.yaml"  # クラスパス内のパス
    root_key: "patterns"                    # ルートキー（List or Map）
    fields:                                 # 使用するフィールド名
      - name: "name"
        type: "String"
        use: "key"                          # "key" | "content" | "metadata"
      - name: "description"
        type: "String"
        use: "content"
      - name: "problem"
        type: "String"
        use: "content"
      - name: "solution"
        type: "String"
        use: "content"
      - name: "code_example"
        type: "String"
        use: "content"
      - name: "category"
        type: "String"
        use: "metadata"

# トピック/キー定義
valid_keys:                                 # 有効なキー値（URI末尾）
  - "session-management"
  - "multi-db"
  - "csrf-protection"
  - "file-upload"

# 出力先
output_package: "com.tis.nablarch.mcp.resources"
output_dir: "src/main/java/com/tis/nablarch/mcp/resources"
test_dir: "src/test/java/com/tis/nablarch/mcp/resources"

# オプション
description: "Design pattern catalog for Nablarch applications"
content_type: "text/markdown"               # MIMEタイプ（デフォルト: text/markdown）
```

## Output Format

以下のファイルを生成する:

```
生成ファイル:
├── {output_dir}/{ProviderClassName}.java       # ResourceProvider本体
├── {test_dir}/{ProviderClassName}Test.java     # ユニットテスト
└── McpServerConfig.java への追加コード片        # Config統合スニペット
```

## Instructions

### Phase 1: YAML構造の解析

#### Step 1.1: YAMLファイルの読み込みと構造確認

```
【実行手順】

1. 指定されたYAMLファイルを全て読み込む:
   Read: {yaml_files[].path} の各ファイル

2. 各YAMLファイルの構造を確認:
   - ルートキー（root_key）の存在確認
   - ルートキーの値がList<Map> か Map<String, Object> か判定
   - 各エントリのフィールド名と型を確認
   - null許容フィールドの特定

3. valid_keys の検証:
   - YAMLデータ内に各キーに対応するデータが存在するか確認
   - 存在しないキーがある場合は警告（生成は続行）

4. 解析結果を記録:
   | YAMLファイル | ルートキー | 構造 | エントリ数 |
   |-------------|----------|------|----------|
   | design-patterns.yaml | patterns | List<Map> | 15 |
```

#### Step 1.2: Markdownテンプレートの設計

```
【設計ルール】

生成するMarkdownの構造を設計する。
HandlerResourceProvider / GuideResourceProvider の出力パターンを参考にする。

■ 基本構造:

# Nablarch {Resource名} — {キー名}

## Overview
{description}

## Details
{コンテンツフィールドをMarkdown変換}

---
*Source: {yaml_files}*

■ フィールドのMarkdown変換ルール:
  - use="content" のString型 → 本文に展開
  - use="content" のcode_example型 → ```java コードブロック
  - use="metadata" のString型 → メタデータテーブル行
  - use="key" → セクション見出しまたはフィルタ条件
  - List<String>型 → 箇条書き
  - List<Map>型 → テーブル

■ GuideResourceProvider方式（トピック別ビルドメソッド）:
  - 各キー値に対してbuild{Key}() メソッドを生成
  - switch文でキー→メソッドをディスパッチ
  - 各メソッド内でYAMLデータをフィルタリングしてMarkdown構築

■ HandlerResourceProvider方式（キー値でYAML内を検索）:
  - キー値をYAMLデータ内のMap.get(key)で検索
  - 単一のgetMarkdown(key)メソッドでMarkdown構築
  - 動的にコンテンツを組み立て
```

### Phase 2: Javaクラスの生成

#### Step 2.1: ResourceProvider本体の生成

```
【生成テンプレート】

以下のパターンに従ってJavaクラスを生成する。

package {output_package};

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;
import jakarta.annotation.PostConstruct;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.io.InputStream;
import java.util.*;

/**
 * {resource_name}/{"{"}key{"}"} リソースプロバイダ。
 *
 * <p>{description}</p>
 */
@Component
public class {ProviderClassName} {

    private static final Set<String> VALID_KEYS = Set.of(
            {valid_keys をカンマ区切りで列挙});

    // YAMLデータのフィールド
    {yaml_files ごとのフィールド定義}

    /**
     * ナレッジYAMLファイルを読み込み初期化する。
     */
    @PostConstruct
    @SuppressWarnings("unchecked")
    public void init() throws IOException {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        TypeReference<Map<String, Object>> mapType = new TypeReference<>() {};

        {yaml_files ごとの読み込みコード}
    }

    /**
     * 指定キーのコンテンツをMarkdown形式で返す。
     *
     * @param key リソースキー
     * @return Markdown形式のコンテンツ
     */
    public String getMarkdown(String key) {
        if (key == null || !VALID_KEYS.contains(key)) {
            return "# Unknown Key\n\nUnknown key: " + key
                    + "\n\nValid keys: " + String.join(", ", VALID_KEYS);
        }
        {キー値に応じたMarkdown構築ロジック}
    }

    // --- ヘルパーメソッド ---
    {Markdown構築用のヘルパーメソッド群}
}

【生成ルール】

1. @Component アノテーション必須
2. @PostConstruct でYAML読み込み
3. ObjectMapper(YAMLFactory) を使用
4. TypeReference<Map<String, Object>> でデシリアライズ
5. getClass().getClassLoader().getResourceAsStream() でリソース読み込み
6. @SuppressWarnings("unchecked") で型キャスト警告抑制
7. publicメソッドにはJavadocを付与
8. VALID_KEYS で入力バリデーション
9. 不正キーにはエラーメッセージ（有効キー一覧付き）を返す
10. *Source: {yaml_file}* をMarkdown末尾に付与
```

#### Step 2.2: YAML読み込みコードの生成

```
【パターンA: ルートキーがList<Map>の場合】

private List<Map<String, Object>> {fieldName};

// init() 内
try (InputStream is = getClass().getClassLoader()
        .getResourceAsStream("{yaml_path}")) {
    Map<String, Object> data = mapper.readValue(is, mapType);
    {fieldName} = (List<Map<String, Object>>) data.get("{root_key}");
}

【パターンB: ルートキーがMap<String, Object>の場合】

private Map<String, Object> {fieldName};

// init() 内
try (InputStream is = getClass().getClassLoader()
        .getResourceAsStream("{yaml_path}")) {
    {fieldName} = mapper.readValue(is, mapType);
}

【パターンC: 複数YAMLファイルの場合（GuideResourceProvider方式）】

GuideResourceProviderのloadList()ヘルパーメソッドを踏襲:

private List<Map<String, Object>> loadList(ObjectMapper mapper, String path,
        String key, TypeReference<Map<String, Object>> mapType) throws IOException {
    try (InputStream is = loadResource(path)) {
        Map<String, Object> data = mapper.readValue(is, mapType);
        return (List<Map<String, Object>>) data.get(key);
    }
}

private InputStream loadResource(String path) {
    return getClass().getClassLoader().getResourceAsStream(path);
}
```

#### Step 2.3: Markdown生成ロジックの生成

```
【フィールド別のMarkdown変換テンプレート】

■ String型（description等）:
  sb.append(item.get("description")).append("\n\n");

■ コード例（code_example等）:
  Object code = item.get("code_example");
  if (code != null) {
      sb.append("```java\n").append(code).append("```\n\n");
  }

■ FQCN:
  Object fqcn = item.get("fqcn");
  if (fqcn != null) {
      sb.append("- **FQCN**: `").append(fqcn).append("`\n");
  }

■ List<String>型（箇条書き）:
  List<String> items = (List<String>) item.get("related_handlers");
  if (items != null) {
      for (String s : items) {
          sb.append("- ").append(s).append("\n");
      }
  }

■ テーブル生成（List<Map>型）:
  sb.append("| Name | FQCN | Description |\n");
  sb.append("|------|------|-------------|\n");
  for (Map<String, Object> row : entries) {
      sb.append("| ").append(row.get("name"))
        .append(" | `").append(row.get("fqcn")).append("`")
        .append(" | ").append(row.get("description"))
        .append(" |\n");
  }

■ nullチェック:
  全フィールドアクセスにnullチェックを入れること。
  YAMLのフィールドは任意項目の場合がある。
```

### Phase 3: テストクラスの生成

#### Step 3.1: ユニットテストの生成

```
【テストテンプレート】

package {output_package};

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import static org.assertj.core.api.Assertions.assertThat;

class {ProviderClassName}Test {

    private {ProviderClassName} provider;

    @BeforeEach
    void setUp() throws Exception {
        provider = new {ProviderClassName}();
        provider.init();
    }

    @ParameterizedTest
    @ValueSource(strings = { {valid_keys をカンマ区切り} })
    void getMarkdown_validKey_returnsMarkdown(String key) {
        String result = provider.getMarkdown(key);

        assertThat(result).isNotNull();
        assertThat(result).isNotEmpty();
        assertThat(result).startsWith("#");
        assertThat(result).doesNotContain("Unknown");
    }

    @Test
    void getMarkdown_invalidKey_returnsError() {
        String result = provider.getMarkdown("nonexistent");

        assertThat(result).contains("Unknown");
        assertThat(result).contains("Valid keys");
    }

    @Test
    void getMarkdown_nullKey_returnsError() {
        String result = provider.getMarkdown(null);

        assertThat(result).contains("Unknown");
    }

    {valid_keys ごとの個別テスト: コンテンツ内容の検証}
}

【個別テストの例】

@Test
void getMarkdown_sessionManagement_containsExpectedContent() {
    String result = provider.getMarkdown("session-management");

    assertThat(result).contains("Session");
    assertThat(result).contains("Source:");
}
```

### Phase 4: McpServerConfig統合コードの生成

#### Step 4.1: Config登録スニペットの生成

```
【SyncResourceSpecification 登録パターン】

McpServerConfig.java に以下のコードを追加する。

1. Bean定義メソッドの引数に新しいProviderを追加:

   public List<McpServerFeatures.SyncResourceSpecification> nablarchResources(
           HandlerResourceProvider handlerProvider,
           GuideResourceProvider guideProvider,
           {ProviderClassName} {providerVarName}) {  // ← 追加

2. リソース登録を追加:

   // {resource_name} resources
   {valid_keys.forEach で create{Resource}ResourceSpec() 呼び出し}

3. ヘルパーメソッドを追加:

   private static McpServerFeatures.SyncResourceSpecification
           create{Resource}ResourceSpec(
                   String key, String name, String description,
                   {ProviderClassName} provider) {
       String uri = "nablarch://{resource_name}/" + key;
       return new McpServerFeatures.SyncResourceSpecification(
           new McpSchema.Resource(uri, name, description, "text/markdown", null),
           (exchange, request) -> new McpSchema.ReadResourceResult(
               List.of(new McpSchema.TextResourceContents(
                   request.uri(), "text/markdown",
                   provider.getMarkdown(key))))
       );
   }
```

### Phase 5: 品質検証

#### Step 5.1: 品質チェックリスト

```
□ コードチェック
  □ @Component アノテーションが付いている
  □ @PostConstruct でYAML読み込みが実装されている
  □ ObjectMapper(YAMLFactory) を使用している
  □ VALID_KEYS で入力バリデーションが実装されている
  □ 不正キーにエラーメッセージ（有効キー一覧）が返る
  □ publicメソッドにJavadocが付いている
  □ @SuppressWarnings("unchecked") が適切に付いている
  □ try-with-resources でInputStreamを閉じている
  □ nullチェックが全フィールドアクセスに入っている

□ テストチェック
  □ ParameterizedTest で全valid_keysをテストしている
  □ 不正キー・nullキーのテストがある
  □ Markdown出力に "#" で始まることを確認している
  □ Source行が含まれることを確認している

□ Config統合チェック
  □ SyncResourceSpecification のURI形式が正しい（nablarch://{resource}/{key}）
  □ content_typeが正しい（text/markdown）
  □ ヘルパーメソッドのシグネチャが既存パターンと一致

□ YAML整合性チェック
  □ 指定YAMLファイルが src/main/resources/ に存在する
  □ root_key がYAMLファイル内に存在する
  □ fields で指定したフィールドがYAMLデータに存在する
```

## Examples

### Example 1: PatternResourceProvider（design-patterns.yaml）

```yaml
# 入力
resource_name: "pattern"
provider_class_name: "PatternResourceProvider"
yaml_files:
  - path: "knowledge/design-patterns.yaml"
    root_key: "patterns"
    fields:
      - { name: "name", type: "String", use: "key" }
      - { name: "description", type: "String", use: "content" }
      - { name: "problem", type: "String", use: "content" }
      - { name: "solution", type: "String", use: "content" }
      - { name: "code_example", type: "String", use: "content" }
valid_keys: ["session-management", "multi-db", "csrf-protection"]
```

```java
// 生成される PatternResourceProvider.java（抜粋）
@Component
public class PatternResourceProvider {
    private static final Set<String> VALID_KEYS = Set.of(
            "session-management", "multi-db", "csrf-protection");

    private List<Map<String, Object>> patterns;

    @PostConstruct
    @SuppressWarnings("unchecked")
    public void init() throws IOException {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        try (InputStream is = getClass().getClassLoader()
                .getResourceAsStream("knowledge/design-patterns.yaml")) {
            Map<String, Object> data = mapper.readValue(is,
                    new TypeReference<Map<String, Object>>() {});
            patterns = (List<Map<String, Object>>) data.get("patterns");
        }
    }

    public String getMarkdown(String key) {
        if (key == null || !VALID_KEYS.contains(key)) {
            return "# Unknown Pattern\n\nUnknown pattern: " + key
                    + "\n\nValid patterns: " + String.join(", ", VALID_KEYS);
        }
        // key に一致するパターンをフィルタしてMarkdown生成
        ...
    }
}
```

### Example 2: ConfigResourceProvider（config-templates.yaml）

```yaml
# 入力
resource_name: "config"
provider_class_name: "ConfigResourceProvider"
yaml_files:
  - path: "knowledge/config-templates.yaml"
    root_key: "templates"
    fields:
      - { name: "name", type: "String", use: "key" }
      - { name: "description", type: "String", use: "content" }
      - { name: "template", type: "String", use: "content" }
      - { name: "app_type", type: "String", use: "metadata" }
valid_keys: ["web-component", "rest-component", "batch-component"]
```

## Anti-Patterns

1. **YAMLファイルパスのハードコード漏れ**
   - 生成後にYAMLパスを変更する場合、init()内のパスも変更が必要
   - 対策: 定数化して一箇所で管理

2. **型キャストの未チェック**
   - YAMLの構造が想定と異なる場合のClassCastException
   - 対策: @SuppressWarnings + 構造確認をPhase 1で実施

3. **nullチェックの漏れ**
   - YAMLにオプショナルフィールドがある場合のNullPointerException
   - 対策: 全フィールドアクセスにnullチェック

4. **テストのYAMLファイル不足**
   - src/test/resources/ にYAMLが配置されていないとテスト失敗
   - 対策: テストリソースの配置確認をチェックリストに含める

5. **McpServerConfig の肥大化**
   - 全リソースを1メソッドで登録するとメソッドが長大化
   - 対策: リソースタイプ別のヘルパーメソッドに分割

6. **Markdown内のパイプ文字未エスケープ**
   - テーブル内にYAMLデータの `|` が含まれるとテーブル構造が崩壊
   - 対策: テーブルセル内の `|` を `\|` にエスケープ
