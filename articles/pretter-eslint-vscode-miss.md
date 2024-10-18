---
title: "vscodeでprettierとeslintを設定しているディレクトリ以外のディレクトリーでも走ってしまう"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "typescript", "javascript", "prettier", "eslint"]
published: false
---

# 前提

- frontend(nextjs)と backend(nest)で異なるリポジトリーで管理をしている。
- 異なる repository で異なる eslint,prettier を設定
- vscode は frontend と backend ディレクトリーの親ディレクトリーを開いている

```bash
.
├── backend
│   ├── .eslintrc.js
│   └── .prettierrc
└── frontend
    ├── .eslintrc
    ├── .prettierignore
    └── .prettierrc
```

vscode(一部抜粋)
![vscode](/images/pretter-eslint-vscode-miss/vscode.png)

# 問題

- backend の eslint、prettier の設定が反映されず、frontend の eslint、prettier の設定が反映されてしまった。
- frontend の prettierignore がうまく設定できない

# 解決方法

- frontend と backend を異なる vscode タブで開く

# 原因

- 最初の.prettierrc や.eslintrc などの設定ファイルを置くことで、VSCode とそれぞれの拡張機能がこれらの設定を認識し、プロジェクト全体(vscode で開いているディレクリー)にわたってコードのフォーマットやリントを反映するため、backend のディレクトリー内も frontend の eslint,prettier の設定の可能性がある

# まとめ

どうしても git の repository を分けてるとこのような問題が発生してしまう。そろそろ monorepo を導入しようかな~
https://zenn.dev/burizae/articles/c811cae767965a
