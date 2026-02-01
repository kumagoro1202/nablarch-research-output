---
name: project-scaffold
description: 任意の言語・フレームワークのプロジェクトスケルトンを自動生成し、GitHubリポジトリ作成からサブモジュール追加まで一気通貫で実行する。「新しいPythonプロジェクトを作って」「Javaプロジェクトのスケルトンを生成して」「GitHubリポジトリを作ってプロジェクト雛形を配置して」「新規プロジェクトのセットアップをして」「Spring Bootプロジェクトを作成して」「pyproject.toml + src layoutで初期化して」「gh repo createからサブモジュール追加まで一括で」といった要望に対応する。gh CLI + 言語別テンプレート + CI/CD生成の統合プロジェクト初期化スキル。
---

# Project Scaffold — プロジェクト雛形自動生成

## Overview

任意の言語・フレームワークのプロジェクトを、GitHubリポジトリ作成からプロジェクト構造生成、CI/CD設定、サブモジュール追加まで一気通貫で自動生成する統合スキル。

**統合元スキル:**
- S-001 python-project-scaffold: Python (pyproject.toml + src layout + テストスタブ)
- S-002 java-project-skeleton-creator: Java (Maven/Gradle + Spring Boot + MCP SDK等)
- S-003 github-repo-scaffold: GitHub リポジトリ作成→スケルトン→サブモジュール追加

**主な用途:**
- 新規プロジェクトの迅速な立ち上げ
- 組織標準に準拠したプロジェクト構造の強制
- CI/CD パイプラインの初期設定
- モノレポ内のサブモジュールとしてのプロジェクト追加
- マルチプロジェクト環境での統一的な初期化

**対応言語/フレームワーク:**
| 言語 | フレームワーク/構成 | ビルドツール |
|------|-------------------|------------|
| Python | 標準 (src layout) | pyproject.toml (setuptools/hatch/poetry) |
| Java | Spring Boot, Nablarch, Plain | Maven, Gradle |
| TypeScript | Node.js, Next.js | npm, pnpm |
| Go | 標準 | go mod |
| Rust | 標準 | Cargo |

**拡張構造:** 新しい言語/フレームワークの追加は、Phase 3のテンプレートセクションに追記するだけで対応可能。

## When to Use

以下のいずれかに該当する場合にこのスキルを使用する：

- 「新しいプロジェクトを作って」
- 「○○(言語)のプロジェクトを初期化して」
- 「GitHubリポジトリを作成してスケルトンを配置して」
- 「Spring Boot / Nablarch / Python のプロジェクト雛形を生成して」
- 「pyproject.toml + src layout でPythonプロジェクトをセットアップして」
- 「Maven/Gradleプロジェクトのスケルトンを作って」
- 「サブモジュールとして新プロジェクトを追加して」
- 「CI/CD付きのプロジェクトテンプレートを生成して」
- 新規開発プロジェクトの初期セットアップが必要な場合

**トリガーキーワード**: プロジェクト作成, スケルトン生成, 雛形作成, プロジェクト初期化, リポジトリ作成, テンプレート生成, scaffold, skeleton, init

## Instructions

### Phase 1: 要件確認と設計

プロジェクトの基本要件を確定する。

#### Step 1.1: プロジェクトパラメータの確認

```
【確認事項 — ユーザーから取得】

1. プロジェクト名（kebab-case推奨: my-awesome-project）
2. 言語/フレームワーク（Python, Java + Spring Boot, Java + Nablarch, TypeScript 等）
3. ビルドツール（Maven/Gradle, setuptools/poetry/hatch, npm/pnpm）
4. GitHubリポジトリの作成有無
   - 新規作成 → gh repo create
   - 既存リポジトリ → clone して構造追加
   - ローカルのみ → git init のみ
5. サブモジュール追加の有無（親リポジトリの指定）
6. ライセンス種別（MIT, Apache-2.0, 未指定）
7. CI/CD設定の有無（GitHub Actions）
8. 追加依存ライブラリ（Spring Boot Starter, MCP SDK 等）

【デフォルト値】
- ライセンス: MIT
- CI/CD: GitHub Actions（有効）
- .gitignore: 言語別自動選択
- README: 自動生成
```

