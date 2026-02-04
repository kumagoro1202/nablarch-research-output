---
name: skill-extension-pattern
description: 既存スキルに新機能を追加する際のパターン化されたワークフローを提供するスキル。YAML Front Matter拡張、When to Use追加、新フェーズ追加、Examples追加などの定型作業をテンプレート化し、スキル拡張の品質と一貫性を保証する。「スキルに新しいフェーズを追加したい」「スキルの拡張パターンを教えて」「SKILL.mdの構造を拡張する方法を教えて」「スキル拡張のベストプラクティスを教えて」「既存スキルに機能を統合したい」といった要望に対応する。
---

# Skill Extension Pattern

## Overview

既存のshogunスキル（SKILL.md形式）に新機能を追加・統合する際の標準化されたワークフローを提供する。

### 背景と目的

shogunシステムでは、スキルを継続的に拡張・改善することが求められる。しかし、スキル拡張の方法が標準化されていないと、以下の問題が発生する：

- スキル間でフォーマットの不整合が発生
- 拡張時に必要なセクションの更新漏れ
- 後方互換性の破壊
- Guidelinesの不備による品質低下

本スキルは、スキル拡張作業を8つのフェーズに分解し、各フェーズで更新すべきセクションと注意点を明確化する。

### スキル拡張の種類

| 拡張種別 | 説明 | 対象セクション |
|----------|------|----------------|
| 機能追加 | 既存スキルに新機能を追加 | description, When to Use, Instructions, Examples, Guidelines |
| フェーズ追加 | 新しい処理フェーズを追加 | Instructions, Examples |
| パラメータ追加 | Input/Output に新パラメータ追加 | Input Format, Output Format, Instructions |
| トリガー追加 | 新しい呼び出しパターンを追加 | description, When to Use |
| ガイドライン追加 | 必須ルール・アンチパターンを追加 | Guidelines |

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「スキルに新しいフェーズを追加したい」
- 「スキルの拡張パターンを教えて」
- 「SKILL.mdの構造を拡張する方法を教えて」
- 「スキル拡張のベストプラクティスを教えて」
- 「既存スキルに機能を統合したい」
- 「スキルに新しいトリガーキーワードを追加したい」
- 「スキルのGuidelinesを拡充したい」
- 既存のshogunスキルに新機能を追加する必要がある場合
- 複数の関連機能を1つのスキルに統合する場合
- スキルの対応範囲を拡大する場合

**トリガーキーワード**: スキル拡張, 機能追加, フェーズ追加, SKILL.md編集, スキル統合, 機能統合

## Input Format

```yaml
# スキル拡張リクエスト
target_skill:
  path: "skills/shogun-xxx/SKILL.md"     # 拡張対象スキルのパス
  name: "xxx"                             # スキル名

extension:
  type: "feature_addition"                # 拡張種別
  # feature_addition: 機能追加
  # phase_addition: フェーズ追加
  # parameter_addition: パラメータ追加
  # trigger_addition: トリガー追加
  # guideline_addition: ガイドライン追加

  features:                               # 追加する機能（複数可）
    - name: "feature-name-1"
      description: "機能の説明"
      triggers:                           # この機能を呼び出すキーワード
        - "○○したい"
        - "○○を教えて"
      use_cases:                          # ユースケース
        - "具体的な使用場面1"
        - "具体的な使用場面2"

  new_phases:                             # 追加するフェーズ（フェーズ追加時）
    - phase_number: 6                     # 既存フェーズの後に追加
      title: "新フェーズのタイトル"
      steps:
        - step_number: 6.1
          title: "ステップタイトル"
          content: "ステップの内容"

  new_parameters:                         # 追加するパラメータ（パラメータ追加時）
    input:
      - name: "new_param"
        type: "string"
        description: "パラメータの説明"
        required: false
    output:
      - name: "new_output"
        description: "出力の説明"

  new_guidelines:                         # 追加するガイドライン
    rules:
      - "新しい必須ルール"
    anti_patterns:
      - "新しいアンチパターン"

policy:
  backward_compatible: true               # 後方互換性を維持するか
  preserve_existing: true                 # 既存機能を変更しないか
```

