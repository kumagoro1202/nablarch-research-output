---
name: mcp-integration-test-doc-generator
description: MCP ServerのJSON-RPC出力（JSONL形式）を解析し、統合テスト結果ドキュメントを自動生成する。「MCPテスト結果をドキュメント化して」「JSONL出力からテストレポートを作って」「MCP Inspectorの結果をまとめて」といった要望に対応する。JSONL形式のJSON-RPCレスポンスを入力として受け取り、テスト一覧表・各テスト詳細・カバレッジ分析・結果サマリ・Claude統合テストシナリオテンプレートを含む構造化されたMarkdownドキュメントを出力する。
---

# MCP Integration Test Doc Generator

## Overview

MCP Server の統合テスト実行結果（JSONL形式のJSON-RPCレスポンス群）を入力として受け取り、構造化されたMarkdownテスト結果ドキュメントを自動生成するスキル。

MCP（Model Context Protocol）サーバーの開発サイクルでは、MCP Inspectorやcurlベースのテスト実行により、JSON-RPC 2.0形式のレスポンスがJSONLファイルとして出力される。本スキルはそのJSONLを解析し、テスト一覧表の自動生成、各テストの詳細セクション作成、MCPプリミティブのカバレッジ分析、結果サマリの集計、そしてClaude統合テストシナリオのテンプレート生成までを一括で行う。

**本スキルの特長:**
- JSONL（行区切りJSON）の自動パースとJSON-RPCレスポンス構造の解析
- MCPプロトコルの4プリミティブ（Protocol/Tools/Resources/Prompts）に基づく自動分類
- テスト一覧表の自動生成（テスト番号、カテゴリ、操作、入力、期待結果、判定）
- 各テストの詳細セクション自動生成（リクエスト/レスポンス要約、判定理由）
- MCPプリミティブのカバレッジ分析（テスト済み/未テスト項目の自動検出）
- テスト結果サマリの自動集計（成功/失敗/エラー件数）
- Claude統合テストシナリオ定義テンプレートの生成

**用途:**
- MCP Server開発サイクルでの統合テスト後のドキュメント自動生成
- テスト結果のチーム共有・レビュー用ドキュメント作成
- MCPプロトコル準拠性の確認・記録
- リリース前のテストカバレッジ可視化
- CI/CDパイプラインでのテストレポート自動生成

**実績:**
- cmd_025 Phase 1 において、足軽7号がnablarch-mcp-serverの統合テスト結果ドキュメント（mcp-inspector-test.md、360行）を手動作成
- 入力: /tmp/mcp-all-output.jsonl（15行のJSON-RPCレスポンス）
- 出力: テスト一覧表（15テスト）、各テスト詳細（15セクション）、カバレッジ分析、結果サマリを含むMarkdown

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「MCPテスト結果をドキュメント化して」
- 「JSONL出力からテストレポートを作って」
- 「MCP Inspectorのテスト結果をまとめて」
- 「MCP Serverの統合テスト結果を記録して」
- 「テストカバレッジを分析して」
- 「Claude統合テストのシナリオを設計して」
- MCP Inspector / curl / FIFO パイプ等でMCPサーバーをテストした後、結果をドキュメント化する必要がある場合
- MCPサーバーのリリース前にテストカバレッジを確認・記録する必要がある場合
- テスト結果をチームメンバーやレビュアーに共有するためのレポートが必要な場合

**トリガーキーワード**: MCPテスト, MCP Inspector, JSONL, JSON-RPC, テスト結果, テストレポート, 統合テスト, カバレッジ, test results, test report

## Instructions

### Phase 1: 入力解析（JSONLファイルの読み込みとJSON-RPCレスポンスの構造化）

JSONLファイルを読み込み、各行のJSON-RPCレスポンスを解析・分類する。

#### Step 1.1: JSONLファイルの読み込み

```
【入力ファイルの特定と読み込み】

1. 指定されたJSONLファイルを読み込む
   Read: {jsonl_file_path}

2. ファイルが指定されていない場合、候補を検索
   Glob: /tmp/mcp-*.jsonl
   Glob: /tmp/*output*.jsonl
   Glob: {project_dir}/**/test-output*.jsonl

3. ファイル形式の検証
   - 各行が有効なJSONであることを確認
   - 各行が JSON-RPC 2.0 レスポンス形式であることを確認
     必須フィールド: "jsonrpc": "2.0", "id": <number>
     結果フィールド: "result" または "error"
   - 行数（= テスト数）をカウント

【バリデーションルール】
- 空行はスキップ
- JSON パースエラーの行はエラーとして記録（テスト失敗扱い）
- "jsonrpc" フィールドが "2.0" でないものは警告
```

