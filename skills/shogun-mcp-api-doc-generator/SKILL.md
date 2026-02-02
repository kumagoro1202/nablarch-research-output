---
name: mcp-api-doc-generator
description: MCPサーバーのソースコードを解析し、API仕様書・セットアップガイドを自動生成する。「MCPサーバーのAPI仕様書を生成して」「ソースコードからドキュメントを自動生成して」「Tool/Resource/Promptの仕様を文書化して」「MCPサーバーのセットアップガイドを作成して」「ソースとドキュメントの整合性を検証して」「@Tool/@Resourceの定義からMarkdownを生成して」といった要望に対応する。Javaソースコード解析（@Tool, SyncResourceSpecification, SyncPromptSpecification）→ パラメータ・説明抽出 → Markdown仕様書生成 → ソース-ドキュメント整合性検証の包括的ドキュメント生成スキル。
---

# MCP API Doc Generator — MCPサーバーAPI仕様書自動生成

## Overview

MCPサーバーのJavaソースコードを静的解析し、Tool・Resource・Promptの定義情報を抽出してAPI仕様書およびセットアップガイドをMarkdown形式で自動生成するスキル。さらに、生成されたドキュメントとソースコードの整合性を自動検証する機能を提供する。

**参考実装（実績）:**
- `api-specification.md`: 2 Tools, 12 Resources, 6 Promptsの完全仕様書
- `setup-guide.md`: MCPサーバーのビルド・実行・テスト・統合手順
- ソースコード:
  - `SearchApiTool.java` / `ValidateHandlerQueueTool.java`: @Toolアノテーション + @ToolParam
  - `McpServerConfig.java`: SyncResourceSpecification + SyncPromptSpecification の登録
  - `HandlerResourceProvider.java` / `GuideResourceProvider.java`: リソースプロバイダ

**主な用途:**
- MCPサーバーのAPI仕様書の自動生成・更新
- セットアップガイドの自動生成
- ソースコードとドキュメントの整合性検証
- 新しいTool/Resource/Prompt追加時のドキュメント自動更新
- CI/CDパイプラインでのドキュメント鮮度チェック

**このスキルの特徴:**
- Javaソースの静的解析（@Tool, @ToolParam, SyncResourceSpecification, SyncPromptSpecification）
- ソースコードからパラメータ名・型・説明・必須フラグを自動抽出
- API仕様書テンプレートに基づくMarkdown自動生成
- ソースコード↔ドキュメント整合性の自動検証（diff検出）
- セットアップガイドの build.gradle.kts / application.yml からの自動生成

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「MCPサーバーのAPI仕様書を生成して」
- 「ソースコードからドキュメントを自動生成して」
- 「Tool/Resource/Promptの仕様を文書化して」
- 「MCPサーバーのセットアップガイドを作成して」
- 「ソースとドキュメントの整合性を検証して」
- 「新しいToolを追加したのでAPI仕様書を更新して」
- 「@Tool定義からMarkdownを生成して」
- MCPサーバーの機能追加後にドキュメントを最新化する必要がある場合
- ドキュメントの鮮度に不安がある場合（ソースとの乖離チェック）

**トリガーキーワード**: API仕様書, ドキュメント自動生成, MCP仕様書, セットアップガイド, 整合性検証, @Tool, Resource定義, Prompt定義

## Input Format

```yaml
# 必須パラメータ
source_root: "src/main/java"                    # Javaソースのルートディレクトリ
base_package: "com.tis.nablarch.mcp"            # ベースパッケージ

# 任意パラメータ
output_api_spec: "docs/api-specification.md"     # API仕様書の出力先
output_setup_guide: "docs/setup-guide.md"        # セットアップガイドの出力先
project_name: "Nablarch MCP Server"              # プロジェクト名
project_version: "0.1.0"                         # バージョン
transport: "STDIO"                               # トランスポート（STDIO / Streamable HTTP）
build_file: "build.gradle.kts"                   # ビルドファイル
config_file: "src/main/resources/application.yml" # 設定ファイル
verify_mode: false                               # 整合性検証のみモード（ドキュメント生成なし）
```

## Output Format

```
生成ファイル:
├── {output_api_spec}           # API仕様書（Markdown）
├── {output_setup_guide}        # セットアップガイド（Markdown）
└── 整合性検証レポート（標準出力） # verify_mode=true の場合
```

