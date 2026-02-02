---
name: concurrent-branch-guard
description: 同一リポジトリで複数エージェントが並行作業する際のファイル競合を防止するスキル。RACE-001パターン（同一working directoryで複数足軽が異なるブランチを操作→ファイル破壊）を根本的に解決する。git worktree活用、.lockファイル方式、ファイル排他制御メカニズムを組み合わせ、マルチエージェント環境でのGit操作安全性を保証する。「並行ブランチ作業の競合を防ぎたい」「git worktreeをセットアップして」「ファイルロック機構を導入して」といった要望に対応する。
---

# Concurrent Branch Guard

## Overview

マルチエージェント環境（multi-agent-shogun等）において、同一Gitリポジトリに対して複数のエージェント（足軽）が並行して異なるブランチで作業する際に発生するファイル競合を防止するスキル。

### RACE-001パターンとは

multi-agent-shogunシステムで実際に発生した致命的な競合パターン：

```
【RACE-001: 同一working directory上の並行ブランチ操作によるファイル破壊】

状況:
  足軽A: nablarch-mcp-server/ で feature/phase1-prompts ブランチ作業中
  足軽B: nablarch-mcp-server/ で feature/phase1-resources ブランチ作業中
  → 同一 working directory を共有

発生する問題:
  1. 足軽Aが McpServerConfig.java を編集・コミット
  2. 足軽Bが git checkout feature/phase1-resources を実行
  3. → 足軽Aの未コミット変更が消失、またはコンフリクト発生
  4. 足軽Aが書いたファイルが足軽Bのcheckoutで上書きされる
  5. 足軽Bが書いたファイルが足軽Aのcheckoutで消える

結果:
  - ソースコードの消失・破壊
  - ビルド不能
  - テスト失敗
  - デバッグ困難な「幽霊バグ」の発生
```

**本スキルの対策アプローチ:**

| 対策レベル | 手法 | 適用場面 |
|---|---|---|
| L1: ファイル分離 | 1ファイル=1エージェント | 同一ブランチ内の並列作業 |
| L2: ブランチ分離 | git worktree | 異なるブランチでの並列作業 |
| L3: ロック制御 | .lockファイル | リポジトリ全体の排他制御 |
| L4: タスク設計 | 競合回避のタスク分割 | 家老によるタスク設計時 |

**実績と背景:**
- multi-agent-shogunシステムで最大8足軽の同時並行作業を運用
- RACE-001が複数回発生し、ファイル消失・ブランチ混在を経験
- 現行の対策: instructions/ashigaru.md および instructions/karo.md にRACE-001ルールを明文化
- 本スキルはこれらの運用知見を体系化し、自動化可能な防御メカニズムとして設計

**用途:**
- マルチエージェント開発環境のセットアップ
- 既存プロジェクトへの並行作業ガードの導入
- git worktreeを活用した安全な並行ブランチ作業の確立
- 家老（タスク管理者）のタスク分割設計支援
- CI/CDパイプラインでの並行ビルド競合防止

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「複数のエージェントが同じリポジトリで並行作業する」
- 「git worktreeのセットアップをして」
- 「並行ブランチ作業の競合を防ぎたい」
- 「ファイルロック機構を導入して」
- 「足軽のタスク分割でファイル競合を防ぎたい」
- 「RACE-001対策を導入して」
- 同一リポジトリに対して2名以上のエージェントが同時に作業する状況
- 異なるブランチで並行開発し、各ブランチの成果物を独立に保ちたい場合
- git checkout による意図しないファイル変更が発生している場合
- CI/CDで複数ブランチのビルドが同一マシン上で実行される場合

**トリガーキーワード**: 並行作業, ファイル競合, RACE-001, git worktree, ブランチガード, 排他制御, ファイルロック, 並列ブランチ

## Instructions

### Phase 1: 競合リスク分析

現在の作業環境を調査し、RACE-001発生リスクを評価する。

#### Step 1.1: 環境調査

```
【環境調査チェックリスト】

1. リポジトリ構成の確認
   Bash: ls -la {project_root}/.git
   - bare repository か working directory か
   - submodule の有無

2. 並行作業者の特定
   Bash: tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} #{pane_current_command}'
   - 同一リポジトリに対して作業中のエージェント数
   - 各エージェントの現在のブランチ

3. ブランチ状況の確認
   Bash: git branch -a
   Bash: git status --porcelain
   - アクティブなfeatureブランチの一覧
   - 未コミットの変更の有無

4. ファイル競合ポテンシャルの評価
   - 共通ファイル（build.gradle.kts, McpServerConfig.java等）の編集計画
   - 各エージェントの担当ファイル一覧
   - 重複する編集対象の特定
```

