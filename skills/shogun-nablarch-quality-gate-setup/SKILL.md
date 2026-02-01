---
name: nablarch-quality-gate-setup
description: Nablarchプロジェクト向けの品質ゲートCI/CDパイプラインを自動セットアップする。Checkstyle、SpotBugs（未公開APIチェッカー含む）、OWASP Dependency-Check、gitleaks、JaCoCo等の設定ファイル生成と、GitHub Actions/Jenkinsパイプライン構築を行う。「NablarchプロジェクトにCI/CDを構築したい」「品質ゲートを設定したい」「Nablarchの静的解析を導入したい」時に使用。
---

# Nablarch Quality Gate Setup

## Overview

NablarchフレームワークベースのJavaプロジェクトに対して、多層品質ゲートCI/CDパイプラインをセットアップするスキル。以下の4層で構成される品質検証を、コピー&ペースト可能な設定ファイル群として提供する。

```
Layer 1: 静的解析 ─── Checkstyle / SpotBugs / 未公開APIチェッカー / OWASP Dep-Check
    │
Layer 2: セキュリティスキャン ─── gitleaks / CycloneDX SBOM
    │
Layer 3: LLMベースコードレビュー（オプション） ─── SGCR方式
    │
Layer 4: テスト実行 ─── JUnit + Excelテスト / JaCoCo カバレッジ
    │
    ▼
品質ゲート判定（Hard Gate / Soft Gate）
```

## When to Use

以下の状況でこのスキルを使用せよ:

- 新規Nablarchプロジェクトにci/CDパイプラインを構築する場合
- 既存NablarchプロジェクトにCheckstyle/SpotBugsを導入・更新する場合
- Nablarchの未公開APIチェッカーをCI/CDに統合する場合
- OWASP準拠のセキュリティスキャンを導入する場合
- 品質ゲートの閾値を設定・調整する場合
- GitHub Actions または Jenkins で品質パイプラインを構築する場合
- Xenlon（神龍）で変換されたコードの品質保証体制を構築する場合
- AI生成コード（Copilot / Claude / Cursor等）の品質ゲートを設けたい場合

## Instructions

### 前提条件

- Java 11以上（Java 17推奨）
- Maven 3.6.3以上
- Nablarchフレームワーク 5u8以上

### Step 1: プロジェクト構成の確認

対象プロジェクトの構成を確認し、以下の情報を把握する:

```bash
# pom.xml の存在確認
ls pom.xml

# Nablarchのバージョン確認
grep -A2 'nablarch' pom.xml | head -10

# 既存の静的解析設定の有無
ls checkstyle/ spotbugs/ 2>/dev/null

# テストディレクトリ構成
find src/test -name "*.xlsx" -o -name "*.xls" | head -10
```

---

### Layer 1: 静的解析セットアップ

#### 1-1. Checkstyle設定（Nablarchコーディング規約）

Nablarchスタイルガイドに基づくCheckstyle設定を導入する。

**ディレクトリ作成:**
```bash
mkdir -p checkstyle
```

**pom.xml に追加するプラグイン:**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <configLocation>checkstyle/nablarch-checkstyle.xml</configLocation>
        <consoleOutput>true</consoleOutput>
        <failsOnError>true</failsOnError>
        <violationSeverity>warning</violationSeverity>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>10.12.7</version>
        </dependency>
    </dependencies>
</plugin>
```

**Nablarchコーディング規約の要点（チェック対象）:**

| ルール | 説明 |
|---|---|
| `static final` 命名 | 大文字・アンダースコア・数字のみ |
| 空catch禁止 | catchブロックにコメント必須 |
| static import禁止 | `ClassName.MemberName` 表記を使用 |
| 汎用例外catch禁止 | `Exception`, `Throwable`, `RuntimeException`, `Error` のcatch禁止 |
| リフレクション直接使用禁止 | アーキテクト承認が必要 |
| レガシーAPI使用禁止 | 非推奨Java標準ライブラリの使用を検出 |

Checkstyle設定ファイルはNablarch公式リポジトリから取得可能:
- 取得元: https://github.com/Fintan-contents/coding-standards
- 旧リポジトリ: https://github.com/nablarch-development-standards/nablarch-style-guide

#### 1-2. SpotBugs + 未公開APIチェッカー設定

**ディレクトリ作成:**
```bash
mkdir -p spotbugs/published-config/production
```

**pom.xml に追加するプラグイン:**
```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.3.1</version>
    <configuration>
        <effort>Max</effort>
        <threshold>Medium</threshold>
        <xmlOutput>true</xmlOutput>
        <excludeFilterFile>spotbugs/spotbugs_exclude.xml</excludeFilterFile>
        <jvmArgs>-Dnablarch-findbugs-config=spotbugs/published-config/production</jvmArgs>
        <plugins>
            <plugin>
                <groupId>com.nablarch.framework</groupId>
                <artifactId>nablarch-unpublished-api-checker</artifactId>
                <version>1.0.0</version>
            </plugin>
        </plugins>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs</artifactId>
            <version>4.8.3</version>
        </dependency>
    </dependencies>