## Instructions

### Phase 1: 既存スキルの読み込みと構造分析

拡張対象のスキルを読み込み、現在の構造を把握する。

#### Step 1.1: スキルファイルの読み込み

```
【スキル読み込み手順】

1. 対象スキルのSKILL.mdを読み込む
   Read: {target_skill.path}

2. 構造を確認
   - YAML Front Matter（name, description）の内容
   - When to Use セクションの現在のトリガー
   - Instructions の現在のフェーズ数
   - Input/Output Format の現在のパラメータ
   - Examples の現在の例数
   - Guidelines の現在のルール・アンチパターン
```

#### Step 1.2: 構造分析レポートの作成

```
【構造分析レポート】

スキル名: {name}
現在のフェーズ数: N
現在のExample数: M
現在のトリガー数: K

セクション一覧:
□ YAML Front Matter
  - name: 存在/不存在
  - description: 文字数 XXX
□ Overview: 存在/不存在
□ When to Use: トリガー数 K
□ Input Format: パラメータ数 X
□ Instructions: フェーズ数 N
□ Output Format: 項目数 Y
□ Examples: 例数 M
□ Guidelines
  - 必須ルール数: A
  - アンチパターン数: B

拡張計画:
- 追加フェーズ: Phase N+1, Phase N+2, ...
- 追加Example: Example M+1, Example M+2, ...
- 追加トリガー: K+1, K+2, ...
```

### Phase 2: YAML Front Matter (description) の拡張

スキルの説明文を更新し、新機能を反映する。

#### Step 2.1: description の拡張パターン

```
【description 拡張パターン】

既存の description:
  "既存の説明文。「既存トリガー1」「既存トリガー2」といった要望に対応する。"

拡張後の description:
  "既存の説明文。「既存トリガー1」「既存トリガー2」といった要望に対応する。
   さらに、{新機能名}機能（{新機能の説明}）を提供する。
   「{新トリガー1}」「{新トリガー2}」といった要望にも対応する。"

【拡張ルール】
1. 既存の説明文は変更しない（後方互換性）
2. 新機能は「さらに、」「また、」で接続
3. 新トリガーは「〜といった要望にも対応する」で追記
4. 1行が長くなりすぎる場合は適切に改行
```

#### Step 2.2: Edit コマンドの実行

```
【Edit コマンド例】

Edit:
  file_path: {target_skill.path}
  old_string: |
    description: 既存の説明文...といった要望に対応する。
    ---
  new_string: |
    description: 既存の説明文...といった要望に対応する。さらに、{新機能名}機能（{新機能の説明}）を提供する。「{新トリガー1}」「{新トリガー2}」といった要望にも対応する。
    ---
```

### Phase 3: When to Use にトリガーキーワード追加

新機能の呼び出しパターンを追加する。

#### Step 3.1: トリガー追加の位置特定

```
【When to Use セクションの構造】

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「既存トリガー1」
- 「既存トリガー2」
- 「新トリガー1」（← 追加）
- 「新トリガー2」（← 追加）
- 既存のユースケース説明
- 新しいユースケース説明（← 追加）

**トリガーキーワード**: 既存KW1, 既存KW2, 新KW1, 新KW2（← 追加）
```

#### Step 3.2: トリガー追加の実行