#### Step 1.2: プロジェクト構造の設計

```
【出力パス確定】
- GitHubリポジトリ名: {project-name}
- ローカルパス: {output_dir}/{project-name}/
- サブモジュールの場合: {parent-repo}/{project-name}/

【ディレクトリ構造の選択】
- 言語/フレームワークに応じた標準テンプレートを選択
- Phase 3 のテンプレートセクションを参照
```

### Phase 2: GitHubリポジトリ作成

GitHub上にリポジトリを作成する（オプション）。

#### Step 2.1: リポジトリ作成

```
【gh CLIコマンド】

# パブリックリポジトリ
gh repo create {owner}/{project-name} --public --description "{description}" --clone

# プライベートリポジトリ
gh repo create {owner}/{project-name} --private --description "{description}" --clone

# ローカルリポジトリからの作成（既にディレクトリがある場合）
cd {project-dir}
git init
gh repo create {owner}/{project-name} --source=. --public --push

【確認事項】
- リポジトリ名の重複チェック: gh repo view {owner}/{project-name} 2>/dev/null
- 組織リポジトリの場合: gh repo create {org}/{project-name} --public
```

#### Step 2.2: 初期ブランチ設定

```
【実行手順】
1. デフォルトブランチを main に設定（通常gh repo createで自動設定）
2. .gitignore を言語別に配置
3. README.md の雛形を配置
4. LICENSE ファイルを配置
5. 初回コミット & プッシュ
```

### Phase 3: プロジェクト構造生成

言語・フレームワーク別のプロジェクトスケルトンを生成する。

#### Template 3.1: Python (pyproject.toml + src layout)

```
【ディレクトリ構造】

{project-name}/
├── pyproject.toml
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── src/
│   └── {package_name}/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_main.py
└── docs/
    └── .gitkeep

【pyproject.toml テンプレート】

[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "{project-name}"
version = "0.1.0"
description = "{description}"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.11"
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "ruff>=0.4",
    "mypy>=1.10",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=src/{package_name} --cov-report=term-missing"

[tool.ruff]
target-version = "py311"
line-length = 120
src = ["src"]

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "SIM"]

[tool.mypy]
python_version = "3.11"
strict = true

【テストスタブ — tests/test_main.py】

def test_placeholder():
    """Placeholder test — replace with actual tests."""
    assert True

【conftest.py】

import pytest

# Add shared fixtures here

【.gitignore — Python】

__pycache__/
*.py[cod]
*$py.class
*.egg-info/
dist/
build/
.eggs/
*.egg
.venv/
venv/
.env
.mypy_cache/
.pytest_cache/
.ruff_cache/
.coverage
htmlcov/
```

#### Template 3.2: Java (Maven + Spring Boot)

```
【ディレクトリ構造】

{project-name}/
├── pom.xml
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── {group-path}/
│   │   │       └── {artifact}/
│   │   │           └── Application.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── logback-spring.xml
│   └── test/
│       └── java/
│           └── {group-path}/
│               └── {artifact}/
│                   └── ApplicationTest.java
└── docs/
    └── .gitkeep

【pom.xml テンプレート — Spring Boot】

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
        <relativePath/>
    </parent>

    <groupId>{groupId}</groupId>
    <artifactId>{artifactId}</artifactId>
    <version>0.1.0-SNAPSHOT</version>
    <name>{project-name}</name>
    <description>{description}</description>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- Add additional starters here -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

【Application.java テンプレート】

package {groupId}.{artifactId};

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

【ApplicationTest.java テンプレート】

package {groupId}.{artifactId};

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ApplicationTest {
    @Test
    void contextLoads() {
    }
}

【.gitignore — Java/Maven】

target/
!.mvn/wrapper/maven-wrapper.jar
*.class
*.jar
*.war
*.ear
*.log
.idea/
*.iml
.vscode/
.settings/
.classpath
.project
.gradle/
build/
```

