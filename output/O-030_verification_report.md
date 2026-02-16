# O-030 技術的正確性検証レポート

## 検証サマリ

| # | 検証項目 | 判定 | 根拠概要 |
|---|---------|------|---------|
| 1 | 根本原因（レースコンディション） | ✅正確 | Issue #10997の内容が完全に一致。非同期ロード→スキル先行ロードの構造 |
| 2 | 2層ストレージシステム | ⚠部分的 | ファイル構造は正確。ただしinstalled_plugins.jsonのlastUpdatedフィールドが欠落 |
| 3 | シーケンス分析 | ✅正確 | Issue #10997のRoot CauseセクションおよびGistの分析と整合 |
| 4 | setup-6-cc.shの処理内容 | ✅正確 | 実際のスクリプトで確認。settings.jsonのみへの書き込み |
| 5 | 改善案1のスクリプト | ✅正確 | jq構文・ディレクトリパス・処理フローすべて正しい |
| 6 | .orphaned_atファイルの解釈 | ❌誤り | タイムスタンプ変換が誤り（約16時間のずれ）。キャッシュクリーンアップの解釈も根拠不十分 |
| 7 | ヘッドレスモード/Trust Dialog | ✅正確 | Gistの調査内容と一致。-pフラグでTrust Dialogスキップされる |

**総合判定**: 7項目中 ✅正確5件、⚠部分的1件、❌誤り1件

---

## 詳細検証

### 1. 根本原因の主張（レースコンディション）

**レポートの主張**: マーケットプレイスの非同期ロードによるレースコンディションがスキル未認識の原因。Claude Code Issue #10997に起因する。

**検証結果**: ✅正確

**根拠**:

Issue #10997（[https://github.com/anthropics/claude-code/issues/10997](https://github.com/anthropics/claude-code/issues/10997)）を実際にGitHub APIで取得し確認した。

- **タイトル**: `[BUG] SessionStart hooks don't execute on first run with GitHub marketplace plugins`
- **状態**: CLOSED
- **Issue本文のRoot Causeセクション**（原文引用）:

> Claude Code loads marketplaces asynchronously. The SessionStart hook fires before the asynchronous marketplace fetch completes on first run. On subsequent runs, the cached marketplace loads synchronously and plugins register before SessionStart fires.
>
> **Timeline (First Run)**:
> 1. Settings loaded
> 2. Marketplace fetch begins (async, GitHub download)
> 3. SessionStart hooks fire (plugins not yet registered)
> 4. Marketplace fetch completes (too late)
>
> **Timeline (Subsequent Runs)**:
> 1. Settings loaded
> 2. Cached marketplace loads (instant)
> 3. Plugins registered
> 4. SessionStart hooks fire (plugins available)

O-030レポートのシーケンス分析と完全に一致する。Issueの報告者も再現スクリプトとGitHubリポジトリ（`goodfoot-io/plugin-load-timing-issue-reproduction`）を提供しており、初回起動でhookが実行されず2回目で実行される挙動が確認されている。

**他の関連Issueの検証**:

