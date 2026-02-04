---
name: skill-deployment
description: スキルを複数格納先（nablarch-research-output/skills/、.claude/skills/）に同期コピーし、PR作成・マージまで一括実行するスキル。「スキルをデプロイして」「スキルを正式に格納したい」「スキルのPRを作成して」「スキルを複数の場所に配置して」「新しいスキルを運用ルールに従って格納して」といった要望に対応する。スキル運用の効率化を図る包括的デプロイメントスキル。
---

# Skill Deployment — スキル配置・PR作成一括実行

## Overview

Claude Code スキル（SKILL.md）を正式な格納先に配置し、Git操作（ブランチ作成・コミット・PR作成・マージ）までを一括で実行するスキル。マルチエージェント環境（multi-agent-shogun）において、スキルの運用を標準化・効率化する。

**スキル格納先:**

| 格納先 | 用途 | Git管理 |
|--------|------|---------|
| `nablarch-research-output/skills/` | 正式リポジトリ（成果物） | PR作成・マージ |
| `.claude/skills/` | ローカル利用（即時有効化） | コミット不要 |

**本スキルの特長:**

- 複数格納先への同期コピー
- ブランチ作成・PR作成・マージの自動化
- 複数スキルの一括デプロイ対応
- デプロイ前の検証（SKILL.md形式チェック）
- ロールバック手順の提供

**想定ワークフロー:**

```
┌─────────────────┐
│ スキル開発完了  │
│ (skills/配下)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Phase 1: 検証   │ ← SKILL.md形式チェック
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────────────┐
│.claude │ │nablarch-research│
│/skills │ │-output/skills  │
└────────┘ └───────┬────────┘
                   │
              ┌────┴────┐
              ▼         ▼
         ┌────────┐ ┌────────┐
         │ブランチ│→│PR作成  │→ マージ
         └────────┘ └────────┘
```

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「スキルをデプロイして」
- 「スキルを正式に格納したい」
- 「スキルのPRを作成して」
- 「スキルを複数の場所に配置して」
- 「新しいスキルを運用ルールに従って格納して」
- 「○○スキルをnablarch-research-outputに追加して」
- 「スキルをリリースして」
- 「スキルの配置作業をまとめてやって」
- 新規作成したスキルを正式リポジトリに格納する必要がある場合
- 複数のスキルを一括でデプロイする必要がある場合
- スキルのPR作成・マージを自動化したい場合

**トリガーキーワード**: スキルデプロイ, スキル格納, スキルPR, スキル配置, スキルリリース, skill deployment

## Input Format

```yaml
# スキルデプロイリクエスト
skills:
  - name: "shogun-example-skill"                    # スキル名（ディレクトリ名）
    source_path: "skills/shogun-example-skill"       # ソースパス（相対パス）
    # または絶対パス指定
    # source_path: "/home/kuma/multi-agent-shogun/skills/shogun-example-skill"

# 複数スキルの場合
skills:
  - name: "shogun-skill-a"
    source_path: "skills/shogun-skill-a"
  - name: "shogun-skill-b"
    source_path: "skills/shogun-skill-b"
  - name: "shogun-skill-c"
    source_path: "skills/shogun-skill-c"

# オプション
options:
  branch_prefix: "feature/add-skills"    # ブランチ名プレフィックス（デフォルト: feature/add-skills）
  commit_message: "feat: add new skills" # コミットメッセージ（省略時は自動生成）
  auto_merge: true                       # 自動マージ（デフォルト: false）
  skip_local_copy: false                 # .claude/skills/へのコピーをスキップ（デフォルト: false）
  dry_run: false                         # ドライラン（実際のGit操作を行わない）
```

## Instructions

### Phase 1: スキルファイルの存在確認と検証

#### Step 1.1: ソースファイルの確認

```
【実行手順】

1. 指定されたスキルディレクトリの存在確認:
   Glob: {source_path}/SKILL.md

2. SKILL.md の形式検証:
   - YAMLフロントマター（---で囲まれた部分）の存在
   - name フィールドの存在
   - description フィールドの存在
   - 本文（## Overview 等）の存在

3. 検証結果を記録:
   | スキル名 | SKILL.md | フロントマター | name | description | 検証結果 |
   |----------|----------|----------------|------|-------------|----------|
   | skill-a  | ✅       | ✅             | ✅   | ✅          | OK       |
   | skill-b  | ✅       | ❌             | -    | -           | NG       |
```