```
【Edit コマンド例 — トリガーリスト追加】

Edit:
  file_path: {target_skill.path}
  old_string: |
    - 「既存トリガー2」
    - 既存のユースケース説明
  new_string: |
    - 「既存トリガー2」
    - 「新トリガー1」（{新機能名}機能）
    - 「新トリガー2」（{新機能名}機能）
    - 既存のユースケース説明
    - 新しいユースケース説明

【Edit コマンド例 — トリガーキーワード追加】

Edit:
  file_path: {target_skill.path}
  old_string: |
    **トリガーキーワード**: 既存KW1, 既存KW2
  new_string: |
    **トリガーキーワード**: 既存KW1, 既存KW2, 新KW1, 新KW2
```

### Phase 4: Input Format の拡張（新パラメータ追加）

新機能に必要なパラメータを追加する。

#### Step 4.1: パラメータ追加の判断

```
【パラメータ追加の判断基準】

追加が必要な場合:
- 新機能が追加の入力情報を必要とする
- 既存パラメータでは新機能の動作を制御できない
- 新機能固有のオプションがある

追加が不要な場合:
- 既存パラメータで新機能も制御可能
- 新機能がパラメータなしで動作可能
- 新機能の動作がハードコードで問題ない
```

#### Step 4.2: Input Format 更新

```
【Input Format 拡張パターン】

既存:
```yaml
param1: "value1"
param2: "value2"
```

拡張後:
```yaml
param1: "value1"
param2: "value2"
# 新機能用パラメータ（オプション）
new_param1: "value"           # 新パラメータの説明
new_param2: true              # 新パラメータの説明
```

【注意点】
- 新パラメータはオプションにする（後方互換性）
- コメントで用途を明記
- デフォルト値を設定可能にする
```

### Phase 5: 新フェーズを既存フェーズの後に追加

新機能の処理フローをフェーズとして追加する。

#### Step 5.1: フェーズ番号の決定

```
【フェーズ番号決定ルール】

既存フェーズ: Phase 1 〜 Phase N
新フェーズ: Phase N+1, Phase N+2, ...

例:
  既存: Phase 1 〜 Phase 5
  追加: Phase 6（新機能A）, Phase 7（新機能B）
```

#### Step 5.2: フェーズ構造のテンプレート

```
【新フェーズのテンプレート】

### Phase {N+1}: {フェーズタイトル}（{機能名}機能）

{フェーズの概要説明}

#### Step {N+1}.1: {ステップタイトル}

```
【{ステップの内容タイトル}】

{ステップの詳細説明}

1. 手順1
   コマンドまたはコード例

2. 手順2
   コマンドまたはコード例

3. 手順3
   コマンドまたはコード例
```

#### Step {N+1}.2: {ステップタイトル}

...（以下同様）
```

#### Step 5.3: フェーズ追加の実行

```
【Edit コマンド例 — フェーズ追加】

既存の最終フェーズの後、## Input Format の前に追加

Edit:
  file_path: {target_skill.path}
  old_string: |
    （既存の最終フェーズの最後のコードブロック）
    ```

    ## Input Format
  new_string: |
    （既存の最終フェーズの最後のコードブロック）
    ```

    ### Phase {N+1}: {フェーズタイトル}（{機能名}機能）

    {フェーズの内容...}

    ## Input Format
```

### Phase 6: Output Format の拡張

新機能の出力形式を追加する。

#### Step 6.1: Output 項目の追加

```
【Output Format 拡張パターン】

既存:
```
## 出力項目1
{説明}

## 出力項目2
{説明}
```

拡張後:
```
## 出力項目1
{説明}

## 出力項目2
{説明}

## 出力項目3（新機能用）
{新機能の出力説明}
```

【注意点】
- 新出力項目は既存項目の後に追加
- どの機能の出力かをコメントまたは括弧で明記
```

### Phase 7: Examples の追加

新機能の使用例を追加する。

#### Step 7.1: Example 番号の決定

```
【Example 番号決定ルール】

既存Example: Example 1 〜 Example M
新Example: Example M+1, Example M+2, ...

例:
  既存: Example 1 〜 Example 3
  追加: Example 4（新機能Aの例）, Example 5（新機能Bの例）
```

