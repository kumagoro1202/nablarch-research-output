# Nablarch MCP Server セットアップガイド

## 概要

Nablarch MCP ServerをClaude CodeのMCPサーバーとして利用するためのセットアップ手順。
全担当者のClaude Codeセッションから共通利用可能な構成。

## 前提条件

| 項目 | 要件 |
|------|------|
| Java | 17以上（確認済み: OpenJDK 17.0.18） |
| Maven | 3.9.x（Maven Wrapper同梱、`./mvnw`利用可） |
| PostgreSQL | 16+（`localhost:5432`で稼働、DB名: `nablarch_mcp`） |
| ONNXモデル | **不要**（`provider=api`で回避） |

### PostgreSQLについて

- MCP Serverはデフォルトで`jdbc:postgresql://localhost:5432/nablarch_mcp`に接続する
- DBが必要なのはFlywayマイグレーションとJPAバリデーションのため
- 静的知識（YAML知識ベース）はDB不要だが、起動にはDB接続が必要
- RAG機能（セマンティック検索）にはpgvector拡張が必要

### ONNXモデルについて

- デフォルト設定（`provider=local`）ではONNXモデル（BGE-M3, CodeSage）が必要
- モデルが`/opt/models/`に未配置の場合、起動時にエラーになる
- `--nablarch.mcp.embedding.provider=api`オプションで回避可能
- API Embeddingはキーなしでも起動可（呼び出し時にのみ失敗）

## ビルド手順

```bash
cd ~/nablarch-mcp-server
git checkout main && git pull origin main
./mvnw package -DskipTests
```

ビルド成果物: `target/nablarch-mcp-server-0.1.0-SNAPSHOT.jar`（約186MB）

※ テストはDB統合テストを含むため`-DskipTests`でビルドのみ実行

## Claude Code MCP設定

### 登録方法（推奨）

```bash
claude mcp add -s user -t stdio nablarch -- java -jar /home/kuma/nablarch-mcp-server/target/nablarch-mcp-server-0.1.0-SNAPSHOT.jar --nablarch.mcp.embedding.provider=api
```

これにより`~/.claude.json`のユーザースコープに登録され、全セッションから利用可能になる。

### 登録される設定内容

```json
{
  "nablarch": {
    "type": "stdio",
    "command": "java",
    "args": [
      "-jar",
      "/home/kuma/nablarch-mcp-server/target/nablarch-mcp-server-0.1.0-SNAPSHOT.jar",
      "--nablarch.mcp.embedding.provider=api"
    ],
    "env": {}
  }
}
```

### 設定の削除

```bash
claude mcp remove -s user nablarch
```

## 起動・停止

MCPサーバーはClaude Codeが自動的に起動・管理する（STDIO方式）。
手動での起動・停止は不要。

### 手動テスト（デバッグ用）

```bash
cd ~/nablarch-mcp-server
java -jar target/nablarch-mcp-server-0.1.0-SNAPSHOT.jar --nablarch.mcp.embedding.provider=api
# Ctrl+C で停止
```

起動ログを見る場合（デフォルトではSTDIO用にログ出力が抑制されている）:
```bash
java -Dlogging.pattern.console="%d{HH:mm:ss} %-5level %logger{36} - %msg%n" \
  -jar target/nablarch-mcp-server-0.1.0-SNAPSHOT.jar \
  --nablarch.mcp.embedding.provider=api
```

起動時間: 約9〜10秒（知識ベースYAML読み込み含む）

## 利用可能なTools（10個）

| Tool名 | 説明 |
|---------|------|
| `semanticSearch` | Nablarch知識ベースのセマンティック検索（ハイブリッド検索） |
| `searchApi` | NablarchのAPI仕様（クラス、メソッド、パターン）検索 |
| `design` | アプリケーションタイプに基づくハンドラキュー設計 |
| `validateHandlerQueue` | ハンドラキューXML設定の検証 |
| `optimize` | 既存ハンドラキューの分析・最適化提案 |
| `generateCode` | Nablarch準拠コード生成（Action, Form, SQL, Entity等） |
| `generateTest` | Nablarchアプリケーション向けテストコード生成 |
| `recommend` | 要件に基づくNablarch設計パターン推奨 |
| `troubleshoot` | Nablarch固有エラーのトラブルシューティング |
| `analyzeMigration` | Nablarchバージョン移行の影響分析 |

