---
title: "CIでやりたい最低限のNodeセキュリティ対策"
emoji: "🐿️"
type: "tech"
topics: [nodejs, security, frontend, typescript, javascript]
publication_name: nislab
published: true
---

# 私たちの研究室(NISLab)

https://nisk.doshisha.ac.jp/

# アドベントカレンダー 5 日目~

https://nislab-advendcallender-2025.vercel.app/

# 背景

Node.js プロジェクト（React / Next.js / NestJS など）は、依存パッケージが多く更新も頻繁なため、脆弱性が混入しやすいという弱点があります。また、最近はサプライチェーン攻撃が増えており、自分のコード以外の部分からリスクが発生することも珍しくありません。

こうした問題に早く気づくためには、手動チェックではなく CI にセキュリティ診断を組み込み、自動で検出する仕組みが必要です。

本記事では、Node.js プロジェクトで 最低限 CI に入れておきたいセキュリティチェックを、役割ごとにわかりやすく紹介します。

# セキュリティ診断ツールの全体像

セキュリティ診断ツールは、大きく 3 つのカテゴリに分類されます。

| カテゴリ                  | 対象                            | ツール例                     |
| ------------------------- | ------------------------------- | ---------------------------- |
| SCA（依存パッケージ診断） | Node プロジェクト               | Snyk、OSV-Scanner、npm audit |
| SAST（静的コード解析）    | Node プロジェクト               | Semgrep                      |
| シークレット検出          | リポジトリ全体（言語非依存）    | gitleaks                     |
| docker image の診断       | Node のプロジェクト(Dockerfile) | Snyk、Trivy                  |

## 1. 依存パッケージの脆弱性診断（SCA）

### 対象ツール：Snyk、OSV-Scanner、npm audit

**SCA（Software Composition Analysis）** は、`package.json` や `package-lock.json` を解析し、依存パッケージに既知の脆弱性（CVE）がないかチェックします。
CI では OSV-Scanner をベースにして、さらに必要なら Snyk を足す。npm audit はローカルの軽いチェック or サブとして運用みたい形がいいのかなと思います。

## 2. 静的セキュリティチェック（SAST）

### 対象ツール：Semgrep

**SAST（Static Application Security Testing）** は、ソースコードをパターンマッチでスキャンし、セキュリティリスクを検出します。

### Semgrep が検出する主な脆弱性

- XSS（クロスサイトスクリプティング）につながる危険なコード
- SQL インジェクションの可能性がある生のクエリ
- 安全でない関数の使用（`eval()` など）
- ハードコードされた認証情報

## 3. シークレット検出

### 対象ツール：gitleaks

**gitleaks** は、リポジトリ全体を対象に、シークレット情報の漏洩を検出します。

### gitleaks が検出する対象

- AWS アクセスキー（`AKIA...`）
- GitHub Personal Access Token（`ghp_...`）
- 秘密鍵（`-----BEGIN PRIVATE KEY-----`）
- API キー、パスワード、データベース接続文字列
- `.env` ファイルの認証情報
- config ファイル内のシークレット
- 過去のコミット履歴に含まれるシークレット

## 4. Docker イメージのセキュリティ診断

### 対象ツール：Snyk、Trivy

Docker イメージに含まれる依存パッケージや OS パッケージの脆弱性を検出します。
近年は ECS（AWS）や Cloud Run などのコンテナ実行基盤の利用が一般化 しており、
アプリケーションをコンテナとして本番環境にデプロイするケースが増えています。
そのため、デプロイ前にイメージ自体をスキャンして安全性を確認することが大事になります。
Snyk や Trivy を CI に組み込むことで、
本番環境へ脆弱なコンテナがデプロイされるリスクを事前に防ぐことができます。

## 参考

自分が使っている github action の例を一部載せさせていただきます.

```bash
name: Security CI

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read
  pull-requests: read
  security-events: write
  actions: read
env:
  NODE_VERSION: "20"
  SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}

jobs:
  deps_audit: # ← job_id は英字で開始・英数/ _ / - のみ
    name: Dependencies (npm audit / OSV)
    runs-on: ubuntu-latest # ← これが必須
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install
        run: npm ci

      - name: npm audit (fail on high)
        run: npm audit --audit-level=high

      - name: OSV-Scanner
        uses: google/osv-scanner-action/osv-scanner-action@v2.2.3
        with:
          scan-args: |-
            -r
            .
      - name: Snyk Open Source scan (optional)
        if: ${{ env.SNYK_TOKEN != '' }}
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ env.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --file=package.json

  semgrep:
    name: Semgrep (AppSec static analysis)
    runs-on: ubuntu-latest # ← 必須
    steps:
      - uses: actions/checkout@v4
      - uses: returntocorp/semgrep-action@v1
        with:
          generateSarif: true
          config: >
            p/owasp-top-ten
            p/javascript
            p/ci
      - name: Upload SARIF to code scanning
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif
        continue-on-error: true
  gitleaks:
    name: Secret leaks (Gitleaks)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_LICENSE: ${{ env.GITLEAKS_LICENSE }}
          GITLEAKS_REPORT_PATH: gitleaks.sarif
          GITLEAKS_FORMAT: sarif
          GITLEAKS_FAIL: true
      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: gitleaks.sarif
        continue-on-error: true

  trivy:
    name: Trivy (fs + image)
    runs-on: ubuntu-latest # ← 必須
    steps:
      - uses: actions/checkout@v4

      - name: Trivy FS scan (High以上でfail)
        uses: aquasecurity/trivy-action@0.24.0
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "table"
          exit-code: "1"
          severity: "HIGH,CRITICAL"
          ignore-unfixed: true

      - name: Build backend image
        if: startsWith(github.ref, 'refs/tags/') || endsWith(github.ref, '/release')
        run: dock

```