#### Step 1.2: リスク評価

```
【RACE-001リスクレベル判定】

Level 0: リスクなし
  - 1エージェントのみが作業中
  - または各エージェントが完全に別のリポジトリで作業

Level 1: 低リスク（L1: ファイル分離で対応可能）
  - 複数エージェントが同一リポジトリ・同一ブランチで作業
  - ただし編集対象ファイルが完全に分離されている
  - 例: 足軽Aは prompts/ 配下、足軽Bは tools/ 配下のみ

Level 2: 中リスク（L2: worktree必須）
  - 複数エージェントが同一リポジトリ・異なるブランチで作業
  - git checkout が発生しうる

Level 3: 高リスク（L2 + L3: worktree + ロック必須）
  - 複数エージェントが同一リポジトリ・異なるブランチで作業
  - かつ共通ファイル（build.gradle.kts等）を双方が編集する可能性

Level 4: 極高リスク（設計見直し必須）
  - 3名以上のエージェントが同一ブランチの同一ファイルを編集
  - → タスク分割の再設計が必要（Phase 4を先に実行）
```

### Phase 2: git worktree によるブランチ分離（L2対策）

git worktreeを使って各エージェントに独立したworking directoryを提供する。

#### Step 2.1: worktree の基本概念

```
【git worktree とは】

1つのGitリポジトリに対して、複数のworking directory（worktree）を作成する機能。
各worktreeは独立したブランチをチェックアウトでき、互いに干渉しない。

通常の構成（RACE-001が発生）:
  nablarch-mcp-server/         ← 1つのworking directory
    .git/                      ← 1つのGitデータベース
    src/                       ← 足軽A, B が同時に操作 → 競合！

worktree構成（RACE-001を回避）:
  nablarch-mcp-server/         ← メインworktree（mainブランチ）
    .git/                      ← Gitデータベース（共有）
    src/
  nablarch-mcp-server-wt-prompts/   ← worktree 1（feature/phase1-prompts）
    src/                             ← 足軽A専用
  nablarch-mcp-server-wt-resources/ ← worktree 2（feature/phase1-resources）
    src/                             ← 足軽B専用
```

#### Step 2.2: worktree セットアップ手順

```
【worktree セットアップ】

1. メインリポジトリの状態確認
   cd {project_root}
   git status  # clean state であることを確認
   git fetch origin

2. worktree ディレクトリの命名規則
   {project_root}-wt-{agent_id}/
   例: nablarch-mcp-server-wt-ashigaru3/

   または:
   {project_root}-wt-{branch_short_name}/
   例: nablarch-mcp-server-wt-prompts/

3. worktree の作成
   # 既存ブランチの場合
   git worktree add ../{project}-wt-{id} {branch_name}

   # 新規ブランチの場合
   git worktree add -b {new_branch} ../{project}-wt-{id} origin/main

4. worktree の確認
   git worktree list
   # /home/user/nablarch-mcp-server              abc1234 [main]
   # /home/user/nablarch-mcp-server-wt-prompts   def5678 [feature/phase1-prompts]
   # /home/user/nablarch-mcp-server-wt-resources  ghi9012 [feature/phase1-resources]

5. 各エージェントへの作業ディレクトリ通知
   # 家老から足軽への指示に worktree パスを明記
   target_path: "/home/user/nablarch-mcp-server-wt-ashigaru3/"
```

#### Step 2.3: worktree での作業フロー

```
【worktree 作業フロー】

1. 足軽の作業開始
   cd {worktree_path}
   git status  # 正しいブランチであることを確認
   # → 通常通りの開発作業（edit, add, commit）

2. worktree 間の変更の反映
   # worktree AからworktreeB の変更を取り込みたい場合
   cd {worktree_B}
   git fetch  # ローカルのGitデータベースは共有されているため不要な場合が多い
   git merge origin/{branch_A}  # リモート経由で取り込み

3. コンフリクト解決
   # 各worktreeで独立してコンフリクト解決
   # → 他のworktreeに影響しない

4. worktree の削除（作業完了後）
   git worktree remove {worktree_path}
   # または
   rm -rf {worktree_path}
   git worktree prune
```