## 利用可能なResources（18個）

| URI | 説明 |
|-----|------|
| `nablarch://handler/web` | Webハンドラカタログ |
| `nablarch://handler/rest` | RESTハンドラカタログ |
| `nablarch://handler/batch` | バッチハンドラカタログ |
| `nablarch://handler/messaging` | メッセージングハンドラカタログ |
| `nablarch://handler/http-messaging` | HTTPメッセージングハンドラカタログ |
| `nablarch://handler/jakarta-batch` | Jakarta Batchハンドラカタログ |
| `nablarch://api/modules` | APIモジュールカタログ |
| `nablarch://pattern/list` | 設計パターンカタログ |
| `nablarch://config/list` | 設定テンプレートカタログ |
| `nablarch://example/list` | サンプルカタログ |
| `nablarch://antipattern/list` | アンチパターンカタログ |
| `nablarch://version/info` | バージョン情報 |
| `nablarch://guide/setup` | セットアップガイド |
| `nablarch://guide/handler-queue` | ハンドラキューガイド |
| `nablarch://guide/database` | データベースガイド |
| `nablarch://guide/validation` | バリデーションガイド |
| `nablarch://guide/error-handling` | エラーハンドリングガイド |
| `nablarch://guide/testing` | テスティングガイド |

## 利用可能なPrompts（6個）

| Prompt名 | 説明 |
|-----------|------|
| `setup-handler-queue` | ハンドラキュー構成のセットアップ |
| `create-action` | Nablarchアクションクラスのスケルトン生成 |
| `explain-handler` | 特定ハンドラの詳細説明 |
| `review-config` | XML設定ファイルのレビュー |
| `migration-guide` | バージョン間移行ガイド |
| `best-practices` | 特定トピックのベストプラクティス |

## トラブルシューティング

### 起動失敗: ONNXモデルエラー

```
Error creating bean with name 'bgeM3OnnxEmbeddingClient': Invocation of init method failed
```

**原因**: `provider=local`（デフォルト）で`/opt/models/bge-m3/model.onnx`が存在しない
**対策**: `--nablarch.mcp.embedding.provider=api`を引数に追加

### 起動失敗: PostgreSQL接続エラー

```
Unable to obtain connection from database
```

**原因**: PostgreSQLが起動していない、またはnablarch_mcpデータベースが存在しない
**対策**:
1. PostgreSQLの稼働確認: `pg_isready` または `systemctl status postgresql`
2. DB存在確認: `psql -U nablarch -d nablarch_mcp -c '\dt'`
3. DB接続情報は環境変数で上書き可能: `DB_USERNAME`, `DB_PASSWORD`

### MCP接続タイムアウト

**原因**: 起動に約10秒かかるため、Claude Codeのタイムアウトに引っかかる可能性
**対策**: 初回接続時にリトライを待つ。通常は自動リトライで解決

### RAG機能（semanticSearch等）のエラー

**原因**: `provider=api`でEmbedding APIキーが未設定
**対策**: RAG機能はセマンティック検索に依存。現在の設定では静的知識ベース（YAML）のみ利用可能。RAG機能を有効化するにはONNXモデルの配置またはAPIキーの設定が必要

## 再セットアップ手順

リポジトリ更新やjarの再ビルドが必要な場合:

```bash
# 1. 最新コード取得
cd ~/nablarch-mcp-server
git checkout main && git pull origin main

# 2. 再ビルド
./mvnw clean package -DskipTests

# 3. Claude Codeセッション再起動（MCPサーバーが新しいjarを読み込む）
# 各エージェントのセッションで /clear を実行
```

MCP設定自体の変更は不要（jarパスが変わらないため）。

## 制限事項

- ONNXモデル未配置のため、`provider=api`で運用（Embedding API非呼び出し）
- RAG機能（semanticSearch等のベクトル検索部分）は利用不可（静的知識ベースの検索のみ）
- 起動に約10秒かかる（Spring Boot + 知識ベース初期化）
- セッション毎にJavaプロセスが起動されるため、メモリ使用量に注意（約300MB/プロセス）