#### Step 1.2: JSON-RPCレスポンスの分類

```
【MCPプリミティブへの自動分類】

各レスポンスを以下の基準でMCPプリミティブに分類する:

1. Protocol（プロトコル初期化）
   - id: 1（通常）
   - result に "protocolVersion", "capabilities", "serverInfo" を含む
   → カテゴリ: Protocol, テスト内容: initialize

2. Tools
   - result に "tools" 配列を含む → tools/list
   - result に "content" 配列を含み、直前が tools/call の文脈
     → tools/call（ツール名は content のテキストから推定）

3. Resources
   - result に "resources" 配列を含む → resources/list
   - result に "contents" 配列を含み、uri が "nablarch://" 等で始まる
     → resources/read（リソースURIから対象を特定）

4. Prompts
   - result に "prompts" 配列を含む → prompts/list
   - result に "description" と "messages" を含む
     → prompts/get（description からプロンプト名を推定）

【分類結果の構造化】
各レスポンスに以下のメタデータを付与:
  - test_number: レスポンスの行番号（1始まり）
  - category: Protocol | Tools | Resources | Prompts
  - method: initialize | tools/list | tools/call | resources/list | resources/read | prompts/list | prompts/get
  - input_summary: テスト入力の要約（引数等）
  - result_summary: 結果の要約
  - judgment: OK | NG | ERROR
  - is_error: result.isError の値（存在する場合）
```

#### Step 1.3: テスト環境情報の抽出

```
【テスト環境メタデータの収集】

1. initialize レスポンスから抽出:
   - サーバー名: result.serverInfo.name
   - サーバーバージョン: result.serverInfo.version
   - プロトコルバージョン: result.protocolVersion
   - capabilities: result.capabilities のキー一覧

2. テスト実行環境（ユーザー入力または推定）:
   - テスト方式: MCP Inspector / curl / FIFO パイプ / その他
   - トランスポート: STDIO / SSE / Streamable HTTP
   - テスト日時: ファイルのタイムスタンプまたはユーザー指定
   - テスト結果ファイルパス: 入力JSONLのパス

3. 追加情報（ユーザーから取得、または既存ドキュメントから参照）:
   - Java/Node.js バージョン
   - ランタイム環境の詳細
```

### Phase 2: テスト一覧表の生成

Phase 1 で構造化したデータからテスト一覧表を生成する。

#### Step 2.1: テスト一覧テーブルの構築

```
【テスト一覧テーブル】

以下の列で Markdown テーブルを生成:

| # | カテゴリ | テスト内容 | 入力 | 期待結果 | 判定 |
|---|---------|----------|------|---------|------|

各行の値:
- #: test_number（連番）
- カテゴリ: category（Protocol / Tools / Resources / Prompts）
- テスト内容: method + 補足情報
  例: "search_api (正常)", "search_api (該当なし)", "resources/read handler/web"
- 入力: input_summary
  例: keyword="UniversalDao", URI, app_type=web
  入力なしの場合は "—"
- 期待結果: result_summary
  例: "サーバー情報・capabilities", "2ツール一覧", "検証NG (7件エラー)"
- 判定: OK / NG / ERROR

【入力要約の生成ルール】
- tools/call: arguments の主要パラメータをキー=値形式で表示
- resources/read: URI のパス部分を表示
- prompts/get: arguments の主要パラメータをキー=値形式で表示
- その他: "—"

【結果要約の生成ルール】
- initialize: "サーバー情報・capabilities"
- tools/list: "{N}ツール一覧"（N = tools配列の長さ）
- tools/call: contentのテキストから主要な結果を30文字以内に要約
- resources/list: "{N}リソース一覧"
- resources/read: contentの最初のタイトルまたはuri
- prompts/list: "{N}プロンプト一覧"
- prompts/get: descriptionの値
- error レスポンス: エラーメッセージの要約

【判定ルール】
- OK: result が存在し、isError が false または未設定
- NG: result.isError が true、またはレスポンスに error フィールドがある
- ERROR: JSONパースエラー、または予期しないレスポンス形式
```

### Phase 3: テスト詳細セクションの生成

各テストの詳細情報をMarkdownセクションとして生成する。

#### Step 3.1: 個別テスト詳細の生成