#### Template 3.3: Java (Maven + Nablarch)

```
【ディレクトリ構造】

{project-name}/
├── pom.xml
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── {group-path}/
│   │   │       └── {artifact}/
│   │   │           └── action/
│   │   │               └── SampleAction.java
│   │   └── resources/
│   │       ├── {app-type}-component-configuration.xml
│   │       ├── common.config
│   │       └── env.config
│   └── test/
│       ├── java/
│       │   └── {group-path}/
│       │       └── {artifact}/
│       │           └── action/
│       │               └── SampleActionTest.java
│       └── resources/
│           └── testdata/
│               └── .gitkeep
└── docs/
    └── .gitkeep

【pom.xml テンプレート — Nablarch】

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.nablarch.profile</groupId>
        <artifactId>nablarch-bom</artifactId>
        <version>6u3</version>
    </parent>

    <groupId>{groupId}</groupId>
    <artifactId>{artifactId}</artifactId>
    <version>0.1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>{project-name}</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Nablarch Core -->
        <dependency>
            <groupId>com.nablarch.framework</groupId>
            <artifactId>nablarch-fw-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.nablarch.framework</groupId>
            <artifactId>nablarch-common-dao</artifactId>
        </dependency>
        <!-- Test -->
        <dependency>
            <groupId>com.nablarch.framework</groupId>
            <artifactId>nablarch-testing-junit5</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

【注意】
- Nablarchプロジェクトは nablarch-bom を parent に指定
- アプリ種別（Web/REST/Batch）に応じた component-configuration.xml を配置
- 公式の nablarch-example-* リポジトリを参考にハンドラキュー構成を設定
```

#### Template 3.4: Java (Gradle + Spring Boot)

```
【ディレクトリ構造】
Java Maven版と同等。ビルドファイルのみ異なる。

【build.gradle.kts テンプレート】

plugins {
    java
    id("org.springframework.boot") version "3.3.0"
    id("io.spring.dependency-management") version "1.1.5"
}

group = "{groupId}"
version = "0.1.0-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test> {
    useJUnitPlatform()
}

【settings.gradle.kts テンプレート】

rootProject.name = "{project-name}"
```

#### Template 3.5: TypeScript / Node.js

```
【ディレクトリ構造】

{project-name}/
├── package.json
├── tsconfig.json
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── index.ts
├── tests/
│   └── index.test.ts
└── docs/
    └── .gitkeep

【package.json テンプレート】

{
  "name": "{project-name}",
  "version": "0.1.0",
  "description": "{description}",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src/",
    "format": "prettier --write ."
  },
  "engines": {
    "node": ">=20"
  },
  "license": "MIT",
  "devDependencies": {
    "typescript": "^5.4",
    "vitest": "^2.0",
    "eslint": "^9.0",
    "prettier": "^3.3"
  }
}

【tsconfig.json テンプレート】

{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "lib": ["ES2022"],
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}

【.gitignore — TypeScript/Node.js】

node_modules/
dist/
.env
*.log
.DS_Store
coverage/
```

#### Template 3.6: Go

```
【ディレクトリ構造】

{project-name}/
├── go.mod
├── main.go
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── internal/
│   └── .gitkeep
├── pkg/
│   └── .gitkeep
└── main_test.go

【go.mod テンプレート】

module github.com/{owner}/{project-name}

go 1.22

【main.go テンプレート】

package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}

【main_test.go テンプレート】

package main

import "testing"

func TestPlaceholder(t *testing.T) {
	// Replace with actual tests
}
```

#### Template 3.7: Rust

```
【ディレクトリ構造】

{project-name}/
├── Cargo.toml
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── main.rs
│   └── lib.rs
└── tests/
    └── integration_test.rs

【Cargo.toml テンプレート】

[package]
name = "{project-name}"
version = "0.1.0"
edition = "2021"
description = "{description}"
license = "MIT"

[dependencies]

[dev-dependencies]
```

### Phase 4: 共通設定ファイル生成

言語に依存しない共通ファイルを生成する。