#### Step 2.4: worktree の注意事項

```
【worktree 制約と注意点】

1. 同一ブランチの重複チェックアウト不可
   - 1つのブランチは1つのworktreeでのみチェックアウト可能
   - 別のworktreeで同じブランチをcheckoutしようとするとエラー
   - → エージェントごとに異なるブランチを割り当てる

2. ディスク使用量
   - 各worktreeはsrc/ 等の実ファイルのコピーを持つ
   - .git/ はシンボリックリンクで共有（容量は最小限）
   - 大規模プロジェクトでは worktree 数を制限する

3. reflog の共有
   - 全worktreeで1つのreflogを共有
   - コミット履歴は全worktreeから参照可能

4. submodule との併用
   - worktree内でsubmodule init/updateが必要
   - メインリポジトリのsubmodule設定は共有されない

5. IDE との連携
   - 各worktreeを別プロジェクトとしてIDEで開く
   - .idea/ や .vscode/ は各worktreeに独立して存在
```

### Phase 3: .lock ファイルによる排他制御（L3対策）

共通ファイルの同時編集を防ぐためのロック機構を実装する。

#### Step 3.1: ロック機構の設計

```
【.lock ファイル方式】

1. ロックファイルの配置
   {project_root}/.locks/
   ├── build.gradle.kts.lock
   ├── McpServerConfig.java.lock
   └── README.md.lock

2. ロックファイルの形式（YAML）
   # .locks/McpServerConfig.java.lock
   locked_by: ashigaru3
   locked_at: "2026-02-02T15:30:00"
   branch: "feature/phase1-prompts"
   reason: "nablarchPrompts() Bean定義の更新"
   expires_at: "2026-02-02T16:30:00"  # 最大1時間

3. ロック取得手順
   # ロックファイルが存在しないことを確認
   if [ ! -f .locks/{filename}.lock ]; then
     # ロック取得
     cat > .locks/{filename}.lock << 'EOF'
     locked_by: {agent_id}
     locked_at: "{timestamp}"
     branch: "{branch}"
     reason: "{reason}"
     expires_at: "{timestamp + 1h}"
     EOF
     echo "ロック取得成功"
   else
     echo "ロック取得失敗: $(cat .locks/{filename}.lock)"
     # → 待機またはタスク変更
   fi

4. ロック解放手順
   rm .locks/{filename}.lock

5. 期限切れロックのクリーンアップ
   # expires_at を過ぎたロックを自動削除
   for lock in .locks/*.lock; do
     expires=$(grep expires_at "$lock" | cut -d'"' -f2)
     if [ "$(date -u +%Y-%m-%dT%H:%M:%S)" > "$expires" ]; then
       rm "$lock"
       echo "期限切れロック削除: $lock"
     fi
   done
```

#### Step 3.2: ロック運用ルール

```
【ロック運用ルール】

1. ロック対象ファイル
   - build.gradle.kts（ビルド設定）
   - McpServerConfig.java（MCP統合設定）
   - application.properties（アプリケーション設定）
   - その他、複数エージェントが編集しうるファイル

2. ロック取得タイミング
   - ファイル編集開始前に取得
   - Write / Edit ツール使用前に確認

3. ロック解放タイミング
   - ファイル編集完了後に即座に解放
   - コミット完了後が望ましい
   - 異常終了時は expires_at で自動解放

4. ロック競合時の対応
   - 待機: 5分間隔でリトライ（最大30分）
   - タスク切替: ロック不要な別タスクに切り替え
   - エスカレーション: 30分以上待機の場合、家老に報告

5. .locks/ ディレクトリの管理
   - .gitignore に .locks/ を追加（ロックファイルはGit管理外）
   - ディレクトリ自体は Git 管理（.locks/.gitkeep）
```

### Phase 4: タスク設計による競合回避（L4対策）

家老がタスクを分割・割り当てする際に、RACE-001を構造的に回避する設計手法。

#### Step 4.1: タスク分割の原則

```
【RACE-001回避のタスク分割原則】

原則1: 1ファイル = 1エージェント
  - 同一ファイルを複数エージェントが編集する設計にしない
  - 統合ファイル（McpServerConfig.java等）は1名に集約

原則2: ディレクトリ境界で分割
  - prompts/ → 足軽A
  - tools/ → 足軽B
  - resources/ → 足軽C
  - パッケージ単位で排他制御

原則3: 依存関係の順序化
  - 共通ファイルに依存するタスクは直列実行
  - 独立したタスクは並列実行
  - 例: Prompt実装（並列可） → McpServerConfig統合（直列）

原則4: ブランチ戦略との整合
  - 各足軽に専用ブランチを割り当て
  - 共通ブランチでの並行作業は禁止
  - PR単位でmainにマージ（マージは家老が順次実施）
```