#### Step 7.2: Example テンプレート

```
【新 Example のテンプレート】

### Example {M+1}: {Exampleタイトル}（{機能名}機能）

```
【状況】
- 状況の説明1
- 状況の説明2
- 状況の説明3

【セットアップ/対策】

1. 手順1
   コマンドまたはコード例

2. 手順2
   コマンドまたはコード例

3. 手順3
   コマンドまたはコード例

【結果】
- 結果1
- 結果2
- 結果3
```
```

#### Step 7.3: Example 追加の実行

```
【Edit コマンド例 — Example 追加】

既存の最終Exampleの後、## Guidelines の前に追加

Edit:
  file_path: {target_skill.path}
  old_string: |
    （既存の最終Exampleの結果部分）
    - 結果X
    ```

    ## Guidelines
  new_string: |
    （既存の最終Exampleの結果部分）
    - 結果X
    ```

    ### Example {M+1}: {Exampleタイトル}（{機能名}機能）

    ```
    【状況】
    ...

    【結果】
    - 結果1
    ```

    ## Guidelines
```

### Phase 8: Guidelines（必須ルール・アンチパターン）の追加

新機能に関するルールと禁止事項を追加する。

#### Step 8.1: 必須ルールの追加

```
【必須ルール追加パターン】

既存:
### 必須ルール

1. **既存ルール1**
   - 説明

2. **既存ルール2**
   - 説明

拡張後:
### 必須ルール

1. **既存ルール1**
   - 説明

2. **既存ルール2**
   - 説明

3. **新ルール1（{機能名}機能）**
   - 新機能に関するルールの説明

4. **新ルール2（{機能名}機能）**
   - 新機能に関するルールの説明
```

#### Step 8.2: アンチパターンの追加

```
【アンチパターン追加パターン】

既存:
### アンチパターン（避けるべきこと）

1. **既存アンチパターン1**
   - 説明
   - 対策:

2. **既存アンチパターン2**
   - 説明
   - 対策:

拡張後:
### アンチパターン（避けるべきこと）

1. **既存アンチパターン1**
   - 説明
   - 対策:

2. **既存アンチパターン2**
   - 説明
   - 対策:

3. **新アンチパターン1（{機能名}機能）**
   - 新機能に関する禁止事項の説明
   - 対策: 正しい方法の説明

4. **新アンチパターン2（{機能名}機能）**
   - 新機能に関する禁止事項の説明
   - 対策: 正しい方法の説明
```

#### Step 8.3: 効率化のコツの追加（オプション）

```
【効率化のコツ追加パターン】

既存の効率化のコツの後に追加:

5. **新機能の効率化のコツ1**
   - コツの説明

6. **新機能の効率化のコツ2**
   - コツの説明
```

## Output Format

```
# スキル拡張完了レポート

## 対象スキル
- パス: {target_skill.path}
- スキル名: {name}

## 拡張内容

### 変更セクション一覧

| セクション | 変更種別 | 変更内容 |
|------------|----------|----------|
| YAML Front Matter | 拡張 | description に新機能の説明を追記 |
| When to Use | 追加 | トリガー N 件、ユースケース M 件追加 |
| Input Format | 追加/なし | 新パラメータ X 件追加 |
| Instructions | 追加 | Phase {N+1}, Phase {N+2} を追加 |
| Output Format | 追加/なし | 出力項目 Y 件追加 |
| Examples | 追加 | Example {M+1}, Example {M+2} を追加 |
| Guidelines | 追加 | 必須ルール A 件、アンチパターン B 件追加 |

### 後方互換性
- 既存フェーズ: 変更なし
- 既存パラメータ: 変更なし
- 既存Example: 変更なし

### 追加された機能
1. {機能名1}: {機能の説明}
2. {機能名2}: {機能の説明}

## 変更ファイル
- {target_skill.path}
```

## Examples

