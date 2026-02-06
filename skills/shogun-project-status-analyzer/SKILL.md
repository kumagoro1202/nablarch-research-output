---
name: project-status-analyzer
description: プロジェクトの現状を体系的に調査し、残タスク・完了定義・実装計画を報告する。「プロジェクトの状況を調査して」「残タスクを洗い出して」「実装完了までに何が必要か調べて」「プロジェクトの現状分析をして」といった要望に対応する。README/構造/設定/計画ドキュメント/テスト状況を順次確認し、「実装完了」の定義特定と残タスク一覧を出力するプロジェクト分析スキル。
---

# Project Status Analyzer — プロジェクト現状分析パターン

## Overview

プロジェクトの現状を体系的に調査し、以下を明らかにするスキル:

1. プロジェクト概要・目的の把握
2. 現在の実装状況（完了済み機能）
3. 「実装完了」の定義を特定
4. 残タスク一覧（優先度付き）
5. 推定工数（小/中/大）
6. 実装計画の提案

**主な用途:**
- 引き継いだプロジェクトの現状把握
- 実装完了までのロードマップ作成
- プロジェクト健全性の評価
- リソース見積もりの基礎情報収集

**このスキルの特徴:**
- 標準化された調査手順（README→構造→設定→計画→残タスク）
- 「実装完了」定義の明確化
- 優先度付き残タスク一覧
- 実装計画の提案まで含む包括的分析

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「プロジェクトの現状を調査して」
- 「残タスクを洗い出して」
- 「実装完了までに何が必要か調べて」
- 「プロジェクトの状況を分析して」
- 「○○プロジェクトの健全性をチェックして」
- 「このプロジェクトを完成させるには何が必要？」
- 引き継いだプロジェクトの全体像を把握したい場合
- 実装計画を立てる前に現状を正確に把握したい場合

**トリガーキーワード**: 現状調査, 残タスク, 実装完了, 状況分析, プロジェクト分析, 健全性チェック

## Instructions

### Phase 1: 基本情報収集

#### Step 1.1: README.mdの確認

```
【実行手順】

1. README.mdを読み込む
   Read: {project_path}/README.md

2. 以下を抽出:
   - プロジェクト名
   - 目的・概要（1-2文）
   - 主要機能一覧
   - 技術スタック
   - セットアップ手順
   - 使用方法

3. 「完了」の定義に関する記述を探す
   - "TODO", "WIP", "Coming Soon" 等のマーカー
   - ロードマップ、マイルストーン
   - v1.0 リリース条件
```

#### Step 1.2: CLAUDE.mdの確認（あれば）

```
【実行手順】

1. CLAUDE.mdの存在確認と読み込み
   Read: {project_path}/CLAUDE.md

2. 以下を抽出:
   - 開発ガイドライン
   - コーディング規約
   - 禁止事項
   - テスト要件
```

### Phase 2: 構造分析

#### Step 2.1: ディレクトリ構造の確認

```
【実行手順】

1. プロジェクト構造を確認
   tree -L 3 {project_path} または ls -laR

2. 以下を特定:
   - ソースコードディレクトリ（src/, lib/, app/）
   - テストディレクトリ（tests/, test/, __tests__/）
   - 設定ファイル（config/, .env.example）
   - ドキュメント（docs/, doc/）
   - ビルド成果物（dist/, build/, target/）
```

#### Step 2.2: ビルド設定の確認

```
【実行手順】

1. ビルド設定ファイルを読み込む
   - Java/Maven: pom.xml
   - Java/Gradle: build.gradle
   - Node.js: package.json
   - Python: pyproject.toml, setup.py
   - Rust: Cargo.toml
   - Go: go.mod

2. 以下を確認:
   - 依存関係
   - ビルドコマンド
   - テストコマンド
   - スクリプト定義
```

### Phase 3: 計画ドキュメント確認

#### Step 3.1: 計画系ファイルの探索

```
【確認対象ファイル】

優先度高:
- WBS.md, wbs.md
- ROADMAP.md, roadmap.md
- TODO.md, todo.md
- PLAN.md, plan.md
- TASKS.md, tasks.md

優先度中:
- docs/plan.md
- docs/roadmap.md
- .github/ROADMAP.md

優先度低:
- GitHub Issues
- GitHub Projects
```

#### Step 3.2: 既存テスト状況の確認

```
【実行手順】

1. テストファイルの確認
   Glob: {project_path}/tests/**/*.{py,java,ts,js}
   または
   Glob: {project_path}/test/**/*.{py,java,ts,js}

2. テスト実行（可能であれば）
   - Python: pytest tests/ -v --collect-only
   - Java: mvn test -DskipTests=false
   - Node.js: npm test -- --listTests

3. テストカバレッジの確認（あれば）
   - coverage.xml
   - lcov.info
   - jacoco/
```