### API仕様書の構成

```markdown
# {project_name} API仕様書

## サーバー情報
| 項目 | 値 |
|------|-----|
| サーバー名 | ... |
| バージョン | ... |

## Tools仕様
### {tool_name}
- Tool定義（JSON Schema）
- 入力パラメータ（テーブル）
- 出力形式
- 使用例

## Resources仕様
### {resource_uri}
- URI パターン
- 説明
- レスポンス形式

## Prompts仕様
### {prompt_name}
- 引数定義（テーブル）
- 説明
- レスポンスの構成
```

## Instructions

### Phase 1: ソースコード解析

#### Step 1.1: プロジェクト構造の把握

```
【実行手順】

1. ソースディレクトリのスキャン:
   Glob: {source_root}/{base_package_path}/**/*.java

   期待される構造:
   com/tis/nablarch/mcp/
   ├── tools/          ← @Tool アノテーション付きクラス
   ├── resources/      ← ResourceProvider クラス
   ├── prompts/        ← Prompt クラス
   └── config/         ← McpServerConfig

2. 各ディレクトリ内のJavaファイルを一覧化:
   | ディレクトリ | ファイル | 種別 |
   |-------------|---------|------|
   | tools/ | SearchApiTool.java | Tool |
   | resources/ | HandlerResourceProvider.java | Resource |
   | prompts/ | SetupHandlerQueuePrompt.java | Prompt |
   | config/ | McpServerConfig.java | Config |

3. build.gradle.kts と application.yml も読み込む
```

#### Step 1.2: Tool定義の抽出

```
【実行手順】

tools/ ディレクトリ内の各Javaファイルから以下を抽出する。

1. @Tool アノテーションの検出:
   パターン: @Tool(description = "...")
   抽出: Tool名（メソッド名）、description

2. @ToolParam アノテーションの検出:
   パターン: @ToolParam(description = "...") String paramName
   抽出: パラメータ名、型、description、必須/任意

3. 戻り値の型:
   パターン: public String methodName(...)
   抽出: 戻り値の型

4. 抽出結果のデータ構造:
   {
     "tool_name": "search_api",
     "method_name": "searchApi",
     "description": "Search the Nablarch API documentation...",
     "parameters": [
       {
         "name": "keyword",
         "type": "String",
         "description": "Search keyword...",
         "required": true
       }
     ],
     "return_type": "String"
   }

【Grep パターン】

@Tool アノテーション:
  pattern: '@Tool\(description\s*=\s*"([^"]+)"'

@ToolParam アノテーション:
  pattern: '@ToolParam\(description\s*=\s*"([^"]+)"\)\s+(\w+)\s+(\w+)'

メソッドシグネチャ:
  pattern: 'public\s+(\w+)\s+(\w+)\s*\('
```

#### Step 1.3: Resource定義の抽出

```
【実行手順】

McpServerConfig.java から SyncResourceSpecification の登録を抽出する。

1. SyncResourceSpecification の検出:
   パターン: new McpSchema.Resource(uri, name, description, mimeType, null)
   抽出: URI, name, description, mimeType

2. ヘルパーメソッドの解析:
   createHandlerResourceSpec(type, name, description, provider)
   → URI = "nablarch://handler/" + type

   createGuideResourceSpec(topic, name, description, provider)
   → URI = "nablarch://guide/" + topic

3. ResourceProviderのメソッドシグネチャ:
   パターン: public String getHandlerMarkdown(String appType)
   → 引数: appType（リソースキー）

4. VALID_APP_TYPES / VALID_TOPICS の抽出:
   パターン: Set.of("web", "rest", "batch", ...)
   → 有効なキー値一覧

5. 抽出結果:
   {
     "uri_pattern": "nablarch://handler/{app_type}",
     "name": "Nablarch Web Handler Catalog",
     "description": "Web application handler specifications...",
     "mime_type": "text/markdown",
     "provider_class": "HandlerResourceProvider",
     "valid_keys": ["web", "rest", "batch", "messaging", ...]
   }
```

#### Step 1.4: Prompt定義の抽出