### Example 1: concurrent-branch-guard への2機能統合

```
【状況】
- 対象スキル: shogun-concurrent-branch-guard
- 追加機能:
  1. concurrent-git-branch-committer（並行ブランチコミット）
  2. atomic-git-multiagent-committer（アトミックマルチエージェントコミット）
- 既存フェーズ: Phase 1 〜 Phase 5
- 既存Example: Example 1 〜 Example 3

【実行手順】

Phase 1: 構造分析
  - 既存フェーズ数: 5
  - 既存Example数: 3
  - 既存トリガー数: 8

Phase 2: description 拡張
  Edit: description に「さらに、concurrent-git-branch-committer機能...」を追記

Phase 3: When to Use 追加
  Edit: トリガー4件追加
  - 「複数足軽の作業を1つのコミットにまとめて」
  - 「並行ブランチでの安全なコミット方法を教えて」
  - 「分散作業を統合してコミットしたい」
  Edit: トリガーキーワードに「並行コミット, アトミックコミット」追加

Phase 4: Input Format
  - 追加なし（既存パラメータで対応可能）

Phase 5: 新フェーズ追加
  - Phase 6: 並行ブランチコミット（Step 6.1〜6.3）
  - Phase 7: アトミックマルチエージェントコミット（Step 7.1〜7.4）

Phase 6: Output Format
  - 追加なし

Phase 7: Examples 追加
  - Example 4: 並行ブランチコミットの使用例
  - Example 5: アトミックマルチエージェントコミットの使用例

Phase 8: Guidelines 追加
  - 必須ルール: 2件追加（並行ブランチコミット、アトミックコミット選択基準）
  - アンチパターン: 3件追加（マージ順序無視、待ち長期化、乱用）

【結果】
- Phase数: 5 → 7
- Example数: 3 → 5
- トリガー数: 8 → 12
- 必須ルール数: 6 → 8
- アンチパターン数: 10 → 13
- 後方互換性: 維持
```

### Example 2: mcp-server-scaffold へのテストパターン追加

```
【状況】
- 対象スキル: shogun-mcp-server-scaffold
- 追加機能: Spring MockMvc テストパターン生成
- 既存フェーズ: Phase 1 〜 Phase 4（スキャフォールド生成）

【実行手順】

Phase 1: 構造分析
  - 既存はスキャフォールド生成のみ
  - テストコード生成機能がない

Phase 2: description 拡張
  Edit: 「また、MockMvc/WireMock/JUnit5を組み合わせたToolテストパターンも生成可能。」追記

Phase 3: When to Use 追加
  Edit:
  - 「MCPエンドポイントのテストコードを生成して」
  - 「MockMvcのテストパターンを教えて」

Phase 4: Input Format 追加
  新パラメータ:
  - test_pattern: "mockvc" | "wiremock" | "both"
  - test_framework: "junit5"

Phase 5: 新フェーズ追加
  - Phase 5: MockMvc テストパターン生成（Step 5.1〜5.3）

Phase 6: Output Format 追加
  - テストクラステンプレート出力

Phase 7: Examples 追加
  - Example 4: Tool用MockMvcテスト生成例

Phase 8: Guidelines 追加
  - 必須ルール: テストカバレッジ80%以上
  - アンチパターン: 本番環境へのHTTPアクセス

【結果】
- テスト生成機能が統合された
- 既存のスキャフォールド機能は変更なし
```

### Example 3: トリガー追加のみの軽量拡張

```
【状況】
- 対象スキル: shogun-doc-translator-ja
- 追加機能: 新しい呼び出しパターンの追加のみ
- 処理ロジックは変更なし

【実行手順】

Phase 1: 構造分析
  - 既存機能で「READMEスワップパターン」に対応済み
  - ただしトリガーキーワードに明記されていない

Phase 2: description 拡張
  Edit: 「README_ja.mdを作成してスワップして」を追記

Phase 3: When to Use 追加
  Edit:
  - 「README_ja.mdを作成してスワップして」

Phase 4〜8: 変更なし
  - 既存機能で対応可能なため、処理フローの追加は不要

【結果】
- トリガーキーワードの追加のみ
- 実質的な機能追加なし（ドキュメント改善）
- 最小限の変更で発見性を向上
```