### Phase 4: 残タスク特定

#### Step 4.1: NotImplementedError/TODO の検出

```
【実行手順】

1. 未実装マーカーの検索
   Grep: "NotImplementedError" in {project_path}/src/
   Grep: "TODO" in {project_path}/src/
   Grep: "FIXME" in {project_path}/src/
   Grep: "raise NotImplementedError" in {project_path}/
   Grep: "pass  # TODO" in {project_path}/

2. テストスタブの検索
   Grep: "@pytest.mark.skip" in {project_path}/tests/
   Grep: "@Ignore" in {project_path}/src/test/
   Grep: "test.skip" in {project_path}/tests/
```

#### Step 4.2: 「実装完了」の定義特定

```
【分析観点】

1. 明示的な完了条件
   - README.mdの「Goals」「Objectives」セクション
   - ROADMAP.mdのマイルストーン
   - GitHub Milestones

2. 暗黙的な完了条件
   - 全テストがパス
   - NotImplementedErrorが0
   - ドキュメントが整備されている
   - CI/CDが正常動作

3. プロジェクトタイプ別の完了条件
   - ライブラリ: API完備、ドキュメント完備、テストカバレッジ
   - アプリケーション: 全機能実装、E2Eテスト、デプロイ可能
   - テンプレート: サンプル完備、セットアップ手順明確
```

### Phase 5: レポート作成

#### Step 5.1: 残タスク一覧の作成

```
【フォーマット】

| ID | タスク | 優先度 | 工数 | 依存 |
|----|--------|--------|------|------|
| T-01 | {タスク名} | 高/中/低 | 小/中/大 | - |
| T-02 | {タスク名} | 高/中/低 | 小/中/大 | T-01 |

【優先度基準】
- 高: ブロッカー、コア機能、他タスクの依存元
- 中: 重要機能、テスト、ドキュメント
- 低: Nice-to-have、最適化、リファクタリング

【工数基準】
- 小: 1-2時間程度
- 中: 半日〜1日程度
- 大: 2日以上
```

#### Step 5.2: 実装計画の提案

```
【フォーマット】

## 実装計画

### Phase 1: {フェーズ名}（並列可能）
- T-01: {タスク}
- T-02: {タスク}

### Phase 2: {フェーズ名}（Phase 1完了後）
- T-03: {タスク}
- T-04: {タスク}

### Phase 3: 最終確認
- 全テスト実行
- ドキュメント整備
- PR作成
```

## Output Format

```
═══════════════════════════════════════════════════════════════
【プロジェクト現状分析レポート】
═══════════════════════════════════════════════════════════════

■ プロジェクト概要
名称: {project_name}
パス: {project_path}
概要: {1-2文の説明}

■ 技術スタック
- 言語: {言語}
- フレームワーク: {FW}
- ビルドツール: {ビルドツール}
- テストFW: {テストFW}

■ 現在の実装状況
完了済み:
- {完了機能1}
- {完了機能2}

未完了:
- {未完了機能1}
- {未完了機能2}

■ 「実装完了」の定義
{明確な完了条件を記載}

■ 残タスク一覧
| ID | タスク | 優先度 | 工数 |
|----|--------|--------|------|
| T-01 | {タスク} | 高 | 中 |
| T-02 | {タスク} | 中 | 小 |

■ 推定総工数: {小/中/大}

■ 実装計画
Phase 1: {内容}
Phase 2: {内容}

■ テスト実行コマンド
{ビルド/テストコマンド}

■ 備考
{追加メモ、リスク、注意点}

═══════════════════════════════════════════════════════════════
```

## Examples

### Example 1: Pythonライブラリの分析

```
ユーザー: ~/my-python-lib の現状を調査して

実行:
1. README.md確認 → "データ変換ライブラリ"
2. pyproject.toml確認 → Python 3.11, pytest
3. src/確認 → 5モジュール中2モジュールがNotImplementedError
4. tests/確認 → 15テスト中9がskip

レポート:
- 完了: core, utils, models
- 未完了: transformers, exporters
- 残タスク: T-01〜T-05
- 推定工数: 中
```

### Example 2: 複数プロジェクトの比較分析

```
ユーザー: project-a と project-b の状況を比較して

実行:
1. 両プロジェクトを並列調査
2. 残タスク数、工数を比較
3. どちらを先に完了すべきか提案

レポート:
- project-a: 残タスク3件、工数小
- project-b: 残タスク8件、工数大
- 提案: project-aを先に完了させることを推奨
```

## Notes

- 調査は実装を伴わない（読み取り専用）
- 計画提案は家老/将軍の承認後に実行
- テスト実行は環境が整っている場合のみ
- 機密情報（API キー等）を含むファイルは報告に含めない
- 複数プロジェクトの調査は並列実行可能（足軽に分担）
