# Nablarch MCP Server 知識データ カバレッジ監査レポート


**監査日**: 2026-02-10
**監査者**: 担当者A
**対象リポジトリ**: ~/nablarch-mcp-server

---

## 1. 監査サマリ

### 現状: 10 YAMLファイル → ライブラリカバレッジ 約30%

Nablarch公式ドキュメントに掲載されている23のライブラリ機能のうち、専用の知識YAMLでカバーされているのはハンドラ関連（handler-catalog, handler-constraints）と設計パターン的な知識のみ。ライブラリ固有の機能詳細（FQCN、API、使用パターン）は大部分が欠落している。

### FQCN誤り・version-info誤り: 修正済み

品質評価レポートで指摘された以下の問題は**既に修正済み**であることを確認：
- FQCN誤り5件: MultipartHandler, HttpAccessLogHandler, StatusCodeConvertHandler, InjectForm, StreamResponse → 全て正しいFQCNに修正済み
- version-info.yaml誤り8件: release_date, WildFly版, OpenLiberty版, 5u系最新版, PostgreSQL版, サポートDB, APサーバー, H2 Database → 全て修正済み

---

## 2. Nablarch公式ライブラリ vs 知識YAMLカバレッジ対照表

| # | 公式ライブラリ | カテゴリ | 現状YAML | カバー状況 | 優先度 |
|---|--------------|---------|---------|:--------:|:------:|
| 1 | **データバインド（data_bind）** | データI/O | なし | ❌ 欠落 | **P0** |
| 2 | **データフォーマット（data_format）** | データI/O | なし | ❌ 欠落 | **P0** |
| 3 | **入力値チェック（validation）** | ビジネスロジック | module-catalogに概要のみ | ⚠️ 不十分 | **P1** |
| 4 | **メール送信（mail）** | コア | なし | ❌ 欠落 | **P1** |
| 5 | **メッセージ管理（message）** | コア | module-catalogに概要のみ | ⚠️ 不十分 | **P1** |
| 6 | **コード管理（code）** | キャッシュ/データ | なし | ❌ 欠落 | **P1** |
| 7 | **ログ出力（log）** | コア | なし | ❌ 欠落 | **P2** |
| 8 | **日付管理（date）** | キャッシュ/データ | module-catalogに概要のみ | ⚠️ 不十分 | **P2** |
| 9 | **ファイルパス管理（file_path）** | コア | なし | ❌ 欠落 | **P2** |
| 10 | **認可チェック（permission_check）** | ビジネスロジック | なし | ❌ 欠落 | **P2** |
| 11 | **セッションストア（session_store）** | トランザクション/状態 | api-patternsに一部パターン | ⚠️ 不十分 | **P2** |
| 12 | **トランザクション管理（transaction）** | トランザクション/状態 | handler-catalogにハンドラのみ | ⚠️ 不十分 | **P2** |
| 13 | **データベースアクセス（database）** | コア | api-patternsにパターンのみ | ⚠️ 不十分 | P3 |
| 14 | **排他制御（exclusive_control）** | ビジネスロジック | なし | ❌ 欠落 | P3 |
| 15 | **静的データキャッシュ（static_data_cache）** | キャッシュ/データ | なし | ❌ 欠落 | P3 |
| 16 | **二重サブミット防止（double_submit）** | トランザクション/状態 | api-patternsに一部 | ⚠️ 不十分 | P3 |
| 17 | **サービス提供可否チェック** | ビジネスロジック | なし | ❌ 欠落 | P3 |
| 18 | **システムリポジトリ（repository）** | コア | config-templatesに一部 | ⚠️ 不十分 | P3 |
| 19 | **BeanUtil** | ユーティリティ | api-patternsに一部 | ⚠️ 不十分 | P3 |
| 20 | **ステートレスWeb** | トランザクション/状態 | なし | ❌ 欠落 | P3 |
| 21 | **JSPカスタムタグ** | UI | なし | ❌ 欠落 | P3 |
| 22 | **汎用ユーティリティ** | ユーティリティ | なし | ❌ 欠落 | P3 |
| 23 | **フォーマッタ** | ユーティリティ | なし | ❌ 欠落 | P3 |