#### Step 4.2: タスク割当チェックリスト

```
【家老用: タスク割当前チェックリスト】

□ 各足軽の編集対象ファイルを列挙したか？
□ 編集対象ファイルの重複がないか確認したか？
□ 重複がある場合、以下のいずれかで対応したか？
  □ タスクを直列化（先行タスク完了後に後続を開始）
  □ ファイルを分割（統合は後で1名が実施）
  □ git worktreeで分離（L2対策）
□ 各足軽のブランチが異なることを確認したか？
□ 共通ファイル（build.gradle.kts等）の編集者が1名に限定されているか？
□ target_path にworktreeパスを指定したか？（L2適用時）
□ ロック対象ファイルを明示したか？（L3適用時）

【判定マトリクス】

| 作業内容                 | 推奨対策      |
|--------------------------|---------------|
| 作業内容が独立している   | 分割して並列投入 |
| 前工程の結果が次工程に必要 | 順次投入（車懸りの陣） |
| 同一ファイルへの書き込みが必要 | RACE-001に従い1名で |
| 異なるブランチで並行作業   | git worktreeで分離 |
| 共通設定ファイルの編集あり | ロック制御 + 1名限定 |
```

#### Step 4.3: multi-agent-shogun向けYAML指示テンプレート

```
【タスクYAMLテンプレート — worktree対応版】

task:
  task_id: subtask_xxx
  parent_cmd: cmd_xxx
  project: {project_name}
  description: |
    【タスク説明】
    ...

    ## RACE-001対策
    - 作業ディレクトリ: {worktree_path}（専用worktree）
    - 編集対象ファイル: {file_list}
    - ロック対象: なし / {locked_files}
    - 他足軽との共通ファイル: なし / {shared_files}（→直列実行）

  target_path: "{worktree_path}"  # worktreeパスを指定
  branch: "{branch_name}"
  status: pending
  race_guard:
    level: L2                     # L1/L2/L3/L4
    worktree: "{worktree_path}"
    exclusive_files:              # この足軽のみが編集可能なファイル
      - "src/main/java/.../prompts/*.java"
    locked_files: []              # ロック取得が必要なファイル
    depends_on: []                # 先行タスク（直列実行が必要な場合）
```

### Phase 5: 自動化スクリプトの導入

競合防止メカニズムを自動化するスクリプトを提供する。

#### Step 5.1: worktree管理スクリプト

```bash
#!/bin/bash
# worktree-manager.sh — git worktree のライフサイクル管理

REPO_ROOT=$(git rev-parse --show-toplevel)
PROJECT_NAME=$(basename "$REPO_ROOT")

case "$1" in
  create)
    # worktree 作成
    AGENT_ID=$2
    BRANCH=$3
    WT_PATH="${REPO_ROOT}/../${PROJECT_NAME}-wt-${AGENT_ID}"

    if [ -d "$WT_PATH" ]; then
      echo "エラー: worktree既に存在: $WT_PATH"
      exit 1
    fi

    if git show-ref --verify --quiet "refs/heads/$BRANCH"; then
      git worktree add "$WT_PATH" "$BRANCH"
    else
      git worktree add -b "$BRANCH" "$WT_PATH" origin/main
    fi
    echo "worktree作成完了: $WT_PATH ($BRANCH)"
    ;;

  remove)
    # worktree 削除
    AGENT_ID=$2
    WT_PATH="${REPO_ROOT}/../${PROJECT_NAME}-wt-${AGENT_ID}"

    if [ ! -d "$WT_PATH" ]; then
      echo "エラー: worktree不存在: $WT_PATH"
      exit 1
    fi

    git worktree remove "$WT_PATH"
    echo "worktree削除完了: $WT_PATH"
    ;;

  list)
    git worktree list
    ;;

  cleanup)
    # 期限切れworktreeと孤立worktreeの削除
    git worktree prune
    echo "孤立worktree削除完了"
    ;;

  *)
    echo "使用法: $0 {create|remove|list|cleanup} [agent_id] [branch]"
    exit 1
    ;;
esac
```

