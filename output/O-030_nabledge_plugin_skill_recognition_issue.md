# O-030: nabledge-6プラグイン スキル認識タイミング問題の調査

## 概要

nabledge-6プラグインのセットアップスクリプト（`setup-6-cc.sh`）を実行後、Claude Codeを起動すると`/plugin`コマンドではプラグインが認識されるにもかかわらず、スキルとしては認識されないという問題の原因を調査した。

## 調査対象

- セットアップスクリプト: `https://raw.githubusercontent.com/nablarch/nabledge/main/setup-6-cc.sh`
- プラグイン: nabledge-6（nablarch/nabledgeリポジトリ）
- 影響環境: Claude Code CLI（インタラクティブモード）

## 再現手順と現象

### 再現手順

1. `curl -sSL https://raw.githubusercontent.com/nablarch/nabledge/main/setup-6-cc.sh | bash` を実行
2. Claude Codeを通常モード（インタラクティブモード）で起動
3. `/plugin`コマンドでプラグインの認識状態を確認
4. スキル一覧でnabledge-6の認識状態を確認

### 観測された現象

| 起動回数 | 起動モード | Trust Dialog | `/plugin`の表示 | スキル認識 |
|----------|-----------|-------------|----------------|-----------|
| 1回目    | インタラクティブ | 表示されず | installed ✅    | 未認識 ❌  |
| 2回目（再起動後） | インタラクティブ | 表示されず | installed ✅ | 認識 ✅ |

両回ともインタラクティブモード（通常起動）であり、ヘッドレスモード（`-p`フラグ）は使用していない。また、いずれの起動時もTrust Dialog（承認ダイアログ）は表示されなかった。

### 再現確認（2026-02-16実施）

`~/.claude/plugins/` 配下のnabledge-6関連ファイルを削除した状態でClaude Codeを起動したところ、初回起動時にプラグインは認識されるがスキルは認識されないという事象が再現された。

#### 削除したファイル

- `~/.claude/plugins/known_marketplaces.json` からnabledgeエントリを削除
- `~/.claude/plugins/installed_plugins.json` からnabledge-6@nabledgeエントリを削除
- `~/.claude/plugins/cache/nabledge/` ディレクトリを削除
- `~/.claude/plugins/marketplaces/nabledge/` ディレクトリを削除

#### 前提条件

プロジェクトレベルの`.claude/settings.json`は、セットアップスクリプト実行後の状態（`extraKnownMarketplaces`と`enabledPlugins`の設定を含む）を維持したまま実施。

#### 再現結果

初回起動時に `/plugin` コマンドではプラグインが「installed」と表示されたが、スキル一覧には表示されず、上記の観測された現象（1回目）と同一の挙動を確認した。

## 原因

### 根本原因: マーケットプレイスの非同期ロードによるレースコンディション