| Issue | レポートの記載タイトル | 実際のタイトル | 状態 | 判定 |
|-------|----------------------|--------------|------|------|
| [#10997](https://github.com/anthropics/claude-code/issues/10997) | SessionStart hooks don't execute on first run with GitHub marketplace plugins | [BUG] SessionStart hooks don't execute on first run with GitHub marketplace plugins | CLOSED | ✅一致 |
| [#17832](https://github.com/anthropics/claude-code/issues/17832) | Directory marketplace plugins not auto-enabled in settings.json | Directory marketplace plugins not auto-enabled in settings.json | OPEN | ✅一致 |
| [#20661](https://github.com/anthropics/claude-code/issues/20661) | Plugins installed but not added to enabledPlugins | Plugins installed but not added to enabledPlugins - don't appear in Installed tab | CLOSED | ⚠末尾省略あるが実質一致 |
| [#16453](https://github.com/anthropics/claude-code/issues/16453) | Plugin cache grows indefinitely without automatic cleanup | Plugin cache grows indefinitely without automatic cleanup | CLOSED | ✅一致 |
| [#18426](https://github.com/anthropics/claude-code/issues/18426) | Plugin Cache Not Invalidated When Files Are Deleted | [BUG] Plugin Cache Not Invalidated When Files Are Deleted | CLOSED | ✅一致（[BUG]プレフィックス省略のみ） |
| [#9641](https://github.com/anthropics/claude-code/issues/9641) | Plugin Manager Shows 'No plugins installed' Despite Successful Installation | Plugin Manager Shows 'No plugins installed' Despite Successful Installation | CLOSED | ✅一致 |

全6件のIssueが実在し、タイトル・内容ともにレポートの記述と整合する。なお、レポートの「状態」列は実際にはIssueの状態（OPEN/CLOSED）ではなく関連性の説明になっている点は記法上の注意点。

---

### 2. 2層ストレージシステムの説明

**レポートの主張**: settings.json / known_marketplaces.json / installed_plugins.json / cache/ の4層構造でプラグインを管理。

**検証結果**: ⚠部分的に正確

**根拠**:

実際のファイルをローカル環境で確認した。

**known_marketplaces.json**（`~/.claude/plugins/known_marketplaces.json`）:
- 実在する ✅
- nabledgeエントリの内容はレポートと完全に一致 ✅
- claude-plugins-officialエントリも存在（レポートには記載なしだが、対象外なので問題なし）

**installed_plugins.json**（`~/.claude/plugins/installed_plugins.json`）:
- 実在する ✅
- レポートの引用では `lastUpdated` フィールドが**欠落**している ⚠

レポートの引用:
```json
{
  "installedAt": "2026-02-16T13:00:47.400Z",
  "gitCommitSha": "507a8956d281e5fffb02f37e1cf6366674991c85",
  "projectPath": "/home/kuma/nabledge-test"
}
```

実際のファイル:
```json
{
  "installedAt": "2026-02-16T13:00:47.400Z",
  "lastUpdated": "2026-02-16T13:00:47.400Z",
  "gitCommitSha": "507a8956d281e5fffb02f37e1cf6366674991c85",
  "projectPath": "/home/kuma/nabledge-test"
}
```

`lastUpdated` フィールドが実際には存在するが、レポートの引用からは省略されている。

**cache/ディレクトリ** (`~/.claude/plugins/cache/nabledge/nabledge-6/0.1/`):
- 実在する ✅
- SKILL.md、knowledge/、docs/ 等のファイルが存在 ✅
- パス構成 `cache/{marketplace}/{plugin}/{version}/` はレポートの説明と一致 ✅

**settings.json**（プロジェクトレベル）:
- レポートの内容と整合する構造（検証はローカルのsettings.jsonではなくsetup-6-cc.shが生成する内容で確認）

**レポートの表の正確性**:
- `/plugin`コマンドが`enabledPlugins`を参照するという記述 → 合理的（Issue #10997の挙動と整合）
- スキルシステムが`cache/`を参照するという記述 → 合理的（Issue #10997のTimeline分析と整合）

---

### 3. シーケンス分析の正確性

**レポートの主張**: 初回起動時に非同期fetchが完了する前にスキルローディングが実行され、2回目はキャッシュから同期的にロードされる。extraKnownMarketplacesはTrust Dialog承認イベントハンドラ内でのみ処理される。

**検証結果**: ✅正確

**根拠**:

**Issue #10997のRoot Causeセクション**が初回/2回目のタイムラインを明示的に記述しており、レポートのシーケンスと一致する（検証項目1の引用参照）。

**alexey-pelykh氏のGist**（[https://gist.github.com/alexey-pelykh/566a4e5160b305db703d543312a1e686](https://gist.github.com/alexey-pelykh/566a4e5160b305db703d543312a1e686)）をWebFetchで取得し確認した。Gistの内容:

- `extraKnownMarketplaces`は**インタラクティブなTrust Dialogのイベントハンドラ内で**処理される → レポートの主張と一致
- Trust Dialogでユーザーが承認 → プロジェクト設定が処理される → マーケットプレイスのauto-installがトリガーされる
- ヘッドレスモード（`-p`フラグ）ではTrust Dialogがスキップされるため、`extraKnownMarketplaces`は処理されない

`hasTrustDialogAccepted: true`の手動設定についても、Gistは「インタラクティブなダイアログイベントなしではインストールがトリガーされない」と指摘しており、レポートの記述と整合する。

---

### 4. setup-6-cc.shの処理内容

**レポートの主張**: スクリプトはプロジェクトルートの`.claude/settings.json`に書き込むのみで、ユーザーレベルのファイル（`~/.claude/plugins/`配下）には一切書き込みを行わない。

**検証結果**: ✅正確

**根拠**:

`https://raw.githubusercontent.com/nablarch/nabledge/main/setup-6-cc.sh` から実際のスクリプトを取得し、全体を確認した。スクリプトの処理内容:

1. `git rev-parse --show-toplevel` でプロジェクトルートを特定 ✅
2. `jq`のインストール確認（未インストール時はOS別にインストール） ✅
3. `$PROJECT_ROOT/.claude/settings.json` に以下を書き込み:
   - `extraKnownMarketplaces.nabledge`: マーケットプレイスソース定義 ✅
   - `enabledPlugins["nabledge-6@nabledge"]`: `true` ✅

スクリプト全体（約90行）を精査したが、`~/.claude/` や `$HOME/.claude/` への書き込み処理は**一切存在しない**。`$PROJECT_ROOT/.claude/settings.json` のみが操作対象である。

レポートの記述は正確。

---

### 5. 改善案1のスクリプト

**レポートの主張**: ユーザーレベルのキャッシュを事前構築するbashスクリプトが提案されている。

**検証結果**: ✅正確（技術的に正しく動作する）

**根拠**:

スクリプトの各部分を検証:

**jq構文の正確性**:
- `jq --arg install_loc "$MARKETPLACE_DIR" '.nabledge = {...}'` → 有効なjq構文 ✅
- `now | todate` → jqの組み込み関数で、ISO 8601形式のタイムスタンプを生成する ✅
- `.plugins["nabledge-6@nabledge"] = [...]` → 有効なjq配列代入 ✅

**ディレクトリ構造の想定**:
- `$PLUGINS_DIR/marketplaces/nabledge` → 実際の `~/.claude/plugins/marketplaces/nabledge/` と一致 ✅
- `$PLUGINS_DIR/cache/nabledge/nabledge-6/0.1` → 実際の `~/.claude/plugins/cache/nabledge/nabledge-6/0.1/` と一致 ✅
- `$MARKETPLACE_DIR/plugins/nabledge-6/` → nabledgeリポジトリの実際のディレクトリ構造（`plugins/nabledge-6/`配下にSKILL.md等）と一致 ✅

**処理フローの妥当性**:
1. `git clone --depth 1` でマーケットプレイスを取得 → Claude Codeの実際のインストール挙動と整合
2. `known_marketplaces.json` の事前構築 → 2回目起動時の同期ロードと同じ状態を再現
3. `cp -r` でcacheにコピー → スキルシステムが参照するファイルを事前配置
4. `installed_plugins.json` の事前構築 → プラグインマネージャが認識できる状態を再現

**軽微な注意点**:
- `.orphaned_at` ファイルの扱いについては言及がない（Claude Codeが自動生成する可能性があるが、スクリプトでは特に対処不要と思われる）

---

### 6. .orphaned_atファイルの解釈

**レポートの主張**: タイムスタンプ `1771246847392` は「2026-02-17T05:00:47 UTC頃」であり、キャッシュクリーンアップ機構がスキルを孤立状態としてマークした可能性がある。

**検証結果**: ❌誤り

**根拠**:

**.orphaned_atファイルの実在確認**:
- パス: `~/.claude/plugins/cache/nabledge/nabledge-6/0.1/skills/nabledge-6/.orphaned_at`
- ファイルは実在する ✅
- 内容: `1771246847392` ✅

**タイムスタンプ変換の誤り**:

`1771246847392` はミリ秒精度のUnixタイムスタンプである。正しい変換:

```
1771246847392 ms ÷ 1000 = 1771246847.392 秒
→ 2026-02-16T13:00:47.392 UTC
```

| | レポートの記載 | 正しい変換 |
|---|---|---|
| UTC | 2026-02-17T05:00:47 | **2026-02-16T13:00:47** |
| 差分 | — | **約16時間の誤差** |

レポートの「2026-02-17T05:00:47 UTC頃」は**誤り**である。正しくは「2026-02-16T13:00:47 UTC」。

**キャッシュクリーンアップ解釈への疑問**:

さらに重要な発見として、このタイムスタンプは `installed_plugins.json` の `installedAt` フィールド（`2026-02-16T13:00:47.400Z`）と**ほぼ同一**（差は8ミリ秒）である。

| タイムスタンプ | 値 | UTC |
|---|---|---|
| .orphaned_at | 1771246847392 | 2026-02-16T13:00:47.392Z |
| installedAt | — | 2026-02-16T13:00:47.400Z |
| 差分 | — | **わずか8ミリ秒** |

これは `.orphaned_at` がインストールとほぼ同時に作成されたことを示している。レポートが主張する「キャッシュクリーンアップ機構がスキルを孤立状態としてマーク」という解釈は、**タイミング的に疑わしい**。

より妥当な解釈として考えられるのは:
- プラグインが `projectPath: "/home/kuma/nabledge-test"` にscope=projectで紐づいているため、別のプロジェクトディレクトリからアクセスした場合に「孤立」扱いされる可能性
- または、インストールプロセス自体が一時的にorphaned状態を経由する可能性

いずれにせよ、レポートの「クリーンアップ機構による後発マーキング」という解釈は根拠が不十分である。

---

### 7. ヘッドレスモード/Trust Dialog関連

**レポートの主張**:
1. ヘッドレスモード（`-p`フラグ）ではTrust Dialogがスキップされる
2. `extraKnownMarketplaces`は処理されない
3. `hasTrustDialogAccepted: true`の手動設定が効かない

**検証結果**: ✅正確

**根拠**:

**Gist（alexey-pelykh氏の調査レポート）**:

WebFetchで取得したGistの内容から:

> "print mode (`-p`) skips the trust dialog entirely, which means the settings processing never triggers. The plugin system only consults user-level storage (`~/.claude/plugins/known_marketplaces.json`), completely bypassing project-defined configurations."

→ `-p`フラグでTrust Dialogがスキップされるという主張は**Gistで確認** ✅

> "The auto-installation logic is executed during the interactive trust dialog event handler"

→ `extraKnownMarketplaces`がTrust Dialog承認イベント内でのみ処理されるという主張は**Gistで確認** ✅

**Issue #10997の補足情報**:

Issue本文にも以下の記載がある:

> "This issue only occurs in interactive mode where `process.stdout.isTTY` is true. Non-interactive mode (using `-p` flag) does not load marketplaces at all, which is separate behavior."

→ `-p`フラグでのマーケットプレイス非ロードが**Issue報告者によっても確認** ✅

**`hasTrustDialogAccepted: true`の手動設定について**:

Gistの分析結果に基づく主張であり、Trust Dialogのイベントハンドラ内でのみ処理がトリガーされるという構造から論理的に導かれる結論である。ただし、Claude Codeのソースコードを直接確認した検証ではなく、Gistの調査結果に依拠している点は留意すべき。

---

## 全体評価

O-030レポートは全体として**高い技術的正確性**を持っている。7項目中5項目が完全に正確であり、根本原因の分析、シーケンス分析、スクリプト検証、改善案の技術的妥当性はいずれも確かな根拠に基づいている。

**修正が必要な箇所**:

1. **`.orphaned_at`のタイムスタンプ変換**（検証項目6）: 「2026-02-17T05:00:47 UTC頃」→正しくは「2026-02-16T13:00:47 UTC」。約16時間の誤差がある
2. **`.orphaned_at`の解釈の見直し**: installedAtとのタイミング一致（8ms差）を考慮し、「キャッシュクリーンアップ機構による後発マーキング」ではなく、インストールプロセスに起因する可能性を併記すべき
3. **installed_plugins.jsonの引用補完**（検証項目2）: `lastUpdated`フィールドの追記

## 検証日

2026-02-16

## 検証者

Claude Code（Claude Opus 4.6）/ ashigaru5
