# Nablarch Agent Skills 設計書

**作成日**: 2026-02-10

---

## 1. 目的

Nablarchフレームワークの静的知識を**Claude Code Agent Skills形式**（.md）で提供する。
MCPサーバー（動的ツール）と並行する**静的知識配布レイヤー**として、サーバー不要・軽量・クロスプラットフォームな知識提供を実現する。

## 2. 設計方針

| 方針 | 詳細 |
|------|------|
| **FQCN正確性** | 既存知識YAML（handler-catalog.yaml等）と公式Javadocで検証済みのFQCNのみ使用 |
| **コード例の実用性** | コピー&ペースト可能な実動作コード |
| **Progressive Disclosure** | description（約100トークン）でスキル選択、フル内容は呼び出し時のみロード |
| **知識YAMLとの整合性** | MCP Server側の知識YAMLと内容は同一。Markdown形式に変換して人間可読性を向上 |
| **日本語** | 全ドキュメント日本語で記述 |

## 3. スキル一覧

| # | ファイル名 | 目的 | 情報源 |
|---|-----------|------|--------|
| 1 | `nablarch-handler-queue-guide.md` | ハンドラキュー構成の知識・設計ガイド | handler-catalog.yaml, handler-constraints.yaml |
| 2 | `nablarch-api-design-guide.md` | Action/Form/Entity設計パターン | api-patterns.yaml, design-patterns.yaml |
| 3 | `nablarch-error-handling-guide.md` | エラーハンドリング・例外設計 | error-catalog.yaml, antipattern-catalog.yaml |
| 4 | `nablarch-component-xml-guide.md` | コンポーネント定義XML設定 | config-templates.yaml |
| 5 | `nablarch-testing-guide.md` | テスティングフレームワーク利用ガイド | module-catalog.yaml, api-patterns.yaml (testing) |
| 6 | `nablarch-migration-guide.md` | Nablarch 5→6移行ガイド | version-info.yaml, module-catalog.yaml |

## 4. 各スキルの構成

### 4.1 nablarch-handler-queue-guide.md
- **想定質問**: 「ハンドラキューの設計」「ハンドラの順序」「バッチのハンドラ構成」
- **含む内容**: Web/REST/バッチ/メッセージングの基本パターン（XML設定例付き）、順序制約表、全ハンドラFQCN一覧、カスタムハンドラ作成例
- **FQCN検証**: handler-catalog.yaml記載の25ハンドラ全FQCNを使用

### 4.2 nablarch-api-design-guide.md
- **想定質問**: 「アクションの作り方」「REST API」「バリデーション」「UniversalDao」
- **含む内容**: Webアクション（JSP画面遷移）、RESTアクション（JAX-RS）、バッチアクション、フォームバリデーション、エンティティ、UniversalDao、外部SQLファイル、セッション管理、排他制御
- **FQCN検証**: api-patterns.yaml記載のFQCNを使用

### 4.3 nablarch-error-handling-guide.md
- **想定質問**: 「エラーハンドリング」「例外処理」「ApplicationException」
- **含む内容**: 例外分類（業務/システム）、多層エラー処理構造、GlobalErrorHandler、HttpErrorHandler、ApplicationException、エラーメッセージ定義、よくあるエラーの原因と対処法
- **FQCN検証**: error-catalog.yaml記載のFQCNを使用

### 4.4 nablarch-component-xml-guide.md
- **想定質問**: 「XML設定」「コンポーネント定義」「DIコンテナ」「SystemRepository」
- **含む内容**: XML基本構造、component/component-ref/config-file/import要素、web.xml、DB接続設定、セッションストア設定、スレッドコンテキスト設定、ルーティング設定
- **FQCN検証**: config-templates.yaml記載のFQCNを使用

### 4.5 nablarch-testing-guide.md
- **想定質問**: 「テストの書き方」「リクエスト単体テスト」「Excelテストデータ」
- **含む内容**: SimpleRestTestSupport、DbAccessTestSupport、Excelテストデータ、バッチテスト、テスト設定、Maven依存関係
- **FQCN検証**: module-catalog.yaml (testing) 記載のFQCNを使用

### 4.6 nablarch-migration-guide.md
- **想定質問**: 「Nablarch 5→6移行」「Jakarta EE」「javax→jakarta」
- **含む内容**: 移行チェックリスト、パッケージ名変更一覧、web.xml更新、エンティティ/バリデーション/JAX-RS/Jakarta Batchのアノテーション変更、BOM設定、動作確認済みプラットフォーム
- **FQCN検証**: version-info.yaml, module-catalog.yaml記載の情報を使用

## 5. FQCN検証結果

全スキルで使用したFQCNは、以下の情報源で検証済み:

| 情報源 | 検証対象 | 件数 |
|--------|---------|------|
| handler-catalog.yaml | ハンドラ全FQCN | 25件 |
| api-patterns.yaml | API/ライブラリFQCN | 20件 |
| error-catalog.yaml | エラー関連FQCN | 10件 |
| config-templates.yaml | 設定関連FQCN | 12件 |
| module-catalog.yaml | モジュール/キークラスFQCN | 30件 |
| design-patterns.yaml | 設計パターンFQCN | 15件 |

**総検証FQCN数**: 約112件（重複除く約70件のユニークFQCN）

注: FQCNは既存の知識YAMLファイルに記載されているものを使用。知識YAMLの精度がそのままスキルの精度となる。品質評価で指摘されたFQCN誤りは、知識拡充タスクで修正される予定。

## 6. MCP ServerのResourceとの関係

| Skills | MCP Resource | 関係 |
|--------|-------------|------|
| handler-queue-guide | `handler/{app_type}` | 同一知識の異なるビュー（Markdown vs YAML） |
| api-design-guide | `pattern/{name}` | Skillsは全パターン統合、Resourceは個別パターン |
| error-handling-guide | `guide/{topic}` | Skillsはエラー特化、Resourceは汎用ガイド |
| component-xml-guide | `config/{name}` | Skillsは設定ガイド統合、Resourceは個別テンプレート |
| testing-guide | （対応なし） | Skills独自（MCP Resourceに対応するものなし） |
| migration-guide | `version` | Skillsは移行手順、Resourceはバージョン情報 |

**方針**: MCP Resourceは削除しない。Skillsは「サーバーなしでも使える軽量版」、Resourcesは「MCPサーバー接続時の高精度版」として併存。

## 7. 配置先

```
~/nablarch-mcp-server-skills/.claude/skills/
├── nablarch-handler-queue-guide.md
├── nablarch-api-design-guide.md
├── nablarch-error-handling-guide.md
├── nablarch-component-xml-guide.md
├── nablarch-testing-guide.md
└── nablarch-migration-guide.md
```