#### Step 4.1: CI/CD設定（GitHub Actions）

```
【Python用 — .github/workflows/ci.yml】

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -e ".[dev]"
      - run: ruff check src/
      - run: mypy src/
      - run: pytest

【Java Maven用 — .github/workflows/ci.yml】

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - run: mvn verify --batch-mode

【TypeScript用 — .github/workflows/ci.yml】

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm test

【Go用 — .github/workflows/ci.yml】

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"
      - run: go vet ./...
      - run: go test -v ./...

【Rust用 — .github/workflows/ci.yml】

name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo fmt -- --check
      - run: cargo clippy -- -D warnings
      - run: cargo test
```

#### Step 4.2: README.md 生成

```
【README.md テンプレート】

# {project-name}

{description}

## Prerequisites

- {language} {version}+
- {build-tool} {version}+

## Getting Started

### Installation

{language-specific install instructions}

### Running

{language-specific run instructions}

### Testing

{language-specific test instructions}

## Project Structure

{tree output of the generated structure}

## License

{license-type}
```

#### Step 4.3: LICENSE ファイル

```
【MIT License テンプレート】
MIT License

Copyright (c) {year} {author}

Permission is hereby granted, free of charge, to any person obtaining a copy
...（標準MIT全文）

【Apache License 2.0】
gh repo createの--license apache-2.0 オプションで自動配置可能。
```

### Phase 5: サブモジュール追加（オプション）

生成したプロジェクトを既存リポジトリのサブモジュールとして追加する。

#### Step 5.1: サブモジュール登録

```
【実行手順】

1. 親リポジトリに移動
   cd {parent-repo}

2. サブモジュールとして追加
   git submodule add https://github.com/{owner}/{project-name}.git {project-name}

3. .gitmodules の確認
   cat .gitmodules

4. コミット
   git add .gitmodules {project-name}
   git commit -m "feat: add {project-name} as submodule"

5. プッシュ
   git push origin main
```

#### Step 5.2: サブモジュール初期化の案内

```
【他の開発者向け手順 — READMEに記載】

# クローン時にサブモジュールも取得
git clone --recurse-submodules https://github.com/{owner}/{parent-repo}.git

# 既存クローンにサブモジュールを追加
git submodule update --init --recursive
```

### Phase 6: 検証と最終確認

生成されたプロジェクト構造を検証する。

#### Step 6.1: 構造検証

```
【チェックリスト】
□ 全てのディレクトリが正しく作成されている
□ ビルドファイル（pom.xml / pyproject.toml / package.json等）が正しい
□ テストスタブが実行可能（mvn test / pytest / npm test）
□ .gitignore に必要なパターンが含まれている
□ CI/CDワークフローのシンタックスが正しい
□ README.md にプロジェクト情報が記載されている
□ LICENSE ファイルが存在する

【ビルド検証コマンド】
- Python: cd {project-dir} && pip install -e ".[dev]" && pytest
- Java Maven: cd {project-dir} && mvn verify
- Java Gradle: cd {project-dir} && ./gradlew build
- TypeScript: cd {project-dir} && npm install && npm test
- Go: cd {project-dir} && go test ./...
- Rust: cd {project-dir} && cargo test
```

#### Step 6.2: 最終コミット

```
【実行手順】
1. 全ファイルをステージング: git add -A
2. 初回コミット: git commit -m "feat: initial project structure"
3. プッシュ: git push origin main
4. リポジトリURLを報告
```

## Input Format

スキル実行時に以下のパラメータを指定する。