```
【各テストの詳細セクション】

テストごとに以下の構造で Markdown セクションを生成:

### Test #{test_number}: {テスト名}

**リクエスト:**
```json
{推定されるJSON-RPCリクエスト}
```

**レスポンス要約:**
- {結果の箇条書き（3〜8項目）}

**判定:** {判定理由の1〜2文の説明}

---

【リクエストの推定ルール】
JSONLには通常レスポンスのみが記録される。リクエストは以下のルールで推定:

1. initialize (id: 1):
   {"jsonrpc":"2.0","id":1,"method":"initialize","params":{...}}

2. tools/list (id: 2, 通常):
   {"jsonrpc":"2.0","id":{id},"method":"tools/list","params":{}}

3. tools/call:
   {"jsonrpc":"2.0","id":{id},"method":"tools/call","params":{"name":"{tool_name}","arguments":{...}}}
   ※ tool_name と arguments は result.content のテキストから逆推定

4. resources/list:
   {"jsonrpc":"2.0","id":{id},"method":"resources/list","params":{}}

5. resources/read:
   {"jsonrpc":"2.0","id":{id},"method":"resources/read","params":{"uri":"{uri}"}}
   ※ uri は result.contents[0].uri から取得

6. prompts/list:
   {"jsonrpc":"2.0","id":{id},"method":"prompts/list","params":{}}

7. prompts/get:
   {"jsonrpc":"2.0","id":{id},"method":"prompts/get","params":{"name":"{prompt_name}","arguments":{...}}}
   ※ prompt_name は result.description から推定

【レスポンス要約の生成ルール】
- JSON全体を表示するのではなく、主要なフィールドを箇条書きで要約
- 大きなテキストフィールド（Markdown本文等）はタイトルと構成要素を列挙
- 配列は要素数と代表的な要素名を表示
- isError フィールドがある場合は明記

【判定理由の記述ルール】
- 何を検証し、何が確認できたかを簡潔に記述
- 数値（件数、要素数等）がある場合は含める
- エラーの場合はエラー内容と影響を記述
```

### Phase 4: カバレッジ分析

MCPプリミティブ全体に対するテストカバレッジを分析する。

#### Step 4.1: MCPプリミティブカバレッジの算出

```
【カバレッジ分析テーブル】

以下の列でテーブルを生成:

| Primitive | メソッド | テスト数 | カバー率 |
|-----------|---------|---------|---------|

【カバレッジの算出方法】

1. Protocol:
   - initialize: テスト有無（0 or 1）
   - カバー率 = テスト数 / 1 × 100%

2. Tools:
   - tools/list: テスト有無（0 or 1）
   - tools/call: テスト済みツール数 / 全ツール数
     ※ 全ツール数は tools/list レスポンスの tools 配列長から取得
   - 各ツールについて: 正常系 + 異常系のテスト数
   - カバー率 = テスト済みツール数 / 全ツール数 × 100%

3. Resources:
   - resources/list: テスト有無（0 or 1）
   - resources/read: テスト済みリソース数 / 全リソース数
     ※ 全リソース数は resources/list レスポンスの resources 配列長から取得
   - カバー率 = テスト済みリソース数 / 全リソース数 × 100%

4. Prompts:
   - prompts/list: テスト有無（0 or 1）
   - prompts/get: テスト済みプロンプト数 / 全プロンプト数
     ※ 全プロンプト数は prompts/list レスポンスの prompts 配列長から取得
   - カバー率 = テスト済みプロンプト数 / 全プロンプト数 × 100%

【未テスト項目の列挙】

カバレッジ分析テーブルの後に以下を記載:

### 未テスト項目

- {primitive}: {未テストのメソッド/リソース/ツール/プロンプトの一覧}

例:
- resources/read: rest, batch, messaging, http-messaging, jakarta-batch (handler)
- resources/read: testing, validation, database, handler-queue, error-handling (guide)
- search_api: カテゴリフィルタ付き検索
- validate_handler_queue: 正常なハンドラキュー（検証OK）ケース
- エラー系: 不正なリソースURI、不正なプロンプト名
```

### Phase 5: テスト結果サマリの生成

全テストの合格/不合格/エラーを集計する。

#### Step 5.1: 結果サマリテーブルの生成