## Guidelines

### 必須ルール

1. **後方互換性を必ず維持すること**
   - 既存のフェーズ番号、Example番号を変更しない
   - 既存のパラメータを削除・変更しない
   - 新パラメータはオプション（デフォルト値あり）にする
   - 既存の動作を壊さない

2. **拡張は追記方式で行うこと**
   - description は「さらに、」「また、」で接続して追記
   - When to Use は既存リストの後に追加
   - Instructions は既存フェーズの後に新フェーズを追加
   - Examples は既存Exampleの後に追加
   - Guidelines は既存ルールの後に追加

3. **フェーズ番号・Example番号の連番を維持すること**
   - 既存: Phase 1〜5 → 追加: Phase 6, 7
   - 既存: Example 1〜3 → 追加: Example 4, 5
   - 番号の飛びや重複は禁止

4. **新機能を明示的にマークすること**
   - フェーズタイトルに「（{機能名}機能）」を付与
   - Exampleタイトルに「（{機能名}機能）」を付与
   - Guidelines項目に「（{機能名}機能）」を付与
   - どの拡張で追加されたかを追跡可能にする

5. **8フェーズの順序を守ること**
   - Phase 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 の順で実行
   - フェーズをスキップする場合は「変更なし」と明記
   - 順序を入れ替えない

6. **構造分析レポートを必ず作成すること**
   - Phase 1 で現在の構造を把握
   - 拡張計画を明確化
   - 変更漏れを防止

### 効率化のコツ

1. **軽量拡張の判断**
   - 新トリガーの追加のみで済む場合はPhase 2, 3のみ実行
   - 処理ロジックの追加が不要な場合はPhase 5をスキップ
   - オーバーエンジニアリングを避ける

2. **複数機能の同時追加**
   - 関連する複数機能は1回の拡張で追加
   - 各機能に対応するフェーズ・Exampleをまとめて追加
   - 拡張回数を減らし、一貫性を維持

3. **Exampleの具体性**
   - 抽象的な例よりも具体的なシナリオを記述
   - 実際のコマンド・コード例を含める
   - 「結果」セクションで効果を明示

4. **Guidelinesの優先順位**
   - 重要なルールほど先に記述
   - アンチパターンには必ず「対策」を記述
   - 実際に発生しうる問題を記載

### アンチパターン（避けるべきこと）

1. **既存フェーズの番号変更**
   - Phase 3 を Phase 4 にずらして新フェーズを挿入
   - 対策: 新フェーズは必ず末尾に追加

2. **既存パラメータの削除・変更**
   - 既存の必須パラメータをオプションに変更
   - パラメータ名を変更
   - 対策: 既存パラメータは変更せず、新パラメータを追加

3. **description の全面書き換え**
   - 既存の説明を削除して新しい説明に置換
   - 対策: 「さらに、」で追記、既存説明は維持

4. **Exampleの上書き**
   - 既存の Example 3 を新しい内容に置換
   - 対策: Example 4 として追加

5. **Guidelinesの矛盾**
   - 既存ルールと矛盾する新ルールを追加
   - 対策: 既存ルールとの整合性を確認してから追加

6. **トリガーの重複**
   - 既存トリガーと同じ文言を追加
   - 対策: 既存トリガーを確認してから追加

7. **機能マークの省略**
   - どの拡張で追加されたか分からない
   - 対策: 「（{機能名}機能）」を必ず付与

8. **構造分析のスキップ**
   - 既存構造を把握せずに拡張を開始
   - 対策: Phase 1 を必ず実行