```yaml
# 必須パラメータ
project_name: "my-awesome-project"          # プロジェクト名（kebab-case）
language: "python"                           # python / java / typescript / go / rust
description: "A description of the project"  # 説明文

# オプションパラメータ
framework: "spring-boot"                     # spring-boot / nablarch / next / plain（デフォルト: plain）
build_tool: "maven"                          # maven / gradle / setuptools / poetry / npm / pnpm（自動選択）
java_version: "21"                           # Javaバージョン（デフォルト: 21）
python_version: "3.11"                       # Pythonバージョン（デフォルト: 3.11）
node_version: "20"                           # Node.jsバージョン（デフォルト: 20）
group_id: "com.example"                      # Java: groupId（デフォルト: com.example）
artifact_id: "my-project"                    # Java: artifactId（デフォルト: project_nameから生成）
package_name: "my_awesome_project"           # Python: パッケージ名（デフォルト: project_nameから生成）

# GitHub設定
github:
  create_repo: true                          # リポジトリ作成（デフォルト: true）
  owner: "my-org"                            # オーナー（デフォルト: 認証済みユーザー）
  visibility: "private"                      # public / private（デフォルト: private）

# サブモジュール設定
submodule:
  enabled: false                             # サブモジュールとして追加するか（デフォルト: false）
  parent_repo: ""                            # 親リポジトリの絶対パス
  parent_remote: ""                          # 親リポジトリのリモートURL

# 共通設定
license: "MIT"                               # MIT / Apache-2.0 / none（デフォルト: MIT）
ci_cd: true                                  # GitHub Actions CI/CD生成（デフォルト: true）
output_dir: "."                              # 出力先ディレクトリ（デフォルト: カレント）
```

### パラメータ説明

| パラメータ | 必須 | デフォルト | 説明 |
|----------|------|----------|------|
| `project_name` | yes | -- | プロジェクト名（kebab-case） |
| `language` | yes | -- | 対象言語 |
| `description` | yes | -- | プロジェクト説明 |
| `framework` | -- | `plain` | フレームワーク指定 |
| `build_tool` | -- | 自動選択 | ビルドツール |
| `github.create_repo` | -- | `true` | GitHubリポジトリ作成 |
| `github.visibility` | -- | `private` | リポジトリの可視性 |
| `submodule.enabled` | -- | `false` | サブモジュール追加 |
| `license` | -- | `MIT` | ライセンス種別 |
| `ci_cd` | -- | `true` | CI/CD生成 |

## Output Format

### 成功時の出力

```
## Project Scaffold Complete

**Project**: {project-name}
**Language**: {language} ({framework})
**Repository**: https://github.com/{owner}/{project-name}
**Local Path**: {output_dir}/{project-name}

### Generated Structure
{tree output}

### Next Steps
1. cd {project-name}
2. {language-specific setup command}
3. {language-specific run command}
4. {language-specific test command}
```

## Examples

### Example 1: Python プロジェクト（標準構成）

```
入力:
  project_name: "nablarch-mcp-proxy"
  language: "python"
  description: "MCP proxy server for Nablarch applications"
  github:
    create_repo: true
    owner: "kumagai-group"
    visibility: "private"
  ci_cd: true

Phase 1: 要件確認
  - Python 3.11+, pyproject.toml (setuptools)
  - src layout

Phase 2: GitHub リポジトリ作成
  gh repo create kumagai-group/nablarch-mcp-proxy --private --clone

Phase 3: 構造生成
  Template 3.1 (Python) を適用
  - pyproject.toml に pytest, ruff, mypy を dev依存に追加
  - src/nablarch_mcp_proxy/__init__.py, main.py 生成
  - tests/test_main.py テストスタブ生成

Phase 4: 共通設定
  - .github/workflows/ci.yml (Python用)
  - README.md, LICENSE (MIT), .gitignore (Python)

Phase 5: スキップ（サブモジュール不要）

Phase 6: 検証
  pip install -e ".[dev]" && pytest → 成功
  git add -A && git commit && git push
```

### Example 2: Java Spring Boot + MCP SDK