```
【テスト結果サマリ】

以下のテーブルを生成:

| 結果 | 件数 |
|------|------|
| 成功 | **{ok_count}** |
| 失敗 | **{ng_count}** |
| エラー | **{error_count}** |
| 合計 | **{total}** |

**判定: {全体判定}**

【全体判定ルール】
- 全テスト成功: "全テスト成功"
- 失敗あり: "{ng_count}件の失敗あり — 要修正"
- エラーあり: "{error_count}件のエラーあり — 調査必要"
- 失敗+エラーあり: "{ng_count}件の失敗、{error_count}件のエラー — 要対応"
```

### Phase 6: Claude統合テストシナリオテンプレートの生成

MCP Inspectorテストで各プリミティブの個別動作が確認できた後、
Claude Desktop / Claude Code との統合テストシナリオを定義するテンプレートを生成する。

#### Step 6.1: シナリオテンプレートの生成

```
【Claude統合テストシナリオ定義】

MCPサーバーの capabilities と tools/resources/prompts の情報を基に、
実際のユースケースを想定したシナリオを設計する。

シナリオの構造:
## シナリオ{N}: {シナリオ名}

### ユーザー入力
> 「{自然言語でのユーザー入力例}」

### 期待されるMCP呼び出しフロー
```
Step 1: {Primitive} → {method} ({params})
  ↓ {取得する情報の説明}
Step 2: {Primitive} → {method} ({params})
  ↓ {取得する情報の説明}
...
```

### AIの期待出力
1. {出力項目1}
2. {出力項目2}
...

【シナリオ設計の方針】
1. 各Primitiveが少なくとも1回は使用されるようにする
2. Primitive間の連携パターンを網羅する:
   - パターンA: Prompt → Resource（知識補完）
   - パターンB: Prompt → Tool（自動検証）
   - パターンC: Tool → Resource（結果の文脈化）
   - パターンD: Prompt → Tool → Resource（フル連携）
3. 実際のユーザーが発する自然言語の入力を想定する
4. 各シナリオに期待されるMCP呼び出しフローを明示する

【テスト結果テーブル】
| シナリオ | Prompt | Tool | Resource | 連携パターン | 状態 |
|---------|--------|------|----------|-------------|------|
```

### Phase 7: ドキュメント組み立て（最終Markdownの生成）

全フェーズの出力を統合し、最終的なMarkdownドキュメントを組み立てる。

#### Step 7.1: ドキュメント構造の組み立て

```
【最終ドキュメント構造】

# {テストドキュメントタイトル}

## テスト環境            ← Phase 1 Step 1.3
## テスト方式            ← ユーザー入力 or テンプレート
## テスト結果サマリ      ← Phase 5
## テスト一覧            ← Phase 2
## テスト詳細            ← Phase 3
  ### Test #1: ...
  ### Test #2: ...
  ...
## カバレッジ分析        ← Phase 4
  ### MCP Primitive カバレッジ
  ### 未テスト項目
## Claude統合テストシナリオ  ← Phase 6（オプション）

【セクション配置の意図】
- テスト環境とサマリを冒頭に配置: 読者が最初にテスト概要を把握できる
- テスト一覧: サマリの次に全体像を表形式で提示
- テスト詳細: 各テストの深掘り情報（必要に応じて参照）
- カバレッジ分析: テストの網羅性を最後に分析
- Claude統合テストシナリオ: 次のテストサイクルへの橋渡し
```

#### Step 7.2: 出力先の決定と書き込み

```
【出力先の決定】

1. ユーザーが出力先を指定した場合: その指定パスに書き込み
2. 指定なしの場合のデフォルト:
   - {project_dir}/docs/test-results/{test_type}-test.md
   - {test_type} は "mcp-inspector" / "claude-integration" / "curl" 等

【ファイル書き込み】
Write: {output_path}

【既存ファイルの扱い】
- 同名ファイルが存在する場合、上書き（テスト結果は最新が正）
- 過去の結果を残す場合は日時付きバックアップを提案
```

## Input Format

### 必須入力

```yaml
jsonl_file: /path/to/mcp-output.jsonl  # JSONL形式のMCPテスト結果ファイルパス
```

### オプション入力

```yaml
# テスト環境情報（指定しない場合はinitializeレスポンスから自動抽出）
server_name: "nablarch-mcp-server"     # MCPサーバー名
server_version: "0.1.0"                # MCPサーバーバージョン
transport: "STDIO"                     # トランスポート: STDIO / SSE / Streamable HTTP
test_method: "FIFO パイプ"              # テスト方式の説明
test_date: "2026-02-02"                # テスト実施日
java_version: "JDK 17"                 # Java/Node.jsバージョン

# 出力設定
output_path: docs/test-results/mcp-inspector-test.md  # 出力先パス
include_claude_scenarios: true         # Claude統合テストシナリオを含めるか（デフォルト: true）

# リクエスト情報（JSONLにレスポンスのみの場合にリクエストを補完）
request_log: /path/to/request-log.jsonl  # リクエストログ（存在する場合）
```