</plugin>
```

**SpotBugs除外フィルタ（spotbugs/spotbugs_exclude.xml）:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<FindBugsFilter>
    <!-- 自動生成コードを除外する場合 -->
    <Match>
        <Source name="~.*Generated.*\.java"/>
    </Match>
    <!-- テストコードの一部ルール緩和 -->
    <Match>
        <Class name="~.*Test"/>
        <Bug pattern="UPU_UNPUBLISHED_API_USAGE"/>
    </Match>
</FindBugsFilter>
```

**未公開APIチェッカーの仕組み:**
- バグパターン: `UPU_UNPUBLISHED_API_USAGE`
- ホワイトリスト方式: `spotbugs/published-config/production` ディレクトリ内の設定ファイルで許可APIを定義
- 役割別設定:
  - **プログラマー向け**: ビジネス機能実装に必要なNAF API + テスト用NTF API
  - **アーキテクト向け**: NAF拡張機能用API + NTF拡張用API

許可API設定ファイルはNablarch公式リポジトリから取得:
```bash
# Nablarch公式の許可API設定を取得
git clone --depth 1 https://github.com/nablarch/nablarch-unpublished-api-checker.git /tmp/nablarch-api-checker
cp -r /tmp/nablarch-api-checker/config/* spotbugs/published-config/production/
```

#### 1-3. OWASP Dependency-Check導入

**pom.xml に追加するプラグイン:**
```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.3</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
        <formats>
            <format>HTML</format>
            <format>JSON</format>
        </formats>
        <outputDirectory>${project.build.directory}/dependency-check</outputDirectory>
        <!-- NVD API Keyの設定（推奨） -->
        <nvdApiKey>${env.NVD_API_KEY}</nvdApiKey>
    </configuration>
</plugin>
```

**実行コマンド:**
```bash
# 脆弱性チェック実行
mvn org.owasp:dependency-check-maven:check

# CVSS 7以上でビルド失敗
mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7
```

---

### Layer 2: セキュリティスキャン

#### 2-1. gitleaks（Secret Detection）

**ローカル実行:**
```bash
# インストール
brew install gitleaks  # macOS
# または
go install github.com/gitleaks/gitleaks/v8@latest

# スキャン実行
gitleaks detect --source . --report-path gitleaks-report.json
```

**gitleaks設定ファイル（.gitleaks.toml）:**
```toml
title = "Nablarch Project Gitleaks Config"

# Nablarch固有の除外パターン
[allowlist]
  description = "Nablarch specific allowlist"
  paths = [
    '''spotbugs/published-config/.*''',
    '''checkstyle/.*\.xml''',
    '''src/test/.*'''
  ]
```

#### 2-2. SBOM生成（CycloneDX）

