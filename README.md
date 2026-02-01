# Nablarch AI時代戦略 調査・研究成果物

Nablarchフレームワークの生成AI時代における繁栄戦略プロジェクトの調査レポート、ナレッジベース、スキル定義を管理するリポジトリです。

## 概要

TIS社のJavaフレームワーク「Nablarch」に対して、AIコーディングツールとの統合を中心とした4つの戦略領域を調査・計画しています。

### 4つの戦略領域

1. **MCPサーバー構築** — RAG-enhanced Nablarch MCPサーバーによるAIコーディング支援
2. **RAGシステム構築** — Nablarchドキュメント・ソースコードのベクトル化と検索基盤
3. **ハンドラキュー自動設計** — 要件定義からNablarchハンドラキュー構成を自動生成
4. **AI品質ゲート** — AI生成コードに対する多層品質検証パイプライン

## 成果物一覧

### 計画文書（cmd_001）

| 番号 | ファイル | 概要 |
|------|---------|------|
| O-001 | [output/O-001_nablarch_mcp_server_plan.md](output/O-001_nablarch_mcp_server_plan.md) | Nablarch MCPサーバー構築計画。アーキテクチャ、技術スタック、フェーズ分けを含む |
| O-002 | [output/O-002_nablarch_rag_system_plan.md](output/O-002_nablarch_rag_system_plan.md) | Nablarch RAGシステム構築計画。パイプライン設計、ベクトルDB選定、UX設計 |
| O-003 | [output/O-003_nablarch_handler_queue_tool_plan.md](output/O-003_nablarch_handler_queue_tool_plan.md) | ハンドラキュー自動設計ツール計画。入出力フォーマット、制約エンジン設計 |
| O-004 | [output/O-004_nablarch_quality_gate_plan.md](output/O-004_nablarch_quality_gate_plan.md) | AI品質ゲート計画。SpotBugsプラグイン拡張、Excelテスト自動生成、CI/CD統合 |

### ナレッジベース（cmd_007）

6足軽並列出撃による網羅的情報収集。合計264KB。

| 番号 | ファイル | サイズ | 概要 |
|------|---------|--------|------|
| O-005 | [output/O-005_nablarch_kb_official_docs.md](output/O-005_nablarch_kb_official_docs.md) | 59KB | 公式ドキュメント全体像。7アプリ種別ハンドラキュー構成、17カテゴリライブラリ |
| O-006 | [output/O-006_nablarch_kb_architecture.md](output/O-006_nablarch_kb_architecture.md) | 62KB | アーキテクチャ詳細。全6種別ハンドラキュー構成、ハンドラFQCN一覧、XML定義 |
| O-007 | [output/O-007_nablarch_kb_dev_standards.md](output/O-007_nablarch_kb_dev_standards.md) | 51KB | 開発標準v2.3、Checkstyle57ルール、Excelテスト詳細、JUnit5拡張 |
| O-008 | [output/O-008_nablarch_kb_github.md](output/O-008_nablarch_kb_github.md) | 35KB | GitHub全106リポジトリの10カテゴリ分類、メトリクス、依存関係 |
| O-009 | [output/O-009_nablarch_kb_fintan.md](output/O-009_nablarch_kb_fintan.md) | 31KB | Fintanコンテンツ25+ページ。テスト観点カタログ、AI自動化事例、ライセンス |
| O-010 | [output/O-010_nablarch_kb_community.md](output/O-010_nablarch_kb_community.md) | 25KB | コミュニティ分析。Qiita14記事、GitHub Stars42、Spring比較、CVE記録 |

### 調査レポート（cmd_011, cmd_013, cmd_015）

| 番号 | ファイル | 行数 | 概要 |
|------|---------|------|------|
| O-022 | [output/O-022_nablarch_mcp_feasibility_study.md](output/O-022_nablarch_mcp_feasibility_study.md) | 446行 | MCPサーバーのNablarch実装可能性調査。結論: 条件付き可能だがSpring Boot確定（cmd_016） |
| O-023 | [output/O-023_nablarch_rag_mcp_analysis.md](output/O-023_nablarch_rag_mcp_analysis.md) | 1190行 | RAGとMCPの関連性分析。7ユースケース、フロー図20枚以上、3パターン比較 |
| O-024 | [output/O-024_nablarch_batch_training.md](output/O-024_nablarch_batch_training.md) | 926行 | Nablarchバッチエンジニア育成学習計画。4レベルロードマップ、9ステップ、URL19件検証済み |

### スキル定義（cmd_002, cmd_008, cmd_009, cmd_018）

Claudeエージェントが再利用可能なタスク実行パターンを定義したスキルファイル。

| 番号 | スキル名 | 行数 | 概要 |
|------|----------|------|------|
| O-011 | [shogun-framework-mcp-server-planner](skills/shogun-framework-mcp-server-planner/) | — | 任意FW向けMCPサーバー構築計画の策定 |
| O-012 | [shogun-rag-system-planner](skills/shogun-rag-system-planner/) | — | 任意技術のRAGシステム構築計画の策定 |
| O-013 | [shogun-nablarch-handler-queue-designer](skills/shogun-nablarch-handler-queue-designer/) | — | Nablarchハンドラキュー自動設計 |
| O-014 | [shogun-nablarch-quality-gate-setup](skills/shogun-nablarch-quality-gate-setup/) | — | Nablarch品質ゲートCI/CDセットアップ |
| O-015 | [shogun-fintan-content-scraper](skills/shogun-fintan-content-scraper/) | — | Fintan等ナレッジサイトのコンテンツ網羅収集・Markdown化 |
| O-016 | [shogun-community-research-report](skills/shogun-community-research-report/) | 739行 | 技術FW/ツールのコミュニティ情報を体系的に調査・レポート化 |
| O-017 | [shogun-nablarch-architecture-explorer](skills/shogun-nablarch-architecture-explorer/) | 823行 | Nablarch公式ドキュメント・GitHub・Javadocから技術情報収集 |
| O-018 | [shogun-kb-builder](skills/shogun-kb-builder/) | 914行 | 技術ナレッジベース構築の調査・文書化パターン |
| O-026 | [shogun-project-scaffold](skills/shogun-project-scaffold/) | 1135行 | 5言語対応プロジェクトスケルトン自動生成（S-003、S-001+S-002統合） |
| O-027 | [shogun-framework-knowledge-builder](skills/shogun-framework-knowledge-builder/) | 1091行 | FW技術ナレッジベース包括構築（S-012、S-006+S-007+S-008+S-009統合） |

## 運用ルール

1. **調査タスク完了時**: O-XXXファイルをこのリポジトリにコミット → プッシュ → PR作成
2. **スキル化承認時**: スキルをこのリポジトリ内のskills/に作成 → プッシュ → PR作成
3. **README更新**: 新しい成果物追加時にこのREADME.mdに説明を追記
4. **PR**: タイトル・本文は日本語で記述

## ライセンス

本リポジトリの調査レポートは独自の分析・整理を含みますが、参照元の情報は各ファイル内に記載のソースURLに帰属します。

- Nablarch公式ドキュメント: Apache License 2.0
- Fintanコンテンツ: CC BY 4.0（ドキュメント）/ Apache 2.0（コード）/ CC BY-SA 4.0（開発標準）