#### Step 1.2: YAMLフロントマター検証

```yaml
# 必須項目
---
name: "skill-name"           # 必須: スキル名
description: "説明文..."     # 必須: スキルの説明（skill-listに表示される）
---

# 検証ルール
- name: 英小文字、ハイフンのみ（例: example-skill）
- description: 1行で完結、要望例を含む
```

### Phase 2: nablarch-research-output/skills/ へのコピー

#### Step 2.1: ディレクトリ作成とコピー

```bash
# ディレクトリ作成
mkdir -p nablarch-research-output/skills/{skill_name}

# SKILL.md コピー
cp {source_path}/SKILL.md nablarch-research-output/skills/{skill_name}/

# 追加ファイルがある場合（テンプレート等）
cp -r {source_path}/* nablarch-research-output/skills/{skill_name}/
```

#### Step 2.2: コピー結果の確認

```bash
# ファイル一覧確認
ls -la nablarch-research-output/skills/{skill_name}/

# SKILL.md の内容確認（先頭20行）
head -20 nablarch-research-output/skills/{skill_name}/SKILL.md
```

### Phase 3: .claude/skills/ へのコピー（コミット不要）

#### Step 3.1: ローカルスキルディレクトリへのコピー

```bash
# ディレクトリ作成
mkdir -p .claude/skills/{skill_name}

# コピー
cp -r {source_path}/* .claude/skills/{skill_name}/
```

**注意:**
- `.claude/skills/` は `.gitignore` に含まれている（Git管理対象外）
- ローカルで即時にスキルが有効化される
- PR作成・マージは不要

#### Step 3.2: スキル有効化確認

```
【確認方法】

Claude Code で /skill-name と入力してスキルが認識されるか確認。
または skill-list コマンドで一覧に表示されるか確認。
```

### Phase 4: ブランチ作成

#### Step 4.1: サブモジュールでのブランチ作成

```bash
# nablarch-research-output ディレクトリに移動
cd nablarch-research-output

# 最新の main を取得
git fetch origin
git checkout main
git pull origin main

# 新規ブランチ作成
git checkout -b {branch_name}
# 例: git checkout -b feature/add-skills-20260204
```

#### Step 4.2: ブランチ命名規則

```
【命名規則】

feature/add-{skill_name}           # 単一スキル
feature/add-skills-{YYYYMMDD}      # 複数スキル（日付付き）
feature/add-skills-{cmd_id}        # コマンドID付き

例:
- feature/add-skill-deployment
- feature/add-skills-20260204
- feature/add-skills-cmd046
```

### Phase 5: PR作成

#### Step 5.1: 変更のコミット

```bash
# ステージング
git add skills/{skill_name}/

# 複数スキルの場合
git add skills/

# コミット
git commit -m "feat: add {skill_name} skill

- Add SKILL.md for {skill_name}
- {機能の簡潔な説明}

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

#### Step 5.2: プッシュとPR作成

```bash
# プッシュ
git push -u origin {branch_name}

# PR作成
gh pr create --title "feat: add {skill_name} skill" --body "$(cat <<'EOF'
## Summary

新規スキルを追加:

| スキル名 | 用途 |
|----------|------|
| {skill_name} | {description} |

## Changes

- `skills/{skill_name}/SKILL.md` - 新規スキル定義

## Checklist

- [x] SKILL.md のYAMLフロントマター検証済み
- [x] .claude/skills/ への同期コピー完了
- [x] スキルの動作確認済み

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Phase 6: PRマージ

#### Step 6.1: PRマージ実行

```bash
# PRマージ（squash merge）
gh pr merge --squash --delete-branch

# または通常マージ
gh pr merge --merge --delete-branch
```

#### Step 6.2: mainブランチの更新

```bash
# mainに戻る
git checkout main
git pull origin main
```

### Phase 7: 配置結果の確認

#### Step 7.1: 最終確認チェックリスト

