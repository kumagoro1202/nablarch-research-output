# Nablarch AI生成コード品質ゲート・自動生成ツール構築計画

> **作成日**: 2026-01-31
> **タスクID**: subtask_004 (parent: cmd_001)
> **プロジェクト**: nablarch_strategy
> **ペルソナ**: DevSecOps・QAエンジニア

---

## 目次

1. [現状調査結果](#1-現状調査結果)
2. [多層品質ゲートの実装計画](#2-多層品質ゲートの実装計画)
3. [Excelテスト自動生成の実装計画](#3-excelテスト自動生成の実装計画)
4. [CI/CD統合の具体的設計](#4-cicd統合の具体的設計)
5. [段階的構築ロードマップ](#5-段階的構築ロードマップ)
6. [実現可能性の評価](#6-実現可能性の評価)
7. [リスクと対策](#7-リスクと対策)

---

## 1. 現状調査結果

### 1.1 Nablarchフレームワークの概要

Nablarchは、TIS株式会社が金融業界を中心としたミッションクリティカルシステム構築経験を集約して開発したJavaアプリケーションフレームワークである。Apache License 2.0でOSS公開されており、GitHub上に113リポジトリを持つ。

**主な特徴:**
- パイプライン型処理モデルに基づく共通アーキテクチャ
- Web/バッチ/メッセージング等の多様な処理方式をサポート
- ハンドラキュー方式による柔軟な処理制御
- 長期サポート（有償契約中はEOSL設定なし）
- JPCERT/CCによる脆弱性情報の収集・対応

**対応アプリケーション種別:**
- Webアプリケーション（オンライン画面）
- RESTful Webサービス / HTTPメッセージング
- バッチアプリケーション（JSR352準拠 / Nablarchネイティブ）
- MOMメッセージング / テーブルキューベースメッセージング

### 1.2 既存品質ツール

#### 1.2.1 SpotBugsプラグイン（未公開APIチェッカー）

- **リポジトリ**: [nablarch/nablarch-unpublished-api-checker](https://github.com/nablarch/nablarch-unpublished-api-checker)
- **仕組み**: ホワイトリスト方式。許可されたAPIを設定ファイルで定義し、未許可APIの使用をSpotBugsのバグパターン `UPU_UNPUBLISHED_API_USAGE` として検出
- **設定**: Maven `spotbugs-maven-plugin`（4.5.0.0）にプラグインとして追加。`-Dnablarch-findbugs-config=spotbugs/published-config/production` で設定ディレクトリを指定
- **役割別設定**:
  - **プログラマー向け**: ビジネス機能実装に必要なNablarch Application Framework API + テスト用NTF API
  - **アーキテクト向け**: NAF拡張機能用API + NTF拡張用API
- **IDE対応**: Eclipse（SpotBugsメニューからFind Bugs）、IntelliJ IDEA（[nablarch-intellij-plugin](https://github.com/nablarch/nablarch-intellij-plugin)）

#### 1.2.2 Checkstyle

- **リポジトリ**: [nablarch-development-standards/nablarch-style-guide](https://github.com/nablarch-development-standards/nablarch-style-guide)（2022年10月にFintan-contents/coding-standardsへ移行）
- **主要ルール**:
  - `static final`フィールドの命名規約（大文字・アンダースコア・数字）
  - 空catchブロックにはコメント必須
  - static import禁止
  - 過度に一般的な例外（`Exception`, `Throwable`, `RuntimeException`, `Error`）のcatch禁止
- **Nablarch固有規約**:
  - リフレクションの直接使用禁止
  - アーキテクト承認なしの例外クラス作成禁止
  - レガシーJava標準ライブラリAPIの使用禁止

#### 1.2.3 SpotBugs（標準）

- SpotBugs本体による標準バグ検出（FindBugsの後継、Java 8+対応）
- プロジェクト毎のフィルタファイルでカスタマイズ可能
- SonarQubeとの互換性あり（ただしCheckstyle + SpotBugsの方がMaven実行だけで完結するため導入が容易）

#### 1.2.4 コードフォーマッター

- `nablarch-code-formatter.xml`によるコードフォーマット標準化
- Javaスタイルガイドの前提条件: コードフォーマッターで整形済み + Checkstyle違反解消済み + SpotBugs実行済み

### 1.3 Excelテストフレームワーク

#### 1.3.1 概要

NablarchのテストフレームワークはJUnit4ベースであり、テストケース・テスト結果をExcelファイルで定義する独自の仕組みを持つ。

**設計思想**: データベース準備データや検索結果等はJavaソースコードよりスプレッドシートの方が可読性・編集容易性に優れるため、Excelファイルで管理する。

**JUnit 5対応**: JUnit Vintageを利用したJUnit 4テストの継続実行に加え、`nablarch-testing-junit5`によるネイティブJUnit 5拡張も提供（`@NablarchTest`アノテーション）。

#### 1.3.2 Excelファイルの構造

**命名規約:**
- ファイル名: テストソースコードと同名（拡張子のみ異なる）
- 配置: テストソースコードと同じディレクトリ
- シート名: テストメソッド名と同名

**データタイプ:**

| データタイプ | 説明 |
|---|---|
| `SETUP_TABLE` | テスト実行前の準備データ（DB投入用） |
| `EXPECTED_TABLE` | 期待テスト結果（省略カラムは比較対象外） |
| `EXPECTED_COMPLETE_TABLE` | 期待結果（省略カラムはデフォルト値設定） |
| `LIST_MAP` | `List<Map<String,String>>`形式データ |

**特殊記法:**
- `null` → null値
- ダブルクォート囲み → 文字列として処理
- `${systemTime}` → システム日時自動挿入
- `${文字種,文字数}` → 指定文字の自動生成

#### 1.3.3 リクエスト単体テストの種別

| テスト種別 | スーパークラス |
|---|---|
| Webアプリケーション | `HttpRequestTestSupport` |
| RESTful Webサービス | `RestTestSupport` / `SimpleRestTestSupport` |
| バッチ処理 | `BatchRequestTestSupport` |
| メッセージ受信処理 | `MessagingRequestTestSupport` |
| 同期応答メッセージ送信 | `SendSyncMessageRequestTestSupport` |

#### 1.3.4 リクエスト単体データ作成ツール

HTMLからのリクエストパラメータを自動的にExcel形式で取得するツールが提供されている。ブラウザでHTMLを操作し、次画面へのリクエストパラメータをキャプチャすることで人的ミスを削減する。

---

## 2. 多層品質ゲートの実装計画

### 2.1 アーキテクチャ概要

AI生成コード（Copilot / Claude / Cursor等）に対して、以下の多層検証を実施する。

```
┌─────────────────────────────────────────────────────────────┐
│                         AI生成コード                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: 静的解析（既存ツール拡張）                           │
│  ├─ Checkstyle（Nablarchコーディング規約）                    │
│  ├─ SpotBugs（標準バグ検出）                                  │
│  ├─ SpotBugs Nablarch Plugin（未公開APIチェッカー）            │
│  └─ OWASP Dependency-Check（依存ライブラリ脆弱性）             │
└──────────────────────────┬──────────────────────────────────┘
                           │ Pass
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 2: セキュリティスキャン                                 │
│  ├─ SAST: SonarQube / Checkmarx（脆弱性パターン検出）         │
│  ├─ SCA: OWASP Dependency-Check（OSS脆弱性）                 │
│  └─ Secret Detection: gitleaks / TruffleHog                  │
└──────────────────────────┬──────────────────────────────────┘
                           │ Pass
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 3: LLMベースコードレビュー（Nablarch固有ルール）         │
│  ├─ Specification-Grounded Review（SGCR方式）                 │
│  │   └─ Nablarch設計規約・セキュリティ要件をSpecとして供給      │
│  ├─ RAGによるコードベースコンテキスト参照                      │
│  │   └─ 既存プロジェクトの実装パターンをベクトルDB化           │
│  └─ AI-on-AI Review（多モデル交差検証）                       │
└──────────────────────────┬──────────────────────────────────┘
                           │ Pass
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: 自動テスト実行                                      │
│  ├─ AI生成Excelテストケースの実行                             │
│  ├─ 既存リグレッションテスト                                  │
│  └─ IAST（ランタイム脆弱性検出）                              │
└──────────────────────────┬──────────────────────────────────┘
                           │ All Pass
                           ▼
                    ✅ 品質ゲート通過
```

### 2.2 Layer 1: 静的解析（既存ツール拡張）

**目的**: Nablarch固有のコーディング規約への準拠を機械的に検証

**拡張内容:**

| ツール | 現状 | 拡張内容 |
|---|---|---|
| Checkstyle | 標準Nablarch規約 | AI生成コード向けカスタムルール追加（過剰な型変換、不要なnullチェック等のAI特有パターン） |
| SpotBugs | 標準 + 未公開APIチェッカー | AI生成コードに多い脆弱性パターン（ハードコードされた認証情報、非推奨暗号化ライブラリ使用等）を検出するカスタムディテクター追加 |
| OWASP Dependency-Check | 未導入 | 新規導入。AI生成コードが参照する依存ライブラリの脆弱性を自動スキャン |

**未公開APIチェッカーの拡張:**
- 現行のホワイトリスト設定を見直し、AI生成コードが誤って使用しがちな内部APIパターンを追加

### 2.3 Layer 2: セキュリティスキャン

**目的**: OWASP Top 10に基づくセキュリティ脆弱性の体系的検出

**実装方針:**

| カテゴリ | ツール | 検出対象 |
|---|---|---|
| SAST | SonarQube Community / Checkmarx | SQLインジェクション、XSS、コマンドインジェクション等 |
| SCA | OWASP Dependency-Check | 既知の脆弱性を持つOSSライブラリ |
| Secret Detection | gitleaks | ハードコードされたAPIキー、パスワード、トークン |
| SBOM生成 | CycloneDX Maven Plugin | ソフトウェア部品表の自動生成（EU CRA対応準備） |

**Nablarch固有の考慮事項:**
- Nablarchのハンドラキューアーキテクチャに特化した入力バリデーションパターンの検証
- `SystemRepository`経由のコンポーネント設定におけるXMLインジェクションリスクの検出
- Nablarch DBアクセスフレームワーク使用時のSQLインジェクション対策の自動検証

### 2.4 Layer 3: LLMベースコードレビュー

**目的**: 従来の静的解析では検出困難な、Nablarch設計思想・アーキテクチャパターンへの準拠を検証

**アーキテクチャ:**

```
┌──────────────────────────────────────────────┐
│            LLMコードレビューエンジン            │
│                                              │
│  ┌─────────────────────────────┐            │
│  │  Specification Store         │            │
│  │  ├─ Nablarch設計規約         │            │
│  │  ├─ セキュリティ要件          │            │
│  │  ├─ プロジェクト固有ルール    │            │
│  │  └─ AI生成コード品質基準      │            │
│  └──────────────┬──────────────┘            │
│                  │ Grounding                 │
│                  ▼                           │
│  ┌─────────────────────────────┐            │
│  │  LLM (Claude / GPT-4o)      │            │
│  │  + RAG (Vector DB)           │◄── PR Diff │
│  │  + LSP Integration           │            │
│  └──────────────┬──────────────┘            │
│                  │                           │
│                  ▼                           │
│  ┌─────────────────────────────┐            │
│  │  Review Results              │            │
│  │  ├─ 問題分類・重要度          │            │
│  │  ├─ 修正提案                 │            │
│  │  └─ Specification根拠         │            │
│  └─────────────────────────────┘            │
└──────────────────────────────────────────────┘
```

**SGCR（Specification-Grounded Code Review）方式の採用:**
- 研究論文に基づくアプローチ。人間が作成した仕様書（コーディング規約、セキュリティ要件、ドメイン固有ビジネスルール）をLLMの推論の根拠として明示的に供給
- 先行事例では、ベースラインLLMアプローチに比べてレビュー提案の採用率が90.9%向上（42%の採用率を達成）
- LLMの確率的な性質に起因する矛盾したフィードバックを抑制

**Nablarch固有のSpecification:**
- ハンドラキューの正しい構成パターン
- `SystemRepository`の適切な使用方法
- 業務アクション・フォーム・エンティティの設計パターン
- DB アクセスのベストプラクティス（Universal DAO / SQL Loader）
- バリデーションルールの正しい定義方法

**RAG構成:**
- ベクトルDB（Qdrant / Chroma）にNablarch公式ドキュメント、既存プロジェクトの高品質コード、設計書をインデクシング
- PRの差分コードに対して、関連する設計パターン・実装例を検索し、LLMのコンテキストに供給
- Language Server Protocol（LSP）連携による型情報・参照解析の活用

### 2.5 Layer 4: 自動テスト実行

**目的**: AI生成テストケースの実行によるランタイム品質の検証

（詳細はセクション3にて記述）

---

## 3. Excelテスト自動生成の実装計画

### 3.1 現状のExcelテストフレームワーク構造

```
テストクラス: FooActionRequestTest.java
テストデータ: FooActionRequestTest.xlsx
              ├── Sheet: testInsert    (SETUP_TABLE + EXPECTED_TABLE)
              ├── Sheet: testUpdate    (SETUP_TABLE + EXPECTED_TABLE)
              └── Sheet: testDelete    (SETUP_TABLE + EXPECTED_TABLE)
```

### 3.2 AI自動生成アーキテクチャ

```
┌─────────────────────────────────────────────────────┐
│              Excelテスト自動生成エンジン                │
│                                                     │
│  ┌───────────────────────────────────────────┐     │
│  │  Input: ソースコード解析                     │     │
│  │  ├─ Actionクラス（ビジネスロジック）          │     │
│  │  ├─ Formクラス（バリデーション定義）          │     │
│  │  ├─ Entityクラス（DB定義）                  │     │
│  │  ├─ SQLファイル（クエリ定義）               │     │
│  │  └─ コンポーネント定義XML                    │     │
│  └──────────────────┬────────────────────────┘     │
│                      │ AST解析 + LLM推論            │
│                      ▼                              │
│  ┌───────────────────────────────────────────┐     │
│  │  テストケース生成                            │     │
│  │  ├─ 正常系: 全パスのバリデーション成功         │     │
│  │  ├─ 異常系: バリデーションエラー各パターン     │     │
│  │  ├─ 境界値: 文字数上限・数値範囲等            │     │
│  │  └─ セキュリティ: インジェクション等           │     │
│  └──────────────────┬────────────────────────┘     │
│                      │                              │
│                      ▼                              │
│  ┌───────────────────────────────────────────┐     │
│  │  Output: Excel + Javaテストクラス            │     │
│  │  ├─ SETUP_TABLE生成                         │     │
│  │  ├─ EXPECTED_TABLE / EXPECTED_COMPLETE_TABLE │     │
│  │  ├─ LIST_MAP生成                            │     │
│  │  ├─ テストJavaクラス（JUnit4/5）             │     │
│  │  └─ テストデータの${systemTime}等の自動適用    │     │
│  └───────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────┘
```

### 3.3 実装方式

#### 3.3.1 Phase 1: テンプレートベース生成

- Actionクラスのメソッドシグネチャ・バリデーションアノテーションからExcelテンプレートを機械的に生成
- Form/Entityクラスのフィールド定義からSETUP_TABLE/EXPECTED_TABLEの列構造を自動生成
- SQLファイルからDB操作の期待結果テンプレートを生成
- **ツール**: Apache POI（Excel生成）+ JavaParser（AST解析）

#### 3.3.2 Phase 2: LLM支援テストケース生成

- Phase 1のテンプレートをベースに、LLMがビジネスロジックを読み取り、具体的なテストデータ値を生成
- バリデーション定義から異常系テストケースを網羅的に生成
- 境界値分析に基づくテストデータの自動生成
- **ツール**: Claude API / OpenAI API + RAG（既存テストパターン参照）

#### 3.3.3 Phase 3: インテリジェント生成

- 既存のExcelテストデータをRAGでインデクシングし、プロジェクト固有のテストパターンを学習
- コード変更差分から影響範囲を特定し、必要なテストケースの追加・更新を提案

### 3.4 Excelフォーマット生成の技術仕様

```java
// テスト自動生成エンジンのコア処理（概念設計）
public class NablarchTestGenerator {

    // Actionクラスを解析してExcelテストデータを生成
    public Workbook generateTestData(Class<?> actionClass) {
        // 1. Actionクラスのメソッド解析
        List<Method> testTargets = analyzeActionMethods(actionClass);

        // 2. 関連Form/Entity/SQLの解析
        FormAnalysis formAnalysis = analyzeFormValidation(actionClass);
        EntityAnalysis entityAnalysis = analyzeEntityDefinition(actionClass);

        // 3. Excelワークブック生成
        Workbook workbook = new XSSFWorkbook();
        for (Method method : testTargets) {
            Sheet sheet = workbook.createSheet(method.getName());
            // SETUP_TABLE生成
            generateSetupTable(sheet, entityAnalysis);
            // EXPECTED_TABLE生成
            generateExpectedTable(sheet, method, formAnalysis);
        }

        // 4. LLMによるテストデータ値の具体化
        enrichWithLLM(workbook, actionClass);

        return workbook;
    }
}
```

### 3.5 リクエスト単体テスト種別ごとの生成戦略

| テスト種別 | 入力解析対象 | 生成戦略 |
|---|---|---|
| Web（画面） | Form + HTMLテンプレート | リクエストパラメータ + 画面遷移パターン |
| REST | Form + OpenAPI定義 | JSON/XMLリクエストボディ + HTTPステータス |
| バッチ | 入力ファイル定義 + Entity | 入力データパターン + DB状態遷移 |
| メッセージ | メッセージ定義 + Entity | メッセージフォーマット + 応答パターン |

---

## 4. CI/CD統合の具体的設計

### 4.1 GitHub Actionsパイプライン設計

```yaml
# .github/workflows/nablarch-quality-gate.yml
name: Nablarch AI Code Quality Gate

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [develop]

jobs:
  # ═══════════════════════════════════════════
  # Layer 1: 静的解析
  # ═══════════════════════════════════════════
  static-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}

      # Checkstyle（Nablarchコーディング規約）
      - name: Run Checkstyle
        run: mvn checkstyle:check -Dcheckstyle.config.location=checkstyle/nablarch-checkstyle.xml

      # SpotBugs + 未公開APIチェッカー
      - name: Run SpotBugs with Nablarch Plugin
        run: mvn spotbugs:check -Dspotbugs.plugins=com.nablarch.framework:nablarch-unpublished-api-checker:1.0.0

      # SpotBugs結果のPR注釈
      - name: Annotate SpotBugs Results
        if: always()
        uses: jwgmeligmeyling/spotbugs-github-action@v1
        with:
          path: '**/spotbugsXml.xml'

  # ═══════════════════════════════════════════
  # Layer 2: セキュリティスキャン
  # ═══════════════════════════════════════════
  security-scan:
    runs-on: ubuntu-latest
    needs: static-analysis
    steps:
      - uses: actions/checkout@v4

      # OWASP Dependency-Check
      - name: OWASP Dependency Check
        run: mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7

      # Secret Detection
      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2

      # SBOM生成
      - name: Generate SBOM
        run: mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: target/bom.json

  # ═══════════════════════════════════════════
  # Layer 3: LLMベースコードレビュー
  # ═══════════════════════════════════════════
  llm-review:
    runs-on: ubuntu-latest
    needs: static-analysis
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # PRの差分取得
      - name: Get PR Diff
        id: diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD -- '*.java' > pr_diff.txt

      # Nablarch固有ルールでのLLMレビュー
      - name: LLM Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          python scripts/nablarch_llm_review.py \
            --diff pr_diff.txt \
            --specs specs/nablarch_coding_standards.yaml \
            --rag-index index/nablarch_patterns \
            --output review_results.json

      # レビュー結果をPRにコメント
      - name: Post Review Comments
        uses: actions/github-script@v7
        with:
          script: |
            const results = require('./review_results.json');
            // PRコメントとして投稿

  # ═══════════════════════════════════════════
  # Layer 4: 自動テスト
  # ═══════════════════════════════════════════
  test:
    runs-on: ubuntu-latest
    needs: [static-analysis, security-scan]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      # 単体テスト + Excelテスト実行
      - name: Run Tests
        run: mvn test

      # テストカバレッジ
      - name: JaCoCo Coverage Report
        run: mvn jacoco:report

      # カバレッジ閾値チェック
      - name: Check Coverage Threshold
        run: mvn jacoco:check -Djacoco.line.ratio=0.80

  # ═══════════════════════════════════════════
  # 品質ゲート判定
  # ═══════════════════════════════════════════
  quality-gate:
    runs-on: ubuntu-latest
    needs: [static-analysis, security-scan, llm-review, test]
    steps:
      - name: Quality Gate Decision
        run: |
          echo "All quality gates passed"
```

### 4.2 Jenkins統合設計

Jenkinsを使用する場合は、以下のパイプラインステージで構成する：

```groovy
pipeline {
    agent any

    stages {
        stage('Static Analysis') {
            parallel {
                stage('Checkstyle') {
                    steps { sh 'mvn checkstyle:check' }
                }
                stage('SpotBugs') {
                    steps { sh 'mvn spotbugs:check' }
                }
            }
        }
        stage('Security Scan') {
            steps { sh 'mvn org.owasp:dependency-check-maven:check' }
        }
        stage('LLM Review') {
            when { changeRequest() }
            steps { sh 'python scripts/nablarch_llm_review.py ...' }
        }
        stage('Test') {
            steps { sh 'mvn test jacoco:report jacoco:check' }
        }
    }

    post {
        always {
            recordIssues tools: [checkStyle(), spotBugs()]
            publishHTML target: [reportDir: 'target/site/jacoco', reportFiles: 'index.html']
        }
    }
}
```

### 4.3 品質ゲート閾値設定

| メトリクス | 閾値 | ゲートタイプ |
|---|---|---|
| Checkstyle違反数 | 0 | Hard（必須） |
| SpotBugs High/Medium | 0 | Hard（必須） |
| 未公開API使用 | 0 | Hard（必須） |
| OWASP Dependency-Check CVSS | < 7.0 | Hard（必須） |
| gitleaks検出 | 0 | Hard（必須） |
| LLMレビュー Critical | 0 | Soft（要確認） |
| テストカバレッジ（行） | >= 80% | Soft（要確認） |
| テスト成功率 | 100% | Hard（必須） |

### 4.4 レポーティング自動化

- **PRコメント**: 各ゲートの結果をGitHub PR上にサマリコメントとして自動投稿
- **ダッシュボード**: SonarQube / Grafana連携による品質メトリクスの可視化
- **SBOM**: CycloneDXフォーマットでのSBOM自動生成・アーカイブ
- **週次レポート**: AI生成コード率、品質ゲート通過率、検出パターン傾向のレポート

---

## 5. 段階的構築ロードマップ

### Phase 1: 基盤整備

**目標**: 既存ツールの最適化とCI/CDパイプライン構築

| 項目 | 内容 |
|---|---|
| 既存ツール整備 | Checkstyle / SpotBugs / 未公開APIチェッカーの設定最新化 |
| CI/CD構築 | GitHub Actions / Jenkins パイプラインの構築 |
| セキュリティ基盤 | OWASP Dependency-Check + gitleaks 導入 |
| SBOM | CycloneDX Maven Plugin 導入 |
| カバレッジ | JaCoCo導入・閾値設定 |

**成果物**:
- 静的解析 + セキュリティスキャン + テスト実行のCI/CDパイプライン
- 品質ゲート閾値の初期設定

### Phase 2: AI品質ゲート導入

**目標**: LLMベースコードレビューの構築と初期運用

| 項目 | 内容 |
|---|---|
| Specification整備 | Nablarchコーディング規約・設計規約のSpecification文書化 |
| RAG構築 | Nablarch公式ドキュメント + 既存高品質コードのベクトルDB化 |
| LLMレビューエンジン | SGCR方式のレビューエンジン構築 |
| CI/CD統合 | LLMレビューをPRワークフローに統合 |
| チューニング | プロンプト調整、誤検知率の最適化 |

**成果物**:
- LLMベースコードレビューのCI/CD統合
- Nablarch固有Specificationドキュメント

### Phase 3: Excelテスト自動生成

**目標**: テストケース自動生成エンジンの構築

| 項目 | 内容 |
|---|---|
| テンプレート生成 | Action/Form/Entity解析によるExcelテンプレート自動生成 |
| LLMテストデータ | LLMによるテストデータ値の自動生成 |
| テストクラス生成 | JUnit4/5テストクラスの自動生成 |

**成果物**:
- Excelテスト自動生成CLIツール
- テストカバレッジ向上

### Phase 4: 高度化・最適化

**目標**: 全体の精度向上と運用最適化

| 項目 | 内容 |
|---|---|
| AI-on-AI Review | 多モデル交差検証の導入 |
| IAST | ランタイム脆弱性検出の導入 |
| ダッシュボード | 品質メトリクス統合ダッシュボード構築 |
| 継続改善 | 誤検知率・検出精度のモニタリングと改善サイクル |

**成果物**:
- 統合品質ダッシュボード
- 運用改善レポート

---

## 6. 実現可能性の評価

### 6.1 技術的実現可能性

| 要素 | 評価 | 根拠 |
|---|---|---|
| 既存ツール拡張（Layer 1） | **高** | Checkstyle / SpotBugs は設定変更のみで対応可能。Maven統合済み |
| セキュリティスキャン（Layer 2） | **高** | OWASP Dependency-Check / gitleaks は成熟したOSSツール |
| LLMコードレビュー（Layer 3） | **中〜高** | SGCR方式の研究実績あり（42%採用率）。ただしNablarch固有Specificationの整備に工数が必要 |
| Excelテスト自動生成（Layer 4） | **中** | テンプレート生成は機械的に可能。LLMによるテストデータ値生成の精度がボトルネック |
| CI/CD統合 | **高** | GitHub Actions / Jenkinsとの統合パターンは確立されている |

### 6.2 組織的実現可能性

| 要素 | 評価 | 根拠 |
|---|---|---|
| TIS社内のNablarch知見 | **高** | 開発元であり、設計規約・ドキュメントが充実 |
| LLM活用の技術力 | **中** | 2026年時点でLLM活用は一般化しているが、Specificationの品質が成否を左右 |
| CI/CD運用体制 | **高** | 既存のMaven統合パターンが確立されている |
| プロジェクトへの適用 | **中** | 既存プロジェクトへの導入には段階的アプローチが必要 |

### 6.3 コスト評価

| 項目 | 概算 | 備考 |
|---|---|---|
| Phase 1 | 低 | OSSツールの設定・統合のみ |
| Phase 2 | 中 | LLM API利用料 + Specification整備工数 |
| Phase 3 | 中〜高 | テスト自動生成エンジンの開発工数 |
| Phase 4 | 低〜中 | Phase 2-3の改善・最適化 |
| LLM API運用コスト | 変動 | PR数・コード量に依存。先行研究では7,500件のレビューが$35未満 |

---

## 7. リスクと対策

### 7.1 技術リスク

| リスク | 影響度 | 対策 |
|---|---|---|
| LLMの誤検知（False Positive） | 高 | Layer 3はSoftゲート（ブロックではなく要確認）として運用。SGCR方式で誤検知を抑制 |
| LLMのハルシネーション | 高 | Specification-Groundedアプローチで根拠を明示。人間レビューとの併用 |
| Excelテスト生成の精度不足 | 中 | Phase 1でテンプレート生成から開始し、段階的にLLM生成を追加 |
| Nablarchバージョンアップ時のSpecification更新 | 中 | バージョン更新時のSpecification自動更新パイプライン構築 |
| LLM APIの可用性・レイテンシ | 中 | ローカルLLM（Llama等）のフォールバック構成。キャッシュ層導入 |

### 7.2 セキュリティリスク

| リスク | 影響度 | 対策 |
|---|---|---|
| LLM APIへのソースコード送信 | 高 | オンプレミスLLMの採用検討。ゼロリテンションポリシーのプロバイダー選定。Secret Detectionの前処理で機密情報を除去 |
| AI生成コード自体の脆弱性 | 高 | 多層ゲートで検出。OWASP Top 10ベースのスキャン必須化 |
| サプライチェーン攻撃 | 高 | SBOM生成・監視。OWASP Dependency-Checkの定期実行 |

### 7.3 運用リスク

| リスク | 影響度 | 対策 |
|---|---|---|
| 品質ゲートによる開発速度低下 | 中 | Layer 1-2は並列実行。LLMレビューは非同期。閾値の段階的厳格化 |
| チーム学習コスト | 中 | ドキュメント整備。品質ゲート違反時の自動修正提案 |
| Specificationの陳腐化 | 中 | 定期的なSpecification見直しサイクル（四半期毎） |
| ツール間の設定不整合 | 低 | 設定の一元管理（Maven POM / CI設定の標準化） |

---

## 参考文献・情報源

- [Nablarch公式ドキュメント](https://nablarch.github.io/)
- [Nablarch GitHub Organization](https://github.com/nablarch)
- [nablarch-unpublished-api-checker](https://github.com/nablarch/nablarch-unpublished-api-checker)
- [Nablarch Style Guide](https://github.com/nablarch-development-standards/nablarch-style-guide)
- [Fintan-contents/coding-standards](https://github.com/Fintan-contents/coding-standards)（移行先）
- [Nablarch 自動テストフレームワーク](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/06_TestFWGuide/01_Abstract.html)
- [リクエスト単体データ作成ツール](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/08_TestTools/01_HttpDumpTool/01_HttpDumpTool.html)
- [Nablarch Application Framework（Fintan）](https://fintan.jp/en/page/1660/)
- [SpotBugs Maven Plugin](https://github.com/spotbugs/spotbugs-maven-plugin)
- [SpotBugs GitHub Action](https://github.com/jwgmeligmeyling/spotbugs-github-action)
- [SGCR: Specification-Grounded Framework for Code Review](https://arxiv.org/html/2512.17540)
- [LLM Code Review Service with RAG](https://medium.com/@random.droid/llm-code-review-service-with-rag-observability-93e6befa6330)
- [OWASP AI Testing Guide 2026](https://www.practical-devsecops.com/owasp-ai-testing-guide-explained/)
- [DevSecOps Trends 2026](https://debuglies.com/2026/01/07/devsecops-trends-2026-ai-agents-revolutionizing-secure-software-development/)
- [AI Code Review Tools 2026（Qodo）](https://www.qodo.ai/blog/best-ai-code-review-tools-2026/)
- [Ericsson LLM Code Review Experience Report](https://arxiv.org/html/2507.19115)