```
【実行手順】

McpServerConfig.java から SyncPromptSpecification の登録を抽出する。

1. promptSpec() 呼び出しの検出:
   パターン: promptSpec("name", "description", List.of(...), handler::execute)
   抽出: Prompt名, description, arguments, ハンドラクラス

2. arg() 呼び出しの検出:
   パターン: arg("name", "description", required)
   抽出: 引数名, description, required

3. Promptクラスの解析（各Promptクラス）:
   - execute() メソッドの引数バリデーションから有効値を抽出
   - VALID_* 定数から有効値リストを抽出
   - Markdown出力のセクション構造を解析

4. 抽出結果:
   {
     "prompt_name": "setup-handler-queue",
     "description": "Set up a Nablarch handler queue configuration",
     "arguments": [
       {
         "name": "app_type",
         "description": "Application type: web, rest, batch, messaging",
         "required": true
       }
     ],
     "handler_class": "SetupHandlerQueuePrompt"
   }
```

### Phase 2: API仕様書の生成

#### Step 2.1: サーバー情報セクション

```
【テンプレート】

# {project_name} API仕様書

## サーバー情報

| 項目 | 値 |
|------|-----|
| サーバー名 | `{project_name のkebab-case}` |
| バージョン | `{project_version}` |
| プロトコル | MCP (Model Context Protocol) over JSON-RPC 2.0 |
| トランスポート | {transport} |
| サーバータイプ | SYNC |

### Capabilities

```json
{
  "tools": { "listChanged": false },
  "resources": { "listChanged": false },
  "prompts": { "listChanged": false }
}
```

【データソース】
- project_name, project_version: 入力パラメータ
- transport: 入力パラメータ
- Capabilities: McpServerConfig の構成から推定
```

#### Step 2.2: Tools仕様セクション

```
【各Toolのテンプレート】

### {tool_name}

{description}

#### Tool定義

```json
{
  "name": "{tool_name}",
  "description": "{description}",
  "inputSchema": {
    "type": "object",
    "properties": {
      {parameters ごとのJSON Schema}
    },
    "required": [{required=true のパラメータ名}]
  }
}
```

#### 入力パラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
{parameters ごとの行}

#### 出力形式

{戻り値の説明。ソースコードのreturn文やフォーマット処理から推定}

---

【生成ルール】
- JSON Schema はソースの@ToolParam定義から構築
- "required" 配列は required=true のパラメータのみ含める
- 入力パラメータのテーブルは全パラメータを含める
- 出力形式はソースコードのreturn文を解析して記述
```

#### Step 2.3: Resources仕様セクション

```
【各Resourceのテンプレート】

### {uri_pattern}

| 項目 | 値 |
|------|-----|
| URI | `{uri_pattern}` |
| 名前 | {name} |
| 説明 | {description} |
| Content-Type | `{mime_type}` |

#### 有効なキー値

{valid_keys のテーブル}

#### レスポンス形式

{content_type に応じた形式説明}
```

#### Step 2.4: Prompts仕様セクション

```
【各Promptのテンプレート】

### {prompt_name}

{description}

#### 引数

| 引数 | 説明 | 必須 |
|------|------|------|
{arguments ごとの行}

#### 実行結果

{Promptの応答構造の説明。Promptクラスのexecute()の出力内容から推定}
```

### Phase 3: セットアップガイドの生成

#### Step 3.1: ビルド・実行手順の生成

```
【テンプレート】

# {project_name} セットアップガイド

## 前提条件

| 項目 | バージョン |
|------|----------|
{build.gradle.kts から抽出したJavaバージョン、依存ライブラリ}

## ビルド

```bash
./gradlew build
```

## 実行

```bash
./gradlew bootRun
```

## AIツール統合設定

### Claude Code / Claude Desktop

`.claude/mcp.json` に追加:

```json
{
  "mcpServers": {
    "nablarch": {
      "command": "java",
      "args": ["-jar", "build/libs/{jarファイル名}"]
    }
  }
}
```

## テスト

```bash
./gradlew test
```

## MCP Inspector でのテスト

{MCP Inspectorの使用手順}

【データソース】
- build.gradle.kts: 依存ライブラリ、Javaバージョン、JARファイル名
- application.yml: サーバー設定
- プロジェクトのREADME.md: 既存の手順情報
```

### Phase 4: 整合性検証

#### Step 4.1: ソース↔ドキュメント整合性チェック