#### Step 5.2: ファイルロックスクリプト

```bash
#!/bin/bash
# file-lock.sh — ファイルロックの取得・解放

LOCK_DIR=".locks"
mkdir -p "$LOCK_DIR"

case "$1" in
  acquire)
    FILE=$2
    AGENT_ID=$3
    LOCK_FILE="${LOCK_DIR}/$(basename "$FILE").lock"

    if [ -f "$LOCK_FILE" ]; then
      # 期限切れチェック
      EXPIRES=$(grep expires_at "$LOCK_FILE" | cut -d'"' -f2)
      NOW=$(date -u +%Y-%m-%dT%H:%M:%S)
      if [[ "$NOW" > "$EXPIRES" ]]; then
        echo "期限切れロックを解放: $LOCK_FILE"
        rm "$LOCK_FILE"
      else
        echo "ロック取得失敗: $(cat "$LOCK_FILE")"
        exit 1
      fi
    fi

    NOW=$(date -u +%Y-%m-%dT%H:%M:%S)
    EXPIRES=$(date -u -d "+1 hour" +%Y-%m-%dT%H:%M:%S)
    cat > "$LOCK_FILE" << EOF
locked_by: $AGENT_ID
locked_at: "$NOW"
reason: "editing $FILE"
expires_at: "$EXPIRES"
EOF
    echo "ロック取得成功: $FILE (by $AGENT_ID)"
    ;;

  release)
    FILE=$2
    LOCK_FILE="${LOCK_DIR}/$(basename "$FILE").lock"

    if [ -f "$LOCK_FILE" ]; then
      rm "$LOCK_FILE"
      echo "ロック解放: $FILE"
    else
      echo "ロック不存在: $FILE"
    fi
    ;;

  status)
    echo "=== アクティブなロック ==="
    if [ -z "$(ls -A "$LOCK_DIR" 2>/dev/null)" ]; then
      echo "ロックなし"
    else
      for lock in "$LOCK_DIR"/*.lock; do
        echo "--- $(basename "$lock" .lock) ---"
        cat "$lock"
        echo ""
      done
    fi
    ;;

  cleanup)
    NOW=$(date -u +%Y-%m-%dT%H:%M:%S)
    for lock in "$LOCK_DIR"/*.lock; do
      [ -f "$lock" ] || continue
      EXPIRES=$(grep expires_at "$lock" | cut -d'"' -f2)
      if [[ "$NOW" > "$EXPIRES" ]]; then
        echo "期限切れロック削除: $lock"
        rm "$lock"
      fi
    done
    ;;

  *)
    echo "使用法: $0 {acquire|release|status|cleanup} [file] [agent_id]"
    exit 1
    ;;
esac
```

#### Step 5.3: pre-commitフックによる自動チェック

```bash
#!/bin/bash
# .git/hooks/pre-commit — コミット前のロックチェック

LOCK_DIR=".locks"

# ステージングされたファイルを取得
STAGED_FILES=$(git diff --cached --name-only)

for file in $STAGED_FILES; do
  LOCK_FILE="${LOCK_DIR}/$(basename "$file").lock"
  if [ -f "$LOCK_FILE" ]; then
    LOCKED_BY=$(grep locked_by "$LOCK_FILE" | awk '{print $2}')
    CURRENT_AGENT=${AGENT_ID:-"unknown"}

    if [ "$LOCKED_BY" != "$CURRENT_AGENT" ]; then
      echo "エラー: $file は $LOCKED_BY がロック中です"
      echo "ロック解放を待つか、家老に報告してください"
      exit 1
    fi
  fi
done

exit 0
```

## Input Format

```yaml
# 競合防止セットアップリクエスト
project_root: "/path/to/repository"       # リポジトリルートパス
agents:                                    # 並行作業するエージェント一覧
  - id: "ashigaru3"
    branch: "feature/phase1-prompts"
    files:                                 # 編集対象ファイル
      - "src/main/java/.../prompts/*.java"
      - "src/test/java/.../prompts/*.java"
  - id: "ashigaru5"
    branch: "feature/phase1-resources"
    files:
      - "src/main/java/.../resources/*.java"
      - "src/test/java/.../resources/*.java"
shared_files:                              # 共通編集対象ファイル
  - "build.gradle.kts"
  - "src/main/java/.../config/McpServerConfig.java"
guard_level: "L2"                          # L1/L2/L3/L4
```