### JSONL入力フォーマット仕様

```
各行は JSON-RPC 2.0 レスポンスオブジェクト:

成功レスポンス:
{"jsonrpc":"2.0","id":<number>,"result":{...}}

エラーレスポンス:
{"jsonrpc":"2.0","id":<number>,"error":{"code":<number>,"message":"<string>"}}

行の区切り: LF (\n) または CRLF (\r\n)
エンコーディング: UTF-8
```

## Output Format

### Markdownドキュメント構造

```markdown
# {サーバー名} 統合テスト結果

## テスト環境

| 項目 | 値 |
|------|-----|
| サーバー | {server_name} {server_version} |
| Java | {java_version} |
| トランスポート | {transport} |
| テスト方式 | {test_method} |
| テスト日時 | {test_date} |
| テスト結果ファイル | {jsonl_file_path} |

## テスト方式

{テスト方式の説明（FIFO パイプ、MCP Inspector等のコマンド例）}

## テスト結果サマリ

| 結果 | 件数 |
|------|------|
| 成功 | **{ok_count}** |
| 失敗 | **{ng_count}** |
| 合計 | **{total}** |

**判定: {全体判定}**

## テスト一覧

| # | カテゴリ | テスト内容 | 入力 | 期待結果 | 判定 |
|---|---------|----------|------|---------|------|
| 1 | Protocol | initialize | — | サーバー情報・capabilities | OK |
| ... | ... | ... | ... | ... | ... |

## テスト詳細

### Test #1: initialize

**リクエスト:**
```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{...}}
```

**レスポンス要約:**
- protocolVersion: `2024-11-05`
- serverInfo.name: `{server_name}`
- capabilities: {capabilities一覧}

**判定:** {判定理由}

---

{... 全テストの詳細 ...}

## カバレッジ分析

### MCP Primitive カバレッジ

| Primitive | メソッド | テスト数 | カバー率 |
|-----------|---------|---------|---------|
| Protocol | initialize | {n} | {rate}% |
| ... | ... | ... | ... |

### 未テスト項目

- {未テスト項目の一覧}

## Claude統合テストシナリオ（オプション）

{シナリオ定義テンプレート}
```

## Examples

### Example 1: 基本的なMCP Inspectorテスト結果のドキュメント化

**入力:**

```
jsonl_file: /tmp/mcp-all-output.jsonl
```

JONLファイルの内容（15行のJSON-RPCレスポンス）:

```jsonl
{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2024-11-05","capabilities":{...},"serverInfo":{"name":"nablarch-mcp-server","version":"0.1.0"}}}
{"jsonrpc":"2.0","id":2,"result":{"tools":[{"name":"validateHandlerQueue",...},{"name":"searchApi",...}]}}
{"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":"\"検索結果: ...5件...\""}],"isError":false}}
...
```

**実行フロー:**

1. Phase 1: JSONLを読み込み、15行を解析。initializeからサーバー名・バージョン・capabilitiesを抽出。各レスポンスを Protocol(1) / Tools(3) / Resources(3) / Prompts(8) に分類
2. Phase 2: 15行のテスト一覧テーブルを生成
3. Phase 3: 15セクションのテスト詳細を生成（リクエストの推定含む）
4. Phase 4: カバレッジ分析 — Tools 100%, Resources 16.7%(2/12), Prompts 100%
5. Phase 5: サマリ — 成功15件、失敗0件 → "全テスト成功"
6. Phase 6: 3シナリオのClaude統合テストテンプレートを生成
7. Phase 7: 全体を組み立てて約360行のMarkdownドキュメントを出力

**出力先:** `docs/test-results/mcp-inspector-test.md`

### Example 2: エラーを含むテスト結果

**入力:**

```
jsonl_file: /tmp/mcp-error-test.jsonl
```

JONLファイルの内容（エラーレスポンスを含む）:

```jsonl
{"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2024-11-05",...}}
{"jsonrpc":"2.0","id":2,"error":{"code":-32601,"message":"Method not found: tools/invalid"}}
{"jsonrpc":"2.0","id":3,"result":{"content":[{"type":"text","text":"..."}],"isError":true}}
```