**pom.xml に追加するプラグイン:**
```xml
<plugin>
    <groupId>org.cyclonedx</groupId>
    <artifactId>cyclonedx-maven-plugin</artifactId>
    <version>2.8.0</version>
    <configuration>
        <projectType>application</projectType>
        <schemaVersion>1.5</schemaVersion>
        <includeBomSerialNumber>true</includeBomSerialNumber>
        <includeCompileScope>true</includeCompileScope>
        <includeRuntimeScope>true</includeRuntimeScope>
        <outputFormat>json</outputFormat>
        <outputName>bom</outputName>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>makeBom</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**実行コマンド:**
```bash
mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom
# 出力: target/bom.json
```

---

### Layer 3: LLMベースコードレビュー（オプション）

このレイヤーは段階的に導入する。即座の導入は不要。

#### 3-1. SGCR（Specification-Grounded Code Review）方式の概要

SGCR方式は、LLMの推論をNablarch固有の仕様書（Specification）で根拠付けるアプローチ。

**Specificationに含めるべきNablarch固有ルール:**
- ハンドラキューの正しい構成パターン
- `SystemRepository`の適切な使用方法
- 業務アクション・フォーム・エンティティの設計パターン
- DBアクセスのベストプラクティス（Universal DAO / SQL Loader）
- バリデーションルールの正しい定義方法

**Specification文書の構造例（specs/nablarch_coding_standards.yaml）:**
```yaml
specifications:
  - id: NAB-001
    category: architecture
    rule: "ハンドラキューは公式ドキュメントのパターンに従うこと"
    severity: critical
    reference: "https://nablarch.github.io/docs/LATEST/doc/application_framework/"

  - id: NAB-002
    category: database
    rule: "DBアクセスはUniversal DAOまたはSQL Loaderを使用すること。直接JDBCは禁止"
    severity: critical

  - id: NAB-003
    category: validation
    rule: "入力バリデーションはNablarchのバリデーションフレームワークを使用すること"
    severity: high

  - id: NAB-004
    category: security
    rule: "CSRF対策にはNablarch標準のトークン機能を使用すること"
    severity: critical

  - id: NAB-005
    category: coding
    rule: "未公開APIは使用禁止。SpotBugs未公開APIチェッカーで検出されるAPIは使用しないこと"
    severity: critical
```

#### 3-2. CI/CD統合時の設定

PR時にLLMレビューを実行する場合:
```bash
# PRの差分を取得してLLMに送信
git diff origin/main...HEAD -- '*.java' > pr_diff.txt

# LLMレビュー実行（自前スクリプトを用意）
python scripts/nablarch_llm_review.py \
    --diff pr_diff.txt \
    --specs specs/nablarch_coding_standards.yaml \
    --output review_results.json
```

---

### Layer 4: テスト実行

#### 4-1. JaCoCo設定

**pom.xml に追加するプラグイン:**
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <!-- テスト前にエージェント準備 -->
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <!-- テスト後にレポート生成 -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <!-- カバレッジ閾値チェック -->
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.70</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### 4-2. NablarchのExcelテスト実行設定

NablarchのExcelテストはJUnit経由で実行される。特別な設定は不要だが、以下を確認:

```xml
<!-- Nablarch Testing Framework 依存 -->
<dependency>
    <groupId>com.nablarch.framework</groupId>
    <artifactId>nablarch-testing</artifactId>
    <scope>test</scope>
</dependency>

<!-- JUnit 5で実行する場合 -->
<dependency>
    <groupId>com.nablarch.framework</groupId>
    <artifactId>nablarch-testing-junit5</artifactId>
    <scope>test</scope>
</dependency>

<!-- JUnit 4テストをJUnit 5で実行（Vintage） -->
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
</dependency>
```

**テストデータ構造の確認:**
```
src/test/java/com/example/action/FooActionRequestTest.java
src/test/java/com/example/action/FooActionRequestTest.xlsx  ← 同名・同ディレクトリ
```

**Excelテストデータのデータタイプ:**

| データタイプ | 用途 |
|---|---|
| `SETUP_TABLE` | テスト前のDB準備データ |
| `EXPECTED_TABLE` | 期待結果（省略カラムは比較対象外） |
| `EXPECTED_COMPLETE_TABLE` | 期待結果（省略カラムはデフォルト値） |
| `LIST_MAP` | `List<Map<String,String>>`形式データ |

---

### CI/CD統合

#### GitHub Actionsワークフロー

以下のYAMLファイルを `.github/workflows/nablarch-quality-gate.yml` として保存する。

```yaml
name: Nablarch AI Code Quality Gate

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [develop]

env:
  JAVA_VERSION: '17'
  JAVA_DISTRIBUTION: 'temurin'