```
【確認項目】

□ nablarch-research-output/skills/{skill_name}/SKILL.md が存在する
□ .claude/skills/{skill_name}/SKILL.md が存在する
□ PRがマージ済み（gh pr view で確認）
□ スキルがClaude Codeで認識される

【確認コマンド】

# ファイル存在確認
ls -la nablarch-research-output/skills/{skill_name}/
ls -la .claude/skills/{skill_name}/

# PR状態確認
gh pr list --state merged --limit 5

# Git状態確認
git status
git log --oneline -5
```

#### Step 7.2: 親リポジトリのサブモジュール更新

```bash
# multi-agent-shogun に戻る
cd /home/kuma/multi-agent-shogun

# サブモジュール更新をステージング
git add nablarch-research-output

# コミット（必要に応じて）
git commit -m "chore: update nablarch-research-output submodule (add {skill_name} skill)"
```

## Output Format

```
# スキルデプロイ結果レポート

## デプロイ対象

| # | スキル名 | ソースパス | 検証結果 |
|---|----------|------------|----------|
| 1 | {skill_name} | {source_path} | ✅ OK |

## 配置結果

### nablarch-research-output/skills/
| スキル名 | パス | 状態 |
|----------|------|------|
| {skill_name} | nablarch-research-output/skills/{skill_name}/SKILL.md | ✅ 配置完了 |

### .claude/skills/
| スキル名 | パス | 状態 |
|----------|------|------|
| {skill_name} | .claude/skills/{skill_name}/SKILL.md | ✅ 配置完了 |

## PR情報

| 項目 | 値 |
|------|-----|
| ブランチ | feature/add-{skill_name} |
| PR URL | https://github.com/{owner}/{repo}/pull/{number} |
| PR状態 | Merged ✅ |

## 次のアクション

- [ ] 親リポジトリでサブモジュール更新をコミット
- [ ] スキルの動作確認
```

## Examples

### Example 1: 単一スキルのデプロイ

```yaml
# 入力
skills:
  - name: "shogun-skill-deployment"
    source_path: "skills/shogun-skill-deployment"
```

```
【実行フロー】

Phase 1: 検証
  - skills/shogun-skill-deployment/SKILL.md 存在確認 → OK
  - YAMLフロントマター検証 → OK

Phase 2: nablarch-research-output へコピー
  $ mkdir -p nablarch-research-output/skills/shogun-skill-deployment
  $ cp skills/shogun-skill-deployment/SKILL.md nablarch-research-output/skills/shogun-skill-deployment/

Phase 3: .claude/skills へコピー
  $ mkdir -p .claude/skills/shogun-skill-deployment
  $ cp skills/shogun-skill-deployment/SKILL.md .claude/skills/shogun-skill-deployment/

Phase 4: ブランチ作成
  $ cd nablarch-research-output
  $ git checkout -b feature/add-skill-deployment

Phase 5: PR作成
  $ git add skills/shogun-skill-deployment/
  $ git commit -m "feat: add skill-deployment skill"
  $ git push -u origin feature/add-skill-deployment
  $ gh pr create --title "feat: add skill-deployment skill" ...

Phase 6: マージ
  $ gh pr merge --squash --delete-branch

Phase 7: 確認
  - PR #8 マージ完了
  - 両格納先にファイル存在確認
```

### Example 2: 複数スキルの一括デプロイ

```yaml
# 入力
skills:
  - name: "shogun-spring-mockvc-test-pattern"
    source_path: "skills/shogun-spring-mockvc-test-pattern"
  - name: "shogun-mcp-server-scaffold"
    source_path: "skills/shogun-mcp-server-scaffold"
  - name: "shogun-concurrent-branch-guard"
    source_path: "skills/shogun-concurrent-branch-guard"
options:
  branch_prefix: "feature/add-skills-cmd046"
  auto_merge: true
```