### カバレッジ集計

| カバー状況 | 件数 | 割合 |
|-----------|:----:|:----:|
| ❌ 専用YAML完全欠落 | 14 | 61% |
| ⚠️ 部分的（他YAMLに概要のみ） | 9 | 39% |
| ✅ 専用YAMLで十分カバー | 0 | 0% |

---

## 3. 既存YAML 10ファイルのカバー範囲

| # | ファイル名 | カバー範囲 | 評価 |
|---|----------|-----------|------|
| 1 | handler-catalog.yaml | ハンドラ名・FQCN・説明・順序（6アプリタイプ） | ✅ 良好 |
| 2 | handler-constraints.yaml | ハンドラ順序制約 | ✅ 良好 |
| 3 | api-patterns.yaml | Web/バッチ/REST開発パターン | ✅ 良好 |
| 4 | design-patterns.yaml | 設計パターン11件 | ✅ 良好 |
| 5 | error-catalog.yaml | エラー17件 | ✅ 良好 |
| 6 | module-catalog.yaml | Mavenモジュール一覧 | ✅ 良好 |
| 7 | config-templates.yaml | XML設定テンプレート | ✅ 良好 |
| 8 | example-catalog.yaml | コード例4件 | ✅ 良好 |
| 9 | antipattern-catalog.yaml | アンチパターン7件 | ✅ 良好 |
| 10 | version-info.yaml | バージョン・プラットフォーム情報 | ✅ 修正済み |

**既存YAMLの品質**: FQCN修正・version-info修正が適用済みで、精度は高い。問題は**カバー範囲の狭さ**。

---

## 4. 今回の拡充計画

### Phase 2-P0（最優先）— 1ファイル新規作成

| ファイル名 | 内容 |
|-----------|------|
| `data-io-catalog.yaml` | データバインド（data_bind）+ データフォーマット（data_format）の統合知識 |

### Phase 2-P1 — 4ファイル新規作成

| ファイル名 | 内容 |
|-----------|------|
| `validation-catalog.yaml` | Bean Validation + Nablarch独自バリデーション |
| `mail-catalog.yaml` | メール送信（テンプレート/フリーフォーマット） |
| `message-catalog.yaml` | メッセージ管理 + コード管理（統合） |
| `utility-catalog.yaml` | 日付管理 + ファイルパス管理 + BeanUtil |

### Phase 2-P2 — 2ファイル新規作成

| ファイル名 | 内容 |
|-----------|------|
| `log-catalog.yaml` | ログ出力（ログカテゴリ・Writer・Formatter） |
| `security-catalog.yaml` | 認証/認可 + セッションストア + CSRF防止 + 排他制御 |

### 拡充後の予想カバレッジ

| 項目 | 現状 | 拡充後 |
|------|:----:|:------:|
| 知識YAMLファイル数 | 10 | 17 |
| ライブラリカバレッジ | ~30% | ~85% |
| P0欠落（data_io） | 欠落 | カバー |
| P1欠落（validation等） | 欠落 | カバー |
| P2欠落（log/security等） | 欠落 | カバー |

---

## 5. FQCN誤り修正状況（品質評価での指摘分）

| # | ファイル | 修正対象 | 状態 |
|---|---------|---------|:----:|
| 1 | handler-catalog.yaml | MultipartHandler FQCN | ✅ 修正済み |
| 2 | handler-catalog.yaml | HttpAccessLogHandler FQCN | ✅ 修正済み |
| 3 | handler-catalog.yaml | StatusCodeConvertHandler FQCN | ✅ 修正済み |
| 4 | api-patterns.yaml | InjectForm FQCN | ✅ 修正済み |
| 5 | api-patterns.yaml | StreamResponse FQCN | ✅ 修正済み |
| 6 | version-info.yaml | release_date等8件 | ✅ 修正済み |

---

*監査レポート作成完了: 2026-02-10 担当者A*