jobs:
  # ═══════════════════════════════════════════════════════
  # Layer 1: 静的解析
  # ═══════════════════════════════════════════════════════
  static-analysis:
    name: "Layer 1: Static Analysis"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: ${{ env.JAVA_DISTRIBUTION }}

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      # Checkstyle（Nablarchコーディング規約）
      - name: Run Checkstyle
        run: mvn checkstyle:check -B

      # SpotBugs + 未公開APIチェッカー
      - name: Run SpotBugs with Nablarch Plugin
        run: mvn spotbugs:check -B

      # SpotBugs結果をPR注釈として表示
      - name: Annotate SpotBugs Results
        if: always()
        uses: jwgmeligmeyling/spotbugs-github-action@v1
        with:
          path: '**/spotbugsXml.xml'

  # ═══════════════════════════════════════════════════════
  # Layer 2: セキュリティスキャン
  # ═══════════════════════════════════════════════════════
  security-scan:
    name: "Layer 2: Security Scan"
    runs-on: ubuntu-latest
    needs: static-analysis
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: ${{ env.JAVA_DISTRIBUTION }}

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      # OWASP Dependency-Check
      - name: OWASP Dependency Check
        run: mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7 -B
        env:
          NVD_API_KEY: ${{ secrets.NVD_API_KEY }}

      # Secret Detection
      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # SBOM生成
      - name: Generate SBOM (CycloneDX)
        run: mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom -B

      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: target/bom.json
          retention-days: 90

      # Dependency-Checkレポートのアップロード
      - name: Upload Dependency-Check Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: target/dependency-check/dependency-check-report.html
          retention-days: 30

  # ═══════════════════════════════════════════════════════
  # Layer 3: LLMベースコードレビュー（オプション）
  # ═══════════════════════════════════════════════════════
  llm-review:
    name: "Layer 3: LLM Code Review (Optional)"
    runs-on: ubuntu-latest
    needs: static-analysis
    if: github.event_name == 'pull_request'
    continue-on-error: true  # Soft Gate: 失敗してもパイプラインは継続
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # PRの差分をJavaファイルに限定して取得
      - name: Get PR Diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD -- '*.java' > pr_diff.txt
          echo "Diff size: $(wc -l < pr_diff.txt) lines"

      # LLMレビュー実行
      # 注意: scripts/nablarch_llm_review.py は別途実装が必要
      - name: LLM Code Review
        if: hashFiles('scripts/nablarch_llm_review.py') != ''
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          python scripts/nablarch_llm_review.py \
            --diff pr_diff.txt \
            --specs specs/nablarch_coding_standards.yaml \
            --output review_results.json

      # レビュー結果をPRにコメント
      - name: Post Review Comments
        if: hashFiles('review_results.json') != ''
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            if (fs.existsSync('review_results.json')) {
              const results = JSON.parse(fs.readFileSync('review_results.json', 'utf8'));
              const body = results.summary || 'LLM Review completed. No critical issues found.';
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                body: `## LLM Code Review Results\n\n${body}`
              });
            }

  # ═══════════════════════════════════════════════════════
  # Layer 4: テスト実行
  # ═══════════════════════════════════════════════════════
  test:
    name: "Layer 4: Test & Coverage"
    runs-on: ubuntu-latest
    needs: static-analysis
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: ${{ env.JAVA_DISTRIBUTION }}

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      # 単体テスト + Excelテスト実行
      - name: Run Tests
        run: mvn test -B

      # JaCoCoカバレッジレポート生成
      - name: JaCoCo Coverage Report
        run: mvn jacoco:report -B

      # カバレッジ閾値チェック（Soft Gate）
      - name: Check Coverage Threshold
        run: mvn jacoco:check -B
        continue-on-error: true  # Soft Gate

      # カバレッジレポートのアップロード
      - name: Upload Coverage Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-report
          path: target/site/jacoco/
          retention-days: 30

  # ═══════════════════════════════════════════════════════
  # 品質ゲート最終判定
  # ═══════════════════════════════════════════════════════
  quality-gate:
    name: "Quality Gate Decision"
    runs-on: ubuntu-latest
    needs: [static-analysis, security-scan, test]
    # llm-review は continue-on-error のため needs に含めない
    if: always()
    steps:
      - name: Check Quality Gate Results
        run: |
          echo "=== Quality Gate Results ==="
          echo "Static Analysis: ${{ needs.static-analysis.result }}"
          echo "Security Scan:   ${{ needs.security-scan.result }}"
          echo "Test & Coverage: ${{ needs.test.result }}"
          echo "============================"

          if [[ "${{ needs.static-analysis.result }}" != "success" ]]; then
            echo "FAILED: Static analysis did not pass"
            exit 1
          fi
          if [[ "${{ needs.security-scan.result }}" != "success" ]]; then
            echo "FAILED: Security scan did not pass"
            exit 1
          fi
          if [[ "${{ needs.test.result }}" != "success" ]]; then
            echo "FAILED: Tests did not pass"
            exit 1
          fi

          echo "All quality gates PASSED"
