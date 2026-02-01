# Nablarch MCPサーバー構築計画

> **作成日**: 2026-01-31
> **タスクID**: subtask_001 (parent: cmd_001)
> **作成者**: ashigaru1（シニアソフトウェアエンジニア ペルソナ）

---

## 目次

1. [現状調査結果](#1-現状調査結果)
   - [1.1 MCP仕様のまとめ](#11-mcp仕様のまとめ)
   - [1.2 Nablarchの特性分析](#12-nablarchの特性分析)
2. [具体的な実装計画](#2-具体的な実装計画)
   - [2.1 アーキテクチャ設計](#21-アーキテクチャ設計)
   - [2.2 技術スタック](#22-技術スタック)
   - [2.3 公開リソース一覧](#23-公開リソース一覧)
3. [段階的な構築ロードマップ](#3-段階的な構築ロードマップ)
4. [実現可能性の評価](#4-実現可能性の評価)
5. [リスクと対策](#5-リスクと対策)
6. [参考資料](#6-参考資料)

---

## 1. 現状調査結果

### 1.1 MCP仕様のまとめ

#### MCP（Model Context Protocol）とは

MCPはAnthropicが2024年11月に発表した、AIアプリケーションと外部システムを接続するためのオープン標準プロトコルである。Linux FoundationのAgentic AI Foundationに参画し、OpenAI・Google DeepMindなど主要AIプロバイダーにも採用されている。

#### アーキテクチャ

MCPはクライアント-サーバーアーキテクチャを採用する。

```
┌─────────────────────────────────────────┐
│          MCP Host（AIアプリケーション）       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │MCP Client│ │MCP Client│ │MCP Client│ │
│  └─────┬────┘ └─────┬────┘ └─────┬────┘ │
└────────┼────────────┼────────────┼──────┘
         │            │            │
    ┌────▼────┐  ┌────▼────┐  ┌───▼─────┐
    │MCP Server│  │MCP Server│  │MCP Server│
    │(Local)   │  │(Local)   │  │(Remote)  │
    └─────────┘  └─────────┘  └─────────┘
```

- **MCP Host**: AIアプリケーション（Claude Desktop, VS Code, Claude Code等）
- **MCP Client**: サーバーとの接続を維持するコンポーネント
- **MCP Server**: コンテキストをクライアントに提供するプログラム

#### プロトコル層構造

| 層 | 役割 | 詳細 |
|---|---|---|
| **データ層** | JSON-RPC 2.0ベースのプロトコル | ライフサイクル管理、プリミティブ（Tools/Resources/Prompts）、通知 |
| **トランスポート層** | 通信チャネル管理 | STDIO（ローカル）、Streamable HTTP（リモート） |

#### 3つのコアプリミティブ

| プリミティブ | 制御主体 | 用途 | 例 |
|---|---|---|---|
| **Tools** | AIモデル | AIが呼び出す実行可能な関数 | API呼び出し、DB操作、ファイル操作 |
| **Resources** | アプリケーション | 読み取り専用のデータソース | ファイル内容、DBスキーマ、APIドキュメント |
| **Prompts** | ユーザー | 再利用可能なテンプレート | タスク実行のガイド、Few-shot例 |

#### MCP Java SDK

公式Java SDKはSpring AIとの共同メンテナンス下にある。

- **最新バージョン**: v0.17.2（2026年1月22日リリース）
- **Java要件**: Java 17以上
- **プログラミングモデル**: Project Reactorによるリアクティブストリーム
- **JSON処理**: Jackson
- **サーバートランスポート**: Jakarta Servlet（コア）、Spring WebFlux/WebMVC（オプション）
- **Spring Boot Starters**:
  - `spring-ai-starter-mcp-server`（STDIO）
  - `spring-ai-mcp-server-webmvc-spring-boot-starter`（WebMVC SSE）
  - `spring-ai-mcp-server-webflux-spring-boot-starter`（WebFlux SSE）

### 1.2 Nablarchの特性分析

#### フレームワーク概要

NablarchはTIS株式会社が開発した、ミッションクリティカルシステム構築の知見を集約したJavaアプリケーション開発・実行基盤である。Apache License 2.0のOSSとして公開されている。

- **最新バージョン**: Nablarch 6u3
- **Java要件**: Java 17以上（Java 21対応）
- **準拠規格**: Jakarta EE 10
- **GitHub**: 113リポジトリ（nablarch organization）

#### 対応アプリケーション種別

| 種別 | 説明 |
|---|---|
| Webアプリケーション | サーブレットベースのWeb UI |
| RESTful Webサービス | REST API提供 |
| バッチアプリケーション | 定期実行・常駐バッチ |
| メッセージング | キューベースの非同期処理 |

#### ハンドラキューアーキテクチャ（最重要特性）

Nablarchの根幹をなすアーキテクチャ。サーブレットフィルタチェーンと同様のパイプライン処理モデルである。

```
Request → [Handler1] → [Handler2] → [Handler3] → ... → [業務Action]
                                                              │
Response ← [Handler1] ← [Handler2] ← [Handler3] ← ... ← [業務Action]
```

**特徴**:
- リクエスト時は先頭から順に処理
- レスポンス時は逆順に処理（スタック構造）
- 各ハンドラは単一責任（フィルタリング、変換、リソース管理等）
- インターセプタによる動的なハンドラ追加が可能
- ハンドラの順序制約が存在（ドキュメント参照必須）

**主要ハンドラ例**:
- アクセス権限検証
- リクエスト/レスポンス変換
- DB接続管理（リソース取得・解放）
- DispatchHandler（URIベースのアクション特定）

#### ライブラリ群

ハンドラから呼び出されるコンポーネント群:
- データベースアクセス（独自実装、後方互換性維持）
- ファイルアクセス
- ログ出力
- バリデーション
- コード管理

#### 開発支援体系

Nablarchはフレームワーク単体ではなく、以下を包括的に提供:
- **開発標準**: 要件定義フレームワーク、テスト観点カタログ、コーディング規約
- **開発ツール**: Collaborage（クラウド開発環境テンプレート）、ブランクプロジェクト、テスティングフレームワーク（Excel連携テスト自動化）
- **ガイド**: システム開発ガイド、SPA + REST APIアーキテクチャリファレンス
- **静的解析**: コーディングスタイルチェックツール

#### 設計方針

- スレッドセーフティ重視
- コレクション/配列のnull非返却原則
- 検査例外の非使用
- 外部OSSへの非依存（コアコンポーネント）
- 後方互換性の維持

---

## 2. 具体的な実装計画

### 2.1 アーキテクチャ設計

#### 全体アーキテクチャ

```
┌──────────────────────────────────────────────────────────┐
│              AI コーディングツール                           │
│    (Claude Code / Copilot / Cursor / VS Code)            │
│  ┌────────────────────────────────────────────────────┐  │
│  │                   MCP Client                       │  │
│  └──────────────────────┬─────────────────────────────┘  │
└─────────────────────────┼────────────────────────────────┘
                          │ JSON-RPC 2.0
                          │ (STDIO / Streamable HTTP)
┌─────────────────────────▼────────────────────────────────┐
│              Nablarch MCP Server                          │
│                                                           │
│  ┌─────────┐  ┌──────────┐  ┌─────────┐               │
│  │  Tools  │  │ Resources│  │ Prompts │               │
│  │         │  │          │  │         │               │
│  └────┬────┘  └────┬─────┘  └────┬────┘               │
│       │            │             │                     │
│  ┌────▼────────────▼─────────────▼─────────────────┐  │
│  │           Nablarch Knowledge Base                │  │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐       │  │
│  │  │ API  │ │設計   │ │Handler│ │コード     │       │  │
│  │  │ Docs │ │Pattern│ │Queue │ │Template  │       │  │
│  │  └──────┘ └──────┘ └──────┘ └──────────┘       │  │
│  └─────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

#### サーバー構成方針

1. **Java実装**（MCP Java SDK + Spring Boot）
   - NablarchはJavaフレームワークであり、Java SDKとの親和性が最も高い
   - Spring AI MCP Boot Starterを活用した効率的な開発
2. **デュアルトランスポート対応**
   - STDIO: ローカル開発用（Claude Desktop, Claude Code向け）
   - Streamable HTTP: リモート/チーム共有用
3. **知識ベースの静的構築 + 動的クエリ**
   - Nablarchドキュメント・API仕様を事前インデックス化
   - ユーザーの質問に応じた動的検索・生成

### 2.2 技術スタック

| コンポーネント | 技術選定 | 理由 |
|---|---|---|
| **言語** | Java 17+ | Nablarchとの一貫性、MCP Java SDK要件 |
| **フレームワーク** | Spring Boot 3.3+ | MCP Boot Starter活用、エコシステム |
| **MCP SDK** | MCP Java SDK v0.17+ | 公式SDK、Spring AI共同メンテナンス |
| **サーバートランスポート** | STDIO + WebMVC | ローカル＆リモート両対応 |
| **知識ベース** | Embedded（JSON/YAML） | 軽量、依存最小化 |
| **検索エンジン** | Apache Lucene（埋め込み） | 高速全文検索、Java純正 |
| **テンプレートエンジン** | Handlebars.java | コード生成テンプレート用 |
| **ビルド** | Gradle / Maven | Nablarch標準に合わせMavenも可 |
| **テスト** | JUnit 5 + MCP Inspector | 単体テスト＋プロトコル検証 |

### 2.3 公開リソース一覧

#### Tools（AIが呼び出す関数）

| ツール名 | 説明 | 入力 | 出力 |
|---|---|---|---|
| `generate_handler` | ハンドラキュー構成の生成 | アプリ種別、要件 | XMLハンドラキュー定義 |
| `generate_action` | 業務アクションクラスの生成 | アクション名、ルーティング情報 | Javaソースコード |
| `generate_form` | フォームBean/Entity生成 | テーブル定義、バリデーション要件 | Javaソースコード |
| `generate_sql` | SQL定義ファイル生成 | エンティティ情報、クエリ種別 | SQLファイル |
| `validate_handler_queue` | ハンドラキュー構成の妥当性検証 | XML定義 | 検証結果、修正提案 |
| `search_api` | Nablarch API検索 | キーワード、カテゴリ | API情報一覧 |
| `generate_test` | テストコード生成 | テスト対象クラス、テスト種別 | JUnitテストコード |

#### Resources（読み取り専用データ）

| リソースURI | 説明 | MIME |
|---|---|---|
| `nablarch://api/{module}/{class}` | API Javadocリファレンス | text/markdown |
| `nablarch://handler/{type}` | ハンドラ仕様（Web/Batch/REST等） | text/markdown |
| `nablarch://pattern/{name}` | 設計パターン・ベストプラクティス | text/markdown |
| `nablarch://antipattern/{name}` | アンチパターン集 | text/markdown |
| `nablarch://config/{type}` | 標準ハンドラキュー構成テンプレート | application/xml |
| `nablarch://library/{name}` | ライブラリ仕様（DB,Log,File等） | text/markdown |
| `nablarch://guide/{topic}` | 開発ガイド | text/markdown |
| `nablarch://example/{type}` | 実装例・サンプルコード | text/plain |
| `nablarch://version` | Nablarch最新バージョン情報 | application/json |

#### Prompts（再利用可能テンプレート）

| プロンプト名 | 説明 | パラメータ |
|---|---|---|
| `create-web-app` | Webアプリケーション新規作成ガイド | プロジェクト名、DB種別、機能一覧 |
| `create-rest-api` | RESTful API構築ガイド | エンドポイント一覧、認証方式 |
| `create-batch` | バッチアプリケーション構築ガイド | バッチ種別、入出力定義 |
| `setup-handler-queue` | ハンドラキュー設計支援 | アプリ種別、要件 |
| `review-code` | Nablarchコードレビュー | ソースコード、レビュー観点 |
| `troubleshoot` | トラブルシューティング支援 | エラー情報、環境情報 |

---

## 3. 段階的な構築ロードマップ

### Phase 1: 基盤構築（MVP）

**目標**: 最小限の動作するMCPサーバーを構築し、概念実証を行う

**成果物**:
- MCPサーバー基盤（STDIO トランスポート）
- 基本的なResources（API仕様、ハンドラ一覧）
- 基本的なTools（API検索、ハンドラキュー検証）
- Claude Desktop / Claude Codeでの動作検証

**技術タスク**:
1. Spring Boot + MCP Java SDKのプロジェクトスケルトン作成
2. Nablarch公式ドキュメントのクローリング・インデックス化
3. `nablarch://api/*` リソースの実装
4. `nablarch://handler/*` リソースの実装
5. `search_api` ツールの実装
6. `validate_handler_queue` ツールの実装
7. MCP Inspectorでの動作テスト

### Phase 2: コード生成機能

**目標**: AIによるNablarchコード生成の精度を大幅に向上させる

**成果物**:
- コード生成Tools一式（handler, action, form, sql, test）
- コード生成用テンプレート群
- 設計パターン・アンチパターンResources
- Prompts（create-web-app, create-rest-api, create-batch）

**技術タスク**:
1. Nablarchプロジェクト構造のテンプレート化
2. ハンドラキュー構成テンプレート（Web/REST/Batch/Messaging）作成
3. `generate_handler` ツールの実装
4. `generate_action` / `generate_form` / `generate_sql` ツールの実装
5. `generate_test` ツールの実装
6. 設計パターンResourcesの整備
7. Promptsの実装・チューニング
8. 生成コードの品質評価・ベンチマーク

### Phase 3: リモート展開・運用基盤

**目標**: チーム利用可能なリモートサーバーと運用基盤の構築

**成果物**:
- Streamable HTTPトランスポート対応
- チーム共有用サーバー構成
- 認証・認可機能
- コンテナ化パッケージ

**技術タスク**:
1. Streamable HTTPトランスポートの追加（WebMVC）
2. OAuth 2.0認証の実装
3. Docker/Kubernetes対応パッケージング
4. CI/CDパイプライン構築
5. 監視・ログ基盤の整備

### Phase 4: エコシステム拡張

**目標**: 開発ライフサイクル全体をカバーする統合環境

**成果物**:
- IDE統合（VS Code拡張、IntelliJ IDEA プラグイン）
- Nablarchプロジェクト分析機能
- 動的リソース更新（Nablarchバージョン追従）
- コミュニティ貢献用のプラグインシステム

**技術タスク**:
1. Nablarch Mavenリポジトリとの動的連携
2. プロジェクト構造分析ツールの実装
3. IDE統合モジュールの開発
4. プラグインアーキテクチャの設計・実装
5. ドキュメント・チュートリアルの整備
6. OSSコミュニティへの公開準備

---

## 4. 実現可能性の評価

### 技術的実現可能性: 高

| 評価項目 | 評価 | 根拠 |
|---|---|---|
| **MCP SDK成熟度** | ◎ | Java SDK v0.17.2が安定版、Spring AI共同メンテナンス |
| **Nablarch情報公開度** | ◎ | 全ドキュメント・ソースコードがOSS公開済み |
| **Java技術スタック整合** | ◎ | Nablarch=Java、MCP Java SDK=Java、自然な統合 |
| **先行事例** | ○ | MCP Serversリポジトリに多数のリファレンス実装あり |

### ビジネス的実現可能性: 高

| 評価項目 | 評価 | 根拠 |
|---|---|---|
| **市場ニーズ** | ◎ | AI支援開発の需要急増、Nablarch利用企業の生産性向上ニーズ |
| **差別化** | ◎ | Nablarch専用MCPサーバーは世界初、独自の強み |
| **TIS戦略整合** | ◎ | Nablarchエコシステム強化、AI支援開発の推進 |
| **導入障壁** | ○ | MCP対応AIツール（Claude, Copilot等）が前提 |

### Nablarch独自の強みを活かすポイント

1. **ハンドラキューの正確な理解**: 汎用フレームワーク（Spring等）では得られない、Nablarch固有のハンドラキュー設計・制約をAIが正しく扱えるようにする
2. **開発標準の組み込み**: TISの開発標準・コーディング規約をPrompts/Resourcesとして提供し、品質基準を自動的に満たすコード生成
3. **テスティングフレームワーク連携**: Nablarch独自のExcelベーステスト自動化との連携
4. **大規模基幹システム対応**: ミッションクリティカルシステム向けの設計パターン・ベストプラクティスの提供

---

## 5. リスクと対策

### 技術リスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| MCP仕様の破壊的変更 | 高 | 低 | SDK抽象化層を活用、バージョン固定＋段階的追従 |
| Nablarchバージョンアップ追従 | 中 | 中 | ドキュメント自動クローリング、バージョン別リソース管理 |
| コード生成の品質問題 | 高 | 中 | テンプレートベース生成、人間レビュー必須フロー、ベンチマーク整備 |
| パフォーマンス劣化（大規模知識ベース） | 中 | 低 | インデックス最適化、キャッシュ戦略、遅延ロード |

### ビジネスリスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| MCP普及の遅延 | 高 | 低 | マルチプロトコル対応（MCP＋独自API）の検討 |
| 競合の出現 | 中 | 中 | 先行者優位の確保、Nablarch知識の深さで差別化 |
| AIツール依存リスク | 中 | 低 | 複数AIツール対応（Claude, Copilot, Cursor等） |
| 知的財産リスク | 中 | 低 | OSS公開情報のみを利用、ライセンスコンプライアンス確認 |

### セキュリティリスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| プロンプトインジェクション | 高 | 中 | 入力検証、サンドボックス実行、ツール権限制限 |
| 機密情報漏洩（企業固有設定） | 高 | 低 | ローカル実行モード推奨、データ分離 |
| 不正なツール実行 | 高 | 低 | OAuth認証、ユーザー承認フロー、監査ログ |

---

## 6. 参考資料

### MCP関連

- [MCP公式サイト](https://modelcontextprotocol.io/)
- [MCP仕様書](https://modelcontextprotocol.io/specification/2025-11-25)
- [MCP Java SDK（GitHub）](https://github.com/modelcontextprotocol/java-sdk)
- [Spring AI MCP ドキュメント](https://docs.spring.io/spring-ai-mcp/reference/mcp.html)
- [MCP Server構築ガイド](https://modelcontextprotocol.io/docs/develop/build-server)
- [MCPリファレンスサーバー集](https://github.com/modelcontextprotocol/servers)
- [MCP Best Practices](https://modelcontextprotocol.info/docs/best-practices/)

### Nablarch関連

- [Nablarch公式ドキュメント](https://nablarch.github.io/docs/LATEST/doc/)
- [Nablarch GitHub Organization](https://github.com/nablarch)
- [Nablarch アーキテクチャ概要](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/nablarch/architecture.html)
- [Nablarch System Development Guide (Fintan)](https://fintan.jp/en/page/1667/)
- [Nablarch概要 (Fintan)](https://fintan.jp/en/page/1954/)

### その他

- [MCP Model Context Protocol 開発者ガイド 2026](https://publicapis.io/blog/mcp-model-context-protocol-guide)
- [MCP Java Server Guide](https://modelcontextprotocol.io/sdk/java/mcp-server)