## Output Format

```
# セットアップ結果

## worktree構成（L2適用時）
{project_root}/                            # メインworktree (main)
{project_root}-wt-ashigaru3/               # worktree 1 (feature/phase1-prompts)
{project_root}-wt-ashigaru5/               # worktree 2 (feature/phase1-resources)

## ロック設定（L3適用時）
{project_root}/.locks/                     # ロックディレクトリ
{project_root}/.locks/.gitkeep             # Git管理用
{project_root}/.gitignore                  # .locks/*.lock を追加

## スクリプト（L2/L3適用時）
{project_root}/scripts/worktree-manager.sh
{project_root}/scripts/file-lock.sh

## タスクYAML（L4適用時）
各足軽のタスクYAMLに race_guard セクションを追加

## 確認レポート
- 各worktreeのブランチ確認結果
- ロック機構の動作テスト結果
- 競合ポテンシャルの評価結果
```

## Examples

### Example 1: nablarch-mcp-server Phase 1 での並行作業

```
【状況】
- 足軽3名が同一リポジトリ nablarch-mcp-server で作業
- 足軽A: feature/phase1-prompts（Prompt 6種実装）
- 足軽B: feature/phase1-resources（Resource 12種実装）
- 足軽C: feature/phase1-tools（Tool 2種実装）
- 共通ファイル: McpServerConfig.java, build.gradle.kts

【対策: L2 + L3】

1. worktree セットアップ
   git worktree add ../nablarch-mcp-server-wt-ashigaru3 feature/phase1-prompts
   git worktree add ../nablarch-mcp-server-wt-ashigaru5 feature/phase1-resources
   git worktree add ../nablarch-mcp-server-wt-ashigaru6 feature/phase1-tools

2. タスク割当
   足軽A: target_path: ../nablarch-mcp-server-wt-ashigaru3/
          exclusive_files: [prompts/*.java]
   足軽B: target_path: ../nablarch-mcp-server-wt-ashigaru5/
          exclusive_files: [resources/*.java]
   足軽C: target_path: ../nablarch-mcp-server-wt-ashigaru6/
          exclusive_files: [tools/*.java]

3. McpServerConfig.java の統合
   - 各足軽が自分の担当プリミティブのBean定義のみを実装
   - 統合は最後に1名（足軽A）が全ブランチの変更をマージ
   - または家老が統合タスクとして別途割り当て

【結果】
- ファイル競合ゼロ
- 3名が完全に独立して並行作業可能
- 各ブランチのPRが独立してレビュー可能
```

### Example 2: ドキュメント生成の並列作業

```
【状況】
- 足軽6名がドキュメント生成タスクを並行実行
- 各足軽が異なるSKILL.mdファイルを作成
- 全員が同一ブランチ feature/skills-s017-s019 で作業

【対策: L1（ファイル分離のみ）】

1. タスク分割
   足軽A: skills/shogun-nablarch-knowledge-builder/SKILL.md
   足軽B: skills/shogun-mcp-server-scaffold/SKILL.md
   足軽C: skills/shogun-concurrent-branch-guard/SKILL.md
   → 各足軽が専用ディレクトリで作業。ファイル競合の可能性なし。

2. コミット順序
   - 全員が作業完了後、順番にコミット
   - または各足軽が独自コミットを作成（同一ブランチでも可）

【結果】
- L1（ファイル分離）のみで十分
- worktreeやロック不要
- 最もシンプルで効率的
```

### Example 3: CI/CDパイプラインでの並行ビルド

```
【状況】
- GitHub Actions で複数ブランチの同時ビルド
- セルフホストランナー1台で並行実行
- ビルドキャッシュの競合が発生

【対策: L2（worktree）】

1. GitHub Actions ワークフロー
   jobs:
     build:
       steps:
         - uses: actions/checkout@v4
         - name: Setup worktree
           run: |
             git worktree add ../build-${{ github.run_id }} ${{ github.ref }}
             cd ../build-${{ github.run_id }}
         - name: Build
           working-directory: ../build-${{ github.run_id }}
           run: ./gradlew clean build
         - name: Cleanup
           run: git worktree remove ../build-${{ github.run_id }}

【結果】
- 各ビルドが独立したworking directoryで実行
- キャッシュ競合なし
- ビルド結果が互いに干渉しない
```

## Guidelines

### 必須ルール