```

#### Jenkinsパイプライン

以下を `Jenkinsfile` として保存する。

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    environment {
        NVD_API_KEY = credentials('nvd-api-key')
    }

    stages {
        // ═══════════════════════════════════════════
        // Layer 1: 静的解析
        // ═══════════════════════════════════════════
        stage('Layer 1: Static Analysis') {
            parallel {
                stage('Checkstyle') {
                    steps {
                        sh 'mvn checkstyle:check -B'
                    }
                    post {
                        always {
                            recordIssues tools: [checkStyle(pattern: '**/checkstyle-result.xml')]
                        }
                    }
                }
                stage('SpotBugs + Nablarch API Checker') {
                    steps {
                        sh 'mvn spotbugs:check -B'
                    }
                    post {
                        always {
                            recordIssues tools: [spotBugs(pattern: '**/spotbugsXml.xml')]
                        }
                    }
                }
            }
        }

        // ═══════════════════════════════════════════
        // Layer 2: セキュリティスキャン
        // ═══════════════════════════════════════════
        stage('Layer 2: Security Scan') {
            parallel {
                stage('OWASP Dependency-Check') {
                    steps {
                        sh 'mvn org.owasp:dependency-check-maven:check -DfailBuildOnCVSS=7 -B'
                    }
                    post {
                        always {
                            publishHTML target: [
                                reportDir: 'target/dependency-check',
                                reportFiles: 'dependency-check-report.html',
                                reportName: 'OWASP Dependency-Check'
                            ]
                        }
                    }
                }
                stage('SBOM Generation') {
                    steps {
                        sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom -B'
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'target/bom.json', fingerprint: true
                        }
                    }
                }
            }
        }

        // ═══════════════════════════════════════════
        // Layer 3: LLMベースコードレビュー（オプション）
        // ═══════════════════════════════════════════
        stage('Layer 3: LLM Review') {
            when {
                changeRequest()
            }
            steps {
                script {
                    try {
                        sh '''
                            git diff origin/${CHANGE_TARGET}...HEAD -- "*.java" > pr_diff.txt
                            if [ -f scripts/nablarch_llm_review.py ]; then
                                python scripts/nablarch_llm_review.py \
                                    --diff pr_diff.txt \
                                    --specs specs/nablarch_coding_standards.yaml \
                                    --output review_results.json
                            else
                                echo "LLM review script not found, skipping"
                            fi
                        '''
                    } catch (Exception e) {
                        echo "LLM Review failed (Soft Gate): ${e.message}"
                        // Soft Gate: 失敗してもパイプライン継続
                    }
                }
            }
        }

        // ═══════════════════════════════════════════
        // Layer 4: テスト実行
        // ═══════════════════════════════════════════
        stage('Layer 4: Test & Coverage') {
            steps {
                sh 'mvn test jacoco:report -B'
            }
            post {
                always {
                    junit '**/surefire-reports/*.xml'
                    publishHTML target: [
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Coverage Report'
                    ]
                }
            }
        }

        // ═══════════════════════════════════════════
        // カバレッジ閾値チェック（Soft Gate）
        // ═══════════════════════════════════════════
        stage('Coverage Threshold Check') {
            steps {
                script {
                    try {
                        sh 'mvn jacoco:check -B'
                    } catch (Exception e) {
                        echo "Coverage threshold not met (Soft Gate): ${e.message}"
                        unstable('Coverage below threshold')
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'All quality gates PASSED'
        }
        failure {
            echo 'Quality gate FAILED'
        }
    }
}
```

---

### 品質ゲート閾値設定

#### Hard Gate（必須: 違反でビルド失敗）