```
【実行フロー】

Phase 1: 全スキル検証
  | スキル名 | 検証結果 |
  |----------|----------|
  | shogun-spring-mockvc-test-pattern | ✅ OK |
  | shogun-mcp-server-scaffold | ✅ OK |
  | shogun-concurrent-branch-guard | ✅ OK |

Phase 2-3: 各スキルをコピー（並列実行可）

Phase 4: ブランチ作成
  $ git checkout -b feature/add-skills-cmd046

Phase 5: PR作成
  $ git add skills/
  $ git commit -m "feat: add 3 skills (mockvc-test, mcp-scaffold, branch-guard)"
  $ gh pr create --title "feat: add 3 new skills" ...

Phase 6: 自動マージ
  $ gh pr merge --squash --delete-branch

【PR本文例】
## Summary

新規スキルを3件追加:

| スキル名 | 用途 |
|----------|------|
| shogun-spring-mockvc-test-pattern | MockMvcテストパターン生成 |
| shogun-mcp-server-scaffold | MCPサーバースキャフォールド生成 |
| shogun-concurrent-branch-guard | 並行ブランチ競合防止 |

## Changes

- `skills/shogun-spring-mockvc-test-pattern/SKILL.md`
- `skills/shogun-mcp-server-scaffold/SKILL.md`
- `skills/shogun-concurrent-branch-guard/SKILL.md`
```

### Example 3: ドライラン（確認のみ）

```yaml
# 入力
skills:
  - name: "shogun-new-skill"
    source_path: "skills/shogun-new-skill"
options:
  dry_run: true
```

```
【ドライラン結果】

以下の操作が実行されます（dry_run: true のため実際には実行されません）:

1. ファイルコピー:
   - skills/shogun-new-skill/SKILL.md
     → nablarch-research-output/skills/shogun-new-skill/SKILL.md
     → .claude/skills/shogun-new-skill/SKILL.md

2. Git操作:
   - ブランチ作成: feature/add-shogun-new-skill
   - コミット: "feat: add shogun-new-skill skill"
   - PR作成: "feat: add shogun-new-skill skill"

実行する場合は dry_run: false を指定してください。
```

## Guidelines

### 必須ルール

1. **SKILL.md のYAMLフロントマターは必須**
   - `name` と `description` フィールドが必須
   - フロントマターがないスキルはデプロイ不可

2. **nablarch-research-output と .claude/skills の両方に配置すること**
   - 正式リポジトリ（PR管理）と即時利用（ローカル）の両方が必要
   - 片方のみの配置は運用ルール違反

3. **PRはサブモジュール内で作成すること**
   - nablarch-research-output 内でブランチ作成・PR作成
   - 親リポジトリ（multi-agent-shogun）のサブモジュール更新は別途

4. **コミットメッセージは Conventional Commits 形式**
   - `feat: add {skill_name} skill`
   - `Co-Authored-By` 行を含める

5. **複数スキルは1つのPRにまとめる**
   - 個別PRは作成しない（レビュー効率化）
   - ブランチ名に日付またはコマンドIDを含める

### アンチパターン

1. **フロントマターなしのSKILL.md**
   - skill-list に表示されない
   - 対策: Phase 1 で検証し、不備があれば中断

2. **親リポジトリで直接コミット**
   - サブモジュールの履歴が壊れる
   - 対策: 必ずサブモジュール内で作業

3. **.claude/skills/ をGitコミット**
   - 個人環境設定がリポジトリに混入
   - 対策: .gitignore を確認

4. **マージ前のブランチ削除**
   - PRがクローズされる
   - 対策: `--delete-branch` はマージ時のみ指定

5. **検証スキップ**
   - 不完全なSKILL.mdがデプロイされる
   - 対策: Phase 1 を必ず実行

6. **サブモジュール更新忘れ**
   - 親リポジトリのサブモジュール参照が古いまま
   - 対策: Phase 7 で親リポジトリ更新を確認

## Troubleshooting

### よくある問題と解決策

| 問題 | 原因 | 解決策 |
|------|------|--------|
| スキルが認識されない | .claude/skills/ にコピーされていない | Phase 3 を再実行 |
| PR作成失敗 | ブランチがプッシュされていない | `git push -u origin {branch}` 実行 |
| マージ競合 | 同じファイルが別PRで変更された | main をマージしてから再プッシュ |
| サブモジュール更新失敗 | detached HEAD 状態 | `git checkout main` してから作業 |
| skill-list に表示されない | フロントマターの形式エラー | YAML構文を確認 |

### ロールバック手順

```bash
# nablarch-research-output のロールバック
cd nablarch-research-output
git checkout main
git reset --hard origin/main

# .claude/skills/ のロールバック
rm -rf .claude/skills/{skill_name}

# PRのクローズ（マージ前の場合）
gh pr close {pr_number}
```