1. **RACE-001を構造的に回避すること**
   - 「気をつける」だけでは不十分。仕組みで防止する
   - L1（ファイル分離）が最低限。リスクに応じてL2〜L4を追加
   - 「1ファイル = 1エージェント」の原則を厳守

2. **worktreeのライフサイクルを管理すること**
   - 作業開始時に作成、作業完了・PR作成後に削除
   - 孤立worktreeを定期的にprune（`git worktree prune`）
   - worktreeの一覧を `git worktree list` で確認可能な状態を維持

3. **ロックの取得・解放を確実に行うこと**
   - ファイル編集前にロック取得、編集完了後に即座に解放
   - 異常終了時のためexpires_at（最大1時間）を設定
   - ロック取得失敗時は待機またはタスク切替

4. **タスクYAMLに競合防止情報を明記すること**
   - race_guard セクションでレベル・worktree・排他ファイルを指定
   - 家老はタスク割当前にチェックリストを実施
   - 足軽は指定された target_path でのみ作業

5. **共通ファイルの編集者を1名に限定すること**
   - build.gradle.kts, McpServerConfig.java 等の統合ファイル
   - 複数人が編集する場合はロック制御（L3）を適用
   - 統合タスクは最後に1名に割り当て

6. **worktreeパスの命名規則を統一すること**
   - `{project}-wt-{agent_id}` または `{project}-wt-{branch_short_name}`
   - メインリポジトリと同じ親ディレクトリに配置
   - 一貫した命名でworktree管理を容易にする

### 効率化のコツ

1. **リスクレベルに応じた対策選択**
   - 全ての状況にL3（ロック制御）を適用する必要はない
   - L1（ファイル分離）で十分な場合が最も多い
   - オーバーエンジニアリングを避ける

2. **タスク設計時の予防**
   - 実装後に競合を解決するより、タスク設計時に回避する方が効率的
   - 家老のタスク分割チェックリストを活用

3. **worktreeの事前セットアップ**
   - 足軽に指示を出す前にworktreeを準備しておく
   - 足軽がworktreeセットアップに時間を使わなくて済む

4. **統合タスクの分離**
   - 共通ファイルの統合を独立したサブタスクとして設計
   - 個別実装（並列） → 統合（直列） の2段階構成

5. **.gitignoreの活用**
   - .locks/*.lock はGit管理外にする
   - worktreeの設定ファイル等もignore対象

### アンチパターン（避けるべきこと）

1. **「気をつけているから大丈夫」という油断**
   - 人間もAIエージェントも、並行作業時にファイル競合を完全に避けることはできない
   - 対策: 仕組み（worktree、ロック）で物理的に防止する

2. **同一working directoryでの複数ブランチ操作**
   - `git checkout` で頻繁にブランチを切り替える運用
   - 対策: 各ブランチにworktreeを割り当て、checkoutを不要にする

3. **ロックの取得忘れ・解放忘れ**
   - 編集前にロックを取得せず、他エージェントの変更を上書き
   - 編集後にロックを解放せず、他エージェントがブロック
   - 対策: スクリプト化して手順を自動化。expires_atでデッドロック回避

4. **worktreeの放置**
   - 作業完了後もworktreeを削除せず、ディスクを消費し続ける
   - 対策: タスク完了報告時にworktree削除を手順に含める

5. **共通ファイルの無計画な並行編集**
   - 2名以上がbuild.gradle.ktsを同時に編集してコンフリクト
   - 対策: 共通ファイル編集は1名に限定。統合タスクとして分離

6. **タスク分割なしの投入**
   - ファイル競合の可能性を検討せずに足軽を投入
   - 対策: 家老のタスク割当チェックリストを必ず実施

7. **worktree内でのgit checkout**
   - worktree内でブランチを切り替える（worktreeの意味がなくなる）
   - 対策: 各worktreeは固定ブランチ。ブランチ変更が必要な場合は新worktreeを作成

8. **過剰なロック粒度**
   - ファイル単位ではなくリポジトリ全体をロックする
   - 対策: ロックはファイル単位で最小限に。リポジトリロックは最終手段

9. **エラー時のロック未解放**
   - エージェントが異常終了し、ロックが残り続ける
   - 対策: expires_at を設定。cleanup スクリプトを定期実行

10. **worktreeの命名不統一**
    - `wt-1`, `worktree_prompts`, `tmp-branch` 等バラバラな命名
    - 対策: `{project}-wt-{agent_id}` の命名規則を厳守