```
入力:
  project_name: "nablarch-mcp-server"
  language: "java"
  framework: "spring-boot"
  build_tool: "maven"
  description: "MCP server for Nablarch framework knowledge"
  group_id: "com.example.nablarch"
  java_version: "21"
  github:
    create_repo: true
    owner: "kumagai-group"
  submodule:
    enabled: true
    parent_repo: "/home/kuma/multi-agent-shogun"

Phase 1: 要件確認
  - Java 21, Maven, Spring Boot 3.3
  - 追加依存: spring-ai-mcp-server-spring-boot-starter

Phase 2: GitHub リポジトリ作成
  gh repo create kumagai-group/nablarch-mcp-server --private --clone

Phase 3: 構造生成
  Template 3.2 (Java Maven + Spring Boot) を適用
  - pom.xml に MCP SDK 依存を追加
  - Application.java, ApplicationTest.java 生成

Phase 4: 共通設定
  - .github/workflows/ci.yml (Java Maven用)
  - README.md, LICENSE, .gitignore

Phase 5: サブモジュール追加
  cd /home/kuma/multi-agent-shogun
  git submodule add https://github.com/kumagai-group/nablarch-mcp-server.git

Phase 6: 検証
  mvn verify → 成功
```

### Example 3: TypeScript Node.jsプロジェクト

```
入力:
  project_name: "skill-analyzer"
  language: "typescript"
  description: "Skill analysis tool for multi-agent-shogun"
  github:
    create_repo: false  # ローカルのみ
  output_dir: "/home/kuma/projects"

Phase 1: 要件確認
  - Node.js 20+, npm, TypeScript 5.4+

Phase 2: スキップ（ローカルのみ）
  cd /home/kuma/projects && mkdir skill-analyzer && cd skill-analyzer && git init

Phase 3: 構造生成
  Template 3.5 (TypeScript/Node.js) を適用
  - package.json (vitest, eslint, prettier)
  - tsconfig.json (strict mode)
  - src/index.ts, tests/index.test.ts

Phase 4: 共通設定
  - .github/workflows/ci.yml (TypeScript用)
  - README.md, LICENSE, .gitignore

Phase 5: スキップ

Phase 6: 検証
  npm install && npm test → 成功
```

## Guidelines

### 必須ルール

1. **生成前にユーザー確認を取ること**
   - 言語/フレームワーク/ビルドツールの選択は必ず確認
   - デフォルト値で進める場合もその旨を通知

2. **テストスタブは必ず実行可能であること**
   - 生成したテストが `pass` するか検証
   - プレースホルダーテスト（`assert True`等）でもテストランナーが正常動作すること

3. **CI/CDワークフローはシンタックスが正しいこと**
   - GitHub Actions のYAMLフォーマットに準拠
   - 使用するアクションのバージョンは最新安定版

4. **パッケージ名・モジュール名の命名規則を遵守すること**
   - Python: snake_case (`my_awesome_project`)
   - Java: ドメイン逆順 (`com.example.myproject`)
   - TypeScript: kebab-case (`my-awesome-project`)
   - Go: 小文字 (`myproject`)

5. **機密情報をリポジトリに含めないこと**
   - .env ファイルは .gitignore に含める
   - 認証情報のプレースホルダーは環境変数参照にする

6. **既存リポジトリを上書きしないこと**
   - gh repo create 前に重複チェック
   - ローカルディレクトリが既に存在する場合はエラーにする

### 拡張方法（新言語/フレームワーク追加）

```
新しい言語・フレームワークのテンプレートを追加する手順:

1. Phase 3 に新しい Template セクション (3.N) を追加
   - ディレクトリ構造
   - ビルドファイルのテンプレート
   - エントリーポイントファイル
   - テストスタブ
   - .gitignore パターン

2. Phase 4 の CI/CD テンプレートに言語用ワークフローを追加

3. Input Format の language 選択肢に追加

4. Examples に新言語の例を追加
```

### アンチパターン（避けるべきこと）

- **確認なしのリポジトリ作成**: publicリポジトリを確認なしに作成
- **テスト未検証**: テストスタブが実行できない状態で納品
- **ハードコードされたバージョン**: バージョンはパラメータで指定可能にする
- **.gitignore の欠如**: ビルド成果物がリポジトリに混入
- **CI/CDなし**: 最低限のCIワークフローは必ず含める
- **過剰な依存追加**: 最小構成から始め、必要に応じて追加する方針
- **不要なディレクトリ**: docs/ 等、使用しないディレクトリの乱造（.gitkeepで最小限に）