Claude Codeの既知の問題（[Issue #10997](https://github.com/anthropics/claude-code/issues/10997)）に起因する。

### プラグインロードのシーケンス分析

#### 初回起動時（問題が発生）

```
1. Claude Code起動（インタラクティブモード）
2. settings.json読み込み
   → enabledPlugins: {"nabledge-6@nabledge": true} を認識
   → /plugin で "installed" と表示される ✅
3. extraKnownMarketplaces の処理開始
   → GitHubからマーケットプレイスを【非同期】でfetch
4. この間に、スキルローディングが【先に】実行される
5. ~/.claude/plugins/cache/ にまだコンテンツがない
   → スキル未認識 ❌
6. （バックグラウンドで）マーケットプレイスfetch完了
   → ~/.claude/plugins/marketplaces/nabledge/ に書き込み
   → ~/.claude/plugins/known_marketplaces.json を更新
   → ~/.claude/plugins/cache/nabledge/nabledge-6/0.1/ にキャッシュ書き込み
   → ~/.claude/plugins/installed_plugins.json を更新
```

#### 2回目の起動時（正常動作）

```
1. Claude Code起動（インタラクティブモード）
2. settings.json読み込み
   → enabledPlugins を認識 ✅
3. known_marketplaces.json にキャッシュあり
   → 【同期的】にロード
4. cache/ にプラグインコンテンツあり
   → SKILL.md読み込み → スキル認識 ✅
```

### extraKnownMarketplacesの処理トリガー

[調査Gist](https://gist.github.com/alexey-pelykh/566a4e5160b305db703d543312a1e686)によると、`extraKnownMarketplaces`は**インタラクティブなTrust Dialogの承認イベントハンドラ内で**処理されるとされている。しかし、今回の事象ではTrust Dialogが表示されなかったにもかかわらず、初回起動中にマーケットプレイスfetchとプラグインインストールが実行された（`known_marketplaces.json`と`installed_plugins.json`が初回セッション中に作成されている）。

これは、Trust Dialog以外にも`extraKnownMarketplaces`を処理するコードパスが存在することを示している。考えられるメカニズム:

- **既にTrust済みの環境**: 過去に一度でもTrust Dialogを承認済みの場合、以降のセッションではダイアログをスキップしつつ、設定処理（`extraKnownMarketplaces`の読み込みを含む）は実行される
- **起動時の設定処理フロー**: Claude Codeの起動シーケンスにおいて、Trust Dialog表示の有無とは独立して`settings.json`の`extraKnownMarketplaces`を処理するパスが存在する可能性がある

いずれの場合も、マーケットプレイスfetchが**非同期**で行われるという根本的な構造は変わらないため、初回起動時のレースコンディションは発生する。

### 2層のストレージシステム

Claude Codeはプラグイン管理に2層のストレージを使用している。`/plugin`コマンドとスキルシステムが参照するデータソースが異なることが、「pluginでは認識、skillsでは未認識」という非対称な挙動を生む。

| レベル | ファイル | 用途 | 参照元 |
|--------|----------|------|--------|
| プロジェクト | `.claude/settings.json` (`enabledPlugins`) | プラグインの有効化宣言 | `/plugin`コマンド |
| ユーザー | `~/.claude/plugins/known_marketplaces.json` | マーケットプレイスカタログ | マーケットプレイスローダー |
| ユーザー | `~/.claude/plugins/installed_plugins.json` | インストール済みプラグインの実体管理 | プラグインマネージャ |
| ユーザー | `~/.claude/plugins/cache/{marketplace}/{plugin}/{version}/` | スキルファイルの実体 | スキルシステム |

### ヘッドレスモードについて（参考情報）

ヘッドレスモード（`-p`フラグ）ではTrust Dialogがスキップされ、`extraKnownMarketplaces`は一切処理されない。`hasTrustDialogAccepted: true` を手動設定しても、インタラクティブなダイアログイベントなしではインストールがトリガーされない。

ただし、**今回の事象はインタラクティブモードで発生しており、ヘッドレスモードは無関係**である。ヘッドレスモードの制約は、CI/CDパイプラインやスクリプトからの自動実行を検討する場合に留意すべき事項となる。

## 現在の状態（調査時点のファイル構造）

### settings.json（プロジェクトレベル）

```json
{
  "extraKnownMarketplaces": {
    "nabledge": {
      "source": {
        "source": "github",
        "repo": "nablarch/nabledge",
        "ref": "main"
      }
    }
  },
  "enabledPlugins": {
    "nabledge-6@nabledge": true
  }
}
```

### known_marketplaces.json（ユーザーレベル）

```json
{
  "nabledge": {
    "source": {
      "source": "github",
      "repo": "nablarch/nabledge",
      "ref": "main"
    },
    "installLocation": "/home/kuma/.claude/plugins/marketplaces/nabledge",
    "lastUpdated": "2026-02-16T12:31:10.672Z"
  }
}
```

### installed_plugins.json（ユーザーレベル）

```json
{
  "version": 2,
  "plugins": {
    "nabledge-6@nabledge": [
      {
        "scope": "project",
        "installPath": "/home/kuma/.claude/plugins/cache/nabledge/nabledge-6/0.1",
        "version": "0.1",
        "installedAt": "2026-02-16T13:00:47.400Z",
        "lastUpdated": "2026-02-16T13:00:47.400Z",
        "gitCommitSha": "507a8956d281e5fffb02f37e1cf6366674991c85",
        "projectPath": "/home/kuma/nabledge-test"
      }
    ]
  }
}
```

### .orphaned_at ファイル

`~/.claude/plugins/cache/nabledge/nabledge-6/0.1/skills/nabledge-6/.orphaned_at` にタイムスタンプ `1771246847392`（ミリ秒精度Unixタイムスタンプ、**2026-02-16T13:00:47.392 UTC**）が記録されている。

このタイムスタンプは`installed_plugins.json`の`installedAt`（2026-02-16T13:00:47.400Z）とわずか**8ミリ秒**の差であり、インストールとほぼ同時に作成されている。この同時性から、`.orphaned_at`はキャッシュクリーンアップ機構による後発的なマーキングではなく、インストールプロセス自体の一部として作成されたと考えられる。具体的には、プラグインが`projectPath: "/home/kuma/nabledge-test"` にscope=projectで紐づいているため、別のプロジェクトディレクトリからアクセスした場合や、プロジェクトスコープ外の状態判定において「孤立」扱いされる可能性がある。

## setup-6-cc.sh の処理内容

セットアップスクリプトは以下のみを実行する：

1. プロジェクトルートの特定（`git rev-parse --show-toplevel`）
2. `jq`のインストール確認
3. `$PROJECT_ROOT/.claude/settings.json` に以下を書き込み：
   - `extraKnownMarketplaces.nabledge`: マーケットプレイスソース定義
   - `enabledPlugins["nabledge-6@nabledge"]`: `true`

**スクリプトはユーザーレベルのファイル（`~/.claude/plugins/`配下）には一切書き込みを行わない。** これが初回起動時にスキルが認識されない根本的な原因である。

## 関連Issue

| Issue | タイトル | 状態 |
|-------|---------|------|
| [#10997](https://github.com/anthropics/claude-code/issues/10997) | SessionStart hooks don't execute on first run with GitHub marketplace plugins | 同一の非同期ロード問題 |
| [#17832](https://github.com/anthropics/claude-code/issues/17832) | Directory marketplace plugins not auto-enabled in settings.json | enabledPluginsの自動追加漏れ |
| [#20661](https://github.com/anthropics/claude-code/issues/20661) | Plugins installed but not added to enabledPlugins | プラグイン認識の不整合 |
| [#16453](https://github.com/anthropics/claude-code/issues/16453) | Plugin cache grows indefinitely without automatic cleanup | キャッシュ管理の問題 |
| [#18426](https://github.com/anthropics/claude-code/issues/18426) | Plugin Cache Not Invalidated When Files Are Deleted | キャッシュ無効化の問題 |
| [#9641](https://github.com/anthropics/claude-code/issues/9641) | Plugin Manager Shows 'No plugins installed' Despite Successful Installation | プラグイン表示の不整合 |

## 改善案

### 案1: セットアップスクリプトでユーザーレベルのキャッシュも事前構築する（推奨）

`settings.json`への書き込みに加えて、`~/.claude/plugins/`配下のファイルも事前にセットアップする。

```bash
PLUGINS_DIR="$HOME/.claude/plugins"
MARKETPLACE_DIR="$PLUGINS_DIR/marketplaces/nabledge"
CACHE_DIR="$PLUGINS_DIR/cache/nabledge/nabledge-6/0.1"

# 1. マーケットプレイスをgit cloneで事前取得
if [ ! -d "$MARKETPLACE_DIR" ]; then
  git clone --depth 1 --branch main \
    https://github.com/nablarch/nabledge.git "$MARKETPLACE_DIR"
fi

# 2. known_marketplaces.json を事前構築（既存エントリとマージ）
KNOWN_MARKETPLACES_FILE="$PLUGINS_DIR/known_marketplaces.json"
if [ ! -f "$KNOWN_MARKETPLACES_FILE" ]; then
  echo '{}' > "$KNOWN_MARKETPLACES_FILE"
fi
CURRENT_KM=$(cat "$KNOWN_MARKETPLACES_FILE")
UPDATED_KM=$(echo "$CURRENT_KM" | jq \
  --arg install_loc "$MARKETPLACE_DIR" \
  '.nabledge = {
    "source": {
      "source": "github",
      "repo": "nablarch/nabledge",
      "ref": "main"
    },
    "installLocation": $install_loc,
    "lastUpdated": (now | todate)
  }')
echo "$UPDATED_KM" > "$KNOWN_MARKETPLACES_FILE"

# 3. cacheディレクトリにプラグインコンテンツをコピー
mkdir -p "$CACHE_DIR"
cp -r "$MARKETPLACE_DIR/plugins/nabledge-6/"* "$CACHE_DIR/"

# 4. installed_plugins.json を事前構築（既存エントリとマージ）
INSTALLED_PLUGINS_FILE="$PLUGINS_DIR/installed_plugins.json"
if [ ! -f "$INSTALLED_PLUGINS_FILE" ]; then
  echo '{"version": 2, "plugins": {}}' > "$INSTALLED_PLUGINS_FILE"
fi
GIT_SHA=$(cd "$MARKETPLACE_DIR" && git rev-parse HEAD)
CURRENT_IP=$(cat "$INSTALLED_PLUGINS_FILE")
UPDATED_IP=$(echo "$CURRENT_IP" | jq \
  --arg install_path "$CACHE_DIR" \
  --arg git_sha "$GIT_SHA" \
  --arg project_path "$PROJECT_ROOT" \
  '
  .version = 2 |
  .plugins["nabledge-6@nabledge"] = [
    {
      "scope": "project",
      "installPath": $install_path,
      "version": "0.1",
      "installedAt": (now | todate),
      "lastUpdated": (now | todate),
      "gitCommitSha": $git_sha,
      "projectPath": $project_path
    }
  ]
  ')
echo "$UPDATED_IP" > "$INSTALLED_PLUGINS_FILE"
```

この方法により、Claude Codeの初回起動時にキャッシュが既に存在するため、同期的にスキルがロードされる。

### 案2: ドキュメントに「初回は2回起動が必要」と明記

最小限の対応。根本解決にはならないが、ユーザーの混乱は防げる。

### 案3: Claude Code側のIssue修正を待つ

[Issue #10997](https://github.com/anthropics/claude-code/issues/10997)がまさにこの問題。修正されれば`settings.json`への書き込みだけで初回起動時もスキルが認識されるようになる。

## 結論

`setup-6-cc.sh`が`.claude/settings.json`に`extraKnownMarketplaces`と`enabledPlugins`を書き込むだけでは、Claude Code初回起動時にマーケットプレイスのfetchが非同期で行われるため、スキルローディングとのレースコンディションが発生する。今回の事象では、両回ともインタラクティブモードで起動しTrust Dialogは表示されなかったが、初回起動時のバックグラウンドfetchが完了した後、2回目の起動でキャッシュから同期的にロードされスキルが認識された。`~/.claude/plugins/cache/`およびユーザーレベルのメタデータファイルへの事前書き込みをスクリプトに追加することで、初回起動時からスキルが認識されるようにできる。

## 調査日

2026-02-16

## 調査者

Claude Code（Claude Opus 4.6）