```
【検証項目】

1. Tool整合性:
   □ ソースの@Tool定義の数 = ドキュメントのTool数
   □ 各ToolのパラメータがドキュメントのJSON Schemaと一致
   □ description がソースとドキュメントで一致

2. Resource整合性:
   □ McpServerConfigのResource登録数 = ドキュメントのResource数
   □ 各ResourceのURIパターンが一致
   □ 各Resourceの説明が一致

3. Prompt整合性:
   □ McpServerConfigのPrompt登録数 = ドキュメントのPrompt数
   □ 各Promptの引数定義が一致
   □ 各Promptの説明が一致

4. バージョン整合性:
   □ build.gradle.kts のバージョン = ドキュメントのバージョン

【検証結果フォーマット】

整合性検証レポート
=================

✅ Tools: 2/2 一致
  ✅ search_api: パラメータ2/2、description一致
  ✅ validate_handler_queue: パラメータ2/2、description一致

✅ Resources: 12/12 一致
  ✅ nablarch://handler/web: 一致
  ...

⚠️ Prompts: 5/6 一致
  ✅ setup-handler-queue: 引数1/1、description一致
  ❌ best-practices: 引数descriptionが不一致
     ソース: "Topic: handler-queue, action, validation, database, testing"
     ドキュメント: "Topic: handler-queue, action, validation, database"
     → ドキュメントの更新が必要

検証結果: WARN（1件の不一致）
```

#### Step 4.2: 差分レポートの生成

```
【差分がある場合の出力】

## 不一致箇所

| # | 種別 | 項目 | ソース | ドキュメント | 推奨アクション |
|---|------|------|--------|-------------|-------------|
| 1 | Prompt | best-practices.topic description | "...testing" | "...database" | ドキュメント更新 |
| 2 | Tool | new_tool | 存在する | 存在しない | ドキュメント追加 |

## 自動修正案

{不一致箇所のドキュメント修正テキスト}
```

### Phase 5: 品質検証

```
□ API仕様書チェック
  □ 全Toolが文書化されている
  □ 全Resourceが文書化されている
  □ 全Promptが文書化されている
  □ JSON Schemaの構文が正しい
  □ テーブルのフォーマットが正しい
  □ コードブロックの言語指定が正しい

□ セットアップガイドチェック
  □ ビルドコマンドが正しい
  □ 実行コマンドが正しい
  □ MCP設定JSONが正しい
  □ バージョン番号が最新

□ 整合性検証チェック
  □ 全Tool/Resource/Promptの数が一致
  □ 全パラメータ定義が一致
  □ 全descriptionが一致
  □ バージョン番号が一致
```

## Examples

### Example 1: Nablarch MCP Server のAPI仕様書生成

```yaml
source_root: "src/main/java"
base_package: "com.tis.nablarch.mcp"
output_api_spec: "docs/api-specification.md"
project_name: "Nablarch MCP Server"
project_version: "0.1.0"
transport: "STDIO"
```

```
# 実行結果

解析結果:
- Tools: 2件（search_api, validate_handler_queue）
- Resources: 12件（handler×6, guide×6）
- Prompts: 6件

生成ファイル:
- docs/api-specification.md（約400行）

整合性検証: PASS（全項目一致）
```

### Example 2: 整合性検証のみ

```yaml
source_root: "src/main/java"
base_package: "com.tis.nablarch.mcp"
output_api_spec: "docs/api-specification.md"
verify_mode: true
```

```
# 実行結果（整合性検証のみ、ドキュメント生成なし）

整合性検証レポート
=================
✅ Tools: 2/2 一致
✅ Resources: 12/12 一致
✅ Prompts: 6/6 一致
✅ Version: 一致

検証結果: PASS
```

### Example 3: Tool追加後のドキュメント更新

```
# シナリオ
1. 開発者が SemanticSearchTool.java を追加
2. verify_mode=true で整合性チェック → FAIL（Toolが1件不足）
3. verify_mode=false で再実行 → ドキュメント自動更新

# 差分レポート
❌ Tools: 2/3 — 新規Tool "semantic_search" がドキュメントに未記載
推奨: ドキュメント再生成を実行

# 再生成後
✅ Tools: 3/3 一致（semantic_search を追加）
```

## Anti-Patterns