| メトリクス | 閾値 | ツール |
|---|---|---|
| Checkstyle違反数 | 0 | maven-checkstyle-plugin |
| SpotBugs High/Medium | 0 | spotbugs-maven-plugin |
| 未公開API使用数 | 0 | nablarch-unpublished-api-checker |
| OWASP CVSS スコア | < 7.0 | dependency-check-maven |
| gitleaks検出数 | 0 | gitleaks-action |
| テスト成功率 | 100% | maven-surefire-plugin |

#### Soft Gate（推奨: 違反は警告のみ）

| メトリクス | 閾値 | ツール | 備考 |
|---|---|---|---|
| LLMレビュー Critical | 0 | nablarch_llm_review.py | 段階的に導入 |
| 行カバレッジ | >= 80% | jacoco-maven-plugin | プロジェクトに応じて調整可 |
| ブランチカバレッジ | >= 70% | jacoco-maven-plugin | プロジェクトに応じて調整可 |

## Examples

### Example 1: pom.xml に追加すべきプラグイン設定（完全版）

以下を `pom.xml` の `<build><plugins>` セクションに追加する。

```xml
<build>
    <plugins>
        <!-- ═══ Layer 1: 静的解析 ═══ -->

        <!-- Checkstyle -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.3.1</version>
            <configuration>
                <configLocation>checkstyle/nablarch-checkstyle.xml</configLocation>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
                <violationSeverity>warning</violationSeverity>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>com.puppycrawl.tools</groupId>
                    <artifactId>checkstyle</artifactId>
                    <version>10.12.7</version>
                </dependency>
            </dependencies>
        </plugin>

        <!-- SpotBugs + Nablarch未公開APIチェッカー -->
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.8.3.1</version>
            <configuration>
                <effort>Max</effort>
                <threshold>Medium</threshold>
                <xmlOutput>true</xmlOutput>
                <excludeFilterFile>spotbugs/spotbugs_exclude.xml</excludeFilterFile>
                <jvmArgs>-Dnablarch-findbugs-config=spotbugs/published-config/production</jvmArgs>
                <plugins>
                    <plugin>
                        <groupId>com.nablarch.framework</groupId>
                        <artifactId>nablarch-unpublished-api-checker</artifactId>
                        <version>1.0.0</version>
                    </plugin>
                </plugins>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>com.github.spotbugs</groupId>
                    <artifactId>spotbugs</artifactId>
                    <version>4.8.3</version>
                </dependency>
            </dependencies>
        </plugin>

        <!-- ═══ Layer 2: セキュリティ ═══ -->

        <!-- OWASP Dependency-Check -->
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>10.0.3</version>
            <configuration>
                <failBuildOnCVSS>7</failBuildOnCVSS>
                <formats>
                    <format>HTML</format>
                    <format>JSON</format>
                </formats>
                <outputDirectory>${project.build.directory}/dependency-check</outputDirectory>
                <nvdApiKey>${env.NVD_API_KEY}</nvdApiKey>
            </configuration>
        </plugin>

        <!-- CycloneDX SBOM生成 -->
        <plugin>
            <groupId>org.cyclonedx</groupId>
            <artifactId>cyclonedx-maven-plugin</artifactId>
            <version>2.8.0</version>
            <configuration>
                <projectType>application</projectType>
                <schemaVersion>1.5</schemaVersion>
                <includeBomSerialNumber>true</includeBomSerialNumber>
                <includeCompileScope>true</includeCompileScope>
                <includeRuntimeScope>true</includeRuntimeScope>
                <outputFormat>json</outputFormat>
                <outputName>bom</outputName>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>makeBom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- ═══ Layer 4: テスト・カバレッジ ═══ -->

        <!-- JaCoCo -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.12</version>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
                <execution>
                    <id>check</id>
                    <goals>
                        <goal>check</goal>
                    </goals>
                    <configuration>
                        <rules>
                            <rule>
                                <element>BUNDLE</element>
                                <limits>
                                    <limit>
                                        <counter>LINE</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.80</minimum>
                                    </limit>
                                    <limit>
                                        <counter>BRANCH</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.70</minimum>
                                    </limit>
                                </limits>
                            </rule>
                        </rules>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Example 2: セットアップ後のプロジェクト構成

```
project-root/
├── .github/
│   └── workflows/
│       └── nablarch-quality-gate.yml    ← GitHub Actions
├── .gitleaks.toml                       ← gitleaks設定
├── Jenkinsfile                          ← Jenkins（使用する場合）
├── checkstyle/
│   └── nablarch-checkstyle.xml          ← Checkstyle設定
├── spotbugs/
│   ├── spotbugs_exclude.xml             ← SpotBugs除外フィルタ
│   └── published-config/
│       └── production/                  ← 未公開APIチェッカー許可設定
├── specs/
│   └── nablarch_coding_standards.yaml   ← LLMレビュー用Specification（オプション）
├── pom.xml                              ← プラグイン設定追加済み
└── src/
    ├── main/java/...
    └── test/
        └── java/
            └── com/example/action/
                ├── FooActionRequestTest.java
                └── FooActionRequestTest.xlsx  ← Excelテストデータ
