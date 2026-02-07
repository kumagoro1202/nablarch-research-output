---
name: project-registration
description: 新規プロジェクトを調査し、context/{project}.md と projects/{project}.yaml を自動生成してshogun管理下に登録する。「プロジェクトを登録して」「新しいリポジトリを管理下に追加して」「○○プロジェクトをshogunに登録して」といった要望に対応する。README.md解析、技術スタック特定、プロジェクト状況判定を自動化し、config/projects.yamlへのエントリ追加まで一気通貫で実行するスキル。
---

# Project Registration — プロジェクト登録パターン

## Overview

新規プロジェクトをshogun管理システムに登録するスキル。プロジェクトの調査から登録まで以下を自動化する:

1. プロジェクトディレクトリの調査（README.md、構成ファイル）
2. 技術スタックの自動特定
3. プロジェクト状況の判定（active/planning/completed等）
4. context/{project}.md の生成（足軽向け要約情報）
5. projects/{project}.yaml の生成（詳細情報）
6. config/projects.yaml へのエントリ追加

**主な用途:**
- 新規プロジェクトのshogun管理下への登録
- 複数プロジェクトの一括登録
- プロジェクト情報の標準化

**このスキルの特徴:**
- README.md解析による自動情報抽出
- 技術スタック自動検出（package.json, pom.xml, pyproject.toml等）
- プロジェクト状況の自動判定
- 3層ファイル構造の自動生成（config, context, projects）

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「プロジェクトを登録して」
- 「○○をshogun管理下に追加して」
- 「新しいリポジトリを登録して」
- 「○○プロジェクトの情報を登録して」
- 「複数プロジェクトを一括登録して」
- 新規にクローンしたリポジトリをshogunで管理したい場合
- 外部プロジェクトをshogunの管理対象に追加したい場合

**トリガーキーワード**: プロジェクト登録, 管理下に追加, リポジトリ登録, shogun登録

## Instructions

### Phase 1: プロジェクト調査

#### Step 1.1: 基本情報の収集

```
【実行手順】

1. プロジェクトディレクトリの存在確認
   ls -la {project_path}

2. README.mdの読み込み
   Read: {project_path}/README.md
   → プロジェクト名、説明、目的を抽出

3. 構成ファイルの確認（技術スタック特定）
   - package.json → Node.js/TypeScript
   - pom.xml → Java/Maven
   - build.gradle → Java/Gradle
   - pyproject.toml / setup.py → Python
   - Cargo.toml → Rust
   - go.mod → Go
   - Gemfile → Ruby

4. ディレクトリ構造の把握
   tree -L 2 {project_path} (または ls -la)
```

#### Step 1.2: プロジェクト状況の判定

```
【判定基準】

| 状況 | 条件 |
|------|------|
| active | 最近のコミットあり（30日以内）、開発中機能あり |
| planning | READMEのみ、ソースコード少ない、WIP状態 |
| completed | 安定版リリース済み、メンテナンスモード |
| archived | 更新停止、非推奨明記 |
| template | テンプレートリポジトリ、Javaソースなし |

【確認コマンド】
git log --oneline -5  # 最近のコミット確認
git describe --tags   # タグ確認
```

### Phase 2: ファイル生成

#### Step 2.1: context/{project}.md の生成

```markdown
# {project_name} コンテキスト

## 概要
{README.mdから抽出した1-2文の説明}

## 技術スタック
- 言語: {検出した言語}
- フレームワーク: {検出したFW}
- ビルドツール: {検出したビルドツール}
- テストFW: {検出したテストFW}

## プロジェクト構造
{主要ディレクトリの説明}

## 開発コマンド
- ビルド: {ビルドコマンド}
- テスト: {テストコマンド}
- 実行: {実行コマンド}

## 注意事項
{README.mdから抽出した注意点、または判明した制約}
```

#### Step 2.2: projects/{project}.yaml の生成

```yaml
# {project_name} 詳細情報
# Git管理外（機密情報を含む可能性）

project:
  id: {project_id}
  name: {project_name}
  path: {absolute_path}

  # 基本情報
  description: |
    {詳細な説明}

  # 技術情報
  tech_stack:
    language: {言語}
    framework: {FW}
    build_tool: {ビルドツール}
    test_framework: {テストFW}

  # 状況
  status: {active|planning|completed|archived|template}
  priority: {high|medium|low}

  # 関連情報
  repository:
    url: {GitHubのURL（あれば）}
    branch: {デフォルトブランチ}

  # タスク（任意）
  tasks: []

  # メモ
  notes: |
    {追加メモ}
```

#### Step 2.3: config/projects.yaml への追加

```yaml
# 以下のエントリを追加
- id: {project_id}
  name: "{project_name}"
  path: "{absolute_path}"
  priority: {high|medium|low}
  status: {active|planning|completed|archived|template}
```

### Phase 3: 検証

```
【チェックリスト】

□ context/{project}.md が作成されている
□ projects/{project}.yaml が作成されている
□ config/projects.yaml にエントリが追加されている
□ プロジェクトIDが他と重複していない
□ パスが正しい絶対パスである
□ 技術スタックが正確に記載されている
```

## Output Format

```
【登録完了レポート】

プロジェクト: {project_name}
ID: {project_id}
パス: {absolute_path}

技術スタック:
- 言語: {言語}
- FW: {フレームワーク}
- ビルド: {ビルドツール}

状況: {status}
優先度: {priority}

生成ファイル:
- context/{project_id}.md
- projects/{project_id}.yaml
- config/projects.yaml (更新)
```

## Examples

### Example 1: Pythonプロジェクトの登録

```
ユーザー: ~/my-python-app をshogunに登録して

実行:
1. README.md読み込み → "FastAPIベースのREST API"
2. pyproject.toml検出 → Python 3.11, FastAPI, pytest
3. git log確認 → 3日前にコミットあり → active
4. ファイル生成:
   - context/my-python-app.md
   - projects/my-python-app.yaml
   - config/projects.yaml更新
```

### Example 2: 複数プロジェクトの一括登録

```
ユーザー: ~/project-a, ~/project-b, ~/project-c を登録して

実行:
1. 各プロジェクトを並列調査（足軽3人に分担可能）
2. 各プロジェクトのファイル生成
3. config/projects.yaml に3エントリ追加
4. 登録完了レポート出力
```

## Notes

- projects/{project}.yaml はGit管理外（.gitignoreに追加済み）
- クライアント情報等の機密情報はprojects/*.yamlにのみ記載
- context/*.md は足軽が参照する軽量な要約情報
- プロジェクトIDはディレクトリ名をそのまま使用（kebab-case推奨）