1. **正規表現のみに依存したソース解析**
   - @Toolアノテーションが複数行にまたがる場合に検出漏れ
   - 対策: 行結合してからパターンマッチ、またはAST解析ライブラリ使用

2. **ハードコードされたパッケージパス**
   - `com.tis.nablarch.mcp` 以外のパッケージに対応できない
   - 対策: base_package パラメータで動的に指定

3. **既存ドキュメントの完全上書き**
   - 手動で追加した補足情報が消える
   - 対策: 自動生成セクションとカスタムセクションを分離（マーカーコメント使用）

4. **整合性検証の false positive**
   - 軽微な文言差異（空白、改行）で不一致判定
   - 対策: 比較前に正規化（trim、連続空白の圧縮）

5. **非公開クラスの文書化**
   - package-private クラスや内部APIを文書化してしまう
   - 対策: @Tool / SyncResourceSpecification に登録されたもののみ文書化

6. **JSON Schemaの型推定ミス**
   - Java String → JSON "string" は明確だが、List等の変換は注意
   - 対策: @ToolParam の型情報を正確にJSON Schemaに変換するマッピング表を使用

7. **ドキュメント生成時のソース変更**
   - 生成中にソースが変更されると整合性が崩れる
   - 対策: 生成前にソースのスナップショットを取得（git stashまたはコミット確認）

8. **セットアップガイドの環境依存**
   - OS固有のパス表記やコマンドがハードコード
   - 対策: OS非依存の表記を使用し、OS固有の注記はセクション分離

## Appendix

### A. ソースコード解析パターン集

```
# @Tool アノテーション抽出
pattern: '@Tool\(\s*description\s*=\s*"((?:[^"\\]|\\.)*)"\s*\)'

# @ToolParam 抽出
pattern: '@ToolParam\(\s*description\s*=\s*"((?:[^"\\]|\\.)*)"\s*\)\s+(\w+)\s+(\w+)'

# SyncResourceSpecification 内の Resource 構築
pattern: 'new McpSchema\.Resource\(\s*([^,]+),\s*"([^"]+)",\s*"([^"]+)",\s*"([^"]+)"'

# promptSpec 呼び出し
pattern: 'promptSpec\(\s*"([^"]+)",\s*\n?\s*"([^"]+)"'

# arg 呼び出し
pattern: 'arg\(\s*"([^"]+)",\s*"([^"]+)",\s*(true|false)\s*\)'

# VALID_* 定数
pattern: 'VALID_\w+\s*=\s*(?:Set|List)\.of\(\s*([^)]+)\s*\)'
```

### B. JSON Schema型マッピング

| Java型 | JSON Schema型 | 備考 |
|--------|--------------|------|
| String | "string" | 基本型 |
| int / Integer | "integer" | |
| long / Long | "integer" | |
| boolean / Boolean | "boolean" | |
| double / Double | "number" | |
| List<String> | "array" + items: "string" | |
| Map<String, Object> | "object" | additionalProperties |

### C. 整合性検証スクリプト例

```bash
#!/bin/bash
# MCPサーバーのソース↔ドキュメント整合性チェック

echo "=== Tool 整合性チェック ==="
# ソースの@Tool数
SRC_TOOLS=$(grep -r '@Tool(' src/main/java/ | grep -v Test | wc -l)
# ドキュメントのTool数
DOC_TOOLS=$(grep -c '### search_api\|### validate_handler_queue' docs/api-specification.md)
echo "ソース: ${SRC_TOOLS}件, ドキュメント: ${DOC_TOOLS}件"

echo "=== Resource 整合性チェック ==="
# McpServerConfigのResource登録数
SRC_RES=$(grep -c 'createHandlerResourceSpec\|createGuideResourceSpec' \
  src/main/java/com/tis/nablarch/mcp/config/McpServerConfig.java)
# ドキュメントのResource数
DOC_RES=$(grep -c 'nablarch://' docs/api-specification.md | head -1)
echo "ソース: ${SRC_RES}件, ドキュメント: ${DOC_RES}件"

echo "=== Prompt 整合性チェック ==="
# promptSpec呼び出し数
SRC_PROMPTS=$(grep -c 'promptSpec(' \
  src/main/java/com/tis/nablarch/mcp/config/McpServerConfig.java)
echo "ソース: ${SRC_PROMPTS}件"
```