```

### Example 3: ローカルでの品質チェック実行

```bash
# 全レイヤーを順次実行
mvn checkstyle:check spotbugs:check test jacoco:report -B

# Layer 1のみ（静的解析）
mvn checkstyle:check spotbugs:check -B

# Layer 2のみ（セキュリティ）
mvn org.owasp:dependency-check-maven:check -B
gitleaks detect --source .

# Layer 4のみ（テスト＋カバレッジ）
mvn test jacoco:report jacoco:check -B

# SBOM生成
mvn org.cyclonedx:cyclonedx-maven-plugin:makeBom -B
```

## Guidelines

### 必須ルール

1. **Hard Gateは全て通過させること**
   - Checkstyle違反0、SpotBugs High/Medium 0、未公開API使用0は絶対条件
   - これらが通らないPRはマージ不可

2. **セキュリティツールの導入順序**
   - まず OWASP Dependency-Check（既知脆弱性の依存ライブラリ検出）
   - 次に gitleaks（ハードコードされた秘密情報の検出）
   - 最後に CycloneDX SBOM生成（部品表の自動化）

3. **Soft Gateは段階的に導入**
   - LLMレビュー（Layer 3）は最初から `continue-on-error: true` で運用
   - カバレッジ閾値は初期は60%から開始し、段階的に80%へ引き上げ

4. **未公開APIチェッカーの設定は必ずカスタマイズ**
   - プロジェクトのNablarchバージョンに合った許可API設定を使用
   - `spotbugs/published-config/production/` 内の設定ファイルをプロジェクト要件に合わせて調整

### Xenlon変換コード向けの追加考慮事項

Xenlon（神龍）Migratorで COBOL/PL/I から Java に変換されたコードには、以下の追加対応が必要:

1. **SpotBugs除外フィルタの拡張**
   - Xenlon変換コードに特有の非Java的パターン（COBOL由来の命名規則等）をフィルタに追加
   - 変換ツールが生成するボイラープレートコードの除外設定

2. **Checkstyleルールの緩和検討**
   - 変換コードは自動生成のため、命名規約の一部を緩和する場合がある
   - ただしセキュリティ関連ルールは緩和不可

3. **未公開APIチェッカーの許可設定拡張**
   - Xenlon変換後のコードが使用するNablarch APIパターンを許可リストに追加
   - 変換元言語固有のライブラリラッパーに対応する許可設定

4. **テストカバレッジの閾値調整**
   - 変換初期はカバレッジが低い場合があるため、閾値を段階的に引き上げ
   - 変換元のCOBOLテストケースからの自動マッピングを検討

### NVD API Keyの取得

OWASP Dependency-Checkの高速実行にはNVD API Keyが必要:
1. https://nvd.nist.gov/developers/request-an-api-key でアカウント作成
2. 取得したキーをCI/CDのSecretに設定:
   - GitHub Actions: `Settings > Secrets > NVD_API_KEY`
   - Jenkins: `Credentials > nvd-api-key`

### プロジェクト固有のカスタマイズ

このスキルの設定はNablarchプロジェクトの標準構成に基づく。以下はプロジェクトに応じて調整すること:

- `checkstyle/nablarch-checkstyle.xml`: プロジェクト固有のルール追加・除外
- `spotbugs/spotbugs_exclude.xml`: プロジェクト固有の除外パターン
- `failBuildOnCVSS`: CVSS閾値（デフォルト7、厳格化する場合は4に下げる）
- JaCoCoの `minimum`: カバレッジ閾値（プロジェクトの成熟度に応じて調整）