**実行フロー:**

1. Phase 1: 3行を解析。id:2 はJSON-RPCエラー、id:3 はアプリケーションエラー（isError: true）
2. Phase 2: テスト一覧で id:2 を NG、id:3 を NG と判定
3. Phase 3: エラー内容を詳細セクションに記載
4. Phase 5: サマリ — 成功1件、失敗2件 → "2件の失敗あり — 要修正"

### Example 3: Claude統合テストシナリオの重点生成

**入力:**

```
jsonl_file: /tmp/mcp-all-output.jsonl
include_claude_scenarios: true
```

**実行フロー:**

Phase 6 で以下のシナリオを生成:

- シナリオ1: ハンドラキュー設定（Prompt → Resource → Resource）
- シナリオ2: アクションクラス実装ガイド（Prompt → Tool → Resource）
- シナリオ3: 設定ファイルレビュー（Prompt → Tool → Resource）

各シナリオにユーザー入力例、MCP呼び出しフロー、期待出力を含む。

## Guidelines

### 必須ルール（Must）

1. **JSONLの全行を処理せよ**: 空行を除き、全てのJSON行をテスト結果として処理する。行をスキップしてはならない
2. **JSON-RPC 2.0 仕様に準拠せよ**: レスポンスの判定は JSON-RPC 2.0 の仕様（result / error フィールド）に基づく
3. **MCPプロトコルの4プリミティブで分類せよ**: Protocol, Tools, Resources, Prompts の4カテゴリに必ず分類する
4. **テスト一覧表は全テストを含めよ**: 1つも漏れなく全テストをテスト一覧テーブルに記載する
5. **カバレッジ分析は list レスポンスを基準にせよ**: 全ツール数・全リソース数・全プロンプト数は各 list レスポンスから取得する
6. **未テスト項目を明示せよ**: カバレッジ100%でないプリミティブについて、未テストの具体的な項目名を列挙する
7. **判定理由を記述せよ**: 各テストの判定（OK/NG/ERROR）には、その根拠を1〜2文で記述する
8. **結果サマリは正確に集計せよ**: OK/NG/ERROR の件数と合計が一致することを検証する

### 推奨ルール（Tips）

1. **リクエストの推定は保守的に**: JSONLにレスポンスのみの場合、リクエストは推定だと明記する。確実でないパラメータには `{...}` を使用する
2. **大きなレスポンスは要約せよ**: Markdown本文（resources/read等）は全文を載せず、タイトル・構成要素・行数で要約する
3. **テスト環境セクションを充実させよ**: テスト再現のために、コマンド例やセットアップ手順を含める
4. **カテゴリ順に整理せよ**: テスト詳細セクションは Protocol → Tools → Resources → Prompts の順が読みやすい（ただしテスト番号順でも可）
5. **Claude統合テストシナリオは実用的に**: 架空のシナリオではなく、MCPサーバーの実際のユースケースに基づくシナリオを設計する
6. **過去のテスト結果と比較可能にせよ**: テーブル形式や項目名を統一し、過去の結果との差分が分かるようにする

### アンチパターン（Don't）

1. **JSONの全文貼り付け禁止**: レスポンスのJSON全体をそのまま貼り付けない。要約して記述する（特にMarkdown本文を含むresources/readレスポンス）
2. **曖昧な判定禁止**: "おそらくOK" "問題なさそう" 等の曖昧な判定をしない。OK/NG/ERROR を明確に判定する
3. **カバレッジの過大評価禁止**: 同じツールを異なるパラメータでテストしても、カバレッジは1ツール分としてカウントする（ただしテスト数は加算）
4. **テスト環境情報の省略禁止**: テスト環境セクションを省略しない。再現性のために必須
5. **未テスト項目の隠蔽禁止**: カバレッジが低い場合でも、未テスト項目を正直に列挙する
6. **リクエストログとレスポンスの混同禁止**: JSONLにリクエストとレスポンスが混在する場合、レスポンス（id + result/error）のみをテスト結果として処理する
7. **手動編集の痕跡を残さない**: 自動生成されたドキュメントに手動編集を加える場合は、編集箇所を明記するか、別セクションに記載する
8. **エラーレスポンスの無視禁止**: JSON-RPCエラーレスポンス（error フィールド）は必ず NG として記録する。"error" を "期待通りのエラー" として OK にする場合は、その旨を判定理由に明記する
